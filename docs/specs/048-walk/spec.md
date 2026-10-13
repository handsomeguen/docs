# Feature Specification: 걷는 그림

**Feature Branch**: `048-walk`

**Created**: 2026-10-12

**Status**: Implemented

**Input**: User description: "응 나머지 끝날때까지 쭉 진행해줘" (047 다음 마을 순서)

**다루는 명세 요구사항**: TOWN-17 걷는 그림(방향 4개 × 서 있기 + 걷기 4칸, 1초에 8칸, 멈추면 마지막 방향으로 서 있기, 막혀 제자리면 걷지 않음).

**명세와 다른 점**:
- 한 칸은 24×24가 아니라 캐릭터와 같은 16×16 도트이고, 그림판을 따로 그리지 않고 입은 모습(캐릭터 + 아이템)을 한 장으로 겹친 뒤 코드로 만든다(`src/lib/art/walk.ts`). 옆모습은 얼굴 줄을 한 칸 앞으로 돌리고, 뒷모습은 얼굴 줄을 머리색으로 덮는다. 팔 흔들기는 없다.
- Phaser 대신 캔버스에 직접 그린다(038부터 같음). 시험 훅은 명세대로 캔버스 `data-walk`(`walk-left`, `idle-down` 등).
- 예전의 위아래 출렁임(bob)은 걷기 칸의 몸 올리기로 바뀌었다.

## User Scenarios & Testing *(mandatory)*

### User Story 1 - 걷는 모습 (Priority: P1) 🎯

1. 움직이면 그 방향(대각선은 더 크게 움직이는 쪽)의 걷기 4칸을 1초에 8칸으로 돌린다. 1·3칸은 몸이 한 칸 올라가고 한쪽 발을 든다.
2. 멈추면 마지막 방향으로 서 있는 그림. 벽·분수에 막혀 제자리면 서 있는 그림.

## Success Criteria *(mandatory)*

- **SC-001**: `tests/unit/walk.test.ts`(칸 크기·반전·발 들기·옆/뒷모습), `tests/e2e/town.spec.ts`(걷는 그림)가 통과한다.
