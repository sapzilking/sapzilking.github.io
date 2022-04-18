---
title:  "커스텀 태그" 
excerpt: "커스텀 태그 사용법에 대해 알알보자"

categories:
  - JUnit
tags:
  - [customTag]

toc: true
toc_sticky: true

date: 2022-04-18
last_modified_at: 2022-04-18
---

인프런에 있는 백기선 님의 **더 자바, 애플리케이션을 테스트하는 다양한 방법** 강의를 듣고 정리한 내용 입니다. 😀    
[🌜 [더 자바, 애플리케이션을 테스트하는 다양한 방법]강의 들으러 가기!](https://www.inflearn.com/course/the-java-application-test/dashboard){:target="_blank"}
{: .notice--warning}

커스텀 태그를 만드는 방법에 대해 알아보자.

## 🔔 커스텀 태그

아래와 같은 테스트가 있다.

**StudyTest.java**

```java
//커스텀 태그 적용 전
@DisplayName("스터디 만들기 fast")
@Tag("fast")
@Test
void create_new_study() {
    System.out.println("fast_test 실행");
}

//커스텀 태그 적용 후
@DisplayName("스터디 만들기 fast")
@FastTest
void create_new_study() {
    System.out.println("fast_test 실행");
}
```

**FastTest.java**

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
@Test
@Tag("fast")
public @interface FastTest {
}
```
@Test와 @Tag 두개의 어노테이션을 `메타 어노테이션`으로 사용해서 FastTest라는 `composed 어노테이션`을 만들었다.  
이렇게 만든 FastTest 어노테이션은 기존의 방식과 동일한 `semantic`으로 사용할 수 있다.

기존에 사용하던 방식은 `@Tag("fast")` 부분이 문자열 이므로 `type-safe` 하지 않다.  
충분히 오타가 발생할 수 있으므로 가급적 오타를 줄이고 테스트를 진행 하기 위해 커스텀 어노테이션을 만든 후 사용하는게 좋을 것이다.


<br>

[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}
<br>
