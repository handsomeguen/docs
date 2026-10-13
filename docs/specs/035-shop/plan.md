# Implementation Plan: 마을 상점

**Branch**: `035-shop` | **Date**: 2026-10-12 | **Spec**: [spec.md](./spec.md)

## Summary

마이그레이션 `0016_shop`: `items`에 `description`·`required_level`(1~99), 종류 CHECK(`character`·`hat`·`clothes`·`accessory`·`furniture`·`background`), 파는 아이템 CHECK(값 > 0, 기본 아님, 캐릭터 아님), 원장 사유 `purchase`와 CHECK(구매만 코인 음수·경험치 0), 부분 고유 인덱스 `point_ledger_purchase_uq`, 판매 아이템 20개.

## 결정

1. 버튼 상태·정렬·안내 문구는 `src/lib/shop.ts` 순수 함수(단위 시험). 서버 `buyItem`도 같은 `buttonState`로 판단한다.
2. `buyItem`(`src/server/shop.ts`): 탈퇴와 같은 순서로 회원 행을 먼저 잡고 `lockUser` → `getWallet(userId, tx)` → INSERT → `addLedgerEntry`(경험치 0이라 레벨업 없음).
3. 그림: `src/lib/art/items.ts`(아이템·마네킹), `findArt`(`src/lib/art/index.ts`), `PixelLayers`(여러 장 겹쳐 그리기).
4. `getPendingLevelUp`은 `{ level, items, more }`. 남은 수는 `count(*) over ()`.

## Project Structure

```text
drizzle/0016_shop.sql, src/db/schema.ts
src/lib/{shop,game,notifications}.ts, src/lib/art/{items,index}.ts
src/server/{shop,points,notifications}.ts
src/app/shop/{page,actions}.ts(x), src/components/shop/ShopView.tsx, src/components/game/{PixelLayers,LevelUpDialog}.tsx
src/app/layout.tsx(🏪 상점), src/app/wallet/page.tsx(상점 링크), src/app/notifications/actions.ts(go = shop)
tests/unit/{shop,pixel-art}.test.ts, tests/e2e/{shop,level-up}.spec.ts
```
