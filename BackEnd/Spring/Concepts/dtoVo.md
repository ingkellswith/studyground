DTO vs VO vs Entity
===
개념 정리
# Dto
레이어 간 데이터 교환을 위해 사용하는 **가변 객체**.

# Vo
의미 있는 값을 표현하는 **불변 객체**.  
핵심 역할은 equals()와 hashcode()를 오버라이딩해서  
내부에 선언된 속성의 모든 값들이 vo객체마다 값이 같아야 같은 객체라고 판단할 수 있게 하는 것이다.

# Entity
repository와 맞닿아 있는 클래스, 데이터베이스-테이블을 객체 지향적으로 프로그래밍하기 위해 사용하는 클래스이다.  

dto vs entity를 설명해보면,
entity 클래스 자체를 객체로 생성해서 dto처럼 사용할 수 있지만 지양하는 것이 좋다.  

1. **entity를 requeest/response를 위한 클래스로 사용하게 되면 여러 클래스에 영향을 준다.**  
    entity클래스는 많은 서비스 클래스와 비즈니스 로직의 중심점 역할을 하는데 response를 바꾸기 위해서   
    entity클래스를 바꾸는 것은 다른 많은 클래스에 영향을 줄 수 있다.  
    또한 entity클래스의 코드가 길어질 우려가 있다.  
    그리고 entity를 request/response로 사용하겠다는 것은 ddd규칙에 어긋난다.  
    request/response는 view layer에 맞닿아 있기 때문이다.  

2. **entity클래스에는 setter를 두지 않는다.**  
    (setter를 만들지 않는 것은 사실 dto에도 적용된다.  
    절대적으로 dto에서 setter를 만들지 말라는 것이 아니라, 정말 setter가 필요한지에 대해서  
    충분히 고민 한 후 setter를 적용시켜야 한다는 것이다.  
    setter가 없을수록 로직이 깔끔해지기 때문이다.)    

    setter를 두게 되면 해당 클래스의 인스턴스 값들이 언제, 어떻게 변할 지 파악하는 것이 힘들게 된다.   
    값 변경이 필요하면 매개변수가 없는 public method를 만들어서 정해진 값으로만 변경가능하게 한다.  

정리하면, entity의 속성값을 변경하면 영속성 모델을 표현한 entity의 순수성이 모호해지기 때문에  
entity의 값을 전달하기 위한 dto객체를 추가적으로 만드는 것이 좋다.  

# 어떻게 dto,vo를 구분지어 사용해야 하는가?
dto, vo의 명확한 구분보다는 프로젝트 내부의 규칙에 맞게 사용하는 것이 좋다.  
이를테면, kotlin을 사용한 spring boot프로젝트를 사용할 때 dto를 data class로 사용한다면  
equals()와 hashcode()가 자동으로 오버라이딩되어 dto가 곧 vo의 속성인  
'**프로퍼티**의 값이 같다면 같은 객체로 취급한다.'을 가지게 된다.  
```text
fun main(args: Array<String>) {
  val person1 = Person("John")
  val person2 = Person("John")
  person1.age = 10
  person2.age = 20
  println(person1)
  println(person2)
}

data class Person(val name: String) {
    var age: Int = 0
}
```
위와 같이 선언하게 되면 Person 클래스의 primary constructor에 선언된 name에 대해서 equals()와 hashcode()를 오버라이딩하게 된다.  
주의할 점은 primary constructor에 선언된 프로퍼티만 data class의 속성이 적용된다는 것이다.  
dataClass는 [데이터 클래스](https://app.gitbook.com/@allover3773/s/studyground/kotlin)폴더에 쓴 글을 참고하자.  
아래는 결과값이다.  
```text
Person(name=John)
Person(name=John)
```
name 프로퍼티만 data class의 속성을 가지므로 age는 toString()에서 제외된 것을 볼 수 있다.  

각설하고, 중요 포인트는 dto, vo는 모두 효과적인 데이터 전달을 위해서 사용하는 개념이므로   
효과적인 데이터 전달에 중점을 두고 이해해야 하고, strict한 개념 구분에는 너무 얽매이지 말도록 하는 것이다.  
 