---
title:  "프록시 팩토리" 
excerpt: "프록시 팩토리에 대해 알아보자."

categories:
  - Spring
tags:
  - [프록시 팩토리]

toc: true
toc_sticky: true

date: 2022-04-28
last_modified_at: 2022-04-28
---

인프런에 있는 김영한 님의 **스프링 핵심 원리 - 고급편** 강의를 듣고 정리한 내용 입니다. 😀    
[🌜 [스프링 핵심원리 - 고급편]강의 들으러 가기!](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-%ED%95%B5%EC%8B%AC-%EC%9B%90%EB%A6%AC-%EA%B3%A0%EA%B8%89%ED%8E%B8/dashboard){:target="_blank"}
{: .notice--warning}

지난번에 알아 본 [JDK동적 프록시](https://sapzilking.github.io/java/JdkDynamicProxy/) 는 인터페이스가 필수였다.  
인터페이스 없이 클래스만 있는 경우에 동적 프록시를 적용하는 방법에 대해 알아보자.  
이것은 일반적인 방법으로는 어렵고 CGLIB 라는 바이트코드를 조작하는 특별한 라이브러리를 사용해야 한다.

## 🔔 CGLIB - 소개



<br>

[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}
<br>
