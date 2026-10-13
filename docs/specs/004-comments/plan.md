# Implementation Plan: 댓글과 답글

**Branch**: `004-comments` | **Date**: 2026-10-11 | **Spec**: [spec.md](./spec.md)

## Summary

`comments`·`replies` 표, Server Action 4개(`addComment`·`addReply`·`deleteComment`·`deleteReply`, `src/app/blog/actions.ts`), 글 상세 아래 `CommentSection`(클라이언트). 목록 카드의 `💬 N`은 삭제되지 않은 댓글+답글 수를 서브쿼리로 센다.

## 결정

1. **폼은 Action을 직접 부른다**: `startTransition` 안에서 `await addComment(formData)` → 성공이면 입력칸 비우기, 실패면 문구만. 실패해도 입력이 남는다.
2. **삭제 자리**: 행은 지우지 않고 내용을 비운다. 화면으로 넘기는 데이터에 작성자 회원 ID를 넣지 않고, 서버가 `canDelete`·`canReply`를 계산해 넘긴다.
3. **명세에서 뺀 것**: 보상·알림·캐릭터·탈퇴 정리는 해당 기능이 생길 때 붙인다. 트랜잭션 자리는 남겨 둔다.

## Project Structure

```text
src/db/schema.ts             # comments, replies
drizzle/0005_comments.sql
src/lib/comment-rules.ts     # checkComment, 문구, formatDateTime
src/server/comments.ts       # getCommentThread, createComment, createReply, removeComment, removeReply, 댓글 수
src/app/blog/actions.ts
src/components/post/CommentSection.tsx
tests/unit/comment-rules.test.ts, tests/e2e/comments.spec.ts
```
