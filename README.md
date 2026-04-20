# prom-insight

Claude Cowork 프로젝트 `프롬인사이트_개인프로젝트`의 산출물을 **여러 컴퓨터에서 이어서 작업**할 수 있도록, GitHub를 경유해 파일을 동기화하는 레포지토리.

본 레포는 현재 **다중 PC 동기화 관심사에 한정된 구성**만 포함한다. 프로젝트 기획·설계(폴더 구조, 산출물 정책, 파일명 규약 등)는 별도 세션에서 정의·반영한다.

---

## 동기화 모델

Claude Cowork의 한 프로젝트는 세 층위로 구성되며, 각 층위의 동기화 주체가 다르다.

| 층위 | 내용 | 동기화 주체 | 자동 여부 |
|---|---|---|---|
| 대화·프로젝트 컨텍스트 | 대화 기록, 프로젝트 지식, 사용자 선호 | Claude 계정 (Anthropic 서버) | 자동 |
| 작업 파일 | 이 레포의 모든 내용 | GitHub 원격 ↔ 각 PC 로컬 | **수동** (`git push` / `git pull`) |
| 작업 폴더 바인딩 | "프로젝트 ↔ 로컬 어느 폴더" 매핑 | 각 PC의 Claude Desktop 로컬 설정 | **수동** (PC별 최초 1회) |

Claude Cowork 자체는 파일 수준 동기화를 수행하지 않기 때문에, 본 레포가 그 간극을 채운다.

### 작동 메커니즘 — 세션과 대화의 관계

"한 프로젝트를 여러 PC에서 연다"는 것은 내부적으로 **하나의 대화 스레드를 서로 다른 PC의 서로 다른 세션이 교대로 로드**하는 것이다.

- **대화 스레드**는 Anthropic 서버에 저장되며 프로젝트 UUID만 기억한다. 로컬 폴더 경로는 모른다.
- **세션**은 Claude Desktop이 각 PC에서 띄우는 샌드박스 VM 인스턴스다. 각 세션은 자기 PC의 로컬 폴더로 마운트되며, 그 마운트는 **세션 수명 내내 고정**된다.
- PC-B에서 같은 대화를 이어 열면 **그 PC에서 새 세션이 뜨고** 해당 세션의 마운트는 PC-B의 로컬 폴더를 가리킨다. PC-A의 경로와는 무관하다.

따라서 과거 메시지에 남아 있는 절대 경로(Windows `C:\...`, macOS `/Users/...`, Cowork VM `/sessions/{hash}/...`)는 그 메시지가 생성된 당시의 세션 상태일 뿐이며, 현재 세션의 실제 파일 접근과는 별개다. 이것이 [PATHS.md](./PATHS.md)의 상대 경로 규약이 필요한 근본 이유다.

---

## PC별 최초 세팅 (1회)

1. 이 레포(`https://github.com/Rizingblare/prom-insight.git`)를 원하는 위치에 클론
2. 같은 Anthropic 계정으로 Claude Desktop 로그인
3. Cowork에서 프로젝트 `프롬인사이트_개인프로젝트` 열기
4. 작업 폴더 선택 UI에서 클론한 `prom-insight`의 **상위 폴더**를 지정
5. 해당 PC에 `git push` 자격증명 설정 (Windows Credential Manager / macOS Keychain / SSH key 중 택1)

---

## 세션 전환 워크플로

```
[작업 종료 측 PC]
  Claude가 Cowork에서 git commit 수행
  사용자가 로컬 터미널에서 git push

[작업 시작 측 PC]
  Claude Desktop에서 동일 프로젝트 열기 → 대화 맥락 자동 복원
  로컬 터미널 또는 Claude를 통해 git pull
  작업 재개
```

---

## Push 전략

두 전략을 상황별로 병행한다.

- **평상시 (권장)** — Claude는 Cowork 안에서 commit까지만 수행하고, 사용자가 로컬 터미널에서 push. Cowork 세션에 자격증명이 주입되지 않아 시크릿 취급 위험이 낮다.
- **자동화가 필요할 때** — GitHub PAT을 세션 환경변수로 일시 주입해 Claude가 push까지 수행. 세션 종료 시 토큰도 소멸.

---

## 파일 내부 경로 규약

모든 커밋되는 파일의 본문에서 다른 파일을 참조할 때는 **절대 경로 금지, 레포 루트 기준 상대 경로(`./...`)만 사용**. 이유와 구체 규칙은 [PATHS.md](./PATHS.md) 참조.

---

## 관련 파일

- [PATHS.md](./PATHS.md) — 파일 내부 경로 참조 규약
- [.gitattributes](./.gitattributes) — OS 간 줄바꿈 차이 제거 (LF 고정)
- [.gitignore](./.gitignore) — OS·에디터·Cowork 임시물의 동기화 대상 제외
- [LICENSE](./LICENSE)
