코틀린에서의 객체 비교
======================
# Reference
[equals와 hashCode를 같이 재정의해야 하는 이유](https://dundung.tistory.com/224)

# 코틀린의 객체 비교  

    fun main(args: Array<String>) {
        val c = Test("yap",10)
        val d = Test("yap",10)
        println(c==d)
        println(c===d)
    }

    class Test(
        val a: String,
        val b: Long
    ){

    }

결과값

    false
    false

코틀린에서 ==는 논리적 동등성 비교를, ===는 주소 비교(물리적 동등성 비교)를 한다.  
Test class에서는 equals(), hashcode()가 재정의 되지 않았으므로 결과값들이 false가 나온다.  
data를 선언한 아래 코드를 보자.  

    fun main(args: Array<String>) {
        val c = Test("yap",10)
        val d = Test("yap",10)
        println(c==d)
        println(c===d)
    }

    data class Test(
        val a: String,
        val b: Long
    ){

    }

결과값

    true
    false

Test 클래스에 data만 붙인 것으로 객체 내부 값의 비교가 가능해졌다.  
이 data class는 dto에서 자주 사용하게 되므로 유념해두어야 한다.  

(참고로 코틀린에서 ==는 내부적으로 equals()를 호출한다.)    

또한, data class의 hashcode는 논리적 값이 같다면 같은 고유값을 반환한다.  
즉, 위의 c.hashCode()와 d.hashCode()는 같은 값을 반환한다.  
일반 class의 hashcode는 논리적 값이 같아도 다른 고유값을 반환할 것이다.    

그럼에도 불구하고 println(c===d)의 결과값이 false가 나오는 것은  
hashCode()는 같지만 실제 주소값은 다르기에 그렇다.  

data class는 equals, hashcode 멤버만 derive한 것이 아니라  
다른 자주 쓰는 toString(), componentN(), copy()도 derive해준다.   

코틀린 공식문서
: https://kotlinlang.org/docs/data-classes.html