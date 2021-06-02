# Git pull request시 Conflict 발생시 처리하는 방법

pull request하면 거의 80%이상으로 Conflict가 발생한다. 그러면 잘 merge하기 위해서는 이 Conflict를 잘 해소해야하는데, 문제는 이걸 잘 해결하기위해서는 두가지 방식이 존재한다.

1. 웹 에디터로 고치기
2. 파이참이나 기타 IDE에서 다시 update해서 끌고 ide에 도움을 받는방식

2번이 단점은 고치고 난다음에 Pull request를 하는 방식을 취해야한다. 즉, 결국 pull request를 하고 Conflict발생 -&gt; update -&gt; ide 내부적으로 고침 -&gt; pull request 또보냄. 이런식의 절차를 걸쳐야한다는 것이다.

즉, 일이 또 발생하는 것.

그래서 편하지는 않지만 웹에디터로 고치면 바로 pull Request내부에서 고쳐지고 그걸 바로 Merge 할 수 있다.

