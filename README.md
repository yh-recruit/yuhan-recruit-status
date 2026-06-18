# 유한양행 채용 전형 현황 시스템

부서별 채용 전형 진행 상황을 한 화면에서 관리하는 사내 웹 도구.

## 🌐 접속 주소 (바로 사용)

**https://yh-recruit.github.io/yuhan-recruit-status/**

- 체험 계정: 관리자 `admin / 0000`, 팀장(예시) `sales1 / 0000`
- 팀원에게는 위 주소만 공유하면 됩니다. 설치/다운로드 불필요.
- ℹ️ 2026-06-18 주소 변경: 기존 `contacteveryone69-max.github.io`가 사내 웹 필터에 호스트명 단위로 차단되어, GitHub username을 `yh-recruit`로 변경(주소도 함께 변경). 사내망 접속 통과 확인 완료.

## ✅ 현재 상태 (2026-06-17 기준)

- **Firebase 실시간 동기화 연결·검증 완료** (프로젝트 ID: `yuhan-recurit`)
  - 한 명이 데이터를 바꾸면 모든 접속자에게 몇 초 안에 자동 반영됩니다.
  - 데이터는 Firebase Firestore(`app/state` 문서)에 저장됩니다.
  - 익명 로그인 + 보안 규칙(`request.auth != null`) 적용. 읽기/쓰기 동작 확인됨.
- **GitHub Pages 호스팅 완료** — 위 주소로 누구나 접속 가능 (저장소 Public).

### 트러블슈팅 기록 (해결됨)
처음에 동기화가 안 됐던 원인 2가지를 잡았습니다:
1. **Cloud Firestore API 미활성화** → Google Cloud Console에서 Enable 처리.
   (재발 시: https://console.cloud.google.com/apis/library/firestore.googleapis.com?project=yuhan-recurit )
2. **projectId 철자 오류** (`yuhan-recruit` → 실제값 `yuhan-recurit`) → HTML 설정값 수정.
   ⚠️ 실제 프로젝트 ID는 `yuhan-recurit` (re-c**u-r**-it) 입니다. 헷갈리지 마세요.

## 🔧 구조 한눈에

| 구성 | 역할 | 누가 관리 |
|------|------|-----------|
| `recruit-status.html` | 앱 본체 (기능·디자인) | 코드 수정 시 GitHub에 push → 자동 반영 |
| `index.html` | 기본 주소 접속 시 앱으로 자동 이동 | 건드릴 일 없음 |
| Firebase Firestore | 데이터 저장소 (전형/계정 내용) | 자동, 건드릴 일 없음 |
| GitHub Pages | 호스팅(주소 공개) | 자동, 건드릴 일 없음 |

> **기능 추가·개선**은 `recruit-status.html`을 수정해서 GitHub에 올리면 됩니다.
> Firebase에는 코드를 올리지 않습니다(데이터 저장소일 뿐).

## 💻 회사 PC로 가져오기

```bash
git clone https://github.com/yh-recruit/yuhan-recruit-status.git
```

또는 GitHub 저장소 페이지 → **Code → Download ZIP**.

> 그냥 사용만 할 거라면 다운로드 없이 위 **접속 주소**로 바로 쓰면 됩니다.
> 다운로드는 코드를 수정하거나 백업할 때만 필요합니다.

## ⚠️ 보안 참고 (다음 단계 과제)

- 현재는 **익명 로그인** 기반이라, 주소를 아는 사람은 데이터에 접근할 수 있습니다.
  앱 안의 `admin/0000` 로그인은 화면 가리개 수준입니다.
- 저장 데이터는 대부분 숫자(지원자 수 등)와 부서 일정이지만,
  본격 운영 시 **실제 로그인(Firebase Auth)**으로 강화하는 것을 권장합니다.

## 📄 관련 문서

- [`FIREBASE-SETUP.md`](FIREBASE-SETUP.md) — Firebase 설정 방법 (이미 완료됨)
- [`CLAUDE.md`](CLAUDE.md) — 코드 구조 / 작업 인수인계 상세
- [`HANDOFF.md`](HANDOFF.md) — 프로젝트 전체 맥락
