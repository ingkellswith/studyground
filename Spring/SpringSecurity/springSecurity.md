Spring Security
======================
스프링 시큐리티 학습 정리글입니다.
# Reference
[Spring Security 5.6.2(2022-03-24 current version) 공식문서](https://docs.spring.io/spring-security/reference/index.html)   
[Spring Security 공식 가이드 - Securing a Web Application](https://spring.io/guides/gs/securing-web/)  
[Spring Security 공식 토픽 - Spring Security Architecture](https://spring.io/guides/topicals/spring-security-architecture)  

위 **3가지 링크**가, 2022-03-26 현재 스프링 공식문서 프로젝트 페이지에서 스프링 시큐리티를 소개하는 모든 페이지이다.  
Spring Security Oauth 프로젝트는 deprecated되어 Spring Security에서 Oauth2.0을 지원한다.  
따라서 Spring Security를 잘 이해하기 위해서는 Spring Security 프로젝트에서 공식적으로 소개하는 위 링크의 정보에 대해서 이해하는 것이 중요하다.  

# Spring Security 공식 토픽 - Spring Security Architecture

## Spring Security Architecture 공식 토픽 소개글
- "To resolve the confusion experienced by developers who use Spring Security, we take a look at the way security is applied in web applications by using **filters** and, more generally, by using **method annotations**."
- 웹 애플리케이션에 보안이 적용 되는 것을 **필터**와 **메소드 어노테이션**을 사용해서 살펴보겠다.

## 인증 및 인가(액세스 제어)
- '**인증**(Authorization):당신은 누구인가?' 와 '**인가**(Authorization or Access Control): 당신은 무엇을 할 수 있나?' 가 웹 애플리케이션 보안의 핵심이다.

## 인증(Authentication) 
"The main strategy interface for authentication is **AuthenticationManager**, which has only one method"
```text
public interface AuthenticationManager {
  Authentication authenticate(Authentication authentication) throws AuthenticationException;
}
```
- 메인 전략 인터페이스는 **AuthenticationManager**로 **authenticate**라는 하나의 메소드만을 가지고 있다.
- 이 **AuthenticationManager**가 인증을 처리한다.
- 예를 들어, formLogin 진행 시 아이디와 비밀번호를 이 메소드를 통해 처리하는 것이다.
- 인터페이스의 메소드를 살펴보면 authenticate함수에 Authentication객체를 전달하고 Authentication객체를 반환한다.
- 이 메소드는 3가지 중 하나를 수행한다. 
  - #1 입력으로 주체를 확인할 수 있는 경우 **Authentication 객체**를 반환한다(보통 이 객체 내부에 'authenticated=true'가 포함되어 있음).
  - #2 입력이 유효하지 않은 주체를 나타낸다고 판단되면 **AuthenticationException**을 throw한다.
  - #3 결정할 수 없는 경우 **null**을 반환한다.

```text
public interface AuthenticationProvider {
  Authentication authenticate(Authentication authentication) throws AuthenticationException;
  boolean supports(Class<?> authentication);
}
```
- 가장 일반적으로 사용되는 **AuthenticationManager 인터페이스를 구현한 구현체는 ProviderManager**(which delegates to a chain of AuthenticationProvider instances: AuthenticationProvider의 체인에 위임하는 인스턴스)이다.
- AuthenticationProvider는 AuthenticationManager와 상당히 유사하지만, 위 인터페이스에서 볼 수 있듯이 given Authentication type을 support하는 지 체크하는 기능이 추가되어 있다.
- 위 support 메소드의 Class<?> argument는 사실상 Class<? extends Authentication>로 봐도 무방하다.
- 따라서 **ProviderManager**는 인증을 **AuthenticationProviders의 체인**에 위임함으로서 다양한 인증 매커니즘을 지원할 수 있게 된다.
- **ProviderManager는 선택적으로 parent를 가진다**. 모든 provider가 null을 반환하면 parent를 찾아간다. parent까지 null을 반환할 경우, null Authentication은 AuthenticationException을 throw한다.
- 몇몇 애플리케이션은 보호된 자원에 접근하기 위한 논리적으로 분리된 그룹을 가진다.(예를 들어, all web resources that match a path pattern, such as /api/**)또한, 각각의 그룹은 그룹만의 AuthenticationManager(보통 ProviderManager로 불림)를 가질 수 있다. 그리고 이 ProviderManager는 부모를 공유한다. 부모는 “global” resource처럼 동작하며 모든 provider의 fallback(대체자)으로서 동작한다.
 
**ProviderManager를 사용한 An AuthenticationManager hierarchy**  
![spring-security-providermanager](https://user-images.githubusercontent.com/55550753/160243644-df118bea-a02f-4136-9930-dbb31886d3c7.PNG)  

- 위 그림을 보며 인증 섹션을 정리해보자면 아래와 같다.
  - #1 기본 인증은 AuthenticationManager라는 인터페이스의 authenticate method를 사용한다(기본 구현체와 다른 검증 로직이 필요하면 authenticate 메소드를 override하면 된다).
  - #2 하지만 authentication객체 뿐만 아니라 authentication클래스를 상속한 더 많은 객체들에 대한 인증을 지원하기 위해서 supports메소드를 가지고 있는 AuthenticationProvider 인터페이스를 사용한다.
  - #3 이 **AuthenticationProvider 인터페이스를 구현한 구현체**를 **ProviderManager**라고 부른다.(그림에서도 볼 수 있듯이 ProviderManager가 AuthenticationProvider 인터페이스를 구현한다.)
  - #4 ProviderManager는 하나만 존재해도 되지만 그림처럼 여러 개 존재할 수도 있는데, 이 경우 각각의 ProviderManager는 부모를 가진다.
  - #5 ProviderManager의 부모도 역시 AuthenticationProvider 인터페이스를 구현한 구현체이고, 자식 ProviderManager의 인증이 성공하지 못했을 때 fallback으로서 동작한다.

