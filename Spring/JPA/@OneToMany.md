연관관계 매핑 기본 - 일대다 관계
=============
일대다 매핑은 매핑한 객체가 관리하는 외래 키가 다른 테이블에 있다는 특징이 있다.  
이 경우 성능 문제도 있지만 관리가 부담스럽기 때문에 다대일 양방향 매핑을 사용하는 것이 좋다.  

## 일대다 단방향 연관관계

![onetomany](https://user-images.githubusercontent.com/55550753/135619316-3f48c88b-e6ec-4fd0-a56b-0de2a1512235.PNG)

## 일대다 단방향 매핑

```text
@Entity
public class Team{
    @Id@GeneratedValue
    @Column(name="TEAM_ID")
    private Long id;

    private String name;

    @OneToMany
    @JoinColumn(name="TEAM_ID")
    private List<Member> members = new ArrayList<Member>();

    ...
}

@Entity
public class Member{
    @Id@GeneratedValue
    @Column(name = "MEMBER_ID")
    private Long id;

    private String username;

    ...
}
```
## 다대일 단방향 연관관계

![20210824relation1](https://user-images.githubusercontent.com/55550753/130499027-e3b87601-df9a-4828-aa55-70eec8ab1393.PNG)  

## 다대일 단방향 매핑

```text
@Entity
public class Member {
    @Id
    @Column(name="MEMBER_ID")
    private String id;
    private String username;

    // 연관관계 매핑
    @ManyToOne
    @JoinColumn(name="TEAM_ID")
    private Team team;

    ...
}

@Entity
public class Team {
    @Id
    @Column(name="TEAM_ID")
    private String id;

    private String name;

    ...
}
```
비교를 위해 다대일 연관관계를 가져와봤다. 차이점이 보이는가?  

데이터베이스에서 외래키는 무조건 '다'쪽인 테이블이 관리하므로  
다대일 관계, 일대다 관계 모두 '다'쪽에 외래키가 있을 수 밖에 없다.  
따라서 일대다 관계에서의 @JoinColumn은 반대편 테이블의 외래키를 말하고,  
다대일 관계에서의 @JoinColumn은, @JoinColumn을 사용한 엔티티가 매핑된 테이블의 외래키를 말한다.   

일대다 양방향은 어찌저찌 그렇게 보이게 구현은 가능하지만 일대다 단방향의 단점을 그대로 가지기 때문에  
책으로만 훑고 넘어가겠다.  
일대다 양방향 매핑이 꼭 필요하다고 생각될 때 이 글을 다시 이어가는 것이 좋을 것 같다.  