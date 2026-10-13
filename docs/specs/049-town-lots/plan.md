# Implementation Plan: 이웃 집 자리 고르기

**Branch**: `049-town-lots` | **Date**: 2026-10-12 | **Spec**: [spec.md](./spec.md)

## Summary

`follows.town_lot`(smallint, 마이그레이션 `0023_town_lots`)에 고른 자리를 적는다. `getMemberHouses`는 즐겨찾기 이웃을 `assignLots`로 자리에 놓는다(고른 자리 먼저, 나머지는 남은 자리를 작은 번호부터). `setTownLot`은 `fav:` 잠금 안에서 지금 배치를 구해 `moveToLot`으로 맞바꾸고, 바뀌는 행을 먼저 비운 뒤 새 자리를 적는다(부분 유일 인덱스). `townLayout(…, chooseLots)`가 내 마을의 빈 집터에 `lots:{n}` 입구를 만들고 `TownGame`이 `bv-lots` 이벤트로 `TownHud`의 `LotsPanel`을 연다.

## Project Structure

```text
drizzle/0023_town_lots.sql, src/db/schema.ts
src/lib/town/lots.ts, src/lib/town/structures.ts
src/server/follows.ts (setTownLot, toggleFavorite), src/server/town.ts (getMemberHouses)
src/app/town/actions.ts, src/app/town/page.tsx
src/components/town/LotsPanel.tsx, TownHud.tsx, TownGame.tsx
tests/unit/lots.test.ts, tests/e2e/town-lots.spec.ts
```
