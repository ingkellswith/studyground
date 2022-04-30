# 스프링 시큐리티에서의 XSS(Cross Site Scripting)와 CSRF(Cross Site Request Forgery) 해결방안

# Reference

1. **Prevent Cross-Site Scripting (XSS) in a Spring Application** : [https://www.baeldung.com/spring-prevent-xss](https://www.baeldung.com/spring-prevent-xss)
2. **A Guide to CSRF Protection in Spring Security**: [https://www.baeldung.com/spring-security-csrf](https://www.baeldung.com/spring-security-csrf)

# XSS

- xss는 바로 실행되는 **reflected xss**가 있고, db에 저장된 후 실행되는 **stored xss**가 있다. 보통 xss를 말하면 **stored xss**를 말한다.
- **stored xss**의 예시로, 공격할 대상 페이지의 댓글에 스크립트를 써놓고, 댓글이 db에 저장되면 다른 사용자가 그 페이지를 로딩했을 때 스크립트가 실행되는 방식이다.
- 이는 스크립트를 필터링함으로써 해결가능하다.
- 스프링 시큐리티는 아래의 방법을 지원한다. **CSP(Content Security Policy)**는 xss와 data injection을 막아주는 스프링 시큐리티에서 만드는 하나의 레이어이다.

```xml
@Configuration
public class SecurityConf extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
          .headers()
          .xssProtection()
          .and()
          .contentSecurityPolicy("script-src 'self'");
    }
}
```

# CSRF(XSRF)

- 쿠키를 이용한 공격으로, 실제 사이트로부터 받은 쿠키를 브라우저에 저장한 후, 다른 사이트가 이 쿠키를 사용해 실제 사이트로 요청하는 방식이다. (쿠키는 요청에 자동으로 포함되는 특성을 이용) 쿠키를 사용하므로 쿠키를 사용하지 않는 jwt는 csrf를 막을 필요가 없다.
- 예시로, 실제 사이트와 비슷하게 위조한 사이트를 만들어서 피해자가 이 위조 페이지에 접속하면 클릭 혹은 페이지 로딩과 동시에 실제 사이트로 피해자가 위조된 요청을 보내는 방식이다.
- 아래는 스프링 시큐리티에서 제공하는 **서버사이드 렌더링**과 **클라이언트 사이드 렌더링** 방식의 **csrf**를 막는 방식이다.(spring security 4.x이상 버전에서는 csrf가 default로 활성화되어 있다.)
  - 첫번째로 **스프링과 통합된 서버 사이드 렌더링 기술**을 사용할 경우 렌더링된 html에 **csrf token**을 적용하는 방식이 있다.
    - ### form 전송 방식

        모든 form에 아래의 hidden input을 추가한다.
            
        ```xml
        <input type="hidden" name="${_csrf.parameterName}" value="${_csrf.token}"/>
        ```
            
        이는 모든 form에 위의 hidden input을 추가해야 한다는 단점이 존재한다.
        
        또한 multipart형식으로 전송해야 할 경우 요청 페이로드에 있는 _csrf필드를 스프링이 인식해야 하는데, 이 때 Multipartfilter를 사용한다. 부트가 아닌 스프링에서 Multipartfilter를 사용하기 위해서는 tomcat에서 MultipartParsing을 allow하는지 확인해야 한다.
        
        아래는 외장 tomcat의 MultipartParsing을 allow하는 context.xml설정이다.
        
        ```xml
        <Context allowCasualMultipartParsing="true">
            <WatchedResource>WEB-INF/web.xml</WatchedResource>
                <-- ... -->
        </Context>
        ```
            
        
    - ### json 전송 방식
        
        root html에 다음의 meta tag를 추가한다. (jquery를 사용했다.)
        
        ```xml
        <meta name="_csrf" content="${_csrf.token}"/>
        <meta name="_csrf_header" content="${_csrf.headerName}"/>
        ```
        
        root 자바스크립트에 모든 XHR request(XMLHttpRequest)에 선행하는 설정을 해준다.
        
        ```xml
        $(document).ajaxSend(function(e, xhr, options) {
            xhr.setRequestHeader(header, token);
        });
        ```
        
        root html에 위의 2번의 meta tag와 3번의 자바스크립트만 설정하면 json전송 방식은 form전송 방식과 달리 따로 csrf token을 설정할 필요가 없다.
        
  - 두번째는 **Stateless Spring API**를 사용할 때 클라이언트 측에서 **csrf token**을 사용하는 방법이다.  
    - ### 쿠키에서 csrf token 꺼내서 사용하기
        백엔드 애플리케이션에서는 html을 렌더링하지 않기 때문에 위에서 설명했던 방식을 사용할 수 없다. 따라서 쿠키에 CSRF Token을 담는다. CookieCsrfTokenRepository.withHttpOnlyFalse()을 매개변수로 넣으면 `cookieHttpOnly` 라는 `CookieCsrfTokenRepository` 의 멤버 변수가 false로 설정된다. httponlycookie는 클라이언트에서 스크립트를 사용해서 쿠키에 접근하는 것을 막기 위해 사용하는데, 클라이언트에서 쿠키에 들어 있는 csrf token을 사용해야 하므로 false로 설정하는 것이다.(이 헤더를 지원하는 브라우저에 한해서 HttpOnlyCookie가 true이면 스크립트를 사용해서 쿠키를 조작하는 것이 불가능하다.) ****HttpOnly cookie에 대한**** 자세한 정보 : [https://www.whitehatsec.com/glossary/content/httponly-session-cookie](https://www.whitehatsec.com/glossary/content/httponly-session-cookie)
        
        ```xml
        @Configuration
        public class SpringSecurityConfiguration extends WebSecurityConfigurerAdapter {
            @Override
            public void configure(HttpSecurity http) throws {
                http
                    .csrf()
                    .csrfTokenRepository(CookieCsrfTokenRepository.withHttpOnlyFalse());
            }
        }
        ```
            
        두 번째로 클라이언트 측에서 서버 측 데이터를 변경하는 모든 요청에 적용되는 설정을 해준다. 스프링에서는 헤더의 **X-XSRF-TOKEN**을 기본적으로 받는데, multipart요청으로 **_csrf**필드에 넣는 것을 받을 수도 있다.
            
        ```xml
        const csrfToken = document.cookie.replace(/(?:(?:^|.*;\s*)XSRF-TOKEN\s*\=\s*([^;]*).*$)|^.*$/, '$1');
        
        fetch(url, {
            method: 'POST',
            body: /* data to send */,
            headers: { 'X-XSRF-TOKEN': csrfToken },
        })
        ```