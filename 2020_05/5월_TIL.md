# [2020.05](5월_TIL.md)

# INDEX

* [5월 15일](#15일)
* [5월 16일](#16일)
* [5월 18일](#18일)



* ## 15일

  1. ## 장고 다른 OS에서 환경설정 동일화하는 방식
  
     예륻 들어서 Window에서 가상환경을 만들면 그걸 OSX에서는 사용할 수 없기때문에, 가상환경 자체를 OSX에서 만들고 그 다른 윈도우에서 적용된 환경설정을 그대로 받아오는 방식이 존재함.
  
     `pip install -r requirements.txt` 이걸 통해서 OSX에서 가상환경을 만든 후에 거기에 다시 설치하는 방법을 택하면된다.
  
     근데 이러면 번거로운 작업이 생기게됨. **(단점!)**
  
     결국 window에서 안에 라이브러리를 추가하면 그걸 다시 받아서 window환경에서 깔아보고 로컬에서도 확인해보는 두번하는 과정이 필요.
  
     즉, 이런 번거로움을 줄여주는 작업은 **Docker!**
  
     Docker는 환경을 동일한 상태 즉, OSX든 Window든 간에 걍 상관없이 Docker Container에서 돌기때문에 환경변수가 바뀌는 것에 대해서는 걱정하지 않아도됨. 
  
     [참고 링크 장고 Docker](https://github.com/geusan/dockerfiles)
  
     
  
  2. ## Django를 pycharm에서 적용하는 방식
  
     ![스크린샷 2020-05-15 오후 2 37 25](https://user-images.githubusercontent.com/17822723/82040171-2291a600-96e1-11ea-9bfa-3f1128242708.png)
     깃에서 바로 적용하면 interpreter가 없다는 식으로 말해버리는데
  
     이때는 project interpreter를 통해서 가상 환경을 만든 폴더를 적용하는 방식 .
  
     ![스크린샷 2020-05-15 오후 8 39 50](https://user-images.githubusercontent.com/17822723/82046651-3f7fa680-96ec-11ea-9d36-b9aa844ac891.png)
  
     여기서 가상환경을 등록하면 알아서 run configuration에서 잡아준다. 
  
     ![스크린샷 2020-05-15 오후 2 59 09](https://user-images.githubusercontent.com/17822723/82040169-21607900-96e1-11ea-94ae-47e33562db7b.png)
  
     근데 pycharm CE버젼에서는 django Run이 없기때문에 Python으로 돌리는 방식말고는 없다. 
  
     https://blueshw.github.io/2016/02/02/django-setting-for-pycharm-community/
  
     여기 확인해보면 쉽게 이해 할수 있다. 그냥 Run하는 명령어와 파일을 파이썬으로 실행시키는 방식임.
  
     
  
  3. ## Git pull request시 Conflict 발생시 처리하는 방법
  
  pull request하면 거의 80%이상으로 Conflict가 발생한다. 그러면 잘 merge하기 위해서는 이 Conflict를 잘 해소해야하는데, 문제는 이걸 잘 해결하기위해서는 두가지 방식이 존재한다.
  
  1. 웹 에디터로 고치기
  2. 파이참이나 기타 IDE에서 다시 update해서 끌고 ide에 도움을 받는방식
  
  2번이 단점은 고치고 난다음에 Pull request를 하는 방식을 취해야한다. 즉, 결국 pull request를 하고 Conflict발생 -> update -> ide 내부적으로 고침 -> pull request 또보냄. 이런식의 절차를 걸쳐야한다는 것이다. 
  
  즉, 일이 또 발생하는 것.
  
  그래서 편하지는 않지만 웹에디터로 고치면 바로 pull Request내부에서 고쳐지고 그걸 바로 Merge 할 수 있다. 



-------



* ## 16일

  1. ### 장고 URL html 템플릿 URL 설정시 참고사항

     ```html
     <h1><a href="{% url 'board:post_detail' pk=post.pk %}">{{ post.title }}</a></h1>
     ```

     `board:post_detail` 이부분 대충 보면 url -> 의 주소로 이동하겠다는 뜻인데 이게 먹힐려면 장고걸스 듀토리얼에는 요런식으로 적혀져있다. 

     

     ```html
     <h1><a href="{% url 'post_detail' pk=post.pk %}">{{ post.title }}</a></h1>
     ```

     문제는 이러면 재대로 출력이 안되고 reverse문제가 발생하는데, 이 url.py에서 post_detail의 를 재대로 찾아주지 못해서 발생하는 문제 같았다.

     어디에서 이 url을 찾아야하는지 못하는 문제가 발생하므로 

     앞에 이함수를 들고 있을 범직한 곳으로 가서 전달해줘야하는 문제가 발생하는 것같다. 즉, 그 app의 이름 : url의 def이런식의 구조를 만들어놔야지 되는 듯했다. 

     

     ### :airplane: 결론

     URL 규칙은 url,py의 name ='~~' 의 String값을 알아서 mapping시켜주는 것 같다.

     ```python
     path('main/',HomeView.as_view(),name='home'),
     혹은
     path('', views.post_list, name='post_list'),
     ```

     단, board폴더에 있는 url.py의 경우는 `board:post_detail`이런식으로 적어야지 인식했고 `config`에 있는 `home`의 경우는 다르게 그냥 적어도 바로 인식했다. 즉 앞의 :는 폴더를 뒤의 url은 url.py에 설정해준 url값 인듯하다.



-----

* ## 18일

  1. ### Django 파일 업로드시 안올라갈때 대처법

     Django 파일 업로드시 분명히 model Form도 다 만들고 했는데 안올라가는 상황이 있다. ?? 왜 안되나했드니만

     ```html
     <form method="POST" class="post-form" enctype="multipart/form-data">
     ```

     enctype에서 multipart/form-data를 하지 않았기 때문. 파일 저장이 안되서 실제로 보여줘야할때 재대로 보여줄 수 없었다 .

     즉, 저 **enctype**을 추가하지 않은 경우 파일 자체는 넘어가지지 않는다. 

     

  2. ### 특정 확장자만 파일 업로드 가능하게 만들기 (mp3)

     https://stackoverflow.com/questions/3648421/only-accept-a-certain-file-type-in-filefield-server-side

     

     ```python
     from django.core.validators import FileExtensionValidator
     from django.db import models
     
     class MyModel(models.Model):
         pdf_file = models.FileField(upload_to='foo/',
                                     validators=[FileExtensionValidator(allowed_extensions=['pdf'])])
     ```

     간단하게 vaildator를 통해서 처리하는 방식이 가장 이상적이다.  다른 방식은 아예 validator를 함수화하거나 모듈화해서 그걸 끌어와서 해결하는 방식

     링크에서도 확인했지만 위의 **코드는 안전하지 않다는 것**이 문젠데 즉, 이렇게 유효성 검사를 하면 실제로 파일 뒤의 글자 .mp3만되면 통과가 되기 때문. 

     근데 솔직히 말하면, front에서 처리하는 게  더 직관적인 것 같음. submit후 에 안된다고 하면 좀 열받을 듯.

     

  3. ### 원격 저장소에 올라간 커밋 날리기

     일단 2가지 방식이 존재 

     1. git reset 

        깃 리셋은 애초 커밋 이전으로 돌려버리겠다는 것이다. 특히 --HARD로 하면 아에 완전히 다 날려버리고 commit없에는 방식인데, 이러면 기존 커밋 내역이 다날라간다. 즉, 외부인과의 협업을 하는 경우 비추

        push시에는 -f 옵션 추가해서 강제 푸쉬해야함.

     2. git revert

        특정 커밋을 돌리는 것도 commit에 만들어버리는 것. 

     

     참고: https://jupiny.com/2019/03/19/revert-commits-in-remote-repository/

     

     

  4. ### favicon.ico 404 Not found

     ```html
     <link rel="icon" href="data:;base64,iVBORw0KGgo=">
     ```

     webBean에 발생하는 오류라는데 jetBrain 계열의 IDE 오류인 듯하다. 

