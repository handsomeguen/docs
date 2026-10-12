# Feature Specification: 글쓰기 글자 수

**Feature Branch**: `028-char-count`

**Created**: 2026-10-12

**Status**: Implemented

**Input**: User description: "계속 진행해줘" (027 다음 순서, `plans/gap-survey-2026-10-12.md` 15번)

**다루는 명세 요구사항**: POST-01 아래 상자 왼쪽 첫 줄 `N자`(천 단위 쉼표), 수정 중이면 `· 글을 고치고 있어요`. N은 서버가 저장하는 `content_text` 길이와 같다(#18).

**명세와 다른 점**:
- 게임(GAME-05)이 없어 보상 안내(`저장하면 ✨ 경험치 30 …`, `비공개 글은 보상이 없어요`, `100자 이상 쓰면 …`)는 없다.
- 세는 규칙은 이 코드의 서버 `htmlToText`를 따른다. 명세 본문과 다른 곳: `Shift+Enter` 줄바꿈도 1자, 이어진 띄어쓰기·빈 문단의 줄바꿈은 합치지 않는다(줄 끝 공백과 앞뒤 공백만 뺀다). 서버 규칙을 바꾸면 저장된 글의 요약·검색이 바뀌므로 그대로 둔다.

## User Scenarios & Testing *(mandatory)*

### User Story 1 - 쓰는 동안 글자 수 보기 (Priority: P1) 🎯

1. **Given** 글쓰기, **Then** 아래 줄 왼쪽에 `0자`. 쓰는 대로 바뀐다. 문단 사이 줄바꿈 1자, 이모지 `😀`는 2자(JS `length`), 사진·파일 카드는 0자, `1,234자`처럼 쉼표.
2. **Given** 발행, **Then** 저장된 `content_text`의 길이(JS 기준)가 화면 숫자와 같다.
3. **Given** 글 고치기, **Then** `N자 · 글을 고치고 있어요`.

## Success Criteria *(mandatory)*

- **SC-001**: `tests/unit/text-length.test.ts`(에디터 모양 HTML 24가지에서 화면 셈 = 서버 셈), `tests/e2e/char-count.spec.ts`가 통과한다.
