스프링 시큐리티 주요 개념 정리
===
# [스프링 시큐리티 공식 토픽](https://spring.io/guides/topicals/spring-security-architecture)
1. 인증을 처리하는 `AuthenticationManager`는 authenticate라는 하나의 메소드만을 가지고 있다. 예를 들어, formLogin 진행 시 아이디와 비밀번호를 이 메소드를 통해 처리한다.
2. `authenticate`함수에 `Authentication객체`를 전달하고 `Authentication객체`를 반환하는데 인증되기 전에는 `Authentication`의 필드인 `isAuthenticated`가 false인 상태이고, 함수 실행 후에 인증이 완료되면 `isAuthenticated`는 true가 된다.
3. `AuthenticationManager` 인터페이스를 구현한 구현체는 일반적으로 `ProviderManager`이다. `AuthenticationManager`와 기능이 유사하지만 다양한 `Authentication`타입을 지원하는 `supports`메소드가 추가되어 있다.`(ProviderManager -> AuthenticationProvider -> AuthenticationManager)`
4. `ProviderManager`는 선택적으로 parent를 가질 수 있는데 이 parent는 모든 supports메소드에서 타입 지원을 체크했음에도 실패할 시 `fallback`처럼 동작한다.
5. 인가의 권한을 판단하기 위해서 `AccessDecisionVoter`를 사용하는데, vote메소드에서 `authentication`객체와 접근 대상인 `Object`, 접근 권한인 `ConfigAttribute` 파라미터로 넘긴다. 이후 메소드가 실행되면 **vote여부**(권한 인가 성공 여부)를 int로 반환한다.
6. default `AccessDecisionManager`는 한 개의 voter라도 affirm하면 해당 자원에 액세스를 허용한다.

# [Servlet Applications - Architecture](https://docs.spring.io/spring-security/reference/servlet/architecture.html)
1. `DelegatingFilterProxy`는 서블릿 컨테이너의 lifecycle과 Spring’s ApplicationContext를 잇는 역할을 한다. 서블릿 필터는 스프링 컨테이너의 bean들을 인식할 수 없으므로 bean인 `FilterChainProxy`에 모든 작업을 위임한다.
2. 서블릿 컨테이너의 `DelegatingFilterProxy`에 의해 역할을 위임받는 `FilterChainProxy`는 일반적으로 bean이다.
3. `FilterChainProxy`는 스프링 시큐리티의 서블릿 지원의 시작점이다. 따라서 디버그하기 좋은 위치이다.
4. `FilterChainProxy`는 스프링 시큐리티의 핵심이다. 따라서 옵션으로 적용되지 않는 필수적인 기능을 실행하기에 좋은 위치이다. 예를 들어, 메모리 누수를 막기 위해 `ThreadLocal`로 사용된 SecurityContext를 clear하는 것이 있다.
5. 서블릿 컨테이너에서 **서블릿 Filter**는 오직 `url`에 근거해서 발동되지만, `FilterChainProxy`는 `HttpServletRequest`내부의 필드들을 사용할 수 있어 세밀한 매칭이 가능하다. 따라서 `FilterChainProxy`는 어떤 `Security Filter Chain`이 발동되어야 하는지를 결정할 수 있다.
6. 아래 그림에서 Bean Filter를 `FilterChainProxy`라고 해석할 수 있다.

![image](https://user-images.githubusercontent.com/55550753/162989585-7fc980ca-bc14-4443-b07f-59fd92b2780a.png)

# [Servlet Applications - Authentication - Authentication Architecture](https://docs.spring.io/spring-security/reference/servlet/authentication/architecture.html)
1. `SecurityContextHolder > SecurityContext > Authentication > principal, credentials, authorities` 순으로 객체의 필드가 정해지는 구조에서 `SecurityContextHolder`는 `ThreadLocal`을 사용해서 `SecurityContext`에 값을 할당한다. 따라서 같은 스레드의 메소드라면 굳이 파라미터로 넘길 필요 없이 항상 available하다.
![image](https://docs.spring.io/spring-security/reference/_images/servlet/authentication/architecture/securitycontextholder.png)
2. 스레드풀을 사용하는 구조에서 보안을 이유로 하거나 정확한 처리를 이유로 해서 **ThreadLocal인 SecurityContext를 사용 후에는 항상 thread를 clear**해야 하는데, 이를 `FilterChainProxy`에서 처리해준다.  
3. `GrantedAuthority`는 유저에게 허락된 권한을 나타낸다. **하나의 역할은 여러 개의 권한을 가질 수 있다.** 권한은 보통 `ROLE_ADMINISTRATOR` or `ROLE_HR_SUPERVISOR`이러한 형태로 쓰인다. `GrantedAuthority`s는 보통 `UserDetailsService`에 의해 load된다.
4. `AuthenticationEntryPoint`를 구현하는 것은 클라이언트로부터 `credential`을 요청하는 역할을 한다.이는 로그인 페이지로 리다이렉트하거나, WWW-Authenticate header로 응답하는 것이 될 수 있겠다.
5. `AbstractAuthenticationProcessingFilter`는 유저의 `credential`을 인증하는 base filter로 작동하고, 전형적으로 `AuthenticationEntryPoint`을 사용하여 `credential`을 요청한다.
![image](https://docs.spring.io/spring-security/reference/_images/servlet/authentication/architecture/abstractauthenticationprocessingfilter.png)  
   1. 유저가 `credentials`을 제출하면, `AbstractAuthenticationProcessingFilter`는 `HttpServletRequest`로부터 인증 과정을 거칠 `Authentication`객체를 생성한다. 생성될 `Authentication` 객체는 `AbstractAuthenticationProcessingFilter`의 하위클래스에 따라 다르다. 예를 들어,  For example, `UsernamePasswordAuthenticationFilter`는 `HttpServletRequest`로부터 `UsernamePasswordAuthenticationToken`을 생성한다.
   2. `Authentication`객체는 인증 과정을 거치기 위해 `AuthenticationManager`로 전달된다. 현재 `isAuthenticated`의 상태는 true가 아닌 것이다.
   3. 인증이 실패하면 3번 과정을 거치며, `SecurityContextHolder`는 clear out된다.
   4. 인증이 성공하면 4번 과정을 거친다.
      1. `SessionAuthenticationStrategy`가 새로운 로그인을 감지한다.
      2. `SecurityContextHolder`에 `Authentication`객체가 set된다. 그 후, `SecurityContextPersistenceFilter`가 `HttpSession`에 `SecurityContext`를 저장한다.
      3. 리멤버미 기능을 활성화했을 경우 `RememberMeServices.loginSuccess`가 발동된다.
      4. `ApplicationEventPublisher`는 `InteractiveAuthenticationSuccessEvent`를 발행한다.
      5.` AuthenticationSuccessHandler`가 동작한다.

# [Servlet Applications - Authentication - Username/Password - Reading Username/Password - Form](https://docs.spring.io/spring-security/reference/servlet/authentication/architecture.html)
1. 아래는 Form Login인증 과정이다.
![image](https://docs.spring.io/spring-security/reference/_images/servlet/authentication/unpwd/loginurlauthenticationentrypoint.png)
   1. 먼저 유저가 인증되지 않은 요청을 **/private** (유저는 이 url에 대해 권한이 없음)로 보낸다.
   2. `FilterSecurityInterceptor`는 인증되지 않은 요청에 대해 `AccessDeniedException`을 throw한다.
   3. 유저가 인증되지 않았기 때문에, `ExceptionTranslationFilter`는 설정된 `AuthenticationEntryPoint`을 사용해 로그인 페이지로 요청을 리다이렉트한다. 대부분의 케이스에서 `AuthenticationEntryPoint`은 `LoginUrlAuthenticationEntryPoint`의 인스턴스로 나타난다.
   4. 브라우저는 로그인 페이지로 get요청을 보낸다.
   5. 서버에서 로그인 페이지를 내려준다.
2. 아래는 username & password 인증과정이다. **Servlet Applications - Authentication - Authentication Architecture** 에서 살펴봤던 `AbstractAuthenticationProcessingFilter`의 인증과정과 상당히 유사하며, `AbstractAuthenticationProcessingFilter`이 `UsernamePasswordAuthenticationFilter`로 바뀌고 `Authentication` 객체가 `UsernamePasswordAuthenticationToken`으로 바뀌었다. 여기에서 `UsernamePasswordAuthenticationFilter`는 `AbstractAuthenticationProcessingFilter`을 상속한 클래스이다.
![image](https://docs.spring.io/spring-security/reference/_images/servlet/authentication/unpwd/usernamepasswordauthenticationfilter.png)

# [Servlet Applications - Authentication - Username/Password - Password Storage](https://docs.spring.io/spring-security/reference/servlet/authentication/passwords/user-details.html)
1. `UserDetails`는 `UserDetailsService`에 의해 리턴된다.
2. `DaoAuthenticationProvider`는 `UserDetails`이 유효한지 체크한 후 `Authentication`을 리턴한다.
3. `UserDetailsService`는 username, password, and other attributes를 받기 위해 `DaoAuthenticationProvider`에 의해 사용된다. Spring Security는 `UserDetailsService`를 구현한 in-memory와 JDBC의 두 가지 방식의 구현체를 제공한다.
4. ` DaoAuthenticationProvider`는 `AuthenticationProvider`의 구현체 중 하나로 `UserDetailsService`와`PasswordEncoder`를 사용해서 username과 password를 인증한다.
![image](https://docs.spring.io/spring-security/reference/_images/servlet/authentication/unpwd/daoauthenticationprovider.png)
   
# [Servlet Applications - Authorization - Authorization Architecture](https://docs.spring.io/spring-security/reference/servlet/authorization/architecture.html)
1. 