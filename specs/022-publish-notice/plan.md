# Implementation Plan: 새 글 발행 안내

**Branch**: `022-publish-notice` | **Date**: 2026-10-12 | **Spec**: [spec.md](./spec.md)

## Summary

DB 변경 없음. `savePost`가 새 글일 때만 `?new=1`을 붙이고, 글 상세가 그것을 읽어 `PublishNotice`를 그린다.

## 결정

1. 새 글과 수정은 폼의 `postId`가 비었는지로 가른다(`checkPostInput` 결과의 `postId === null`).
2. 보일지는 순수 함수 `showsPublishNotice(주인인가, 만든 시각, 지금)`(`src/lib/post-rules.ts`): 주인이고 만든 지 10분 안(시계 차이 1분까지 허용). 주소의 `?new`는 아무나 붙일 수 있으므로 서버가 주인·시각으로 다시 본다.
3. `PublishNotice`(`src/components/post/PublishNotice.tsx`, 클라이언트)는 그린 뒤 `window.history.replaceState`로 주소의 `?new`만 지운다(Next 라우터와 이어진다). 그래서 새로고침에는 안내가 없다.
   - 뒤로·앞으로 가기는 Next가 저장해 둔 화면(`?new=1`로 그린 RSC)을 다시 꺼내 안내도 다시 붙는다. 그래서 보일지는 처음 붙을 때 `useSearchParams()`에 `new`가 있는지로 정해 `useState`에 담는다. 다시 꺼낸 화면은 이미 `?new`가 빠진 주소에서 붙어 아무것도 그리지 않는다(깜빡임 없음).
   - 주소를 처음 열 때(전체 새로 불러오기)는 이 효과가 Next가 `history`를 감싸는 효과보다 먼저 돈다. 그때 원래 `replaceState(null)`을 부르면 Next의 기록 상태가 지워지고 라우터 주소에는 `?new=1`이 남아, 다른 화면에 갔다가 뒤로 오면 주소만 바뀌고 화면은 그대로다. 그래서 `setTimeout`으로 한 틈 미뤄 Next가 감싼 `replaceState`를 쓴다.
4. 노란 카드는 `.notice-yellow`(`#fff8e1` 바탕, `#f2c94c` 테두리, 본문 글자색). `.btn`처럼 Tailwind 레이어 밖 전용 클래스로 둔다.

## Project Structure

```text
src/app/write/actions.ts (새 글이면 ?new=1)
src/lib/post-rules.ts (showsPublishNotice), src/components/post/PublishNotice.tsx, src/app/blog/[slug]/[postId]/page.tsx, src/app/globals.css
tests/e2e/publish-notice.spec.ts, tests/unit/post-rules.test.ts
```
