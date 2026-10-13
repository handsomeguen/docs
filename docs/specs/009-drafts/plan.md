# Implementation Plan: 임시 저장

**Branch**: `009-drafts` | **Date**: 2026-10-11 | **Spec**: [spec.md](./spec.md)

## Summary

`src/lib/draft.ts`(키·읽기·쓰기·지우기·형식 검사·`임시 저장됨 HH:mm`). `PostForm`은 `draftOwnerId`가 있으면(새 글 화면만) 먼저 빈 자리(`aria-busy`)를 그리고, 그린 뒤 임시 글을 읽어 `confirm`으로 묻고, 고른 값으로 안쪽 `Form`을 만든다. `Form`은 사용자가 바꾼 칸이 있을 때만 2초 debounce로 쓴다(에디터가 처음 만들어질 때의 `onChange`는 바뀜이 아님). 발행 버튼은 남은 예약을 바로 쓰고 멈추며, 오류면 다시 풀고, 성공해 화면이 사라지면 지운다. `SignOutButton`은 클라이언트 컴포넌트가 되어 누를 때 지운다.

## 결정

1. **성공 판단**: `savePost`는 성공하면 글 상세로 `redirect`한다. 오류면 같은 화면에 남으므로, "발행을 보낸 뒤 화면이 사라짐"을 성공으로 본다. 발행 안내 화면을 따로 만들지 않았다.
2. **최신 값**: 타이머가 오래된 값을 쓰지 않게 마지막 값을 ref에 두고(렌더 뒤 effect에서 갱신) 저장할 때 읽는다.

## Project Structure

```text
src/lib/draft.ts
src/components/post/PostForm.tsx    # draftOwnerId, 상태 줄
src/app/write/page.tsx              # draftOwnerId = 회원 ID
src/components/auth/SignOutButton.tsx, src/app/layout.tsx
tests/unit/draft.test.ts, tests/e2e/drafts.spec.ts
```
