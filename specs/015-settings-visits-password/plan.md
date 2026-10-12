# Implementation Plan: 방문자 그래프·카테고리 글 수·비밀번호 바꾸기

**Branch**: `015-settings-visits-password` | **Date**: 2026-10-12 | **Spec**: [spec.md](./spec.md)

## Summary

DB 변경 없음. 방문자 그래프는 `getBlogVisitDays(blogId)`(`blog_visits`를 최근 7일 날짜별로 세고 `lastDays`·`fillDays`로 빈 날을 0으로)와 서버 컴포넌트 `VisitChart`(CSS 막대, 그림 라이브러리 없음). 카테고리 글 수는 이미 있던 `getCategoryTree(blog.id, true)`의 `postCount`를 보여 주기만. 비밀번호는 Server Action `changePassword`: `checkPasswordChange`(순수 함수) → `verifyMemberPassword` → `changeMemberPassword`(한 트랜잭션에서 `accounts` 비밀번호 해시를 바꾸고 지금 세션을 뺀 세션을 지운다).

## 결정

1. 비밀번호 확인 순서는 빈 칸 → 길이 → 확인 다름 → 같은 값 → 지금 비밀번호. 형식 문제는 비밀번호 검사(느린 해시) 전에 알려 준다.
2. 다른 기기 로그인은 세션 줄을 지워 끊는다(쿠키 캐시를 쓰지 않아 다음 요청부터 바로 끊긴다).
3. 내 방문은 세지 않는다(5단계 규칙 그대로).

## Project Structure

```text
src/lib/kst.ts (lastDays, fillDays), src/lib/auth-rules.ts (checkPasswordChange, 문구)
src/server/stats.ts (getBlogVisitDays), src/server/account.ts (changeMemberPassword)
src/components/settings/VisitChart.tsx, CategoryManager.tsx, SettingsForms.tsx (PasswordForm)
src/app/settings/blog/page.tsx, src/app/settings/account/{page,actions}.ts(x), src/app/layout.tsx
tests/unit/settings.test.ts, tests/e2e/account-settings.spec.ts
```
