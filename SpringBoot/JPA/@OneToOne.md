연관관계 매핑 기본 - 일대일 관계
=============
# 1. 주 테이블에 외래 키

### @OneToOne 단방향

![onetoone](https://user-images.githubusercontent.com/55550753/135110429-6e3f0e11-ce36-4c80-9683-c8e77fcd998c.PNG)

위는 일대일 주 테이블에 외래 키, 단방향이다.  

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

![onetoone2](https://user-images.githubusercontent.com/55550753/135111199-07ca4048-a4ad-4101-850d-f04b5d33f30c.PNG)

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