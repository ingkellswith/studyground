스프링 시큐리티를 사용한 카카오 네이버 로그인
===
스프링부트가 아닌 스프링에서 스프링 시큐리티와 OAuth2를 기반으로 한 SNS로그인을 구현하면서 얻게 된 지식을 적습니다. Spring Security로 구현하는 OAuth2는 대부분의 한글 자료가 스프링 부트 기반이었기에, 스프링 기반으로 구현하기 위해서 클래스 상속 구조와 여러 객체의 메소드를 보는 것이 코드 작성에 많은 도움이 되었습니다.

# OAuth 2.0와 주요 3단계 프로세스
공식문서 : [https://oauth.net/2/](https://oauth.net/2/)

OAuth 2.0는 OAuth 2.0은 인증을 위한 업계 표준 프로토콜으로 커스텀 애플리케이션에서 다른 리소스 서버(ex. 카카오, 네이버)에 인증된 사용자 정보(ex. 이메일, 생년월일 등의 제한적 정보)를 가져올 수 있다. **가져온 정보로 인증, 인가 등의 로직을 구현하는 것은 사용자의 몫이다.** 보통 인가 코드 발급, 토큰 발급, 사용자 정보 가져오기의 3단계의 프로세스로 나눌 수 있다. 아래는 설명이 너무 깔끔하게 되어 있어서 카카오 로그인 공식문서에서 가져와 봤다.

![kakao login process](https://developers.kakao.com/docs/latest/ko/assets/style/images/kakaologin/kakaologin_sequence.png)

**#1.** 인가 코드 발급 : 사용자가 리소스 서버에 인증된 사용자인지 검증하는 과정이다. 예를 들어 카카오를 리소스 서버로 사용할 것이라면 사용자가 카카오에 인증된 사용자임을 확인해야 한다. 위 그림에서도 볼 수 있듯이 이는 보통 자바스크립트로 클라이언트 측에서 리소스 서버로 요청을 보내는 형태로 이루어진다. 리소스 서버 측에서는 클라이언트 측의 요청을 받으면 로그인 화면을 내려주고, 동의 항목까지 응답받으면 이 정보를 사용할 서버로 **인가 코드**와 함께 **redirect**하게 된다.
spring security에서는 `"{baseUrl}/{action}/oauth2/code/{registrationId}"`이 default설정의 redirect url이다. 예를 들어, example.com에서 사용할 것이고, 로그인 액션에 사용할 것이며, 리소소 서버로는 카카오를 사용할 것이라면 `"https://example.com/login/oauth2/code/kakao"`가 redirect url이 되겠다. 당연히 이 redirect url은 바꿀 수 있다. 다만 스프링에서 바꾸었다면, 리소스 서버에도 그에 해당하는 redirect url설정을 필수적으로 해야 한다.

**#2.** 토큰 발급 : 인가 코드를 받았다면, 인가 코드를 요청에 담아 토큰 발급 url로 보낸다. 그 후 리소스 서버는 받은 요청이 유효한 클라이언트가 보냈는지 확인하고 토큰을 발급해준다. 이 토큰은 사용하는 입장에서는 이 토큰이 무엇을 의미하는지 몰라도 된다. 사용자는 그저 이 토큰이 마치 호텔키처럼 유효하게 동작할 것이라는 점만 알면 된다. 물론 당연히 리소스 서버에서는 이 토큰을 해석하여 어떤 유저의 정보에 접근할 수 있는지 판단해야 한다. 호텔 락도어가 호텔키를 해석하여 방번호와 호텔키가 상응하는지 확인하는 것처럼 말이다.

access token를 호텔 키카드에 빗대어 간단히 설명한 유튜브 : [https://www.youtube.com/watch?v=BNEoKexlmA4&ab_channel=OktaDev](https://www.youtube.com/watch?v=BNEoKexlmA4&ab_channel=OktaDev)

**#3.** 사용자 정보 가져오기 : 토큰을 받았다면, 토큰을 Authorization헤더에 담아 사용자 정보 조회 url로 요청한다. 

# spring(without spring boot) + spring security 조합의 카카오 로그인 구현

OAuth 2.0은 REST API를 사용하는 모든 백엔드 애플리케이션에서 구현이 가능하다. 이 말인 즉슨 스프링 프레임워크가 없어도 구현이 가능하다는 말이다. 이 말을 하는 이유는 스프링 시큐리티를 사용하고 있지만, 현재 사용하고 있는 스프링의 버전이 너무 낮아 스프링 시큐리티의 OAuth2를 지원하지 않는다면, 퓨어하게 REST API를 사용해서 OAuth2를 구현한 후 이를 스프링 시큐리티와 통합하는 로직을 구현 해야 하기 때문이다. (**메이븐 리포지토리 기준으로 스프링 시큐리티 5.x이상 버전부터 oauth2 client를 지원한다.**) 하지만 스프링에서는 스프링 시큐리티를 사용해서 보안 관련 로직을 구현하는 것을 권장하고, 인증, 인가, 시큐리티 필터 같은 기능들을 간편하게 사용할 수 있으니, 되도록이면 버전업을 해서 스프링 시큐리티를 사용해서 OAuth2를 구현하는 것이 옳겠다. 
    
**#1.** **리소스 서버(ex. 카카오)에 사용자(애플리케이션) 등록하기** 
    
OAuth2.0을 사용하기 위해서 사용하고 싶은 서비스에 로그인해서 어떤 애플리케이션에서 이 서비스에 등록된 사용자의 정보를 가져오고 싶다고 알려줘야 한다. 예를 들어 카카오를 리소스 서버로 사용할 수 있다. 사용법은 카카오 공식 문서에 자세히 나와 있다.

카카오 로그인 공식문서 : [https://developers.kakao.com/docs/latest/ko/kakaologin/common](https://developers.kakao.com/docs/latest/ko/kakaologin/common)
  
**#2.** **사용자(애플리케이션)에 등록정보 설정하기**
    
clientregistration을 진행하는데 구글, 페이스북, 깃허브를 리소스 서버로 사용할 경우 `CommonOAuth2Provider` 을 사용하면 간편하게 설정할 수 있다. 하지만 카카오, 네이버같은 경우 직접 OAuth2Provider를 만들어줘야 한다. 아래 코드를 참고해서 설정하자.
    
```text
/** @Configuration **/

@Bean
public ClientRegistrationRepository clientRegistrationRepository() {
    // 다른 client registration이 필요할 경우 아래 생성자의 매개변수에 카카오를 등록한 것처럼 추가해주면 된다.
    return new InMemoryClientRegistrationRepository(this.kakaoClientRegistration());
}

@Bean
public OAuth2AuthorizedClientService authorizedClientService(
        ClientRegistrationRepository clientRegistrationRepository) {
    return new InMemoryOAuth2AuthorizedClientService(clientRegistrationRepository);
}

@Bean
public OAuth2AuthorizedClientRepository authorizedClientRepository(
        OAuth2AuthorizedClientService authorizedClientService) {
    return new AuthenticatedPrincipalOAuth2AuthorizedClientRepository(authorizedClientService);
}

private ClientRegistration kakaoClientRegistration() {
    /** 리소스 서버로 카카오를 사용한다.**/
    Builder builder = ClientRegistration.withRegistrationId("kakao");
    /** grant_type으로는 인가 코드 받기, 토큰 갱신 등이 있는데 현재는 인가 코드를 받을 때 사용할 것이므로 아래처럼 사용한다.**/
    builder.authorizationGrantType(AuthorizationGrantType.AUTHORIZATION_CODE);
    /** 리소스 서버로부터 받을 정보 범위이고, 리소스 서버에 앱을 등록할 때 적었던 정보
    범위와 일치해야 한다.**/
    builder.scope(new String[]{"profile_nickname", "account_email", "birthday"});
    /** 클라이언트 인증 메소드는 CLIENT_SECRET_POST를 사용할 것이다. 꼭 적어야 한다. **/
    builder.clientAuthenticationMethod(ClientAuthenticationMethod.CLIENT_SECRET_POST);
    /** 아래 4개의 URI를 적으면 자동으로 인가 코드 요청, 리다이렉트 url 설정, 토큰 요청,
    유저 정보 조회 요청을 다 구현해준다. **/
    builder.authorizationUri("https://kauth.kakao.com/oauth/authorize");
    builder.redirectUri("{baseUrl}/{action}/oauth2/code/{registrationId}");
    builder.tokenUri("https://kauth.kakao.com/oauth/token");
    builder.userInfoUri("https://kapi.kakao.com/v2/user/me");
    /** UserInfo Response에 있는 id를 userNameAttributeName으로 사용하겠다는 의미이다.
    사용자 구별 용도로 사용한다. 존재하지 않는 값을 적으면 에러가 발생한다. 리소스 서버
    별로 userNameAttributeName이 다를 수 있으니 체크해야 한다. **/
    builder.userNameAttributeName("id");
    builder.clientName("Kakao");
    builder.clientId("앱 등록할 때 받은 clientId");
    builder.clientSecret("앱 등록할 때 받은 clientSecret");
    return builder.build();
}
```
    
spring security without boot 공식 문서 참고 코드 : [https://docs.spring.io/spring-security/reference/servlet/oauth2/login/core.html#oauth2login-javaconfig-wo-boot](https://docs.spring.io/spring-security/reference/servlet/oauth2/login/core.html#oauth2login-javaconfig-wo-boot)
  
**#3.** **OAuth2 로그인 등록하기**
    
```text
@RequiredArgsConstructor
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {

        private final CustomOAuth2UserService customOAuth2UserService;

        @Override
        protected void configure(HttpSecurity http){
                http.oauth2Login()
                .userInfoEndpoint()
                .userService(customOAuth2UserService);
        }
}
```

```xml
@RequiredArgsConstructor
@Service
public class CustomOAuth2UserService implements OAuth2UserService<OAuth2UserRequest, OAuth2User> {

    private final UserService userService;

    // 리소스 서버로부터 유저 정보를 받아온 후 실행되는 메소드
    @Override
    public OAuth2User loadUser(OAuth2UserRequest oAuth2UserRequest) throws OAuth2AuthenticationException {
        OAuth2UserService oAuth2UserService = new DefaultOAuth2UserService();
        OAuth2User oAuth2User = oAuth2UserService.loadUser(oAuth2UserRequest);

        /** 권한 부여 **/
        Set<GrantedAuthority> authorities = new LinkedHashSet();

        /** 유저 정보를 가져오는 로직, 토큰 제공자별로 응답 json의 구조가 다르다.**/
        Map<String, Object> kakaoAccount = (Map<String, Object>) oAuth2User.getAttributes().get("kakao_account");
        String email = (String) kakaoAccount.get("email");

        /** 이 부분에 findUserByEmail같은 메서드로 유저를 찾고 있으면 update
        없으면 insert하는 로직을 넣으면 된다.**/
        
        /** 아래에서 리턴하는 OAuth2User는 principal이 되기 때문에 기존 시큐리티 사용 로직과 통합
        하기 어려울 수 있는데, OAuth2User를 상속하고 CustomOAuth2User를 구현하면 원하는 
        로직을 만들 수 있습니다.**/

        return new DefaultOAuth2User(
                authorities,
                oAuth2User.getAttributes(),
                "id");
    }
}
```
