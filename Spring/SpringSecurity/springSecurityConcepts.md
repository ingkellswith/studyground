스프링 시큐리티 주요 개념 정리
===
# [스프링 시큐리티 공식 토픽](https://spring.io/guides/topicals/spring-security-architecture)
1. 인증을 처리하는 **AuthenticationManager**는 authenticate라는 하나의 메소드만을 가지고 있다. 예를 들어, formLogin 진행 시 아이디와 비밀번호를 이 메소드를 통해 처리한다.
2. **authenticate**함수에 **Authentication객체**를 전달하고 **Authentication객체**를 반환하는데 인증되기 전에는 **Authentication**의 필드인 **isAuthenticated**가 false인 상태이고, 함수 실행 후에 인증이 완료되면 **isAuthenticated**는 true가 된다.
3. **AuthenticationManager** 인터페이스를 구현한 구현체는 일반적으로 **ProviderManager**이다. **AuthenticationManager**와 기능이 유사하지만 다양한 **Authentication**타입을 지원하는 **supports**메소드가 추가되어 있다.**(ProviderManager -> AuthenticationProvider -> AuthenticationManager)**
4. **ProviderManager**는 선택적으로 parent를 가질 수 있는데 이 parent는 모든 supports메소드에서 타입 지원을 체크했음에도 실패할 시 **fallback**처럼 동작한다.
5. 인가의 권한을 판단하기 위해서 **AccessDecisionVoter**를 사용하는데, vote메소드에서 **authentication**객체와 접근 대상인 **Object**, 접근 권한인 **ConfigAttribute** 파라미터로 넘긴다. 이후 메소드가 실행되면 **vote여부**(권한 인가 성공 여부)를 int로 반환한다.
6. default **AccessDecisionManager**는 한 개의 voter라도 affirm하면 해당 자원에 액세스를 허용한다.

# [Servlet Applications - Architecture](https://docs.spring.io/spring-security/reference/servlet/architecture.html)
1. **DelegatingFilterProxy**는 서블릿 컨테이너의 lifecycle과 Spring’s ApplicationContext를 잇는 역할을 한다. 서블릿 필터는 스프링 컨테이너의 bean들을 인식할 수 없으므로 bean인 **FilterChainProxy**에 모든 작업을 위임한다.
2. 서블릿 컨테이너의 **DelegatingFilterProxy**에 의해 역할을 위임받는 **FilterChainProxy**는 일반적으로 bean이다.
3. **FilterChainProxy**는 스프링 시큐리티의 서블릿 지원의 시작점이다. 따라서 디버그하기 좋은 위치이다.
4. **FilterChainProxy**는 스프링 시큐리티의 핵심이다. 따라서 옵션으로 적용되지 않는 필수적인 기능을 실행하기에 좋은 위치이다. 예를 들어, 메모리 누수를 막기 위해 **ThreadLocal**로 사용된 SecurityContext를 clear하는 것이 있다.
5. 서블릿 컨테이너에서 **서블릿 Filter**는 오직 **url**에 근거해서 발동되지만, **FilterChainProxy**는 **HttpServletRequest**내부의 필드들을 사용할 수 있어 세밀한 매칭이 가능하다. 따라서 **FilterChainProxy**는 어떤 **Security Filter Chain**이 발동되어야 하는지를 결정할 수 있다.

![image](https://user-images.githubusercontent.com/55550753/162989585-7fc980ca-bc14-4443-b07f-59fd92b2780a.png)

# [Servlet Applications - Authentication - Authentication Architecture](https://docs.spring.io/spring-security/reference/servlet/authentication/architecture.html)
1. **SecurityContextHolder > SecurityContext > Authentication > principal** 순으로 객체의 필드가 정해지는 구조에서 **SecurityContextHolder**는 **ThreadLocal**을 사용해서 **SecurityContext**에 값을 할당한다. 따라서 같은 스레드의 메소드라면 굳이 파라미터로 넘길 필요 없이 항상 available하다.
2. 스레드풀을 사용하는 구조에서 보안을 이유로 하거나 정확한 처리를 이유로 해서 ThreadLocal인 SecurityContext를 사용 후에는 항상 thread를 clear해야 하는데, 이를 **FilterChainProxy**에서 처리해준다.  