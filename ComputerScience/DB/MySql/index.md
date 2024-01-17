인덱스(Index)
===
# 참조
[B-Tree 인덱스 구조](https://beelee.tistory.com/37)   
[DB 인덱스를 효과적으로 설정하는 방법 - 고려해야 할 4가지](https://yurimkoo.github.io/db/2020/03/14/db-index.html)  
[[mysql] 인덱스 정리 및 팁](https://jojoldu.tistory.com/243)  
[[mysql] MySQL IN절을 통한 성능 개선 방법](https://jojoldu.tistory.com/565?category=761883)  

# 카디널리티
cardinality : 차수
column의 차수가 높을수록 해당 column 데이터의 중복도가 낮음을 의미하고,
column의 차수가 낮을수록 해당 column 데이터의 중복도가 높음을 의미한다.

카디널리티가 높을수록 인덱스를 걸기 좋은 조건이며, 성능이 개선된다.  
이유는 인덱스를 통해 한 번에 많은 것을 걸러낼 수 있기 때문이다.  
[[mysql] 인덱스 정리 및 팁](https://jojoldu.tistory.com/243)가 기억나지 않는다면 다시 읽어보자.   

# 인덱스 제대로 사용하기
여러 컬럼으로 인덱스를 잡는다면 카디널리티가 높은순에서 낮은순으로 구성하는게 더 성능이 뛰어나다.

인덱스를 여러 컬럼으로 구성 시 첫 번째 컬럼이 조회 쿼리에 존재하는지의 유무가 중요하다.  
컬럼의 유무에 따라 인덱스를 타느냐, 풀스캔을 하느냐가 갈리기 때문이다.  
조회 쿼리 사용시 인덱스를 태우려면 최소한 첫번째 인덱스 조건은 조회조건에 포함되어야 한다.  
첫번째 인덱스 컬럼이 조회 쿼리에 없으면 인덱스를 타지 않는다는 점을 기억하면 된다.   

다만 인덱스 컬럼이 순서에 상관없이 조회 쿼리에 존재한다면,  
'옵티마이저'가 조회 조건의 컬럼을 인덱스 컬럼 순서에 맞춰 재배열하는 과정을 추가하여 성능상의 이슈는 크게 없다고 한다.  