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

대부분 이렇게 두 종류의 타입을 사용한다는 것은 알겠다.  
첫번째 사진처럼 json으로 통신할 경우에는 클라이언트 측에서 서버에 json을 요청한 후   
json응답을 반환하면 그 때 클라이언트측에서 렌더링하게된다. 내가 지금까지 해왔던 리액트는 CSR이므로 이런 방식을 사용했다.  

그런데 text/html의 경우 preview를 보면 유저들이 보던 페이지와 똑같은 페이지가 렌더링되었다.  
### preview
![htmltype-preview](https://user-images.githubusercontent.com/55550753/128596366-0a545f25-ee37-4bc2-94f1-07440b515db0.PNG)
이 response를 보내주기 위해서 서버측에서 ssr을 사용해 html+css를 구현한 것이다.  
(react도 ssr을 지원하는데 next.js라는 프레임워크를 많이 쓴다고 한다.)  

react ssr에 대해서 잘 설명해주신 영상이 있으니 복습이 필요하면 다시 보도록 하자.  
링크 : https://tv.naver.com/v/16970015#comment_focus
### 영상 안의 포인트 : waterfall
![ssr-effect](https://user-images.githubusercontent.com/55550753/128600426-268fe4f8-d755-4062-8c8b-b336f79b8e58.PNG)  
블루 라인까지 : 브라우저가 HTML을 전부 읽고 DOM 트리를 완성하기까지의 기간 (이미지 파일이나 스타일시트 등의 기타 자원은 기다리지 않는다.)  
블루 라인부터 레드 라인까지 : componentdidmount부터 시작하는 리액트 초기화 및 이미지, 외부스타일시트까지 불러오는 것이 끝나기까지의 기간  
레드 라인 이후 : 서비스js 로딩 및/실행. 예를 들면 댓글란이 맨 아래에 있어서 보이지 않는데 레드 라인안에서 렌더링할 필요는 없으므로  
레드 라인 이후부터 렌더링하는 것이다. 블루 라인부터 레드라인까지, 즉 DOMContentLoaded부터 load까지의 기간을 줄이기 위한 것이다.  
다만 기술적으로 구현이 힘들다면 ssr만 사용하는 것도 방법이라고 했다. 버킷플레이스에서도 댓글창은 csr로 구현하고 있는 것 같다.

