---
title: 트랜잭션과 ACID
date: 2025-03-01 00:00:00
categories: [CS]
tags: [CS, DB, Transaction]
---

## **DB 부분 성공/실패 시나리오**

당신은 이커머스(E-commerce)의 결제 시스템을 개발하고 있다.
고객이 상품을 주문하면 다음과 같은 작업들이 순차적으로 이루어진다:

1. 상품 재고 감소
2. 고객 계좌에서 금액 차감 (출금)
3. 판매자 계좌에 금액 추가 (입금)
4. 주문 정보 DB에 저장
5. 배송 정보 생성

모든 과정이 순조롭게 진행되고 문제가 발생하지 않아 보였다.
하지만 어느날, 시스템에 다음과 같은 상황이 발생했다:

> 고객 A는 10만원짜리 상품을 주문했습니다.
> 시스템은 재고를 차감하고, 고객 계좌에서 10만원을 인출했습니다.
> 그런데 네트워크 오류로 인해 판매자 계좌에 금액이 입금되지 않았고, 주문 정보도 저장되지 않았습니다.

결과적으로 고객은 돈을 지불했지만 주문은 기록되지 않았고 재고만 감소했다.
고객은 정정 및 환불을 요청하지만 시스템에는 관련된 주문 기록이 존재하지 않는다.
재고 관리자는 실제 재고와 시스템상의 재고가 일치하지 않아 혼란스럽다.

이러한 상황을 해결하기 위해 개발자는 여러 예외 상황을 처리하는 복잡한 코드를 작성해야 한다:

```java
public void processOrder(Order order) {
    try {
        inventoryService.reduceStock(...); // 재고 감소
        try {
            paymentService.debitFromCustomer(...); // 출금
            try {
                paymentService.creditToSeller(...); // 입금
                try {
                    orderRepository.save(order); // 주문 저장
                    try {
                        shippingService.createShippingInfo(order); // 주문 정보 생성
                    } catch (Exception e) {
                        // 배송 정보 생성 실패 예외 처리
                        orderRepository.delete(order);
                        paymentService.refundToSeller(...);
                        paymentService.refundToCustomer(...);
                        inventoryService.increaseStock(...);
                        throw e;
                    }
                } catch (Exception e) {
                    // 주문 저장 실패 예외 처리
                    paymentService.refundToSeller(...);
                    paymentService.refundToCustomer(...);
                    inventoryService.increaseStock(...);
                    throw e;
                }
            } catch (Exception e) {
                // 판매자 입금 실패 예외 처리
                paymentService.refundToCustomer(...);
                inventoryService.increaseStock(...);
                throw e;
            }
        } catch (Exception e) {
            // 고객 출금 실패 예외 처리
            inventoryService.increaseStock(...);
            throw e;
        }
    } catch (Exception e) {
        // 재고 감소 실패 예외 처리
        throw e;
    }
}
```

실패 시 예외를 처리하기 위한 중첩된 `try-catch` 문으로 인해 코드의 가독성이 떨어지고, 유지보수가 어려우며, 실수할 가능성이 매우 높다.
게다가 시스템 장애나 전원의 손실이 발생하면 예외 처리 자체가 실행되지 않을 수도 있다.

## **트랜잭션(Transaction)**

이런 상황에서 필요한 것이 바로 **트랜잭션**이다.
트랜잭션은 DB의 상태를 변화시키는 하나의 논리적 작업 단위이다.
여러 개의 작업을 마치 하나의 작업처럼 처리하여, 모든 작업이 성공적으로 완료되거나 아니면 전혀 실행되지 않은 상태로 유지되도록 보장한다.

간단한 예시로 은행 계좌 간 송금을 생각해보자:

1. A 계좌에서 10만원 출금
2. B 계좌에 10만원 입금

이 두 작업은 반드시 함께 성공하거나 함께 실패해야 한다.
만약 첫 번째 작업만 성공하고 두 번째 작업은 실패한다면 매우 심각한 문제가 발생한다.
트랜잭션은 이러한 상황을 방지하기 위해 존재한다.

## **ACID 원칙: 트랜잭션의 네 가지 핵심**

ACID는 데이터베이스 트랜잭션이 안전하게 수행된다는 것을 보장하기 위한 네 가지 속성의 두문자어이다.
다음으로는 ACID의 각 속성을 알아보겠다.

### **원자성(Atomicity)**

> **원자성(Atomicity)**: 트랜잭션의 모든 작업은 전부 성공하거나 전부 실패해야 합니다. 부분적 성공은 허용되지 않습니다.
{: .prompt-info }
tmp
tmp

### **일관성(Consistency)**

> **일관성(Consistency)**: 트랜잭션이 완료된 후에도 데이터베이스는 일관된 상태를 유지해야 합니다. 모든 데이터는 정의된 규칙과 제약조건을 만족해야 합니다.
{: .prompt-info }

### **격리성(Isolation)**

> **격리성(Isolation)**: 동시에 실행되는 트랜잭션들은 서로 영향을 미치지 않아야 합니다. 각 트랜잭션은 다른 트랜잭션의 중간 상태를 볼 수 없어야 합니다.
{: .prompt-info }

### **지속성(Durability)**

> **지속성(Durability)**: 트랜잭션이 성공적으로 완료(커밋)된 후에는 시스템 장애가 발생하더라도 그 결과가 영구적으로 보존되어야 합니다.
{: .prompt-tip }

## **트랜잭션 관리의 실제 구현**

### **Spring Framework에서의 트랜잭션 관리**

### **트랜잭션 속성 설정**

## **트랜잭션 관련 실수와 해결책**

### **트랜잭션 경계 설정 오류**

### **트랜잭션 내 과도한 외부 시스템 호출**

## **결론: 트랜잭션과 ACID의 중요성**
