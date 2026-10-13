# Implementation Plan: 성장 아이템

**Branch**: `045-growth-items` | **Date**: 2026-10-12 | **Spec**: [spec.md](./spec.md)

## Summary

`items.type`에 `growth`, `items.growth_value`(성장 아이템이면 있고 0보다 큼), `user_items.quantity`(1 이상), 원장 사유 `growth_purchase`를 더하고 아이템 3개를 넣는다(`0021_growth_items`). 상점은 성장 아이템이면 보유와 관계없이 사고 개수를 +1, 농장은 `feedGrowthItem`이 [잠금 → 키우는 내 동물 → 아이템 행 `FOR UPDATE` → 수량 −1(마지막이면 지움) → `addGrowth`].

## Project Structure

```text
src/db/schema.ts, drizzle/0021_growth_items.sql, src/lib/{shop,farm,game}.ts, src/lib/art/animals.ts
src/server/{shop,farm}.ts, src/app/farm/actions.ts, src/components/{shop/ShopView,farm/FarmView}.tsx
tests/unit/{shop,farm}.test.ts, tests/e2e/{shop,farm}.spec.ts
```
