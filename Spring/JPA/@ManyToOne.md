연관관계 매핑 기본 - 다대일 관계
=============
## Reference
김영한 - 책 : 자바 ORM 표준 JPA 프로그래밍  
김영한 - 인프런 강의 : 자바 ORM 표준 JPA 프로그래밍
 
## 연관관계의 기초  

처음부터 짚고 기억해야할 개념은  
기본적으로 테이블간의 관계는 양방향 관계이지만 객체 간의 관계는 단방향 관계라는 것이다.  
객체끼리 서로 의존성을 가져서 객체 간의 양방향 관계도 존재한다고 볼 수 있지만  
엄밀히 따지면 양방향 관계이기 이전에 두 개의 단방향 관계로 이루어져있는 것이다.  

테이블간의 관계가 양방향 관계인 이유는 외래키 하나만 있으면 조인이 가능하기 때문이다.   

## 단방향 다대일(N to 1) 관계  

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

em.persist()는 엔티티 매니저가 관리하는 영속성 콘텍스트에 엔티티를 넣는 것이다.  
위 코드는 객체 지향적으로 member1의 연관관계에 있는 team을 참조하는 모습을 보여준다.  

이 사실이 별 것 아닌 것처럼 보일 수 있지만 nodejs에서는 객체 지향 프로그래밍을 사용하지 않았기 때문에   
항상 sql를 사용해서 조인하거나, sql을 한 번 작성해서 불러온 결과를 다른 연관관계에 있는 테이블을 참조하기 위해서 2차적으로 다시 sql를 써야 했다.   
그렇기에 sql문이 코드 전체에 만연할 수 밖에 없었다.   

다대일은 연관관계 매핑의 기초가 된다.  
반드시 숙지해야할 개념이다.  

## 양방향 다대일(N to 1) 관계  

단방향 다대일 관계에서는 객체 관점으로 바라봤을 때 '다'쪽에서 '일'을 참조할 수 있었고   
'일'쪽에서는 '다'를 참조할 수 없었다.  
하지만 양방향 다대일 관계에서는 객체 관점에서 '일'쪽에서 '다'를 참조할 수 있고 그 반대도 가능하다.  

아래 코드를 보자.  

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

        @OneToMany(mappedBy="team")
        private List<Member> members = new ArrayList<Member>();

        //Getter, Setter ...
    }

단방향 다대일 관계와 달라진 코드는 Team엔티티에 '다'쪽을 참조할 수 있는 필드가 어노테이션과 함께 추가된 것이다.   

당연하지만 Member엔티티에서 참조할 객체에 대해 @ManyToOne 어노테이션을 사용했으므로, Team엔티티는 참조할 객체에 대해 @OneToMany를 사용하게 된다.   
@OneToMany의 mappedBy는 양방향 관계에서 매핑을 위해 사용하는데, 반대쪽 엔티티의 참조 필드 이름을 값으로 주면 된다.   
mappedBy는 연관관계의 주인을 결정하기 위해 사용한다.   
연관관계의 주인이란 간단하게 테이블에 외래 키가 있는 엔티티를 말한다.    
 
연관관계의 주인을 정하는 이유는, 객체레벨에서 객체의 참조는 둘이어도,   
외래 키는 하나이기 때문에, 어떤 관계를 사용해서 외래 키를 관리해야할지 알려줄 필요가 있는 것이다.    
jpa에서는 이를 연관관계의 주인이라는 개념을 만들어 한 쪽 엔티티에서 테이블의 외래키를 관리한다.   

예시로 다대일 관계에서는 항상 '다'쪽에 외래 키가 존재하므로 '다'가 항상 연관관계의 주인이 될 것이다.    

![20210825relation1](https://user-images.githubusercontent.com/55550753/130648122-eef00c72-d472-4771-b14d-ab0fa6b43923.PNG)   

위는 객체레벨의 다대일 양방향 관계를 보여준다.   

양방향 연관관계는 주의해야할 점이 있다.  

## 객체의 양방향 연관관계는 양쪽 모두 관계를 맺는 것이 안전하다.  
   
간단히 설명하면 위의 코드처럼 다대일 양방향 관계를 사용할 것이라면 @ManyToOne을 '다'에 @OneToMany를 '일'에 넣는 것이다.  
이 어노테이션을 어느 한 쪽에서 생략해서는 안된다는 것이다.   

이 관계를 위해서 연관관계 편의 메소드라는 것이 존재한다.  

    public class Member {

        private Team team;

        public void setTeam(Team team) {
            this.team = team;
            team.getMembers().add(this);
        }

        //...
    }

위 코드는 member에서 team에 대한 참조를 set할 때 team에도 참조 관계를 적용해준 것이다.  
즉 메소드 하나로 양방향 관계를 설정해준 것이다.  

    public class Member {

        private Team team;

        public void setTeam(Team team) {
            if(this.team != null){
                this.team.getMembers().remove(this);
            }
            this.team = team;
            team.getMembers().add(this);
        }

        //...
    }
      
또한, 위 코드는 team에 대해 null체크를 진행해 현재 참조된 team이 없다면 기존 로직과 동일하게 작동하지만,    
참조된 team이 있다면 list에서 remove를 해서 과거의 양방향 관계를 끊어준다.  
양방향 다대일 관계에서 '다'쪽은 관계가 바뀌었어도 과거의 '일'은 관계가 그대로 남아 있기 때문이다.  

또한 연관관계 편의 메소드는 다대일에서 '다'쪽이든 '일'쪽이든 한 곳에만 사용하는 것이 설계상 좋다고 한다.   

## 정리

### 단방향 매핑만으로 테이블과 객체의 연관관계 매핑은 이미 완료되었다!  
### 단방향을 양방향으로 만들면 반대방향으로 객체 그래프 탐색 기능이 추가된다!  
### 양방향 연관관계를 사용하려면 객체의 양쪽 방향을 모두 관리해야 한다!  

따라서 단방향 매핑을 먼저 사용해보고, 반대방향의 객체 그래프 탐색이 필요하다 싶으면 양방향 매핑을 사용하면 될 것 같다.  
