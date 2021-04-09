# 장고에 file 이동 위치 S3로 경로 설정

## 1. 옮기려면 일단 IAM의 AWS\_ACCESS\_KEY\_ID가 필요.

```text
AWS_ACCESS_KEY_ID = 'YOUR_ACCESS_KEY_HERE'
AWS_SECRET_ACCESS_KEY = 'YOUR_SECRET_ACCESS_KEY_HERE'
```

접근 권한을 위해서 이 키 ID와 ACCESS KEY값이 필요하다. 노출되지 않기위해서 주의하고 그 노출시키지 않는 방식은 여러가지 방식이 있을수 있다.

## 2. 라이브러리를 통해서 세팅을 해두자.

```text
pip install boto3
pip install django-storages
```

위의 두가지 라이브러리를 설치하는 이유는

```text
DEFAULT_FILE_STORAGE = 'storages.backends.s3boto3.S3Boto3Storage'
AWS_STORAGE_BUCKET_NAME = 'insert-your-bucket-name-here'
AWS_S3_REGION_NAME = 'eu-west-2'
```

이것만 Setting.py에 있게되면, s3로 직접적으로 파일이 이동하게 된다.

경로 설정만 잘해줘도 바로 s3 자신의 버킷으로 이동될 것이다.

[https://blog.theodo.com/2019/07/aws-s3-upload-django/](https://blog.theodo.com/2019/07/aws-s3-upload-django/)

