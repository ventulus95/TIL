# 장고 다른 OS에서 환경설정 동일화하는 방식

예륻 들어서 Window에서 가상환경을 만들면 그걸 OSX에서는 사용할 수 없기때문에, 가상환경 자체를 OSX에서 만들고 그 다른 윈도우에서 적용된 환경설정을 그대로 받아오는 방식이 존재함.

`pip install -r requirements.txt` 이걸 통해서 OSX에서 가상환경을 만든 후에 거기에 다시 설치하는 방법을 택하면된다.

근데 이러면 번거로운 작업이 생기게됨. **\(단점!\)**

결국 window에서 안에 라이브러리를 추가하면 그걸 다시 받아서 window환경에서 깔아보고 로컬에서도 확인해보는 두번하는 과정이 필요.

즉, 이런 번거로움을 줄여주는 작업은 **Docker!**

Docker는 환경을 동일한 상태 즉, OSX든 Window든 간에 걍 상관없이 Docker Container에서 돌기때문에 환경변수가 바뀌는 것에 대해서는 걱정하지 않아도됨.

[참고 링크 장고 Docker](https://github.com/geusan/dockerfiles)

