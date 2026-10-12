# Implementation Plan: 연못 낚시터

**Branch**: `046-fishing` | **Date**: 2026-10-12 | **Spec**: [spec.md](./spec.md)

## Summary

`fishing_catches`(기본 키 (회원, 한국 날짜))와 원장 사유 `fishing`(ref = 날짜, 부분 고유 인덱스)을 더한다(`0022_fishing`). `castLine`은 서버가 `randomInt(100)`으로 고르고 [회원 행 KEY SHARE → `lockUser` → 기록 넣기(충돌이면 오늘 이미 낚음) → 코인 원장 또는 `growth_feed` 수량 +1]. 마을에는 `pondArt` 연못과 입구.

## Project Structure

```text
src/lib/fishing.ts, src/lib/game.ts, src/db/schema.ts, drizzle/0022_fishing.sql
src/server/fishing.ts, src/app/fishing/{page,actions}.ts(x), src/components/fishing/FishingView.tsx
src/lib/art/town.ts(pondArt), src/lib/town/{layout,structures}.ts, src/components/town/TownHud.tsx
tests/unit/{fishing,town}.test.ts, tests/e2e/{fishing,town-menu}.spec.ts
```
