# S3에 Static파일 저장 경로 잡기

실제로 Static file은 경로잡지 않고 EC2에 저장하는 방식을 이용할 수 도 있다. 물론, EC2에 저장해도 되지만, 그렇게 되면 EC2 용량이 초과하는 경우에는 감당할 수 없는 재앙이 일어날테니

실제로 용량만 많이 차지하고 서버랑은 1도 관련 없는 파일들은 다른 쪽으로 쭉 빼두는 것이 심신안정에 편할 것이다.

그러니까 S3로 경로를 잡아줘야하는데 좀 귀찮은 점이 많다.

## 1. S3 설정단계

버킷이 만들어져있는 상태에서 그 버킷에 권한을 설정해줘야하는 데, 권한 설정은 IAM의 USER ARN과 버킷 이름과 같은 설정을 넣어서 퍼블릭한 설정을 좀 추가하는 방식으로 만들어진다.

## 2. 장고 내부에서도 또 설정

그러니까 static 파일은 경로 설정을 따로 잡아줘야한다. 뭐 굳이 안그래도 되는 것 같긴한데, 어차피 가장 상위폴더 /media인가 /static가의 차이가 생기기 때문에, 반강제적인 처리가 필요하긴 하다.

참고: [https://blog.myungseokang.dev/posts/django-use-s3/](https://blog.myungseokang.dev/posts/django-use-s3/)
