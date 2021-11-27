JVM(Java Virtual Machine)의 Garbage Collector
======================
[던의 JVM의 Garbage Collector](https://www.youtube.com/watch?v=vZRmCbl871I)을 참고했습니다. 

# JRE(Java Runtime Environment)
JRE는 JVM 이 자바 프로그램을 동작시킬 때 필요한 라이브러리 파일들과 기타 파일들을 가지고 있다.   
JRE는 JVM의 실행환경을 구현했다고 할 수 있다.  
그러나 새 프로그램을 만드는 데에는 사용할 수 없다.  
**구성요소**
- 자바 API  
- JVM  

# JDK(Java Development Kit)
JDK는 자바의 모든 기능을 갖춘 SDK이다.  
JDK는 JRE + 개발을 위해 필요한 도구(javac, java등)들을 포함한다.  
따라서 프로그래밍 + 실행이 가능하다.  

때로는 컴퓨터에서 Java 개발을 수행할 계획이 없더라도 JDK가 필요한 상황이 존재한다.  
예를 들어, JSP로 웹 애플리케이션을 배포할 때 애플리케이션 서버 내에서 Java 프로그램을 실행하는 경우가 있다.  
이 상황에서는 애플리케이션 서버가 JSP를 Java 서블릿으로 변환하고, JDK를 사용하여 서블릿을 컴파일해야하므로 JDK가 필요하다.  

자바 서블릿(Servlet)은 자바를 사용하여 웹페이지를 동적으로 생성하는 서버측 프로그램을 말한다.
예로, 스프링부트에서는 DispatcherServlet을 사용해 MVC를 관리한다.

# JVM(Java Virtual Machine)
자바 바이트코드를 해석하고 실행하는 프로그램  
- 메모리 관리, Garbage Collector 수행
- 자바 애플리케이션을 클래스 로더(Class Loader)를 통해 읽어 들여서 자바 API와 함께 실행하는 것

# Garbage Collector란
자바 런타임 실행중 동적으로 할당한 메모리 영역 중 사용하지 않는 영역을 탐지하여 해제하는 기능

# Stack, Heap
### Stack
- 정적으로 할당한 메모리 영역
- 원시 타입의 데이터가 값과 함께 할당, Heap영역에 생성된 Object타입의 데이터의 참조 값 할당
### Heap
- 동적으로 할당한 메모리 영역
- 모든 Object 타입의 데이터가 할당, Heap영역의 Object를 가리키는 참조 변수가 Stack에 할당

(참고로 코틀린에서는 프로그래밍 단계에서 원시 타입, 래퍼 타입을 구분하지 않는다. 컴파일 시에 코틀린이 원시 타입 또는 래퍼 타입으로 자동변환한다.)

# Garbage Collector의 처리 과정
1. Mark : Garbage Collector가 Stack의 모든 변수를 스캔하면서 각각 어떤 객체를 참조하고 있는지 찾아서 마킹한다.  
2. Mark : Reachable Object가 참조하고 있는 객체도 찾아서 마킹한다.  
3. Sweep : 마킹되지 않은 객체를 Heap에서 제거한다.  

Garbage Collector의 work는 Mark와 Sweep의 반복과정이라고 할 수 있다. 

### 예시
category는 List(Reachable Object)를 참조하고 있고 List(Reachable Object)는 String(Reachable Object)을 참조하고 있다.  

![gc1](https://user-images.githubusercontent.com/55550753/136663149-27b6db46-4131-4e6d-ab84-e7cc369d4f13.PNG)  

이 과정에서 마킹되지 않은 String("요리") : Unreachable Object 는 Sweep된다.  

![gc2](https://user-images.githubusercontent.com/55550753/136663179-6aff2e73-4a96-48a8-8113-fd38a58090d7.PNG)  
   
# Stop The World
Garbage Collector를 실행하기 위해서 jvm이 애플리케이션 실행을 멈추는 것이다.  
Stop The World가 발생하면 Garbage Collector를 실행하는 스레드를 제외한 나머지 스레드는 모두 작업을 멈춘다.  
Garbage Collector 작업을 완료한 이후에 중단한 작업을 다시 시작한다.  

'실행하는 스레드를 제외한 나머지 스레드는 모두 작업을 멈춘다'는 개념은 꼭 Garbage Collector의 스레드에만 국한되는 것이 아니라 다른 곳에도 사용될 수 있다.  
ex) 메시지 브로커
: 메시지 브로커 토픽 내의 파티션을 각 인스턴스가 consume하는 경우  

![sqs-rebalancing](https://user-images.githubusercontent.com/55550753/136662469-e217813e-44ca-48fe-8e0c-c2dd81d3efad.PNG)  

한 인스턴스에 이상이 발생할 경우 rebalancing을 진행하는데 모든 인스턴스에 대한 파티션 소유권을 회수하고   
진행하는 경우 stop the world에 가깝다고 볼 수 있다.   

![sqs-rebalancing2](https://user-images.githubusercontent.com/55550753/136662690-bb9c637b-cc21-4e28-a336-13e4aa067bd3.PNG)  

kafka 2.3 이전 버전에서는 이런 전체적인 소유권 회수가 이슈가 되었다고 한다.  
이 경우 모든 인스턴스가 작동하지 않는 시간이 존재해 서비스에 장애가 발생하기 때문이다.  
