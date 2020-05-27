# 장고 URL html 템플릿 URL 설정시 참고사항

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