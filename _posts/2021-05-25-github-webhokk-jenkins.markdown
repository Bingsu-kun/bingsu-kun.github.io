---
layout: post	
title: "github webhook과 jenkins로 배포 자동화하기"
date: 2021-05-25
categories:
  - Back-End-Study
description:
image: https://res.cloudinary.com/danhdvla9/image/upload/v1617008667/Thumbnails/jenkins_vhzpmh.png
image-sm: https://res.cloudinary.com/danhdvla9/image/upload/v1617008667/Thumbnails/jenkins_vhzpmh.png
image-me: https://res.cloudinary.com/danhdvla9/image/upload/v1614694302/Blacksmith_vqd5bz.png
---

<br>
<br>

## 깃허브에 코드만 올렸을 뿐인데 자동으로 배포까지?

<br>
<br>

오늘은 배포 자동화에 대해 공부하면서 배운 내용을 정리해보고자 한다.  
우리는 흔히 프로젝트의 소스코드들을 깃허브에 레포지토리를 만들어서 관리한다. 열심히 코딩해서 릴리즈를 배포해야 될 시기가 왔다고 할 때, 어떤 과정을 거쳐서 배포가 이루어지는지 먼저 살펴보자.  

Spring 프로젝트를 배포한다고 가정하면, 우선 수정된 사항을 브랜치에 push한다. 
다음 로컬 환경에서 프로젝트를 빌드하고 빌드 후 생긴 jar나 war파일을 서버로 전송한다. 
그 후 배포 스크립트를 통해서 배포한다. 하지만 이것은 개인 프로젝트일 때의 이야기이고, 실무에서는 중간에 테스트 서버에서 테스트를 거치고 QA까지 진행된 후 배포가 된다. 

위 과정 중 push - build - 전송 - 배포 과정이 반복되는 경우가 많다. 이러한 과정을 github webhook과 jenkins를 이용하여 단축할 수 있다. 소스코드를 push만하면 github webhook을 통해 push된 사실을 감지하여 jenkins가 자동으로 코드를 build하고 배포까지 진행한다.

<br>
<br>

#### Jenkins 빌드 구성

<br>
<br>

빌드 구성은 여느 일반 jenkins 사용과 다를 것이 없다.  
우선 아이템을 만들고, 아이템 구성에서 소스코드 관리를 git으로 변경해준다. 다음 URL에 git 프로젝트 레포지토리 주소를 적어주고, 배포를 위한 브랜치가 따로 있다면 아래 branch를 적는 곳에 추가로 적어준다.
그리고 빌드유발에 github hook trigger를 체크해준다. 이를 체크해주어야 github webhook에 의한 빌드 시작이 가능해진다. 다음 빌드 항목에서 빌드 스텝을 생성해주고 **Excute shell에 `./mvnw clean package`를 입력한다.**  

이는 mvnw를 이용해서 target 폴더 clean을 진행 후 package 명령을 실행한다는 의미이다.  
**여기서 주의할 점이 있는데, 개발환경은 윈도우인데 배포환경이 Linux인 경우 애플리케이션 권한 체계가 다르므로 위 명령어 전에 `chmod 544 ./mvnw`를 통해 권한을 주어야 정상 작동한다.**  

여기까지 진행하면 코드를 push하면 jenkins에서 자동 build한다. 이제 배포를 자동화 해보자.  

<br>
<br>

#### Jenkins 배포 구성

<br>
<br>

빌드 된 후 실행되어야하는 파일을 지정해줘야 하므로 "빌드 후 조치" 항목에서 Transfer Set을 수정해주자.  

 - Source files를 target/~~~.jar 로 수정 (Spring, maven 프로젝트 기준)
 - Remove prefix에 target 작성 (~~~.jar 앞의 target을 지워주는 역할)
 - exec command에는 다음과 같이 작성.

```
	sudo kill -15 $(sudo lsof -t -i:8080)
	nohup java -jar ~~~ > nohup.out 2 > &1 &

```  

첫째 줄의 코드는 기존에 실행되고있던 애플리케이션을 중단한 후 새로 시작하기 위해 넣은 코드이다. 만약 이 코드를 넣지 않는다면 두번째 배포시에 이미 8080포트가 이용 중이라서 실행할 수 없다는 에러메세지가 출력된다. 시간이 충분하다면 첫째 줄을 넣지않고 진행 해보는 것을 추천한다 :) 그리고 첫째 줄의 lsof 명령어는 사용되고 있는 포트를 출력해주는 명령인데 기본 제공되는 명령어가 아니므로 apt나 yum을 이용해서 별도 설치해주어야 한다. 또한 kill -15가 아닌 -9도 사용가능한데 -15는 terminate, -9는 kill을 뜻한다. 이 둘의 차이는 terminate의 경우 이미 할당된 프로세스를 전부 마무리하고 종료시키지만 kill은 그 즉시 종료시킨다. 따라서 -9 보다는 -15가 권장된다. ( ~미리 업데이트를 공지한 경우에는 -9가 더 편할 것 같긴하다...~ )  
둘째 줄의 코드는 jar를 백그라운드로 실행함과 동시에 프로세스가 끝나도 종료하지말고 표준 출력과 표준 에러 출력을 모두 nohup.out 파일로 리다이렉션하는 코드이다. 이와 관련된 내용은 더욱 자세히 설명해주신 다른 분의 블로그를 [참조](https://joonyon.tistory.com/98)바란다.  

<br>
<br>

#### Github Webhook 설정 

<br>
<br>

github webhook은 설정이 간편하다. github의 repository에서 settings - webhooks - add webhook으로 새로운 hook을 만들어준다.  
payload url은 jenkins 인스턴스의 ip:port를 복사 붙여넣기 해준 후, 뒤에 추가로 api를 작성해준다. 이 api는 jenkins에서 빌드 유발에 github hook trigger를 체크하고 나왔던 url링크를 수정해줬다면 그와 똑같이 적어야한다. 수정해주지 않았다면 디폴트로 github-webhook일 것이다. 이는 추후에 바뀔 수 있으니 참고만 바란다. 그리고 content type은 json으로 해주면 마무리된다. 아마 json 이외의 타입으로 전송될 경우는 적다고 생각하지만 혹시 다른 타입으로(ex. xml) 전송한다면 변경해 주어야한다.

<br>
<br>

#### 보완사항

<br>
<br>

여기까지 하면 자동 배포까지는 완료된다. 하지만 jenkins의 exec command를 이렇게 작성할 경우 모든 서버 인스턴스가 동시에 업데이트에 들어가게된다. 그러면 배포가 진행 중인 동안에는 서비스가 끊기게되어 무중단 배포가 이루어지지 않는다. 이를 해결하기위해 간단하게 롤링업데이트 하는 방법을 생각해보았다.  
만약 서버 인스턴스가 3개라고 가정했을 때, 두번째와 세번째 exec command의 첫째줄에 sleep 명령어를 추가해서 한 박자 늦게 시작하게끔하면 모든 인스턴스가 동시에 업데이트에 들어가지 않고 차례대로 진행하게 된다. 하지만 이 방법은 야매(?)스러운 방법이라 문제점이 많다. 예를들어, 제대로된 롤링 업데이트의 경우 두번째나 세번째 인스턴스의 업데이트를 진행하던 도중 에러가 발생하여 진행하지 못하게되면 첫째 인스턴스도 다시 롤백되어야 한다. 그렇지만 이 코드는 롤백의 기능은 없다. 더불어 sleep 커맨드로 시간을 지정해주기 때문에 (ex. 30) 업데이트에 걸리는 시간을 예측해야하는데, 이를 잘 맞추지 못할 경우 필요 이상으로 시간이 오래걸리거나 모든 인스턴스가 업데이트에 들어가있는 사태가 벌어질 수도 있다.
그래서 이는 수정이 필요한데 jenkins에서 롤링 업데이트나 카나리 업데이트를 하는 방법을 조금 생각해 봐야할 것 같다.  

또한 요새는 유연한 확장을 위해 애플리케이션을 서비스 단위로 쪼개서 개발하는 MSA (Micro Service Architecher)가 일반적이다. 그리고 그 중심 기술이 컨테이너 기술인데, 이를 이용하기 위해서는 Docker가 필수라고 할 수 있다. Dockerhub에 automated build라는 것을 이용해서 github의 소스를 컨테이너화 시킬수 있다고 하던데 오늘 정리한 github webhook과 jenkins사이에 dockerhub automated build를 추가하여 컨테이너화 된 애플리케이션을 jenkins가 배포할 수 있도록 하는 방법도 공부해봐야겠다.