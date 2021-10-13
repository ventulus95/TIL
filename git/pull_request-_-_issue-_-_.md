# Pull Request 올릴때 같은 Issue를 링크하는 법

## Pull Request를 올릴때 항상 하나의 문제점이 존재했느니...

이런 PR(Pull Request 이하 PR)를 올릴때는 대부분 Issue에서 파생된 문제를 해결하거나 Issue에 포함된 기능을 만들어서 올리곤 한다. 하지만 이 PR을 머지한 뒤에 Issue를 직접 닫아줘야하는 불편함이 있는데,

![스크린샷 2020-08-30 오후 12 23 02](https://user-images.githubusercontent.com/17822723/91650783-c51e3b00-eabe-11ea-9bc2-70a024dc3d6f.png)

매번 내가 닫아줘야하는 귀찮다 정말 상당히.... 이런 문제를 해결하는 방법이 있었다.

## Linked issues...?

이 Linked Issue가 정확하게 뭘 말하는 거냐면, **PR이 merge한 경우 자동으로 Issue까지 Closed되는 효과를 준다**. 즉, PR을 머지한 이후에 사용자가 직접 issue를 닫아줘야하는 수고를 줄일 수 있다는 것이다.

이 기능은 이렇게 사용할 수 있다. PR의 오른쪽하단에 보면 Linked issue라는 탭이 있다.

![스크린샷 2020-08-30 오후 12 24 58](https://user-images.githubusercontent.com/17822723/91650785-c6e7fe80-eabe-11ea-90b0-1578167af9e9.png)

하지만 PR을 올리고 난다음에는 버튼을 눌러 Issue를 직접 등록할수 있지만, PR을 생성한 시점에서는 이 버튼이 안눌린다. 도대체 왜 이렇게 됬나 싶었지만..

[https://docs.github.com/en/github/managing-your-work-on-github/linking-a-pull-request-to-an-issue](https://docs.github.com/en/github/managing-your-work-on-github/linking-a-pull-request-to-an-issue)

를 보면 이야기가 좀 달라진다.

## Github DOC에서는 이런식으로 소개한다.

* close
* closes
* closed
* fix
* fixes
* fixed
* resolve
* resolves
* resolved

다음과 같은 키워드와 Issue의 넘버 #{숫자}를 같이 기입하게 된다면, 자동으로 Issue와 함께 Linked된다.

구체적으로는 3가지를 정도의 방법으로 여러가지 이슈를 한번에 Tracking이 가능하다.

| 링크 이슈           | 문법                                            | 예시                                                             |
| --------------- | --------------------------------------------- | -------------------------------------------------------------- |
| 같은 레포지토리에 있는 이슈 | _KEYWORD_ #_ISSUE-NUMBER_                     | `Closes #10`                                                   |
| 다른 레포지토리에 있는 이슈 | _KEYWORD_ _OWNER_/_REPOSITORY_#_ISSUE-NUMBER_ | `Fixes octo-org/octo-repo#100`                                 |
| 여러개의 이슈.        | Use full syntax for each issue                | `Resolves #10, resolves #123, resolves octo-org/octo-repo#100` |

굳이 코드 해결을 한 개발자가 PR이 종료된 이후에 귀찮게 같은 번호의 Issue에가서 Closed Issue를 할 필요가 없어진다. 그냥 Comment에 다음과 같은 키워드와 issue 숫자를 붙혀주기만 하면 자동으로 연결해준다.

개발하다보면 짜잘한 이슈가 많아져서 issue가 너무 많아져서 정리해야할 필요가 생기는 경우가 종종 있는데, 그 경우를 조금이나마 더 편하게 해줄 수 있는 것이 Linked Issue라고 생각한다. 매우 편리한 기능이니 사용하면 좋을 것 같다.
