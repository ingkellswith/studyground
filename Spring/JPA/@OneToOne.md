연관관계 매핑 기본 - 일대일 관계
=============
# 1. 주 테이블에 외래 키

### @OneToOne 단방향

![onetoone](https://user-images.githubusercontent.com/55550753/135110429-6e3f0e11-ce36-4c80-9683-c8e77fcd998c.PNG)

위는 일대일 주 테이블에 외래 키, 단방향이다.  
(erd에는 깜빡하고 못 넣었지만 Member테이블의 LOCKER_ID는 **fk이자 uk**이다.)  

```text
@Entity
public class Member{
    @Id @GeneratedValue
    @Column(name="MEMBER_ID")
    private Long id;

    private String username;

    @OneToOne
    @JoinColumn(name="LOCKER_ID")
    private Locker locker;

    ...
}

@Entity
public class Locker{
    @Id @GeneratedValue
    @Column(name="LOCKER_ID")
    private Long id;

    private String name;

    ...
}
```
코드는 다대일 단방향과 상당히 유사하다.  

### @OneToOne 양방향

![onetoone2](https://user-images.githubusercontent.com/55550753/135113877-8e3c1b6c-6869-4fc3-8cb1-012be9d9ab1d.PNG)

위는 일대일 주 테이블에 외래 키, 양방향이다.  

```text
@Entity
public class Member{
    @Id @GeneratedValue
    @Column(name="MEMBER_ID")
    private Long id;

    private String username;

    @OneToOne
    @JoinColumn(name="LOCKER_ID")
    private Locker locker;

    ...
}

@Entity
public class Locker{
    @Id @GeneratedValue
    @Column(name="LOCKER_ID")
    private Long id;

    private String name;

    @OneToOne(mappedBy="Locker")
    private Member member;

    ...
}
```
역시 코드는 다대일 양방향과 상당히 유사하다.  

# 2. 대상 테이블에 외래 키

### @OneToOne 단방향

대상 테이블에 외래 키를 둘 때 @OneToOne 단방향 매핑은 존재하지 않는다.

### @OneToOne 양방향

(역시 erd에는 못 넣었지만 Locker테이블의 MEMBER_ID는 **fk이자 uk**이다.)  

![onetoone3](https://user-images.githubusercontent.com/55550753/135113358-e27f744f-ce16-489d-910c-92cee796d011.PNG)

```text
@Entity
public class Member{
    @Id @GeneratedValue
    @Column(name="MEMBER_ID")
    private Long id;

    private String username;

    @OneToOne(mappedBy="LOCKER_ID")
    private Locker locker;

    ...
}

@Entity
public class Locker{
    @Id @GeneratedValue
    @Column(name="LOCKER_ID")
    private Long id;

    private String name;

    @OneToOne
    @JoinColumn(name="MEMBER_ID")
    private Member member;

    ...
}
```

**주 테이블에 외래 키를 기본적으로 사용하되**, db설계의 방향성을 고려해서 어떻게 사용할 지 팀원들과 충분한 협의를 거치고 사용해야 한다.  
member한 명이 여러 locker를 사용하도록 db설계가 바뀔수도 있고, locker하나가 여러 member에 의해 사용되도록 db설계가 바뀔수도 있기 때문이다.  

member한 명이 여러 locker를 사용하도록 설계가 바뀔 경우에 대상 테이블에 외래 키 구조를 가져갈 경우,  
locker테이블의 외래키에 unique조건만 빼주면 코드 변경없이 유연하게 사용될 수도 있다는 장점이 있다.   
물론 locker하나가 member여러 명에 의해 사용되는 반대의 경우, member테이블의 외래키에 unique조건을 빼면 되므로, 주 테이블에 외래키 구조를 사용하는 것이 맞을 것이다.    

## 참고
또한 자체 PK를 생성하는 대신, 외래키 자체를 PK를 만들고 싶다면 @MapsId를 사용하면 된다.  