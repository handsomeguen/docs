# Implementation Plan: 걷는 그림

**Branch**: `048-walk` | **Date**: 2026-10-12 | **Spec**: [spec.md](./spec.md)

## Summary

DB 변경 없음. `walkFrame(입은 그림들, 방향, 칸)`이 16×16 색 칸을 만들고, `TownGame`이 시작할 때 방향 4 × 5칸을 캔버스로 구워 둔다. 매 틀마다 `walkDir`로 방향을, 실제로 움직였는지로 걷기/서 있기를 정하고 `data-walk`를 적는다.

## Project Structure

```text
src/lib/art/walk.ts, src/components/town/TownGame.tsx
tests/unit/walk.test.ts, tests/e2e/town.spec.ts
```
