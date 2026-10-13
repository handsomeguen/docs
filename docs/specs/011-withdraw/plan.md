# Implementation Plan: 회원 탈퇴

**Branch**: `011-withdraw` | **Date**: 2026-10-11 | **Spec**: [spec.md](./spec.md)

## Summary

DB 변경 없음 (회원을 가리키는 외래 키는 이미 CASCADE, `comments.author_id`는 SET NULL). `src/server/account.ts`(`verifyMemberPassword`, `deleteMember`), Action `deleteAccount`(`src/app/settings/account/actions.ts`), `DeleteAccountForm`(`useActionState`), `.btn-danger-solid`. 블로그 관리에 [👤 내 정보] 링크.

## 결정

1. **쿠키 먼저**: `auth.api.signOut`으로 세션과 쿠키를 지운 뒤 회원을 지운다. 회원을 먼저 지우면 세션 행이 사라져 로그아웃 처리가 쿠키를 지우지 못할 수 있다.

## Project Structure

```text
src/server/account.ts
src/app/settings/account/actions.ts, page.tsx
src/components/settings/SettingsForms.tsx   # DeleteAccountForm
src/app/settings/blog/page.tsx              # 👤 내 정보 링크
tests/e2e/withdraw.spec.ts
```
