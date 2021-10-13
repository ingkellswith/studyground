Http Keep Alive
===
# 참조
[HTTP Keep Alive and TCP keep alive](https://stackoverflow.com/questions/9334401/http-keep-alive-and-tcp-keep-alive)  
[HTTP is statel-less,so what does it mean by keep-alive?](https://stackoverflow.com/questions/6060959/http-is-statel-less-so-what-does-it-mean-by-keep-alive)  
[[Stateful/Stateless] Stateful vs. Stateless 서비스와 HTTP 및 REST](https://5equal0.tistory.com/entry/StatefulStateless-Stateful-vs-Stateless-%EC%84%9C%EB%B9%84%EC%8A%A4%EC%99%80-HTTP-%EB%B0%8F-REST)  

# Stateful vs Stateless
OSI 7 Layer 4계층에서 사용하는 TCP를 베이스로 7계층에서 HTTP가 사용된다.  
그렇다면 TCP는 연결형 서비스인데 HTTP에서 사용하는 Stateless, Keep-Alive 개념은 무엇인가?  

Http는 stateless하다.  
stateless는 server의 응답이 client와의 세션 상태와 독립적인 것을 말한다.   

반대로 stateful은 server와 client간 세션의 '상태'에 기반하여 client에 response를 보낸다.  
이를 위해 세션 '상태'를 포함한 Client와의 세션 정보를 server에 저장하게 된다.  

그런데 이제 stateful한 구조를 많이 안 쓰는 이유는   
stateful에서는 세션 정보를 인스턴스 간에 공유하기 위해서 세션 공유를 해야하고, 그로 인해 확장성이 떨어지기 때문이다.    

반면, stateless한 구조는 세션을 db에 저장하기 때문에(성능이 좋으려면 redis같은 In-Memory DB사용) 인스턴스를 얼마든지 확장해도 상관이 없다.    

# Keep Alive
Http는 stateless하다.  
그런데 Keep Alive하다니 무언가 역설적인 것 같아 보인다.  
하지만 위에 설명한 stateful과 stateless를 잘 이해했다면 keep alive와 stateful과 stateless는 딱히 관련이 없다는 것을 알 수 있다.  
**Connection : Keep-Alive**는 통신 비용을 최소화하기 위해 http를 사용할 때 연결을 끊지 않겠다는 의미이다.  
따라서 서버에서 client의 상태를 판단해 통신하는 stateful, stateless와 keep-alive는 별개로 이해해야 한다.   









