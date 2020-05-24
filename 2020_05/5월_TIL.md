# [2020.05](5월_TIL.md)

# INDEX

* [5월 15일](#15일)
* [5월 16일](#16일)
* [5월 18일](#18일)
* [5월 24일](#24일)



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

----

- ## 24일

  1. ### 장고 테이블 지우고 난 후 다시 마이그레이션 하는 방법!

     장고에서 DB table이 꼬이면, ㄹㅇ 개노답인 상황이 생긴다. 특히, 어디서부터 table이 생긴건지, 어디서부터 없는건지 파악할 수 없기때문에 진짜 말그대로 sqlite3에 직접들어가서 확인해보는 것과 같은 방식으로 찾는 것도 있다. 

     그리고 장고 자체 버그인지 아니면 내가 장고에 대한 이해도가 떨어지는지는 모르겠는데 modeling을 바꾸고 난다음 python manage.py makemigrations 과 python manage.py migrate 후에 modeling이 재대로 적용되지 않는 이상한 버그가 있는 듯하다. 

     특히, migration을 날리고 난다음에 table은 있는 경우 오히려 테이블은 이전 모델이라 재대로 key가 적용이 안되는 이상한... 별의 별 상황을 다 겪었다. 

     즉, 요약해보자면 

     - modeling이 변경되면 migration이 잘 변경됨 
     - 하지만 가끔 migration을 그냥 물리적으로 파일을 없에버리면, makemigrations 혹은 migrate을 통해서 직접 테이블이 변경되지 않는 이상한 버그가 있다. 
     - 또 거기에 직접 SQLite3에 들어가서 테이블을 삭제하고 다시 migrate작업을 해도 다시 테이블이 생성되지 않아서 난감.

     

     이런 버그를 해결하는 방법은 다음과 같다.

     

     1. 해당 앱에 있는 migrations 폴더안의  __init__.py 파일을 제외하고 모두 삭제하기!

     2. database에 직접 접속하여 테이블 중 django_migrations 라는 테이블에서 해당 앱에 대한 raw를 삭제.

     `DELETE FROM django_migrations WHERE app = '앱 이름'`

     3. python3 manage.py makemigrations // python3 manage.py migrate 수행 하면 해결

     

     참고: https://forybm.tistory.com/2 

     

  2. ### 장고에서 Migrate란?

     이런 근본적인 문제를 해결하기위해서는 migrate가 뭔가에 대해서 이해를 하는 것이 중요할 것 같다.

     1. 마이그레이션 파일 (초안) 생성하기 : `makemigrations`
     2. 해당 마이그레이션 파일을 DB에 반영하기 : `migrate`

     예상보다는 조금 심플한 모습을 가지고 있다. 

     

     쉽게 이야기하면 장고에서 model을 만들면 -> `makemigrations`를 통해서 일단 SQL문으로 table화를  어떤 필드를가지고 table을 만들지 혹은 어떤 필드가 변경 됬는지 삭제 되었는지를 저장해놓는 파일이라고 보면 된다. 

     -> `migrate`의 경우 이 migrations를 통해 만들어진 파일을 바탕으로 DB에 직접적으로 SQL문으로 자동으로 변환해주는 시스템이다.

     

     1번 같은 위의 같은 문제 (no such table, column 등의 오류) 로 실제 DB를 날리거나 할 것이 아니면 여러가지 방법을 강구해야 하는데, 찾아보는 블로그에 의하면 migration오류라고 했다. 

     

     이런 오류들을 확인하는 방법은 결국 DB에서 직접 확인하던가 혹은 migration 확인하는 방법으로 처리하는게 제일 좋아보였다. (오늘 해결할때 가장 원인을 잘 찾았던 것도 결국에는 `showmigrations` 로 어디까지 모델링 되어있는지 확인하는 방식으로 처리했었다.)

     

     참고: https://wayhome25.github.io/django/2017/03/20/django-ep6-migrations/

     

  3. ### 장고 One To One modeling 하는 방식 (장고 기본 User 확장하는 방법)

     그 어떤 블로그보다 이 영상 한 편이 훨씬 더 유익했다. 특히나 Forms.py로 기본 User 모델도 이용하고, views.py를 훨씬 코드량을 줄일 수 있었기 때문에, 개인적으로는 훨씬 좋았다. 대부분의 One to One Field를 알려주는 블로그는 내 모델에서는 재대로 적용이 안됬다는 것도 컸다.

     차후에 블로그에 정리해서 올려보는 것도 괜찮아 보인다.

     https://www.youtube.com/watch?time_continue=533&v=Tja4I_rgspI&feature=emb_title

     이 영상에서 사용하는 방식을 간단하게 요약해보자면, 대략 이렇다.

     1. Admin에 이 앱을 등록한다. 
     2. Model 만들기 one to one으로 user연결
     3. Forms.py만들기 특이한건 User extradata도  Form으로 확장시킴. 
     4. views.py에 Form등록. 
     5. html에 template를 통해서 form 등록.

     

  4. ### 장고 Forms.py input type 변경

     결론은 field 내부의 widget 매개변수를 통해서 type을 스스로 정할 수 있다. 

     ```python
     FRUIT_CHOICES= [
         ('orange', 'Orange s'),
         ('cantaloupe', 'Cantaloupes'),
         ('mango', 'Mangoes'),
         ('honeydew', 'Honeydews'),
         ]
     
     class UserForm(forms.Form):
         first_name= forms.CharField(max_length=100)
         last_name= forms.CharField(max_length=100)
         email= forms.EmailField()
         age= forms.IntegerField()
         favorite_fruit= forms.CharField(label='What is your favorite fruit?', widget=forms.RadioSelect(choices=FRUIT_CHOICES))
     ```

     아래 favorite_fuit변수처럼 forms의 CharField내부의 매개변수로 input type을 변경할 수 있다. 지금 예시는 radio 타입

     장고 widget 공식 문서 링크: https://docs.djangoproject.com/en/3.0/ref/forms/widgets/

  

  

  

  