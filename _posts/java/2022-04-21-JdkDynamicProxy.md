---
title:  "JDK 동적 프록시" 
excerpt: "JDK 동적 프록시에 대해 알아보자."

categories:
  - Java
tags:
  - [JDK 동적 프록시]

toc: true
toc_sticky: true

date: 2022-04-21
last_modified_at: 2022-04-21
---

인프런에 있는 김영한 님의 **스프링 핵심 원리 - 고급편** 강의를 듣고 정리한 내용 입니다. 😀    
[🌜 [스프링 핵심원리 - 고급편]강의 들으러 가기!](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-%ED%95%B5%EC%8B%AC-%EC%9B%90%EB%A6%AC-%EA%B3%A0%EA%B8%89%ED%8E%B8/dashboard){:target="_blank"}
{: .notice--warning}

## 🔔 JDK 동적 프록시 - 소개
[지난번에 적용한 프록시](https://sapzilking.github.io/design%20patterns/interfaceBaseClassBaseProxy/) 에서는 프록시를 적용하기 위해 적용 대상의 숫자 만큼 많은 프록시 클래스를 만들었다. 적용 대상이 100 개면 프록시 클래스도 100개 만들었다. 그런데 앞서 살펴본 것과 같이 프록시 클래스의 기본 코드와 흐름은 거의 같고, 프록시를 어떤 대상에 적용하는가 정도만 차이가 있었다. 쉽게 이야기해서 프록시의 로직은 같은데, 적용 대상만 차이가 있는 것이다.

이 문제를 해결하는 것이 바로 동적 프록시 기술이다.  
동적 프록시 기술을 사용하면 개발자가 직접 프록시 클래스를 만들지 않아도 된다. 이름 그대로 프록시 객체를 동적으로 런타임에 개발자 대신 만들어준다. 그리고 동적 프록시에 원하는 실행 로직을 지정할 수 있다.

사실 동적 프록시는 말로는 이해하기 쉽지 않다. 바로 예제 코드를 보자.

> 주의
JDK 동적 프록시는 인터페이스를 기반으로 프록시를 동적으로 만들어준다. 따라서 인터페이스가 필수이다.

먼저 자바 언어가 기본으로 제공하는 JDK 동적 프록시를 알아보자.

## 🔔 기본 예제 코드
JDK 동적 프록시를 이해하기 위해 아주 단순한 예제 코드를 만들어보자.  
간단히 `A` , `B` 클래스를 만드는데, JDK 동적 프록시는 인터페이스가 필수이다. 따라서 인터페이스와 구현체로 구분했다.

**AInterface**
```java
public interface AInterface {
    String call();
}
```
**AImpl**
```java
@Slf4j
public class AImpl implements AInterface {
    @Override
    public String call() {
        log.info("A 호출");
        return "a";
    }
}
```

**BInterface**
```java
public interface BInterface {
    String call();
}
```

**BImpl**
```java
@Slf4j
public class BImpl implements BInterface {
    @Override
    public String call() {
        log.info("B 호출");
        return "b";
    }
}
```

## 🔔 JDK 동적 프록시 - 예제 코드
JDK 동적 프록시에 적용할 로직은 `InvocationHandler` 인터페이스를 구현해서 작성하면 된다.

**JDK 동적 프록시가 제공하는 InvocationHandler**
```java
public interface InvocationHandler {
    public Object invoke(Object proxy, Method method, Object[] args)
        throws Throwable;
}
```
**제공되는 파라미터는 다음과 같다.**
* `Object proxy` : 프록시 자신
* `Method method` : 호출한 메서드
* `Object[] args` : 메서드를 호출할 때 전달한 인수

이제 구현 코드를 보자.

**TimeInvocationHandler**

```java
@Slf4j
public class TimeInvocationHandler implements InvocationHandler {
    
    private final Object target;
      
    public TimeInvocationHandler(Object target) {
        this.target = target;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        log.info("TimeProxy 실행");
        long startTime = System.currentTimeMillis();
        
        Object result = method.invoke(target, args);

        long endTime = System.currentTimeMillis();
        long resultTime = endTime - startTime; log.info("TimeProxy 종료 resultTime={}", resultTime);
        return result;
    }
}
```
* `TimeInvocationHandler` 은 `InvocationHandler` 인터페이스를 구현한다. 이렇게해서 JDK 동적 프록시에 적용할 공통 로직을 개발할 수 있다.
* `Object target` : 동적 프록시가 호출할 대상
* `method.invoke(target, args)` : 리플렉션을 사용해서 `target` 인스턴스의 메서드를 실행한다. `args` 는 메서드 호출시 넘겨줄 인수이다.

이제 테스트 코드로 JDK 동적 프록시를 사용해보자.

**JdkDynamicProxyTest**
```java
@Slf4j
public class JdkDynamicProxyTest {
    @Test
    void dynamicA() {
        AInterface target = new AImpl();
        TimeInvocationHandler handler = new TimeInvocationHandler(target);
        AInterface proxy = (AInterface) Proxy.newProxyInstance(AInterface.class.getClassLoader(), new Class[]{AInterface.class}, handler);
        proxy.call();
        log.info("targetClass={}", target.getClass());
        log.info("proxyClass={}", proxy.getClass());
    }
      
    @Test
    void dynamicB() {
        BInterface target = new BImpl();
        TimeInvocationHandler handler = new TimeInvocationHandler(target);
        BInterface proxy = (BInterface) Proxy.newProxyInstance(BInterface.class.getClassLoader(), new Class[]{BInterface.class}, handler);
        proxy.call();
        log.info("targetClass={}", target.getClass());
        log.info("proxyClass={}", proxy.getClass());
    }
}
```

* `new TimeInvocationHandler(target)` : 동적 프록시에 적용할 핸들러 로직이다.
*  `Proxy.newProxyInstance(AInterface.class.getClassLoader(), new Class[]{AInterface.class}, handler)`
  * 동적 프록시는 `java.lang.reflect.Proxy` 를 통해서 생성할 수 있다.
  * 클래스 로더 정보, 인터페이스, 그리고 핸들러 로직을 넣어주면 된다. 그러면 해당 인터페이스를 기반으로 동적 프록시를 생성하고 그 결과를 반환한다.

**dynamicA() 출력 결과**
```
TimeInvocationHandler - TimeProxy 실행
AImpl - A 호출
TimeInvocationHandler - TimeProxy 종료 resultTime=0
JdkDynamicProxyTest - targetClass=class hello.proxy.jdkdynamic.code.AImpl
JdkDynamicProxyTest - proxyClass=class com.sun.proxy.$Proxy1
```
출력 결과를 보면 프록시가 정상 수행된 것을 확인할 수 있다.


## 🔔 설명

**생성된 JDK 동적 프록시**  
`proxyClass=class com.sun.proxy.$Proxy1` 이 부분이 동적으로 생성된 프록시 클래스 정보이다.  
이것은 우리가 만든 클래스가 아니라 JDK 동적 프록시가 이름 그대로 동적으로 만들어준 프록시이다. 이 프록시는 `TimeInvocationHandler` 로직을 실행한다.

**실행 순서**
1. 클라이언트는 JDK 동적 프록시의 `call()` 을 실행한다.
2. JDK 동적 프록시는 `InvocationHandler.invoke()` 를 호출한다. `TimeInvocationHandler` 가 구현체로 있으로 `TimeInvocationHandler.invoke()` 가 호출된다.
3. `TimeInvocationHandler` 가 내부 로직을 수행하고, `method.invoke(target, args)` 를 호출해서 `target` 인 실제 객체( `AImpl` )를 호출한다.
4. `AImpl` 인스턴스의 `call()` 이 실행된다.
5. `AImpl` 인스턴스의 `call()` 의 실행이 끝나면 `TimeInvocationHandler` 로 응답이 돌아온다. 시간 로그를 출력하고 결과를 반환한다.

**실행 순서 그림**

![img30](https://user-images.githubusercontent.com/93430103/164450471-2f332a43-8296-4b4d-b628-48cab2f189f2.png)

**동적 프록시 클래스 정보**  
`dynamicA()` 와 `dynamicB()` 둘을 동시에 함께 실행하면 JDK 동적 프록시가 각각 다른 동적 프록시 클래스를 만들어주는 것을 확인할 수 있다.

```
proxyClass=class com.sun.proxy.$Proxy1 //dynamicA
proxyClass=class com.sun.proxy.$Proxy2 //dynamicB
```

## 🔔 정리
예제를 보면 `AImpl` , `BImpl` 각각 프록시를 만들지 않았다. 프록시는 JDK 동적 프록시를 사용해서 동적으로 만들고 `TimeInvocationHandler` 는 공통으로 사용했다.  
JDK 동적 프록시 기술 덕분에 적용 대상 만큼 프록시 객체를 만들지 않아도 된다. 그리고 같은 부가 기능 로직을 한번만 개발해서 공통으로 적용할 수 있다. 만약 적용 대상이 100개여도 동적 프록시를 통해서 생성하고, 각각 필요한 `InvocationHandler` 만 만들어서 넣어주면 된다.  
결과적으로 프록시 클래스를 수 없이 만들어야 하는 문제도 해결하고, 부가 기능 로직도 하나의 클래스에 모아서 단일 책임 원칙(SRP)도 지킬 수 있게 되었다.

JDK 동적 프록시 없이 직접 프록시를 만들어서 사용할 때와 JDK 동적 프록시를 사용할 때의 차이를 그림으로 비교해보자.

**JDK 동적 프록시 도입 전 - 직접 프록시 생성**

![img31](https://user-images.githubusercontent.com/93430103/164451260-cc327bc9-650d-4b29-8fd4-a6bfafa5823c.png)

**JDK 동적 프록시 도입 후**

![img32](https://user-images.githubusercontent.com/93430103/164451262-d02c31c0-d3c4-4bd4-be59-9666517e29c3.png)

* 점선은 개발자가 직접 만드는 클래스가 아니다.

**JDK 동적 프록시 도입 전**

![img33](https://user-images.githubusercontent.com/93430103/164451544-f4675acb-fe84-45d3-ae17-61551174d37c.png)

**JDK 동적 프록시 도입 후**

![img34](https://user-images.githubusercontent.com/93430103/164451560-d640d244-a444-41ef-8c25-210c0e2cd62b.png)

## 🔔 JDK 동적 프록시 - 한계
JDK 동적 프록시는 인터페이스가 필수이다.  
그렇다면 인터페이스 없이 클래스만 있는 경우에는 어떻게 동적 프록시를 적용할 수 있을까?  
이것은 일반적인 방법으로는 어렵고 `CGLIB` 라는 바이트코드를 조작하는 특별한 라이브러리를 사용해야 한다.

`CGLIB`에 대해서는 다음번에 이어서 알아보자.

<br>

[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}
<br>
