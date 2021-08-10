rebase
======
rebase를 남발하면 안된다고 생각하지만, 어떤 프로젝트에 있는지, 어떤 커밋들을 헀는지에 따라 rebase가 필요한 상황이 올 것이기 때문에 정리의 필요성을 느꼈다.  

git rebase : 말 그대로 커밋한 내용을 rebase하는 것이다. base를 다시 잡는다고 생각하면 편하다. 


rebase에 여러 옵션이 있지만 자주 사용할 것 같은 옵션만 적겠다.  


1.
git rebase -i HEAD~2 는 헤드가 가리키는 커밋으로부터 2개의 커밋을 리베이스하겠다는 뜻이다.   
-i는 interactive옵션으로 터미널에서 대화형식으로 어떻게 rebase할 지를 정할 수 있다.  
자주 사용할 옵션은 squash로 s로 줄여서 사용해도 된다.   

    pick 7c65355 Task 1/3
    pick 2639543 Task 2/3
    pick d442427 Task 3/3  

위 세줄을 아래처럼 바꾸면 한 개의 커밋으로 합쳐진다.  

    pick 7c65355 Task 1/3
    squash 2639543 Task 2/3
    squash d442427 Task 3/3


2.
git pull이 git fetch + merge라면  
git pull -rebase는 git fetch + rebase이다.  
git pull –rebase master : 피쳐 브랜치 내부에서 이 명령어를 입력하면 master로 rebase된다.

그림으로 설명하면   
![rebase1](https://user-images.githubusercontent.com/55550753/128602510-c9fe9b65-acf1-4705-aa47-e3ecdd1a1949.PNG)  

위 상태에서 rebase하게 되면 아래 그림처럼 된다.  
![rebase2](https://user-images.githubusercontent.com/55550753/128602540-1d1eeb23-71db-4edd-aac1-8c3fc1666481.PNG)  

이 상태에서 merge하게 되면 아래 그림처럼 된다.  
![rebase3](https://user-images.githubusercontent.com/55550753/128602574-95fec4c6-4ca3-47be-a027-3661117ddcd8.PNG)  

또한 merge할 때 fast-forward관계에 대해서 알아야 할 필요가 있다.  
간단하게 말하면 커밋 a히스토리에 커밋 b히스토리가 포함되어 있는 것이다. 

fast-forward 참고 링크 : https://koreabigname.tistory.com/65  
git-flow 참고 링크 : https://techblog.woowahan.com/2553/