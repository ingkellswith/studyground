Spring WebFlux
===
# 참조
[Spring WebClient 쉽게 이해하기](https://happycloud-lee.tistory.com/220?category=902419)   
[Spring WebFlux는 어떻게 적은 리소스로 많은 트래픽을 감당할까?](https://alwayspr.tistory.com/44)   
[동기와 비동기](https://musma.github.io/2019/04/17/blocking-and-synchronous.html)  
[PM2를 활용한 Node.js 무중단 서비스하기](https://engineering.linecorp.com/ko/blog/pm2-nodejs/)  
[코틀린(kotlin) + 스프링부트(springboot) + 웹플럭스(webflux) + 코루틴(coroutine) - 웹플럭스에서 코루틴 사용해보기](https://appleg1226.tistory.com/16)  

# Spring WebFlux
**Spring MVC**가 멀티 스레드 + blocking + 동기 방식이라면,  
**Spring WebFlux**는 1코어당 1스레드 + non-blocking + 비동기 방식이다.  
1코어당 1스레드 + non-blocking + 비동기 방식은 **Node.js**에서도 사용하는 방식이다.  
(참고 : [PM2를 활용한 Node.js 무중단 서비스하기](https://engineering.linecorp.com/ko/blog/pm2-nodejs/)를 읽어보면   
싱글스레드로 동작하는 nodejs의 단점을 어떻게 보완하는지 살펴볼 수 있다.)   

Spring WebFlux는 이벤트 드라이븐 구조로  
Event Loop를 통해서 작업이 처리된다.  
Event Loop이 입력을 위해 루프롤 돌고 있고, 이벤트 발생 시 비동기적으로 로직을 처리한다.  

![event-loop](https://user-images.githubusercontent.com/55550753/136983329-19e874e7-8f52-4c7e-bd8b-4c879d9bf096.PNG)  
