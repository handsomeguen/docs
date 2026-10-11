# Implementation Plan: 관리자

**Branch**: `012-admin` | **Date**: 2026-10-11 | **Spec**: [spec.md](./spec.md)

## Summary

DB 변경 없음 (`users.role`은 처음부터 있음). `requireAdmin()`(`src/server/dal.ts`, 아니면 `notFound()`), `src/server/admin.ts`(통계·최근 가입·최근 글), 화면 `src/app/admin/page.tsx`, Action `adminDeletePost`, `AdminDeleteButton`(확인 창), 헤더 👑, `scripts/create-admin.ts`(`npm run admin:create`).

## 결정

1. **오늘 새 글**: 한국 날짜 0시(`date_trunc('day', now() at time zone 'Asia/Seoul')`) 이후.
2. **오래된 글 삭제**: 명세의 열린 질문 그대로 두었다 (목록은 최근 30일·30개).

## Project Structure

```text
src/server/dal.ts (requireAdmin), src/server/admin.ts
src/app/admin/page.tsx, src/app/admin/actions.ts
src/components/admin/AdminDeleteButton.tsx
src/app/layout.tsx                # 👑 관리자, 좁은 화면 줄바꿈
scripts/create-admin.ts, package.json (admin:create), .env.example
tests/e2e/admin.spec.ts
```
