---
layout: post	
title: "Nginx Load Balancer"
date: 2021-04-01
categories:
  - Back-End-Study
description:
image: https://res.cloudinary.com/danhdvla9/image/upload/v1617008667/Thumbnails/jenkins_vhzpmh.png
image-sm: https://res.cloudinary.com/danhdvla9/image/upload/v1617008667/Thumbnails/jenkins_vhzpmh.png
image-me: https://res.cloudinary.com/danhdvla9/image/upload/v1614694302/Blacksmith_vqd5bz.png
---

<br>
<br>

## Nginx 설치 및 간단 Load Balancer 설정


<br>
<br>

#### Nginx 란? 

<br>
<br>

엔진엑스(Nginx)는 Igor Sysoev라는 러시아 개발자가 동시접속 처리에 특화된 웹 서버 프로그램이다. Apache보다 동작이 단순하고, 전달자 역할만 하기 때문에 동시접속 처리에 특화되어 있다.
Nginx의 역할 중 가장 중요한 두 가지 역할은 다음과 같다.

 - 정적 파일을 처리하는 HTTP 서버로서의 역할
 - 응용프로그램 서버에 요청을 보내는 [리버스 프록시](https://m.blog.naver.com/alice_k106/221190043948)로서의 역할

또한 Nginx는 비동기 처리 방식(Event-Drive) 방식을 채택하고 있다.

 - 동기(Synchronous) : A가 B에게 데이터를 요청했을 때, 이 요청에 따른 응답을 주어야만 A가 다시 작업 처리가 가능 (하나의 요청, 하나의 작업에 충실)

 - 비동기(Asynchronous) : A의 요청을 B가 즉시 주지 않아도, A의 유휴시간으로 또 다른 작업 처리가 가능한 방식

[동기식 처리 방식과 비동기식 처리 방식의 차이](https://poiemaweb.com/js-async)

<br>
<br>

#### Nginx 설치

<br>
<br>

*작성자는 Ubuntu 20.04 환경에서 실행했습니다.*

`sudo apt-get install nginx` 명령어로 간단하게 설치 할 수 있다. 



<br>
<br>

#### Nginx 실행

<br>
<br>

설치 된 상태 그대로 `sudo systemctl start nginx` 를 통해 Daemon을 실행해주면 바로 기본 index 페이지가 뜬다. 이 상태 그대로 웹 서버처럼 사용할 수도 있지만, Load Balacer 로써 사용하기 위해서는 `nginx.conf` 파일을 설정해주어야 한다. `sudo vi /etc/nginx/nginx.conf` 명령어를 통해 nginx의 설정을 수정하자.

**Load Balancing 설정을 하기 전, [Nginx 공식 기술 문서](https://docs.nginx.com/nginx/admin-guide/load-balancer/http-load-balancer/)를 참조 할 것을 추천한다.**

내가 듣는 강의에서 사용한 예시 코드 :

```
upstream cpu-bound-app {
  server {instance_1번의_ip}:8080 weight=100 max_fails=3 fail_timeout=3s;
  server {instance_2번의_ip}:8080 weight=100 max_fails=3 fail_timeout=3s;
  server {instance_3번의_ip}:8080 weight=100 max_fails=3 fail_timeout=3s;
}

location / {
  proxy_pass http://cpu-bound-app;
  proxy_http_version 1.1;
  proxy_set_header Upgrade $http_upgrade;
  proxy_set_header Connection 'upgrade';
  proxy_set_header Host $host;
  proxy_cache_bypass $http_upgrade;
} 

#code by foo
```

설정을 이렇게 바꿔주었다면 `sudo systemctl restart nginx` 또는 `sudo systemctl reload nginx`를 통해 Daemon을 재시작 해주어야 한다. 

<br>
<br>

여기까지하면 Nginx가 알아서 Load Balancing을 잘 해줄것 같지만 테스트해보면 그렇지가 않다. 이를 해결하기위한 과정이 아래 이어진다.

<br>
<br>

#### TroubleShooting

<br>
<br>

위와 같이 설정을 마친 후, 다시 nginx에 접속해보면 404 Error 페이지가 맞이해준다. 어떤 문제점이 있었는지 확인하기 위해 nginx 서버의 log 파일을 볼 필요가 있다. 

`sudo tail -f /var/log/nginx/error.log;` 명령어를 통해 log를 열어보면, connect() 가 실패했고 에러코드 13: 권한없음 이라고 적힌 에러가 많이 나와있다. 이 에러코드를 복사해서 구글링하면 해결하기 위해 아래 명령어를 사용해야한다고 나온다. 

`sudo setsebool -P httpd_can_network_connect on`

이는 connect() 메서드가 기본적으로 사용을 막아놓았기 때문에 발생하는 에러였다. 위 명령어를 실행해주고 다시 접속해보면 정상적으로 nginx가 load balancing 하고 있는 것을 확인 할 수 있다.
