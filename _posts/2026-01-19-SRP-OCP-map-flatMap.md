---
layout: post
title: "단일 책임 원칙(SRP) vs 개방-폐쇄 원칙(OCP), map vs flatMap 정리"
---

SRP와 OCP를 공부하면서  
원칙 자체보다 “코드에 어떻게 녹여낼 것인가”가 더 어렵게 느껴졌다.

컬렉션을 다루는 과정에서 자주 사용하는 `map`, `flatMap` 또한  
단순한 변환을 넘어 책임 분리와 확장성 관점에서 다시 볼 필요가 있다고 느꼈다.

이 글은 그런 고민을 정리하면서 
**객체지향 원칙과 컬렉션 연산 그리고 활용법에 대한** 기록이다.

---

## 1. 단일 책임 원칙(SRP) vs 개방-폐쇄 원칙(OCP)

### 1-1) 단일 책임 원칙(Single Responsibility Principle)

`SRP`는 하나의 클랙스는 **하나의 책임만 가져야 한다**는 객체지향 원칙이다.

여기서 말하는 책임이란 단순히 메서드 개수나 코드 양이 아니라,  
**변경의 이유**를 기준으로 생각하는 것이 중요하다.

즉, 어떤 클래스를 수정해야 하는 이유가 여러 가지라면  
그 클래스는 이미 여러 책임을 가지고 있다고 볼 수 있다.

SRP를 지키면 코드의 역할이 명확해지고,  
변경이 발생했을 때 영향을 받는 범위를 최소화할 수 있다.

---

다음의 예시를 보자.

아래는 단일 책임 원칙을 적용한 코드로
저장, 이메일 전송, 로그 기록을 별도의 클래스로 분리했다.

```java

class UserRepository {
    public void save(String email, String password) {
        System.out.println("Saving user to DB: " + email);
    }
}

class EmailSender {
    public void sendWelcomeEmail(String email) {
        System.out.println("Sending welcome email to: " + email);
    }
}

class AuditLogger {
    public void log(String message) {
        System.out.println("[LOG] " + message);
    }
}

```

각각 분리된 클래스는 변경 이유가 1개이고,
UserService는 유저 등록이라는 하나의 책임만을 가진다.

```java

public class UserService {
    private final UserRepository userRepository;
    private final EmailSender emailSender;
    private final AuditLogger auditLogger;

    public UserService(UserRepository userRepository, EmailSender emailSender, AuditLogger auditLogger) {
        this.userRepository = userRepository;
        this.emailSender = emailSender;
        this.auditLogger = auditLogger;
    }

    public void register(String email, String password) {
        userRepository.save(email, password);
        emailSender.sendWelcomeEmail(email);
        auditLogger.log("User registered: " + email);
    }
}

```
위 코드의 예시로 
각 클래스는 명확히 하나의 책임만을 가진다.

- 저장 방식이 바뀌면 `UserRepository`만 수정
- 이메일 정책이 바뀌면 `EmailSender`만 수정
- 로깅 정책이 바뀌면 `AuditLogger`만 수정

즉,  
변경의 영향 범위가 작아지고  
코드를 이해하고 테스트하기도 훨씬 쉬워진다.

이것이 `SRP`가 가져다주는 가장 큰 장점이다.

---

### 1-2) 개방-폐쇄 원칙(Open-Closed Principle)

`OCP`는 소프트웨어 요소가 
**확장에는 열러 있고, 변경에는 닫혀 있어야 한다**는 원칙이다.

여기서 말하는 “변경”이란  
단순한 기능 추가가 아니라,  
**이미 검증된 기존 코드를 직접 수정하는 것**을 의미한다.

`OCP`의 핵심은  
새로운 요구사항이 생겼을 때 기존 로직을 고치기보다  
**새로운 코드 추가만으로 기능을 확장할 수 있도록 설계하는 것**이다.

---

다음의 예시를 보자.

아래는 개방-폐쇄 원칙을 적용하기 위해
결제 방식에 따라 달라지는 로직을
인터페이스로 추상화한 코드이다.

```java

public interface PaymentStrategy {
    void pay(int amount);
}

```

이제 각 결제 방식은 `PaymentStrategy`
인터페이스를 구현하는 구현체 클래스로 분리한다.

```java

class CardPayment implements PaymentStrategy {
    @Override
    public void pay(int amount) {
        System.out.println("Pay by card: " + amount);
    }
}

class KakaoPayment implements PaymentStrategy {
    @Override
    public void pay(int amount) {
        System.out.println("Pay by kakao: " + amount);
    }
}

class BankTransferPayment implements PaymentStrategy {
    @Override
    public void pay(int amount) {
        System.out.println("Pay by bank transfer: " + amount);
    }
}

```

이제 서비스는 결제 방식이 아닌
**결제를 수행할 수 있는 전략(PaymentStrategy)에 의존**한다.

```java

import java.util.Map;

public class PaymentService {

    private final Map<String, PaymentStrategy> strategies;

    public PaymentService(Map<String, PaymentStrategy> strategies) {
        this.strategies = strategies;
    }

    public void pay(String method, int amount) {
        PaymentStrategy strategy = strategies.get(method);
        if (strategy == null) {
            throw new IllegalArgumentException("Unknown payment method");
        }
        strategy.pay(amount);
    }
}

```

이 구조의 장점은 명확하다.

새로운 결제 방식이 추가되더라도  
`PaymentService`는 수정하지 않고
새로운 `PaymentStrategy` 구현 클래스만 추가하면 된다

즉, **기존 코드는 닫혀 있고, 기능 확장은 열려 있는 상태**가 된다.

이것이 OCP가 지향하는 설계 방식이다.

---

#### 핵심 요약

- **SRP(단일 책임 원칙)**는  
  하나의 클래스가 **하나의 변경 이유만** 갖도록 책임을 분리하는 원칙이다.  
  이를 통해 변경의 영향을 최소화하고, 코드의 이해와 테스트를 쉽게 만든다.

- **OCP(개방-폐쇄 원칙)**는  
  기능 확장은 가능하되,  
  **기존 코드는 수정하지 않도록** 설계하는 원칙이다.  
  새로운 요구사항을 “코드 수정”이 아니라  
  “새로운 코드 추가”로 해결하는 것을 목표로 한다.

- SRP는 **책임의 분리**에 초점을 두고,  
  OCP는 **확장 방식의 설계**에 초점을 둔다.

- 두 원칙을 함께 고려하면  
  변경에 강하고, 확장에 유연한 객체지향 구조를 만들 수 있다.

---

## 2. Stream API에서의 map vs flatMap

### 2-1) map

`map`은 스트림의 각 요소를  
**다른 형태의 요소로 변환하는 연산**이다.

중요한 점은  
`map`은 항상 **하나의 입력에 대해 하나의 결과를 만든다**는 것이다.

즉, `map`은  
1 : 1 변환을 수행하는 연산이라고 볼 수 있다.

---

다음의 예시를 보자.

```java

import java.util.List;

public class MapExample {
    public static void main(String[] args) {
        List<String> words = List.of("java", "stream", "api");

        List<Integer> lengths = words.stream()
                .map(String::length)
                .toList();

        System.out.println(lengths); // [4, 6, 3]
    }
}

```

위 코드에서
각 문자열은 하나의 정수 값(문자열 길이)로 반환된다.

입력과 출력의 개수는 동일하며,
중간 구조가 바뀌지 않는다.

---

문제는  
변환 결과가 또 다른 컬렉션(또는 스트림)일 때 발생한다.

이 경우 `map`을 사용하면  
스트림 안에 또 다른 스트림이 들어가는  
**중첩 구조**가 만들어진다.

```java

import java.util.Arrays;
import java.util.List;

public class MapLimitExample {
    public static void main(String[] args) {
        List<String> sentences = List.of(
                "hello world",
                "java stream"
        );

        List<String[]> result = sentences.stream()
                .map(s -> s.split(" "))
                .toList();

        System.out.println(result.size()); // 2
    }
}

```

이 코드의 결과는  
단어들의 리스트가 아니라  
**문장별로 분리된 배열의 리스트(List<String[]>)**이다.

즉,  
스트림의 구조가 한 단계 더 깊어졌고,  
이 상태로는 바로 후속 처리를 하기 어렵다.

---

### 2-2) flatMap

`flatMap`은  
각 요소를 스트림으로 변환한 뒤,  
그 결과를 **하나의 스트림으로 평탄화(flatten)** 하는 연산이다.

즉, `flatMap`은  
- 변환(map)
- 중첩 제거(flatten)
를 동시에 수행한다.

결과적으로  
중첩된 구조를 한 단계 낮춘 스트림을 만들어준다.

---

다음의 예시를 보자.

위 예시에서 `flatMap`을 활용해
중첩을 제거한 코드이다.

```java

import java.util.Arrays;
import java.util.List;

public class FlatMapExample {
    public static void main(String[] args) {
        List<String> sentences = List.of(
                "hello world",
                "java stream"
        );

        List<String> words = sentences.stream()
                .flatMap(s -> Arrays.stream(s.split(" ")))
                .toList();

        System.out.println(words); // [hello, world, java, stream]
    }
}

```

각 문장은  
단어 배열 → 단어 스트림으로 변환되고,  
`flatMap`에 의해 하나의 스트림으로 합쳐진다.

이제 결과는  
**모든 단어가 한 줄로 나열된 스트림**이 된다.

---

다음 예시도 보자.

```java

import java.util.List;

public class FlattenListExample {
    public static void main(String[] args) {
        List<List<String>> tags = List.of(
                List.of("java", "oop"),
                List.of("stream", "lambda")
        );

        List<String> flat = tags.stream()
                .flatMap(List::stream)
                .toList();

        System.out.println(flat);
    }
}

```

이는 중첩 컬렉션 평탄화로

`List<List<T>>` 형태의 데이터를  
`List<T>`로 변환해야 할 때  
`flatMap`은 거의 필수적으로 사용된다.

---

#### 핵심 요약
- `map`은 각 요소를 다른 형태로 변환할 때 사용한다.
- `flatMap`은 변환 결과가 컬렉션(또는 스트림)일 때,
  중첩 구조를 제거하기 위해 사용한다.
- 결과에 중첩이 남는다면 `flatMap`을 고려해볼 수 있다.

---
