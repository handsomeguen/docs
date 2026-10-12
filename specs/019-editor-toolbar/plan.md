# Implementation Plan: 에디터 도구 모음 정리

**Branch**: `019-editor-toolbar` | **Date**: 2026-10-12 | **Spec**: [spec.md](./spec.md)

## Summary

DB·서버 변경 없음. StarterKit 3에 Underline·Strike·UndoRedo가 이미 들어 있고 정화(`src/lib/sanitize.ts`)도 `u`·`s`를 허용하므로 `RichEditor`의 도구 모음만 고친다.

## 결정

1. 도구를 묶음 배열로 두고 묶음 사이에 `role="separator"`를 그린다(`role="toolbar"` 안).
2. [↶]·[↷]는 `editor.can().undo()/redo()`가 거짓이면 `disabled`. 켜고 끄는 도구가 아니므로 `aria-pressed`가 없다(구분선도 같다).
3. 고른 도구는 `.btn-current`(018에서 만든 진한 바탕). 전에는 연두색 `.btn-selected`였다.
4. 링크 창: 값을 다듬어 `''`이나 `https://`면 `extendMarkRange("link").unsetLink()`. 전에는 `https://`만 남기면 `href="https://"`인 깨진 링크가 생겼다. 규칙은 `linkHref`(`src/lib/post-rules.ts`): scheme이 없으면 `https://`를 붙이고(그대로면 글 주소 기준 상대 주소라 404), http·https·mailto·`/…` 밖의 주소는 저장할 때 정화가 href를 지워 죽은 링크가 되므로 거부한다.
5. 초점: `.tiptap { outline: none }`은 두고 `[data-editor-root]:has(.tiptap:focus)`에 3px 초점선을 준다(본문 칸만 초점선이 없던 NF-18 빈틈).
6. 지우기 확인: `confirmFirst(문구, 서버 액션)`(`src/lib/confirm-action.ts`)이 확인을 묻는 클라이언트 함수를 만들어 폼 `action`에 단다. 서버 액션을 바로 달면 서버가 진짜 POST 폼을 그려 확인 없이 실행될 수 있다. 글 상세·글 관리·댓글·카테고리·관리자 [삭제]에 쓴다.

## Project Structure

```text
src/components/editor/RichEditor.tsx (Toolbar)
src/app/globals.css (에디터 초점선)
src/lib/confirm-action.ts, src/components/post/{DeletePostButton,CommentSection}.tsx, src/components/settings/{PostRowActions,CategoryManager}.tsx, src/components/admin/AdminDeleteButton.tsx (확인 창)
tests/e2e/editor.spec.ts
```
