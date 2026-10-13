# Implementation Plan: 우리 집

**Branch**: `037-house` | **Date**: 2026-10-12 | **Spec**: [spec.md](./spec.md)

## Summary

마이그레이션 `0018_house`: `house_furniture(user_id, slot, item_id)` 기본 키 `(user_id, slot)`, 고유 `(user_id, item_id)`, `slot` CHECK 0~7, `user_items` 복합 외래 키 ON DELETE CASCADE. 기본 가구 `furn_plant`·`furn_chair`(starter)를 모든 회원에게 주고 0·1번 칸에 놓는다.

## 결정

1. `src/lib/house.ts`: `houseStage`·`furnitureSlots`·`houseInfo`·`houseSummary`, 문구 `HOUSE_ERRORS`, `parseSlot`("0"~"7"만).
2. `placeFurniture`(`src/server/house.ts`): 회원 행 KEY SHARE → `lockUser`(구매·보상과 같은 순서) → 레벨로 열린 칸 수 → 칸 확인 → `null`이면 비우기 → 가진 가구인지 → 다른 칸의 같은 가구를 지우고 그 칸에 upsert. 레벨을 잠근 채 읽어 칸 수가 엇갈리지 않는다.
3. `HouseRoom`(클라이언트): 서버 결과로 칸을 다시 받지 않고, 성공하면 화면 상태를 바꾸고 실패하면 alert 문구만 보인다. 가입·시드·관리자 만들기도 기본 가구를 준다.

## Project Structure

```text
drizzle/0018_house.sql, src/db/schema.ts
src/lib/{house.ts,shop.ts,art/items.ts}, src/server/{house,signup}.ts, scripts/{seed,create-admin}.ts
src/app/house/actions.ts, src/components/house/HouseRoom.tsx
src/app/blog/[slug]/page.tsx, src/app/shop/page.tsx, src/components/shop/ShopView.tsx, src/app/layout.tsx
tests/unit/{house,shop}.test.ts, tests/e2e/{house,shop,characters}.spec.ts
```
