# AWS CLI를 통해서 S3 파일 다운/업로드하는 방법



## CLI를 통해서 직접 S3에 접근하려면...

AWS CLI를 통해서 S3를 이용하려면 AWS CLI를 설치하고, AWS access key id, secret access key를 통해 등록하여 사용하면된다.

CLI를 접근하는 방식은 다루지 않겠지만,  s3 명령어 중에서도 소개할만한 것이 있어서 포스팅한다.



## S3 업로드 / 다운로드

```shell
aws s3 cp s3://~~~ s3 주소 /user/~~{현재 내 파일 디렉토리 주소}
```

예를들어서 s3:// -> /user/~~ 이런식이면 S3에서 현재 컴퓨터로 다운로드 하겠다는 뜻.

/user -> s3:// 이면 로컬 파일에서 s3로 파일 업로드 하는 개념

**--recursive** 명령어를 붙이면, 폴더 전체를 다운 받을 수도, 폴더 전체를 올릴수도 있습니다. 

```shell
aws s3 cp s3://~~~ s3 주소 /user/~~{현재 내 파일 디렉토리 주소} --recursive
```

와 같이 이용하면 됩니다. 



## S3 폴더나 현재 컴퓨터와 S3의 폴더의 파일 동기화

```shell
aws s3 sync s3://~~~ s3 주소 /user/~~{현재 내 파일 디렉토리 주소}
```

Sync를 통해서 폴더 안의 모든 파일을 동기화 할 수 있는데, 가장 좋은 점은 내 S3의 파일을 지우고 나면 실제로 현제 내 컴퓨터에서도  Sync를 통해서 그 파일을 직접지우지않아도 알아서 지워준다.

근데 Sync는 **--recursive** 명령어를 사용해야 폴더 전체를 동기화하는 것이 아닐까?

**정답을 이야기하자면, 필요가 없다.**

왜냐하면 이미 Sync에서는  --recursive를 작동하고 있어서 실제로 sync의 옵션을 주지 않아도 자동으로 폴더 안의 모든 파일이 동기화된다. 또한 Sync에서는 --recursive를 옵션조차도 없다.



출처: https://stackoverflow.com/questions/50874552/aws-s3-sync-recursive-not-working-in-windows

