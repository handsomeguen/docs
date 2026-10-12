# Implementation Plan: 걸어 다니는 이웃

**Branch**: `050-town-npc` | **Date**: 2026-10-12 | **Spec**: [spec.md](./spec.md)

## Summary

DB 변경 없음. 마을 집 목록에 주인의 입은 모습(`layers`, `avatarLayersOf`로 한 번에)을 붙인다. `npcGraph`는 흙길 띠(`ROADS`) 가운데선을 끝·교차점·집 앞에서 끊어 이웃한 점끼리 잇고, 6px마다 발 상자를 몸체·절벽과 비교해 막히는 줄과 내 집 앞 190px 안을 지나는 줄을 뺀다. `TownGame`은 이웃마다 걷기 그림판을 구워 두고 매 틀 `updateNpcs`로 옮기며 깊이 순서에 섞어 그린다.

## Project Structure

```text
src/lib/town/npc.ts, src/lib/town/structures.ts (TownHouseInput.layers)
src/server/items.ts (avatarLayersOf), src/server/town.ts
src/components/town/TownGame.tsx
tests/unit/npc.test.ts, tests/e2e/town-npc.spec.ts
```
