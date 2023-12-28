MVVM에 대해
===
- MVVM
[1. Level up your React architecture with MVVM](https://medium.cobeisfresh.com/level-up-your-react-architecture-with-mvvm-a471979e3f21)   
[2. React에서 MVVM 패턴 알아보기](https://velog.io/@dlrmsghks7/whatismvvmpattern) 

## MVVM
리액트에서의 Container, Presenter패턴으로 설명될 수 있는 MVVM은 아래와 같다.  

**View** : 화면에 데이터를 보여주는 컴포넌트로, **Presenter**라고 불리기도 하며 데이터를 가공하는 로직이 존재하면 안 된다.  
**ViewController** : View에 데이터를 전달하는 컴포넌트로, **Container**라고 불리기도 하며 이벤트를 처리하는 함수, setState를 처리하는 함수같은 데이터 가공 로직이 존재한다.  
**ViewModel** : Model의 데이터를 추출해 쉽게 재사용 가능하게 만든 컴포넌트이다. Model에서 받아온 데이터를 가공하는 로직이 포함된다. 개인적으로 뷰 모델은 스프링에 비유했을 때 dto에 가깝다고 생각한다.  
**Model** : 데이터 소스 역할을 한다. 개인적으로 모델은 스프링에 비유했을 때 Entity에 가깝다고 생각한다.  
**Provider** : Container들을 조합하는 역할을 한다. **Container간 연결**만이 목적이므로 로직이 없어야 한다.  

여기에서 **ViewController, ViewModel, Model**은 **로직**을 포함하므로 각 역할군 간의 책임을 명확하게 해야 한다.  