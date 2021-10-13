# 장고 One To One modeling 하는 방식 (장고 기본 User 확장하는 방법)

그 어떤 블로그보다 이 영상 한 편이 훨씬 더 유익했다. 특히나 Forms.py로 기본 User 모델도 이용하고, views.py를 훨씬 코드량을 줄일 수 있었기 때문에, 개인적으로는 훨씬 좋았다. 대부분의 One to One Field를 알려주는 블로그는 내 모델에서는 재대로 적용이 안됬다는 것도 컸다.

차후에 블로그에 정리해서 올려보는 것도 괜찮아 보인다.

[https://www.youtube.com/watch?time_continue=533\&v=Tja4I_rgspI\&feature=emb_title](https://www.youtube.com/watch?time_continue=533\&v=Tja4I_rgspI\&feature=emb_title)

이 영상에서 사용하는 방식을 간단하게 요약해보자면, 대략 이렇다.

1. Admin에 이 앱을 등록한다. 
2. Model 만들기 one to one으로 user연결
3. Forms.py만들기 특이한건 User extradata도  Form으로 확장시킴. 
4. views.py에 Form등록. 
5. html에 template를 통해서 form 등록.
