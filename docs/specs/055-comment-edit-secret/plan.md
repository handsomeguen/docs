# Implementation Plan: 댓글 수정·비밀 댓글

**Branch**: `055-comment-edit-secret` | **Spec**: [spec.md](spec.md)

- DB: `comments`·`replies`에 `is_secret boolean not null default false`, `edited_at timestamptz` (마이그레이션 `0026_comment_edit_secret`, 열 추가만).
- 순수 함수: `src/lib/comment-rules.ts` — `canSeeSecret(보는 회원, 블로그 주인 ID, 작성자 ID들)`, `isCommentHidden`, `isReplyHidden`, 문구 `SECRET_PLACEHOLDER`·`COMMENT_ERRORS.editNotFound`.
- 조회: `getCommentThread`가 줄마다 `secret`·`hidden`·`editedAt`·`canEdit`를 계산하고, 가려지는 줄은 `content`를 빈 글자로. 볼 수 없는 비밀 댓글은 `canReply` 거짓. `getBlogSidebar(blogId, 주인인지, 보는 회원)`은 같은 규칙을 SQL `case`로 써서 내용을 비우고 `hidden`을 돌려준다.
- 처리: `editComment`·`editReply` (`src/server/comments.ts`) — `UPDATE … WHERE id AND author_id = 나 AND deleted_at IS NULL AND 글이 공개이거나 내 글`, `edited_at`은 내용이 바뀔 때만 `now()`. Action `updateComment`·`updateReply` (`src/app/blog/actions.ts`): `requireMember` → `parseId` → `checkComment`. `addComment`·`addReply`는 `isSecret`을 받고, `createReply`는 볼 수 없는 비밀 원댓글을 거절한다.
- 화면: `CommentSection`의 공용 `CommentForm`에 `🔒 비밀 댓글` 체크(누르는 영역 44px)와 수정 모드(채워진 내용, 자동 초점, Esc, [취소] [저장]). `Line`에 [수정]·`(수정됨)`·🔒·`🔒 비밀 댓글이에요`. 옆 칸 `BlogSidebar`, 댓글 관리 화면에 🔒.
