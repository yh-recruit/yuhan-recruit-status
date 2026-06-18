# Firebase 실시간 공유 설정 (무료)

> ✅ **이 설정은 2026-06-17에 완료되었습니다** (프로젝트: `yuhan-recruit`).
> 접속 주소: https://yh-recruit.github.io/yuhan-recruit-status/
> 아래는 참고용 기록입니다. 새 프로젝트를 다시 만들 때만 따라 하면 됩니다.

이 문서대로 따라 하면 **여러 명이 같은 데이터를 실시간으로** 보고 쓸 수 있게 됩니다.
신용카드 필요 없음. 구글 계정만 있으면 됩니다.

> 설정 전까지는 앱이 기존처럼 **이 브라우저(localStorage) 단독 모드**로 정상 작동합니다.
> 아래 6단계를 마치고 키를 넣으면 그때부터 실시간 공유가 켜집니다.

---

## 1. Firebase 프로젝트 만들기
1. https://console.firebase.google.com 접속 → 구글 로그인
2. **프로젝트 추가** 클릭
3. 프로젝트 이름: 예) `yuhan-recruit` → 계속
4. "Google 애널리틱스" 화면 → **사용 안 함**으로 두고 → 프로젝트 만들기
5. 잠시 후 "프로젝트가 준비되었습니다" → 계속

## 2. Firestore 데이터베이스 만들기
1. 왼쪽 메뉴 **빌드 → Firestore Database**
2. **데이터베이스 만들기** 클릭
3. 위치: `asia-northeast3 (서울)` 선택 → 다음
4. 보안 규칙: 일단 **프로덕션 모드**로 시작 → 사용 설정
   (규칙은 5단계에서 정확히 바꿉니다)

## 3. 익명 로그인 켜기 (보안용 최소 장치)
1. 왼쪽 메뉴 **빌드 → Authentication**
2. **시작하기** 클릭
3. 로그인 방법 목록에서 **익명(Anonymous)** 선택 → **사용 설정** 켜기 → 저장

## 4. 웹 앱 등록하고 설정값 복사
1. 왼쪽 상단 **프로젝트 개요** 옆 톱니바퀴 → **프로젝트 설정**
2. 아래로 스크롤 → "내 앱" → **웹 아이콘(`</>`)** 클릭
3. 앱 닉네임: 예) `recruit-web` → **앱 등록** (호스팅 체크는 안 해도 됨)
4. `firebaseConfig` 값이 나옵니다. 이렇게 생긴 부분:
   ```js
   const firebaseConfig = {
     apiKey: "AIzaSy........",
     authDomain: "yuhan-recruit.firebaseapp.com",
     projectId: "yuhan-recruit",
     storageBucket: "yuhan-recruit.appspot.com",
     messagingSenderId: "1234567890",
     appId: "1:1234567890:web:abcd1234"
   };
   ```
5. 이 값을 그대로 복사

## 5. 설정값을 HTML에 붙여넣기
1. `recruit-status.html` 파일을 메모장/편집기로 열기
2. 상단 `<head>` 안의 이 부분을 찾기:
   ```js
   window.YH_FIREBASE_CONFIG = {
     apiKey: "",
     ...
   };
   ```
3. 따옴표 안을 4단계에서 복사한 값으로 채우기 (apiKey, authDomain, projectId, storageBucket, messagingSenderId, appId)
4. 저장

> ⚠️ 이 키들은 비밀번호가 아니라 "주소" 같은 것이라 HTML에 넣어도 됩니다.
> 실제 데이터 보호는 아래 6단계 보안 규칙이 담당합니다.

## 6. Firestore 보안 규칙 설정
1. **Firestore Database → 규칙** 탭
2. 내용을 아래로 통째로 교체 → **게시**
   ```
   rules_version = '2';
   service cloud.firestore {
     match /databases/{database}/documents {
       match /app/state {
         allow read, write: if request.auth != null;
       }
     }
   }
   ```
   → 익명 로그인한 사용자(=우리 앱 접속자)만 데이터에 접근 가능.

---

## 끝! 확인 방법
- `recruit-status.html`을 두 개의 다른 브라우저(또는 두 대의 PC)에서 열기
- 한쪽에서 전형을 수정 → **다른 쪽 화면에 몇 초 안에 자동 반영**되면 성공 ✅

## 다른 사람에게 배포하려면 (무료)
지금은 파일을 각자 열어야 합니다. 주소(URL) 하나로 공유하려면 무료 호스팅이 필요해요. 둘 중 하나:
- **GitHub Pages**: 이미 GitHub에 올려둔 저장소를 Pages로 공개 (가장 간단)
- **Firebase Hosting**: `firebase deploy` 명령으로 배포

→ 이 부분은 준비되면 따로 단계별로 안내해 드립니다.

---

## 참고 / 한계 (v1)
- **동시 편집**: 두 사람이 *정확히 같은 순간* 저장하면 나중 사람이 덮어쓸 수 있음(소규모 팀에선 거의 발생 안 함). 본격적인 충돌 방지는 다음 단계 과제.
- **보안**: 익명 로그인은 "아무 크롤러나 막는" 최소 수준입니다. 지원자 개인정보를 다루므로, 다음 단계에서 **실제 로그인(Firebase Auth)**으로 강화하는 것을 권장합니다.
- **무료 한도**: 소규모 사내 사용은 무료 등급으로 충분합니다(일 5만 읽기/2만 쓰기). 초과 시에도 자동 과금 없이 멈춥니다(카드 미등록 시).
