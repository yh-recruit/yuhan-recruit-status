# 채용전형 현황 시스템 관리

유한양행 채용 전형 현황 시스템 — 부서별 채용 진행 단계를 한 화면에서 파악·관리하는 사내 웹 도구 프로토타입.

## 역할
- 담당: 인사(HR) 업무 IT 개발자. UI 텍스트·도메인 용어는 모두 한국어 인사 실무 기준.

## 작업 파일
- **`recruit-status.html`** — 단일 HTML 파일(인라인 CSS + 바닐라 JS). 프레임워크/번들러 없음.
- 브라우저에서 바로 열면 동작. 빌드/서버 불필요.
- 체험 계정: 관리자 `admin / 0000`, 팀장(예) `sales1 / 0000`.

## 기술 원칙
- 데이터는 브라우저 **localStorage** 저장 (key: `yh_recruit_state_v3`).
  - v1/v2 자동 마이그레이션 + `normalize()`로 스키마 보정.
- **선택적 클라우드 동기화(Firebase Firestore)**: `<head>`의 `window.YH_FIREBASE_CONFIG`에 키를 채우면 ON, 비우면 localStorage 단독 모드(기본). 전체 `state`를 JSON 문자열로 단일 문서 `app/state`에 저장(Firestore 중첩배열·키 제약 회피), `onSnapshot`으로 실시간 반영. `cloudSave()`는 `saveState()`가 호출, `cloudApplySnapshot`은 모달 열려 있으면 `cloud.pending`으로 보류 후 닫히면 `rerenderCurrent()`. 익명 인증(`signInAnonymously`) 후 접근. 자기 쓰기 echo는 `hasPendingWrites`로 무시. 설정·배포·보안규칙은 `FIREBASE-SETUP.md` 참고. (동시편집 충돌 방지·실제 로그인 보안은 다음 단계 과제)
- 폰트: Pretendard(CDN). 색상은 CSS 변수(`--yh-green` `#00833D` 메인, 골드 보조).
- 클라이언트 측 게이팅 수준의 로그인(보안 로그인 아님, 프로토타입).

## 채용 절차 (STAGES)
`공고준비 → 공고진행 → 서류전형 → AI역량검사 → 면접전형 → 입사` (단계명 모두 붙여쓰기 통일. 구버전 `공고 진행` 키는 `normalize()`가 `공고진행`으로 자동 이관)
- `STAGE_SCHED`: 공고준비=날짜 없음, 입사=시작일만, 나머지(면접전형 포함)=시작/마감. 면접전형 마감일이 비면 시작일 당일.
- 면접전형은 **면접 세션 배열**(`sessions`)로 다일·회차 관리: 각 세션 `{round(통합/1·2·3차), date, ts, te, site(본사/연구소/공장), place, panel, obs(배석자)}`. 시간 입력은 **숫자만 쳐도 `maskTime`이 자동 `HH:MM`**(예 1000→10:00). **장소는 site 셀렉트(본사/연구소/공장, `IV_SITES`) + place 상세 2칸** — 본사는 표기 생략(`siteTagHTML`/`sessPlaceHTML`이 연구소·공장만 색 태그로 강조). 구버전 세션엔 `normalize()`가 place 텍스트로 site 추정(없으면 본사). **배석자는 회차(세션)별**(`s.obs`) — 구버전 공통 `observers`는 `normalize()`가 각 세션 obs로 이관. 일정표 면접 일정 상세는 같은 부서 1·2차를 **한 행으로 병합**(셀 내 줄바꿈).
- **캘린더 공휴일**: `HOLIDAYS`(YYYY-MM-DD→명칭, 2026 기준) + `holidayName()`. 전형 일정표·업무 일정표 캘린더에서 공휴일 날짜를 빨강(`.cal-hol`)으로 표시(일요일과 동일 톤). 신정/설/삼일절/어린이날/부처님오신날/현충일/광복절/추석/개천절/한글날/성탄절 + 대체공휴일 포함.
- **전형 일정표 면접 바 장소 표기**: `calendarHTML` 면접전형 바에 본사 외 장소(연구소/공장)를 `.cb-site` 태그로 표기(`g.sites` Set 수집) — 외부 이동 필요 여부를 바만 보고 판단. 본사는 표기 없음.

## 데이터 모델
**전형(부서) 객체**: `{ id, name, type(채용형태), reqCount(요청인원), ftTO(정규직 전환 TO), stage, completed(bool), completedAt, schedule{[stage]:{s,e,place,panel,observers,timeStart,timeEnd,sessions[]}}, accountUsers[user 참조], notes(=비고), counts{applicants,doc,ai,hired}, checklist{"colId:idx":true}, updated }`
**계정 객체(전역 `state.accounts`)**: `{ user, code, name, title, dept(정보용 표기), role("normal"|"admin"), createdAt }`
- **계정은 전형과 분리된 전역 자원.** '계정 관리' 탭에서 전형 오픈과 무관하게 생성하고, 전형 추가·편집 모달의 **계정 선택 체크박스**(`accountPickHTML`)로 전형에 등록(`d.accountUsers`에 user 참조 추가). **한 계정을 여러 전형에 등록 가능**(예: 인사팀장 `hrlead` 시드가 영업1팀·마케팅팀·중앙연구소 3곳에 등록). 전형 삭제해도 계정은 유지. 계정 삭제 시 모든 전형의 `accountUsers`에서 참조 제거.
- 구버전 `dept.accounts`(전형 종속) → `normalize()`가 전역 `state.accounts` + `dept.accountUsers`로 자동 이관(user 기준 dedup).
- `role`: `"admin"`이면 로그인 시 관리자 권한(전체 편집), `"normal"`이면 조회 전용. ADMIN 상수(admin/0000)는 기본 관리자. 계정 모달 '구분' 셀렉트로 일반↔관리자 전환, 기본값 일반. **로그인 시 `session.user`에 아이디 저장(admin role 계정 포함)** → 상단바·기타업무 등록자 표기에 사용(`currentUserName()`: 계정 name, ADMIN 상수는 "시스템 관리자").
- `dept`(소속 부서)는 **정보용 자유 텍스트** — 조회 권한과 무관(권한은 `accountUsers` 등록 여부로 결정).
- `notes`는 화면상 **비고**(특이사항). 카드의 진행 상태는 체크리스트에서 자동 계산(`nextAction`)되는 "지금 진행할 차례"로 표시. `toggleCheck`/`closeBoard`에서 `renderAll()` 호출로 카드 자동 갱신.
- `completed`=true면 대시보드/요약에서 제외, '완료된 전형' 탭에 표시. `done` 컬럼 마지막 항목((선택)그룹웨어 게시판 게재) 완료가 트리거.
- 채용형태(EMP_TYPES): 인턴 / 경력 / 계약직(일반·대체·보훈·장애).

## 화면(탭) 구성
1. **전형 현황(대시보드)**: 단계 요약 칩(필터), **채용 현황표 — 좌7:우3 2박스**(`renderStatusBoard`, `.sb-wrap` flex, ≤760px 세로 적층): **좌(부서 목록)**=단계별 행, 각 단계 안은 **채용형태별 그룹**으로 `형태 : 팀 / 채용요청인원 N(정규직 TO M)` 표시 + 헤더 '자세히 보기'=`openTodoOverview` 모달(부서별 '지금 진행할 차례' 단계순). **우(채용요청인원 계)**=단계별 `계 합계(정규직 TO 분리)` + 채용형태별 `형태 합계(정규직 TO 분리)` — 서류전형·면접전형 등 단계별 요청 인원 합계를 한눈에(인턴 등 정규직 TO 값은 별도 산정). **검색/필터 툴바**(`deptSearch` 부서명·채용형태 텍스트 검색 + `typeFilter` 채용형태 셀렉트 + 단계칩 `stageFilter`, `clearDeptFilters`로 초기화, `deptCount` 건수), 부서 카드(진행중만), 카드 클릭→전형 상세 보드. 카드에 **전형 진행률 바**(`deptProgress`/`progressHTML` — 체크리스트 완료/전체 기준 %, 100%는 골드) + **D-day 배지**(`deptDday`/`ddayMetaHTML` — 현재 단계의 마감일/면접일/입사일 기준, 3일↓ 빨강·당일 노랑·경과 회색) + 체크리스트 기반 "지금 진행할 차례" 자동 표시 + 비고. (진행률·D-day는 팀장 조회 카드에도 표시) 헤더 **🖨 인쇄/PDF**(`printStatusReport` — 새 창에 현황표 렌더 후 `window.print()`, PDF 저장 겸용)·**⬇ 백업/⬆ 복원**(`exportBackup` 전체 state를 `yuhan-recruit-backup-YYYY-MM-DD.json`으로 다운로드 / `importBackup` 파일 선택→확인 후 `normalize` 거쳐 교체, `state`는 `let`이라 재할당). **카드는 그립(⠿) 드래그로 순서 변경**(`moveDept`, `state.departments` 배열 재정렬 후 저장 — 관리자·일반 화면 공통). 카드 하단 액션 = 단계 셀렉트·상세 편집·**완료**(초록)·**삭제**(빨강), 둘 다 확인 절차. **완료**(`completeDept`)=체크리스트 전체 완료 처리 + `입사` 단계 전환 + `completed=true`로 완료탭 이동. `curScheduleHTML`: 기간 단계라도 마감일 비면 시작일+"(당일)"만 표시.
2. **완료된 전형**: 보드의 '채용 종료 → 그룹웨어 게시판 게재' 완료 시 카드가 이 탭으로 이동(`d.completed`). 통계에는 계속 포함. '전형 재개' 가능.
3. **계정 관리**(구 부서 관리): 전역 계정 표(체크박스/구분/아이디/성명/부서/직책직급/등록일/관리). **정렬=관리자 먼저 그 다음 일반**(시스템 admin 최상단, `renderManage` 내 rank), **검색**(아이디·성명·부서·직책)·**페이지네이션(기본 20, 50/100/전체)**(`mgSearch/mgPage/mgPageSize`). 구분=role(관리자/일반), 부서 셀은 정보용 dept + "전형 N건 등록" 표시. 계정 추가/계정 일괄 관리/**체크박스 다중선택 후 '선택 삭제'**(`deleteSelectedAccts`, 전체선택 `toggleAllAcct`). **계정 생성은 전형과 독립** — 여기서만 만들고 전형 추가·편집에서 등록(**전형 일괄 등록은 계정을 자동 생성하지 않음**). ('전형 추가'·일괄 등록은 대시보드 헤더, 전형 삭제는 상세편집 모달.)
4. **상세편집 모달**('전형 추가'/'상세편집'): 부서·일정·면접 세션·**등록 계정 선택(transfer UI)**·비고·인원 현황. 등록 계정은 **좌(전체 계정+검색)/우(등록됨) 2단 + 화살표(→ 등록 / ← 해제)** 방식(`accountPickHTML`+`renderAvail`/`renderPicked`, 작업 상태 `pickUsers`). 면접 시작/종료 시간은 **타이핑 입력**(텍스트, 예 14:00). 하단 '삭제'로 전형 삭제(계정은 유지). (날짜 8자리 입력 자동 정렬)
5. **전형 상세 보드**: 채용담당자 체크리스트 5컬럼(apply/doc/ai/interview/done), 다음 항목 자동 안내, 채용 성과 리포트 팝업. (면접전형 컬럼에 '기면접자 확인' 포함) 각 컬럼에 **'일괄 완료'** 버튼(`completeColumn`)=그 컬럼 전 항목 완료+다음 단계 전환. 컬럼 마지막 항목 체크 시도 **자동으로 다음 단계 전환**(`advanceStageIfColumnDone`). 역으로 `d.stage`를 정하면 **이전 단계 컬럼 체크리스트가 자동 완료 반영**(`autoCheckPriorColumns` — 일괄 등록·단계 변경·저장 시 호출). 전형 추가/일괄 등록 모두 등록 후 **입사→공고준비 순 정렬**(`sortDeptsByStage`, 단일 신규 추가·일괄 등록 시 호출). 일괄 등록 그리드는 부서명/채용형태(기본 인턴)/요청인원/**공고 시작·마감**/현재 단계. 상세편집 2행에 **헤드헌팅**(`d.headhunt` boolean, 기본 해당없음) 셀렉트. 대시보드 헤더 **'일괄 일정 편집'**(`openBulkSchedModal`)=진행 중 부서 다중선택 + 단계별(공고진행~입사) 시작/마감 입력 → 입력한 항목만 선택 부서에 일괄 반영(`saveBulkSched`). **간편카드 헤드헌팅은 O일 때만 표시**(X 숨김). 일정표 달력은 **전/다음 달 날짜를 흐릿하게**(`.cal-out`) 표시. 면접전형 체크리스트 첫 항목 **'(경력)레퍼런스 체크'**.
6. **메일 작성**: 부서 드롭다운 라벨 = `공고시작일(YY.MM.DD) 부서명-채용형태`. 템플릿 = **🔔 부서안내 2종**(공고 문안 확인 요청 / 평가자 계정·서류전형 안내) + **🔔 입사자안내 1종**((경력) 입사 전 제출서류·절차 — 가변 항목만 입력, 네이버폼·검진·서류·교육 안내 고정 양식. 인턴 등 다른 형태는 라벨에 `(형태)` 표기해 추가 예정) + 기존 보고 메일 6종. 모두 사용자 실제 양식 기반. textarea 필드 지원(자격요건·담당업무 등 줄별 → `·`/`1)` 자동 변환). *기존 보고 메일은 틀 확정 후 정리 예정.*
7. **통계**: 기간 필터, KPI, 채용형태 분포, 전형 퍼널(완료 전형 포함). **부서별 상세 표**=컬럼(부서/채용형태/채용요청인원/현재 전형 단계/기준일(공고시작일)/지원자/채용(예정)인원), **헤더 클릭 정렬(오름/내림)**·**팀 검색**·**페이지네이션(기본 15, 30/50/100/전체)**(`renderDeptDetailTable`+`STAT_COLS`, 상태 `statSort/statPage/statPageSize/statSearch`).
- 간편카드 메타에 **헤드헌팅 O/X**(`d.headhunt`) 표시(등록 계정 수 대신). **일반 계정 화면은 완료(`d.completed`) 전형을 숨김**(진행 중만 표시).
8. **일정표**: 월간 캘린더 — **탭 진입 시 항상 오늘 달로 표시**(좌우 화살표로 이동). 면접 일정 상세(`interviewAgendaHTML`)는 **오늘 이후 예정 면접만**, 가까운 면접일이 위로(지난 면접·과거 세션 숨김). 기간 단계는 **연속 간트 바**(주 경계서 분할, 시작·끝 셀에 진한 컬러 border 표시), 동일 단계+동일 기간은 한 바로 병합·마감일 다르면 분리, 레인 누적(자름 없음). 면접전형=빨강(`#C0392B`), 입사=녹색으로 구분. 바 클릭=상세 팝업, 날짜칸(흰 바탕) 클릭=그날 전형 현황 팝업.
9. **기타 업무**(관리자): 채용 외 자유 입력 할 일 관리(`state.tasks`=전역 배열, `{id,title,day,done,important,author,createdAt,color,subs[{id,text,done}]}`). **`author`=등록 시점 로그인자 이름**(`currentUserName()`, `normalize`가 구버전 빈값 보정). 등록: **‘+ 오늘 / + 내일’ 빠른 버튼** + **날짜 지정(`<input type=date>` 타이핑·달력) ‘+ 지정일’**(`addTask('today'|'tomorrow'|'date')`, ‘중요’ 체크 시 `important=true`, **색상 셀렉트**(`TASK_COLORS`: 기본/파랑/보라/주황/청록/회색)로 업무 종류 구분 — `color`, `taskColor()`; 중요는 색상 무관 빨강). **업무 기준일은 매일 08:30에 새 ‘오늘’로 전환**(`workToday()`), 미완료 업무 다음 날 자동 **이월**(`carryOverTasks`, '이월' 배지). **필터 탭 전체/오늘/내일/완료**(`taskFilter`): **전체=미완료 전 업무를 가까운 날짜순(미래가 아래로)·20개씩 페이지네이션**(`taskPage`/`TASK_PAGE_SIZE`), 오늘=`day<=오늘`, 내일=`day===내일`(그보다 먼 미래는 전체/일정표에서 조회), 완료=완료만. 카드에 **날짜칩(D-day)·색상 점·날짜 변경(`<input type=date>`→`setTaskDate`)·색상 변경(`setTaskColor`)·오늘↔내일 이동·상세 업무 % 진행률 바**(상세 업무=서브태스크, UI 명칭 ‘상세 업무’로 통일). 그립(⠿) 드래그 정렬(`moveTask`), 중요=빨강(`toggleImportant`). 모든 변경은 `afterTaskChange()`가 기타 업무·업무 일정표·상세 모달을 동시 갱신.
10. **업무 일정표**(관리자, 기타 업무 옆 탭 `worksched`): `state.tasks`를 **월간 캘린더**로 표시(`renderWorkSched`/`workCalendarHTML`, 탭 진입 시 오늘 달). 각 업무 = 해당 `day` 셀의 **색상 칩**(`taskColor`, 중요=빨강 굵게, 완료=취소선). **칩 클릭→상세 모달**(`openWkTask`/`renderWkTaskBody`: 완료 토글·중요 토글·상세 업무 체크/추가/진행률). **빈 날짜 셀 클릭→그 날짜로 업무 추가 모달**(`openWkAdd`/`wkAddSubmit` — 내용·색상·중요). **칩을 다른 날짜 셀로 드래그→`day` 변경**(`wkChipDragStart`/`wkCellDrop`→`setTaskDate`, 기타 업무에도 즉시 반영). 완료 체크는 기타 업무·일정표·모달 **양방향 연동**(단일 `state.tasks` 소스 + `afterTaskChange`). 공휴일·일요일 빨강. **달력 폭 고정**: 칸은 `minmax(0,1fr)`·`overflow:hidden`, 칩 제목은 길면 `…`(ellipsis)로 잘라 좌우 레이아웃 불변, 위아래로만 쌓임.
- **Firebase 비동기 로딩(중요)**: SDK 3종을 동적 `<script async>`로 순차 주입(`app→auth→firestore`)하고, 메인 스크립트는 `waitFirebase` 폴링으로 준비되면 `cloudInit()`. **동기 `<script src>`로 두면 사내 프록시가 gstatic을 막거나 지연시킬 때 페이지 전체가 멈춰 '접속 불가'가 됨** — 비동기 로딩이면 CDN이 막혀도 앱은 localStorage 모드로 즉시 표시.
- **일반 계정 조회 화면**: 로그인 시 본인이 등록된 전형 목록(카드) → 클릭하면 해당 전형 상세 조회(읽기 전용). 여러 전형 등록 시 모두 표시. role=admin 계정은 관리자 화면으로 진입.

## 디자인/에셋
- CI는 `assets/` 실제 파일 사용: `yh-symbol.jpg`(버드나무 심볼=로고 마크/배지), `yh-logo.jpg`(가로 워드마크), `yh-hq.jpg`(본사 사옥, 로그인 배경; System.Drawing으로 1200px 경량화). **임의 SVG 재디자인 금지 — 원본 파일 그대로.**

## 확정된 제약
- 단일 HTML(+`assets/` 이미지) 프로토타입, 부서/직무는 샘플 데이터, 직무명 미사용.
- 일반 계정 조회 전용, 관리자(ADMIN 상수 또는 role=admin 계정)만 편집.

## 검증
- 핵심 로직(날짜 마스크 `maskDate`, 통계 집계, 캘린더 배치, 체크리스트 다음 항목)은 Node 단위 테스트 가능. 브라우저로 직접 확인.

## 백로그
- 나머지 전형 단계별 부서안내 메일 템플릿 추가(사용자가 단계별로 제공 중) + 틀 확정 후 기존 보고 메일 6종 정리 / 체크리스트 커스터마이징 UI / 실데이터 반영 / 실서버 배포(인증·DB) / 캘린더 알림·내보내기·mailto 연동.
- (완료) 전형 진행률 % 바 · D-day 배지 · 대시보드 검색/필터 · 기타업무 등록자 표기 · 인쇄/PDF · JSON 백업/복원.

> 상세 코드 구조 맵은 `HANDOFF.md` 참고.
