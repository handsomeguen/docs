# Implementation Plan: 내 이웃 목록과 이웃 취소

**Branch**: `018-neighbor-list` | **Date**: 2026-10-12 | **Spec**: [spec.md](./spec.md)

## Summary

DB 변경 없음. `listMyNeighbors(userId)`(`src/server/follows.ts`)가 `follows(follower = 나)`에 블로그·프로필을 붙이고, 공개 글 수·최근 공개 글 시각(하위 쿼리, `posts_blog_list_idx` 사용)·`서로 이웃`(`exists follows 상대 → 나`)을 함께 가져온다. 전체 수는 따로 센다. 화면은 서버 컴포넌트 `NeighborList`(`<details>`), 취소는 클라이언트 `UnfollowButton`(확인 창) → Server Action `removeNeighbor`.

## 결정

1. 취소 전용 `unfollow`(DELETE만). 블로그 홈의 `toggleFollow`를 쓰면 두 번 눌렀을 때 다시 추가된다.
2. `removeNeighbor`는 1~64자 문자열만 받고, 조건이 `follower_id = 나`라 남의 이웃 관계는 바꿀 수 없다.
3. 목록은 `<details>`로 접어 둔다(이웃이 많아도 글 카드가 밀리지 않게). 열림 상태는 브라우저가 들고 있어 취소 뒤 다시 그려도 펼친 채다.
4. [✓ 이웃]은 이름이 같으므로 `aria-describedby`로 블로그 이름을 붙인다.
5. 지금 탭: `.btn`이 Tailwind 레이어 밖에 있어 `bg-ink` 유틸리티가 지고 글자만 흰색이 됐다. `.btn-current` 클래스를 만들어 쓴다.
6. 긴 이름 넘침: `break-words`(`overflow-wrap: break-word`)가 body의 `anywhere`를 덮어 inline-flex 링크의 최소 폭이 줄지 않았다. 링크에서 빼서 `anywhere`를 물려받게 한다.

## Project Structure

```text
src/server/follows.ts (listMyNeighbors, unfollow, NEIGHBOR_LIST_LIMIT)
src/app/blog/actions.ts (removeNeighbor)
src/components/feed/{NeighborList,UnfollowButton}.tsx, FeedView.tsx(top 칸, 지금 탭)
src/app/feed/following/page.tsx
src/app/globals.css (.btn-current), src/app/blog/[slug]/page.tsx (제목 링크 줄바꿈)
tests/e2e/neighbor-list.spec.ts
```
