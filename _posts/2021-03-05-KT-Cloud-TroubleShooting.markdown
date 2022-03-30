---
layout: post	
title: "KT Cloud TroubleShooting (ubuntu account)"
date: 2021-03-05
categories:
  - Trouble-Shooting
description:
image: https://res.cloudinary.com/danhdvla9/image/upload/v1615973205/Thumbnails/Troubleshooting-Icon_ou9s2b.png
image-sm: https://res.cloudinary.com/danhdvla9/image/upload/v1615973205/Thumbnails/Troubleshooting-Icon_ou9s2b.png
image-me: https://res.cloudinary.com/danhdvla9/image/upload/v1614694302/Blacksmith_vqd5bz.png
---


### 리눅스 OS (ubuntu)에서 KT Cloud Instance OS (ubuntu) ssh 접속 문제

<br>
<br>

 - Description
 
KT Cloud의 D1을 이용하다가, 사용 중인 노트북의 os가 ubuntu였기 때문에 시작된 문제이다. ssh 접속 시 openssh의 버전에 따라 client에서의 접속이 차단 될 수 있다는 점을 새로 알게되었고, KT Cloud 기술 문서에서 오점을 찾아내었다. 

<br>

 - Error Code
 
ubuntu에서 ssh 명령어로 원격 접속하려 했으나 Connection Closed 됨.

Windows에서 putty를 이용해 접속하려 했으나 not existing current key 출력 됨.

<br> 

 - TroubleShooting
 
Connection Closed가 되었다는건 연결은 되나 무언가의 오류로 인해 연결을 막고 있다는 뜻으로 해석했다. 

그리고 언제나 나를 괴롭혀왔던 버전문제... 이번에도 역시나였다. (~~운이 좋았다. 헤헤~~)

client와 server의 버전이 다를 경우 ssh 접속이 차단되는지 구글링을 시작하였고, 얻어낸 답은 

ssh는 client의 버전이 server 버전보다 높을 경우 Connection Closed로 접속을 차단한다.

<br>

그래서 Windows로 먼저 접속해서 openssh의 버전을 update하고 다시 ubuntu로 시도해보기로 했다. 

그런데 Windows로 접속 시도 중 예상치 못하게 키페어가 없다는 오류가 뜨면서 막막해지기 시작했다. 

![KT Cloud User Guide](https://res.cloudinary.com/danhdvla9/image/upload/v1615814654/ScreenShots/21-03-05/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7_2021-03-15_22-12-33_ranxex.png "너 때문이야")

공식 기술 문서를 다시 읽어보고 여러번 시도해보고 실패하고 반복하던 도중, 기술 문서에는 ubuntu os 는 계정명을 ubuntu으로 사용하라는 문장을 발견. 

설마... 하는 느낌이 와서 계정을 ubuntu가 아닌 root로 바꿔보았다. CentOS는 root 로그인이 가능한데 ubuntu는 따로 언급이 없다는 것이 흠칫했다. 

<br>

접속에 성공했다.....

<br>

허무했다. 하지만 기술 문서에서 보완할 점을 하나 찾았다는 것이 꽤 뿌듯했다. 앞으로 클라우드 서버를 자주 이용하게 될테니 신고식 한 번한 셈 쳐야겠다.


