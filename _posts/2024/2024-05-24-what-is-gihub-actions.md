---
title: CI/CD의 새로운 패러다임 - Github Actions
date: 2024-05-24 15:36:00 +09:00
categories: [CI/CD]
tags:
  [
    CI/CD,
    github actions
  ]
---

## CI/CD
- Continous Integration(지속적 통합)과 Continuous Delivery/Deploy(지속적 배포)
- 소프트웨어 개발에서 지속적으로 수행해야하는 반복 작업
- 개발 주기를 단축하고 코드 품질을 높인다.

<br>

## 기존 CI/CD 툴들의 문제점
- github action 등장 전에는 `Jenkins`, `Circle CI`, `Travis CI` 등이 존재했다.
- 낮지 않은 진입장벽, 복잡한 설정 등으로 인해 `CI/CD`가 DevOps 엔지니어들의 전유물로만 여겨지곤 했었다.

> Jenkins로 CI/CD를 구축해봤지만, 아직까지도 어떻게 설정해야하는지 모르겠을 정도다..

<br>

## Github Actions의 등장
- 대부분의 개발자들이 깃허브 계정이 있다고 볼 수 있는 상황에서 Github는 간편한 설정을 내세워 비교적 최근인 `2019년 11월` 출시되었다.
- Github와의 강력한 호환성으로 편리하게 사용이 가능하다.
- 코드 저장소에서 어떤 event가 발생했을 때 특정 작업이 일어나게 하거나 주기적으로 어떤 작업들을 반복해서 실행시킬 수도 있다.

<br>

## Workflows
- 자동화된 전체 프로세스
- 하나 이상의 Job으로 구성되고, Event에 의해 예약되거나 트리거될 수 있는 자동화된 절차이다.
- Workflow는 코드 저장소 내 `.github/workflows` 폴더 아래에 위치한 YAML 파일로 설정한다.
- 하나의 코드 저장소에는 여러 개의 워크플로우(YAML 파일)을 생성할 수 있다.
- 크게 2가지, `on`과 `job`을 정의해야 한다.

<br>

## on 정의 예시
- `on` 속성은 해당 워크플로우가 언제 실행되는지를 정의한다.

<br>

#### [main 브랜치에 push 이벤트가 발생]
```yaml
on:
    push:
        branches: [ "main" ]

jobs:
    # 생략
```

<br>

#### [매일 자정에 워크플로우 실행]
```yaml
on:
    schedule:
        - cron: "0 0 * * *"

jobs:
    # 생략
```

<div class="spotlight2">
cron 은 Unix 계열 운영체제에서의 시간 작업 스케쥴러 이다.
</div>

<br>

## Jobs
- 독립된 가상 머신 또는 컨테이너에서 돌아가는 하나의 처리 단위를 의미
- 모든 작업은 기본적으로 동시에 실행되며, 필요시 작업 간에 의존 관계를 설정하여 작업이 실행되는 순서를 제어할 수도 있다.
- 필수로 들어가야 할 속성은 `runs-on`으로, 리눅스나 윈도우즈와 같은 실행 환경을 지정해줘야 한다.

<br>

#### [Jobs 예시1]
```yaml
# 생략

jobs:
    job1:
        # job1 내용
    job2:
        # job2 내용
    job3:
        # job3 내용
```

<br>

#### [Jobs 예시2 - 실행 환경을 ubuntu로 지정]
```yaml
# 생략

jobs:
    job1:
        runs-on: ubuntu-latest
        steps:
            # 생략
```

<br>

## Steps
- 하나의 작업은 일반적으로 여러 단계의 명령을 순차적으로 실행한다.
- 작업 단계는 단순한 커맨드나 스크립트가 될 수도 있고 좀 더 복잡한 명령인 `action` 이 될 수도 있다.
- 커맨드나 스크립트를 실행할 때는 `run` 속성을, 액션을 사용할 때는 `uses` 속성을 사용한다.
- 각 단계 앞에서는 반드시 `-` 를 붙여줘야 한다.

<br>

#### [Steps 예시 - 자바스크립트 프로젝트 테스트 실행]
```yaml
# 생략

jobs:
    test:
        runs-on: ubuntu-latest
        steps:
            - uses: actions/checkout@v3
            - run: npm install
            - run: npm test
```

<br>

## Actions
- Github Actions의 핵심 기능이다.
- job을 구성하기 위한 step들의 조합으로 구성된 독립적인 명령
- action을 구성하기 위해서 레포지토리와 상호작용하는 커스텀 코드를 만들 수도 있다.
- 사용자가 직접 커스터마이징하거나, **마켓플레이스에 있는 action을 가져다 사용할 수 있다.**

> 바로 위의 예제에서 코드 저장소로부터 작업 실행 환경으로 내려받는 `actions/checkout` 을 사용했다.

<br>

## Runner
- Github Actions Runner 애플리케이션이 설치된 머신으로, Workflow가 실행될 인스턴스
- 사용자의 서버를 사용하는 `Self-hosted runners`, 깃허브의 서버를 사용하는 `GitHub-hosted runners` 가 존재한다.
- 대부분 간편한 `GitHub-hosted runners` 를 사용하는데 `public` 레포일 시, **무료**이다.

<br>

## 누군가가 물어본다면
<div class="spotlight1" markdown="1">
설정이 복잡한 대부분의 CI/CD 툴과 다르게, `GitHub Actions`는 github와의 강력한 호환성을 바탕으로 편리하게 CI/CD 환경을 구성할 수 있습니다.
</div>