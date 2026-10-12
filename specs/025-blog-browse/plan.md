# Implementation Plan: 블로그 둘러보기

**Branch**: `025-blog-browse` | **Date**: 2026-10-12 | **Spec**: [spec.md](./spec.md)

## Summary

DB 변경 없음. 공개 글을 블로그별로 묶은 쿼리와 마을 소식 1페이지 위쪽 칸을 더한다.

## 결정

1. `listActiveBlogs(12)`(`src/server/feed.ts`): 공개 글을 `blog_id`로 묶어 가장 최근 시각과 수를 구하고, 블로그·프로필을 붙여 최근 공개 글 순(같으면 블로그 ID 큰 순)으로 정렬한다.
2. `BlogBrowse`(`src/components/feed/BlogBrowse.tsx`, 서버)를 `FeedView`의 `top` 자리에 1페이지일 때만 넣는다. 2페이지 이상은 쿼리도 부르지 않는다.
3. 칸은 글 카드(`article`)와 헷갈리지 않게 목록(`ul`/`li`)으로, 2열(넓으면 3열) 그리드.

## Project Structure

```text
src/server/feed.ts (listActiveBlogs), src/components/feed/BlogBrowse.tsx, src/app/feed/page.tsx
tests/e2e/feed-browse.spec.ts
```
