# Implementation Plan: 즐겨찾는 이웃 ⭐

**Branch**: `026-favorite-neighbors` | **Date**: 2026-10-12 | **Spec**: [spec.md](./spec.md)

## Summary

`follows.is_favorite`(참/거짓, 기본 거짓, `drizzle/0010_follows_favorite.sql`)를 더하고, 내 이웃 목록에 ⭐ 버튼, 이웃 새 글 정렬에 즐겨찾기 묶음을 더한다.

## 결정

1. 규칙은 `src/lib/social.ts`: `FAVORITE_LIMIT = 10`, `FAVORITE_WINDOW_DAYS = 7`, `favoriteWindowStart`(오늘 포함 7일 중 첫날의 한국 0시), `favoriteRefusal`(켤 때만 공지 블로그·10명 확인), 문구.
2. `toggleFavorite`(`src/server/follows.ts`): 트랜잭션 + `pg_advisory_xact_lock(hashtext('fav:' + 나))`로 같은 회원의 요청을 줄 세움 → 이웃 행 확인 → 켤 때만 거절 규칙(공지 블로그를 뺀 즐겨찾기 수) → 뒤집기. 액션 `favoriteNeighbor`(`src/app/blog/actions.ts`)는 값 모양을 보고 결과 문구를 돌려준다.
3. `listMyNeighbors`는 `isFavorite`·`isNotice`·`favorites`(공지 블로그를 뺀 수)를 더하고 즐겨찾기 먼저 정렬.
4. `listFeed({ followerId })`는 `pinned` = 글이 `favoriteWindowStart()` 이후이고 내가 그 주인을 즐겨찾기함. 정렬 `pinned desc, created_at desc, id desc`. 마을 소식·검색은 `pinned`가 늘 거짓.
5. `FavoriteButton`(클라이언트)은 결과 문구를 줄 아래 `role="alert"`로. `UnfollowButton`이 지운 뒤 초점을 옮길 때 ⭐ 버튼이 아닌 옆 줄의 [✓ 이웃]을 찾도록 `data-unfollow`로 고른다.
6. 공지 블로그 시험은 진짜 `/@notice`를 쓰므로 그 주소를 잠깐 옮기는 시험과 한 묶음(`mode: "default"`)으로 차례로 돈다.

## Project Structure

```text
drizzle/0010_follows_favorite.sql, src/db/schema.ts
src/lib/social.ts, src/server/follows.ts, src/server/feed.ts, src/app/blog/actions.ts
src/components/feed/{FavoriteButton,NeighborList,UnfollowButton,FeedView}.tsx, src/components/blog/PostCard.tsx
tests/unit/social.test.ts, tests/e2e/favorites.spec.ts, tests/e2e/admin.spec.ts, tests/e2e/neighbor-list.spec.ts
```
