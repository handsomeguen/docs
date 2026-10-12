# Implementation Plan: 댓글 관리

**Branch**: `024-comment-manage` | **Date**: 2026-10-12 | **Spec**: [spec.md](./spec.md)

## Summary

DB 변경 없음. 내 블로그 글의 댓글·답글을 합쳐 읽는 쿼리와 `/settings/comments` 화면을 더하고, 지우기는 글 상세의 액션(`deleteComment`·`deleteReply`)을 그대로 쓴다.

## 결정

1. `listBlogComments(blogId, page)`(`src/server/comments.ts`): 댓글과 답글을 `union all`로 합쳐 지우지 않은 것만 `created_at desc` 순서로 20개씩, 전체 수도 함께. 비공개 글도 내 블로그라 포함한다. 작성자 회원 ID는 화면으로 넘기지 않는다(닉네임·블로그 주소만).
2. 지우기 권한은 이미 있는 `mayDelete`(작성자·그 글의 블로그 주인·관리자)가 UPDATE 조건으로 다시 본다. 확인 창은 `confirmFirst`로 폼 액션 안에서 묻는다.
3. 주소는 `?page=`만. 거르기·검색은 없다(필요하면 나중에).
4. 탈퇴한 회원의 자리: `getCommentThread`가 작성자가 없는 줄에 `withdrawn`을 넘기고, 글 상세는 그 줄의 닉네임·시각 줄을 그리지 않는다. 탈퇴 정리는 그런 줄을 늘 지운 자리로 남기므로 내용은 이미 `삭제된 댓글이에요`다.

## Project Structure

```text
src/server/comments.ts (listBlogComments, withdrawn), src/app/settings/comments/page.tsx, src/components/settings/CommentRowDelete.tsx
src/app/settings/blog/page.tsx·src/app/blog/[slug]/page.tsx (💬 댓글 관리 링크), src/components/post/CommentSection.tsx
tests/e2e/comment-manage.spec.ts, tests/e2e/manage.spec.ts(도구 막대), tests/e2e/withdraw.spec.ts
```
