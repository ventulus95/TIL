# EB 설정법

사실 EB 설정법이라고 적었는데, 뻑나는 지점에서 이야기를 해보도록 하자.

설정법은 여기를 참고하자.

1. [https://woongsin94.tistory.com/117](https://woongsin94.tistory.com/117) 
2. [https://chohyeonkeun.github.io/2019/06/19/190619-django-elastic-beanstalk/](https://chohyeonkeun.github.io/2019/06/19/190619-django-elastic-beanstalk/)
3. [https://medium.com/@whj2013123218/django-%EC%9E%A5%EA%B3%A0-%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8-elastic-beanstalk%EB%A1%9C-%EC%84%9C%EB%B2%84%EC%97%90-%EC%89%BD%EA%B2%8C-%EB%B0%B0%ED%8F%AC%ED%95%98%EA%B8%B0-4be33ad4f64e](https://medium.com/@whj2013123218/django-장고-프로젝트-elastic-beanstalk로-서버에-쉽게-배포하기-4be33ad4f64e)

어쨌든 EB 설정하다가 뻑난 부분을 더 이야기해보자면,

## 1. Your WSGIPath refers to a file that does not exist.

이건 CLI 내부에서 나는 문제이다. wsgi.py를 재대로 못찾았다는 건데 그냥 wsgi.py가 덮혀있는 폴더가 있는지 부터 파악하자. config.py로 덮어놨다면 config/wsgi.py로 설정만 해주면 된다.

## 2. 502 BadGATE WAY -&gt; nginX

이게 가장 큰 문제를 만들었는데, 이건 어디서 손을 봐야하는 문제인지도 몰라서 엄청 해매고 다녔다. 특히 진짜 어디서부터 만져야할지 이게 뭔가 싶을정도로 좀 얼을 많이 탔다.

아는 분을 통해서 확인한 결과 EB 환경 새팅시 \(eb init 시\) 파이썬 세팅을 잡아줄때 환경을 **3.7로 잡아주지말고 3.6**으로 잡아줘야한다. 정확하게 아는 것은 아니라 모호하게 이야기할 수 있지만, python 세팅을 3.7로 해주면 nginx로 잡히는게 아니라 gunicorn으로 잡히기 때문에, 설정자체를 엄한걸로 잡아서 망한다.

3.6으로 잡아주면 문제는 해결된다.

## 3.  Internal Server Error

이건 에러보면서 잡으면된다.

eb logs를 통해서 직접 처리하는 방식이 있는데, 이러면 꽤나 눈알 빠지게 에러를 찾으러 다니는 방식이여야한다.\(즉, 내가 엄청 찾는 방식\)

혹은 AWS 내부의 로그를 통해서 가장 최신 100줄만 불러와서 보는 방식이 존재한다. 물론 그것도 귀찮기는 매한가지

그래서 **sentry**를 활용하는것이 꽤 좋은데, 이런 오류들을 오류 원인과 함께 어디서 문제가 발생했는지 추적해주는 기능을 제공해서 꽤 잘 쓰는 중이다.

