N+1 문제 해결하기
===
# Reference
[N+1 문제](https://incheol-jung.gitbook.io/docs/q-and-a/spring/n+1)  
[JPA N+1 문제 및 해결방안](https://jojoldu.tistory.com/165)  
[MultipleBagFetchException 발생시 해결 방법](https://jojoldu.tistory.com/457)  
[JPA N+1 발생원인과 해결방법](https://cheese10yun.github.io/jpa-nplus-1/)  

# 페치 조인과 일반 조인의 차이  

- 일반 조인 실행 시 연관된 엔티티를 함께 조회하지 않는다.  
- JPQL은 결과를 반환할 때, select절에 지정한 엔티티만 조회할 뿐 연관관계를 고려하지 않는다.  
- 따라서 JPQL실행 후, 연관된 엔티티 n개가 영속성 컨텍스트에 없을 경우 n개의 쿼리를 추가로 실행한다.   
- 그래서 페치 조인을 사용해서 연관된 엔티티도 함꼐 조회하는 것.  