연관관계 매핑 기본 - 다대다 관계
=========
@ManyToMany는 말 그대로 다대다 관계를 매핑해주는 어노테이션이다.  

    @Entity
    public class Member {
        @Id @Column(name = "MEMBER_ID")
        private String id;

        private String username;

        @ManyToMany
        @JoinTable (name = "MEMBER_PRODUCT", joinColumns = @JoinColumn (name = "MEMBER_ID"), inverseJoinColumns = @JoinColumn(name="PRODUCT_ID")
        private List<Product> products = new ArrayList<Product>();
    }

    @Entity
    public class Product {

        @Id @Column(name = "PRODUCT_ID")
        private String id;

        private String name;
    }

위는 다대다 단방향 매핑이다.  
@ManyToMany 어노테이션을 선언 후 @JoinTable을 써주면 새로운 엔티티를 만들지 않아도 jpa가 알아서 매핑을 해준다.  

@JoinTable과 @JoinColumn의 차이점은 @JoinTable은 연결 테이블을 지정하지만 @JoinColumn은 외래키를 지정한다는 것이다.  

따라서 @JoinTable을 사용하기 위해서는 테이블이 하나 더 필요한 셈이다.   

    @Entity
    public class Product {

        @Id
        private String id;

        @ManyToMany(mappedBy = "products")
        private List<Member> members;
    }

위는 다대다 양방향 관계의 Product 엔티티이다.  
다대일 관계에서 봤던 것처럼 mappedBy를 쓰면 사용가능하다.  

이런 @ManyToMany는 실제로는 사용하지 않는 것이 좋다.   
왜냐하면 연결 테이블에 추가적인 컬럼을 추가할 수 없기 때문이다.   
연결 테이블에 추가적인 컬럼을 추가한다면, Member나 Product엔티티에서 그것을 참조할 수 있어야 하는데,  
그렇게 할 수 없기 때문이다.  

즉 테이블의 확장성이 떨어지게 되는 것이다.  

그렇기에 **@ManyToMany대신 @ManyToOne + @ManyToOne 구조를 가져가야 한다.**  




