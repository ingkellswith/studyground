Spring Security
======================
# Reference
[Spring Security 5.6.2(2022-03-24 current version) 공식문서](https://docs.spring.io/spring-security/reference/index.html)   
[Spring Security 공식 가이드 - Securing a Web Application](https://spring.io/guides/gs/securing-web/)  
[Spring Security 공식 토픽 - Spring Security Architecture](https://spring.io/guides/topicals/spring-security-architecture)  

위 **3가지 링크**가, 스프링 공식문서 프로젝트 페이지에서 스프링 시큐리티를 소개하는 모든 페이지이다.  
Spring Security Oauth 프로젝트는 deprecated되어 Spring Security에서 Oauth2.0을 지원한다.  
따라서 Spring Security를 잘 이해하기 위해서는 Spring Security 프로젝트에서 공식적으로 소개하는 위 링크의 정보에 대해서 이해하는 것이 중요하다.  

# Spring Security 공식 토픽 - Spring Security Architecture

## Spring Security Architecture 공식 토픽 소개글
- To resolve the confusion experienced by developers who use Spring Security, we take a look at the way security is applied in web applications by using **filters** and, more generally, by using **method annotations**.
- 웹 애플리케이션에 보안이 적용 되는 것을 **필터**와 **메소드 어노테이션**을 사용해서 살펴보겠다.

## 인증 및 인가(액세스 제어)
- '**인증**(Authorization):당신은 누구인가?' 와 '**인가**(Authorization or Access Control): 당신은 무엇을 할 수 있나?' 가 웹 애플리케이션 보안의 핵심이다.


