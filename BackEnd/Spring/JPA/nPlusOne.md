N+1 문제 해결하기
===
# Reference
[N+1 문제](https://incheol-jung.gitbook.io/docs/q-and-a/spring/n+1)  
[JPA N+1 문제 및 해결방안](https://jojoldu.tistory.com/165)  
[MultipleBagFetchException 발생시 해결 방법](https://jojoldu.tistory.com/457)  
[JPA N+1 발생원인과 해결방법](https://cheese10yun.github.io/jpa-nplus-1/)  
[fetch join 시 별칭관련 질문입니다](https://www.inflearn.com/questions/15876)

# 페치 조인과 일반 조인의 차이  

- 일반 조인 실행 시 연관된 엔티티를 함께 조회하지 않는다.  
- JPQL은 결과를 반환할 때, select절에 지정한 엔티티만 조회할 뿐 연관관계를 고려하지 않는다.  
- 따라서 JPQL실행 후, 연관된 엔티티 n개가 영속성 컨텍스트에 없을 경우 n개의 쿼리를 추가로 실행한다.   
- 그래서 페치 조인을 사용해서 연관된 엔티티도 함꼐 조회하는 것.  

# 페치 조인의 특징과 한계
- 페치 조인 대상에는 별칭을 줄 수 없다.  
  - 만약 줄 수 있다고 해도, jpa설계상 연관된 엔티티를 가져올 때 제약을 두는 것은 권장하지 않는다.  
  - 예를 들어, team의 member를 list로 가져올 때 where로 조건을 걸어 몇 개만 빼서 가져올 수 없다는 것이다.  
  - 기본적으로 객체 그래프 탐색을 위해 연관된 엔티티는 다 가져오는 것이 맞다.  
- 둘 이상의 컬렉션은 페치 조인할 수 없다.
  - @Batchsize사용
  - 위 'MultipleBagFetchException 발생시 해결 방법' 참고 
- 일대다 필드를 페치 조인하면 페이징 인터페이스를 사용하기가 힘들다.  
  - 소위 말하는 데이터 뻥튀기, 카테시안 곱이 발생하여 페이징하기가 까다롭다.  
  - 예를 들어, db에서 하나의 데이터로 묶여야 할 로우들이 낱개로 취급되는 것을 들 수 있다.  
- 다대일 필드, 일대일 필드는 페치 조인해도 카테시안 곱이 발생하지 않기 때문에 페이징이 사용가능하다.   