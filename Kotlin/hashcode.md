코틀린에서의 객체 비교
======================
비교를 시작하기 전에 이해도를 높이기 위해 자바의 객체 비교부터 알아보자.  

자바의 객체비교는 hashcode()와 equals()가 있다.  

![hashcode](https://user-images.githubusercontent.com/76764942/129033950-61fad4fd-d131-4fa1-89ca-8112462c0a86.png)  

hashcode()를 통과하고 equals()까지 통과해야 같은 객체로 인정되는 것을 볼 수 있다.  
(hashcode특성상 hashcode()가 같아도 논리적인 값은 다를 수 있음을 인지하자.)
그렇다면 hashcode()는 무엇이고 equals()는 무엇인가?  

    class Main {
        public static void main(String[] args) {
            String a = "hello";
            String b = "hello";
            System.out.println(System.identityHashCode(a)==System.identityHashCode(b));
            System.out.println(a.equals(b));
            System.out.println(
            System.identityHashCode(new Test(a))==System.identityHashCode(new Test(b))
            );
            System.out.println(new Test(a).equals(new Test(b)));
        }
    }

    class Test{
        String a;
        Test(String a){
            this.a=a;
        }
    }  

결과값  

    true
    true
    false
    false

간단하게 보면 hashcode()는 주소를 비교한다, 즉 물리적으로 비교하는 것이다.   
equals()는 실제 값을 비교한다, 즉 논리적으로 비교하는 것이다.   

(repl에서 hashcode가 되지 않는 것 같아 System.identityHashCode()로 대체한다.)  
위 코드에서 String의 비교는 System.identityHashCode()에서 같은 상수풀을 가리키고 있음을, equals()는 값이 동등함을 알 수 있다.  

그런데 String이나 Int같은 값의 비교는 hashcode와 equals로 할 수 있지만, 위에서 false가 나온 것으로 보아 알 수 있듯이  
객체 비교를 하려면 equals()와 hashcode()를 override해서 사용해야 한다.  
이 메서드들은 객체의 어떤 필드를 비교의 대상에 넣어야 하는지 모르기 때문이다. 그렇기에 사용자가 직접 equals()와 hashcode()를 재정의하는 것이다.  
한 메서드만 override하면 되지 않느냐는 물음도 있을 수 있다.  
하지만 그렇게 하는 것은 Collection(HashSet, HashMap, HashTable)을 사용할 때 문제가 발생한다.  
이 Collection들은 위에서 봤던 그림과 같은 과정을 거쳐 동등성 비교를 하게 되기 때문이다.  

그렇기에 자바에서의 equals(), hashcode()의 override는 상당히 번거로운 뿐더러, 재정의할 때마다 논리적 이상이 있는지 점검해야하고, 실수하기도 쉬운 것이다.  
하지만 우리에겐 코틀린이 있다..! 자바를 사용하지 않고 코틀린을 사용하게 되면 이 문제에서 자유로워질 수 있다.  

아래 코드를 보자.  

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
결과값이 false가 나왔으므로 메서드 재정의를 해야하나 생각할 수 있다. 하지만 그럴 필요가 없다. 아래 코드를 보자.

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

Test 클래스에 data만 붙인 것으로 객체 비교가 가능해졌다.  
자바에서 객체 비교를 할 때보다 훨씬 편해졌다. data한 단어만 붙이면 되니까 말이다.  
이 data class는 dto에서 자주 사용하게 되므로 유념해두어야 한다.   
또한 equals, hashcode 멤버만 derive한 것이 아니라  
다른 자주 쓰는 toString(), componentN(), copy()도 derive해준다.  
이 멤버들은 추후 포스팅하도록 하겠다.  

코틀린 공식문서
: https://kotlinlang.org/docs/data-classes.html