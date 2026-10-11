# Feature Specification: 공감 (♥)

**Feature Branch**: `007-likes`

**Created**: 2026-10-11

**Status**: Implemented

**Input**: User description: "계속 진행해줘" (006 다음 순서)

**다루는 명세 요구사항**: SOC-03 (공감), POST-05 (글 카드 `♥ N`)

**이번에 하지 않는 것**: 공감 보상(GAME-05)과 공감 알림(GAME-08) — 게임·알림이 아직 없다. 공감한 사람 목록.

## User Scenarios & Testing *(mandatory)*

### User Story 1 - 글에 공감하고 취소한다 (Priority: P1) 🎯

회원이 글 상세 본문·태그 아래 가운데의 [♡ 공감 N]을 누르면 [♥ 공감 N+1](빨간 테두리)이 되고, 다시 누르면 취소된다.

**Acceptance Scenarios**:

1. **Given** 회원, **When** [♡ 공감 0], **Then** 바로 `♥ 공감 1`·`aria-pressed="true"`, 처리 중에는 버튼이 비활성. 새로고침해도 그대로.
2. **Given** 다시 누름, **Then** `♡ 공감 0`. 공감 → 취소 → 공감을 해도 행은 1개.
3. **Given** 다른 탭에서 이미 공감, **When** 예전 화면의 [♡]를 누름, **Then** 서버가 지금 상태대로 취소한다 (잠깐 ♥ → ♡).
4. **Given** 내 글, **Then** 공감할 수 있고 수에 들어간다.
5. **Given** 방문자, **Then** 버튼이 비활성이고 바로 아래 `로그인하면 공감할 수 있어요` (`aria-describedby`).
6. **Given** 남의 비공개 글·`abc`·범위 밖 ID로 조작, **Then** 저장 안 됨, HTTP 200, 문구 없음, 하트는 되돌아간다.
7. **Given** 같은 요청을 동시에 10번, **Then** 모두 200, 행은 1개를 넘지 않는다.
8. **Given** 글 카드, **Then** `♥ N`(0이어도 보임)이 상세의 수와 같다.

### Edge Cases

- 글이나 회원이 지워지면 공감도 지워진다 (`ON DELETE CASCADE`).
- 로그인이 풀린 뒤 누르면 `/`로 간다.

## Requirements *(mandatory)*

- **FR-001**: `post_likes` 기본 키 `(post_id, user_id)`로 한 글에 한 번.
- **FR-002**: 공감인지 취소인지는 보내지 않는다. 서버가 트랜잭션 안에서 내 공감을 지우고, 지운 게 없으면 넣는다(`ON CONFLICT DO NOTHING`).
- **FR-003**: 글 행을 `FOR KEY SHARE`로 잠그고, 공개 글이거나 내 글일 때만 (댓글과 같은 `lockVisiblePost`).

## Success Criteria *(mandatory)*

- **SC-001**: 수용 시나리오가 모두 자동 테스트로 통과한다.
