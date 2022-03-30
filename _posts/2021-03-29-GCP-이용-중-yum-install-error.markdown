---
layout: post	
title: "GCP 이용 중 yum install error"
date: 2021-03-29
categories:
  - Trouble-Shooting
description:
image: https://res.cloudinary.com/danhdvla9/image/upload/v1615973205/Thumbnails/Troubleshooting-Icon_ou9s2b.png
image-sm: https://res.cloudinary.com/danhdvla9/image/upload/v1615973205/Thumbnails/Troubleshooting-Icon_ou9s2b.png
image-me: https://res.cloudinary.com/danhdvla9/image/upload/v1614694302/Blacksmith_vqd5bz.png
---

### Issue

<br>
<br> 

Google Cloud Platform에서 Compute Engine 서비스로 CentOS 7 이용 중 yum install error가 발생하여 설치가 안되는 상황. 

<br>

![error](https://res.cloudinary.com/danhdvla9/image/upload/v1616996607/ScreenShots/21-03-29/1_bqgqeu.png)
**yum signature could not be verified**로 통칭되는 오류이다. 

<br>
<br>

### Steps to reproduce the issue 

<br>
<br>

그저 GCP나 AWS Instance를 LinuxOS로 생성하고 yum, apt 등의 패키지 매니저를 통해 install 하려하면 등장한다.

stackoverflow에서 같은 오류를 겪은 사람이 어떻게 해결했나 보러 다녔는데, 항상 발생하는 것은 아니고 가끔씩 gpg key에 의해 발생하는 오류로 추정된다고 하더라.

GCP나 AWS Linux를 이용하다보면 드물게 발생한다는데.... 
~~이런건 왜 나만 걸리는 걸까~~

<br>
<br> 

### TroubleShooting

<br>
<br>

처음 접근은 에러메세지에서 추천하는 방법 중 `--disablerepo` 옵션을 사용한 방법을 실행하였지만 같은 오류의 반복이었다. 

*이 부분에서 에러메세지를 자세히 보지 않고 간과한 것이 빙 돌아가는 길의 시작이었다.*

<br>

두번째로 gpg key를 몽땅 갈아엎고 새로 업데이트 시키자고 생각하여

`rpm --import <(curl -s -L https://packages.cloud.google.com/yum/doc/yum-key.gpg)`

명령으로 새로운 키를 받아와보려했으나 write body failed가 뜨면서 어려워졌다.


<br>

다시 오류메세지를 제대로 읽어보자고 생각, `--disablerepo=google-cloud-sdk` 옵션을 이용하여 에러메세지를 띄우고 보았더니 `google-compute-engine` 이라는 레포지토리가 하나 더 걸려서 에러가 지속되었던 것. 

최종적으로 

```
sudo yum install wget --disablerepo=google-cloud-sdk,google-compute-engine -y
```

처럼 걸리는 레포지토리들을 전부 `--disablerepo` 옵션으로 걸러내주니 정상 작동하였다. 

![complete](https://res.cloudinary.com/danhdvla9/image/upload/v1616996607/ScreenShots/21-03-29/2_nxe0jc.png)
*예에~*

하지만 명령마다 저 옵션을 써주기엔 너무 번거로워서 조금 더 시간을 내어 완벽하게 오류를 고쳐보려했으나 

![find error](https://res.cloudinary.com/danhdvla9/image/upload/v1616996607/ScreenShots/21-03-29/3_udzgxq.png)
*(처절한 에러와의 싸움.png)*

<br>

고쳐지는건 나였고, 구글의 프로그래머 분들에게 떠넘기기로 했다. ~~헤헤~~

<br>

---

<br>

### 추가 

<br>
<br>

글을 적으면서 문득 떠오른것이, '어 그럼 gpg 키 검사 안하면 되는거 아닌가?' 

그래서 `--nogpgcheck` 옵션으로 해봤더니 또 잘된다. 

gpg key는 무엇이고 왜 없어도 잘 동작되는지 궁금해지기 시작했다.




