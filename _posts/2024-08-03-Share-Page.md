---
title: Spring - 공유 페이지 제공하기
date: 2024-08-03 14:36:00 +09:00
categories: [spring]
tags:
  [
    spring,
    redis
  ]
---

## 공유페이지 제공
- 기존에 진행 중인 사이드 프로젝트 `문덕`은 문화생활에 대한 리뷰를 작성하고 보관할 수 있는 서비스다.
- 커뮤니티 기능 없기 때문에 자신만 자기가 쓴 글을 볼 수 있었다.
- 공유를 원하는 사용자를 위해 우리는 공유페이지를 제공하기로 했다.
- 이 글에서는 임시방편으로 만들었던 공유페이지에서 완성도를 끌어올리기 위해 어떻게 로직을 바꾸었는지를 기술한다.

<br>

## 공유페이지 설계
- 공유페이지를 제공하기 위해 다음 2가지의 로직을 생각했다.

<br>

#### [1. URL에 모든 데이터를 담아 제공]
- 말그대로 모든 데이터를 URL 안에 담아 정적 페이지에 해당 데이터를 파싱하여 제공한다.
- 쿼리 파라미터는 모든 데이터가 URL에 노출되기에 base64로 인코딩하여 경로 변수로 데이터를 담는 방식을 생각했다.
    > 쿼리 파라미터도 상관은 없겠지만 그대로 노출되는건 멋이 없지 않은가?

- 그러나 경로 변수에 모든 데이터를 담는 방식에는 문제가 있었다.
    - 페이지에는 이미지가 들어가고 최대 10MB의 이미지가 제공된다.
    - base64 인코딩을 하면 30% 더 긴 문자열을 생성한다.
    - URL은 일반적으로 2000자 내외로만 허용이 된다.
- 따라서 10MB 이미지를 base64로 인코딩하면 **1300만 길이**의 문자열이 나와 URL에 데이터를 담는 방식은 사용할 수 없다.

<br>

#### [2. 리뷰 인덱스 id를 인코딩하여 동적 페이지 제공 ✅]
- 인덱스 id 값을 인코딩하여 URL로 제공하기 때문에 URL의 길이가 매우 짧다.
- 요청자가 사용자이기 때문에 `ThymeLeaf` 로 동적인 페이지를 만들어 사용자에게 제공한다.

<br>

## 공유페이지 로직
<img src="/assets/img/240803/share1.png" alt="share1" width=600>
<img src="/assets/img/240803/share2.png" alt="share2" width=400>

<br>

#### [예시]
- 리뷰 인덱스가 `1364`인 페이지를 공유한다고 가정하자.
- 서버에서는 `1364`를 base64 인코딩하여 `MTM2NA==` 값을 앱으로 보내준다.
- 앱에서는 서버 주소와 합쳐 완전한 URL 링크를 제공한다.
    - https://moonduck.o-r.kr/share/MTM2NA==
- 다른 유저들은 해당 링크에 접속하여 공유된 페이지를 볼 수 있다.

<br>

## 문제점
- 짧아도 너무 짧다.
- 또한 base64는 암호화 기법이 아닌 데이터가 깨지지 않게 전달하는 `인코딩` 이다.
- 때문에 아무 숫자나 인코딩해서 서버로 요청을 보내면 공유되지 않았음에도 다른 사람들이 언제든지 자유롭게 접근해서 볼 수 있다.
- 해결해야 하는 문제점을 정리하면 다음과 같다.
    1. URL만으로 정보를 얻을 수 있으면 안된다.
    2. 사용자가 원하는 페이지만 공유가 되어야 한다. 공유한 적이 없음에도 다른 사람들에게 보여지면 안된다.
    3. 유효기간이 존재하여 일정 시간이 지난 후 접근할 수 없게 해야 한다.

<br>

## 개선
- 위에서 나열한 문제점들을 다음과 같이 해결했다.

1. 고유한 식별번호인 `UUID` 를 사용한다.
2. 사용자가 공유하기 버튼을 눌렀을 때에만 **UUID를 매핑하여 서버에 저장**한다.
3. Redis의 `TTL` 기능을 사용하여 하루가 지났을 때 자동으로 매핑 데이터가 사라지게 설계한다.

<br>

## 코드
- 핵심 비즈니스 로직으로 개선된 결과를 살펴보겠다.
- 자세한 코드는 [여기서](https://github.com/Moon-Duck-Org/MoonDuckBE) 확인할 수 있다.
- Redis의 TTL은 [여기서](https://ajroot5685.github.io/posts/RTR-With-Redis/) 확인하기 바란다.

<br>

#### [공유페이지의 URL 파라미터 생성]

```java
public String getShareUrl(Long userId, Long boardId) {

    // 올바르지 않은 요청은 에러 반환
    boardRepository.findByIdAndUserId(boardId, userId)
            .orElseThrow(() -> new ErrorException(ErrorCode.BOARD_NOT_FOUND));

    String uuidStr = UUID.randomUUID().toString();  // 랜덤 UUID 문자열 생성
    long expirations = new Date(System.currentTimeMillis() + 24 * 60 * 60 * 1000L).getTime() / 1000;    // 하루를 long 타입의 초단위로 변환
    Share share = new Share(uuidStr, boardId, expirations); // URL 매핑 객체 생성

    shareRepository.save(share);    // 레디스 서버에 저장

    return uuidStr; // 매핑된 UUID 반환
}
```
- 크게 복잡하지 않다.
- 매핑할 UUID, 리뷰 인덱스, 유효기간을 서버에 저장하고 UUID를 반환한다.
- 이 UUID를 통해서 매핑된 인덱스 값을 가져오고 해당 리뷰의 데이터들을 가져올 수 있게 된다.

<br>

#### [공유된 페이지 접속]

```java
public ShareDataResponse getShareData(String param) {   // param = UUID 문자열

    // 인자로 받은 UUID로 매핑 데이터를 조회한다.
    Optional<Share> share = shareRepository.findById(param);
    if (share.isEmpty()) {
        return null;
    }
    Long boardId = share.get().getBoardId();

    // 실제 데이터를 가져온다.
    Optional<Board> boardOp = boardRepository.findByIdWithProgram(boardId); 
    if (boardOp.isEmpty()) {
        return null;
    }
    Board board = boardOp.get();

    // 데이터 파싱 로직 생략
    .
    .

    // 최종 데이터들을 객체에 담아 반환한다.
    return ShareDataResponse.builder()
            .title(board.getTitle())
            .category(category)
            .program(board.getProgram() == null ? null : board.getProgram())
            .content(board.getContent())
            .image(board.getImage1())
            .score(board.getScore())
            .createdAt(createdAt)
            .build();
}
```

```java
public String getSharePage(String param, Model model) {
    // 서비스단에서 데이터를 가져온다.
    ShareDataResponse shareData = shareService.getShareData(param);

    // 잘못된 데이터를 요청하거나 데이터가 삭제된 경우 에러페이지를 호출한다.
    if (shareData == null) {
        return "error";
    }

    // 정상적으로 데이터를 가져왔으면 Model에 담아 타임리프 페이지를 호출한다.
    model.addAttribute("data", shareData);
    return "share";
}
```

<br>

## 결과
- 아래는 정상적으로 공유된 페이지에 접속했을 때와 에러 페이지 화면이다.
- 정상적으로 공유받은 URL에 접속하면 내용들이 보여지고, 잘못된 URL에 접속한 경우 에러페이지가 보여진다.

<div style="display: flex; justify-content: space-between; align-items: center;">
    <img src="/assets/img/240803/success.png" alt="success" width="300">
    <img src="/assets/img/240803/error.png" alt="error" width="300">
</div>

<br>

## 느낀점
- 다양한 서비스들에서 제공했던 공유 기능을 직접 만들어보면서 생각보다 꽤 간단하다는 것을 알 수 있었다.
- 특히 공유페이지의 유효기간을 설정하는 것이 고민이었는데 Redis에 대해 공부하면서 정말 쉽게 해결할 수 있었다.
    > 물론 RDB, 스케쥴러로 구현할 수도 있었지만 최대한 로직의 단숨함을 유지하고 싶었다.
- 구현방식에 폭넓은 선택지를 떠올리고, 개발 편의성을 증진시킬 수 있기 때문에 새로운 기술에 대한 학습은 무조건 필요하다고 생각한다.
- 이렇게 개선한 것 외에도 도메인을 분리시켜 트래픽 분산, 서버 주소 노출로 인한 보안 이슈를 해결할 수도 있을 것 같다.