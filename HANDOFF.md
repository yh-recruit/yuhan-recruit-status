# 유한양행 채용 전형 현황 시스템 — 작업 인수인계

> 아래 내용을 Claude Code 채팅창에 그대로 붙여넣으면 작업을 이어서 진행할 수 있습니다.
> (같은 폴더에 있는 `recruit-status.html` 파일도 Claude Code 작업 폴더로 함께 가져가세요.)

---

안녕하세요. 진행 중이던 **유한양행 채용 전형 현황 시스템** 프로토타입을 이어서 작업하려고 합니다.
작업 파일은 `recruit-status.html` **단일 파일**입니다. 먼저 이 파일을 읽고 아래 맥락을 파악한 뒤 작업해 주세요.

## 1. 프로젝트 개요
- 부서마다 채용 전형 진행 단계가 제각각이라, 이를 **한 화면에서 파악·관리**하기 위한 사내 웹 도구입니다.
- 채용 절차: **공고준비 → 공고 진행 → 서류전형 → AI역량검사 → 면접전형 → 입사**
- 관리자는 전체 부서를 보고 관리하고, 각 팀장(실장 포함)은 본인 부서 현황만 **조회 전용**으로 봅니다.

## 2. 실행 방법
- 별도 빌드/서버 없이 `recruit-status.html`을 브라우저에서 열면 바로 동작합니다.
- 체험 계정: 관리자 `admin / 0000`, 팀장(예시) `sales1 / 0000`.

## 3. 기술 스택 & 원칙
- **단일 HTML 파일**(인라인 CSS + 바닐라 JS). 프레임워크/번들러 없음.
- 데이터는 백엔드 없이 브라우저 **localStorage**에 저장 (key: `yh_recruit_state_v3`, v1/v2 자동 마이그레이션 + `normalize()`).
- 폰트: Pretendard(CDN), 색상은 CSS 변수(`--yh-green` 등) 사용.
- 보안 로그인이 아니라 클라이언트 측 게이팅 수준(프로토타입). 실제 다중 사용자 배포는 추후 과제.

## 4. 디자인 / 브랜드 규칙
- 유한양행 톤: **유한그린**(`#00833D` 계열) 메인, 골드 보조, 깔끔한 코퍼레이트 느낌.
- 로그인 화면은 좌측 그린 브랜드 패널(**버드나무 CI 모티프**) + 우측 폼의 2단 구성.
- UI 텍스트는 모두 한국어.

## 5. 데이터 모델 (부서 객체)
```js
{
  id, name,
  type,            // 채용형태: "인턴" | "경력" | "계약직(일반)" | "계약직(대체)" | "계약직(보훈)" | "계약직(장애)"
  reqCount,        // 채용요청인원
  ftTO,            // 정규직 TO (인턴 등 전환 정원)
  stage,           // 현재 단계 (STAGES 중 하나)
  schedule: {      // 전형별 일정
    [stage]: { s, e, place, panel, observers, timeStart, timeEnd }
    // s=시작일, e=마감일(YYYY-MM-DD). 면접전형만 place/panel(면접관)/observers(배석자)/timeStart/timeEnd 사용
  },
  accounts: [ { user, code } ],   // 한 부서에 팀장·실장 등 여러 계정 가능. 코드 기본 "0000"
  notes,
  counts: { applicants, doc, ai, hired },   // 지원자/서류합격/AI합격/채용인원
  checklist: { "colId:idx": true },         // 전형 상세 보드(체크리스트) 완료 상태
  updated
}
```
- `STAGES = ["공고준비","공고 진행","서류전형","AI역량검사","면접전형","입사"]`
- `STAGE_SCHED`: 단계별 날짜 필드 사용 여부. **공고준비=날짜 없음, 면접전형·입사=시작일만**, 나머지=시작/마감.
- `ADMIN = { user:"admin", code:"0000" }` (관리자 다중접속 허용).

## 6. 화면(탭) 구성 & 구현된 기능
1. **전형 현황(대시보드)**: 단계별 요약 칩(필터), 부서 카드(스텝퍼·현재단계 일정·인원 pill·메모 미리보기 3줄), 카드 클릭 → 전형 상세 보드. 카드 하단에서 단계 즉시 변경 / 상세편집 / 메일 작성. 카드 높이 통일.
2. **부서 관리**: 표 형태. 추가 / **일괄 등록**(엑셀 붙여넣기, 계정·코드 자동) / 편집 / 삭제, 계정 목록 표시.
3. **상세편집 모달**: 부서명·채용형태·채용요청인원·정규직TO·현재단계 / 전형별 일정(숫자 8자리 입력 시 `YYYY-MM-DD` 자동 정렬) / **면접 정보**(시작·종료 시간, 장소, 참석 면접관, 배석자) / 다중 계정(아이디 비우면 삭제) / 메모 / 인원 현황.
4. **전형 상세 보드 = 채용담당자 체크리스트**: 5개 컬럼(지원서접수/서류전형/AI역량검사/면접전형/채용종료). 항목별 **완료 토글**(색 변화·취소선), 다음 할 항목 **금색 테두리 + "지금 진행할 차례"**, 한 전형 완료 시 다음 전형 첫 항목으로 자동 이동, 컬럼별 진행률(n/총), 채용종료의 **'채용 성과 리포트' 클릭 시 팝업**(지원자/서류합격/AI합격/면접합격 수). 완료 상태는 부서별 저장.
5. **메일 작성**: 6개 전형 단계 템플릿. 부서·단계 선택 후 숫자 입력 → 제목·본문 자동 완성 → 복사. **정형 템플릿은 추후 사용자 제공분으로 교체 예정**(현재는 자리표시자).
6. **통계**: 기간 필터(기준일=공고 진행 시작일). KPI 카드(공고 진행 건/총 지원자/최종 채용), 보조 지표, **채용형태 분포 막대**, **전형 퍼널**(지원→서류→AI→채용), 채용형태별·부서별 표.
7. **일정표**: **월간 캘린더**. 전형별 색상, 기간 단계는 여러 날에 걸쳐, 면접·입사는 해당일에 표시(면접은 시간 동반). 월 이동(‹ ›), 일정 많은 달 기본 표시, 면접 일정 상세표. **팀장 조회 화면에도 동일 표시**.
- **팀장 조회 화면**: 본인 부서 hero, 진행 단계, 채용형태/요청인원/TO, 캘린더, 전형별 일정, 면접 정보, 인원 현황, 메모 (모두 읽기 전용).

## 7. 코드 구조 맵 (주요 함수 — `<script>` 내부)
- 상수: `STAGES`, `STAGE_COLORS`, `STAGE_SCHED`, `STAGE_TO_COL`, `EMP_TYPES`, `ADMIN`, `LS3/LS2/LS1`
- 유틸: `id`, `today`, `pad2`, `esc`, `hexA`, `emptySchedule`, `maskDate`, `di`, `dnum`, `ivTimeStr`
- 데이터: `seed()`, `migrateV1()`, `normalize()`, 상단 IIFE에서 state 초기화, `saveState()`
- 인증: `doLogin`, `logout`, `enterApp`
- 탭: `switchTab` (`dash/manage/mail/stats/sched`)
- 대시보드: `renderAll`, `renderSummary`, `toggleFilter`, `stepperHTML`, `badgeHTML`, `countsHTML`, `curScheduleHTML`, `renderDeptGrid`, `quickStage`, `renderManage`, `deleteDept`
- 모달: `scheduleInputsHTML`, `accountRowHTML`, `addAccountRow`, `openDeptModal`, `saveDept`
- 일괄 등록: `genUser`, `parseBulkLines`, `renderBulkPreview`, `saveBulk`
- **전형 상세 보드(체크리스트)**: `boardColumns(d)`(컬럼·항목 정의), `renderBoardCols(d)`(다음 항목 계산·렌더), `openBoard`, `toggleCheck`, `openReport`/`closeReport`, `closeBoard`
- 통계: `repDate`, `computeStats`
- 일정표: `defMonth`, `calendarHTML`, `adminCalMove`, `teamCalMove`, `legendHTML`, `interviewAgendaHTML`, `renderScheduleTab`
- 메일: `MAIL_TEMPLATES`, `initMail`, `onMailDeptChange`, `currentTemplate`, `renderMailFields`, `renderMailOut`, `goMailFor`
- 팀장 화면: `renderTeam`
- 공통: `copyText`, `fallbackCopy`, `toast`

## 8. 이미 확정된 의사결정 / 제약
- 단일 HTML 프로토타입, 백엔드/DB 없음(데이터는 브라우저에 저장).
- 부서/직무는 **샘플 데이터**(화면에서 직접 입력·일괄 등록해 사용).
- 직무명(역할명)은 사용하지 않음 → 메일 제목 등은 "부서명 + 채용형태" 기준.
- 팀장은 조회 전용, 관리자만 편집. 한 부서에 계정 여러 개 등록 가능(부서 간 아이디 중복은 금지).

## 9. 다음에 할 수 있는 작업(백로그)
- 사용자 제공 **정형 메일 템플릿**으로 교체.
- 체크리스트 항목 **커스터마이징 UI**(항목 추가/수정/순서 변경).
- 실제 부서·직무 데이터 반영.
- **실서버 배포**(인증·DB·호스팅) — 다중 사용자 동시 접속이 필요해질 때.
- 캘린더 알림/내보내기, 메일 mailto 연동 등.

## 10. 검증 방법
- 핵심 로직은 Node로 단위 테스트(예: 날짜 마스크 `maskDate`, 통계 집계, 캘린더 날짜 배치, 체크리스트 다음 항목 계산)하고, 브라우저로 직접 열어 확인하면 됩니다.

**먼저 `recruit-status.html`을 읽고 위 구조를 확인한 뒤, 제가 요청하는 수정을 이어서 진행해 주세요.**
