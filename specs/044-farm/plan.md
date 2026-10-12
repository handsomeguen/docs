# Implementation Plan: 동물 농장

**Branch**: `044-farm` | **Date**: 2026-10-12 | **Spec**: [spec.md](./spec.md)

## Summary

`animals`(알·동물, 첫 알·레벨 알은 부분 고유 인덱스로 한 번)와 `animal_cares`((동물, 돌보기, 한국 날짜) 기본 키) 테이블을 더한다(`0020_farm`). 원장에 `farm_care`(보상 규칙, 하루 15번)·`farm_grown`(동물마다 한 번)·`egg_purchase`(알마다 한 번, 코인 −100)를 더한다. `/farm` 화면의 버튼은 모두 `farmAction` 하나를 `intent`로 부른다.

## 결정

1. 모든 처리는 [회원 행 KEY SHARE → `lockUser`] 순서로 잠근 트랜잭션 안. 5마리 상한·코인·레벨을 그 안에서 다시 센다.
2. `addGrowth`가 성장치를 종류의 다 자라는 값까지만 더하고, 다 차면 `grown_at`과 `farm_grown` 원장을 같이 넣는다.
3. 글쓰기 연동은 `savePost`에서 `grantReward(post)`가 true일 때만 `growForPost`(같은 트랜잭션).
4. 마을 농장은 `FARM_POS`(1260, 1080), 입구는 그 아래 30px, 게시판~내 집 길에서 흙길이 이어진다.

## Project Structure

```text
src/lib/farm.ts, src/lib/art/animals.ts, src/lib/art/town.ts(farmArt), src/lib/game.ts
src/db/schema.ts, drizzle/0020_farm.sql
src/server/farm.ts, src/server/post.ts(growForPost)
src/app/farm/{page,actions}.ts(x), src/components/farm/FarmView.tsx
src/lib/town/{layout,structures}.ts, src/components/town/TownHud.tsx
tests/unit/{farm,game,town}.test.ts, tests/e2e/{farm,town-menu}.spec.ts
```
