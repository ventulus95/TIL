# github actions를 통한 React의 build 오류 처리와 yarn install 캐쉬 처리

## 배포 작업은 귀찮아...

현재 진행하는 프로젝트에서 Front-end서버는 React로 다른 친구가 작업하고 있고, 나는 실제로 배포와 백엔드 서버와 같이 진행하고 있었다. 문제는 React를 다루는 친구가 배포에 대해서는 잘 모른다는 것이었는데, 나는 실제 서버에 올려보고 싶었다.

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbLEenS%2FbtqNXGlAf9y%2FJDQdp1t7kCHsQclMgrmkIK%2Fimg.png)

하지만 내 귀여운 t2.mirco는 백엔드도 키면서 yarn build를 돌리기에는 1시간정도 걸릴것만 같아서 외부에서 build를하고 차라리 파일을 가지고 오는게 훨씬 좋다고 판단했다. 근데 일일히 내 컴퓨터에서 굳이 친구가 commit올릴때마다 git pull로 다운 받아서 그걸 build를 해야하나 고민하다가 좋은 아이디어가 떠올랐다.



## 깃허브 액션을 통해서 React build를 해보았다.



일단 이 react를 build 후 파일 이동은 S3를 통해 실제 EC2에 올릴 생각이었는데, 이 과정을 생각해보니까 CI 툴을 이용해서 처리하는 게 좋지않나? 라는 생각이 머리에 스쳤고, 직접 해보았다.

github action을 통한 React app 배포는 타 블로그에 잘 작성이 되어있으니 스킵하도록하고 실제로 내가 발생한 오류에 대해서 설명을 해보겠다.

https://velog.io/@loakick/Github-Action-AWS-S3%EC%97%90-React-%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8-%EB%B0%B0%ED%8F%AC%ED%95%98%EA%B8%B0

(너무 친절하심 정말 큰 도움이 되었다.)

내가 발생한 오류는 build 자체가 안되는 오류였다. 실제로 local PC에서는 잘 됬는데 action에서는 오류가 발생했다. 왜인가를 확인해봤는데 공통적으로 다음과 같은 오류를 뱉어냈다. 

`Treating warnings as errors because process.env.CI = true.`

`Most CI servers set it automatically.`

`Failed to compile.`

저 process.env.CI 때문에 build 자체가 안됬는데, 추측하건데 Process.env.CI가 true로 되어있어서 warning 로그를 모두 error로 취급해서 build가 안되었다. 

그러면 이것을 어떤식으로 해결할까?

```yaml
run: yarn build
	env:
  	CI: ""	
```

이렇게만 해결해줘도 build문제를 해결할 수 있다. 

출쳐: https://github.community/t/treating-warnings-as-errors-because-process-env-ci-true/18032

## yarn install 캐쉬 처리

CI 작업을 하면 결국에 install 작업이 반드시 필요하다. 그러다보면 이 install에 걸리는 시간이 많이 소요되는 것을 알 수 있는데, 그러한 작업의 속도를 줄이기 위해서는 캐싱작업을 미리 땡겨놓는 것이 좋다.

```yaml
- name: Get yarn cache directory path
  id: yarn-cache-dir-path
  run: echo "::set-output name=dir::$(yarn cache dir)"

- uses: actions/cache@v1
  id: yarn-cache # use this to check for `cache-hit` (`steps.yarn-cache.outputs.cache-hit != 'true'`)
  with:
    path: ${{ steps.yarn-cache-dir-path.outputs.dir }}
    key: ${{ runner.os }}-yarn-${{ hashFiles('**/yarn.lock') }}
    restore-keys: |
      ${{ runner.os }}-yarn-
```

다음과 같이 yarn install도 캐싱할 수 있다.



출처: https://github.community/t/how-to-setup-yarn-install-cache-on-github-action/18138