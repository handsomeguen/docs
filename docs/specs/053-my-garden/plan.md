# Implementation Plan: 내 정원

**Branch**: `053-my-garden` | **Date**: 2026-10-13 | **Spec**: [spec.md](./spec.md)

## Summary

DB 변경 없음. 마을을 2400px로 넓히고 동쪽에 타원 정원(`GARDEN`)을 둔다. 둘레는 세로 조각 벽(돌 테 + 축대 앞면)으로 막고 가운데 정원 계단(`GARDEN_STAIR`, `STAIRS` 다섯째)만 연다. 내 집(0번)은 (1870, 730), 농장은 (2150, 740), 둘 다 정원 안 가로 돌길(y 770)에 입구가 있다. 정원 가는 흙길은 동쪽 길 (1510, 1050) 갈림길에서 계단 아래까지. 장식은 아치 문(기둥 둘은 벽)·울타리 10칸·벚나무 둘 구조물과 바닥 굽기(잔디 줄무늬·덤불·돌길·꽃밭·돌 테·축대). 이웃 집 5~8번을 옮기고, 이웃 길 그물은 공개 흙길만 쓰며 정원 둘레 60px 안을 빼고, 나무는 정원 근처에 심지 않는다. 시험 훅 `data-garden`.

## Project Structure

```text
src/lib/town/layout.ts (WORLD, GARDEN·inGarden·gardenEdge, HOUSE_LOTS, FARM_POS, ROADS·GARDEN_APPROACH·GARDEN_PATHS·GARDEN_BEDS)
src/lib/town/terrain.ts (GARDEN_STAIR, GARDEN_WALLS, heightAt(y, x))
src/lib/town/structures.ts (아치 문·울타리·벚나무, 기둥 벽, 나무 거르기)
src/lib/town/npc.ts (GARDEN_CLEARANCE), src/lib/town/lots.ts (lotArea)
src/lib/art/town.ts (gardenArchArt, fenceArt, cherryTreeArt)
src/components/town/ground.ts (drawGarden), src/components/town/TownGame.tsx (data-garden)
tests/unit/garden.test.ts, tests/unit/town.test.ts, tests/e2e/town*.spec.ts·farm·mailbox·friend-town
```
