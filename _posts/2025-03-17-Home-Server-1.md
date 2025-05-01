---
title: 홈서버 구축 1 - 제품 선택 및 우분투 설치
date: 2025-03-17 12:50:00 +09:00
categories: [홈서버]
tags:
  [
    홈서버,
    미니pc
  ]
---

## 들어가며

클라우드 서버는 편리하지만 비용이 꽤 비싸다.
<br>
AWS 프리티어, GCP 무료 크레딧 같은 정책들이 있지만 성능도 안좋고 매번 새로운 클라우드 배포 환경을 구성하는 것도 귀찮은일이다.
<br>
이번 포스팅에서는 어떤 제품을 선택했는지 소개하려 한다.

<br>

## 서버랙 vs 미니PC

홈서버 구축에는 크게 두 종류로 나눌 수 있다.

> 데스크톱과 비슷한 워크스테이션도 있다.

| 항목       | 서버랙                 | 미니 PC                 |
|------------|----------------------|----------------------|
| 크기       | 큼 | 작음    |
| 성능       | 고성능 (확장 가능)   | 중저성능  |
| 전력 소비  | 높음                 | 낮음                 |
| 소음       | 큼                   | 적음                 |
| 가격       | 비쌈                 | 저렴함               |
| 유지보수   | 전문적 관리 필요     | 쉬움                 |
| 활용 용도  | 데이터센터, 기업용   | 개인, 소규모 서버   |

<div style="display: flex; justify-content: space-between; align-items: center; width: 800px">
  <img src="/assets/img/25/03/17/rack.png" style="border-radius:5px" alt="rack" width="380">
  <img src="/assets/img/25/03/17/mini.png" style="border-radius:5px" alt="mini" width="380">
</div>

미니PC는 한손에 들어올 정도로 컴팩트한 제품이고, 서버랙은 여러 장치(전원유지장치, 허브 등)를 연결해 고성능을 제공할 뿐만 아니라 정전에도 대비할 수 있는 제품이다.
<br>
세팅 시간에 있어서도 서버랙이 압도적으로 복잡하고 많은 시간이 소요된다.
<br>
사이드 프로젝트 수준에서는 사실상 미니PC로도 충분하다.

<br>

## 미니PC 모델 선택

미니PC에서 제일 가성비 좋기로 유명한 녀석은 `인텔의 N100`이다.
<br>
이 중에서 홈서버로도 자주 쓰이는 모델군은 Firebat, Chatreey다.
<br>
나는 `Chatreey t9`을 선택했고, 쿠팡 직구를 통해 약 12만원에 베어본을 구매했다.
<br>

#### Chatreey t9 후기

후기를 간략하게 적어보자면, 미니PC답게 컴팩트한 크기가 좋았고 디자인도 깔끔하니 마음에 들었다.
<br>
다만 소음이 없는 것은 아니여서 원룸에서 잘 때 조금 거슬리긴 하다.
<br>
소음에 예민하다면 추가 조치가 필요하다.

<br>

## SSD, 램 추가 구입

바로 윈도우로 작동할 수 있는 옵션도 있는데, 중국 백도어 의혹도 있고 윈도우 OS는 필요없기도 해서 베어본만 산 후 SSD와 램을 따로 구매했다.
<br>
총 20만원 정도가 들었다. 좀 비싸도 어디건지 모르는 부품들보단 삼성 제품이 더 믿음직하다.
<br><br>
SSD - 삼성 PM9A1 M.2 NVMe SSD 256GB
<br>
RAM - 삼성 DDR4 PC4-25600 노트북용 램 16GB
<br><br>
사실 램슬롯이 2개인줄 알고 8GB 2개로 샀는데 하나였다..

> 사진과 헷갈리지 말고 나처럼 2개 사지 말기를 바란다. ~~GPT 혼날래?~~

<br>

## 비용 비교

미니PC 초기 구입 비용은 20만원이다.
<br>
한달 소비 전력은 약 9kWh이고, 2천원 정도다.
<br>
유사한 스펙을 가진 AWS EC2의 `t4g.xlarge` 비용은 한달 기준 12만원이다.
<br>
홈서버를 2달만 쓰더라도 이득을 보는셈이다.

<br>

## 조립 과정

<img src="/assets/img/25/03/17/make1.jpg" style="border-radius:5px" alt="make1" width="300">

두근두근 박스깡

<br>

<img src="/assets/img/25/03/17/make2.jpg" style="border-radius:5px" alt="make2" width="350">

작다는 것에 의심이 갔었는데 진짜 작긴 했다.

<br>

<img src="/assets/img/25/03/17/make3.jpg" style="border-radius:5px" alt="make3" width="350">

드라이버로 낑낑대며 나사를 풀었다. 드릴 있으면 좋을듯
<br>
오밀조밀하게 들어가있는 부품들이 보기 좋다.

<br>

<img src="/assets/img/25/03/17/make4.jpg" style="border-radius:5px" alt="make4" width="350">

NVMe SSD를 구입했으므로 맞는 위치에 딱 소리나게 끼워주고 나사로 조여준다.

<br>

<img src="/assets/img/25/03/17/make5.jpg" style="border-radius:5px" alt="make5" width="350">

사진은 램2개지만.. 16GB짜리 램 하나를 끼워주면 완성이다.

<br>

<img src="/assets/img/25/03/17/make6.jpg" style="border-radius:5px" alt="make6" width="500">

그대로 부팅하면 OS가 없는 상태이므로 위와 같은 화면이 뜬다.

<br>

## Ubuntu OS 깔기

우분투 까는 법은 워낙 많아서 [여기](https://hayden-igm.tistory.com/87?category=1191260)를 참고해서 네트워크 설정, SSH 설치까지 완료해주자.

> 와이파이를 사용할 수도 있지만, 안정적인 통신을 위해 공유기와 랜선 연결을 해주자.

<br>

SSH 터미널로 서버를 관리할 것이기 때문에 필요없긴 하지만, **한/영 설정**을 하고 싶다면 [여기](https://andrewpage.tistory.com/390)를 참고하면 되겠다.

<img src="/assets/img/25/03/17/make7.jpg" style="border-radius:5px" alt="make7" width="700">
