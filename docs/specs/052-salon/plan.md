# Implementation Plan: 미용실

**Branch**: `060-salon` | **Spec**: [spec.md](spec.md)

- 데이터: 마이그레이션 `0025_salon` — `items_type_check`·`avatar_equips_slot_check`에 `hair`·`hair_color`, 머리 모양 7·머리 색 8 넣기.
- 그림: `src/lib/art/hair.ts`(`HAIR_ART` 덧그림 7, `HAIR_COLORS` 8, `bald`, `recolorHair`). `findArt`가 `~bald`·`~#rrggbb` 꾸밈을 읽는다. `avatarLayers`가 몸 → 옷 → 머리 → 소품 → 모자.
- 계산·처리: `src/lib/salon.ts` `styleBill`, `src/server/salon.ts` `applyStyle`(옷가게 `applyOutfit`과 같은 트랜잭션 순서).
- 화면: `src/app/salon/page.tsx`, `src/components/salon/SalonStudio.tsx`, 마을 ☰ `💇 미용실`. 상점은 머리 종류를 뺀다.
