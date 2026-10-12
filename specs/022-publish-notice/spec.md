# Feature Specification: 새 글 발행 안내

**Feature Branch**: `022-publish-notice`

**Created**: 2026-10-12

**Status**: Implemented

**Input**: User description: "계속 진행해줘" (021 다음 순서, `plans/gap-survey-2026-10-12.md` 9번)

**다루는 명세 요구사항**: POST-01 기본 흐름 5(발행하면 `/@주소/글ID?new=1`로 가서 맨 위에 안내), 발행 안내는 글 주인에게 발행 직후 한 번만, 노란 카드 `role="status"`, 수정은 안내 없음.

**명세와 다른 점**: 게임(경험치·코인)이 없어 보상 문구 없이 `🎉 글을 발행했어요!`만 보인다. 임시 글 지우기는 명세처럼 안내가 지우지 않고 지금처럼 글쓰기 화면을 떠날 때 지운다(009 결정 그대로).

## User Scenarios & Testing *(mandatory)*

### User Story 1 - 발행 직후 한 번 보이는 안내 (Priority: P1) 🎯

**Acceptance Scenarios**:

1. **Given** 새 글을 발행, **Then** `/@주소/글ID?new=1`로 가서 글 위에 노란 `role="status"` 카드 `🎉 글을 발행했어요!`가 보이고, 주소의 `?new`는 지워진다.
2. **Given** 새로고침하거나 다른 화면에 갔다가 뒤로 오기, **Then** 안내가 없다.
3. **Given** 글을 고쳐 [수정 완료], **Then** 안내 없이 `/@주소/글ID`.

### User Story 2 - 남에게는 보이지 않는다 (Priority: P1)

1. **Given** 다른 회원이나 방문자가 `?new=1` 주소를 연다, **Then** 안내가 없다.
2. **Given** 글을 만든 지 10분이 지났다, **Then** 주인이 `?new=1`을 열어도 안내가 없다. 10분 안이면 다시 보인다(명세 열린 질문 그대로).

## Success Criteria *(mandatory)*

- **SC-001**: 수용 시나리오가 자동 테스트(`tests/e2e/publish-notice.spec.ts`, 10분 규칙은 `tests/unit/post-rules.test.ts`)로 통과한다.
