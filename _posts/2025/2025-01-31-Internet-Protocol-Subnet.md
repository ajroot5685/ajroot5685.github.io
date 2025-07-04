---
title: Internet Protocol - 서브넷(Subnet)
date: 2025-01-31 16:10:00 +09:00
categories: [network]
tags:
  [
    ip
  ]
math: true
---

## 들어가며

[이전 포스팅](https://ajroot5685.github.io/posts/Internet-Protocol-IPv4/)에서 IPv4와 주소고갈에 대해 다뤘었다.
<br>
이번에는 주소고갈 문제를 개선하기 위해 지금도 쓰이고 있는 CIDR과 서브넷에 대해 알아보겠다.

<br>

## 기존 클래스 방식의 문제

지금은 대체된 클래스 방식의 문제를 다시 살펴보자

<img src="/assets/img/25/01/26/ip_class.png" style="border-radius:5px" alt="ip_class" width="800">

클래스에 따라 네트워크/호스트 부의 비트 수가 **고정되어있다**.
<br><br>
만약 1만개의 호스트가 필요한 기업에서 클래스 B를 할당받았다고 가정해보자
<br>
클래스 B의 사용 가능 호스트 수는 총 6만 5천개이다.
<br>
이 기업은 1만개 정도밖에 사용하지 않으므로, 나머지 5만 5천개는 낭비가 되는 셈이다.
<br>
그래서 주소고갈이 심화되는 클래스 방식 대신 CIDR 방식이 도입되었다.

<br>

## Classless Inter-Domain Routing

그대로 직역해보면 `클래스가 없는 도메인 간 라우팅` 이다.
<br>
클래스에 따라 네트워크/호스트 부의 비트수를 고정적으로 나누지 않고 **유연하게** 나눈다.

#### [표기 방식]

표기 방식은 `x.x.x.x/x` 로 `ip주소/네트워크 부 길이` 이다.

```
ex. 192.168.1.0/24

192.168.1.0 -> ip 주소
24 -> 네트워크 길이
```

#### [라우팅 테이블 축소]

클래스 방식의 주소낭비를 해결한 것 외에도 **라우팅 테이블이 축소**된다는 장점이 있다.
<br>
이는 여러 개의 네트워크를 하나의 요약된 경로로 묶을 수 있게 된다.
<br><br>

`192.168.0.0 ~ 192.168.3.255`범위의 ip주소를 ISP가 한 기업에게 묶어서 할당해준다고 하자.
<br>
CIDR 없이 라우팅을 하게 될 경우 여러 개의 경로를 가져야 한다.

```
192.168.0.0/24
192.168.1.0/24
192.168.2.0/24
192.168.3.0/24
```

CIDR을 사용한다면 하나의 라우트 경로로 요약이 가능해진다.

```
192.168.0.0/22
```

이것은 서브넷 마스크 덕분이다. 바로 뒤에서 자세히 알아보자.

> CIDR은 비트 단위로 네트워크 주소를 요약한다.
> 
> 따라서 2의 제곱 단위로 연속된 주소 블록만 묶을 수 있다.
{: .prompt-warning }

<br>

## 서브넷(Subnet)

> 하나의 네트워크를 여러 개의 작은 네트워크로 나누는 것
{: .prompt-info }

IP주소를 효율적으로 사용하고, 네트워크 성능 및 보안을 향상시킬 수 있다.

<br>

## 서브넷 마스크(Subnet Mask)

#### [요약]

- 서브넷 마스크는 IP 주소에서 네트워크 부와 호스트 부를 구분하는 역할을 한다.
- CIDR과 함께 사용되어 네트워크를 효율적으로 나눈다.
- IP 주소가 특정 네트워크에 속해 있는지 판별할 때 사용한다.

#### [정의]

> IP 주소에서 네트워크 부분을 가려내는 역할로, 비트 연산을 사용해 네트워크 주소를 추출하는 Key역할을 수행한다.
{: .prompt-info }

호스트 부가 모두 0이면 네트워크 주소, 모두 1이면 브로드캐스트 주소를 나타낸다.

#### [예시]

```
[IP 주소]
192.168.1.10   →  11000000.10101000.00000001.00001010

[서브넷 마스크]
255.255.255.0  →  11111111.11111111.11111111.00000000

[네트워크 주소]
192.168.1.0  →  11000000.10101000.00000001.00000000

[브로드캐스트 주소]
192.168.1.255  →  11000000.10101000.00000001.11111111

[호스트 범위]
- 네트워크 주소, 브로드캐스트 주소 제외
192.168.1.1 ~ 192.168.1.254
```

<br>

## 서브넷 마스크의 역할

같은 네트워크인지 판별하는 것 외에도 여러가지 역할을 수행한다.

#### 1. 같은 네트워크 판별

특정 IP주소와 서브넷 마스크를 AND 연산하면 어떤 네트워크에 속해있는지 판별할 수 있다.
<br>
```
  IP 주소:        192.168.1.10  →  11000000.10101000.00000001.00001010
  서브넷 마스크:  255.255.255.0 →  11111111.11111111.11111111.00000000
  --------------------------------------------------------------
  네트워크 주소:  192.168.1.0   →  11000000.10101000.00000001.00000000
```
<br>

#### 2. 서브네팅(Subnetting)

큰 네트워크를 여러 개의 작은 네트워크로 나눈다.
<br>
서브네팅의 주체는 RIR, ISP, 기업 등 자유롭다.

```
  ex. 기업이 192.168.1.0/24 네트워크를 할당받음

  기업은 효율적으로 사용하기 위해 /26으로 나눔
  → 각 네트워크 당 62개 IP 사용 가능

  192.168.1.0/26    →  IT팀 사용
  192.168.1.64/26   →  인사팀 사용
  192.168.1.128/26  →  영업팀 사용
  192.168.1.192/26  →  연구팀 사용
```

<br>

#### 3. 브로드캐스트 도메인 최소화로 네트워크 성능 향상

브로드캐스트는 한 장치가 같은 네트워크 내의 모든 장치에게 데이터를 전송하는 방식이다.
<br>
네트워크 내 모든 장치가 수신해야 하므로, 수신 대상이 많아질수록 트래픽이 증가해 성능이 감소된다.
<br>
서브네팅을 수행하면 불필요한 트래픽이 감소되므로 네트워크 성능이 향상된다

<br>

#### 4. 라우팅 과정에서 사용

라우터는 서브넷 마스크를 이용해 목적지 네트워크를 판별한다
<br><br>
만약 목적지가 같은 네트워크에 있다면
- 라우트를 거치지 않고 스위치로 직접 전달한다

만약 목적지가 다른 네트워크에 있다면
- 라우터(기본 게이트웨이)로 전달된다

```
  라우팅 예시

  출발지 : 192.168.1.10
  서브넷 마스크 : 255.255.255.0
  네트워크 주소 : 192.168.1.0

  1. 목적지 : 192.168.1.100
  → 동일한 192.168.1.0에 속하므로 직접 통신 가능

  2. 목적지 : 192.168.2.20
  → 다른 네트워크 192.168.2.0에 속하므로 기본 게이트웨이로 패킷이 전달됨
```

<br>

#### 5. 보안 정책 및 접근 제어

서브네팅 예시에서 각 부서별로 서브넷을 나누었다.
<br>
방화벽 등을 사용하여 특정 네트워크 또는 장치에 대한 접근을 제한할 수 있다.

<br>

## 정보처리기사 실기 문제 풀이

<img src="/assets/img/25/01/31/problem.png" style="border-radius:5px" alt="problem" width="400">

> 22년 2회 9번 문제

<br>

1) 네트워크 주소
- 네트워크 주소는 IP주소와 서브넷 마스크를 AND 연산하여 구할 수 있다.
- 서브넷마스크 앞부분이 모두 255이므로 139.127.19.x 임은 한눈에 알 수 있다.
- 132와 192를 2진수로 변환하여 AND 연산을 수행하고 10진수로 바꿔보자

```
132 → 10000100
192 → 11000000

AND → 10000000 → 128
정답 : 128
```

2) 호스트 개수
- 서브넷 마스크에 따르면, 호스트 비트 수는 6이다.
- $2^6 = 64$이고, 네트워크/브로드캐스트 주소를 제외해야 하므로 정답은 62이다.

> 좀 더 문제를 풀어보고 싶다면 [여기](https://velog.io/@pyd6119/%EC%A0%95%EC%B2%98%EA%B8%B0-%EC%8B%A4%EA%B8%B0-%EC%A4%80%EB%B9%84-%EB%84%A4%ED%8A%B8%EC%9B%8C%ED%81%AC) 문제를 풀어보자
