---
layout: post
title: "Spring Framework의 탄생 배경과 프레임워크 vs 라이브러리"
categories: [computer-science]
tags: [Spring, Framework, DI, IoC, Library]
---

이 글에서는 Spring Framework가 어떤 배경에서 탄생했는지, 
그리고 Spring을 이해하는 데 중요한 개념인 
프레임워크와 라이브러리의 차이를 제어 흐름의 관점에서 살펴본다.
이를 통해 단순히 기능을 사용하는 수준을 넘어, 
Spring이 해결하고자 했던 문제와 그 설계 철학을 함께 이해해보고자 한다.

---

## 1. Spring Framework 탄생 배경

`Spring Framework`는 엔터프라이즈 Java 
개발 환경의 복잡성과 비효율을 해결하기 위해 등장했다.
Spring 이전의 Java 서버 개발은 주로 
EJB(Enterprise JavaBeans)를 중심으로 이루어졌는데, 
이 방식은 강력했지만 개발자에게 많은 부담을 안겨주었다.

대표적 문제는 비즈니스 로직이 특정 기술과 
컨테이너에 강하게 결합되어 있다는 점이었다. 
객체는 단순한 로직을 담고 있음에도 불구하고 EJB 인터페이스를 구현해야 했고, 
컨테이너 없이는 실행과 테스트가 거의 불가능했다. 
이로 인해 코드의 재사용성과 테스트 가능성은 크게 떨어졌고,
작은 변경에도 시스템 전반에 영향을 미치는 구조가 되었다.

Spring은 이러한 문제를 해결하기 위해 
**POJO(Plain Old Java Object)** 기반 개발을 핵심 철학으로 삼았다.
객체는 순수한 Java 객체로 유지하고, 
객체의 생성과 의존 관계 관리는 프레임워크가 담당하도록 설계한 것이다.
이를 통해 비즈니스 로직과 인프라 로직을 분리하고,
개발자가 핵심 도메인 문제에 집중할 수 있는 환경을 제공했다.

이 과정에서 등장한 개념이 `IoC(Inversion of Control, 제어의 역전)`,
`DI(Dependency Injection, 의존성 주입)` 이다. 
Spring은 객체 생성과 연결의 제어권을 
개발자가 아닌 프레임워크가 가지도록 함으로써, 
유연하고 테스트 가능한 애플리케이션 구조를 가능하게 했다.

---

## 2. Framework vs Library

`Framework`와 `Library`는 모두 개발을 돕는 도구이지만,
두 개념의 본질적인 차이는 제어 흐름을 누가 가지고 있는가에 있다.

`Library`는 개발자가 필요할 때 직접 호출해서 사용하는 도구이다.
전체 프로그램의 흐름은 개발자 코드가 주도하며, 
`Library`는 그 흐름 속에서 기능을 제공하는 역할을 한다. 
예를 들어 Java의 Collections나 StringUtils 
같은 라이브러리는 개발자가 원하는 시점에 호출해서 사용한다.
즉, **개발자가 주체이고 `Library`는 보조 수단**이다.

반면 `Framework`는 제어 흐름의 주체가 개발자가 아니다.
`Framework`가 애플리케이션의 전체 흐름을 관리하며, 
개발자는 정해진 규칙에 따라 필요한 부분만 구현한다. 
`Spring Framework`에서는 객체의 생성, 의존성 주입,
생명주기 관리까지 모두 `Framework`가 담당한다. 
개발자는 단지 “이런 역할의 객체가 필요하다”는 것을 선언할 뿐이다.

이러한 구조를 **제어의 역전(IoC)** 이라고 부른다. 
기존에는 개발자가 객체를 생성하고 조립했다면, 
`Framework`가 객체를 생성하고 필요할 때 개발자 코드를 호출한다. 
Spring은 이러한 구조를 통해 결합도를 낮추고, 
변경과 확장에 강한 애플리케이션 설계를 가능하게 했다.

---

다음의 예시를 보자.

```java

public class SortExample {
    public static void main(String[] args) {
        List<Integer> numbers = new ArrayList<>(List.of(3, 1, 2));

        // 개발자가 직접 라이브러리를 호출
        Collections.sort(numbers);

        System.out.println(numbers);
    }
}

```

이 코드에서 봐야 할 점은 개발자가 직접 제어를 한다.
Collection.sort()는 개발자가 원할 때 호출하며,
실행 권한은 전적으로 개발자에게 있다.
이때 `Library`는 기능 제공 도구일 뿐,
애플리케이션의 전체 흐름에 관여하지 않는다.

즉, **개발자가 호출하는 코드**의 성격을 가진다

---

다음의 예시도 보자.

```java

@Service
public class UserService {
    public void register() {
        System.out.println("유저 등록 로직 실행");
    }
}

@Component
public class UserController {

    private final UserService userService;

    public UserController(UserService userService) {
        this.userService = userService;
    }
}

```

위 코드에서 개발자가 하는 일은 거의 없다.

- UserService 객체를 직접 생성하지 않는다.
- new UserService()를 호출하지 않는다.
- UserService의 생명주기도 신경 쓰지 않는다.

객체의 생성, 의존성 주입, 생명주기 관리는 모두 Spring 컨테이너가 담당한다.
개발자는 단지 “이 객체는 서비스 역할이다”, “이 객체가 필요하다”라고 선언만 한다.

즉, Spring에서는
“`Framework`가 코드를 호출하고, 개발자는 그 안에서 로직을 작성한다.”
이 구조가 바로 **제어의 역전(IoC)** 이다.

---

### 핵심 차이 요약

- 개발자가 흐름을 제어하고, 필요할 때 `Library`를 호출
- `Framework`가 흐름을 제어하고, 필요할 때 개발자 코드를 호출

`Spring Framework`는 이러한 구조를 통해 객체 결합도를 낮추고,
테스트 가능하며 확장에 강한 애플리케이션을 만들 수 있도록 도와준다.
