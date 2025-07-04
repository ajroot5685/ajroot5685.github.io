---
title: Network Address Translation - 사설 IP 장치의 네트워크 통신 방법
date: 2025-02-10 19:28:00 +09:00
categories: [network]
tags:
  [
    ip,
    nat
  ]
---

## 들어가며

[이전 포스팅](https://ajroot5685.github.io/posts/Internet-Protocol-Subnet/)에서 IPv4의 주소고갈 문제를 개선하기 위한 서브넷 마스크, CIDR을 다뤘었다.
<br>
이 방식을 사용함에도 주소고갈 문제는 여전히 존재했다.
<br>
이를 개선하기 위해 추가로 사설 IP(private IP) 개념을 도입하며 NAT(Network Address Translation)이 사용되었다.
<br>
NAT과 함께 여러 개념들을 확인하며 직접 통신이 불가능한 내부 장치가 어떻게 외부와 통신하는지 알아보겠다.

<br>

## Private IP

이름 그대로 외부에 공개되지 않은 내부(가정, 회사 등)에서 사용하는 네트워크 내 주소이다.
<br>
인터넷에 직접 연결되지 않고 NAT로 공인 IP로 변환되어 통신한다.
<br>
주소고갈 문제를 개선하는 핵심 요소는 각 네트워크에서 사설 IP가 중복되어도 된다는 점이다.

<br>

#### [범위]

| 사설 IP 대역 | 사용 가능 범위 | 총 개수 |
|-------------|--------------|----------|
| 10.0.0.0/8 | 10.0.0.1 ~ 10.255.255.254 | 16,777,214개 |
| 172.16.0.0/12 | 172.16.0.1 ~ 172.31.255.254 | 1,048,574개 |
| 192.168.0.0/16 | 192.168.0.1 ~ 192.168.255.254 | 65,534개 |

전체 IP의 약 0.42%가 사설 IP로 예약되어 있다.
<br>
공유기를 다루다보면 흔히 볼 수 있는 `192.168.0.0` 대역이 가장 흔히 볼 수 있는 사설 IP이다.

<br>

#### [보안]

위에서 **인터넷에 직접 연결되지 않는다**고 했다.
<br>
이를 통해 얻을 수 있는 부가적인 장점은 **보안이 향상**된다는 점이다.
<br>
NAT을 통해서만 연결될 수 있기 때문에 공인 IP가 없다면 외부에서 공격할 방법이 사라진다.
<br>
때문에 보안이 필요한 기업에서 사설 네트워크를 구축하기도 한다.

> 다만 내부 네트워크 내에서 발생하는 공격은 막을 수 없으므로 이에 대한 대비가 필요하다.

<br>

## Public IP

전세계에서 유일하며, 인터넷에서 직접 접근 가능한 IP이다.
<br>
ISP로부터 할당 받으며, 웹서버, 클라우드 서버, 네트워크 장비 등이 사용한다.

<br>

#### Elastic IP(탄력적 IP)

AWS와 같은 클라우드 서비스에서 제공하는 공인 IP이다.
<br>
보통 대여받은 서버를 재부팅하면 부여받은 공인 IP는 초기화된다.
<br>
이를 고정시키고자 할 때 탄력적 IP를 부여받을 수 있다.

<br>

## Network Address Translation

> 서로 다른 네트워크 간 IP 주소를 변환하는 역할을 수행한다.
{: .prompt-info }

IPv4의 주소 부족 문제를 개선한다
<br>
라우터 또는 게이트웨이에서 수행된다.

> 가정에서는 주로 라우터에 해당하는 공유기가 NAT를 수행한다.

<br>

#### [주의]

유의해야 할 점은 NAT이라는 기술이 단지 **사설 IP ↔ 공인 IP 변환**에 초점을 둔 것이 아니라는 점이다.
<br>
위의 정의처럼 **IP주소 변환**이 NAT의 핵심이며, 사설 IP ↔ 공인 IP 변환은 한 가지의 활용 사례일 뿐이다.

> 다만 여기서는 사설 IP ↔ 공인 IP 변환에 대해 초점을 맞추어 설명한다.

<br>

#### [NAT의 유형]

크게 정적 NAT, 동적 NAT, PAT이 있다.
<br>
정적 NAT는 1개의 사설 IP와 1개의 공인 IP를 1:1로 매핑한다.
<br>
동적 NAT는 사설 IP 주소를 공인 IP 풀에서 동적으로 할당한다.
<br>
위의 두 유형은 공인 IP를 매우 많이 사용하기 때문에 실제로 주로 사용되는 유형은 아니다.

<br>

## Port Address Translation(PAT)

가장 일반적인 NAT 방식이며, 여러 개의 사설 IP가 하나의 공인 IP를 공유한다.
<br>
포트 번호를 기반으로 변환하여 다중 접속을 구현한다.
<br>

예시
- 사설 IP + 포트 ↔ 공인 IP + 변환된 포트

<br>

#### [PAT 동작 과정]

일반 가정 PC에서 구글 서버에 접속하는 과정을 예시로 보겠다

<img src="/assets/img/25/02/10/pat.png" style="border-radius:5px" alt="pat" width="800">

1. 클라이언트(PC)가 인터넷 접속 요청을 보냄
- 사설 IP : 192.168.1.10:5000 → 목적지 : 8.8.8.8:80(구글)

2. 라우터(공유기)가 사설 IP를 공인 IP로 변환하며 포트도 할당
- 192.168.1.10:5000 → 203.0.113.10:15000
- 패킷의 출발지가 변환됨
  - 203.0.113.10:15000 → 8.8.8.8:80
- 변환 내용은 라우팅 테이블로 관리

3. 패킷이 인터넷을 통해 구글 서버로 전달됨
- 구글 서버는 패킷을 보고 **203.0.113.10:15000에서 요청이 왔다고 인식**

4. 구글 서버가 응답을 보냄
- 8.8.8.8:80 → 203.0.113.10:15000

5. 공유기가 라우팅 테이블을 보고 원래의 사설 IP로 변환
- 8.8.8.8:80 → 192.168.1.10:5000

6. PC가 응답을 받아 추가 작업 처리

<br>

## NAT의 한계

#### [포트 개수 제한]

포트 최대 개수는 65,535개로, 예약된 포트를 제외하면 약 6만개 정도가 사용 가능하다.
<br>
가장 작은 `192.168.0.0` 대역만 하더라도 6만5천개이므로, 전부 다 사용하기에는 제약이 있다.
<br>
이때는 여러 개의 공인 IP를 사용해야 한다.

<br>

#### [P2P, 온라인 게임, 화상회의 환경에서의 제약]

양방향 소통이 필요한 경우 NAT 환경에서는 제약이 발생한다.
<br>
NAT에서 변환된 IP 및 포트 정보는 일시적으로 유지되며, 일반적으로 내부 네트워크에서 외부로 요청을 보낸 후 응답을 받는 방식으로 동작하기 때문이다.
<br>
즉, 사설 IP를 사용하는 내부 장치가 외부에서 직접 접속을 받아야 하는 서버 역할을 수행하기에는 어려움이 있다.
<br>
이러한 문제를 해결하기 위해 주로 `포트포워딩`이라는 기술이 사용된다.

<br>

## 포트포워딩

> 외부 네트워크에서 특정 포트로 들어오는 요청을 내부 네트워크의 지정된 장치로 전달하는 기술이다.
{: .prompt-info }

> NAT와 마찬가지로 사설 IP로 포워딩하는 것이 주 목적이 아니라는 것에 유의하자.

반영구적으로 매핑시킬 수 있으므로, 양방향 소통이 필요한 상황에서 사용할 수 있다.

<br>

## 기타 - Dynamic Host Configuration Protocol(DHCP)

> 네트워크에서 장치(PC, 스마트폰 등)에 자동으로 IP 주소를 할당하는 프로토콜이다.
{: .prompt-info }

공유기는 NAT, 포트포워딩 외에도 DHCP 서버의 역할도 수행한다.
<br>
공유기와 연결된 장치들은 자동으로 사설 IP를 부여받게 된다.
<br>
가정의 공유기에 접속해서 연결된 기기들의 IP주소를 확인할 수 있다.

<img src="/assets/img/25/02/10/dhcp.png" style="border-radius:5px" alt="dhcp" width="600">
