---
layout: post	
title: "Cpu Stress Test -2"
date: 2021-03-15
categories:
  - Back-End-Study
description:
image: https://res.cloudinary.com/danhdvla9/image/upload/v1615614663/Thumbnails/cpu_cieppp.jpg
image-sm: https://res.cloudinary.com/danhdvla9/image/upload/v1615614663/Thumbnails/cpu_cieppp.jpg
image-me: https://res.cloudinary.com/danhdvla9/image/upload/v1614694302/Blacksmith_vqd5bz.png
---

<br>
<br>

#### 목차

<br>
<br>
<ul>
	<li>Artillery 설치</li>
	<li>Artillery를 사용해서 CPU Stress Test</li>
</ul>
<br>
<br>

---

<br>
<br>

#### Artillery 설치 

<br>
<br>

오늘은 Artillery를 사용해서 우리가 만든 웹서버가 얼마나 많은 트래픽을 소화해낼 수 있는지 테스트해보았다.

내가 수많은 테스트 툴 중에서 Artillery를 선택한 이유는 다음 두 가지이다.

 - Http, websocket 프로토콜 지원
 - 시각화가 잘 되어 있어 한눈에 보기 편한 리포트 페이지 제공
 
특히 리포트 페이지는 처음 열어보았을 때 너무 깔끔해서 놀랐을 정도. 

[더 자세한 Artillery에 대한 정보](https://artillery.io/docs/guides/overview/why-artillery.html)

구글 번역을 이용할 경우 artillery가 "포병"으로 해석되서 나오니 주의.

<br>

 1. node.js , npm 설치 
 
Artillery를 설치하기 위해서는 node.js와 npm이 필수이다. 각각 아래의 명령어를 입력해서 설치.

```
	sudo apt install npm
	sudo apt install nodejs
```

 2. Artillery 설치 
 
다음으로 [Artillery 공식 문서](https://artillery.io/docs/guides/getting-started/installing-artillery.html) 를 참조하여 Artillery를 설치하자.

공식 문서에서는 -g (전역 설치) 옵션과 함께 1.6 버전을 설치하지만 나는 조금 다르게 설치하였다.

```
	sudo npm install -D artillery@latest
```

프로세스가 끝나면 설치가 완료되었는지 artillery dino 명령을 사용해서 확인하였다.

![귀여운 공룡](https://res.cloudinary.com/danhdvla9/image/upload/v1615614778/ScreenShots/21-03-13/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7_2021-03-13_14-03-01_nyfkpd.png)

<br>
<br>

#### Artillery를 사용해서 CPU Stress Test

<br>
<br>

지금부터는 Artillery를 본격적으로 사용하기 위해 VSCode를 사용하였다. 

우선 open folder로 artillery 관련 산출물을 관리하기 위한 디렉토리를 하나 생성.

new file로 어떻게 테스트를 진행할지 구성하는 artillery-cpu-test.yaml 파일을 생성.

다음으로 [Artillery 공식 문서](https://artillery.io/docs/guides/getting-started/installing-artillery.html) 에서 core-concept 탭에 가서 맨 아래있는 기본 yaml 템플릿을 전부 복사 후 artillery-cpu-test.yaml에 붙여넣기 하고, 약간의 수정을 거쳤다.

![yaml 파일](https://res.cloudinary.com/danhdvla9/image/upload/v1615614780/ScreenShots/21-03-13/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7_2021-03-13_14-25-53_plikpn.png)

수정 한 이유는 hash/{input} api를 통해 get 명령만 사용할 것 이므로 나머지는 전부 날렸다. (물론 중요한 두 부분은 빼고)

 - address : 웹 서버 주소 (클라우드 외부 IP)
 - durability : 지속시간
 - ?? : 지속시간동안 1초마다 트래픽 증가 수치
 
준비가 끝났으니 웹 서버를 켜주고, 아래 명령을 입력하여 테스트를 시작하자.

```
	artillery run --output report.json artillery-cpu-test.yaml
```

![테스트 중...](https://res.cloudinary.com/danhdvla9/image/upload/v1615614781/ScreenShots/21-03-13/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7_2021-03-13_14-27-18_u0r4ft.png)

테스트가 끝나고나면 왼쪽에 지정해준 이름대로 json 파일이 생성된다. 나는 report.json으로 했으니 테스트가 잘 끝났음을 보여준다.

하지만 json 파일 만으로는 테스트가 어땠는지 파악하기 어려우니 아래 명령어를 입력해서 리포트 페이지를 열었다.


```
	artillery report report.json
```

![리포트페이지](https://res.cloudinary.com/danhdvla9/image/upload/v1615614782/ScreenShots/21-03-13/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7_2021-03-13_14-27-55_hpzhuh.png)

이와같이 깔끔하게 테스트 결과를 시각화해준다. 보아하니 서버가 그냥저냥 거뜬히 견딜 정도의 트래픽이었나보다. 

여기서 특히 P95의 수치에 집중하였다. 서버의 비용과 유저의 만족도 간 최고의 효율을 낼 수 있는 비율이 95%라고 한다. 

다음은 서버가 적당히 스트레스를 받을 정도의 부하를 줘 보자. 트래픽을 초당 15로 수정 후 report-15.json으로 다시 진행하였다.

![리포트페이지-15](https://res.cloudinary.com/danhdvla9/image/upload/v1615614783/ScreenShots/21-03-13/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7_2021-03-13_14-36-32_qc0cfs.png)

900 정도의 트래픽이 몰리니 에러도 나오고 지연 시간이 꽤 오래 걸리기 시작했다. 거의 마지막에는 1만 ms 를 넘겼는데 이는 대충 10초가 지나야 응답이 온다는 뜻이다. 내 입장에선 10 초 정도면 나름 견딜만 한 시간인데 전체 유저 입장에서 보면 적당한 것 일까 하는 의문이 든다. 

비용이 많다면 유저를 만족시키는 정확한 성능을 찾는 것이 중요하겠지만, 비용이 제한된 상황에서라면 어떤 지침을 따르는 것이 맞을지 찾아보았는데 다음 세 가지를 만족하는 방향을 찾으면 된다고 한다.

 - 예상 TPS보다 여유롭게. 예상 3000이라면 적어도 4000이상 
 - 기대 Latency를 만족할 때 까지
 - Scale-out을 해도 성능이 높아지지 않는다면 병목을 의심
 
<br>
<br>
 
이상으로 CPU Stress Test 포스팅을 마무리한다. 처음해본 배포, 처음해본 부하 테스트 였는데 작지만 내가 만든 애플리케이션이 클라우드를 통해 배포되고, 많은 트래픽이 오는 상황을 예측해볼 수 있다니 두근거리는 경험이었다. 후에 이 경험을 살려 실제로 내가 회사나 개인 프로젝트를 띄워서 관리 중일 때 얼마나 많은 트래픽을 견뎌낼 수 있을지 테스트할 수 있을 것 같다.
