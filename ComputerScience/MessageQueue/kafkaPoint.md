Kafka 사용 시 주의점
=== 
# Reference
강의 : [패스트캠퍼스] The RED : 비즈니스 성공을 위한 Java/Spring 기반 서비스 개발과 MSA 구축 by 이희창

# Rebalancing

### kafka와 통신하는 consumer group중 특정 인스턴스가 다운되었을 때, consumer group 에 consumer가 추가 또는 삭제되면 consumer가 소유하고있던 partition 을 재할당 (reassign) 하는 작업이 발생하는데 이를 rebalancing 이라고 한다

![sqs-rebalancing](https://user-images.githubusercontent.com/55550753/137632523-b8ff791a-18e1-4534-8192-9b779630ece6.PNG)

위 상황에서 Instance3가 다운되었을 때 이를 빠르게 rebalancing해야 한다.

![sqs-rebalancing2](https://user-images.githubusercontent.com/55550753/137632536-ff2e134d-5f6a-464b-a0f1-5e53e7396c1d.PNG)

kafka 1.x 에서는 rebalancing 이 발생하면 전체 consumer 가 메시지에 대한 polling 을 순간 중단한다(Stop the World).  
kafka 의 consumer 로 참여하는 애플리케이션의 배포가 있을 때마다 kafka 의 모든 consumer 가 일시적으로 동작하지 않는 상황이 발생할 수 있다.  

다행히, kafka 2.3 버전 이후로는 Incremental Cooperative Rebalancing 라는 디자인이 적용되어 이런 이슈가 해결된 상태이다.  
(Stop the World하지 않고 동작하는 인스턴스만 찾아 재할당)

[Design and Implementation of Incremental Cooperative Rebalancing](https://www.confluent.io/online-talks/design-and-implementation-of-incremental-cooperative-rebalancing-on-demand/) 참고

# 순서 보장

### topic은 순서 보장을 해주지 않지만, topic내의 partition에서는 순서 보장 기능을 지원해준다.

![kafka-rebalancing1](https://user-images.githubusercontent.com/55550753/137632365-6abd6772-7e0a-4425-a5a9-8ae49cc83c51.PNG)

1. 주문완료
2. 결제완료
3. 결제취소

위 세가지 순서가 지켜져야하는 로직에서 주문완료->결제완료->결제취소가 개별 partition에 분배되면 순서 보장을 할 수 없다.  
예를 들면 partition1에 1번인 주문완료가 할당되고, partition2에 2번인 결제완료가, partition3에 3번인 결제취소가 할당되었다면 어떤 메시지가 먼저 처리될 지 알 수 없다.  
그렇기에, 한 partition에 주문완료, 결제완료, 결제취소가 순서대로 할당되어야 한다.  

따라서 kafka에서는 메시지를 send할 때에 key값을 같이 보내, kafka에서 이 key를 hash함수를 사용해 특정 partition한 개에만 분배할 수 있도록 한다(파티셔닝).   
즉 순서가 지켜져야 하는 로직에 대해서는 같은 key값을 부여해 한 partition내에서만 처리할 수 있게 하는 것이다.  
위와 같은 경우 주문완료, 결제완료, 결제취소의 메시지를 보낼 때 한 개의 partition에서 처리할 수 있도록 (같은 주문에 대해서) key값은 모두 똑같이 보내면 된다.  

# 중복 메시지 수신

### kafka에서 consume 이후 offset commit 등을 못하는 등의 이슈가 발생하면, 이미 consumer에게 전달한 메시지를 중복으로 전달할 수도 있게 된다  

kafka에서는 consumer가 어디까지 메시지를 수신했는지를 offset으로 관리한다.  
따라서 다음 순서의 메시지는 몇 번째 인덱스를 보내야 하는지를 kafka가 알 수 있는 것이다.  

그런데 kafka에서 comsumer에게 메시지 전달 후 consumer가 메시지를 수신 후 잘 처리했지만, kafka와의 통신장애가 발생해서, kafka에게 알려줘야하는 offset commit을 하지 못할 경우 중복 메시지 수신이 발생할 가능성이 있다.  
즉, consumer입장에서는 처리했던 메시지를 offset commit하지 못함으로써 kafka와의 offset차이로 인해 수신했던 메시지를 중복으로 더 수신하는 것이다.  

두 가지 해결방법이 있다.  
1. throw exception으로 트랜잭션 롤백
2. 성공(http status 200) 응답
3. 로직에 멱등성(연산을 여러 번 적용하더라도 결과가 달라지지 않는 성질을 의미) 구현

1번 방법은 별도의 테이블을 두거나 새 컬럼을 추가하면 해결가능하다.  
하나의 트랜잭션에 unique index 가 걸려 있는 msg_id 를 insert 하는 로직과 (ex: PROCESSED_MESSAGE 테이블) 실제 비즈니스를 처리하는 CUD 로직을 묶으면 된다.  
unique이기 때문에 중복 insert가 되지 않아 exception이 발생해 롤백이 가능하게 된다.  

2번 방법은 중복 메시지는 어찌 되었든 이전에 한 번 처리되었던 로직이니, exception이 불필요하다고 생각될 때 사용할 수 있겠다.  

또는 중복 메시지를 처리할 때 메시지를 처리하는 consumer 로직을 멱등하게 구현하면 중복 메시지가 전달되더라도 이슈가 발생하지 않는다.  





