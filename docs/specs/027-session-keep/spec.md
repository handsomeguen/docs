# Feature Specification: 로그인 유지 연장과 정리 작업

**Feature Branch**: `027-session-keep`

**Created**: 2026-10-12

**Status**: Implemented

**Input**: User description: "계속 진행해줘" (026 다음 순서, `plans/gap-survey-2026-10-12.md` 14번)

**다루는 명세 요구사항**: AUTH-09 `SessionKeeper`(유지 세션을 쓰는 동안 다시 7일로, `GET /api/auth/get-session` 10분에 한 번까지), 유지로 로그인할 때 남은 `dont_remember` 쿠키 삭제, AUTH-09·NF-10·NF-26 열린 질문 "만료 세션과 오래된 로그인 실패 기록을 지울까?"

**명세와 다른 점**:
- 정리는 자동 실행 없이 기존 `npm run posts:cleanup`(하루 한 번 손으로)에 더했다.
- 세션의 IP·브라우저 정보(`ip_address`, `user_agent`) 저장은 그대로 둔다(열린 질문 유지).

## User Scenarios & Testing *(mandatory)*

### User Story 1 - 쓰는 동안 로그인이 풀리지 않는다 (Priority: P1) 🎯

1. **Given** [로그인 상태 유지]로 로그인한 회원, **When** 화면을 열거나 탭으로 돌아오거나 열어 둔 채 10분이 지나면, **Then** 세션 조회를 한 번 부르고, 마지막 연장에서 1시간이 지났으면 DB 세션과 쿠키가 다시 7일이 된다.
2. 10분 안에는 다시 부르지 않는다(브라우저 저장소의 시각, 못 쓰면 그 화면에서만 센다).
3. **Given** 유지 안 함 세션, **Then** 부르지 않는다(2시간 규칙 그대로).
4. 서버가 화면을 그릴 때는 늘리지 않는다(쿠키를 못 바꾸므로 DB만 늘면 연장이 사라진다).
5. **Given** 유지 안 함으로 쓰던 브라우저, **When** 유지로 다시 로그인, **Then** `dont_remember` 쿠키가 지워진다.

### User Story 2 - 오래된 기록 지우기 (Priority: P2)

1. `npm run posts:cleanup`이 만료 세션(`expires_at < now()`)과 마지막 사용 2시간이 지난 유지 안 함 세션을 지우고, 살아 있는 세션은 남긴다.
2. 잠겨 있지 않고(`locked_until`이 없거나 지남) 하루 넘게 바뀌지 않은 `login_attempts`를 지운다. 잠금 중이거나 최근 기록은 남긴다.
3. `--dry-run`은 세기만 한다.

## Success Criteria *(mandatory)*

- **SC-001**: `tests/e2e/session-keep.spec.ts`와 기존 `tests/e2e/auth.spec.ts`가 통과한다.
