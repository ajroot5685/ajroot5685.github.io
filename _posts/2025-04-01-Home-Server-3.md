---
title: 홈서버 구축 3 - ssh 설정(fail2ban)
date: 2025-04-01 20:47:00 +09:00
categories: [홈서버]
tags:
  [
    홈서버,
    미니pc,
    ssh,
    fail2ban
  ]
---

## 들어가며

공유기 기본 설정이 끝났다면 이번에는 편하게 서버를 관리하기 위한 SSH 설정을 해줄 것이다.
<br>
보안을 위한 공개키+OTP 인증방식과 더불어 fail2ban으로 블랙리스트 관리까지 진행한다.

<br>

## SSH

> Secure SHell 은 네트워크를 통해 다른 컴퓨터에 안전하게 접근하고 명령어를 실행할 수 있게 해주는 프로토콜이다.
{: .prompt-info }

`sudo` 명령어를 통해 시스템을 자유롭게 관리할 수 있기 때문에, SSH에 대한 보안 수준을 엄격하게 끌어올려야 한다.

<br>

#### SSH 설치 명령어

```bash
sudo apt update
sudo apt install openssh-server
```

<br>

## 포트 변경

가장 중요하고 간편하다.
<br>
잘 알려진 포트인 22번 포트에서 만번대의 다른 포트로 바꾸기만 해도 외부 침입 시도횟수가 현저히 낮아진다.
<br><br>
`/etc/ssh/sshd_config` 에서 원하는 포트로 설정할 수 있다.

```shell
#Port 22
Port 10000 # 예시로 10000번으로 설정했다고 가정한다.
```

> ssh는 다른 서버로 접속할 때의 설정이고, sshd는 클라이언트의 접속 요청을 어떻게 처리할지의 설정이다.
{: .prompt-info }

<br>

#### 방화벽 설정

외부에서 해당 포트로 접근할 수 있도록 열어줘야 한다.
<br>
간편한 `ufw`를 사용하자

```bash
# 설치
sudo apt update
sudo apt install ufw

# 방화벽 허용
sudo ufw allow 10000
```

<br>

#### 공유기 포트포워딩

공유기 하위에 홈서버가 연결되어 있으므로 당연히 공유기의 포트포워딩을 설정해주어야 한다.
<br>
이전 포스팅을 참고해서 포트포워딩을 진행한다.

<br>

## 공개키 인증 설정

먼저 공개키 인증부터 설정한다.
<br>
공개키 인증으로 바꾸려면 설정도 변경해야 하지만 OTP 인증 때 같이 바꾼다.

<br>

#### 1. SSH 키 생성

홈서버에 접속할 클라이언트에서 SSH 키를 생성한다.
<br>

```bash
ssh-keygen -t rsa -b 4096
```

<br>

#### 2. 공개키를 서버에 복사

`~/.ssh`에 생성된 공개키 `id_rsa.pub`를 서버에 전달할 것이다.

```bash
# 기본형(22번 포트 사용)
ssh-copy-id <유저명>@<ip주소>

# 포트 지정
ssh-copy-id -p <포트번호> <유저명>@<ip주소>

# 예시
ssh-copy-id -p 10000 ubuntu@192.168.100.0
```

`ssh-copy-id` 명령어는 서버의 `~/.ssh/authorized_keys`에 클라이언트의 공개키를 추가한다.

<br>

## OTP 인증 설정

#### 1. Google Authenticator 설치

구글 앱을 이용한 OTP 인증 방식이다.
<br>
휴대폰에 설치된 앱을 통해 일회용 비밀번호를 자동발급 받는다.

```bash
sudo apt update
sudo apt install libpam-google-authenticator
```

<br>

#### 2. OTP 설정

QR코드가 출력되기 때문에 터미널 창을 최대한 크게 늘려준다.
<br>
그 후, 아래 명령어를 통해 OTP를 설정한다.

```bash
# 기본 명령어
google-authenticator

# 앱에 뜨는 이름 설정
google-authenticator --issuer="홈서버 SSH"
```

> 공개키 + OTP 설정을 한 후에 새로운 클라이언트를 등록하려면 이미 등록되어 있는 클라이언트에서 공개키를 받아 대신 등록해주자.

<br>

## SSH 설정

이제 SSH의 설정 파일을 업데이트해서 공개키+OTP 이중 인증 방식을 적용해보자
<br>
OTP는 PAM(Pluggable Authentication Modules)을 사용해서 적용한다.

```shell
# sudo vim /etc/ssh/sshd_config

PermitRootLogin no # 루트 유저 로그인을 금지한다.

PasswordAuthentication no # 비밀번호 방식을 비활성화한다.
PubkeyAuthentication yes # 공개키 인증을 활성화한다.
KbdInteractiveAuthentication yes # 키보드 상호작용 인증을 활성화한다.
UsePAM yes # PAM을 사용한다.
AuthenticationMethods publickey,keyboard-interactive # 공개키 + 키보드 다중 인증 방식을 사용한다.
```

<br>

## PAM 설정

Google Authenticator의 OTP는 키보드 상호작용 인증 방식에 속한다.
<br>
사용자가 인증을 하는 동안 서버가 클라이언트에게 입력을 요청한다.
<br>
즉, 앱을 통해 생성된 일회용 비밀번호를 입력하도록 요구한다.

```shell
# sudo vim /etc/pam.d/sshd

# 다음 코드들은 주석처리 해야한다. 그렇지 않으면, 다른 인증 방식을 무시할 수 있다.
# @include common-auth
# @include common-password

# 다음 코드를 맨 아래에 추가한다.
auth required pam_google_authenticator.so
```

<br>

## 설정 확인

SSH 서비스를 재시작해서 설정이 잘 적용되었는지 확인한다.
<br>
1차적으로 서버에 클라이언트의 공개키가 있는 경우에 통과하고, 2차적으로 OTP 비밀번호를 입력해야만 접속할 수 있다.

```bash
sudo systemctl restart ssh
```

> 우분투 24.04에서 설치하는 ssh에는 sshd가 통합되어 있다. 따라서 sshd가 아닌 ssh를 재시작한다.

<img src="/assets/img/25/04/01/ssh.png" style="border-radius:5px" alt="ssh" width="400">

<br>

## fail2ban

이중 인증 방식만으로도 충분히 안전하지만, 비정상적으로 수없이 접속을 시도하는 경우도 있을 수 있다.
<br>
따라서 블랙리스트를 관리하여 일정 횟수 이상 접속 실패하면 시도조차 못하게 막아버리자.

#### 1. 설치

```bash
sudo apt update
sudo apt install fail2ban
```

<br>

#### 2. 서비스 시작 및 활성화

```bash
sudo systemctl start fail2ban
sudo systemctl enable fail2ban
```

<br>

#### 3. fail2ban 설정 파일 수정

설정량이 꽤 많기 때문에 `jail.conf` 파일에 직접 수정하지 않고, `jail.local`을 사용하여 설정을 관리한다.
<br>
경로는 `/etc/fail2ban` 이다.

> 만약 의도대로 적용되지 않는 설정이 있다면 `jail.local` 뿐 아니라, `jail.conf`까지 살펴봐야한다.

```shell
[ssh]
enabled = true
backend=systemd # journalctl을 사용하여 systemd 로그를 읽는다.

port=10000 # ssh 포트 설정
filter=sshd # fail2ban에 템플릿으로 있는 sshd 필터다. 그대로 사용하면 되겠다.
logpath=/var/log/auth.log # 로그 경로
maxretry=5 # 허용 시도 횟수
findtime=10m # 판별 기간, 즉 10분 동안 5번 실패하면 블랙 처리
bantime=1h # 밴 시간
```

> 필터는 `/etc/fail2ban/filter.d` 에서 확인할 수 있다.

<br>

#### 4. 블랙 처리 시 이메일 발송

블랙 처리 되었을 때 이메일로 그 결과를 통보받는다면 빠르게 해당 ip에 대한 추가 조치를 수행할 수 있다.

```shell
[ssh]
enabled = true
backend=systemd

port=10000
filter=sshd
logpath=/var/log/auth.log
maxretry=5
findtime=10m
bantime=1h

action=%(action_mwl)s # 아래 설명
destemail=<목적지 메일 주소> # action 변수 1
sender=<보내는이 도메인> # action 변수 2
mta=sendmail # action 변수 3
```

<br>

**action_mwl**

`jail.conf`에 정의되어 있다.

```shell
# ban & send an e-mail with whois report and relevant log lines
# to the destemail.
action_mwl = %(action_)s
             %(mta)s-whois-lines[sender="%(sender)s", dest="%(destemail)s", logpath="%(logpath)s", chain="%(chain)s"]
```

`%(mta)s-whois-lines`에서 mta에 우리가 지정한 `sendmail` 변수가 들어가면서 `sendmail-whois-lines.conf`이 된다.
<br>
이 명령어는 `/etc/fail2ban/action.d` 경로에 있는 `sendmail-whois-lines.conf`를 호출하게 된다.
<br><br>
sendmail은 리눅스에 대부분 기본적으로 내장된 메일 전송 에이전트이다.
<br>
더 편리한 `postfix`가 있지만, fail2ban에서 sendmail에 대한 템플릿을 제공하기 때문에 더 간편하다.

<br>

**sendmail-common.conf**

`sendmail-whois-lines.conf`에서 내부적으로 포함하는 파일이다.
<br>
해당 파일에는 아래의 메일 명령어가 선언되어 있다.

```shell
mailcmd = /usr/sbin/sendmail -f "<sender>" "<dest>"
```

sendmail의 기본 경로가 `/usr/sbin/sendmail`이기 때문에 우리는 별다른 설정 없이 가져다 쓰기만 하면 되는 것이다.

<br>

#### 5. 설정 적용

fail2ban 서비스를 재시작하고, 정상 작동 되는지 확인한다.

```bash
sudo systemctl restart fail2ban
sudo systemctl status fail2ban
```

OTP를 여러번 틀려 밴처리 되었다면, 응답하지 않고 타임아웃 처리된다.

<img src="/assets/img/25/04/01/timeout.png" style="border-radius:5px" alt="timeout" width="600">

<br>

#### fail2ban 명령어

```bash
sudo fail2ban-client status ssh # 차단 현황 확인

sudo tail -f /var/log/fail2ban.log # 로그 확인

sudo fail2ban-client set ssh banip <IP주소> # 해당 IP 차단

sudo fail2ban-client set ssh unbanip <IP주소> # 해당 IP 차단 해제
```

<br>

## 마무리

여러 설정들을 통해 SSH를 안전하게 관리하고, 웬만하면 다른 사람들에게 SSH 접속 허용하지는 말자.
<br>
fail2ban 블랙리스트 외에도 한국 IP만 허용하여 보안 수준을 더 높일 수 있지만, 간단하고 그다지 효과가 좋지는 않기에 생략하겠다.