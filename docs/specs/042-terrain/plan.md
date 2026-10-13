# Implementation Plan: 높낮이 있는 마을

**Branch**: `042-terrain` | **Date**: 2026-10-12 | **Spec**: [spec.md](./spec.md)

## Summary

DB 변경 없음. `src/lib/town/terrain.ts`(절벽 줄·계단·걸을 수 없는 땅·땅 높이)를 더하고, `townLayout`이 `walls`를 함께 돌려준다. 게임은 `walls`를 몸체처럼 막고, 바닥 굽기(`ground.ts`)가 앞면·계단·하늘을 그린다.

## 결정

1. 앞면은 계단 자리를 뺀 조각 사각형(`TERRAIN_WALLS`)이라 기존 `step`(축마다 막기)을 그대로 쓴다. 계단 양쪽 6px은 난간.
2. 깊이는 지금처럼 그림 아랫변 y. 앞면은 바닥에 굽기 때문에 위 단 가장자리에 선 캐릭터는 앞면 위에, 아래 단 캐릭터는 앞면 앞에 그려진다.
3. 나무는 절벽 앞면 위아래 24px 안에는 두지 않는다.
4. 시험 훅: 캔버스 `data-stairs`(계단마다 위·아래 끝 좌표).

## Project Structure

```text
src/lib/town/{terrain,layout,structures}.ts, src/lib/art/town.ts(benchArt)
src/components/town/{ground.ts,TownGame.tsx}
tests/unit/town.test.ts, tests/e2e/town.spec.ts
```
