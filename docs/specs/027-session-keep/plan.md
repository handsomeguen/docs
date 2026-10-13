# Implementation Plan: 로그인 유지 연장과 정리 작업

**Branch**: `027-session-keep` | **Date**: 2026-10-12 | **Spec**: [spec.md](./spec.md)

## Summary

DB 변경 없음. 서버 화면은 쿠키를 바꿀 수 없다. 화면을 새로 열 때(소프트 이동이 아닐 때)는 라이브러리가 DB만 7일로 늘리고 쿠키는 그대로 두어, 쿠키는 로그인 때의 7일에서 늘지 않았다. 이제 서버의 `getSession()`은 늘리지 않고(`disableRefresh`), 클라이언트가 세션 조회를 불러 라이브러리가 DB와 쿠키를 함께 늘리게(`updateAge` 1시간) 한다.

## 결정

1. `getSession()`이 `rememberMe`를 돌려주고, 레이아웃 헤더가 유지 세션일 때만 `SessionKeeper`(`src/components/auth/SessionKeeper.tsx`)를 그린다.
2. `startSession`(`src/app/(auth)/actions.ts`): 유지면 `auth.$context`의 `dontRememberToken` 이름·경로·secure로 쿠키를 지운다. 남아 있으면 라이브러리가 세션 조회에서 갱신을 건너뛰고, 다른 갱신 때 세션 쿠키를 브라우저 세션 쿠키로 바꾼다.
3. `src/server/session-cleanup.ts`의 `cleanupSessions`(만료 세션 + 마지막 사용 2시간이 지난 유지 안 함 세션)·`cleanupLoginAttempts`를 `scripts/cleanup-posts.ts`가 부른다.
4. `reserveLoginAttempt`는 있던 행도 `ON CONFLICT DO UPDATE`로 잠가, 정리 작업이 그 사이에 행을 지워 조회가 빈 값이 되는 일을 막는다.

## Project Structure

```text
src/components/auth/SessionKeeper.tsx, src/app/layout.tsx, src/server/dal.ts, src/app/(auth)/actions.ts
src/server/session-cleanup.ts, src/server/login-attempts.ts, scripts/cleanup-posts.ts
tests/e2e/session-keep.spec.ts
```
