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
  - 데이터는 Firebase Firestore(`app/state` 문서)에 저장됩니다. **문서를 필드로 분해 저장**(`depts`/`accounts`/`tasks`/`order`)하여, 저장 시 건드린 부서만 부분 반영 → 동시 편집 시 남의 부서 덮어쓰기 완화(2026-07 유지보수). 구버전 단일 문자열 문서도 자동 인식.
  - 익명 로그인 + 보안 규칙(`request.auth != null`) 적용. 읽기/쓰기 동작 확인됨.
  - 상단바에 **동기화 상태 배지**(실시간 공유 / 이 브라우저 저장 / 동기화 오류)로 저장 성공·실패를 표시.

### 변경 이력

**2026-07-04 (2) — 지연 표시·면접 일정표 개선 (CEO 라이브 피드백 반영)**
- 지연 판정 로직 변경: 마감 즉시가 아니라 **현재 단계 종료 기준일 + 영업일 5일(주말·`HOLIDAYS` 공휴일 제외) 초과** 시에만 지연(`deptOverdue`/`addBusinessDays`/`deptStageEndDate` 신설). 종료 기준일 = 공고진행·서류·AI=마감일, 면접전형=마지막 세션일, 공고준비·입사=대상 아님.
- 지연 표시를 카드 **빨간 테두리(`.dept-card.overdue`)만**으로 통일(기존 "⚠ 지연" 텍스트 배지 제거). D-day 배지는 임박/당일만 색 강조.
- 면접 일정표 출력(`ivpWeekGridHTML`): **전체/지난 면접 제외 필터**(`ivpPast`/`setIvpPast`), 카드에 **채용형태 병기**(같은 팀명 다른 전형 구분), **카드 텍스트 가운데 정렬**, **짧은 세션 최소 높이 보장**(시간이 항상 보이도록, 시간→인원 순서로 재배치).

**2026-07-04 (1) — 기술 결함 정리 + 기능 개선**
- 면접 시간 정렬 오류(9:30이 14:00 뒤로) 수정, 클라우드 문서 필드 분해·부분 저장, 오염 데이터 방어, 조회 계정 쓰기 차단, 날짜·시간 유효성 검증, 세션 유지(새로고침 시 재로그인 제거), 공휴일 데이터(2027 일부) 보강, 계정 정보 화면 노출 제거.
- 기능 추가: 대시보드 카드 정렬 옵션, 통계 기간 프리셋·리드타임 지표, 상세보드 체크 완료일 기록·체크리스트 커스터마이징, 일정표 당일/D-1 면접 알림 밴드. 상세는 `tasks/사이트전체점검/` 참고.
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
