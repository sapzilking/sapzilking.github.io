---
title:  "동시성 문제란?" 
excerpt: "동시성 문제의 개념에 대해 알아보자"

categories:
  - spring
tags:
  - [spring, 동시성 문제]

toc: true
toc_sticky: true

date: 2022-04-02
last_modified_at: 2022-04-02
---

동시성 문제란?

```java
@Slf4j
public class FieldService {

    private String nameStore;

    public String logic(String name) {
        log.info("저장 name{} => nameStore={}", name, nameStore);
        nameStore = name;
        sleep(1000);
        log.info("조회 nameStore={}", nameStore);
        return nameStore;
    }

    private void sleep(int millis) {
        try {
            Thread.sleep(millis);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```
위의 코드는 param으로 받은 name값을 nameStore에 저장하고 1초 후에 nameStore의 값을 조회하는 간단한 코드이다.  

위의 코드로 아래와 같은 테스트를 진행해 보자.

```java
@Slf4j
public class FieldServiceTest {

    private FieldService fieldService = new FieldService();

    @Test
    void field() {
        log.info("main start");
        Runnable userA = () -> {
            fieldService.logic("userA");
        };
        Runnable userB = () -> {
            fieldService.logic("userB");
        };

        Thread threadA = new Thread(userA);
        threadA.setName("thread-A");
        Thread threadB = new Thread(userB);
        threadB.setName("thread-B");

        threadA.start();
//        sleep(2000); // 동시성 문제 발생X
        sleep(100); // 동시성 문제 발생O
        threadB.start();

        sleep(3000); // 메인 쓰레드 종료 대기
        log.info("main exit");
    }

    private void sleep(int millis) {
        try {
            Thread.sleep(millis);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

위의 테스트의 실행결과 이다.  

이미지  

userB가 2번 조회 되었다.  
thread-A는 userA를 저장했지만 조회해온 값은 userB가 조회 되었다.

정리하자면 다음과 같다.
* Thread-A는 userA를 nameStore에 저장했다.  
* Thread-B는 userB를 nameStore에 저장했다.
* Thread-A는 userB를 nameStore에서 조회했다.
* Thread-B는 userB를 nameStore에서 조회했다.

이 처럼 여러 쓰레드가 동시에 같은 인스턴스의 필드 값을 변경하면서 발생하는 문제를 `동시성 문제`라고 한다. 

>당연한 이야기지만 `지역 변수`는 쓰레드마다 각각 다른 메모리 영역이 할당되므로 동시성 문제가 발생하지 않는다.  
`동시성 문제`가 발생하는 곳은 같은 인스턴스의 필드(주로 싱글톤에서 자주 발생), 또는 static 같은 공용필드에 접근할 때 발생한다.   
`동시성 문제`는 값을 읽기만 하면 발생하지 않는다. 어디선가 값을 변경하기 때문에 발생한다.

스프링의 빈 객체는 기본적으로 싱글톤이기 때문에 이러한 문제가 발생할 수 있다.

그렇다면 이러한 동시성 문제를 해결하기 위한 방법에는 어떤 방법이 있을까?  
다음시간에 이어서 알아보자.

참고 : [스프링 핵심 원리 - 고급편](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-%ED%95%B5%EC%8B%AC-%EC%9B%90%EB%A6%AC-%EA%B3%A0%EA%B8%89%ED%8E%B8/dashboard)

[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}
