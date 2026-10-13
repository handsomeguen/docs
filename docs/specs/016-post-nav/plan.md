# Implementation Plan: 이전 글·다음 글과 블로그 이름 링크

**Branch**: `016-post-nav` | **Date**: 2026-10-12 | **Spec**: [spec.md](./spec.md)

## Summary

DB 변경 없음(인덱스 `posts_blog_list_idx (blog_id, created_at desc, id desc)`가 이미 있다). `getAdjacentPosts(blogId, postId, viewerIsOwner)`가 쿼리 두 개를 함께 보낸다: 이전 글은 `(created_at, id) < 지금 글` 큰 순 1개, 다음 글은 `>` 작은 순 1개. 주인이 아니면 공개 글만, id·제목만 돌려준다. 화면은 `AdjacentPosts`(서버 컴포넌트).

## 결정

1. 지금 글의 `(created_at, id)`는 DB 안의 부분 쿼리로 비교한다. JS `Date`로 넘기면 마이크로초를 잃어 같은 시각 글이나 자기 자신이 잘못 나온다.
2. 이전 = 더 오래된 글, 다음 = 더 새 글 (명세: 가장 최신 글에는 다음 글이 없다).

## Project Structure

```text
src/server/post.ts (getAdjacentPosts)
src/components/post/AdjacentPosts.tsx
src/app/blog/[slug]/[postId]/page.tsx (블로그 이름 링크, 이전·다음 글, 비공개 탭 제목)
src/app/blog/[slug]/page.tsx (제목 링크)
tests/e2e/post-nav.spec.ts
```
