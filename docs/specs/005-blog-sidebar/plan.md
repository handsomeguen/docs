# Implementation Plan: 블로그 옆 칸·조회수·방문자

**Branch**: `005-blog-sidebar` | **Date**: 2026-10-11 | **Spec**: [spec.md](./spec.md)

## Summary

`posts.view_count`와 `post_views`·`blog_visits` 표, 방문자 쿠키(`src/server/visitor.ts`), Server Action 2개(`src/app/blog/actions.ts`), 클라이언트 숫자 `ViewCount`·`VisitCount`, 옆 칸 `BlogSidebar`와 서버 쿼리 `getBlogSidebar`(`src/server/sidebar.ts`). `listBlogPosts`에 `search`·`tag`를 더했다.

## 결정

1. **세기는 Server Action**: Next.js 16 서버 컴포넌트는 쿠키를 읽기만 한다. 화면이 열릴 때 `useEffect`로 한 번 부르므로 링크 미리 불러오기로는 세지 않는다. `revalidatePath`는 부르지 않고 돌려받은 숫자로 그 자리만 바꾼다.
2. **글 상세 한 번에**: `recordPostView`가 조회 기록과 그 블로그 방문 기록을 함께 남긴다 (요청 1개).
3. **처음 숫자**: 글 상세는 오늘 이 브라우저로 센 적이 없으면 저장값 + 1로 그린다 (`hasViewedToday`).
4. **필터 우선순위**: 검색 → 태그 → 카테고리 중 하나만. 검색·태그 중에는 카테고리에서 고른 항목이 없다.
5. **명세에서 뺀 것**: 마을 검색, 관리 화면 7일 그래프, 프로필 카드, 주인 도구 막대·글 관리, 공감 수는 각 기능이 생길 때 붙인다.

## Project Structure

```text
src/db/schema.ts                # posts.view_count, post_views, blog_visits
drizzle/0006_views_visits.sql
src/lib/kst.ts                  # todayKST, previousDay
src/lib/blog-search.ts          # parseSearchQuery, parseTagFilter, likePattern
src/server/visitor.ts           # readVisitorId, ensureVisitorId
src/server/stats.ts             # recordView, recordVisit, hasViewedToday, getBlogVisitStats, cleanupPostViews
src/server/sidebar.ts           # getBlogSidebar
src/app/blog/actions.ts         # recordPostView, recordBlogVisit
src/components/blog/{BlogSidebar,ViewCount,VisitCount,CategoryNav}.tsx
tests/unit/blog-search.test.ts, tests/e2e/sidebar.spec.ts
```
