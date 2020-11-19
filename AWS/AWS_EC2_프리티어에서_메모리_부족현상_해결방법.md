# AWS EC2 프리티어에서 메모리 부족현상 해결방법

![https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fb3DDiw%2FbtqNNh8z0Sk%2FgA0Bh7lBTwE4KXBHDkSIwk%2Fimg.png](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fb3DDiw%2FbtqNNh8z0Sk%2FgA0Bh7lBTwE4KXBHDkSIwk%2Fimg.png)



## AWS free tier를 사용하다보면 2%가 부족할 때가 있다.

AWS 프리티어는 가난한 대학생에게는 한줄기 빛과 같은 존재인데, AWS의 프리티어라서 적게 돈이 나가는 것도 좋고, 실제로 이것저것 해볼 수 있다는 측면에서 한줄기의 빛과 같은 존재이다.

하지만, 이러한 프리티어도 한가지의 문제를 가지고 있다. 

t2.micro의 램이 1GB정도 밖에 안된다는 것인데, 여러 가지의 프로젝트를 동시에 돌리는 것에 엄청나게 문제를 준다는 것이 가장큰 문제일 것이다. 

나의 사례로만 들어도 Spring boot 한개를 킨 상태에서 Spring boot의 gradle을 통한 빌드 작업을 시도 해봤는데, 서버의 가용성이 폭발해버린 사례가 있었다.

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FdCYrB2%2FbtqNQwjFbYP%2Fb6BybwnR70XXFgcR2VRuk0%2Fimg.png)

이때는 마치 SSH가 실제로 가용중인 Gradle을 종료시키지도 못했고 SSH 터미널을 끄고 나가도 재접해도 연결이 한동안 안되서 좀 많이 당황했었다.



## 그럼 이런 문제를 해결할 방법은 없는가?

있으니까 이 블로그 포스팅을 한것이다.나도 역시 아는 동생에게 의견을 들어서 생각하지도 못했고, 다른 사람들이 비슷한 사례에 어떤식으로 대처해야하는지에 대해서 알았으면 좋겠다는 의미에서 이야기를 꺼냈다. 

나처럼 두개 이상의 프로젝트를 한 EC2안에 돌려야하는 상황이라면, (그게 뭐 돈을 저렴하게 사용하게 되었든, 여러가지 프로젝트를 동시에 돌려보고 싶든간에) 이런 방법을 추천해본다. 

리눅스에서는 SWAP 메모리를 지정할 수 있다. 이 SWAP 메모리란 무엇이냐하면, RAM이 부족할 경우가 있으므로 HDD의 일정공간을 마치  RAM처럼 사용하는 것이다.  그래서 우리는 반 강제적으로 RAM을 증설한 듯한 효과를 만들수 있다.

 AWS에서는 다음과 같이 RAM을 증설해야하는 경우 이렇게 추천해준다. 

**스왑 공간 크기 계산**

일반적으로 다음과 같이 스왑 공간을 계산합니다.

| **물리적 RAM의 양**     | **권장 스왑 공간**        |
| ----------------------- | ------------------------- |
| RAM 2GB 이하            | RAM 용량의 2배(최소 32MB) |
| RAM 2GB 초과, 32GB 미만 | 4GB + (RAM – 2GB)         |
| RAM 32GB 이상           | RAM 용량의 1배            |

**참고:** 스왑 공간은 절대로 32MB 미만이 되지 않아야 합니다.



그러면 어쨌든 EC2의 free tier를 통해서 RAM을 획득할 수 있는 것은 1GB이므로, 우리는 2GB정도로 생각하고 잡으면 된다.



## 실제 해결 방법 

공식 홈페이지에서도 제공하지만, 블로그에서도 한번 더 소개하겠다. 

참고로 메모리의 상태를 확인해볼수 있는 명령어는 free이다.

출처: https://aws.amazon.com/ko/premiumsupport/knowledge-center/ec2-memory-swap-file/

1. 일단 dd 명령어를 통해 swap 메모리를 할당한다.

```shell
sudo dd if=/dev/zero of=/swapfile bs=128M count=16
```

128씩 16개의 공간을 만드는 것이여서 우리의 경우 count를 16으로 할당하는 것이 좋다. 즉, 2GB정도 차지하는 것이다.

2. 스왑 파일에 대한 읽기 및 쓰기 권한을 업데이트합니다.

```plainText
$ sudo chmod 600 /swapfile
```

3. Linux 스왑 영역을 설정합니다.

```plainText
$ sudo mkswap /swapfile
```

4. 스왑 공간에 스왑 파일을 추가하여 스왑 파일을 즉시 사용할 수 있도록 만듭니다.  

```plainText
$ sudo swapon /swapfile
```

5. 절차가 성공했는지 확인합니다.

```plainText
$ sudo swapon -s
```

6. **/etc/fstab** 파일을 편집하여 부팅 시 스왑 파일을 활성화합니다.

편집기에서 파일을 엽니다.

```plainText
$ sudo vi /etc/fstab
```

파일 끝에 다음 줄을 새로 추가하고 파일을 저장한 다음 종료합니다.

```plainText
/swapfile swap swap defaults 0 0
```

다음과 같이 적용됬는지 확인을 해봅니다.

```shell
             total       used       free     shared    buffers     cached
Mem:       1009136     941624      67512          4      11696      96836
-/+ buffers/cache:     833092     176044
Swap:      2097148     304972    1792176
```

적어도 이렇게하면 삐걱삐걱 돌아가기는 합니다. 물론 원래 RAM에 비해서는 속도가 현저히 낮아집니다. HDD에 할당하기때문에, RAM이랑 비교하기에는 쨉도 안되는 속도지만...

그래도 느리지만 돌아가야하는 서비스라면, (그리고 저희에 돈을 위해서라면... 눈물..ㅠ ) 이렇게라도 사용한다면, 완전히 뻗는 경우는 적어질 겁니다. 



