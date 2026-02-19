---
title: "Spring AOP와 @Controller와 @RestController의 차이"
categories: [Backend, Spring]
tags: [Spring, AOP, MVC, Controller, RestController, Web]
---

Spring 기반 애플리케이션을 이해하려면 두 가지 핵심 개념을 반드시 짚고 넘어가야 한다.
하나는 `AOP(Aspect Oriented Programming)` 를 통한 공통 관심사 처리이고,
다른 하나는 `Spring MVC`의 요청 처리 흐름이다.

`AOP`는 비즈니스 로직과 분리되어야 할 공통 기능을 어떻게 관리하는지 설명해주고,
`Spring MVC`는 클라이언트의 요청이 어떤 흐름을 거쳐 처리되는지를 보여준다.
이 글에서는 `AOP`의 필요성과 실제 활용 사례, 그리고 `Spring MVC`의 요청 처리 과정을 
`@Controller`와 `@RestController`의 차이를 중심으로 정리한다.

---

## 1. Spring에서 AOP가 필요한 이유와 실제 활용 사례

### 1-1) AOP란 무엇인가?

`AOP(Aspect Oriented Programming)`는 
**공통 관심사를 핵심 비즈니스 로직과 분리**하기 위한 프로그래밍 방식이다.

애플리케이션에는 다음과 같은 기능이 반복적으로 등장하게 된다.
- 로깅 (Logging)
- 트랜젝션 (Transaction)
- 보안 처리 (Security)
- 성능 측정 (Performance Monitoring)
- 예외 처리 (Exeption Handing)

이러한 기능은 비즈니스 로직과 직접적인 관련은 없지만, 여러 계층에 걸쳐 공통적으로 필요하다.
이를 코드 내부에 직접 작성하면 핵심 로직이 가려지고, 중복이 발생하며, 유지보수가 어려워진다.

---
### 1-2) AOP가 필요한 이유

Spring에서 `AOP`가 필요한 이유는 **관심사의 분리(Separation of Concerns)**를 실현하기 위해서이다.

예를 들어 모든 서비스 메서드의 실행 시간을 측정한다고 가정해보자.

```java

public void createUser() {
    long start = System.currentTimeMillis();
    try {
      // 비즈니스 로직
    } finally {
        long end = System.currentTimeMillis();
        System.out.println("실행 시간: " + (end - start));
    }
}

```
이 코드가 여러 클래스에 반복된다면:

- 코드 중복 발생
- 핵심 로직 가독성 저하
- 변경 시 모든 클래스 수정 필요

`AOP`를 사용하면 이러한 공통 로직을 별도의 `@Aspect`로 분리할 수 있다.

다음의 실제 활용 사례들을 보자.

---
#### 로깅 처리

```java

@Aspect
@Component
public class LoggingAspect {

    @Around("execution(* com.example.service..*(..))")
    public Object logExecutionTime(ProceedingJoinPoint joinPoint) throws Throwable {
        long start = System.currentTimeMillis();
        Object result = joinPoint.proceed();
        long end = System.currentTimeMillis();

        System.out.println(joinPoint.getSignature() +
                " 실행 시간: " + (end - start));
        return result;
    }
}

```
비즈니스 로직을 수정하지 않아도 실행 시간 측정을 가능하게 해준다.

---
#### 트랜젝션 관리

```java

@Transactional
public void createOrder() {
    // 주문 생성 로직
}

```
`@Transactional`은 내부적으로 `AOP` 기반으로 동작한다.

정리해보자면, AOP의 장점은 아래와 같다.

- 코드 중복 제거
- 핵심 로직 가독성 향상
- 유지보수성 향상
- OCP(개방-폐쇄 원칙) 실현

---

## 2. Spring MVC 요청 처리 흐름과 @Controller / @RestController 차이

### 2-1) Spring MVC의 요청 처리 흐름

Spring MVC에서 클라이언트의 요청은 다음과 같은 흐름을 거친다.

1. 클라이언트 → HTTP 요청 전송
2. DispatcherServlet이 요청을 수신
3. HandlerMapping이 적절한 컨트롤러 탐색
4. Controller 실행
5. ViewResolver 또는 HttpMessageConverter를 통해 응답 생성
6. 클라이언트에게 응답 반환

`DispatcherServlet`은 Spring MVC의 **프론트 컨트롤러(Front Controller) 역할**을 수행한다.

---
### 2-2) @Controller

`@Controller`는 전통적인 MVC 패턴에서 사용된다.

```java

@Controller
public class UserController {

    @GetMapping("/users")
    public String getUsers(Model model) {
        model.addAttribute("name", "Seongjun");
        return "user";
    }
}

```
특징은 다음과 같다.

- 반환 값은 View 이름
- ViewResolver가 실제 템플릿 파일로 변환
- 주로 Thymeleaf, JSP 등과 함께 사용
- 서버에서 HTML을 생성하여 반환

---
### 2-3) @RestController

`@RestController`는 `@Controller` + `@ResponseBody`가 결합된 형태이다.

```java

@RestController
public class UserController {

    @GetMapping("/users")
    public User getUser() {
        return new User("Seongjun");
    }
}

```
특징은 다음과 같다.

- 반환 객체를 JSON 형태로 자동 변환
- HttpMessageConverter가 객체를 직렬화
- 주로 REST API 개발에 사용
- 클라이언트(React, Vue 등)와 분리된 구조

---
#### @Controller vs @RestController 비교

| 구분 | @Controller | @RestController |
|:--|:--|:--|
| 반환 값 | View 이름 | 객체(JSON) |
| 응답 방식 | ViewResolver | HttpMessageConverter |
| 사용 목적 | 서버 사이드 렌더링 | REST API |
| HTML 생성 주체 | 서버 | 클라이언트 |

---
### 마무리

`Spring AOP`는 공통 관심사를 분리하여 코드의 중복을 제거하고 유지보수성을 높이는 역할을 한다.
`Spring MVC`는 `DispatcherServlet`을 중심으로 요청을 처리하며,
`@Controller`와 `@RestController`는 응답 생성 방식에서 차이를 보인다.

이 두 개념을 함께 이해하면,
Spring 애플리케이션이 요청을 어떻게 처리하고, 
부가 기능을 어떻게 관리하는지 전체적인 구조를 파악할 수 있다.