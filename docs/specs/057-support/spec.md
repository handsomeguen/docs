# Feature Specification: 버그·문의

**Feature Branch**: `feature/support` (코드: handsomeguen/blogville `feature/support`, `develop` 대상)

**Created**: 2026-10-13

**Status**: Implemented

**Input**: 사이트 아래 막대를 눈에 띄게 바꾸고, 회원이 버그·문의를 보내면 관리자나 정해진 시간마다 도는 자동 처리 프로그램이 답하게 한다

**다루는 명세 요구사항**: 새 영역 **4.8 버그·문의(SUP)** — SUP-01 아래 막대 버튼, SUP-02 문의 쓰기·내 문의, SUP-03 관리자 처리, SUP-04 자동 처리 API. GAME-08 알림에 `feedback_reply`를 더한다.

**명세와 다른 점**: 명세에 없던 기능이다. 문의에는 보상(GAME-05)이 없다.

## User Scenarios & Testing *(mandatory)*

### User Story 1 - 아래 막대 (Priority: P1) 🎯

1. 모든 화면 아래 막대에 [🆕 업데이트 소식](`/updates`)과 [🐞 버그·문의하기](`/support?from=<지금 주소>`). 문의 화면에서는 `?from=`을 붙이지 않는다.
2. 최신 GitHub 릴리스가 3일 안에 나왔으면 [업데이트 소식] 옆에 빨간 `NEW`. 릴리스 목록은 `/updates`와 같은 10분 캐시(`listReleases`)를 쓰고, Suspense 안에서 그려 GitHub이 늦어도 화면을 막지 않는다.
3. 휴대폰: 두 버튼이 반씩 한 줄, 44px, 가로 스크롤 없음, 회원의 아래 탭에 가리지 않는다 (body 아래 여백, 040-phone-tabs).
4. PC 마을에는 막대가 없다(038-town 규칙 그대로). ☰ 메뉴 아래쪽에 `🐞 버그·문의`(`/support?from=/town`).

### User Story 2 - 문의 보내기·내 문의 (Priority: P1)

1. `/support`: 종류 라디오(`🐞 버그` 기본 / `💬 문의`), `제목`(1~100자, 한 줄로 합침), `내용`(1~2000자, 글자 수 `N/2000자`), [보내기].
2. 보내면 `접수했어요. 답변이 달리면 알림으로 알려 드릴게요`, 칸이 비고 `내 문의 N` 맨 위에 나온다.
3. `내 문의`: 최신순 10개씩(`?page=N`, `from`은 유지), 상태 표시 `접수`/`확인 중`/`답변 완료`/`고침`, 종류, 제목, 시각, 내용(글자 그대로), 답변이 있으면 `💌 답변`과 답한 시각.
4. 방문자: 칸 대신 `로그인하면 버그·문의를 보낼 수 있어요`. [로그인]은 `loginHref`로 이 화면(보던 주소 포함)에 돌아온다. `내 문의`는 없다.

- 한 회원은 한 시간(지금부터 60분 전 이후)에 5번까지. 6번째는 `문의는 한 시간에 5번까지 보낼 수 있어요. 잠시 뒤 다시 보내 주세요`, 쓴 내용은 남는다. 서버가 회원 행을 잠그고 세므로 동시에 보내도 넘지 않는다.
- 보던 주소는 `safeNext`를 통과한 이 사이트 안의 주소만 저장한다(`/support` 자신 빼고). 브라우저 정보는 서버가 요청 헤더 `User-Agent`에서 받아 300자까지.

### User Story 3 - 관리자 처리 (Priority: P1)

1. `/admin`에 `🐞 버그·문의 새 문의 N건`(새 문의가 있으면 빨강)과 [문의 보기](새 문의가 있으면 `?status=new`).
2. `/admin/feedback`: `전체`·`접수`·`확인 중`·`답변 완료`·`고침` 거르기, 최신순 20개씩. 문의마다 쓴 사람(아이디·닉네임), 종류, 제목, 내용, 보던 화면, 브라우저, 보낸 때, 답한 때(관리자/자동 처리).
3. 상태를 고르고 `답변`을 적어 [저장] → `저장했어요` (알림을 보냈으면 `저장했어요. 쓴 사람에게 알림을 보냈어요`). 답변을 비우고 저장하면 답변을 지운다.
- 상태를 그대로 두고 새 답변만 달면 `접수`·`확인 중`은 `답변 완료`로 오른다 (`nextStatus`).
- 관리자가 아니면 `/admin/feedback`은 404, 저장 요청을 조작해도 바뀌지 않는다 (`requireAdmin`).

### User Story 4 - 알림 (Priority: P2)

- 새 답변이 달리거나 답변이 바뀌거나, 상태가 `답변 완료`·`고침`으로 바뀌면 쓴 사람에게 `🐞 문의에 답변이 달렸어요: {제목}`. 누르면 `/support`. 한 번 저장에 알림 하나 (`shouldNotifyReply`).
- `확인 중`으로만 바꾸거나, 같은 답변을 다시 저장하거나, 답변을 지우면 알림이 없다.
- 행동한 사람이 없는 알림이다 (`actor_id` 없음, `feedback_id` 있음). 문의가 지워지면 알림도 지워진다. 마을 ☰ 메뉴 최근 알림에도 같은 문구.

### User Story 5 - 자동 처리 API (Priority: P2)

정해진 시간마다 도는 자동 처리 프로그램(에이전트)이 새 문의를 읽고 답한다. 쿠키를 쓰지 않고 열쇠만 쓴다.

**인증**: `Authorization: Bearer <FEEDBACK_BOT_TOKEN>`. 환경 변수와 SHA-256 해시끼리 `crypto.timingSafeEqual`로 비교한다. 다른 인증 방법은 없다(관리자 쿠키로도 못 들어온다).

| 상황 | 응답 |
|---|---|
| `FEEDBACK_BOT_TOKEN`이 없거나 32자보다 짧음 | 모든 방법·경로 `404` `Not Found` (API가 없는 것처럼) |
| 헤더가 없거나 `Bearer <열쇠>` 모양이 아니거나 열쇠가 다름 | `401` `{ "error": "열쇠가 없거나 맞지 않아요" }`, `WWW-Authenticate: Bearer` |

**`GET /api/feedback?status=new,in_progress&limit=50`**

- `status`: `new`·`in_progress`·`answered`·`fixed`를 쉼표로. 없으면 전체, 모르는 값이면 `400`.
- `limit`: 1~100, 없으면 50, 아니면 `400`.
- `200` `{ "items": [{ "id": 1, "kind": "bug", "title": "…", "content": "…", "pagePath": "/feed", "userAgent": "Mozilla/…", "status": "new", "reply": null, "createdAt": "2026-10-13T07:00:00.000Z", "author": { "username": "bori" } }] }` — 오래된 것부터. 쓴 사람은 아이디만(회원 ID·닉네임 없음).

**`POST /api/feedback/:id`** 본문 JSON `{ "status"?: "new"|"in_progress"|"answered"|"fixed", "reply"?: "1~2000자" }`

- 둘 중 하나 이상, 다른 칸은 받지 않는다(zod `strictObject`). 답변은 앞뒤 공백을 뗀 뒤 1~2000자(자동 처리는 답변을 지우지 못한다).
- 답변은 `replied_at`·`replied_by = 'bot'`을 남긴다. 상태·알림 규칙은 관리자와 같다.
- `200` `{ "item": { "id": 1, "status": "answered", "reply": "…", "repliedAt": "…", "repliedBy": "bot", "notified": true } }`
- 잘못된 JSON·본문 `400`, 숫자가 아닌 ID·없는 문의 `404`, 16KB 넘는 본문 `413`.
- 응답은 모두 `Cache-Control: no-store`. 목록 경로의 POST 등 쓰지 않는 방법은 `404`.

**배포**: `.env.example`에 빈 `FEEDBACK_BOT_TOKEN`. GitHub 시크릿 `FEEDBACK_BOT_TOKEN`이 있으면 `deploy.yml`이 서버 스크립트에 넘기고 `deploy/run.sh`가 `app.env`에 적는다. 없으면 API는 꺼져 있다.

### 탈퇴 (AUTH-06)

- 탈퇴한 회원의 문의는 지워진다 (`feedback.user_id` CASCADE, 알림도 함께).

### 화면

- 375px에서 가로 스크롤이 없고 버튼·라디오는 44px 이상. 문의 내용·답변은 글자로만 보여 준다 (NF-03).

## Success Criteria *(mandatory)*

- **SC-001**: `tests/unit/support-rules.test.ts`(검사·보던 주소·브라우저 정보·한 시간 구간·상태와 알림 규칙·API 조건·열쇠 확인·NEW)와 `tests/e2e/support.spec.ts` 10개가 통과하고, 알림·관리자·방명록·업데이트 소식·휴대폰·아래 탭·마을 시험이 그대로 통과한다.
- **SC-002**: 일반 회원·방문자는 `/admin/feedback`이 404이고, 조작한 관리자 요청과 열쇠 없는·틀린 API 요청은 DB를 바꾸지 않는다.
- **SC-003**: 한 회원이 한 시간에 6번째 문의를 넣을 수 없다.
