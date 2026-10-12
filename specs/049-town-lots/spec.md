# Feature Specification: 이웃 집 자리 고르기

**Feature Branch**: `049-town-lots`

**Created**: 2026-10-12

**Status**: Implemented

**Input**: User description: "응 나머지 끝날때까지 쭉 진행해줘" (048 다음 마을 순서)

**다루는 명세 요구사항**: TOWN-18 이웃 집 자리 고르기(☰ `🏡 이웃 집 자리`, 내 마을의 빈 집터 Space, 맞바꾸기, ⭐를 끄면 자리 비우기, DB CHECK·부분 유일 인덱스).

**명세와 다른 점**:
- 마이그레이션 번호는 0032가 아니라 `0023_town_lots`. 파일 위치는 `src/lib/town/lots.ts`(`assignLots`·`moveToLot`), `src/server/follows.ts`(`setTownLot`), `src/components/town/LotsPanel.tsx`, 액션 `src/app/town/actions.ts`(`chooseTownLot`).
- 서버 잠금은 `lockUser`가 아니라 ⭐ 켜기·끄기와 같은 회원별 잠금(`fav:{회원}`)이다. 같은 줄에 서야 ⭐를 끄는 것과 자리 고르기가 겹치지 않는다.
- 한 번 고르면 다른 이웃도 **지금 선 자리를 적어 둔다**. 명세대로 안 고른 이웃이 남은 자리를 차례로 받으면, 1번 이웃을 다른 자리로 옮길 때 2·3번 이웃이 한 칸씩 당겨져 집이 뜻밖에 움직이기 때문이다. 새로 ⭐한 이웃은 명세대로 남은 자리를 작은 번호부터 받는다.
- 자리가 있는 곳은 지금 마을 배치대로 `서쪽 길`(1~4)·`동쪽 길`(5~8)·`남쪽 들판`(9·10)이다 (언덕 마을·들판 마을 배치는 아직 없다).
- ✨는 빈 집터에도 있다(그 앞에서 Space면 고르기 창). 휴대폰 ☰ 메뉴 화면에는 마을이 없어 이 창이 없다.

## User Scenarios & Testing *(mandatory)*

### User Story 1 - 자리 고르기 (Priority: P1) 🎯

1. 내 마을 ☰ → `🏡 이웃 집 자리`: 1~10번 자리마다 `n번 · 🏠 {닉네임}의 집`(없으면 `빈 집터`), 있는 곳, [✨]·[고르기].
2. [고르기] → 즐겨찾기 이웃 목록. 이 자리에 사는 이웃은 `지금 이 자리에 살아요`, 다른 이웃은 `지금 n번 자리 → {닉네임}님과 자리 바꾸기`(빈 자리면 `이 자리로 옮기기`). 고르면 자리 목록으로 돌아오고 마을의 집도 옮겨 선다.
3. 내 마을의 빈 집터(1~10번) 앞 `🏡 빈 집터 n번 · Space 이웃 자리 고르기` → 그 자리 고르기 창.
4. 친구 마을(`/town/{주소}`)은 그 회원이 고른 자리대로 보이고, 그곳에는 이 창과 빈 집터 입구가 없다.

### Edge Cases

- 1~10이 아닌 자리 → `이웃 집 자리는 1~10번이에요.` 즐겨찾기 아닌 이웃·공지 블로그 → `⭐ 즐겨찾기한 이웃만 집 자리를 고를 수 있어요.` 아무것도 바뀌지 않는다.
- ⭐를 끄면 `town_lot`도 비운다. DB가 CHECK(`town_lot` 1~10이고 즐겨찾기일 때만)와 회원마다 자리 하나에 한 이웃(`follows_town_lot_uq`)을 지킨다.

## Success Criteria *(mandatory)*

- **SC-001**: `tests/unit/lots.test.ts`, `tests/e2e/town-lots.spec.ts`가 통과한다.
