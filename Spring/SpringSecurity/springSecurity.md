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

Spring Security는 체인에 단일 필터로 설치되며 구체적인 타입은 **FilterChainProxy**이다. Spring Boot 어플리케이션에서 **security filter**는 ApplicationContext의 Bean으로 등록되고 디폴트로 설치되어 모든 request에 적용된다. **Spring Security는 필터로서 SecurityProperties에서 정의한 위치에 설치된다.** FilterRegistrationBean.REQUEST_WRAPPER_FILTER_MAX_ORDER(Spring Boot 어플리케이션이 요구를 랩하여 동작을 변경하는 경우 필터가 가질 것으로 예상되는 최대 순서)에 의해 DEFAULT_FILTER_ORDER는 고정된다. 컨테이너의 관점에서 Spring Security는 **단일 필터**이지만 그 내부에는 각각 특별한 역할을 하는 **추가 필터**가 있다. 다음 그림은 이 관계를 나타낸다.  

![image](https://user-images.githubusercontent.com/55550753/160423522-1d8de7d7-4bfe-4c16-ab19-778d99650b02.png)  
"Spring Security is a single physical Filter but **delegates processing to a chain of internal filters**" : 스프링 시큐리티는 하나의 물리적 필터이지만 내부의 필터 체인에 그 처리를 위임한다.  

실제로 **보안 필터**에는 다음과 같은 간접 레이어가 1개 더 있다: 보통 컨테이너에 **DelegatingFilterProxy**로 설치되고, 이것은 Servlet Spec이기에 Spring Bean이 아니다. **DelegatingFilterProxy는 요청을 FilterChainProxy에 위임한다.** **FilterChainProxy**는 항상 Bean이며 보통 **springSecurityFilterChain**이라는 고정 이름을 가진다. **FilterChainProxy**에는 필터의 체인으로서 모든 보안 로직이 정렬되어 있다. 모든 필터는 동일한 API를 가지며(모든 필터는 Servlet Specification에서 필터 인터페이스를 구현), 모든 필터는 체인의 나머지 부분을 거부할 수 있다.

![image](https://user-images.githubusercontent.com/55550753/160426215-d5bdce54-2f5d-4268-8d99-5d15cfb55772.png)   
출처 : https://logical-code.tistory.com/194  

위는 스프링 시큐리티의 구조를 조금 더 상세하게 설명한 이미지인 것 같아 남겨본다.  

![image](https://user-images.githubusercontent.com/55550753/160637148-6b886c63-a441-49b3-a981-b52a3cedd985.png)  

또한 위 그림처럼 하나의 **FilterChainProxy**에 여러 개의 필터 체인이 있을 수 있다(매칭되는 원리는 /foo/** 가 /** 에 선행되어 매칭된다). 중요한 점은 요청을 처리하는 필터 체인은 여러 개의 필터 체인 중에서 **하나**라는 점이다.  

기본 스프링 부트 애플리케이션에는 여러 개의(n개) 필터 체인이 있으며, 여기서 보통 n은 6이다. **첫 n-1개의 필터 체인**은 /css/* 및 /images/** 등의 스태틱자원 패턴과 오류 뷰인 /error를 무시하기 위해서 존재한다(이 path는 SecurityProperties Configuration bean에서 security.ignored를 사용하여 사용자가 제어할 수 있다). **1개의 마지막 체인**은 catch-all path(/**)와 일치하며 **인증, 인가, 예외 처리, 세션 처리, 헤더 쓰기** 등의 로직이 포함되어 있다. 이 체인에는 기본적으로 총 11개의 필터가 있지만 일반적으로 사용자는 어떤 필터가 언제 사용되는지 걱정할 필요가 없다.

**참고** : **Spring Security 내부의 모든 필터가 스프링 컨테이너에 인식되지 않는다는 사실**은 특히 Filter 유형의 모든 @Beans가 기본적으로 컨테이너에 자동으로 등록되는 Spring Boot 어플리케이션에서 중요하다. 따라서 체인필터에 커스텀필터를 추가할 경우 @Bean으로 하지 않거나, 컨테이너 등록을 명시적으로 disable 하는 FilterRegistrationBean으로 wrap해야 한다.
https://stackoverflow.com/questions/49654751/how-do-i-add-a-filter-to-spring-security-based-on-a-profile

## Creating and Customizing Filter Chains

스프링 부트의 기본 **fallback** 필터 체인(the one with the /** request matcher)은 SecurityProperties.BASIC_AUTH_ORDER의 순서가 기본적으로 정의되어 있다. 이는 security.basic.enabled=false으로 off하거나, you can use it as a fallback and define other rules with a lower order : fallback으로서 사용하고 다른 규칙을 더 우선순위로 선언할 수 있다. 아래는 예시인데, **@Order에서 -10을 한 이유는 위에서 기본 필터가 11개 있다고 말한 것 때문인데 -10을 하면 제일 먼저 사용되는 필터가 된다, @Configuration을 선언해 빈으로 등록한 이유는 이를 fallback필터로 사용하기 위함이다(이 말인 즉슨 fallback필터로 사용하지 않으려면 빈으로 등록하지 않으면 되겠다).**

```text
@Configuration
@Order(SecurityProperties.BASIC_AUTH_ORDER - 10)
public class ApplicationConfigurerAdapter extends WebSecurityConfigurerAdapter {
  @Override
  protected void configure(HttpSecurity http) throws Exception {
    http.antMatcher("/match1/**")
     ...;
  }
}
```

위 빈을 사용하면 Spring Security는 fallback 필터 전에 새로운 필터 체인으로 위 빈을 추가한다(**조금 헷갈릴 수 있는데 위 경우는 새로운 필터를 빈으로 등록해 fallback 필터로 사용함과 동시에 기존 fallback보다는 순위가 낮기에 fallback 필터 체인 중에서 먼저 호출되는 fallback필터 임을 말하는 것 같다**).  

또한 리소스 세트 별로(ex. /api/admin/*, /api/user/*) 액세스 권한이 다른데, **각 리소스 세트에는 고유한 순서와 고유한 request matcher가 있는 자체 WebSecurityConfigurerAdapter가 있다.** 일치 규칙이 겹치면 가장 먼저 정렬된 필터 체인이 이긴다.  

## Request Matching for Dispatch and Authorization

**A security filter chain(또는 WebSecurityConfigrAdapter)**에는 HTTP request에 적용할지를 결정하기 위해 사용되는 request matcher가 있다. **특정 필터 체인을 적용하기로 결정되면 다른 필터 체인은 적용되지 않는다(여기에서 아래의 ApplicationConfigurerAdapter를 하나의 필터 체인이라고 볼 수 있고, 여기에서 request matcher에 의해 매칭이 되면 다른 필터 체인은 적용되지 않는다는 뜻인 것 같다.**). 다만, 필터 체인내에서, 다음과 같이 HttpSecurity configurer에 추가 matcher를 설정하는 것으로, **인가(Authorization)**를 보다 세밀하게 제어할 수 있다.

```text
@Configuration
@Order(SecurityProperties.BASIC_AUTH_ORDER - 10)
public class ApplicationConfigurerAdapter extends WebSecurityConfigurerAdapter {
  @Override
  protected void configure(HttpSecurity http) throws Exception {
    http.antMatcher("/match1/**")
      .authorizeRequests()
        .antMatchers("/match1/user").hasRole("USER")
        .antMatchers("/match1/spam").hasRole("SPAM")
        .anyRequest().isAuthenticated();
  }
}
```

Spring Security를 설정할 때 가장 쉽게 저지르는 오류 중 하나는 이러한 matcher들이 **다른 프로세스에 적용**된다는 것을 잊는 것이다. 하나는 필터 체인 전체에 적용되는 request matcher이며, 다른 하나는 적용할 액세스 규칙을 선택하는 것이다. 

풀이하면, 위 `ApplicationConfigurerAdapter`클래스에서 `http.antMatcher("/match1/**")`는 필터 체인 전체에 적용되는 request matcher이며,
```text
.authorizeRequests()
.antMatchers("/match1/user").hasRole("USER")
.antMatchers("/match1/spam").hasRole("SPAM")
.anyRequest().isAuthenticated();
```
이 부분은 적용할 액세스 규칙을 선택하는 것이다.

## Combining Application Security Rules with Actuator Rules

엔드포인트에 **Spring Boot Actuator**를 사용하는 경우 보호가 필요할 수 있으며 기본적으로는 보호된다. 실제로 **Spring Boot Actuator**를 보안 애플리케이션에 추가하는 즉시 **Spring Boot Actuator** 엔드포인트에만 적용되는 추가 필터 체인이 생성된다. **Spring Boot Actuator**의 엔드포인트에만 일치하는 request matcher와 함께 정의되며 **ManagementServerProperties.BASIC_AUTH_ORDER**의 순서를 가진다. **ManagementServerProperties.BASIC_AUTH_ORDER**는 기본 SecurityProperties fallback 필터보다 5 은 BASIC_AUTH_ORDER이므로 **fallback 전에** 참조된다.

보안 규칙을 **Actuator Endpoint**에 적용하려면, 기본 actuator 필터(바로 위에서 설명한 폴백 필터보다 5만큼 먼저 호출되는 필터)보다 먼저 거쳐야 하고, 모든 **Actuator Endpoint**를 포함하는 **request matcher**가 있는 필터 체인을 추가할 수 있다. 이 경우, **Actuator의 기본 필터**보다 늦게, **fallback 필터**보다 빠른 독자적인 필터를 추가하는 것이 가장 간단하다(예를 들면, **Management Server Properties.BASIC_AUTH_ORDER + 1**). 다음은 이를 나타낸다.

```text
@Configuration
@Order(ManagementServerProperties.BASIC_AUTH_ORDER + 1)
public class ApplicationConfigurerAdapter extends WebSecurityConfigurerAdapter {
  @Override
  protected void configure(HttpSecurity http) throws Exception {
    http.antMatcher("/foo/**")
     ...;
  }
}
```

## Method Security

Spring Security는 웹 응용 프로그램의 보안 보호와 더불어 **Java 메서드 실행**에 액세스 규칙을 적용할 수 있도록 지원한다. Spring Security의 경우 이는 단지 다른 유형의 "보호된 리소스"일 뿐이다(즉 웹 계층의 보안이나, 메소드 보안이나 결국 보호되는 자원인 것이다). 접근 규칙은 같은 형식의 ConfigAttribute 문자열(for example, roles or expressions)을 사용하여 선언된다. 이 메소드 보안을 위한 첫 번째 단계는 **메서드 보안**을 유효하게 하는 것이다. 예를 들어 어플리케이션의 최상위 설정에서는 다음과 같다.

```text
@SpringBootApplication
@EnableGlobalMethodSecurity(securedEnabled = true)
public class SampleSecureApplication {
}
```

이후, 아래 `@Secured("ROLE_USER")` 처럼 메소드에 직접적으로 method security를 설정할 수 있다.

```text
@Service
public class MyService {
  @Secured("ROLE_USER")
  public String secure() {
    return "Hello Security";
  }
}
```

이 예는 @Service를 사용해 method security를 구현한 예이다. **스프링이 이 타입의 빈을 생성하면, 이 빈은 프록시처리되고, 호출자는 메소드가 실제로 호출되기 전에 security interceptor를 반드시 거쳐야 한다.** 또한, Access가 거부되면 AccessDeniedException를 발생시킨다.

메소드에 security constraints을 적용하기 위해 사용할 수 있는 다른 주석(특히 `@PreAuthorize` 및 `@PostAuthorize`)도 있다. 이 역시 `@Secured`처럼 메소드 어노테이션으로 사용하며, **method parameters와 return values**에 대한 참조를 어노테이션 레벨에서 할 수 있다는 장점이 존재한다.

참고 : 필터 체인은 **인증(Authentication)**이나 **로그인 페이지로 리다이렉트** 등의 사용자 경험 기능을 제공할 수 있다.

## Working with Threads

Spring Security는 기본적으로 thread-bound되어 있다. 이는 현재 **인증된 principal(본인)**을 다양한 **다운스트림 소비자**가 이용할 수 있도록 해야 하기 때문이다. **기본 빌딩 블록은 Security Context이다(The basic building block is the SecurityContext)**. SecurityContext는 Authentication객체를 포함할 수 있다(사용자가 로그인하면 Authentication객체는 explicitly authenticated된다). SecurityContextHolder의 정적 편의 메서드를 통해 SecurityContext에 언제든지 액세스하여 조작할 수 있다. 이것은 결국, a ThreadLocal을 조작하는 것과 같다. 다음으로 이것의 예시이다.

```text
SecurityContext context = SecurityContextHolder.getContext();
Authentication authentication = context.getAuthentication();
assert(authentication.isAuthenticated);
```

정리하면, SecurityContextHolder > SecurityContext > Authentication(**isAuthenticated 및 principal을 필드로 가지고 있는 객체이며, 또한 principal은 user 엔티티같은 것을 사용해 얼마든지 사용자 정의할 수 있다** : **The type of the Principal in an Authentication is dependent on the AuthenticationManager used to validate the authentication**)이다.

웹 엔드포인트에서 현재 인증된 사용자에 대한 액세스가 필요한 경우 다음과 같이 **@RequestMapping에서 메서드 파라미터**를 사용할 수 있다.

```text
@RequestMapping("/foo")
public String foo(@AuthenticationPrincipal User user) {
  ... // do stuff with user
}
```

`@AuthenticationPrincipal`는 SecurityContext에서 Authentication을 꺼내고 getPrincipal() 메서드를 호출하여 메서드 파라미터를 생성하는 역할을 수행한다. 또한, Spring Security가 사용 중인 경우 **HttpServletRequest의 Principal은 Authentication**이므로 직접 사용할 수도 있다. 아래는 직접 **Principal**을 사용하는 예시이다. 

```text
@RequestMapping("/foo")
public String foo(Principal principal) {
  Authentication authentication = (Authentication) principal;
  User = (User) authentication.getPrincipal();
  ... // do stuff with user
}
```