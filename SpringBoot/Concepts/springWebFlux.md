Spring WebFlux
===
# 참조
[Spring WebClient 쉽게 이해하기](https://happycloud-lee.tistory.com/220?category=902419)   
[Spring WebFlux는 어떻게 적은 리소스로 많은 트래픽을 감당할까?](https://alwayspr.tistory.com/44)   
[동기와 비동기](https://musma.github.io/2019/04/17/blocking-and-synchronous.html)  

# Spring WebFlux
**Spring MVC**가 멀티 스레드 + blocking + 동기 방식이라면,  
**Spring WebFlux**는 1코어당 1스레드 + non-blocking + 비동기 방식이다.  
1코어당 1스레드 + non-blocking + 비동기 방식은 **Node.js**에서도 사용하는 방식이다.  

Spring WebFlux는 이벤트 드라이븐 구조로  
Event Loop를 통해서 작업이 처리된다.  
Event Loop이 입력을 위해 루프롤 돌고 있고, 이벤트 발생 시 비동기적으로 로직을 처리한다.  

![event-loop](https://user-images.githubusercontent.com/55550753/136983329-19e874e7-8f52-4c7e-bd8b-4c879d9bf096.PNG)
