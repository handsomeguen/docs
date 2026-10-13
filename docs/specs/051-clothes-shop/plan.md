# Implementation Plan: 옷가게

**Branch**: `059-clothes` | **Spec**: [spec.md](spec.md)

- 데이터: 마이그레이션 `0024_clothes_shop`(아이템 9개 넣기만, 표 변경 없음). 상점에서는 `CLOTHES_SHOP_ONLY` 코드로 뺀다.
- 계산: `src/lib/clothes.ts` `outfitBill(outfit, current, catalog, level, coins)` — 화면 계산서와 서버가 같이 쓴다.
- 처리: `src/server/clothes.ts` `applyOutfit` — 회원 행 KEY SHARE → `lockUser` → 고른 것 확인 → 합계 비교 → 레벨·코인 → `user_items`·원장 → `avatar_equips` 부위별 저장/삭제. 액션 `src/app/clothes/actions.ts`(`useActionState`).
- 화면: `src/app/clothes/page.tsx`, `src/components/clothes/ClothesStudio.tsx`(거울은 `walkFrame`으로 그린 16×16 SVG를 1초 8칸으로 돌린다).
- 그림: `src/lib/art/items.ts`에 모자 3·옷 6.
