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

위 그림을 보며 인증 섹션을 **정리**해보자면 아래와 같다.
- #1 기본 인증은 AuthenticationManager라는 인터페이스의 authenticate method를 사용한다(기본 구현체와 다른 검증 로직이 필요하면 authenticate 메소드를 override하면 된다).
- #2 하지만 authentication객체 뿐만 아니라 authentication클래스를 상속한 더 많은 객체들에 대한 인증을 지원하기 위해서 supports메소드를 가지고 있는 AuthenticationProvider 인터페이스를 사용한다.
- #3 이 **AuthenticationProvider 인터페이스를 구현한 구현체**를 **ProviderManager**라고 부른다.(그림에서도 볼 수 있듯이 ProviderManager가 AuthenticationProvider 인터페이스를 구현한다.)
- #4 ProviderManager는 하나만 존재해도 되지만 그림처럼 여러 개 존재할 수도 있는데, 이 경우 각각의 ProviderManager는 부모를 가진다.
- #5 ProviderManager의 부모도 역시 AuthenticationProvider 인터페이스를 구현한 구현체이고, 자식 ProviderManager의 인증이 성공하지 못했을 때 **fallback**으로서 동작한다.

## 사용자 정의 Authentication Managers
다음은 위에서 소개했던 global AuthenticationManager를 보여준다(AuthenticationProvider와 AuthenticationManager는 다르긴 하지만 일단 여기서는 유사하다고 생각해도 된다). @Configuration과 @Autowired를 사용하여 의존성을 주입받는다.  
```text
@Configuration
public class ApplicationSecurity extends WebSecurityConfigurerAdapter {

   ... // web stuff here

  @Autowired
  public void initialize(AuthenticationManagerBuilder builder, DataSource dataSource) {
    builder
    .jdbcAuthentication()
    .dataSource(dataSource)
    .withUser("dave")
    .password("secret").roles("USER");
  }
}
```
WebSecurityConfigurerAdapter를 구현한 클래스에서는 위에서 볼 수 있듯이 AuthenticationManagerBuilder을 사용하면, in-memory나 jdbc를 편하게 세팅(인증에 대해)을 할 수 있다. 추가적으로 이 클래스에서는 **커스텀 UserDetailsService**를 추가하기 용이하다. [스프링부트 customOAuth2UserService을 구현한 코드 예시](https://github.com/ingkellswith/ingkellswith-springboot/blob/master/src/main/java/com/springboot/ingkellswith/springboot/config/auth/SecurityConfig.java)

위는 global AuthenticationManager를 만드는 코드인 반면 아래는 local AuthenticationManager를 만드는 코드이다.

```text
@Configuration
public class ApplicationSecurity extends WebSecurityConfigurerAdapter {

  @Autowired
  DataSource dataSource;

   ... // web stuff here

  @Override
  public void configure(AuthenticationManagerBuilder builder) {
    builder
    .jdbcAuthentication()
    .dataSource(dataSource)
    .withUser("dave")
    .password("secret")
    .roles("USER");
  }
}
```
달라진 점은
- #1 datasource을 autowired로 필드 주입방식으로 주입받는다.
  - datasource에 대해서 global AuthenticationManager에서는 메소드에서 의존성을 주입받았고, local AuthenticationManager에서는 datasource를 필드 주입방식(의존성 주입방식의 한 종류)을 사용했다(사실 이 부분은 크게 중요하진 않지만 차이점을 거론하기 위해 작성했다).
- #2 **메소드가 initialize에서 configure로 바뀌었다.**
  - initialize라는 이름의 메소드는 사용자가 언제든지 이름을 바꿀 수 있지만, configure는 이미 정의된 것이므로 @Override해야 한다.

위 두개의 ApplicationSecurity클래스 중 전역 manager가 되느냐 로컬 manager를 판단하느냐의 여부는 **@Autowired와 @Override**이다. **@Autowired를 사용하면** 전역적으로 사용되는 싱글톤 빈을 주입받는 것이기에 전역 manager가 되고, **@Override를 사용하면** 빈을 주입받는 것이 아닌 단지 메소드 오버라이딩에 해당하기 때문에 local manager가 되고, 이 local manager는 얼마든지 더 생성이 가능하다.

## 인가(Authorization or Access Control)
인증 단계가 성공하면 인가 단계로 이동한다. Spring Security의 코어 전략은 **AccessDecisionManager**이다. 이 프레임워크에서는 이에 대해 3개의 구현체를 제공한다, 그리고 이 3개의 delegate(three delegate to a chain of **AccessDecisionVoter** instances)들은 ProviderManager delegates to AuthenticationProviders와 유사하다.  

하나의 **AccessDecisionVoter**는 하나의 **Authentication**(representing a principal)과 하나의 **secure Object**(which has been decorated with **ConfigAttributes**)을 고려한다.  

```text
boolean supports(ConfigAttribute attribute);

boolean supports(Class<?> clazz);

int vote(Authentication authentication, S object, Collection<ConfigAttribute> attributes);
```

**vote메소드를 보면 Authentication 하나, 제네릭 Object 하나, ConfigAttribute 컬렉션을 사용하고 있음을 알 수 있다.** 여기에서 Object는 in the signatures of the **AccessDecisionManager** and **AccessDecisionVoter**인 한 완전히 generic이다. 이 Object는 유저가 접근을 원하는 어떠한 것이라도 나타낼 수 있다. 일반적으로 그것은 웹 리소스나 java class의 메소드이다. ConfigAttribute는 완전히 generic하진 않지만, fairly generic하고, 앞서 설명한 Object에 대해 메타데이터(that determines the level of permission required to access it)를 마치 장식처럼 추가할 수 있다. 즉 **ConfigAttribute**는 자원에 액세스할 수 있는 권한을 결정한다.

**ConfigAttribute는 인터페이스**이다. 이 인터페이스의 메서드는 1개밖에 없기 때문에(**quite generic하며 String을 반환한다**), 이러한 문자열은 리소스 소유자의 의도를 인코딩하여, 자원에 누가 액세스할 수 있는지 그 규칙을 나타낸다. 일반적으로, **ConfigAttribute**는 사용자 역할의 이름(ROLE_ADMIN이나 ROLE_AUDIT 등)으로, 대부분의 경우 ROLE_Prefix와 같이 나타낸다. 그러니까 기본적으로 db에 권한을 저장할 때 기본적으로 'ROLE_'을 prefix로 사용한다는 것이다.

대부분의 사람들은 AffirmativeBased 속성: (한 개의 voter라도 affirm하면 액세스가 허용되는 것)을 가진 default **AccessDecisionManager**을 사용한다. 커스터마이징은 voter편에서 발생하는 경향이 있다. voter측에서는 **adding new ones** or **modifying the way that the existing ones work** 한다.  

Spring Expression Language(SpEL)표현인 **ConfigAttribute**를 사용하는 것은 매우 일반적이다. 예를 들어 SpEL은 isFullyAuthenticated() && hasRole('user') 같은 것이다. 이는 SpEL을 처리하고 SpEL을 위한 컨텍스트를 만들 수 있는 **AccessDecisionVoter**에 의해 지원된다. 처리할 수 있는 표현의 범위를 확장하려면 **SecurityExpressionRoot** 또는 **SecurityExpressionHandler** 의 커스텀 구현이 필요하다.

## Web Security
웹 계층(UI 및 HTTP 백엔드)의 Spring Security는 **Servlet Filters**를 기반으로 한다. 다음 그림은 단일 HTTP 요청에 대한 핸들러의 일반적인 계층을 보여준다.  

![spring-security-webfilter](https://user-images.githubusercontent.com/55550753/160280935-2dfbacd2-0dc2-471f-8edb-52963e2a6c68.PNG)  

클라이언트는 응용 프로그램에 요청을 전송하고 컨테이너는 요청 URI 경로에 따라 응용 프로그램에 적용할 **필터**와 **서블릿**을 결정한다. 최대 1개의 서블릿이 단일 요청을 처리할 수 있지만 **필터가 체인을 형성**하므로 **필터가 정렬**된다. 실제로 필터는 요청 자체를 처리하려는 경우 체인의 나머지 부분에 **거부권을 행사**할 수 있다. 필터는 다운스트림필터 및 서블릿에서 사용되는 **요청 또는 응답을 수정**할 수도 있다. **필터 체인의 순서는 매우 중요하며 Spring Boot에서는 두 가지 메커니즘을 통해 관리한다.** 이 두가지 방법은 Spring Bean의 @Order를 사용하는 방법 또는 Ordered를 구현하는 방법이다.

또, 필터 체인은 API의 일부로서 순서가 설정되어 있는 **FilterRegistrationBean**의 일부가 될 수도 있다. 일부 기성 필터는 서로 상대적인 순서를 나타내는 데 도움이 되는 자체 상수를 정의한다. 예를 들어, 스프링 세션의 **Session Repository Filter**에는 정수인 DEFAULT_ORDER가 있다. 이 DEFAULT_ORDER에 'MIN_VALUE + 50'라고 작성하면 체인의 초기에 존재하지만, 그 앞에 오는 다른 필터도 배제하지 않음을 나타낸다.
