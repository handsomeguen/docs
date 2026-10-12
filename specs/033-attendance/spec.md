# Feature Specification: 출석 보상

**Feature Branch**: `033-attendance`

**Created**: 2026-10-12

**Status**: Implemented

**Input**: User description: "계속 진행해줘" (032 다음 게임 순서)

**다루는 명세 요구사항**: GAME-04 출석 보상(자동 출석, 7일 주기 보상표, 출석 체크 화면), GAME-03 가입 직후 헤더 🪙 110, GAME-07 `📮 출석`, ERD 3.12 `attendances`·`attendance_rewards`.

**명세와 다른 점**:
- 광장 게시판·휴대폰 ☰ 메뉴가 없어 입구는 헤더 `📮 출석`(좁은 화면은 📮만) 하나다.
- 자동 출석은 `getViewer()`가 아니라 레이아웃 헤더가 그릴 때 한다(`ensureTodayAttendance`). 실패하면 `자동 출석 실패`를 로그에 남기고 화면은 그대로 연다.
- 출석 화면 가운데 카드는 열린 질문의 제안대로 한 줄 `출석 완료! 오늘 N일차`와 `오늘 받은 보상 ✨ N · 🪙 N`으로 줄였다. 광장으로 나가기 버튼은 없다.
- 레벨업 알림(GAME-06)은 다음 단계라 출석으로 레벨이 올라도 아직 알림이 없다.

## User Scenarios & Testing *(mandatory)*

### User Story 1 - 자동 출석 (Priority: P1) 🎯

1. 로그인한 회원이 그날(한국 날짜) 처음 화면을 열면 버튼 없이 출석이 기록되고 일차 보상이 원장 `attendance`(`ref_id` = 날짜)에 들어간다. 헤더 코인·레벨은 보상까지 반영된 값.
2. 일차: 처음 1, 어제 1~6일차면 +1, 어제 7일차면 1, 하루 이상 빠지면 1. 보상 1~7일차 ✨ 10·10·15·15·20·20·30, 🪙 10·20·30·40·50·70·100.
3. 같은 날 다시 열어도, 탭 여러 개가 동시에 열어도 출석 1행·원장 1줄(회원 행 → 회원 잠금, 기본 키, 부분 고유 인덱스).
4. 세션이 지워져도 출석은 남고 `session_id`만 비워진다. 화면을 연 채 0시를 넘기면 이동·탭 다시 보기 때 다시 그려 출석한다(`AttendanceDayWatcher`).

### User Story 2 - 출석 체크 화면 (Priority: P2)

1. `/attendance` `📮 출석 체크`: 🎁 `출석 완료! 오늘 N일차`, `오늘 받은 보상 ✨ N · 🪙 N`. [출석하기] 버튼 없음.
2. 7칸 보상표(오늘 칸 노란 테두리·굵게, `aria-current`), 375px에서도 한 줄.
3. 이번 달 달력(일~토), 출석한 날은 🌟와 초록 칸, 오늘은 노란 테두리, `이번 달 N일 출석 · 현재 N일차`.
4. 방문자는 첫 화면으로.

## Success Criteria *(mandatory)*

- **SC-001**: `tests/unit/game.test.ts`(출석 일차·달력), `tests/e2e/attendance.spec.ts`가 통과하고 기존 시험이 그대로 통과한다.
