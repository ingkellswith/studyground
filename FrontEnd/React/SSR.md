SSR(Server Side Rendering)
===========================
리액트로 개발해오면서 보통 백엔드로 request요청을 보내서 json을 받아오는 요청이 대부분이었다.  
그런데 블로그 페이지 같이 유저가 생성하고 보는 페이지도 application/json response 타입으로 작동하는지 궁금해서
찾아보게 되었다.  
찾아보던 중 application/json 타입으로 통신하는 곳과 text/html로 통신하는 곳이 있었다.  
### application/json 타입 ([d2.naver.](https://d2.naver.com/helloworld/0881672))
![jsontype](https://user-images.githubusercontent.com/55550753/128595221-d6af2797-8d6e-4ffa-b40a-9bca672d2793.PNG)
### text/html 타입 (https://ohou.se/contents/card_collections/9229412?affect_type=CardIndex&affect_id=0)
![htmltype](https://user-images.githubusercontent.com/55550753/128595236-68f5fd1e-521a-442c-8262-b9f2881c42d9.PNG)

이렇게 두 종류의 타입을 사용한다는 것은 알겠다.  
첫번째 사진처럼 json으로 통신할 경우에는 클라이언트 측에서 서버에 json을 요청한 후   
json응답을 반환하면 그 때 클라이언트측에서 렌더링하게된다. 내가 지금까지 해왔던 리액트는 csr이므로 이런 방식을 사용했다.  

그런데 text/html의 경우 preview를 보면 유저들이 보던 페이지와 똑같은 페이지가 렌더링되었다.  
### preview
![htmltype-preview](https://user-images.githubusercontent.com/55550753/128596366-0a545f25-ee37-4bc2-94f1-07440b515db0.PNG)
이 response를 보내주기 위해서 서버측에서 ssr을 사용해 html+css를 구현한 것이다.  

아래 다이어그램을 보자  

![ssr-diagram](https://user-images.githubusercontent.com/55550753/129198716-2f6f204b-dce2-46cd-a1a2-2d336a822690.png)  

이 다이어그램으로 보아하니 ssr은 js및 fetch요청을 나중으로 미루는 것을 알 수 있다.  
그런데 어떻게 리액트 코드를 html, css만 먼저 서버에서 렌더링해서 내려주어야 할까?

사실 이는 csr로 코딩하던 리액트로는 구현하기가 번거로운 부분이 많다.  
서버에서 렌더링하도록 설정해야할 것이 많고 기술적으로도 js만 따로 분리하는 것이 쉽지 않기 때문이다.  
하지만 이를 해결할 수 있도록 리액트에서 서버 사이드 렌더링을 위해 권장하는 방법이 있다.  
바로 next.js라는 프레임워크이다. 이 프레임워크는 사용하는 것만으로도 ssr을 가능하게 해준다.
즉 서버에서 리액트 코드를 실행해서 렌더링해주는 것이다.

이 프레임워크는 개발자가 설정해야할 부분들을 많이 줄여주고, 개발자는 리액트 코드에만 집중할 수 있게 해준다.  
사실상 리액트 서버 사이드 렌더링을 하면 next.js를 쓴다고 보면 된다. 이미 tencent, netflix 등 많은 기업들의 next.js사용으로 인해 검증되었다.  
(네이버 블로그팀에서도 csr에서 서버 사이드 렌더링으로 전환할 때 next.js프레임워크를 선택하지 않은 것을 후회 한다고 할 정도니 뭐..)  

![nextJS](https://user-images.githubusercontent.com/55550753/129199627-7602995a-4992-4f30-a7c0-18bc6d192b23.png)  

그렇다면 많은 기업들이 왜 ssr을 사용할까?  
서버 사이드 렌더링을 함으로써 얻는 이득은 크게 두 가지가 있다. 
1. 검색 엔진 최적화  
   검색 엔진은 자바스크립트를 실행하지 않기 때문에 js는 검색 조건에서 제외된다. (물론 구글은 js도 검색하지만 구글 역시 ssr에 더 가산점을 준다.)  
   js를 실행해 렌더링하는 csr에 비해 ssr은 html,css를 먼저 렌더링해 클라이언트에 보내주기 때문에 검색 엔진에 의해 검색되기 좋은 조건이다.  
2. 빠른 첫 페이지 렌더링   
   사실 나는 이것이 제일 큰 장점이라고 생각한다. csr은 사용자의 기기 성능의 영향을 받기 때문에 사용자마다 렌더링 속도가 다를 수 있고, 기본적으로  
   js를 모두 로딩한 시점에 html이 로딩되기 때문에 첫 페이지 렌더링이 느리다. 반면 ssr은 사용자의 환경에 상관 없이 똑같이 서버에서 렌더링해서 보내주기  
   때문에 첫 페이지 렌더링이 빠르다. 그에 따라 사용자 경험이 좋아지는 것은 당연하다.

물론 ssr에도 '서버 렌더링에 따른 서버 부하'같은 단점이 있겠지만 그보다 사용함으로써 얻는 이득이 현업에서 더 클테니까 사용한다고 생각한다.

react ssr에 대해서 잘 설명해주신 영상이 있으니 복습이 필요하면 다시 보도록 하자.  
링크 : https://tv.naver.com/v/16970015#comment_focus

### 영상 안의 포인트 : waterfall
![ssr-effect](https://user-images.githubusercontent.com/55550753/128600426-268fe4f8-d755-4062-8c8b-b336f79b8e58.PNG)  
블루 라인까지 : 브라우저가 HTML을 전부 읽고 DOM 트리를 완성하기까지의 기간 (이미지 파일이나 스타일시트 등의 기타 자원은 기다리지 않는다.)  

블루 라인부터 레드 라인까지 : componentdidmount부터 시작하는 리액트 초기화 및 이미지, 외부스타일시트까지 불러오는 것이 끝나기까지의 기간  

레드 라인 이후 : 서비스js 로딩 및/실행. 예를 들면 댓글란이 맨 아래에 있어서 보이지 않는데 레드 라인 이전에 동작하게 만들어줄 필요는 없으므로   
레드 라인 이후부터 렌더링하는 것이다. 블루 라인부터 레드라인까지의 기간, 즉 DOMContentLoaded부터 load까지의 기간을 줄이기 위한 것이다.  

