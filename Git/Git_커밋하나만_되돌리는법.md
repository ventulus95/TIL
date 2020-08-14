# Git 커밋 하나만 되돌리는 법

협업을 하다가 보면, branch별로 git을 관리한다. 이번 프로젝트 도중에 기능을 삭제했는데 기능을 다시 되돌려야하는 경우가 발생했다. 문제는 이미 몇개의 커밋을 푸쉬해버린 상황이었다. 그나마 상황이 괜찮았던 것은 내가 Commit을 기능삭제한 부분을 따로 커밋을 했다는 것이다. ~~이럴줄알고~~ 그러면 **이 기능 삭제 커밋만 빼고 ** 나머지 커밋을 살리는 방법은 뭘까?

Commit을 되돌리기위해서는 여러가지 방법이 있으나,  검색해본 결과 깃 리베이스를 사용하여 해결했다.

```shell
git rebase -i HEAD^^ or
git rebase -i <커밋hash값>
```

git Rebase를 통해서 이전의 커밋을 통해서 필요한 부분을 처리할 수 있는 방법이 존재한다.

<img width="809" alt="스크린샷 2020-08-14 오후 1 23 55" src="https://user-images.githubusercontent.com/17822723/90216132-b5ed8b00-de38-11ea-8be3-529262446fc2.png">

다음과 같은 방법으로 이 커밋을 살릴지 죽일지를 git CLI를 통해서 결정을 할 수 있고 실제로 이 커밋은 최신순이 아닌 역순으로 배열되어 한개씩 한개씩 체크하면서 넘어갈 수 있다.

지금 내가 처한 경우는 **기능 삭제 커밋을 지우게되면 삭제된 파일을 다시 되살리는 것이다.**  그래서 그 커밋만 빼고 나머지 커밋은 유지된 상태로 돌아오게 된다.

그러면 이대로 PUSH!하면 될줄 알았는데.... 또 문제가 발생한다.  이 Push 방향이 master branch로 Push가 되는 것이다.  분명 내가 쓰던 feature/000 브랜치로 push하고 싶은데 그게 안된다...
왜 이런 상황이 발생했는지를 알아보면 git branch를 보면 이런 문구가 적혀있다.

`Detached HEAD` 

이런 상황에서는 github Desktop에서는 재대로 어떤 작업도 할 수 없기때문에 CLI를 이용해서 해결해야했다.



> **참고 사항!** 
>
> 왜 `Detached HEAD` 상태가 되었는가에 대해서 생각해봤는데 실제로 내가 이 커밋을 돌릴때는 commit의 주소 즉 hash값을 통해서 돌렸다.
>
> 이 방식은 비슷하게도 checkout에서도 커밋의 해쉬로 실행하게되면  `Detached HEAD` 상태가 된다고 한다. 즉, commit의 주소를 통해서 rebase를 하거나 checkout을 하게된 경우는 발생할 수 있는 부분이다.

CLI를 통해서 `detached Head` 된 branch를 다시 원래 branch에 붙이고 싶다면  커밋한 후에 

```shell
git checkout -b <branch> ex)feature/0000 요롷게?
```

하게된다면 실제로 branch가 생성되어 github에 생성이 될것이다. `checkout -b` 옵션은 새로운 branch를 생성하는 옵션이다 

즉, 이 새롭게 생성된 branch를 바탕으로 원래 branch에 붙이는 방식을 택하거나 혹은 아예  Develop branch에 merge하는 방법을 택해도 된다. 이미 rebase된 상태라 이전 feature/000에서 수정되어진 값은 유지된 상태이기때문에 큰 걱정은 안해도된다.

그렇게 두개의 branch가 생기면  이렇게 차이가 발생하는 것을 볼 수 있다.

![캡처](https://user-images.githubusercontent.com/17822723/90216128-b25a0400-de38-11ea-82fd-eb6431c217f4.jpg)





출처:https://superuser.com/questions/35267/how-can-i-roll-back-1-commit

https://okky.kr/article/440421

http://www.devkuma.com/books/pages/433

