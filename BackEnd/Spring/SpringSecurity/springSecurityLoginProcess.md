스프링 시큐리티 로그인 프로세스
===
2. **로그인 프로세스**
3. `@FunctionalInterface`인 `OAuth2UserService` 의 loaduser 메소드를 `CustomOAuth2UserService` 에서 구현하면 `OAuth2LoginAuthenticationProvider` 에서 이 loaduser를 호출한다.  아래는 `OAuth2LoginAuthenticationProvider` 의 authenticate 메소드의 로직 중 일부이다.

```xml
public Authentication authenticate(Authentication authentication) throws AuthenticationException {

OAuth2User oauth2User = this.userService.loadUser(new OAuth2UserRequest(loginAuthenticationToken.getClientRegistration(), accessToken, additionalParameters));

return authenticationResult;

}
```
    
    b.  loadUser에서 리턴하는 OAuth2User는 principal이 될 것이기에 중요하다. principal을 현재 유저 테이블에 맞게 구현하고 싶다면 OAuth2User 인터페이스만 상속해서 현재 테이블에 맞게 구현하면 된다.
    
    c.  `OAuth2LoginAuthenticationProvider` 의 authenticate 메소드는 `ProviderManager`의 authenticate메소드에서 호출한다. 
    
    d.  `ProviderManager`의 authenticate 메소드는 `OAuth2LoginAuthenticationFilter` 에서 호출한다.
    
    ```xml
    public Authentication attemptAuthentication(HttpServletRequest request, HttpServletResponse response) throws AuthenticationException {
    	OAuth2LoginAuthenticationToken authenticationResult = (OAuth2LoginAuthenticationToken)this.getAuthenticationManager().authenticate(authenticationRequest);
    }
    ```
    
    리턴된  `Authentication` 객체를 `OAuth2LoginAuthenticationToken` 로 다운캐스팅한다.
    
    e.  위에서`OAuth2LoginAuthenticationFilter` 의 `attemptAuthentication` 메소드에서 `AuthenticationManager`를 가져와서 `authenticate`메소드를 호출했다. 이 `attemptAuthentication` 는 `AbstractAuthenticationProcessingFilter` 의 `doFilter` 메소드에서 호출한다.
    
    ```xml
    private void doFilter(HttpServletRequest request, HttpServletResponse response, FilterChain chain) throws IOException, ServletException {
    
      Authentication authenticationResult = this.attemptAuthentication(request, response);
    
    }
    ```
    
    f. 이 `AbstractAuthenticationProcessingFilter` 는 `FilterChainProxy` 에서 호출된다.
    
    g. 그 다음에 `FilterChainProxy` 는 다음 필터인 `OAuth2AuthorizationRequestRedirectFilter`를 호출한다.
    
    h. 그 다음은 `OncePerRequestFilter` 이다.
    
    j.  이제 별의별 필터를 다 거친다.
    
- `delegatingfilterproxy`에서 `filterchainproxy`를 호출하고, filterchainproxy는 20개 안팎의 필터를 생성자 파라미터로 받고 `VirtualFilterChain` 을 생성해서 `nextFilter.doFilter(request, response, VirtualFilterChain);` 처럼 `VirtualFilterChain` 자체를 파라미터로 넘겨서 다음에 어떤 필터를 호출해야 하는지 20개 안팎의 필터가 알아서 다음 필터를 호출하게 한다. 여기에서 `nextFilter` 는  `filterchainproxy` 가 가진 20개 안팎의 순서를 가진 필터가 된다. 예를 들어, `AbstractAuthenticationProcessingFilter` 의 필터 동작이 끝나면 전달 받은 `FilterChain chain` 으로 `chain.doFilter`을 호출한다. 여기서 `FilterChain`은 인터페이스이고, 구현체로 `VirtualFilterChain` 을 사용한다. `chain.doFilter` 가 호출되면 `VirtualFilterChain` 의 doFilter가 동작하게 된다.
- 모든 호출이 끝나면 `filterchainproxy` 는 `SecurityContextHolder`에서 `cleatContext()`로 스레드의 세션을 지운다.
- `AbstractAuthenticationProcessingFilter`는 인증이 성공적으로 완료되면, `successfulAuthentication` 메소드를 호출해 `SecurityContextHolder.setContext(context);` 로 세션을 설정한다.