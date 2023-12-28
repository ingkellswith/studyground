Spring Data Jpa 관련 정리
===

## 벌크성 수정 쿼리 

```text
@Modifying(clearAutomatically = true)
@Query("update Member m set m.age = m.age + 1 where m.age >= :age")
int bulkAgePlus(@Param("age") int age);
```

벌크성 수정 쿼리는 영속성 컨텍스트를 무시하기 때문에 다음에 조회 쿼리가 나올 경우를 대비해 cleatAutomatically를 true로 세팅  

## 사용자 정의 리포지토리

![custom-repository](https://user-images.githubusercontent.com/55550753/145698643-6984421c-3bc0-4a12-8696-9385e3a50bfb.PNG)

출처 : https://jojoldu.tistory.com/372  

querydsl을 사용할 때 커스텀 리포지토리를 위와 같은 방식으로 사용자 정의할 수 있다.  

또한 순수 jpa를 사용한다거나, jdbc template, mybatis 같이 인터페이스의 메소드를 직접 구현하고 싶다면 위와 같은 방식으로 커스텀 리포지토리를 정의할 수 있다.  

```text
public interface MemberRepositoryCustom {
    List<Member> findMemberCustom();
}
```
```text
@RequiredArgsConstructor
public class MemberRepositoryCustomImpl implements MemberRepositoryCustom {
    private final EntityManager em;
    @Override
    public List<Member> findMemberCustom() {
        return em.createQuery("select m from Member m").getResultList();
    }
}
```
```text
public interface MemberRepository extends JpaRepository<Member, Long>, MemberRepositoryCustom {
}
```
```text
List<Member> result = memberRepository.findMemberCustom();
```

interface의 이름은 상관없지만 구현체는 인터페이스이름 + Impl이 되어야 한다.  
위와 같이 정의했을 때, 원래 MemberRepository+Impl 이렇게 구현체 이름을 짜야 하지만  
**스프링 데이터 2.x버전부터는 MemberRepositoryCustom+Impl 이 방식도 지원해준다.**     
새로 만든 커스텀 인터페이스에 Impl을 추가하면 되므로 더 직관적이고, 여러 인터페이스를 분리해서 사용가능하기 때문에 가능하다면 이 방식을 사용하는 것이 맞다.  

위와 같이 사용하는 것은 Spring Data Jpa의 특성을 활용한 것이다.  

그리고 항상 위처럼 사용자 정의 리포지토리가 필요한 것은 아니다. 그냥 임의의 리포지토리를 만들어도 된다.  
예를들어 MemberQueryRepository를 인터페이스가 아닌 클래스로 만들고 스프링 빈(@Repository)으로 등록해서  
그냥 직접 사용해도 된다. 물론 이 경우 스프링 데이터 JPA와는 아무런 관계 없이 별도로 동작한다.  