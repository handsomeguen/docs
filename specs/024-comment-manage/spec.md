# Feature Specification: 댓글 관리

**Feature Branch**: `024-comment-manage`

**Created**: 2026-10-12

**Status**: Implemented

**Input**: User description: "계속 진행해줘" (023 다음 순서, `plans/gap-survey-2026-10-12.md` 11번)

**다루는 명세 요구사항**: SOC-01 열린 질문(제안) "블로그 관리에 [댓글]을 두어 내 글에 달린 댓글을 최신순으로 모아 보고 바로 지우게?", SOC-01 표시 규칙 "탈퇴한 회원의 댓글 자리: 캐릭터·닉네임·시각 없이 `삭제된 댓글이에요`만"(AUTH-06).

**명세와 다른 점**: 명세의 블로그 관리는 한 화면의 칸(`방문자`·`기본 정보`·`카테고리`)이지만, 댓글은 글 관리(`/settings/posts`)처럼 따로 쪽을 넘기는 화면 `/settings/comments`로 둔다. 블로그 관리 위쪽 링크 줄과 블로그 홈 주인 도구 막대에서 들어간다.

## User Scenarios & Testing *(mandatory)*

### User Story 1 - 내 글에 달린 댓글 모아 보고 지우기 (Priority: P1) 🎯

**Acceptance Scenarios**:

1. **Given** 블로그 관리의 `💬 댓글 관리`나 주인 도구 막대의 `💬 댓글 관리`, **Then** `/settings/comments`에 `💬 댓글 관리`와 `내 글에 달린 댓글 N개`.
2. **Given** 내 글(비공개 포함)에 남과 내가 단 댓글·답글, **Then** 지우지 않은 것만 최신순으로 한 줄씩: 답글이면 `↳ 답글`, 작성자 닉네임(그 블로그 링크) · 시각, 내용 한 줄(길면 `…`), `📄 글 제목`(그 글의 그 댓글 자리 `#comment-ID`·`#reply-ID`), [삭제]. 남의 블로그에 단 내 댓글은 없다.
3. **Given** [삭제], **Then** 확인 창 `댓글을 삭제할까요?`. 취소하면 그대로, 확인하면 목록에서 빠지고 글 상세의 그 자리는 `삭제된 댓글이에요`(달린 답글은 그대로).
4. **Given** 20개가 넘는다, **Then** 페이지 번호로 다음 쪽. 이상한 `?page`는 1쪽.
5. **Given** 댓글이 없다, **Then** `아직 달린 댓글이 없어요`. 방문자는 첫 화면(`/`)으로.

### User Story 2 - 탈퇴한 회원의 댓글 자리 (Priority: P2)

1. **Given** 남의 답글이 달린 댓글을 쓴 회원이 탈퇴, **Then** 글 상세의 그 자리에는 닉네임·시각 없이 `삭제된 댓글이에요`만.

## Success Criteria *(mandatory)*

- **SC-001**: 수용 시나리오가 자동 테스트(`tests/e2e/comment-manage.spec.ts`, 탈퇴 자리는 `tests/e2e/withdraw.spec.ts`)로 통과한다.
