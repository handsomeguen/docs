# Implementation Plan: 휴대폰 아래 탭

**Branch**: `040-phone-tabs` | **Date**: 2026-10-12 | **Spec**: [spec.md](./spec.md)

## Summary

DB 변경 없음. 레이아웃이 회원이면 `PhoneTabs`(클라이언트, `usePathname`)를 헤더 옆에 그리고 CSS로 640px 이상에서 숨긴다. `/town?menu=1`은 휴대폰 칸에 `TownHud phone`을 그린다.

## 결정

1. 탭의 안 읽은 수는 헤더 🔔의 로직을 `useUnreadCount`로 떼어 함께 쓴다.
2. 휴대폰 회원 헤더에서 탭·메뉴와 겹치는 링크는 `contents max-sm:hidden` 칸으로 숨긴다(`.btn`은 유틸리티보다 강함).
3. `body:has([data-phone-tabs])`에 639px 이하에서 `padding-bottom: 3.5rem + safe-area`.
4. `TownHud`에 `phone`: 늘 펼친 `<section>`(모달·`dialog` 아님, 게임 키 처리와 무관), 로고·☰ 단추·✕·Esc·텔레포트 없음. 초점은 창이 바뀔 때만 옮긴다(StrictMode 두 번 실행에도 처음에는 안 옮김).
5. 개발 서버의 Next.js 도구 단추가 왼쪽 아래(첫 탭 위)에 떠 있어 시험은 첫 탭을 키보드로 누른다(운영에는 없음).

## Project Structure

```text
src/components/layout/PhoneTabs.tsx, src/components/notifications/NotificationBell.tsx
src/app/layout.tsx, src/app/globals.css, src/app/town/page.tsx, src/components/town/TownHud.tsx, src/app/blog/[slug]/page.tsx
tests/e2e/phone-tabs.spec.ts
```
