# Implementation Plan: 공감 (♥)

**Branch**: `007-likes` | **Date**: 2026-10-11 | **Spec**: [spec.md](./spec.md)

## Summary

`post_likes` 표(마이그레이션 `0007_likes`), `src/server/likes.ts`(`likeCountSql`·`getLikeState`·`toggleLike`), Action `toggleLike`(`src/app/blog/actions.ts`), 클라이언트 `LikeButton`(`useOptimistic` + `useTransition`). 글 목록·태그별 글 쿼리에 `likeCount` 하위 쿼리.

## 결정

1. **낙관적 업데이트**: 누르면 바로 뒤집어 보여 주고, Action 뒤 `revalidatePath`로 서버 값이 다시 그려진다. 실패하거나 저장하지 않은 경우에도 서버 값으로 돌아온다.
2. **보상·알림 자리**: 게임이 생기면 `toggleLike` 트랜잭션의 INSERT 뒤에 붙인다 (명세의 `notifyActivity`·`grantReward`).

## Project Structure

```text
src/db/schema.ts               # postLikes
drizzle/0007_likes.sql
src/server/likes.ts
src/app/blog/actions.ts        # toggleLike
src/components/post/LikeButton.tsx
tests/e2e/likes.spec.ts
```
