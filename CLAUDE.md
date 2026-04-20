# CLAUDE.md

이 파일은 이 레포지토리에서 동작하는 Claude(Claude Desktop의 Cowork, Claude Code 등)를 위한 운영 지침이다. 이 레포의 유일한 관심사는 **다중 PC에서 Claude Cowork 작업을 이어받기 위한 GitHub 기반 동기화 기반 구축**이다. 프로젝트 기획·설계 관심사는 별도 세션에서 분리되어 다뤄진다.

---

## 세션 시작 시 체크리스트

새 세션에서 이 레포에 진입한 Claude는 첫 작업 전에 다음을 확인한다.

1. 현재 작업 폴더의 실제 마운트 확인:
   ```bash
   mount | grep prom-insight
   ```
2. Git 상태와 원격 연결 확인:
   ```bash
   git status
   git remote -v
   git log --oneline -5
   ```
3. 원격 대비 지연 여부 확인 후 필요 시 동기화:
   ```bash
   git fetch
   git pull --ff-only   # 원격이 앞서 있을 때만
   ```

과거 대화에서 언급된 절대 경로(예: `C:\Users\...`, `/Users/...`, `/sessions/{hash}/...`)는 **그 대화가 생성된 시점의 특정 PC·세션 경로**일 뿐이다. 현재 세션이 실제로 참조해야 할 경로는 1번에서 확인한 마운트뿐이다.

---

## 세션과 대화의 구분 — 반드시 이해

| 개념 | 수명 | 저장 위치 | 경로 정보를 가지는가 |
|---|---|---|---|
| Conversation (대화 스레드) | 사용자가 지우기 전까지 영구 | Anthropic 서버 | **아니오** — 프로젝트 UUID만 기억 |
| Session (샌드박스 VM 인스턴스) | Claude Desktop 프로세스와 함께 소멸 | 특정 PC 로컬 | **예** — 생성 시점의 마운트가 수명 내내 고정 |

하나의 대화가 PC-A의 세션에서 시작되어, 나중에 PC-B의 **새** 세션에서 이어질 수 있다. 이때 두 세션은 각자의 PC 로컬 폴더로 마운트되며, Claude는 **자신이 속한 세션의 마운트만 접근 가능**하다. 과거 메시지에 박힌 다른 PC의 경로는 역사적 기록일 뿐, 현재 Claude가 따라가려 해서는 안 된다.

---

## 파일 조작 규칙

1. **상대 경로만 사용** — 커밋되는 파일의 모든 내부 참조는 레포 루트 기준 `./...` 형식. 상세는 [PATHS.md](./PATHS.md).
2. **줄바꿈 LF 고정** — [.gitattributes](./.gitattributes)가 강제한다. 편집 시 CRLF로 되돌리는 도구를 사용하지 않는다.
3. **동기화 대상 제한** — OS·에디터·오피스·Cowork 임시물은 커밋하지 않는다. [.gitignore](./.gitignore) 참조.
4. **구조 변경 시 문서 우선** — 새 하위 폴더 추가나 대규모 구조 변경은 이 지침 파일이나 README.md를 먼저 업데이트한 뒤 수행한다.

---

## Cowork 권한 모델 주의점

사용자가 Claude Desktop에서 선택한 작업 폴더는 기본적으로 **삭제 차단** 상태이다. Git이 `.git/*.lock` 파일을 관리하는 과정에서 삭제가 필요해지고, 차단되면 그 이후 모든 git 명령이 멈춘다. 이 상황이 발생하면:

1. `allow_cowork_file_delete` 툴로 해당 폴더에 대한 삭제 권한을 사용자에게 요청
2. 승인 후 git 작업 재개

권한은 디렉토리 단위로 유지된다. 한 번 승인되면 해당 폴더 내 이후 삭제가 허용된다.

---

## Git 워크플로

### 커밋

Claude가 Cowork 세션 안에서 수행한다. 커밋 메시지 규칙:

- 제목은 영어, Conventional Commits 접두사(`feat:`, `fix:`, `chore:`, `docs:` 등)
- 본문은 필요 시 한국어 허용, "왜" 중심으로 서술
- `Co-Authored-By: Claude Opus 4.7 <noreply@anthropic.com>` 트레일러 포함

### 푸시

두 전략을 상황별로 쓴다.

- **기본 (권장)** — Claude는 commit까지만 수행. 사용자가 로컬 터미널(Windows Credential Manager / macOS Keychain / SSH key)에서 `git push`. Cowork 세션에 자격증명이 주입되지 않으므로 시크릿 노출 위험이 없다.
- **예외 (자동화 필요 시)** — GitHub PAT을 세션 환경변수(`GH_TOKEN` 등)로 사용자가 일시 주입, Claude가 push까지 수행. 세션 종료와 함께 토큰 소멸.

### 풀

세션 시작 시 위 체크리스트 3번에 따라 수행. 사용자가 다른 PC에서 커밋·푸시한 내용이 있을 수 있으므로 생략하지 않는다.

---

## 범위 제약

이 레포는 **다중 PC 동기화 기반**만 담는다. 다음 관심사는 이 레포의 관심사가 아니며, 별도 세션에서 다룬다:

- 프로젝트 기획·설계 산출물
- 폴더 구조 정책(`docs/`, `design/` 등)
- 파일명 규약
- 산출물 배치·버전 관리 정책

Claude가 이 범위를 넘는 요청을 받으면, 작업을 수행하기 전에 범위 확인을 위해 사용자에게 질문한다.

---

## 참고

- [README.md](./README.md) — 사용자 관점의 다중 PC 워크플로 개요
- [PATHS.md](./PATHS.md) — 파일 내부 경로 참조 규약
- [.gitattributes](./.gitattributes) — 줄바꿈 정규화
- [.gitignore](./.gitignore) — 동기화 제외 패턴
