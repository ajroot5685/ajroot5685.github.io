---
title: Git Hook을 이용하여 커밋을 아름답게 관리하기(Husky&commitlint)
date: 2025-02-24 19:58:00 +09:00
categories: [develop]
tags:
  [
    git,
    git hook,
    husky,
    commitlint
  ]
---

## 들어가며

개발을 위해 주로 사용하는 Git에는 어떤 이벤트가 생겼을 때 자동으로 특정 스크립트를 실행할 수 있는 기능이 있다.
<br>
바로 `Hook` 이라는 기능이다.
<br>
훅이 뭔지 알아보고, 커밋의 형식을 강제하고 다듬는 작업을 자동화해보자

<br>

## Git Hook

훅은 위에서 말했듯이, **어떤 이벤트가 생겼을 때 자동으로 특정 스크립트를 실행**할 수 있는 기능을 가진 녀석이다.
<br>
주로 활용하는 용도는 검사, 자동화가 대부분이라고 볼 수 있다.
<br>
즉, 인력이 필요한 부분을 코드로 치환할 수 있어 아주 편리하다고 볼 수 있다.

<br>

## Git Hook 종류

Git Hook은 크게 2종류로 나뉜다.
<br>
**클라이언트 훅**은 로컬에서 실행되며 커밋, 푸시 등의 로컬 작업을 제어할 때 사용한다.
<br>
**서버 훅**은 원격 저장소(Git 서버)에서 실행되며 서버에서 적용되기 전에 검증하거나 자동 배포하는 데 사용된다.
<br>
보기 쉽게 표로 정리하면 다음과 같다

| 구분   | 클라이언트 훅 | 서버 훅 |
|--------|-------------|--------|
| 실행 위치 | 로컬 저장소 | 원격 저장소 (Git 서버) |
| 주요 용도 | 코드 품질 검증, 메시지 검사, 테스트 실행 | 브랜치 보호, 배포 자동화 |
| 예시 | `pre-commit`, `commit-msg`, `pre-push` | `pre-receive`, `post-receive` |

전체 총 19개의 종류가 존재하지만, 이 글에서는 클라이언트 훅 중에서 주로 사용되는 것들만 다룬다

<img src="/assets/img/250224/git_hook.png" style="border-radius:5px" alt="git_hook" width="800">

#### pre-commit

`git commit` 실행 직후, 커밋 메시지 입력 전에 실행된다
<br>

**용도**
- 코드 스타일 검사
- 테스트 실행
- 파일 포맷팅(이미지 압축 등)
- 특정 파일 커밋 방지

<br>

#### prepare-commit-msg

커밋 메시지가 생성되기 직전, 편집기가 열리기 전에 실행된다
<br>

**용도**
- 커밋 메시지 기본 템플릿 설정
- 자동으로 브랜치 이름을 메시지에 포함

<br>

#### commit-msg

사용자가 커밋 메시지를 입력한 후, 커밋이 저장되기 전에 실행된다
<br>

**용도**
- 커밋 메시지 형식 검증
- 특정 패턴이 포함되었는지 확인

<br>

#### post-commit

커밋이 완료된 직후 실행된다
<br>

**용도**
- 커밋 후 추가 작업 실행(Slack 알림, 로그 저장 등)
- 자동 푸시 실행

<br>

#### pre-push

`git push` 실행 전에 실행된다
<br>

**용도**
- 원격 저장소로 푸시하기 전 코드 품질 검사
- 테스트 실행
- 특정 브랜치에 대한 푸시 제한

<br>

## 커밋 메시지 형식을 강제하고 이모지를 추가하여 가독성 높이기

<img src="/assets/img/250224/commit.png" style="border-radius:5px" alt="commit" width="150">

형식을 강제하는 장치가 없을 때 벌어지는 문제점이 보이는가?
<br>
`feat`라는 커밋 타입을 적으면서 오타가 발생할 수 있고, 정해지지 않은 타입 컨벤션에 의해 의미가 비슷하지만 다른 타입을 써 헷갈리게 만들 수 있다.
<br>
우리는 Git Hook을 이용하여 문제를 예방하고 커밋 타입에 따라 이모지도 붙여서 커밋을 한눈에 볼 수 있다. 바로 아래 처럼 말이다

<img src="/assets/img/250224/commit2.png" style="border-radius:5px" alt="commit2" width="150">

<br>

## Husky

Git Hook을 쉽게 관리하고 팀원과 공유할 수 있도록 도와주는 도구다
<br>
기존에는 `.git/hooks/`에 훅 파일들이 있어야 적용이 되고, `.git`은 로컬로만 관리되기 때문에 훅 설정 공유가 어려웠다.
<br>
Husky는 `.husky`에서 훅파일들을 관리하며, `npm install`을 통해 쉽게 훅 설정을 할 수 있다.

<br>

**설치**

```sh
# npm을 기준으로 진행한다.

npm install husky --save-dev # package.json이 설정된다.
npx husky install # .husky 폴더가 생성되고, 훅 경로가 설정된다.
```

<br>

## commitlint

커밋 메시지의 형식을 검증하는 도구다
<br>
json 파일로 메시지의 형식을 정의하면, commitlint 규칙에 따라 커밋 메시지를 검증하게 된다.

<br>

**설치**

```sh
npm install --dev @commitlint/config-conventional @commitlint/cli
```

<br>

## 적용

이제 필요한 도구들도 간단하게 알아봤으니, 실제로 적용해보겠다.
<br>
훅의 종류 중 `prepare-commit-msg`와 `commit-msg`를 사용할 것이다.

> 사용했던 세팅을 공유한다. 필요에 따라 변경해서 사용하면 되겠다.

**커밋 흐름**

```
커밋 요청
  ↓
검사 전 커밋 메시지 변환(prepare-commit-msg)
  ↓
commitlint 도구를 사용하여 커밋 형식 검사(commit-msg)
  ↓
커밋 완료
```

**디렉토리 구조**

```
project-root
├── .husky
│   ├── commit-msg
│   ├── husky.sh
│   └── prepare-commit-msg
├── .commitlintrc.json
└── package.json
```

<br>

#### husky.sh

이 파일에 스크립트를 추가하여 훅들을 더 깔끔하게 관리할 수 있다.

```sh
#!/usr/bin/env sh
if [ -z "$husky_skip_init" ]; then
  readonly hook_name="$(basename -- "$0")"
  echo "🚀 Husky(Start) - 시작된 훅 이름 : $hook_name"

  readonly husky_skip_init=1
  export husky_skip_init

  sh "$0" "$@"
  exitCode="$?"

  if [ $exitCode -ne 0 ]; then
    echo "❌ Husky(Error), 이름:{$hook_name} 에러코드:{$exitCode}"
  fi

  echo "✅ Husky(Success) - 완료된 훅 이름 : $hook_name"
  exit $exitCode
fi
```

<br>

#### prepare-commit-msg

커밋에 이모지를 넣는 작업을 수동으로 할 순 없다
<br>
bash 파일로 스크립트를 작성해서 정해진 규칙에 따라 꾸며지도록 설정해보자

<details>
  <summary>prepare-commit-msg</summary>
  <div markdown="1">

  ```bash
  #!/usr/bin/env bash
  `. "$(dirname -- "$0")/husky.sh"

  # 이모지 정의
  emojies=(
    "Feat ✨"
    "Fix 🐛"
    "Style ⭐️"
    "Refactor ♻️"
    "File 📁"
    "Test ✅"
    "Docs 📝"
    "Remove 🔥"
    "Ci 💚"
    "Release 🔖"
    "Chore 🔧"
  )

  COMMIT_MESSAGE_FILE_PATH=$1
  first_line=$(head -n1 "$COMMIT_MESSAGE_FILE_PATH")
  remaining_lines=$(tail -n +2 "$COMMIT_MESSAGE_FILE_PATH")

  # 변환 제외
  if [[ $first_line =~ ^(Merge|Revert|Amend|Reset|Rebase|Tag) ]]; then
    exit 0
  fi

  type=$(echo "$first_line" | grep -o "^[A-Za-z]*")
  start_case_type="$(echo "$type" | awk '{print toupper(substr($0,1,1)) tolower(substr($0,2))}')"

  # 맞는 이미지를 찾아 헤더 수정
  for entry in "${emojies[@]}"; do
    key="${entry%% *}"
    value="${entry#* }"
    if [[ "$start_case_type" == "$key" ]]; then
      emoji=$value
      first_line=$(echo "$first_line" | sed "s/^$type/$emoji $start_case_type/")
      break
    fi
  done

  branch_name=$(git rev-parse --abbrev-ref HEAD)
  issue_number=$(echo "$branch_name" | sed -n 's/issue\([0-9]\{1,\}\)-.*/\1/p')

  # 합치기
  echo "$first_line" > "$COMMIT_MESSAGE_FILE_PATH"
  echo "$remaining_lines" >> "$COMMIT_MESSAGE_FILE_PATH"

  # 브랜치 이름에서 이슈번호를 뽑아 footer에 추가, 이슈와 연결됨
  last_line=$(tail -n1 "$COMMIT_MESSAGE_FILE_PATH")
  if [[ -n "$issue_number" ]]; then
      if ! grep -q "#$issue_number" "$COMMIT_MESSAGE_FILE_PATH"; then
        echo "" >> "$COMMIT_MESSAGE_FILE_PATH"
        echo "#$issue_number" >> "$COMMIT_MESSAGE_FILE_PATH"
      fi
  fi`
  ```
  </div>
</details>

<br>

#### commit-msg

변환 작업이 끝났으면 규칙에 만족하는지 검사해야 한다.
<br>
commitlint 도구를 사용하기 때문에 호출하는 코드만 넣어준다.

```sh
#!/usr/bin/env sh
. "$(dirname "$0")/husky.sh"

npx --no-install commitlint --edit "$1"
```

<br>

#### .commitlintrc.json

실제로 형식 검사에 사용되는 규칙이다.
<br>
입맛대로 규칙이나 이모지 등을 바꿔서 사용하자

<details>
  <summary>.commitlintrc.json</summary>
  <div markdown="1">

  ```json
  {
    "extends": ["@commitlint/config-conventional"],
    "parserPreset": {
      "parserOpts": {
        "headerPattern": "^(?<type>.+?)(?:\\((?<scope>.+)\\))?:\\s(?<subject>.+)$",
        "headerCorrespondence": ["type", "scope", "subject"]
      }
    },
    "rules": {
      "subject-empty": [2, "never"],
      "subject-full-stop": [2, "never", "."],
      "subject-case": [2, "never", []],
      "subject-empty": [2, "never"],

      "scope-case": [2, "never", []],

      "body-leading-blank": [1, "always"],
      "body-max-line-length": [2, "always", 100],
      "footer-leading-blank": [1, "always"],
      "footer-max-line-length": [2, "always", 100],

      "type-case": [0],
      "type-empty": [2, "never"],
      "type-enum": [
        2,
        "always",
        [
          "✨ Feat",
          "🐛 Fix",
          "⭐️ Style",
          "♻️ Refactor",
          "📁 File",
          "✅ Test",
          "📝 Docs",
          "🔥 Remove",
          "💚 Ci",
          "🔖 Release",
          "🔧 Chore"
        ]
      ]
    }
  }
  ```
  </div>
</details>
