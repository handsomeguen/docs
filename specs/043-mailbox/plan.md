# Implementation Plan: 집 앞 우체통

**Branch**: `043-mailbox` | **Date**: 2026-10-12 | **Spec**: [spec.md](./spec.md)

## Summary

DB 변경 없음. `getMemberHouses`·`getGuestHouses`가 집마다 최근 공개 글 3개(`ROW_NUMBER() OVER (PARTITION BY blog_id …) <= 3`)를 붙인다. `townLayout`이 집마다 우체통 그림·입구(`href = mailbox:{자리}`)를 더하고, 게임은 그 입구에서 `bv-mailbox` 이벤트를 보내 `TownHud`가 우체통 창을 연다.

## 결정

1. 우체통 창은 ☰ 메뉴와 같은 `<dialog open>` 창(게임이 키를 놓는다). 내 우체통은 알림 창과 같은 목록(`notificationList`)을 함께 쓴다.
2. 내 우체통 깃발은 `/town`이 안 읽은 수로 정해 `TownGame mailFlag`로 넘긴다. 시험 훅 캔버스 `data-mail-flag`.
3. 게임에서 연 창을 닫으면 초점을 게임에 돌려준다(`fromGame`).

## Project Structure

```text
src/server/town.ts, src/lib/town/structures.ts, src/lib/art/town.ts(mailboxArt)
src/components/town/{TownGame,TownHud}.tsx, src/app/town/page.tsx
tests/unit/town.test.ts, tests/e2e/mailbox.spec.ts
```
