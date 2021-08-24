연관관계 매핑 기본 - 다대일 관계
=============
(김영한님의 JAVA ORM 표준 JPA 프로그래밍을 읽고 쓴 글입니다.)  

엔티티에서 @ManyToOne, @OneToOne 같은 어노테이션은 자주 볼 수 있다.  
어노테이션의 의미도 직관적이라 @ManyToOne같은 어노테이션을 보면, 다대일 관계겠구나 쉽게 생각할 수 있다.  

근데 조금 더 생각해보니 @ManyToOne을 한 엔티티에 써주면 다른 엔티티에는 @OneToMany를 써주어야 하는가? 라는 물음과   
@JoinColumn과 @JoinTable은 정확히 어떤 때에 쓰는 것인가라는 궁금증이 생겼다.    
또한 객체와 객체 사이의 관계, 테이블과 테이블 사이의 관계도 명확하게 알고 싶었다.    

이 어노테이션을 사용하려면 정확하게 어떤 상황에서 어떻게 써야하는지 이론적으로 정립된 사고를 가지고 싶어서  
공부해 기록을 남긴다.   
 
## 연관관계의 기초  

처음부터 짚고 기억해야할 개념은  
기본적으로 테이블간의 관계는 양방향 관계이지만 객체 간의 관계는 단방향 관계라는 것이다.  
객체끼리 서로 의존성을 가져서 객체 간의 양방향 관계도 존재한다고 볼 수 있지만  
엄밀히 따지면 양방향 관계이기 이전에 두 개의 단방향 관계로 이루어져있는 것이다.  

테이블간의 관계가 양방향 관계인 이유는 외래키 하나만 있으면 조인이 가능하기 때문이다.   

이 기본 개념을 기억하고 다음으로 넘어가보자.  

## 다대일(N to 1) 관계  

여러 관계 중에 제일 먼저 알아야 할 관계인 다대일 관계는 말 그대로 다대일 관계를 말한다.  

![20210823relation1](https://user-images.githubusercontent.com/55550753/130465377-d73db525-27f9-4d26-a801-929c4fa4b72f.PNG)  

위는 클래스 레벨의 단방향 관계를 보여준다.  

![20210823relation2](https://user-images.githubusercontent.com/55550753/130465663-9e0c9404-b47f-4e8a-afdd-b19f2c64609f.PNG)  

위는 인스턴스 레벨의 단방향 다대일 관계를 보여준다.  
여러 멤버가 하나의 팀을 가질 수 있는 것을 알 수 있다.   

jpa를 사용한 Member클래스와 Team클래스 코드를 예시로 보자.  

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

        // 연관관계 설정
        public void setTeam(Team team) {
          this.team = team;
        }
        //Getter, Setter ...
    }

    @Entity
    public class Team {
        @Id
        @Column(name="TEAM_ID")
        private String id;

        private String name;

        //Getter, Setter ...
    }

원래 ddd를 위해서 엔티티에는 setter를 설정하면 안 되고 추가적으로 레이어를 두어야 하지만, 연관관계를 간단하게 설명하기 위해서  
엔티티에 setter를 두어 설명하겠다.  

이 다대일 관계에서 주목할 점은 @ManyToOne, @JoinColumn(name="TEAM_ID") 어노테이션이다.  

@ManyToOne은 다대일 관계를 보여주는 것으로 연관관계를 매핑할 때 필수적으로 필요한 어노테이션이다.  
연관관계 매핑이라 함은 RDBMS의 테이블 간의 관계를, 객체 지향적 코드에 매핑해주는 것을 말한다. (아래 다이어그램 참고)  
이로 인해, 테이블은 테이블대로 구성가능하고, 객체는 객체 지향적으로 코드를 짤 수 있다.   
@JoinColumn은 외래키를 매핑할 때 사용한다. 여기서는 name="TEAM_ID"이므로 Member테이블의 TEAM_ID가 외래키가 되겠다.   

![20210824relation1](https://user-images.githubusercontent.com/55550753/130499027-e3b87601-df9a-4828-aa55-70eec8ab1393.PNG)  

이렇게 하면 테이블의 간의 관계에서, Member테이블의 TEAM_ID컬럼에는 TEAM테이블의 TEAM_ID만 들어가게 되지만,   
객체 관계에서 Member인스턴스에는 Team인스턴스가 포함되어 있으므로 Member인스턴스에서 getTeam을 해주면 연관관계에 있는 team을 참조할 수 있다.    

이 장면에서 주목해야할 점은 객체 지향적으로 설계했기 때문에, 연관관계에 있는 테이블이 필요할 때 Member인스턴스를 만들고 Member인스턴스의 TEAM_ID(외래키)를 사용해서  
2차적으로 sql을 쓸 필요가 없다는 것이다. 그냥 객체에서 team을 참조하기만 하면 된다.   

    Team team1 = new Team("team1","팀1");
    em.persist(team1);

    Member member1 = new Member("member1", "회원1");
    member1.setTeam(team1); // 연관관계 설정 member1 -> team1
    em.persist(member1);

    Member member2 = new Member("member2", "회원2");
    member2.setTeam(team1); // 연관관계 설정 member2 -> team1
    em.persist(member2);

    Member member = em.find(Member.class, "member1");
    Team team = member.getTeam();

em.persist()는 spring data jpa의 repository.save()와 비슷하다고 생각해주면 편하다.  
위 코드는 객체 지향적으로 member1의 연관관계에 있는 team을 참조하는 모습을 보여준다.  

이 사실이 별 것 아닌 것처럼 보일 수 있지만 nodejs에서는 객체 지향 프로그래밍을 사용하지 않았기 때문에   
항상 sql를 사용해서 조인하거나, sql을 한 번 작성해서 불러온 결과를 다른 연관관계에 있는 테이블을 참조하기 위해서 2차적으로 다시 sql를 써야 했다.   
그렇기에 sql문이 코드 전체에 만연할 수 밖에 없었다.   