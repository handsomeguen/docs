# Implementation Plan: 내 정원

**Branch**: `062-garden` | **Spec**: [spec.md](spec.md)

- 배치: `src/lib/town/layout.ts` `GARDEN`(760, 1050, 680×280)·`GARDEN_GATE`·`inGarden`, `FARM_POS` 옮김, `CLOTHES_POS`·`SALON_POS`와 앞 흙길.
- 놓을 것: `src/lib/town/structures.ts` `gardenStructures(name)`(울타리 40px 칸·옆 말뚝 20px마다, 아치, 꽃밭, 벚나무), 옷가게·미용실 건물과 입구, 텔레포트 7곳. 숲 나무 비우기.
- 이웃 길: `src/lib/town/npc.ts` `GARDEN_CLEARANCE`, 정원을 지나는 줄은 뺀다.
- 바닥: `src/components/town/ground.ts` 정원 잔디 줄무늬.
- 그림: `src/lib/art/town.ts` `fenceArt`·`fencePostArt`·`rosesArchArt`·`flowerBedArt`·`cherryTreeArt`·`clothesShopArt`·`salonArt`.
- DB 변경 없음.
