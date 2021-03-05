---
layout: post	
title: "Cpu Stress Test -1"
date: 2021-03-03
categories:
  - Back-End-Study
description:
image: https://res.cloudinary.com/danhdvla9/image/upload/v1614694356/cpu_ncib23.jpg
image-sm: https://res.cloudinary.com/danhdvla9/image/upload/v1614694356/cpu_ncib23.jpg
image-me: https://res.cloudinary.com/danhdvla9/image/upload/v1614694302/Blacksmith_vqd5bz.png
---

<br>
<br>

#### 목차

<br>
<br>
<ul>
	<li>I/O Burst, I/O Bound VS CPU Burst, CPU Bound</li>
	<li>Hash를 이용해서 CPU를 극단적으로 사용하기 (feat. Spring Boot)</li>
	<li>GCP 인스턴스에 배포</li>
</ul>
<br>
<br>

---

<br>
<br>

#### I/O Burst, I/O Bound VS Cpu Burst, Cpu Bound

<br>
<br>

I/O Burst와 I/O Bound는 무엇이고, Cpu Burst와 Cpu Bound는 무엇일까?

<br>

이를 설명하기 위해서는 우선 프로세스가 처리되는 과정을 먼저 알아야한다.
그 전에 프로그램과 프로세스의 차이점부터 짚고 넘어가자.

<br>

**프로그램** : 하드에 저장되어 있는 Application <br>
**프로세스** : 메모리에 적재되어 있는 프로그램

<br>
<br>

이렇게 메모리에 적재되어 처리를 기다리고 있는 프로세스들을 처리하는 순서를 **프로세스 스케줄링**이라 한다.

<br>

이 프로세스 스케줄링에 따라 프로세스들은 CPU에 의해 처리되는데, 이 때 CPU에 I/O(input/output)되는 시간을 **I/O Burst**, CPU가 프로세스를 처리하는 시간을 **CPU Burst**라고 한다.

<br>

그리고 I/O Burst가 큰 프로그램을 **I/O Bound**, CPU Burst가 큰 프로그램을 **CPU Bound**라고 한다.

<br>
<br>

나는 GCP CPU 테스트를 위해 CPU Burst가 높은 간단한 프로그램을 만들었다. 

<br>
<br>
<br>
<br>

#### Hash를 이용해서 CPU를 극단적으로 사용하기

<br>
<br>

아래와 같은 코드를 참조하여 hash (md5)를 이용하는 프로그램을 작성하였다. 

hash가 무엇인지 궁금하다면? 

[해시에 대해](https://medium.com/@yeon22/crypto-%ED%95%B4%EC%8B%9C-hash-%EB%9E%80-6962be197523)

<br> 

	import org.springframework.web.bind.annotation.PathVariable;
	import org.springframework.web.bind.annotation.RequestMapping;
	import org.springframework.web.bind.annotation.RestController;

	import javax.xml.bind.DatatypeConverter;
	import java.security.MessageDigest;
	import java.security.NoSuchAlgorithmException;

	@RestController
	public class HashController {

		@RequestMapping("/hash/{input}")
		public String getDigest(@PathVariable("input") String input) throws NoSuchAlgorithmException {
		    for(int i = 0; i < 100_000; i++) {
		        input = getMD5Digest(input);
		    }
		    return input;
		}

		@RequestMapping("/hello")
		public String hello() {
		    return "hello";
		}

		private String getMD5Digest(String input) throws NoSuchAlgorithmException {
		    MessageDigest md = MessageDigest.getInstance("MD5");
		    md.update(input.getBytes());
		    byte[] digest = md.digest();
		    String myHash = DatatypeConverter
		            .printHexBinary(digest).toUpperCase();

		    return myHash;
		}
	}

<br>

참조 : https://github.com/lleellee0/cpu-bound-application <br>
--추가사항 : application.properties 파일에서 server.port 를 80으로 설정해줘야 한다. <br>

<br>

/hash/{input} 과 /hello 두 가지 매핑이 있고, input에 들어온 문자열을 10만번 해시 연산한 결과를 반환한다.

<br>

---

ERROR FIX

---

<br>

JDK8이 아닌 다른 버전을 쓸 경우 DataTypeConverter를 찾지 못하는 오류가 있었다.

나는 11로 하고 있었기 때문에 해당 패키지가 제외되어있던 상태라 의존성을 추가해주어야 했다. 

	<dependency>
		<groupId>javax.xml.bind</groupId>
		<artifactId>jaxb-api</artifactId>
		<version>2.3.1</version>
	</dependency>

<br>

#### GCP 인스턴스에 배포 

<br>

1 intelliJ 우측에서 maven 탭을 통해 .jar 파일을 만든다. (Maven -> Lifecycle -> deploy 더블클릭으로 생성!)

<br>

> 익숙했던 maven 이지만 처음 알았던 내용인 만큼 조금 충격이었달까...
> 그동안 배포를 한번도 안해봐서 .jar 파일을 만들 생각도, 만들어 볼 못했지만 
> 이렇게해서 배포가 이루어 진다는 것을 배웠을 때 굉장히 희열을 느꼈다.
> ( ~~생각해보니까 패키지 매니저인데 의존성 관리만 하고 패키징을 제대로 해본 적이 없었네...~~ )

<br>

2 깃허브 Repo에 push.

<br>

3 GCP 인스턴스에 가서 wget [repo에 저장된 .jar파일 주소]을 사용해 인스턴스에 다운로드.

<br>

 - 나는 CentOS를 설치했기 때문에 yum install wget을 통해 wget 설치하였다. <br> 
 - GCP 는 한 계정당 3달간 300달러 무료 크레딧을 지급한다. 초과 시 과금되니 계정관리 필수 !! <br>
 - GCP 인스턴스 생성에 대해서는 생략한다. <br>

<br>

4 sudo java -jar [다운받은 .jar 파일명] 명령어를 통해 실행 테스트! 

<br>
<br>

![GCP instance](https://res.cloudinary.com/danhdvla9/image/upload/v1614939147/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7_2021-03-05_19-07-13_m3hhzt.png "구동테스트")

정상적으로 실행되는 모습이다. 

<br>
<br>
<br>
<br>
<br>
<br>

스트레스 테스트를 위한 과정 중 중간까지 왔다. 이제 준비가 다 되었고 스트레스 테스트하는 일만 남았는데 준비하는 과정만해도 새로이 배우는 것들이 많아서 굉장히 재미있었다. <br>

다음 포스팅에서는 스트레스 테스트 툴 중 하나인 Artillery를 이용하여 테스트 결과를 출력해보자.
