조금 더 복잡한 sql
============================================

아래는 실무에서 데이터 추출을 위해서 사용했던 쿼리이다.  

    SELECT
        SUM(ijt.total_amount) - SUM(IFNULL(ljt.payoff_money, 0)) AS '잔액'
    FROM
        bond AS b
        LEFT JOIN (
            SELECT
                ps.bond_id,
                SUM(IFNULL(ps.payoff_money, 0))
            FROM
                payoff_sheet AS ps
            WHERE
                ps.payoff_at < '2021-07-25 00:00:00'
            GROUP BY
                ps.bond_id) 
        AS ljt ON b.id = ljt.bond_id
        JOIN 
            product 
        AS ijt ON b.product_id = ijt.id
    WHERE
        ijt.activated_at < '2021-07-30 00:00:00' 
        and ijt.activated_at >= '2021-07-01 00:00:00'
        and ijt.product_type="house-mortgage"
    ORDER BY
        ijt.id desc;

### bond : 채권 테이블
### payoff_sheet : 상환금 지불 내역 테이블
### product : 채권을 바탕으로 생성한 모집 상품 테이블

여기서의 bond와 product는 일대일 관계이고(채권 하나에 상품 하나),  
payoff_sheet에서는 상환금 지불 내역을 모두 기록하는 장부같은 개념으로 정의한다.  
그러니까 payoff_sheet는 상환금을 지불했을 경우 데이터가 로우로 쌓이는 것이다.  

from구문부터 보자.  
b left join ljt로 left join을 한 테이블을 만들었다.  
ljt는 일반 테이블이 아니라, payoff_sheet에서 bond_id, sum(payoff_money) 두 개의 컬럼만 선택한 커스텀 테이블이다.  
그리고 group by bond_id를 해줬기 때문에 bond_id별로 sum이 적용된 payoff_money를 확인할 수 있을 것이다.  

이 left join을 한 테이블에 더해서 product테이블도 inner join을 해서 총 3개의 테이블을 적절히 매핑한 것이다.  

주의해야할 것은 bond as b left join을 하느냐 bond as b inner join을 하느냐는 것인데  
이 조인의 종류에 따라 결과도 달라진다.  

inner join을 하게 되면 그에 따라 조건을 충족 못하는 bond로우들이 삭제됨에 따라  
ijt.activated_at < '2021-07-30 00:00:00' and ijt.activated_at >= '2021-07-01 00:00:00'  
을 만족시키는 로우들이 줄어들게 된다.    

where절의 ijt.activated_at 조건은 모든 product에 대해서 실행해야 하지만 생략된 product들이 존재하게 되는 것이다.  

AS ljt ON b.id = ljt.bond_id부분에서 상환된 금액이 있다면 ljt에 bond_id가 있어서 조건을 만족시키지만  
상환된 금액이 없다면 ljt에 bond_id가 없어서 조건을 만족시키 못하기 때문에   
inner join할 경우 조건을 만족시키지 못한다면 로우는 삭제된다.  

따라서 inner join과 left join의 차이점을 명확히 알고 적재적소에 사용하는 것이 중요하다.  