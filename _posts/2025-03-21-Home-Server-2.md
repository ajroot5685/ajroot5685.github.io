---
title: 홈서버 구축 2 - 공유기 설정
date: 2025-03-21 12:50:00 +09:00
categories: [홈서버]
tags:
  [
    홈서버,
    미니pc,
    공유기
  ]
---

## 들어가며

저번 포스팅에서 우분투 설치 및 네트워크 설정을 완료했다면 `ping 8.8.8.8`으로 정상적으로 인터넷과 통신 가능한 상태가 될 것이다.
<br>
그리고 게이트웨이로 공유기의 ip주소(`192.168.xxx.1`)를 지정했을 것이다.
<br>
홈서버를 클라이언트가 편하게 이용하게 하려면 변경 가능성이 있는 정보들을 고정시켜줘야 한다.
<br>
이번에는 여러 공유기 설정을 통해 클라이언트에게 고정된 엔드포인트를 제공해주는 환경을 구축해보겠다.

<br>

## 공유기 종류

현재 나는 LG U+ 공유기를 사용하고 있다.
<br>
정확한 모델명은 Davolink의 `GAPD-7300`이다.

> 설정할 때마다 1분 가량의 다운타임이 발생하는게 안좋긴 하다.

<br>
공유기 종류는 상관없지만 DDNS, 포트포워딩 등의 커스텀이 가능한 공유기여야 한다.
<br>
최신 공유기들은 커스텀이 불가한 경우가 있는데, iptime 공유기를 사용하면 되겠다.

<br>

## DHCP 고정 할당

> 네트워크에 연결된 기기에 IP 주소와 네트워크 설정을 자동으로 할당해주는 프로토콜
{: .prompt-info }

보통 가정은 하나의 공개IP 주소를 공유기가 할당받는다.
<br>
다른 장치들은 공유기에 연결되어 사설IP로 구분된다.
<br>
여기서 각 장치들에 사설IP를 부여해줄 `Dynamic Host Configuration Protocol` 서버가 필요한데 공유기가 그 역할을 수행하게 된다.
<br>
공유기가 알아서 각 장치들에게 사설IP를 부여해주는 것이다.
<br><br>
장치 구성에 따라 할당되는 IP가 바뀔 수 있는데, 외부에서 같은 장비에 접속하려면 고정을 시켜주어야 한다.
<br>

<img src="/assets/img/250321/dhcp.png" style="border-radius:5px" alt="dhcp" width="700">

각 공유기 사이트에 접속해서 메뉴들을 찾아보면 DHCP와 관련된 설정 페이지가 있다.
<br>
장치의 MAC 주소와 할당할 IP주소를 부여하여 고정시킬 수 있다.

<img src="/assets/img/250321/mac.png" style="border-radius:5px" alt="mac" width="700">

MAC 주소는 공유기의 검색 기능으로 확인하거나 `ifconfig` 명령어로 확인한다.
<br>
설정했던 네트워크(나는 `enp1s0`이다.)의 ether 값이 MAC 주소가 되겠다.
<br>
할당할 IP주소는 네트워크에 설정한 사설IP 주소를 넣어준다.
<br>
정상적으로 적용되면 할당 정보에서 확인할 수 있다.

<img src="/assets/img/250321/dhcp2.png" style="border-radius:5px" alt="dhcp2" width="700">

<br>

## DDNS(Dynamic Domain Name Server)

> 유동 IP 환경에서 도메인을 고정시켜 외부에서 항상 같은 주소로 접속할 수 있게 하는 서비스
{: .prompt-info }

IP 자원을 절약하기 위해서, 대부분의 가정용 공유기는 IP가 고정이 아닌 유동이다.
<br>
고정된 엔드포인트를 제공하기 위해서는 DDNS를 설정해야 한다.
<br>
iptime 같은 공유기는 자체 도메인(`iptime.org`)을 제공하지만, 나처럼 서드파티 도메인을 사용해야 하는 경우가 있다.

<img src="/assets/img/250321/ddns_setting.png" style="border-radius:5px" alt="ddns_setting" width="700">

나는 무료로 사용 가능한 freedns를 선택했다.
<br>
사용자 등록을 눌러 [https://freedns.afraid.org/](https://freedns.afraid.org/) 홈페이지로 이동해준다.
<br>

<img src="/assets/img/250321/signup.png" style="border-radius:5px" alt="signup" width="800">

Sign up Free를 눌러 회원가입을 진행한다.

<img src="/assets/img/250321/subdomain.png" style="border-radius:5px" alt="subdomain" width="500">

왼쪽 Subdomains 탭에서 add 버튼을 눌러 서브도메인을 등록한다.
<br>
IPv4로 연결해주는 A레코드를 선택하고 서브도메인, 도메인 종류, 목적지 ip를 선택하고 만들어주자

> mooo.com은 가장 짧아서 선택한 것일뿐, 다른 것을 선택해도 무방하다.

이제 공유기 ddns 설정 페이지로 돌아가서 설정 정보들을 넣고 적용해주면 도메인 명으로 공유기에 접근할 수 있다.
<br>
또한 ip주소가 바뀔 때마다 ddns 설정을 자동으로 업데이트하게 될 것이다.

<img src="/assets/img/250321/ddns_setting2.png" style="border-radius:5px" alt="ddns_setting2" width="400">

<br>

## 포트포워딩

하지만 아직 홈서버에 접근할 수는 없다.
<br>
경로가 지정되지 않았기 때문이다.
<br>
경로를 지정하기 전에 우리는 하나의 홈서버로 여러 개의 백엔드 인스턴스를 사용할 것이다.
<br>
`sub2.sub1.domain.com`같은 2단계 서브도메인을 만들고 홈서버의 각 인스턴스에 연결할 수도 있지만 유료 옵션이므로, 포트번호로 매칭시켜줄 것이다.
<br>

```
예시
xxx.mooo.com:10001 -> nginx
xxx.mooo.com:10002 -> project1
xxx.mooo.com:10003 -> project2
xxx.mooo.com:10010 -> DB
```

마찬가지로 포트포워딩은 공유기의 설정 페이지에서 설정할 수 있다.

<img src="/assets/img/250321/port_forwarding.png" style="border-radius:5px" alt="port_forwarding" width="600">

<img src="/assets/img/250321/port_forwarding2.png" style="border-radius:5px" alt="port_forwarding2" width="500">

외부에 노출할 포트와 홈서버 ip, 연결할 내부 포트를 설정하면 된다.

> 0~1023 같은 well-known port는 반드시 피하자. 공개 포트를 최소화하고, 10000번대의 알려지지 않은 포트를 사용하기만 해도 외부 접근 빈도가 낮아진다.
{: .prompt-warning }

<br>

## 결과 확인

<img src="/assets/img/250321/result.png" style="border-radius:5px" alt="result" width="400">

제대로 설정을 완료했다면 인터넷에서 `도메인주소:포트`를 사용하여 정상적으로 접근할 수 있다.
<br>
위 이미지는 홈서버에 Nginx를 설치하고 테스트 사이트를 배포한 것이다.

> Nginx는 다른 포스트에서 소개한다. 지금은 연결할 수 없다는 에러 페이지가 떠도 괜찮다.