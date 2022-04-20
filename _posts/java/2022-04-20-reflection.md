---
title:  "리플렉션" 
excerpt: "리플렉션에 대해 간단하게 알아보자."

categories:
  - Java
tags:
  - [reflection]

toc: true
toc_sticky: true

date: 2022-04-20
last_modified_at: 2022-04-20
---

인프런에 있는 김영한 님의 **스프링 핵심 원리 - 고급편** 강의를 듣고 정리한 내용 입니다. 😀    
[🌜 [스프링 핵심원리 - 고급편]강의 들으러 가기!](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-%ED%95%B5%EC%8B%AC-%EC%9B%90%EB%A6%AC-%EA%B3%A0%EA%B8%89%ED%8E%B8/dashboard){:target="_blank"}
{: .notice--warning}

자바의 리플렉션 기술에 대해 간단하게 알아보자.  
리플렉션 기술을 사용하면 클래스나 메서드의 메타정보를 동적으로 획득하고, 코드도 동적으로 호출할 수 있다.  
여기서는 JDK 동적 프록시를 이해하기 위한 최소한의 리플랙션 기술을 알아보자.

<br>

## 🔔 리플렉션이 필요한 상황
**ReflectionTest**
```java
@Slf4j
public class ReflectionTest {
    @Test
    void reflection0() {
        Hello target = new Hello();
        
        //공통 로직1 시작
        log.info("start");
        String result1 = target.callA(); //호출하는 메서드가 다름
        log.info("result={}", result1); 
        //공통 로직1 종료

        //공통 로직2 시작
        log.info("start");
        String result2 = target.callB(); //호출하는 메서드가 다름
        log.info("result={}", result2);
        //공통 로직2 종료
    }
      
    @Slf4j
    static class Hello {
        public String callA() {
            log.info("callA");
            return "A";
        }
          
        public String callB() {
            log.info("callB");
            return "B";
        }
    }
}
```
* 공통 로직1과 공통 로직2는 호출하는 메서드만 다르고 전체 코드 흐름이 완전히 같다. 
    * 먼저 start 로그를 출력한다.
    * 어떤 메서드를 호출한다.
    * 메서드의 호출 결과를 로그로 출력한다.
* 여기서 공통 로직1과 공통 로직 2를 하나의 메서드로 뽑아서 합칠 수 있을까?

* 쉬워 보이지만 메서드로 뽑아서 공통화하는 것이 생각보다 어렵다. 왜냐하면 중간에 호출하는 메서드가 다르기 때문이다.  
* 호출하는 메서드인 `target.callA()` , `target.callB()` 이 부분만 동적으로 처리할 수 있다면 문제를 해결할 수 있을 듯 하다.

```java
log.info("start");
String result = xxx(); //호출 대상이 다름, 동적 처리 필요
log.info("result={}", result);
```

이럴 때 사용하는 기술이 바로 리플렉션이다. 리플렉션은 클래스나 메서드의 메타정보를 사용해서 동적으로 호출하는 메서드를 변경할 수 있다. 바로 리플렉션 사용해보자.

> **참고**: 람다를 사용해서 공통화 하는 것도 가능하다. 여기서는 람다를 사용하기 어려운 상황이라 가정하자. 그리고 리플렉션 학습이 목적이니 리플렉션에 집중하자.

<br>

### 🔔 리플렉션 적용 - 1
**ReflectionTest - reflection1 추가**
```java
@Test
void reflection1() throws Exception {
    //클래스 정보
    Class classHello = Class.forName("hello.proxy.jdkdynamic.ReflectionTest$Hello");

    Hello target = new Hello();
    //callA 메서드 정보
    Method methodCallA = classHello.getMethod("callA");
    Object result1 = methodCallA.invoke(target);
    log.info("result1={}", result1);

    //callB 메서드 정보
    Method methodCallB = classHello.getMethod("callB");
    Object result2 = methodCallB.invoke(target);
    log.info("result2={}", result2);
}
```
* `Class.forName("hello.proxy.jdkdynamic.ReflectionTest$Hello")` : 클래스 메타정보를 획득한다. 참고로 내부 클래스는 구분을 위해 `$` 를 사용한다.
* `classHello.getMethod("call")` : 해당 클래스의 `call` 메서드 메타정보를 획득한다.
* `methodCallA.invoke(target)` : 획득한 메서드 메타정보로 실제 인스턴스의 메서드를 호출한다. 여기서 `methodCallA` 는 `Hello` 클래스의 `callA()` 이라는 메서드 메타정보이다.  
 `methodCallA.invoke(인스턴스)` 를 호출하면서 인스턴스를 넘겨주면 해당 인스턴스의 `callA()` 메서드를 찾아서 실행한다. 여기서는 `target` 의 `callA()` 메서드를 호출한다.


그런데 `target.callA()` 나 `target.callB()` 메서드를 직접 호출하면 되지 이렇게 메서드 정보를 획득해서 메서드를 호출하면 어떤 효과가 있을까? 여기서 중요한 핵심은 클래스나 메서드 정보를 동적으로 변경할 수 있다는 점이다.

기존의 `callA()` , `callB()` 메서드를 직접 호출하는 부분이 `Method` 로 대체되었다. 덕분에 이제 공통 로직을 만들 수 있게 되었다.

<br>

### 🔔 리플렉션 적용 - 2

**ReflectionTest - reflection2 추가**
```java
@Test
void reflection2() throws Exception {
    Class classHello = Class.forName("hello.proxy.jdkdynamic.ReflectionTest$Hello");
    Hello target = new Hello();

    Method methodCallA = classHello.getMethod("callA");
    dynamicCall(methodCallA, target);
    
    Method methodCallB = classHello.getMethod("callB");
    dynamicCall(methodCallB, target);
  }
  
private void dynamicCall(Method method, Object target) throws Exception {
    log.info("start");
    Object result = method.invoke(target);
    log.info("result={}", result);
}
```

* `dynamicCall(Method method, Object target)`
    * 공통 로직1, 공통 로직2를 한번에 처리할 수 있는 통합된 공통 처리 로직이다.
    * `Method method` : 첫 번째 파라미터는 호출할 메서드 정보가 넘어온다. 이것이 핵심이다. 기존에는 메서드 이름을 직접 호출했지만, 이제는 `Method` 라는 메타정보를 통해서 호출할 메서드 정보가 동적으로 제공된다.
    * `Object target` : 실제 실행할 인스턴스 정보가 넘어온다. 타입이 `Object` 라는 것은 어떠한 인스턴스도 받을 수 있다는 뜻이다. 물론 `method.invoke(target)` 를 사용할 때 호출할 클래스와 메서드 정보가 서로 다르면 예외가 발생한다.


## 🔔 정리
정적인 `target.callA()` , `target.callB()` 코드를 리플렉션을 사용해서 `Method` 라는 메타정보로 추상화했다. 덕분에 공통 로직을 만들 수 있게 되었다.

## 🔔 주의  
리플렉션을 사용하면 클래스와 메서드의 메타정보를 사용해서 애플리케이션을 동적으로 유연하게 만들 수 있다. 하지만 리플렉션 기술은 런타임에 동작하기 때문에, 컴파일 시점에 오류를 잡을 수 없다.  
예를 들어서 지금까지 살펴본 코드에서 `getMethod("callA")` 안에 들어가는 문자를 실수로 `getMethod("callZ")` 로 작성해도 컴파일 오류가 발생하지 않는다. 그러나 해당 코드를 직접 실행하는 시점에 발생하는 오류인 런타임 오류가 발생한다.  
가장 좋은 오류는 개발자가 즉시 확인할 수 있는 컴파일 오류이고, 가장 무서운 오류는 사용자가 직접 실행할 때 발생하는 런타임 오류다.

따라서 리플렉션은 일반적으로 사용하면 안된다. 지금까지 프로그래밍 언어가 발달하면서 타입 정보를 기반으로 컴파일 시점에 오류를 잡아준 덕분에 개발자가 편하게 살았는데, 리플렉션은 그것에 역행하는 방식이다.

리플렉션은 프레임워크 개발이나 또는 매우 일반적인 공통 처리가 필요할 때 부분적으로 주의해서 사용해야 한다.

<br>

[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}
<br>
