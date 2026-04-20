# 경로 규약 (PATHS.md)

다중 컴퓨터에서 이 레포지토리를 공유할 때 **파일 내부의 경로 참조가 깨지지 않도록** 하기 위한 규약이다. 본 문서는 경로 표기에 한정하며, 폴더 구조·파일명 정책은 별도 세션에서 정의한다.

---

## 왜 이 규약이 필요한가

이 레포는 여러 PC에서 클론되어 작업된다. 각 PC의 절대 경로는 서로 다르며, Claude Cowork 세션 내부의 가상 경로도 세션마다 달라진다.

| 환경 | 레포 루트의 절대 경로 (예시) |
|---|---|
| PC-A (Windows) | `C:\Users\24033jeongdo\Documents\Claude\Projects\PromInsight_MyProject_v1\prom-insight` |
| PC-B (macOS, 가정) | `/Users/{username}/Documents/Projects/PromInsight_MyProject_v1/prom-insight` |
| Cowork VM 내부 | `/sessions/{session-hash}/mnt/PromInsight_MyProject_v1/prom-insight` |

어느 파일에서 다른 파일을 절대 경로로 참조하면, 다른 PC 또는 다른 세션에서는 그 참조가 깨진다. 따라서 커밋되는 모든 파일은 "레포 루트를 기준점으로 한 상대 경로"만 사용한다.

---

## 규약

### 허용

- 레포 루트 기준 상대 경로: `./README.md`, `./PATHS.md` 처럼 `./`로 시작
- 레포 외부 리소스 참조는 HTTPS URL (`https://github.com/...`)
- 폴더 구분자는 슬래시(`/`)만 사용

### 금지

- PC별 절대 경로: `C:\...`, `/Users/...`
- Cowork VM 세션 경로: `/sessions/{hash}/mnt/...` — 세션 종료 시 소멸하는 휘발성 경로
- 상위 디렉토리 이동(`../`)으로 레포 바깥 접근
- Windows 역슬래시(`\`) 구분자

---

## Claude가 지키는 약속

Claude가 이 레포에서 파일을 생성·수정할 때:

1. 파일 내용 안에서 다른 파일을 가리킬 때는 반드시 `./...` 형식 상대 경로 사용
2. 사용자에게 파일 위치를 안내할 때 `computer://` 링크(현재 세션용)와 레포 상대 경로(규약 준수용)를 함께 제시
3. 절대 경로(Windows/Unix/VM 세션)를 문서·스크립트 안에 써넣지 않음
