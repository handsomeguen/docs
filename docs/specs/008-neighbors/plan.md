# Implementation Plan: 이웃·마을 소식

**Branch**: `008-neighbors` | **Date**: 2026-10-11 | **Spec**: [spec.md](./spec.md)

## Summary

`follows` 표(마이그레이션 `0008_follows`), `src/server/follows.ts`(`countFollowers`·`isFollowing`·`toggleFollow`), `src/server/feed.ts`(`listFeed({ page, followerId? })`), Action `toggleFollow(formData)`(`src/app/blog/actions.ts`), `FollowButton`(서버 컴포넌트 form), 화면 `/feed`·`/feed/following`과 둘이 같이 쓰는 `FeedView`. `PostCard`에 `author`(작성자 줄). `pageNumbers`(`src/lib/ids.ts`)로 페이지 번호 생략.

## 결정

1. **상대 ID는 숨은 입력칸으로**: 명세는 `toggleFollow.bind(null, ownerId)`지만, 다른 Action과 같이 FormData(`followeeId`)로 받는다. 서버가 어차피 모양·자기 자신·블로그 유무를 다시 본다.
2. **미리 바꾸지 않음**: 공감과 달리 서버 처리 뒤 `revalidatePath`로 다시 그린다 (명세 그대로).
3. **즐겨찾기 없음**: `is_favorite`는 마을이 생길 때 마이그레이션으로 더한다.
4. **페이지 번호 생략**: 마을 소식은 페이지가 수백 개가 될 수 있어 POST-05의 `…` 규칙을 이번에 넣었다.

## Project Structure

```text
src/db/schema.ts                   # follows
drizzle/0008_follows.sql
src/server/follows.ts, src/server/feed.ts
src/app/blog/actions.ts            # toggleFollow
src/app/feed/page.tsx, src/app/feed/following/page.tsx
src/components/feed/FeedView.tsx, src/components/blog/FollowButton.tsx
src/components/blog/PostCard.tsx   # author
src/components/blog/Pagination.tsx, src/lib/ids.ts (pageNumbers)
src/app/layout.tsx                 # 헤더 📋 마을 소식
tests/e2e/neighbors.spec.ts, tests/unit/ids.test.ts
```
