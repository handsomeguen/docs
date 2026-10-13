# Feature Specification: 성장 아이템

**Feature Branch**: `045-growth-items`

**Created**: 2026-10-12

**Status**: Implemented

**Input**: User description: "응 나머지 끝날때까지 쭉 진행해줘" (044 다음 마을 순서)

**다루는 명세 요구사항**: SHOP-01 🌱 성장 아이템 구역(동물 먹이 +20 · 고급 먹이 +50 · 성장 촉진제 +100, 여러 개 살 수 있음, `보유 N개`), TOWN-09 농장의 [🌱 {이름} ×{수량}] 버튼.

**명세와 다른 점**:
- 성장 아이템 구매는 원장 사유 `growth_purchase`(🌱 성장 아이템 구매)로 따로 적는다. `purchase`는 (회원, 아이템)마다 한 번(고유 인덱스)이라 같은 아이템을 또 사면 기록이 막히기 때문.
- 다 쓴 성장 아이템은 `user_items` 행을 지운다(수량은 1 이상, CHECK). 명세의 `0 이상` 대신.
- `consumeGrowthItem`(`src/server/inventory.ts`)을 따로 두지 않고 `feedGrowthItem`(`src/server/farm.ts`) 안에서 수량을 줄인다.
- 농장에 성장 아이템이 하나도 없으면 상점 안내 한 줄과 [🏪 상점 가기] 링크.

## User Scenarios & Testing *(mandatory)*

### User Story 1 - 사서 먹인다 (Priority: P1) 🎯

1. 상점 맨 아래 `🌱 성장 아이템` 구역, 안내 `동물 농장의 친구들을 빨리 키워 주는 먹이예요. 여러 번 살 수 있어요.`
2. 가지고 있어도 [사기]가 그대로이고 카드에 `보유 N개`. 산 뒤 `🎉 … 동물 농장에서 써 보세요.`와 [🐮 동물 농장으로].
3. 농장의 키우는 동물 카드마다 [🌱 {이름} ×{수량}]. 누르면 하나를 쓰고 그만큼 자란다(`🌱 {이름} 먹였어요! 성장 +N`, 다 자라면 다 자란 문구). 하루 제한 없음.
4. 남의 동물·다 자란 동물: `먹일 수 있는 동물이 아니에요`, 없는 아이템: `가지고 있는 성장 아이템이 없어요`.

## Success Criteria *(mandatory)*

- **SC-001**: `tests/unit/{shop,farm}.test.ts`, `tests/e2e/{shop,farm}.spec.ts`가 통과한다.
