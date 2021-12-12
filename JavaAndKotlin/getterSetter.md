kotlin에서의 getter와 setter
============
    data class User(val name: String, val age: Int)

코틀린에서 데이터 클래스란 예시로, 위와 같이 데이터 클래스를 선언했을 때  
선언된 생성자를 통해서 다음 멤버들을 derive해준다.

    - equals()
    - hashcode()
    - toString()
    - componentN()
    - copy()

(equals(), hashcode()는 hashcoded.md를 참고)  

toString이 derive되었는지 확인하는 방법은 간단하다.  
class를 print해보고 data class를 프린트해보면  
class는 주소값이 출력되고, data class는 toString()을 이용한 스트링값이 출력된다.  

class 사용  

    fun main(args: Array<String>) {
    val c = Test("yap",10)
    println(c)
    }

    class Test(
    var a: String,
    var b: Long
    ){
    }

출력값  

    Test@31221be2

data class 사용  

    fun main(args: Array<String>) {
        val c = Test("yap",10)
        println(c)
    }

    data class Test(
        var a: String,
        var b: Long
    ){
    }

출력값   

    Test(a=yap, b=10)

이렇게 출력된다.  

또한, 이러한 "Test(a=yap, b=10)"같은 스트링이 아니라, {"a":"yap", "b":"10"} 이렇게 object형태로 사용하고 싶다면  
jackson의 objectmapper를 사용하면 된다.  

    val requestJson = objectMapper.writeValueAsString()  // 스트링 자료형의 JSON출력
    val jsonSend = JSONParser().parse(requestJson) as JSONObject   // 스트링 자료형의 json을 JSONObject 자료형으로 변경
    objectMapper.convertValue(jsonSend, className) // JSONObject를 className에 해당하는 클래스 타입으로 변경

## Getter, Setter

이제 getter, setter를 살펴보자. data class에서 getter와 setter는 어떻게 처리할 수 있을까?  

    fun main(args: Array<String>) {
        val c = Test("yap",10)
        c.a="yes"
        println(c)
    }

    data class Test(
        var a: String,
        var b: Long
    ){
    }

출력값

    Test(a=yes, b=10)

data class를 이렇게 선언하면 getter, setter가 모두 활성화된다.
Test생성자에서 변수를 var로 선언해서 새로운 값 할당이 가능하고, 변수 선언에 접근 제한자를 붙이지 않으면  
기본적으로 public 접근 제한자가 붙어 있는 것과 같기 때문이다.   
(코틀린의 val, var은 마치 자바스크립트의 const, let과 같다.)  

코틀린에서는 점(.)표기법으로 getter와 setter를 사용한다.   
자바처럼 getName(), setName() 같은 메소드를 만들 필요 없이 instance.name 처럼 쓰면  
코틀린 내부적으로 getter, setter를 만들어 처리해준다는 것이다.  

(점 표기법을 사용하므로 코틀린에서의 getter와 setter의 형태는 같다.   
아래 코드에서 c.a를 그냥 사용하면 getter이고, c.a에 값을 할당하면 setter가 되는 것이다.)  

이제 getter, setter를 제한해보자.  

아래 코드와 같이 변수를 val으로 선언하면 setter를 사용할 수 없게 된다.  
근데 문제는 val이므로 data class내부에서도 값을 바꿀 수가 없다.
또한 아직 접근 제한자가 public이므로 getter는 사용할 수 있다.   

    fun main(args: Array<String>) {
        val c = Test("yap",10)
        c.a="yes"
        println(c)
    }

    data class Test(
        val a: String,
        var b: Long
    ){
    }

출력값

    {error...}

이렇게 되면 한 번 생성한 인스턴스의 멤버들을 바꿀 수 있는 방법이 없다.  
따라서 값을 바꿀 수 있게 설정하려면 결국 var을 선언해야 한다.  

    fun main(args: Array<String>) {
        val c = Test("yap",10)
        c.jump()
        println(c)
    }

    data class Test(
        private var a: String,
        var b: Long
        ){
        fun jump(){
            this.a="power"
        }
    }

var a 선언으로 값을 바꿀 수 있게 설정했고, private로 내부에서만 값을 바꿀 수 있게 했다.  
이렇게 사용하면 setter를 사용제한할 수 있다.  
근데 새로 생긴 문제는 여기에서 a필드에 대해 private를 선언했으므로 getter를 사용할 수 없다는 단점이 존재한다.  

더 알아보고 싶다면 [kotlin entity에서 setter를 막을 수 있을까](https://multifrontgarden.tistory.com/272)를 읽어보자.  

## Custom Getter, Setter

코틀린에서는 getter, setter를 원하는대로 커스텀할 수도 있다.  

    class User(_id: Int, _name: String, _age: Int){
        val id: Int = _id
            get() = field
        
        var name: String = _name
            get() = field
            set(value){
                field = value
            }
        
        var age: Int = _age
            get() = field
            set(value){
                field = value
            }
        
    }

    fun main(){
        val user1 = User(1, "Ingkells", 20)
        user1.age = 35
        println("user1.age = ${user1.age}")
    }

위는 get(), set()의 기본 형태를 커스텀 getter, setter로 만들면 어떻게 되는지 보여준 것이다.  
즉, 위 코드에서는 get(), set()을 굳이 선언하지 않아도 코틀린 내부적으로 getter, setter를 만들어 처리해준다는 것이다.  

프로퍼티를 var로 선언하는 경우 get(), set()을 통한 getter, setter를 만들 수 있고,  
프로퍼티를 val로 선언하는 경우 get()을 통한 getter를 만들 수 있다.  

위에서 get(), set()에 field와 value가 사용되었는데, 이는 예약어이므로 기억해두는 것이 좋겠다.  

커스텀 getter, setter를 사용하면 get할 때 field.toUpperCase()를 사용해서 소문자가 아닌 대문자로 get이 가능하게 되는 등 편의성이 증진될 수도 있다.   

정리하면 자바의 getter, setter와 코틀린의 getter, setter를 비교했을 때 차이점은  
자바는 setName(), getName()이렇게 set, get을 써주어 명시적으로 사용하지만  
코틀린은 getter, setter 모두 instance.name 같은 형태로 사용한다는 점이다.  

