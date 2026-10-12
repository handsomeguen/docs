# Implementation Plan: 관리자 비밀번호 초기화

**Branch**: `029-admin-password-reset` | **Date**: 2026-10-12 | **Spec**: [spec.md](./spec.md)

## Summary

DB 변경 없음. `/admin`에 GET 검색(`?member=아이디`)과 초기화 Server Action을 더한다.

## 결정

1. `makeTempPassword`(`src/lib/temp-password.ts`): `crypto.getRandomValues`로 고르게 뽑는다(치우치는 값은 버림). 영문과 숫자가 모두 들어갈 때까지 다시 만든다.
2. `resetMemberPassword`(`src/server/account.ts`): 한 트랜잭션에서 회원 행을 잠그고, 관리자면 거절, 아이디·비밀번호 로그인 수단이 없으면 거절, 해시 바꾸기, 세션·로그인 실패 기록 삭제.
3. `adminResetPassword`(`src/app/admin/actions.ts`): `requireAdmin()`(아니면 404) → 값 모양 확인 → 위 함수. 결과로 임시 비밀번호를 돌려줄 뿐 로그·DB에 원문을 남기지 않는다.
4. `ResetPasswordButton`(클라이언트)은 확인을 폼 액션 안에서 묻고 결과를 화면 상태로만 보인다(새로 그리거나 떠나면 사라진다).

## Project Structure

```text
src/lib/temp-password.ts, src/server/account.ts, src/server/admin.ts (findMemberByUsername)
src/app/admin/{page,actions}.tsx, src/components/admin/ResetPasswordButton.tsx
tests/unit/temp-password.test.ts, tests/e2e/admin-reset.spec.ts
```
