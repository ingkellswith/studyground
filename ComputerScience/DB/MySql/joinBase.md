Inner Join과 Left Join의 차이
=============================
실무에서 데이터 추출을 하다 보면 join을 해야할 일이 자주 생긴다.
그 중에서도 mysql, mariadb에서는 innerjoin과 leftjoin을 특히 자주 사용한다.

기본적인 inner join, left join의 개념과, 실제 쿼리에서는 join이 어떻게 사용되는지 정리를 할 필요가 있다고 생각해 정리를 하게되었다.

먼저 개념 파트이다.
아래 같이 사람 테이블, 영화 테이블(유저가 시청한 영화를 기록하는 테이블)이 있다고 가정하자. 

## 사람 테이블

|people_id|name|age|viewed_movie|
|------|---|---|---|
|1|김지수|20|앤트맨|
|2|이민혁|21|배트맨|
|3|진세연|22|스파이더맨|
|4|정수정|22|어벤져스|
|5|남주혁|24|슈퍼맨|
|6|김지민|24|어벤져스|
|7|김남준|24|배트맨|

## 영화 테이블

|movie_id|movie_title|viewed_at|
|------|---|---|
|1|앤트맨|2010|
|2|배트맨|2011|
|3|슈퍼맨|2012|
|4|아이언맨|2012|
|5|스파이더맨|2011|

(참고로 mysql에서 join 명령어를 사용하면 inner join 명령어를 사용하는 것과 같다. 기본 join이 inner join이라는 것이다.)

이 두 테이블에 대해 left join과 inner join을 실행해보자.

    select
    *
    from
    people as p
    left join movie as m on p.viewed_movie = m.movie_title
    order by p.people_id asc;  

left join 쿼리의 결과는 아래와 같다.  

![20210826_data01](https://user-images.githubusercontent.com/55550753/130971495-9e00de36-55f1-4f99-88ea-2dbea3b29070.PNG)  

    select
    *
    from
    people as p
    inner join movie as m on p.viewed_movie = m.movie_title
    order by p.people_id asc;  

inner join 쿼리의 결과는 아래와 같다.

![20210826_data02](https://user-images.githubusercontent.com/55550753/130971817-d9f0388e-921b-4f99-ad45-cfe501c471b9.PNG)

차이점이 보이는가?

join을 개념적으로만 생각하면 컬럼에 대해 조인을 실행하는 건가 싶을수도 있다.  
하지만 join은 기본적으로 모든 컬럼을 다 합쳐서 보여준다.  
그래서 일부러 'select *'으로 from 이 어떤 data를 참조하고 있는지를 보여주었다.   

inner join과 left join의 차이는 로우에서 일어난다.  

1. p 테이블에 m을 조인한다면 p를 기준으로 m을 매핑하겠다는 뜻이다.   

2. p 테이블에 m을 left join한다면 p를 기준으로 m을 매핑하되, 외래키를 사용(on에서 외래키 사용)해서 매핑할 값이 없어도
결과 테이블에서 p 테이블의 로우값은 전부 보여달라는 뜻이다.  

1. p 테이블에 m을 inner join한다면 p를 기준으로 m을 매핑하되, 외래키를 사용(on에서 외래키 사용)해서 매핑할 값이 없다면
결과 테이블에서 그에 대한 로우값은 전부 삭제해달라는 뜻이다.

2,3 번은 위에서 보여준 쿼리의 결과로 쉽게 이해할 수 있을 것이다.

1번에 대해 예시로 구체화해 설명하자면,  
아래 쿼리를 실행해보자.  

    select
    *
    from
    movie as m
    left join people as p on p.viewed_movie = m.movie_title
    order by m.movie_id asc;

위 쿼리의 결과는 아래와 같다.  

![20210826_data03](https://user-images.githubusercontent.com/55550753/130974013-94770770-2015-4206-b5c8-46cc138462e6.PNG)  

(사람 테이블과 영화 테이블은 일대다 관계이다.  
한 사람이 여러 영화를 볼 수 있기 때문이다.)  

영화 테이블을 기준으로 했으므로 사람 테이블의 viewed_movie컬럼에 '어벤져스'데이터가 있어도  
결과 테이블에서는 어벤져스가 보이지 않는다.  
왜냐하면 영화 테이블을 기준으로 했기 때문이다.   

영화 테이블은 '어벤져스'에 관한 정보가 없으므로, 그에 따라 사람 테이블과 매핑되지 않은 것이다.    

기준에 대해서 감이 오는가?  

이 기준을 이해하면 right join에 대해서도 이해할 수 있을 것이다.  
간단하다. p right join m을 하면 m left join p와 결과가 동일하게 나온다.  
아래 쿼리를 보자.  

    select
    *
    from
    people as p
    right join movie as m on p.viewed_movie = m.movie_title
    order by m.movie_id asc;  

![20210826_data05](https://user-images.githubusercontent.com/55550753/130976830-9a831c8e-54c4-4bbe-ae3a-f05ac9133646.PNG)

결과가 동일한 것이 보인다.  

이렇게 쿼리를 사용해서 left join과 right join, inner join의 개념을 정리해볼 수 있었다.  
다음에는 좀 더 복잡한 쿼리를 사용해 join에 대한 이해도를 더 높여보겠다.  







