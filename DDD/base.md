DDD(Domain Driven Design)
===========================
(수강 강의 - 패스트캠퍼스, The RED : 비즈니스 성공을 위한 Java/Spring 기반 서비스 개발과 MSA 구축 by 이희창)

스프링 부트를 실무로 진행하고 또 공부해나가면서 궁금한 것이 정말 많았다.  
그 중에서도 스프링 부트 코드의 전반적인 구조를 담당하는 DDD에 대해 먼저 기록하려 한다.  

먼저 도메인 주도 설계, DDD는 아래 그림으로 간략하게 소개할 수 있다.   

![DDD-layer2](https://user-images.githubusercontent.com/55550753/129905407-8aba8cab-a6ca-4d8b-b9dc-54ff752919b2.PNG)  

구글에는 DDD에 대해 복잡하게 설명한 다이어그램이 많은데 위 다이어그램은 정말 깔끔하다...  
DDD는 interfaces, application, domain, infrastructure 이렇게 크게 4가지 계층으로 구성되어 있다.  

interface는 controller, dto가 존재할 수 있고 infrastructure에는 db레벨의 구현체가 존재한다.   
db레벨의 구현체라 함은 대표적으로 spring data jpa가 있겠다.   
interface를 선언하면 jpa에서 알아서 구현체를 만들어주는 것이기 때문이다.  

그렇다면 application과 domain은 어떻게 이해해야 할까?  
사실 그냥 그런게 있구나 하고 넘어가도 되는데 그렇지 못하는 이유가 내가 이전에 알고 있던  
스프링 웹 계층 때문이다. 스프링 웹 계층과 DDD는 어떻게 비교할 지 분석이 필요하다.  

![spring-web-layer](https://user-images.githubusercontent.com/55550753/129907942-d02b8ecb-ec17-4972-820a-d8f3196a4a28.png)  

먼저 스프링 웹 계층의 web layer와 repository layer는 DDD 4계층의 interface와 infrastructure와 상응한다.  
다음으로, dto는 DDD의 interface, 웹 계층의 web layer에 존재하지만, dto를 다른 계층에도 전달하므로 웹 계층의 우측 상단에 위치한 것이 이해된다.  
남은 스프링 웹 계층의 service layer, domain model는 어떻게 매칭 시켜야 DDD의 application, domain layer에 매칭될 수 있을까?  

**나의 결론은 웹 계층의 service layer와 domain model, DDD의 application과 domain layer가 공통으로 가져가는 핵심 개념이 '추상화'라고 이해하는 것이다.**   

추상화라 함은 추상화를 시켜서 domain에 의존하는 것들의 코드를 바꿔도 domain의 로직은 유지될 수 있게 하는 것이다.  
예시로 트래픽이 많아져 db를 mariadb에서 mongodb로 바꿔야 할 상황이 있을 수도 있는데   
그 상황에서 spring data jpa에서 spring data mongodb로 의존성만 교체해주면 되는 것이 있을 수 있다.  
이렇게 로직의 추상화가 된다면 db를 바꾸기 위해서 도메인의 로직을 바꾸지 않아도 되는 것이다.  
그에 따라 확장성과 유지보수가 쉬워지는 것은 덤이고 말이다.  

이것이 가능하려면 코드를 짤 때 추상화를 계속 염두해두고 있어야 하므로 코드를 잘 짜는 연습이 지속적으로 필요하겠다...  
코드를 짜면서 service부분에 if로직을 무작정 넣는 것이 아니라 if로직을 도메인으로 옮기면 추상화가 더 잘 이루어지지 않을까하는 물음같은 것 말이다.    

계속 추상화를 얘기하는데 '추상화'라는 말이 조금 말 뜻처럼 추상적일 수 있어서 적어놓는데 추상화란 말 그대로 추상화이다(?!).  
내가 이해한 추상화는 쉽게 얘기헤서 사과를 넣으면 주스로 만들어주던 평범한 주스 기계가 있다고 해보면,    
주스 기계를 개조해서(개조를 추상화라고 봅시다... ㅎㅎ;)  
바나나를 넣어도 주스를 주고 파인애플을 넣어도 주스를 주는 것이다.  
추상화의 레벨을 더 높이면 과일과 얼음을 같이 넣을 수 있을 것이고  
더 높이면 과일과 얼음과 시럽을 같이 넣을 수 있을 것이다.   

비유가 어떨지 모르겠지만 이렇게 이해해보면 추상화란 말 그대로 추상화일 뿐 그리 어려운 개념이 아니다.  

각설하고 스프링 웹 계층과 DDD를 통합해 정리한 핵심 개념은   
'domain로직를 추상화시키고, 이 도메인을 다른 계층에서는 로직을 거의 배제하고 규칙에 따라 사용하기만 하는 것'이다.  

코드를 짜는 실무적으로는 연습이 많이 필요하겠지만, 이론적으로는 스프링 웹 계층과 DDD의 큰 틀을 전반적으로 알게 되었기 때문에 유익한 시간이 된 것 같다.  





