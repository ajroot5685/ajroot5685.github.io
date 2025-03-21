---
title: jekyll의 chirpy 테마 적용 - 첫포스팅
date: 2024-03-15 19:00:00 +09:00
categories: [blog]
tags:
  [
    blog,
    jekyll,
    chirpy
  ]
---

> 블로그 포스팅을 시작하기 위해 github 블로그를 선택했고 정적 사이트 생성기인 jekyll과 chirpy 를 사용하게 되었습니다.
> 구체적인 진행 과정은 제가 참고한 블로그들을 링크하고 따로 언급하지 않겠습니다.
> 이 포스팅에서는 각 도구를 사용한 이유와 chirpy를 이용해 블로그 생성까지 발생했던 오류를 다룹니다.

## Github 블로그를 선택한 이유

다음과 같은 특색 때문에 github 블로그를 고르게 되었습니다.

<br>

**자유로운 커스터마이징**

- jekyll을 사용하면 블로그에 필요한 최소한의 기능을 제공받습니다.
- 모든 디자인 요소를 취향껏 변경할 수 있습니다.
- 그러나 댓글 기능, 조회수 등 부가적인 기능들은 직접 구현해야 합니다.

<br>

**MarkDown 형식**

- markdown 문법으로 개발자같이(?) 글을 작성합니다.
- github에서 프로젝트에 대한 설명으로 자주 쓰는 README.md 연습에도 좋습니다.

<br>

**Github와의 연동**

- 다른 블로그들과 달리 사이트에서 버튼을 통해 글을 작성하지 않고 코드 편집기에서 직접 작성 후 커밋&푸시합니다.
- 때문에 예쁜 잔디를 채울 수 있습니다.

<br>

**다른 블로그를 선택하지 않은 이유**

- 대중적이지만 그만큼 개성이 떨어집니다.
- 디자인이 마음에 들지 않습니다.

<div class="spotlight1">
🤔 요약하자면, 개발자 친화적이고, 내가 원하는 디자인, 기능들을 자유롭게 넣을 수 있기 때문에 github 블로그를 선택하였습니다.
</div>

<br>

## chirpy 테마 적용까지 참고한 사이트

[Github 블로그 만들기 (1)](https://devpro.kr/posts/Github-%EB%B8%94%EB%A1%9C%EA%B7%B8-%EB%A7%8C%EB%93%A4%EA%B8%B0-(1)/)

[초보자를 위한 GitHub Blog 만들기 - 1](https://wlqmffl0102.github.io/posts/Making-Git-blogs-for-beginners-1/)

<br>

## 발생했던 오류들

`자동 배포 실패`

- github pages에서 main 브랜치가 아닌 다른 브랜치를 생성하여 설정해야합니다.

  [Github Pages 배포 에러](https://shirohoo.github.io/debugging/2021-07-04-debugging-10/)

<br>

`dist 경로 파일을 찾을 수 없음`
```
internal script /assets/js/dist/misc.min.js does not exist (line 1)
- _site/categories/서브-카테고리/index.html
```
- .gitignore에 설정되어있는 `assets/js/dist` 를 주석처리 합니다.

  [ToC is not working #1090](https://github.com/cotes2020/jekyll-theme-chirpy/issues/1090)

<br>

## 추가기능

**왼쪽 아래 인스타그램 추가**

- `_date` - `contact.yml` 파일에서 수정할 수 있습니다.

  ```yaml
  - type: github
    icon: "fab fa-github"

  # - type: twitter
  #   icon: "fa-brands fa-x-twitter"

  - type: instagram
    icon: 'fab fa-instagram'
    url: 'https://www.instagram.com/stable_root/'

  - type: email
    icon: "fas fa-envelope"
    noblank: true # open link in current tab

  # - type: rss
  #   icon: "fas fa-rss"
  #   noblank: true
  ```

<br>

**댓글 기능**

- 깃허브의 이슈를 댓글처럼 띄워주는 utterances 를 사용했습니다.

  [Jekyll 테마에 utterances 댓글 연동하기](https://www.irgroup.org/posts/utternace-comments-system/)

<br>

**조회수 기능**

- 깃허브 프로필을 꾸밀 때에도 자주 사용되는 Hits를 사용했습니다.

  [지킬 블로그 게시물에 조회수 붙이기 with Hits](https://datainclude.me/posts/%EC%A7%80%ED%82%AC_%EB%B8%94%EB%A1%9C%EA%B7%B8_%EA%B2%8C%EC%8B%9C%EB%AC%BC%EC%97%90_%EC%A1%B0%ED%9A%8C%EC%88%98_%EB%B6%99%EC%9D%B4%EA%B8%B0_with_Hits/)

<br>

**SEO 작업**

- jekyll의 플러그인을 사용하여 자동으로 sitemap을 생성하도록 설정했고, 구글 서치 콘솔에 사이트맵을 등록했습니다.

  [Github Blog 구글 검색 노출 시키기](https://taeho-jo.github.io/posts/googleseo/)

  [Github 페이지 Chirpy 테마 변경내용 실시간 적용 방법](https://friendlyvillain.github.io/posts/chirpy-refresh/)

<br>

**jekyll pwa 페이지를 수동이 아닌 자동으로 업데이트 되도록 변경**

- chirpy 테마 기준으로 assets/js/pwa 디렉토리의 app.js 와 sw.js를 다음 코드로 변경합니다.
  > 겹치는 부분만 바꿔주세요.

  <details>
    <summary>app.js</summary>
    <div markdown="1">
    ```js
    if ('serviceWorker' in navigator) {
      const isEnabled = '{{ site.pwa.enabled }}' === 'true';

      if (isEnabled) {
        const swUrl = '{{ '/sw.min.js' | relative_url }}';

        navigator.serviceWorker.register(swUrl).then((registration) => {
          registration.addEventListener('updatefound', () => {
            const installingWorker = registration.installing;
            installingWorker.addEventListener('statechange', () => {
              if (installingWorker.state === 'installed' && navigator.serviceWorker.controller) {
                installingWorker.postMessage('SKIP_WAITING');
              }
            });
          });
        });

        navigator.serviceWorker.addEventListener('controllerchange', () => {
          window.location.reload();
        });
      } else {
        navigator.serviceWorker.getRegistrations().then(function (registrations) {
          for (let registration of registrations) {
            registration.unregister();
          }
        });
      }
    }
    ```
    </div>
  </details>


  <details>
    <summary>sw.js</summary>
    <div markdown="1">
    ```js
    self.addEventListener('install', (event) => {
      event.waitUntil(
        caches.open(swconf.cacheName).then((cache) => {
          return cache.addAll(swconf.resources).then(() => {
            return self.skipWaiting();
          });
        })
      );
    });

    self.addEventListener('activate', (event) => {
      event.waitUntil(
        self.clients.claim()
      );
    });

    self.addEventListener('message', (event) => {
      if (event.data === 'SKIP_WAITING') {
        self.skipWaiting();
      }
    });

    self.addEventListener('fetch', (event) => {
      event.respondWith(
        fetch(event.request).then((response) => {
          let responseToCache = response.clone();
          if (response.status === 200 && response.type === 'basic') {
            caches.open(swconf.cacheName).then((cache) => {
              cache.put(event.request, responseToCache);
            });
          }
          return response;
        }).catch(() => {
          return caches.match(event.request);
        })
      );
    });
    ```
    </div>
  </details>