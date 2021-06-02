# AWS Linux에서 FFMPEG 설치하기 \(ElasticBeanStalk\)

![MP4.1&#xC5D0;&#xC11C; AV1 &#xC9C0;&#xC6D0;&#xC73C;&#xB85C; &#xCD9C;&#xC2DC; &#xB41C; FFMPEG 4 - WebSetNet](https://websetnet.net/wp-content/uploads/2018/11/ffmpeg-4-1-released-with-av1-support-in-mp4.png)

이런 주제를 사용하게 된 이유도 사실 내가 이전에 했었던 프로젝트가 DeepLearning을 사용했다. 그리고 그 딥러닝을 Django 백엔드에서 딥러닝 모듈을 돌리는 터라, 이 딥러닝을 실행시키기 위해서는 FFMPEG이 필요했다.

문제는 FFMPEG이 AWS Linux에서는 일반적인 방법으로 설치가 재대로 안된다는 점이 큰 문제이다. 분명히 Red-hat 계열의 설치방법을 사용하기도 하고 Yum이나 Apt-get을 통해서 패키지 설치를 하고 싶었는데 기존 방법으로는 설치가 안됬다. 아마 Apt-get이나 Yum을 통한 링크가 이미 죽은것 같았다. 재대로 설치가 안되서 실행이 안됬기때문에 다른 방법을 찾아내야 했다.

실제로 나 같은 경우는 Django를 AWS Beanstalk을 통해서 배포하였기 때문에, 설치 방안을 EC2에 설치하는 방법으로만은 해결이 되지 않아서 AWS EB 방식으로 해결한 방법 두가지를 알려주려고 한다. 실제로 EC2에서 설치한 방법은 EC2에서는 잘 돌아갔지만 EB에서는 안돌아갔기 때문에 읊어본다.

## 1. EC2에서 FFMPEG 설치하는 방법

1. 루트 설정
2. bin으로 이동후
3. ffmpeg 폴더 설치후 이동

```text
1. sudo su -
2. cd /usr/local/bin
3. mkdir ffmpeg && cd ffmpeg
```

1. 직접 링크를 통해서 다운로드를 받는다. 자신의 OS 비트에 맞는 파일을 설치합니다. 물론 버젼별 설치도 가능합니다

   이전 버젼 릴리즈는 [https://johnvansickle.com/ffmpeg/old-releases/](https://johnvansickle.com/ffmpeg/old-releases/) 에서 설치가능하고,

   최신버젼은 [https://johnvansickle.com/ffmpeg/](https://johnvansickle.com/ffmpeg/) 에서 다운로드 가능합니다.

2. 다운로드한 파일을 압축해제합니다. 그리고 난뒤에 확인하면 실행이됩니다. 하지만, 이걸 어디서든 사용하기위해서는 ln으로 등록후에 사용해야합니다.
3. ln으로 등록한뒤에 ffmpeg을 어디서든 사용할 수 있습니다. 

```text
4. wget https://johnvansickle.com/ffmpeg/releases/ffmpeg-release-amd64-static.tar.xz
5. tar -xf ffmpeg-release-amd64-static.tar.xz -> ./ffmpeg -version
cp -a /usr/local/bin/ffmpeg/ffmpeg-4.2.1-amd64-static/ . /usr/local/bin/ffmpeg/
6. ln -s /usr/local/bin/ffmpeg/ffmpeg /usr/bin/ffmpeg
```

## 2. EB에서 FFMPEG 설치하는 방식

EB에서는 eb ssh을 통해서 설치하는 방식도 있긴한데, 실제로 EB에서 FFMPEG은 작동하지 않아서 다른 방법을 통해서 설치해야한다.

그방법은 `.config` 파일을 통해서 설치해야한다.

`.ebextensions/03_ffmpeg_package.config` Eb에서 실제 Deploy전에 AWS Linux를 통해서 할 명령어를 실행할 것들을 지정할 수 있습니다.

물론 ImageMagick을 왜까는지는 좀 의문이지만...? 이 방법이 제일 잘되는 방법이긴 해서 imagick과 같은 설치 명령어는 없어도 작동될 것 같습니다.

```text
packages:
  yum:
    ImageMagick: []
    ImageMagick-devel: []
commands:
  01-wget:
    command: "wget -O /tmp/ffmpeg.tar.xz https://www.johnvansickle.com/ffmpeg/old-releases/ffmpeg-3.4.2-64bit-static.tar.xz"
  02-mkdir:
    command: "if [ ! -d /opt/ffmpeg ] ; then mkdir -p /opt/ffmpeg; fi"
  03-tar:
    command: "tar xvf /tmp/ffmpeg.tar.xz -C /opt/ffmpeg"
  04-ln:
    command: "if [[ ! -f /usr/bin/ffmpeg ]] ; then ln -sf /opt/ffmpeg/ffmpeg-3.4.2-64bit-static/ffmpeg /usr/bin/ffmpeg; fi"
  05-ln:
    command: "if [[ ! -f /usr/bin/ffprobe ]] ; then ln -sf /opt/ffmpeg/ffmpeg-3.4.2-64bit-static/ffprobe /usr/bin/ffprobe; fi"
  06-pecl:
    command: "if [ `pecl list | grep imagick` ] ; then pecl install -f imagick; fi"
```

출처:

[https://medium.com/@maskaravivek/how-to-install-ffmpeg-on-ec2-running-amazon-linux-451e4a8e2694](https://medium.com/@maskaravivek/how-to-install-ffmpeg-on-ec2-running-amazon-linux-451e4a8e2694)

[https://stackoverflow.com/questions/39241654/how-to-install-ffmpeg-on-elastic-beanstalk](https://stackoverflow.com/questions/39241654/how-to-install-ffmpeg-on-elastic-beanstalk)

