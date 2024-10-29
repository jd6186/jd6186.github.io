---
layout: post
title: "자바, Spring Boot에서 트랜잭션 관리하는 방법(결제 프로세스 예시)"
tags: [BackEnd, Java, SpringBoot, Transaction, Payment, Order]
---

# Intro
안녕하세요 Noah입니다.

오늘은 Spring Boot에서 트랜잭션 관리하는 방법에 대해 알아보겠습니다.

트랜잭션은 데이터베이스 연산이 여러 작업으로 구성되어 있을 때, 전체 작업이 성공해야만 변경 사항을 반영하고, 실패 시에는 이전 상태로 복구(rollback)하여 데이터 일관성을 유지하는 중요한 역할을 합니다.

Spring Boot는 이런 트랜잭션을 쉽게 관리할 수 있는 `@Transactional` 애너테이션을 제공합니다.

아래에서 `@Transactional`을 사용하여 트랜잭션을 설정하고 사용하는 예제와 함께 트랜잭션 관리 방법을 자세히 설명하겠습니다.

이제부터 시작해보겠습니다.
<br/><br/><br/><br/>



# 목차
* [Intro](#Intro)
* [기본 Transaction 관리 방법](#기본-Transaction-관리-방법)
  1. [@Transactional 기본 설정](#1-Transactional-기본-설정)
  2. [트랜잭션 속성 설정](#2-트랜잭션-속성-설정)
  3. [Isolation Level 설정 예시](#3-Isolation-Level-설정-예시)
  4. [트랜잭션 테스트 및 롤백 확인](#4-트랜잭션-테스트-및-롤백-확인)
* [결제 프로세스에서 트랜잭션 관리해보기](#결제-프로세스에서-트랜잭션-관리해보기)
  1. [결제와 관련된 테이블 설계](#1-결제와-관련된-테이블-설계)
  2. [트랜잭션을 구분하여 관리하는 이유](#2-트랜잭션을-구분하여-관리하는-이유)
  3. [트랜잭션 관리 코드 예시](#3-트랜잭션-관리-코드-예시)
  4. [트랜잭션 흐름 요약](#4-트랜잭션-흐름-요약)
* [Outro](#Outro)
<br/><br/><br/><br/>


## 기본 Transaction 관리 방법
### 1. @Transactional 기본 설정
`@Transactional` 애너테이션을 메서드 또는 클래스에 선언하면 해당 범위에서 트랜잭션이 관리됩니다. 일반적으로, 서비스 레이어의 메서드에 적용하여 데이터베이스 관련 작업이 모두 성공해야만 변경 사항을 적용합니다.

```java
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

@Service
public class UserService {

    private final UserRepository userRepository;
    private final OrderRepository orderRepository;

    public UserService(UserRepository userRepository, OrderRepository orderRepository) {
        this.userRepository = userRepository;
        this.orderRepository = orderRepository;
    }

    @Transactional
    public void createUserAndPlaceOrder(User user, Order order) {
        userRepository.save(user);      // 사용자 생성
        orderRepository.save(order);    // 주문 생성
    }
}

```
<br/><br/>

### 2. 트랜잭션 속성 설정
`@Transactional` 애너테이션에는 트랜잭션 동작을 제어할 수 있는 여러 속성이 있습니다.

- **Propagation**: 트랜잭션 전파 방식을 설정합니다. (기본값은 `REQUIRED`로, 기존 트랜잭션이 있으면 참여하고 없으면 새로 생성합니다.)
- **Isolation**: 트랜잭션 격리 수준을 설정하여 동시에 여러 트랜잭션이 실행될 때 일관성 문제를 제어합니다.
- **RollbackFor**: 특정 예외가 발생했을 때 롤백할지 설정합니다.
- **Timeout**: 트랜잭션 타임아웃 시간을 설정합니다.

아래 예제에서는 트랜잭션 전파와 롤백 조건을 설정합니다.

```java
@Transactional(propagation = Propagation.REQUIRED, rollbackFor = Exception.class)
public void createUserAndPlaceOrder(User user, Order order) throws Exception {
    userRepository.save(user);      // 사용자 생성
    if (order.getAmount() < 0) {
        throw new Exception("Order amount cannot be negative");
    }
    orderRepository.save(order);    // 주문 생성
}

```

이 경우 `order.getAmount()`가 0보다 작으면 예외가 발생하며, `userRepository.save(user)`에서 저장한 사용자 데이터는 롤백됩니다.
<br/><br/>

### 3. Isolation Level 설정 예시

트랜잭션 격리 수준을 설정하여 트랜잭션 간 데이터 일관성 문제를 제어할 수 있습니다. 예를 들어, `Isolation.SERIALIZABLE`로 설정하면 동시에 같은 데이터를 읽거나 쓸 수 없도록 가장 높은 수준의 일관성을 제공합니다.

```java
@Transactional(isolation = Isolation.SERIALIZABLE)
public void createUserAndPlaceOrder(User user, Order order) {
    userRepository.save(user);
    orderRepository.save(order);
}

```
<br/><br/>

### 4. 트랜잭션 테스트 및 롤백 확인

Spring Boot에서 테스트 중에도 `@Transactional`을 사용할 수 있으며, 기본적으로 테스트 완료 후 트랜잭션이 롤백되어 데이터베이스 상태가 원상 복구됩니다.

```java
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.transaction.annotation.Transactional;

@SpringBootTest
public class UserServiceTest {

    @Autowired
    private UserService userService;

    @Test
    @Transactional
    public void testCreateUserAndPlaceOrder() {
        User user = new User("John", "Doe");
        Order order = new Order(100);

        userService.createUserAndPlaceOrder(user, order);
        // 테스트가 끝나면 데이터베이스는 롤백됩니다.
    }
}

```

이 테스트 코드는 `@Transactional`로 설정되어 테스트가 끝나면 자동으로 롤백되므로 데이터베이스에 실제로 변경이 반영되지 않습니다.
<br/><br/><br/><br/>

## 결제 프로세스에서 트랜잭션 관리해보기

결제 프로세스를 설계하고 구현할 때는 결제의 복잡성을 고려하여 트랜잭션을 신중하게 관리해야 합니다.

결제는 다수의 단계와 관련 테이블을 통해 이루어지며, 모든 작업이 성공적으로 처리되어야 사용자와 시스템 모두의 데이터 일관성을 유지할 수 있습니다.
<br/><br/>

### 1. 결제와 관련된 테이블 설계
먼저 결제 프로세스에서 필요한 주요 테이블들을 설계해 보겠습니다. 일반적으로 결제 관련 테이블은 사용자, 주문, 결제, 결제 상태, 포인트/할인 정보 등을 포함합니다.

#### 테이블 설계
1. **Users** (사용자 정보)
  - `user_id` (PK): 사용자 ID
  - `name`: 사용자 이름
  - `email`: 사용자 이메일
  - `balance`: 사용자 잔액 또는 포인트
2. **Orders** (주문 정보)
  - `order_id` (PK): 주문 ID
  - `user_id` (FK): 사용자 ID
  - `total_amount`: 총 주문 금액
  - `discount_id` (nullable FK): 적용된 할인 ID
  - `created_at`: 주문 생성 일자
  - `status`: 주문 상태 (예: "PENDING", "PAID", "CANCELLED")
3. **Payments** (결제 정보)
  - `payment_id` (PK): 결제 ID
  - `order_id` (FK): 주문 ID
  - `user_id` (FK): 사용자 ID
  - `amount`: 결제 금액
  - `status`: 결제 상태 (예: "PENDING", "COMPLETED", "FAILED")
  - `payment_method`: 결제 수단 (카드, 포인트, 쿠폰 등)
  - `created_at`: 결제 시도 일자
4. **Discounts** (할인/쿠폰 정보)
  - `discount_id` (PK): 할인 ID
  - `discount_type`: 할인 종류 (예: "COUPON", "POINTS")
  - `amount`: 할인 금액
  - `expiry_date`: 할인 만료일자
5. **TransactionLogs** (트랜잭션 로그)
  - `transaction_id` (PK): 트랜잭션 ID
  - `payment_id` (FK): 결제 ID
  - `transaction_type`: 트랜잭션 유형 (예: "PAYMENT", "REFUND")
  - `amount`: 트랜잭션 금액
  - `status`: 트랜잭션 상태 (성공, 실패)
  - `created_at`: 트랜잭션 생성 일자
    <br/><br/>

### 2. 트랜잭션을 구분하여 관리하는 이유
결제는 여러 단계로 나뉘며, 각 단계가 성공적으로 처리되지 않으면 이후의 작업을 중단하고 이전 상태로 되돌려야 합니다. 이러한 트랜잭션 관리는 데이터의 일관성 및 결제 실패 시 복구를 위해 필수적입니다.

#### 트랜잭션 구분 방식
1. **주문 생성 트랜잭션**
  - **위치**: `OrderService`의 `createOrder` 메서드
  - **설명**: 사용자가 결제를 진행하기 전에 주문을 생성하여 총 금액, 할인 정보를 저장합니다.
  - **이유**: 결제와 주문이 서로 독립적이므로, 주문을 먼저 생성해두면 결제 실패 시에도 주문 정보를 유지할 수 있습니다.
2. **결제 초기화 트랜잭션**
  - **위치**: `PaymentService`의 `initiatePayment` 메서드
  - **설명**: 결제를 시작하며 결제 상태를 "PENDING"으로 설정합니다. 이때 포인트 또는 할인 금액을 포함한 최종 결제 금액을 계산합니다.
  - **이유**: 결제를 초기화하여 결제 금액 및 할인을 미리 계산하고 검증할 수 있습니다. 결제가 실패하면 `PENDING` 상태에서 `FAILED`로 상태를 변경하고 주문 상태도 업데이트합니다.
3. **결제 승인 및 트랜잭션 처리 트랜잭션**
  - **위치**: `PaymentService`의 `completePayment` 메서드
  - **설명**: 결제가 성공되면 결제 상태를 "COMPLETED"로 변경하고, `TransactionLogs` 테이블에 트랜잭션 로그를 생성합니다.
  - **이유**: 결제 시스템과 상호작용하여 결제가 성공했는지 확인한 후, 트랜잭션 로그를 생성하여 결제 기록을 남깁니다. 이 단계는 주문과 결제 상태를 최종 확정하는 단계이므로, 모든 트랜잭션이 성공해야만 `COMPLETED`로 설정합니다.
4. **주문 및 결제 최종 확정 트랜잭션**
  - **위치**: `OrderService`의 `finalizeOrder` 메서드
  - **설명**: 결제가 완료되면 주문 상태를 "PAID"로 변경하고, 필요한 경우 할인 또는 포인트를 차감합니다.
  - **이유**: 최종 결제 성공 이후에 주문을 "PAID"로 설정하여 데이터 일관성을 보장합니다. 할인이 제대로 적용되지 않거나 포인트가 올바르게 차감되지 않으면 트랜잭션이 롤백됩니다.
    <br/><br/>

### 3. 트랜잭션 관리 코드 예시
아래는 각 단계별로 트랜잭션을 분리하여 결제를 구현한 예제 코드입니다.

```java
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

@Service
public class OrderService {
    private final OrderRepository orderRepository;
    private final PaymentService paymentService;

    public OrderService(OrderRepository orderRepository, PaymentService paymentService) {
        this.orderRepository = orderRepository;
        this.paymentService = paymentService;
    }

    // 1. 주문 생성
    @Transactional
    public Order createOrder(User user, double totalAmount, Long discountId) {
        Order order = new Order(user, totalAmount, discountId, "PENDING");
        return orderRepository.save(order);
    }

    // 4. 주문 최종 확정
    @Transactional
    public void finalizeOrder(Order order) {
        order.setStatus("PAID");
        orderRepository.save(order);
    }
}

@Service
public class PaymentService {
    private final PaymentRepository paymentRepository;
    private final TransactionLogRepository transactionLogRepository;

    public PaymentService(PaymentRepository paymentRepository, TransactionLogRepository transactionLogRepository) {
        this.paymentRepository = paymentRepository;
        this.transactionLogRepository = transactionLogRepository;
    }

    // 2. 결제 초기화
    @Transactional
    public Payment initiatePayment(Order order, double finalAmount, String paymentMethod) {
        Payment payment = new Payment(order, order.getUser(), finalAmount, "PENDING", paymentMethod);
        return paymentRepository.save(payment);
    }

    // 3. 결제 승인 및 트랜잭션 처리
    @Transactional
    public void completePayment(Payment payment) {
        payment.setStatus("COMPLETED");
        paymentRepository.save(payment);

        // 트랜잭션 로그 생성
        TransactionLog log = new TransactionLog(payment, "PAYMENT", payment.getAmount(), "SUCCESS");
        transactionLogRepository.save(log);
    }
}

```
<br/><br/>

### 4. 트랜잭션 흐름 요약
1. **주문 생성**: 주문 정보를 데이터베이스에 저장합니다. 이 단계에서는 사용자가 결제할 준비 상태로 `PENDING` 상태의 주문을 생성합니다.
2. **결제 초기화**: 결제 금액을 계산하여 `PENDING` 상태의 결제를 생성합니다. 이 과정에서 포인트나 쿠폰을 검증합니다.
3. **결제 승인 및 트랜잭션 처리**: 결제 시스템과 상호작용하여 결제를 완료합니다. 이때 결제가 성공하면 트랜잭션 로그를 남깁니다.
4. **주문 및 결제 최종 확정**: 결제가 정상적으로 완료되었을 때 주문 상태를 "PAID"로 변경하고 포인트나 쿠폰을 차감하여 최종 결제를 확정합니다.

이렇게 설계하면 각 단계의 트랜잭션이 독립적으로 관리되므로 특정 단계에서 문제가 발생해도 그 이전 단계의 데이터가 안전하게 보호됩니다.

하지만 모든 상황에서 저렇게 간단하게 처리되진 않겠죠? 아래 예시를 통해 복잡한 트랜잭션을 살펴보겠습니다.
<br/><br/><br/><br/>


## 복잡한 Transaction 관리 방법
복잡한 트랜잭션 처리는 주로 **하나의 API가 여러 트랜잭션을 필요로 할 때** 발생합니다. 

예를 들어, 결제 API에서는 사용자의 포인트, 할인 쿠폰, 결제 상태 업데이트, 결제 시스템 호출 등 여러 단계가 필요하며, 각 단계가 개별 트랜잭션으로 처리될 수 있습니다. 

이런 복잡한 프로세스에서는 전체 트랜잭션 관리가 매우 중요하며, 한 단계라도 실패하면 모든 작업을 롤백해야 합니다.

여기서는 예시로 **포인트 결제와 외부 결제 시스템 호출을 포함한 복잡한 결제 API**를 설계해 보겠습니다. 

각 단계별로 트랜잭션이 진행되며, 일부 작업은 독립 트랜잭션으로 처리되어야 하고, 일부는 전체 트랜잭션으로 관리됩니다.
<br/><br/>

### 1. 예제 시나리오: 결제 API의 복잡한 트랜잭션 처리
**결제 API 단계:**

1. 사용자 포인트 차감
2. 할인 쿠폰 적용 여부 확인 및 차감
3. 결제 요청 상태를 "PENDING"으로 설정
4. 외부 결제 시스템에 요청 (실패 가능성이 존재)
5. 결제 성공 시 결제 완료 상태 "COMPLETED"로 설정하고 트랜잭션 로그 기록

각 단계는 트랜잭션 관리가 필요하며, 특히 외부 결제 시스템에서 실패 시에는 모든 작업을 롤백해야 합니다.
<br/><br/>

### 2. 코드 예시: 결제 API의 트랜잭션 처리
```java
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

@Service
public class PaymentService {

    private final UserRepository userRepository;
    private final OrderRepository orderRepository;
    private final PaymentRepository paymentRepository;
    private final TransactionLogRepository transactionLogRepository;
    private final ExternalPaymentGateway externalPaymentGateway;

    public PaymentService(UserRepository userRepository,
                          OrderRepository orderRepository,
                          PaymentRepository paymentRepository,
                          TransactionLogRepository transactionLogRepository,
                          ExternalPaymentGateway externalPaymentGateway) {
        this.userRepository = userRepository;
        this.orderRepository = orderRepository;
        this.paymentRepository = paymentRepository;
        this.transactionLogRepository = transactionLogRepository;
        this.externalPaymentGateway = externalPaymentGateway;
    }

    @Transactional
    public void processPayment(Long userId, Long orderId, double paymentAmount, Long discountId) {
        // 1. 사용자 포인트 차감
        User user = userRepository.findById(userId).orElseThrow(() -> new IllegalArgumentException("User not found"));
        if (user.getBalance() < paymentAmount) {
            throw new IllegalArgumentException("Insufficient balance");
        }
        user.setBalance(user.getBalance() - paymentAmount);
        userRepository.save(user);

        // 2. 할인 쿠폰 적용 및 차감
        if (discountId != null) {
            Discount discount = discountRepository.findById(discountId).orElseThrow(() -> new IllegalArgumentException("Discount not found"));
            if (discount.isExpired()) {
                throw new IllegalArgumentException("Discount expired");
            }
            discountRepository.delete(discount);  // 할인 쿠폰 사용 완료
        }

        // 3. 결제 정보 초기화 및 PENDING 상태로 설정
        Payment payment = new Payment(orderId, userId, paymentAmount, "PENDING");
        paymentRepository.save(payment);

        // 외부 결제 시스템 호출을 위한 독립 트랜잭션
        try {
            completeExternalPayment(payment);
        } catch (Exception e) {
            // 외부 결제 실패 시 전체 롤백
            throw new RuntimeException("Payment failed with external provider", e);
        }
    }

    /**
     * 외부 결제 시스템 호출을 별도의 트랜잭션으로 처리하여,
     * 외부 결제 실패 시 전체 트랜잭션을 롤백합니다.
     */
    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void completeExternalPayment(Payment payment) throws Exception {
        // 외부 결제 시스템 호출
        boolean paymentSuccess = externalPaymentGateway.process(payment.getAmount());

        if (!paymentSuccess) {
            throw new Exception("External payment failed");
        }

        // 4. 결제 상태를 COMPLETED로 업데이트
        payment.setStatus("COMPLETED");
        paymentRepository.save(payment);

        // 5. 트랜잭션 로그 생성
        TransactionLog log = new TransactionLog(payment.getPaymentId(), "PAYMENT", payment.getAmount(), "SUCCESS");
        transactionLogRepository.save(log);
    }
}

```
<br/><br/>

#### 트랜잭션 처리 단계
1. **사용자 포인트 차감**
    - 포인트 잔액을 차감하고, 사용자 정보를 업데이트합니다.
    - 이 작업이 실패하면 결제 전체가 롤백됩니다.
2. **할인 쿠폰 적용 및 차감**
    - 할인 쿠폰의 만료 여부를 확인하고, 만료되지 않았으면 쿠폰을 삭제하여 차감 처리합니다.
    - 이 작업이 실패해도 전체 트랜잭션이 롤백됩니다.
3. **결제 정보 초기화 및 상태 설정**
    - `Payment` 엔터티를 생성하고 상태를 "PENDING"으로 설정하여 결제를 초기화합니다.
    - 결제 상태가 아직 확정되지 않았으므로 외부 결제가 성공해야만 이후 상태를 "COMPLETED"로 업데이트합니다.
4. **외부 결제 시스템 호출** (`@Transactional(propagation = Propagation.REQUIRES_NEW)`)
    - 외부 결제 시스템을 호출하여 실제 결제를 수행합니다.
    - 이 단계는 `Propagation.REQUIRES_NEW`로 설정하여 별도의 트랜잭션으로 처리됩니다.
    - 외부 결제가 실패하면 예외가 발생하여 상위 트랜잭션이 롤백됩니다.
    - 외부 결제가 성공하면 결제 상태를 "COMPLETED"로 업데이트하고 트랜잭션 로그를 생성합니다.
5. **트랜잭션 로그 생성**
    - 결제가 성공적으로 완료되면 트랜잭션 로그에 기록하여 결제 내역을 추적합니다.

#### 중요 부분 요약 설명
- **독립 트랜잭션**: 외부 결제 시스템 호출과 같은 외부 작업은 실패 확률이 높으므로 `Propagation.REQUIRES_NEW` 트랜잭션으로 관리합니다. 이를 통해 외부 결제 실패 시 전체 롤백이 쉽게 관리됩니다.
- **전체 트랜잭션 롤백**: 외부 결제에서 실패하면 `processPayment`의 트랜잭션도 모두 롤백됩니다. 이는 결제 과정에서 발생할 수 있는 불일치를 방지하는 역할을 합니다.
- **예외 처리**: 결제 실패나 외부 호출 실패 시 적절한 예외를 발생시켜 사용자에게 알리고, 필요한 경우 오류를 기록할 수 있습니다.
<br/><br/><br/><br/>

# Outro
이번 글에서는 Spring Boot에서 트랜잭션을 관리하는 방법을 실무에서 사용할 법한 결제 프로세스를 예시로 하여 자세히 알아보았습니다. 

트랜잭션의 기본 개념부터 @Transactional 애너테이션의 다양한 속성 설정, 그리고 실무에서 자주 사용되는 트랜잭션 관리 예시까지 다룰려고 노력했는데 도움이 되었으면 좋겠습니다 ^^

트랜잭션이 데이터 일관성을 유지하는데 얼마나 중요한 역할을 하는지 인지하셨으면 좋겠구요.

Spring Boot를 통해 안정적이고 견고한 시스템을 구축하고자 하는 분들에게 도움이 되었기를 바랍니다. 

긴 글 읽어주셔서 감사합니다! 궁금한 점이나 더 알고 싶은 내용이 있다면 언제든지 댓글로 남겨주세요.
<br/><br/><br/><br/>
