# Implementation Plan: 태그

**Branch**: `003-tags` | **Date**: 2026-10-11 | **Spec**: [spec.md](./spec.md)

## Summary

`PostForm`에 태그 칸을 더하고, `savePost` 트랜잭션에서 `parseTags` 결과로 `tags`·`post_tags`를 맞춘다. 글 상세에 배지, 새 화면 `/tags/[name]`에 공개 글 목록과 인기 태그 상자.

## Technical Context

001·002와 같다. 새 의존성 없음.

## Constitution Check

명세 문구·숫자 그대로(조용히 빼기, 300자 문구, `%` 태그 탭 제목). 다르게 정한 점(마을 소식 없음 → 인기 태그는 태그 화면에만)은 spec 맨 위에 적었다.

## 결정

1. **`parseTags`는 순수 함수** (`src/lib/post-rules.ts`): 브라우저와 서버가 같이 쓴다.
2. **태그 화면 주소의 `%`**: Next 16은 페이지 `params`에 인코딩된 값을 넘기므로 `decodeURIComponent`를 한 번 하고, 잘못된 인코딩이면 빈 목록. `generateMetadata`도 같은 함수로 맞춘다 (명세 #20의 차이는 이 버전에서 다시 확인).
3. **목록 카드**: `PostCard`에 블로그 이름·주소를 받는 선택 칸을 더해 재사용한다.

## Project Structure

```text
src/db/schema.ts               # tags, post_tags
drizzle/0004_tags.sql
src/lib/post-rules.ts          # parseTags, TAGS_MAX_CHARS, 문구
src/server/tags.ts             # syncPostTags, getPostTags, listTagPosts, getPopularTags
src/server/post.ts             # savePost에서 syncPostTags
src/app/tags/[name]/page.tsx
src/components/post/TagList.tsx, src/components/blog/PopularTags.tsx
tests/unit/tags.test.ts, tests/e2e/tags.spec.ts
```
