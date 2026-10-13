# Feature Specification: 걸어 다니는 이웃

**Feature Branch**: `050-town-npc`

**Created**: 2026-10-12

**Status**: Implemented

**Input**: User description: "응 나머지 끝날때까지 쭉 진행해줘" (049 다음 마을 순서)

**다루는 명세 요구사항**: TOWN-19 걸어 다니는 이웃(집이 선 이웃이 입은 모습 그대로 걷기 그림으로 흙길을 걷고, 갈림길에서 잠깐 쉬고, 내 집 앞 190px 안에는 오지 않으며, 가까이 가거나 마우스를 올리면 이름표, 누르면 그 집 우체통).

**명세와 다른 점**:
- 파일은 `scene.ts`가 아니라 `src/lib/town/npc.ts`(`npcGraph`·`segmentClear`·`createNpcs`·`updateNpcs`)이고 그리기는 `TownGame`에 있다.
- 내 정원(TOWN-20)이 아직 없어 "정원 길을 걷지 않는다"는 내 집(0번) 앞 190px 규칙만으로 지킨다. 그래서 내 집으로 가는 길 아래쪽과 농장 앞길에는 오지 않는다.
- 분수 옆을 지나는 가운데 세로 길처럼 몸체에 막히는 줄은 빠져, 이웃은 막히지 않은 길만 다닌다. 이웃과 내 캐릭터는 서로 막지 않는다(지나쳐 걷는다).
- 시험 훅은 `data-npcs`·`data-npc-walking`에 더해 화면에 보이는 이웃의 자리 번호와 몸 가운데 `data-npc-at`(`1:382,228 2:…`).
- 캐릭터가 없는 이웃은 방문자와 같은 회색 주민 그림으로 걷는다.

## User Scenarios & Testing *(mandatory)*

### User Story 1 - 마을을 걷는 이웃 (Priority: P1) 🎯

1. 마을에 집이 선 이웃(1~10번, 방문자 마을은 그 10채의 주인)이 자기 집 앞 길에서 출발해 흙길을 따라 걷는다(초당 70px, 걷기 그림 TOWN-17). 갈림길·막다른 길에서는 0.6~1.8초 쉬고, 온 길로 바로 돌아가지 않는다.
2. 내 캐릭터가 140px 안에 오거나 마우스를 올리면 이름표가 보이고, 누르면 그 이웃 집 우체통 창(새 글·집에 들어가기)이 열린다.

## Success Criteria *(mandatory)*

- **SC-001**: `tests/unit/npc.test.ts`(막히지 않는 길, 내 집 앞 190px, 집마다 가까운 길 점, 30초 걸어도 길 위), `tests/e2e/town-npc.spec.ts`가 통과한다.
