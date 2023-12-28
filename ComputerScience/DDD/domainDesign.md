DDD의 domain layer에 대한 구현
==============================
(수강 강의 - 패스트캠퍼스, The RED : 비즈니스 성공을 위한 Java/Spring 기반 서비스 개발과 MSA 구축 by 이희창)
 
이 글에서는 base.md에서 다뤘던 ddd의 기본 개념에서 더 나아가
domain layer에 대한 구현을 다룹니다.

1. domain layer에서의 service에서는 전체 도메인 로직의 흐름을 파악할 수 있도록 구현한다.
- 이를 위해서 추상화의 레벨을 많이 높여야 한다.
- 도메인은 interface를 사용하여 추상화하고 실제 구현은 다른 layer에 맡긴다.
  
> 이는 service만 읽어도 도메인의 로직 흐름을 파악할 수 있기 위함이다.

2. dip(dependency inversion principle)을 활용하여 도메인이 사용하는 interface의 실제 구현체를 주입받아 사용할 수 있도록 한다.
- 영속화된 객체를 로딩하기 위해 jpa를 사용할 수도 있지만 querydsl을 사용할 수도 있기 때문이다.
- domain layer에서는 객체를 로딩하기 위한 추상화된 interface를 사용하고, 실제 동작은 하위 layer의 기술 구현체에 맡긴다는 것이 핵심이다.
- interface로 추상화된 것을 구현하는 구현부는 언제든지 교체 가능

> 도메인 주도 설계 원칙이자 핵심 개념

### dip(dependency inversion principle)이란?  

![ddd-dip](https://user-images.githubusercontent.com/55550753/129925663-f0780dea-a15e-4c53-9445-9d0335313495.PNG)  

위 구조가 dip인데 처음 보면 이해하기 힘들 수 있으므로 예시를 들어보겠다.

![ddd-dip2](https://user-images.githubusercontent.com/55550753/129926539-29797ecf-68c0-40c2-92a6-09a25f7efd07.PNG)

위는 간단하게 이해할 수 있는, 자동차가 스노우 타이어에 의존하는 구조이다.

![ddd-dip3](https://user-images.githubusercontent.com/55550753/129926659-3857eebc-071d-4bce-a56c-22504392f26c.PNG)

위는 타이어를 다른 종류의 타이어로 교체할 수 있는 구조이다.  
정말 간단한 구조이지만 이것이 dip이다.  
어떤 것에도 의존하지 않던 스노우 타이어가, 타이어라는 인터페이스에 의존하게 되었으므로  
의존 관계의 역전이 일어난 것이다.  
용어로만 봤을 때는 이게 왜 역전인건지 이해하기 힘들었는데, 예시가 정말 직관적이라서 이 예시가 꼭 필요하다고 생각했다.  

여기에서 쓰이는 inversion 개념은 ioc(inversion of container)에서도 쓰이기 때문에 다시 짚고 넘어가겠다.   
간단하게, ioc란 di(dependency injection)을 통해서 의존성 주입을 개발자가 아니라 프레임워크가 하는 것을 말한다.  
개발자가 아니라 프레임워크가 의존성을 관리하기 때문에 제어의 권한의 역전이라고 말하는 것이다.  

