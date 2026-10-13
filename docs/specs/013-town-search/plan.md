# Implementation Plan: 마을 검색

**Branch**: `013-town-search` | **Date**: 2026-10-12 | **Spec**: [spec.md](./spec.md)

## Summary

DB 변경 없음. `listFeed`(마을 소식)에 `search`(제목·본문 `ILIKE`, `likePattern`)를 더하고, `searchBlogs(term, 8)`(`src/server/blog.ts`)로 블로그 이름·닉네임을 찾는다. 블로그 홈은 주인일 때만 `parseSearchQuery(q)`를 읽고, `q`가 있으면 블로그 검색·태그·카테고리보다 먼저 쓴다.

## 결정

1. **블로그 순서**: 공개 글의 가장 최근 `created_at` 내림차순(글 없으면 뒤) → 블로그 `created_at` → `id`.
2. **성능**: 명세 열린 질문(NF-07)대로 `ILIKE '%…%'` 그대로 둔다.

## Project Structure

```text
src/server/feed.ts (listFeed search), src/server/blog.ts (searchBlogs)
src/lib/blog-search.ts (문구)
src/app/blog/[slug]/page.tsx, src/components/blog/BlogSidebar.tsx
tests/e2e/town-search.spec.ts
```
