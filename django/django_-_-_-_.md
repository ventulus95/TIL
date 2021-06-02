# Django 파일 업로드시 안올라갈때 대처법

Django 파일 업로드시 분명히 model Form도 다 만들고 했는데 안올라가는 상황이 있다. ?? 왜 안되나했드니만

```markup
<form method="POST" class="post-form" enctype="multipart/form-data">
```

enctype에서 multipart/form-data를 하지 않았기 때문. 파일 저장이 안되서 실제로 보여줘야할때 재대로 보여줄 수 없었다 .

즉, 저 **enctype**을 추가하지 않은 경우 파일 자체는 넘어가지지 않는다.

