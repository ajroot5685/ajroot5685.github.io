---
title: 홈서버 구축 4 - Nginx로 트래픽 분배하기
date: 2025-04-08 16:22:00 +09:00
categories: [홈서버]
tags:
  [
    홈서버,
    미니pc,
    nginx,
    ssl
  ]
---

## 들어가며

홈서버에서 여러 프로젝트로 트래픽을 분배하려면 웹서버가 유용하다.
<br>
물론 포트포워딩 설정으로 하나하나 직접적으로 연결할 수도 있겠지만 웹서버인 nginx를 이용하는 것이 훨씬 편리하다.
<br>
이번 포스팅에서는 내가 사용한 nginx의 기능들과 필요한 설정들을 공유한다.

<br>

## Nginx

> 빠르고, 가볍고, 정적 파일 서빙과 트래픽 분산에 강한 웹서버이다.
{: .prompt-info }

아파치 웹서버를 구조적으로 개선하여 더 많은 동시 커넥션 양을 가질 수 있고 기능도 다양하다.

<br>

## 주로 사용한 기능들

정말 다양한 기능들이 많지만, 홈서버를 운영하면서 내가 사용한 기능들을 소개하겠다.

<br>

#### 리버스 프록시

주로 nginx를 쓰는 이유의 대부분은 리버스 프록시로 활용하기 위함일 것이다.

> **리버스 프록시**는 클라이언트 요청을 대신 받아 백엔드 서버로 전달하고, 응답도 대신 전달하는 중개 서버이다.
{: .prompt-info }

리버스 프록시 역할을 수행함으로써 얻는 부가 기능들에는 `로드밸런싱, SSL 처리, 캐싱, 보안 제어, 경로 기반 라우팅`들이 있다.

<br>

#### 로드밸런서

로드밸런서는 클라이언트의 요청을 여러 서버에 분산시켜 부하를 줄이고, 성능과 가용성을 높이는 기능이다.
<br>
사실 위의 목적보다는 동일한 공유기 도메인으로 알맞는 다른 프로젝트들로 요청을 분배하기 위한 목적으로 사용한다.

<img src="/assets/img/25/04/08/load_balancer.png" style="border-radius:5px" alt="load_balancer" width="800">

> 당연히 포트포워딩 정보에 따라 실제 목적지 포트 번호는 다를 수 있다.

> 서브도메인을 다르게 두지 않고 포트번호를 다르게 둔 이유는 `freedns`에서 무료로는 서브도메인을 이용할 수 없기 때문이다.

<br>
nginx는 4계층과 7계층을 동시에 지원한다.
<br>
AWS의 `Network LoadBalancer`는 4계층만 지원하고, `Application LoadBalancer`는 7계층만 지원하므로 nginx는 더 유연하게 트래픽을 관리할 수 있다.
<br>
보통 7계층인 `server` 블록 설정을 통해 웹서버로 요청들을 전달하게 되고, 4계층은 `stream` 블록 설정을 통해 HTTP 통신이 아닌 DB와 같은 TCP 통신을 매개한다.

<br>

#### SSL

<img src="/assets/img/25/04/08/ssl.png" style="border-radius:5px" alt="ssl" width="500">

HTTPS 통신을 위해 TLS 인증서를 적용한다.
<br>
이를 통해 전송 중 암호화(Encryption in transit)를 달성한다.
<br>
보통 가벼운 프로젝트들은 이 용도로 nginx을 채택하는 경우가 많다.
<br>
애플리케이션에 직접 SSL을 적용하는 것은 귀찮기도 하고, 확장성 측면에서 매우 좋지 않기 때문이다.
<br><br>
클라이언트와 nginx 간 SSL 설정을 하면, nginx는 복호화를 수행하여 서버와 HTTP 통신을 수행한다.
<br>
이를 `SSL 종료(Termination)`이라 한다.
<br>
백엔드 서버에서는 SSL을 설정할 필요가 없고, 성능도 좋아진다.

<br>

#### 보안 강화

nginx를 리버스 프록시로 사용하면 보안이 향상된다.

1. 클라이언트는 nginx와만 통신하므로 백엔드 정보를 알 수 없다.
2. 특정 IP를 차단할 수 있다.
3. 간단한 DoS 공격을 완화할 수 있다.
4. 여러 헤더를 이용해 브라우저 공격을 방지할 수 있다.

<br>

## nginx 설치

이제 편리하게 웹서버를 구성하기 위한 설정들을 공유하겠다.
<br>
`nginx.conf` 파일 하나로 관리하면 결합성이 높아져 관리가 점점 어려워지므로, `sites-available` 디렉토리와 **심볼릭 링크**를 이용해서 서로 다른 프로젝트의 설정 파일을 분리시킨다.

```bash
sudo apt update
sudo apt install nginx -y

# 만약 4계층 트래픽 전달을 위해 stream 블록이 필요하다면
sudo apt update
sudo apt install nginx-full -y
```

> `stream` 블록은 DB와의 커넥션(DB 툴)을 위해 추후에 사용할 예정이다.

<br>

## 기본적인 설정

nginx를 설치하면 `/etc/nginx/nginx.conf` 파일에 기본적인 설정들이 있을 것이다.
<br>
우리는 `http` 블록 안에 `include /etc/nginx/sites-enabled/*;` 구문을 그대로 활용해서 추가 설정들을 할 것이다.
<br>
이외에 기본적인 공통 설정들은 `nginx.conf`에 추가해주어야 한다.

```conf
.
.

http {
    .
    .
    # Basic Settings
    autoindex off;
    disable_symlinks on;
    client_max_body_size 20M;

    # Logging Settings
    access_log /var/log/nginx/access.log combined;

    # SSL Settings
    ssl_protocols TLSv1.2 TLSv1.3
}
```

**autoindex**

`on`이면 기본 인덱스 파일이 없을 때 디렉토리 안의 파일 목록이 클라이언트에게 노출된다.

<br>

**disable_symlinks**

클라이언트는 기본적으로 nginx에서 설정된 웹루트 디렉토리 하위에만 접근할 수 있다.
- 웹루트가 `/var/www/project` 로 지정되어 있으면 `/var/www` 나 `/etc/nginx` 등에는 접근할 수 없다는 뜻이다.

하지만 심볼릭 링크를 사용하면 다른 경로에도 접근이 가능하다.

> **심볼릭 링크**
> 
> 다른 파일이나 디렉토리를 가리키는 특수 파일. 윈도우의 **바로가기**와 비슷한 개념이다.
{: .prompt-info }

따라서 웹루트 안에만 접근할 수 있도록 `disable_symlinks` 옵션을 `on`으로 설정해주어야 한다.

<br>

**client_max_body_size**

클라이언트가 너무 무거운 요청들을 보내면 서버가 느려질 수 있다.
<br>
이를 막기 위해 바디 사이즈를 제한한다.
<br>
파일 용량을 더 허용해야 하는 경우에 해당 프로젝트 파일에 추가로 설정해주자.

> nginx는 파일을 위에서부터 아래로 읽어 마지막에 선언된 설정이 우선 적용된다.

<br>

**access_log combined**

`combined` 옵션은 IP, 요청 URL, 브라우저 정보 등이 포함된 자세한 형식으로 기록하게 한다.

```
192.168.0.1 - - [08/Apr/2025:10:00:00 +0000] "GET / HTTP/1.1" 200 612 "-" "Mozilla/5.0 ..."
```

<br>

**ssl_protocols**

tls1.0과 tls1.1 버전은 취약점이 발견되었다고 한다.

[TLS 1.0, TLS 1.1 버전이 취약한 이유](https://eagerprotector.tistory.com/57)

<br>

## 실행파일 다운 및 실행 방지

공격자가 웹루트에 실행 가능한 스크립트 파일을 업로드하고, 이를 실행하여 서버를 공격할 수 있다.
<br>
이를 막기 위해서는 특정 확장자에 대한 접근을 아예 차단시켜야만 한다.
<br>

```conf
# sudo vim /etc/nginx/global_security.conf

location ~* \.(php|sh|pl|py|cgi)$ {
    deny all;
}
```

conf 파일을 정의해놓고 뒤에서 `include`로 넣어줄 것이다.

<br>

## Debian-style Nginx Config

각 설정에 앞서 nginx에 설정을 추가하는 방식을 설명한다.
<br>
심볼릭 링크를 이용한 이 방식은 `sites-available`에 각 설정들을 담고, `sites-enabled`에 심볼릭 링크로 활성화시켜서 `nginx.conf`에서 이를 포함한다.
<br>
설정 관리도 편하고, `unlink`를 통해 비활성화도 쉽게 할 수 있다는 장점을 가지고 있다.
<br>
7계층인 `http` 블록 안의 `server` 블록 각각을 `sites-available`에 나눠서 관리한다.

<img src="/assets/img/25/04/08/sites_available.png" style="border-radius:5px" alt="sites_available" width="700">

위 이미지에서 프로젝트1, 프로젝트2의 심볼릭 링크만 있으므로 프로젝트3 설정은 활성화되지 않는다.

<br>

## 테스트 사이트 추가

#### 웹루트 디렉토리 생성

```bash
sudo mkdir /var/www/test
```

<br>

#### 기본 웹페이지 추가

```html
<!-- sudo vim /var/www/test/index.html -->

<h1>Welcome to test</h1>
```

<br>

#### 사이트 설정

```nginx
# sudo vim /etc/nginx/sites-available/test

server {
    listen 10000;
    server_name domain.mooo.com;

    root /var/www/test; # 웹루트 지정
    index index.html; # 웹루트 안에 있는 기본 파일

    # 요청 파일, 디렉토리 없으면 404 반환
    location / {
        try_files $uri $uri/ =404;
    }

    include /etc/nginx/global_security.conf; # 위에서 정의한 스크립트 실행 방지 파일 포함
}
```

<br>

#### 설정 적용

```bash
sudo ln -s /etc/nginx/sites-available/test /etc/nginx/sites-enabled/test # 상대 경로가 아닌 절대 경로를 설정해야함

sudo nginx -t # 설정 테스트, 오류 있을 시 고친 후 진행

sudo systemctl reload nginx # 설정 반영, 메모리로 올림

sudo ufw allow 10000 # 해당 사이트에 대한 방화벽 허용
```

> 추가로 포트포워딩 설정이 필요하다.

<img src="/assets/img/25/04/08/test.png" style="border-radius:5px" alt="test" width="400">

<br>

## SSL 설정

같은 도메인으로 포트만 나누어 사용하기 때문에 해당 도메인에 대한 인증서만 추가하면 된다.

> 여러 서브도메인을 사용하는 경우, 각각에 대해 SSL 설정을 해줘야만 한다.

테스트 사이트를 추가했던 것처럼, SSL 전용 사이트를 추가한다.
<br>
Let's encrypt는 인증 시 기본 HTTP 포트(80)를 사용한다.

```nginx
# sudo vim /etc/nginx/sites-available/test

server {
    listen 80; # 80 고정
    server_name domain.mooo.com;

    location / {
        try_files $uri $uri/ =404;
    }

    # 인증 경로
    location /.well-known/acme-challenge/ {
        root /var/www/certbot;
    }

    include /etc/nginx/global_security.conf;
}
```

마찬가지로 심볼릭 링크 설정, nginx 설정 반영, 방화벽, 포트포워딩 설정 등을 마친 후 `certbot`으로 인증을 시도한다.

```bash
# certbot 설치
sudo snap install core
sudo snap install --classic certbot

# 심볼릭 링크 연결
sudo ln -s /snap/bin/certbot /usr/bin/certbot

# 인증 시도
sudo certbot certonly --webroot -w /var/www/certbot -d domain.mooo.com
```

<br>

## 사이트 SSL 설정 추가

성공적으로 인증되었다면, 이제 HTTPS로 통신할 수 있도록 conf 파일을 수정해야 한다.

```nginx
# sudo vim /etc/nginx/sites-available/test

server {
    listen 10000 ssl; # ssl 추가 - HTTPS 요청 강제
    server_name domain.mooo.com;

    root /var/www/test;
    index index.html;

    location / {
        proxy_pass http://localhost:15000; # 실제 내부 서버 포트
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }

    # 인증서 파일
    ssl_certificate /etc/letsencrypt/live/domain.mooo.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/domain.mooo.com/privkey.pem;

    include /etc/nginx/global_security.conf;
}
```

> nginx reload 수행

<img src="/assets/img/25/04/08/ssl_complete.png" style="border-radius:5px" alt="ssl_complete" width="300">

<br>

## 인증서 자동 갱신

Let's encrypt의 인증서 유효기간은 90일이므로 자동으로 갱신되도록 설정해줘야 한다.
<br>
간편하게 자동 갱신 설정을 할 수 있도록 스크립트를 만들었다.
<br>
그대로 사용해도 되고, 좀 더 커스텀해서 사용해도 된다.

<br>

#### 구성

3개의 스크립트를 제공한다.
<br>
인증을 위해서 80포트를 수신하는 server 블록을 추가해야 한다.
<br>
Let's Encrypt는 인증용 경로로 `.well-known`을 사용한다.
<br>
이를 계속 열어두면 인지하지 못한 취약점 때문에 문제가 발생할 수 있다.
<br>
따라서 평소에는 비활성화 시켜두었다가, `crontab`으로 갱신할 때에만 활성화시켜서 안전하게 처리한다.
<br><br>
이를 위해 포트포워딩은 80포트로 미리 열어두어야 한다.
<br>
비활성화된 평소에는 해당 포트로 요청해도 타임아웃이 발생하므로 안전하다.

<br>

#### enable_site.sh

사이트를 활성화시키는 스크립트다.
<br>
심볼릭 링크를 연결하고 nginx reload를 수행한다.
<br><br>

실행 예시 : `./enable_site.sh test`

```shell
#!/bin/bash

# 인자로 사이트 이름을 입력받음
if [ -z "$1" ]; then
    echo "❌ 사용법: $0 <사이트이름>"
    exit 1
fi

SITE_NAME=$1
NGINX_ENABLED="/etc/nginx/sites-enabled/$SITE_NAME"
NGINX_AVAILABLE="/etc/nginx/sites-available/$SITE_NAME"

echo "🚀 Nginx 심볼릭 링크 연결 중: $SITE_NAME"

if [ -L "$NGINX_ENABLED" ]; then
    sudo unlink "$NGINX_ENABLED"
fi
sudo ln -s "$NGINX_AVAILABLE" "$NGINX_ENABLED"

if sudo nginx -t; then
    sudo systemctl reload nginx
    echo "🎉 $SITE_NAME 활성화 완료!"
else
    echo "❌ Nginx 설정 테스트 실패, Nginx 설정을 확인하세요."
    exit 1
fi
```

<br>

#### disable_site.sh

사이트를 비활성화시키는 스크립트다.
<br>
심볼릭 링크를 삭제하고 nginx reload를 수행한다.
<br><br>

실행 예시 : `./disable_site.sh test`

<br>

#### certbot_renew.sh

crontab에 등록할 자동 갱신 스크립트다.
<br>
실행 흐름은 다음과 같다.

1. 사이트를 활성화시킨다.
2. 80포트 방화벽을 개방한다.
3. 인증서를 갱신한다.
4. 80포트 방화벽을 닫는다.
5. 사이트를 비활성화 시킨다.

상위에 선언된 변수들이 설정되었는지 잘 확인하기 바란다.

```shell
#!/bin/bash

# 로그 파일 경로
LOGFILE="/var/log/certbot/renew.log"
SITE_NAME="ssl"
NGINX_SITE_AVAILABLE_PATH="/etc/nginx/sites-available"
ENABLE_SITE_SCRIPT_PATH=""
DISABLE_SITE_SCRIPT_PATH=""

if ! sudo find "$NGINX_SITE_AVAILABLE_PATH" -maxdepth 1 -name "$SITE_NAME" | grep -q .; then
    echo "❌ Let's encrypt의 HTTP-01 인증을 위해 $SITE_NAME server block이 필요합니다."
    exit 1
fi

if [ -z "$ENABLE_SITE_SCRIPT_PATH" ] || [ -z "$DISABLE_SITE_SCRIPT_PATH" ]; then
    echo "❌ 스크립트의 경로를 설정해주세요."
    exit 1
fi

if ! "$ENABLE_SITE_SCRIPT_PATH" "$SITE_NAME"; then
    exit 1
fi

# 날짜 출력
echo "[$(date)] Starting certbot renew process..." >> "$LOGFILE"

# Certbot 갱신
sudo ufw allow 80
sudo certbot renew >> "$LOGFILE" 2>&1
sudo ufw delete allow 80

if ! "$DISABLE_SITE_SCRIPT_PATH" "$SITE_NAME"; then
    exit 1
fi
```

<br>

#### crontab 등록

`sudo crontab -e`를 통해 배치 작업을 등록하자.

> sudo를 빼면 현재 사용자에 대한 작업을 등록한다.

```
0 3 * * 0 /opt/certbot/certbot_renew.sh # 매주 일요일 오전 3시에 실행
```

<br>

## 기타

홈서버를 구축하면서 편리하게 관리하기 위해 여러 기능을 묶은 스크립트를 만들었다.
<br>
그대로 사용해도 되고, 자신만의 커스텀 스크립트를 만들어도 된다.

<https://github.com/ajroot5685/Script/tree/main/nginx>

<br>

---

위 방법들을 이용해서 API 서버를 운영하고 있다.

<https://manittomaker.com/>