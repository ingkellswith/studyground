redux-thunk
============
# Reference
[리덕스(Redux)를 왜 쓸까? 그리고 리덕스를 편하게 사용하기 위한 발악 (i)](https://velopert.com/3528)  

# redux-thunk란
기존에 redux를 사용할 때 api를 요청하게 되면 redux-thunk를 사용했는데 redux-thunk의 정확한 개념은 무엇인지 알고 있지 않았다.  
그래서 이번에 정리를 하게 되었다.  

redux-thunk는 한 줄로 정리하면 redux를 사용할 때 비동기 dispatch를 하기 위한 비동기 미들웨어이다.

코드로 예시를 들어보자.  


    export default connect(mapStateToProps, mapDispatchToProps)(Container);  

보통 이렇게 컨테이너 마지막에 connect(mapStateToProps, mapDispatchToProps)이렇게 해주고 redux를 사용하게 된다.  
mapStateToProps 같은 경우는 일반 redux와 같이 store에서 가져와서 사용하면 된다.  
그런데 mapDispatchToProps에서 redux-thunk를 사용해 비동기 dispatch를 할 경우에는 일반 redux와 다른 부분이 있다.  

    function mapDispatchToProps(dispatch) {
        return {
            noThunk: (data) => dispatch({type:"example", data}),
        };
    } 
위는 redux-thunk를 사용하지 않은 일반 redux이다.  

    const mapDispatchToProps = (dispatch) => {
        return {
            withThunk: (data) => dispatch(asyncFunction(data)),
        };
    };
    const function asyncFunction(data) {
        return (dispatch) => {
            return axios.get(`/${data}`)
            .then((response) => {
                dispatch(asyncFunction2(response));
            }).catch((err) => {
                console.log(err);
            });
        };
    }  

위는 redux-thunk를 사용한 예시 redux 구조이다.   

redux-thunk에서 function이 function을 리턴하는 것을 thunk라고 부른다. 위에서는 asyncFunction이 되겠다.  
위와 같이 코드를 짜면 비동기적으로 redux의 store에 state들을 저장할 수 있게 된다.  

react-thunk 공식 문서에 설명이 깔끔하게 잘 되어 있다. 복습이 필요하면 다시 가보자.  
링크 : https://www.npmjs.com/package/redux-thunk  

### 유념해야 할 코드
위에서도 설명했지만 예시가 다양할수록 좋으니 공식문서에서 유의깊게 봐야 할 코드를 남겨두고 정리를 마치도록 하겠다.

    function makeSandwichesForEverybody() {
        //store의 state는 필요한 경우 getState()로사용할 수 있다.
        return function (dispatch, getState) {
            if (!getState().sandwiches.isShopOpen) {
                return Promise.resolve();
            }   
            // 당연하지만, plain action과 thunk를 혼합할 수 있다.(참고)
            return dispatch(
            makeASandwichWithSecretSauce('My Grandma')
            ).then(() =>
            Promise.all([
                dispatch(makeASandwichWithSecretSauce('Me')),
                dispatch(makeASandwichWithSecretSauce('My wife'))
            ])
            ).then(() =>
            dispatch(makeASandwichWithSecretSauce('Our kids'))
            ).then(() =>
            dispatch(getState().myMoney > 42 ?
                withdrawMoney(42) :
                apologize('Me', 'The Sandwich Shop')
            )
            );
        };
    }  

