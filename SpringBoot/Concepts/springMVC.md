Spring MVC
======================
# Reference
[[10분 테코톡]타미의 Servlet vs Spring](https://www.youtube.com/watch?v=2pBsXI01J6M)  
[[10분 테코톡]크로플의 싱글턴과 정적클래스](https://www.youtube.com/watch?v=C6CczyrkYXU)  
[스프링 빈은 thread-safe할까?](https://alwayspr.tistory.com/11)  
[Spring MVC Request 객체는 매번 새로 생성 됨](https://fntg.tistory.com/197)  
[싱글톤 패턴(Singleton pattern)을 쓰는 이유와 문제점](https://jeong-pro.tistory.com/86)  
[디스패처 서블릿이란?](https://mangkyu.tistory.com/18)  
[서블릿 생명주기](https://kadosholy.tistory.com/47)  
[운영체제 - 프로세스, 스레드](https://rebas.kr/849)  
[스택, 힙, 코드, 데이터영역](https://selfish-developer.com/entry/%EC%8A%A4%ED%83%9D-%ED%9E%99-%EC%BD%94%EB%93%9C-%EB%8D%B0%EC%9D%B4%ED%84%B0%EC%98%81%EC%97%AD)  
[자바 메모리관리 - 스택 & 힙](https://yaboong.github.io/java/2018/05/26/java-memory-management/)  
[멀티 스레드 환경과 스프링 빈](https://doflamingo.tistory.com/44)  
[멀티 스레드 환경에서 스프링 빈 주의사항](https://beyondj2ee.wordpress.com/2013/02/28/%EB%A9%80%ED%8B%B0-%EC%93%B0%EB%A0%88%EB%93%9C-%ED%99%98%EA%B2%BD%EC%97%90%EC%84%9C-%EC%8A%A4%ED%94%84%EB%A7%81%EB%B9%88-%EC%A3%BC%EC%9D%98%EC%82%AC%ED%95%AD/)  

# Servlet이란
자바로 동적 웹페이지를 보여주기 위해 사용하는 인터페이스

# Spring Web MVC란

### 용어 풀이 
Spring : 스프링 프레임워크를 사용  
Web : 웹 서비스에 사용  
MVC : MVC패턴을 사용  

# Spring Web MVC에 대한 이해를 위한 기반

Servlet과 Spring Web MVC에 대한 이해를 하기 위해서는 process와 thread의 메모리 구조에 대해 알아야 한다.  
추가적으로, 싱글톤(Singleton)을 함께 익혀 스프링 빈을 사용한 스레드 처리까지 이해해보자.   

## process와 thread

![process-structure](https://user-images.githubusercontent.com/55550753/136681509-c0d84f99-d69c-472d-acc7-d1ae7b5277c1.PNG)

- **code** : 스레드 간 자원 공유, 명령어 저장
- **data** : 스레드 간 자원 공유, 전역변수 또는 static 변수 저장
- **heap** : 스레드 간 자원 공유, method area에 클래스 데이터(멤버 변수 등)를 저장하고, 프로그램 실행 도중 생성한 객체를 heap에 저장한다. 따라서 런타임 중 크기가 결정된다.
- **stack** : 스레드 간 자원을 공유하지 않음, 지역 변수 저장, 컴파일 시점에 스택 크기 결정

한 프로세스에는 1개 이상의 스레드가 존재한다.  

아래는 스프링 빈과 스레드 간의 관계이다. 
여러 스레드 간에 스프링 빈을 공유하고 있는 것을 볼 수 있다.   

![bean-thread](https://user-images.githubusercontent.com/55550753/136683619-03fa410b-14ff-4c4c-835a-d0bd003c4a42.PNG)  

## 싱글톤

싱글톤이라 함은 애플리케이션이 시작될 때 어떤 클래스가 최초 한번만 메모리를 할당하고(Static) 그 메모리에 인스턴스를 만들어 사용하는 디자인패턴이다.

### 싱글톤의 장점

1. 고정된 메모리 영역을 얻으면서 한번의 new로 인스턴스를 사용하기 때문에 메모리 낭비를 방지할 수 있다.
2. 또한 싱글톤으로 만들어진 클래스의 인스턴스는 전역 인스턴스이기 때문에 다른 클래스의 인스턴스들이 데이터를 공유하기 쉽다.
- DBCP(DataBase Connection Pool)처럼 공통된 객체를 여러개 생성해서 사용해야하는 상황에서 많이 사용

**스프링 빈**은 힙 영역에 할당되어 **싱글톤**으로 관리된다.    
그리고 그 스프링 빈을 여러 스레드가 접근하여 사용하게 된다.  

따라서 스프링에서 사용하는 @Service, @Configuration, @Controller, @Component는 모두 스프링 빈이고 싱글톤 객체라는 것을 알 수 있다.  

### 이제 여러 요청이 왔을 때를 생각해보자.  

1. 여러 개의 요청이 오면 웹 컨테이너에서 각 요청에 대해 thread를 생성한다.   

2. thread내부에서 request를 받아 새로운 request객체를 생성해 handler mapping을 통해 적절한 컨트롤러로 전달한다.  
   
3. thread내부에서 여러 싱글톤 스프링 빈을 사용해 서비스 로직을 실행한다.  

4. thread내부에서 서비스 로직 실행 후 새로운 response객체를 생성해 view로 응답한다.

- 당연한 이야기일 수도 있지만 request객체는 싱글톤이 아니다.
- 상식적으로 request가 가지는 속성값이 모두 다를텐데 모든 스레드가 공유하는 하나의 인스턴스인 싱글톤으로 관리할 수 없다. (조금만 생각해보면 request를 받고 dto를 전달할 때 새로운 인스턴스를 생성해 전달할 때도 있지 않았는가?)
- 어떤 스레드에서 스프링 빈에 접근하더라도 실행 결과는 예상한 값이 나와야 한다. (thread-safe) 
- 따라서 thread-safe하게 스프링 빈을 싱글톤으로 관리할 수 있다는 것은 객체가 **불변객체**라는 말이다.  
- 불변객체는 간단하게 말하면 클래스 내부의 멤버변수가 변하지 않는 객체를 말한다.  
- 가변객체는 클래스 내부의 멤버변수의 값이 변하는 객체를 말한다.  

아래는 불변 객체를 싱글톤으로 사용한 모습이다.  

```text
public class Singleton {

    private static Singleton singleton = new Singleton();

    private Singleton(){}

    public static Singleton getInstance(){
        return singleton;
    }

    public int add(int num){
        return ++ num;
    }

}

class AddTest{
    public static void main(String[] args) {
        Singleton singleton = Singleton.getInstance();

        int[] array = { 0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19 };

        for(int i : array) {

            new Thread(() -> {
                System.out.println(singleton.add(i));
            }).start();

        }
    }
}
``` 
아래는 가변 객체를 싱글톤으로 사용한 모습이다.  

```text
public class Singleton {

    int num;


    private static Singleton singleton = new Singleton();

    private Singleton(){}

    public static Singleton getInstance(){
        return singleton;
    }

    public int add(){
        return ++ num;
    }

}

class AddTest{
    public static void main(String[] args) {
        Singleton singleton = Singleton.getInstance();

        int[] array = { 0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19 };

        for(int i : array) {

            new Thread(() -> {
                System.out.println(singleton.add());
            }).start();

        }
    }
}
```
- 가변 객체를 싱글톤으로 구현했을 때 예상 가능한 값이 나올 수도 있지만 예상하지 못한 값이 나올 수도 있다.  
- 가변 객체를 싱글톤으로 관리할 수 없다는 것은 스레드에서 싱글톤 객체에 접근할 때 멤버 변수를 사용할 떄가 있을 것인데 그 떄 멤버 변수에 들어있는 값이, 다른 스레드에서 변형시켰다면 예상가능한 값이 아닐 수 있게 되기 때문이다.  
- 물론 이 문제는 스레드간 동기화를 시켜주면 되지만, 프로그래밍적으로 스레드를 능숙히 다루는 것은 상당한 노력과 지식이 요구되고 관리 또한 불편하다.
- 참고하자면, Spring Microservices in Action의 저자인 존 카넬이 말하기를 자신도 스레드를 능숙히 다루기는 어렵다고 말했다. 
- 종합하면 싱글톤 패턴을 사용하려면 **불변객체**를 사용하는 것이 맞다.  
- 또한 Spring Bean을 thread-safe하게 사용하기 위해서 멤버변수와 지역변수를 명확하게 구분하고 사용해야 하는 것이 중요하다.  
- 메소드 내부의 지역변수는 thread-safe한데 반해 멤벼변수는 그렇지 않기 때문이다.  
- **따라서 스프링에서 멤버변수는 Dependency Injection에 사용하는 bean일 경우만 사용하는 것이 바람직하다.**

### 결론 

스프링 프레임워크에서 싱글톤을 빈으로(IOC, DI) 관리하기 때문에, 개발자가 직접 싱글톤을 구현해 관리할 필요는 없지만,  
싱글톤과 멀티 스레드 동작방식을 이해하고 있어야 thread-safe하게 빈을 사용할 수 있다.  
  
### 멀티 스레드 참고

[스프링 빈은 thread-safe할까?](https://alwayspr.tistory.com/11)  
[멀티 스레드 환경에서 스프링 빈 주의사항](https://beyondj2ee.wordpress.com/2013/02/28/%EB%A9%80%ED%8B%B0-%EC%93%B0%EB%A0%88%EB%93%9C-%ED%99%98%EA%B2%BD%EC%97%90%EC%84%9C-%EC%8A%A4%ED%94%84%EB%A7%81%EB%B9%88-%EC%A3%BC%EC%9D%98%EC%82%AC%ED%95%AD/)

# Servlet in Spring Web MVC

![servlet-structure](https://user-images.githubusercontent.com/55550753/136682849-612d9349-d966-466d-af11-a949c1661f79.PNG)

Web Container(Servlet Container) : 요청이 들어오면 thread를 생성하고, servlet을 실행시킨다. 
servlet 인터페이스에 따라 servlet의 생명주기를 관리한다. 

### 서블릿 생명주기
- init() : 서블릿을 초기화하며 맨 처음 한번만 실행한다.
- service() : 요청응답을 처리하며 doGet(), doPost()등의 HTTP 메소드로 분기한다.
- doGet() 
- doPost()
- destroy() : 서블릿을 종료할 때 한 번만 실행한다.
  
![servlet-structure2](https://user-images.githubusercontent.com/55550753/136683045-20211b90-443f-4649-9b85-f8ef846895a2.PNG)

위에서 설명한 메모리 구조와 servlet동작 방식을 조합해서 보면 위 그림과 같이 동작한다.  
웹 컨테이너에서 요청별로 스레드를 만들어 서블릿과 통신하는 것이다.  

# Dispatcher Servlet : 기본적인 Servlet과 비교했을 때 Spring Web MVC가 갖는 장점

![dispatcher-servlet2](https://user-images.githubusercontent.com/55550753/136683226-dfddd887-6c30-4bcf-80e6-7a060f0baa3c.PNG)

Spring Web MVC에서는 url마다 servlet 구현체를 생성해 비효율적인 처리를 할 필요 없이 Dispatcher Servlet하나로 모든 요청을 처리한다.  

![dispatcher-servlet3](https://user-images.githubusercontent.com/55550753/136683271-ffa3a90b-cb4c-471a-98c6-4a2599059dd8.PNG)

기본 servlet에서는 MVC를 한 곳에서 처리했다면, Spring Web MVC는 Dispatcher Servlet을 사용한 모델, 뷰, 컨트롤러의 분리를 꾀했다.  
이로 인해 좀 더 확장성있는 개발을 진행할 수 있게 되었다.   
예로, 뷰만 수정하고 싶을 경우 Servlet코드 전체를 보고 수정하지 않고, 뷰 코드만 보고 수정하면 된다는 것이다.  

# Dispatcher Servlet 동작 과정

![dispatcher-servlet](https://user-images.githubusercontent.com/55550753/136683127-cb88498c-ed63-4387-96f3-2724d16483df.PNG)



