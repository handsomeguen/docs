# Implementation Plan: 새 글 발행 안내

**Branch**: `022-publish-notice` | **Date**: 2026-10-12 | **Spec**: [spec.md](./spec.md)

## Summary

DB 변경 없음. `savePost`가 새 글일 때만 `?new=1`을 붙이고, 글 상세가 그것을 읽어 `PublishNotice`를 그린다.

## 결정

1. 새 글과 수정은 폼의 `postId`가 비었는지로 가른다(`checkPostInput` 결과의 `postId === null`).
2. 보일지는 순수 함수 `showsPublishNotice(주인인가, 만든 시각, 지금)`(`src/lib/post-rules.ts`): 주인이고 만든 지 10분 안(시계 차이 1분까지 허용). 주소의 `?new`는 아무나 붙일 수 있으므로 서버가 주인·시각으로 다시 본다.
3. `PublishNotice`(`src/components/post/PublishNotice.tsx`, 클라이언트)는 그린 뒤 `window.history.replaceState`로 주소의 `?new`만 지운다(Next 라우터와 이어진다). 그래서 새로고침·뒤로 가기에는 안내가 없다.
4. 노란 카드는 `.notice-yellow`(`#fff8e1` 바탕, `#f2c94c` 테두리, 본문 글자색). `.btn`처럼 Tailwind 레이어 밖 전용 클래스로 둔다.

## Project Structure

```text
src/app/write/actions.ts (새 글이면 ?new=1)
src/lib/post-rules.ts (showsPublishNotice), src/components/post/PublishNotice.tsx, src/app/blog/[slug]/[postId]/page.tsx, src/app/globals.css
tests/e2e/publish-notice.spec.ts, tests/unit/post-rules.test.ts
```
