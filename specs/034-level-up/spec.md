# Feature Specification: 레벨업 알림

**Feature Branch**: `034-level-up`

**Created**: 2026-10-12

**Status**: Implemented

**Input**: User description: "계속 진행해줘" (033 다음 게임 순서)

**다루는 명세 요구사항**: GAME-06 레벨업 알림(오른 레벨마다 알림 1줄, 헤더 가운데 팝업, 읽음 처리), GAME-08 알림함의 레벨업 줄, ERD `notifications`의 `level_up`.

**명세와 다른 점**:
- 상점(SHOP)이 아직 없어 팝업 버튼은 [상점 가기] 대신 **[내역 보기]**(`/wallet`, 경험치·코인 내역)와 [확인]이다. 읽음 요청의 `go`도 `shop` 대신 `wallet`/`stay`. 상점이 생기면 [상점 가기]로 바꾼다.
- 새로 살 수 있게 된 아이템 줄(`이제 이런 친구를 데려올 수 있어요`)은 판매 아이템이 없어 아직 없다. 그 자리에 `활동하면서 경험치를 모았어요.` 한 줄.
- 팝업은 레이아웃 헤더의 `LevelUpDialog`(`src/components/game/LevelUpDialog.tsx`) 하나다(`LevelUpPopup` 서버 컴포넌트 없음). 헤더는 화면을 옮겨도 다시 그려지지 않으므로 🔔 숫자처럼 주소가 바뀔 때마다 `/api/notifications/unread`에서 안 읽은 가장 높은 레벨을 새로 받는다.
- 알림함의 레벨업 줄을 누르면 경험치·코인 내역(`/wallet`)으로 간다.
- 이번 단계 전에 이미 레벨이 오른 회원에게는 지난 레벨업 알림을 만들지 않는다(마이그레이션이 한꺼번에 팝업을 띄우지 않게).

## User Scenarios & Testing *(mandatory)*

### User Story 1 - 레벨업 알림 기록 (Priority: P1) 🎯

1. 경험치가 있는 원장 기록은 모두 `addLedgerEntry`(`src/server/points.ts`)로 넣는다(보상 `grantReward`, 출석). 넣기 전후 누적 경험치로 `levelsGained`를 구해 오른 레벨마다 `notifications`(`kind = 'level_up'`, `level = N`)를 같은 트랜잭션에 넣는다.
2. 같은 레벨은 한 줄만(부분 고유 인덱스 `notifications_level_up_uq`), 레벨은 2~99(CHECK). 레벨업 알림에는 행동한 사람·글이 없다(`notifications_shape_check`).
3. 레벨이 오르지 않은 보상, 경험치 없는 기록(가입 축하)은 알림이 없다.

### User Story 2 - 레벨업 팝업 (Priority: P1)

1. 안 읽은 레벨업이 있으면 헤더가 화면 가운데 `<dialog>`를 띄운다: 🎉, `Lv.N이/가 되었어요!`(받침, Lv.99는 `최고 레벨 Lv.99가 되었어요!`), [내역 보기]·[확인](처음 초점 [확인]).
2. 내 활동으로 오르면 그 화면에서 바로, 남의 공감으로 오르면 다음에 여는 화면에서 뜬다.
3. 여러 개 쌓이면 가장 높은 레벨 하나만. [확인]·[내역 보기]·Esc는 바로 닫고 그 레벨 이하를 모두 읽음으로 한다. 레벨 0·100·글자로 조작한 요청은 아무것도 바꾸지 않는다.
4. 알림함에 `🎉 Lv.N이 되었어요!`로 남는다. 🔔 숫자에도 센다.

## Success Criteria *(mandatory)*

- **SC-001**: `tests/unit/josa.test.ts`, `tests/unit/game.test.ts`(오른 레벨), `tests/e2e/level-up.spec.ts`가 통과하고 기존 시험이 그대로 통과한다(다른 시험에서는 `tests/e2e/fixtures.ts`가 팝업을 [확인]으로 닫는다).
