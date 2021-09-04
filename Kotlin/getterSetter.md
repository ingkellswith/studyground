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

    objectMapper.writeValueAsString()  

그리고 data class에서 getter와 setter는 어떻게 처리할 수 있을까?  

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

data class를 이렇게 선언하면 setter, getter가 모두 활성화된다.
Test생성자에서 변수를 var로 선언해서 새로운 값 할당이 가능하고, 변수 선언에 접근 제한자를 붙이지 않으면  
기본적으로 public 접근 제한자가 붙어 있는 것과 같기 때문이다.   
(코틀린의 val, var은 마치 자바스크립트의 const, let과 같다.)  

이제 setter, getter를 제한해보자.  

아래 코드와 같이 변수를 val으로 선언하면 setter를 사용할 수 없게 된다.  
또한 아직 접근 제한자가 public이므로 getter는 사용할 수 있다.   

(참고롤 코틀린에서의 getter와 setter의 형태는 같다.  
아래 코드에서 c.a를 그냥 사용하면 getter이고, c.a에 값을 할당하면 setter가 되는 것이다.  )


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

하지만 이렇게 되면 한 번 생성한 인스턴스의 멤버들을 바꿀 수 있는 방법이 없게 된다.
따라서 인스턴스 내부에서만 바꿀 수 있게 설정할 필요가 있다.

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
이렇게 사용하면 setter를 사용제한할 수 있고,  
getter도 마찬가지로 private접근 제한자를 사용해서 외부의 사용을 차단할 수 있다.  

자바의 setter, getter와 코틀린의 setter, getter를 비교했을 때 차이점은  
자바는 setName(), getName()이렇게 set, get을 써주어 명시적으로 사용하지만  
코틀린은 setter, getter 모두 instance.name 같은 형태로 사용한다는 점이다.  

그리고 자바는 setter, getter가 없는 상태에서 getter, setter를 롬복을 사용해 만들어나가는 느낌이라면,  
코틀린은 기본적으로 setter, getter가 만들어져 있고 접근 제한자, 변수 타입을 통해서   
setter, getter의 사용을 제한하는 느낌이 강하다.  

    `자바스크립트에 익숙한 나에게, 코틀린의 getter, setter는   
    자바스크립트에서 object를 사용할 때와 비슷하기도 하고,  
    get, set같은 자주 쓰는 반복된 코드를 덜어줄 수 있어서 좋은 특성이라고 생각된다.`

