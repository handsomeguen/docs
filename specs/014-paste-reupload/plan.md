# Implementation Plan: 내 다른 글 첨부 붙여 넣기

**Branch**: `014-paste-reupload` | **Date**: 2026-10-12 | **Spec**: [spec.md](./spec.md)

## Summary

DB 변경 없음. 판정은 순수 함수 `pasteAction`(`src/lib/attachments.ts`: keep / reupload / drop). 에디터 `handlePaste`가 HTML에 `/files/키`가 있으면 `useUpload().pasteHtml`로 넘기고, 이것이 Server Action `classifyPastedAttachments(keys, postId)`(1~50개)로 판정을 받은 뒤 reupload마다 `reuploadPastedAttachment(key, postId)`(서버에서 다시 판정 → `copyFile`로 새 키 → `attachments` INSERT, `post_id` NULL)를 부르고, 주소를 바꾸거나 빼서 `insertContent`로 넣는다.

## 결정

1. 실패 문구의 이름은 내 첨부의 원래 이름 (남의 첨부 이름은 브라우저에 보내지 않는다).
2. 다시 올린 첨부는 저장하지 않으면 다른 붙지 않은 첨부처럼 정리 작업이 지운다.

## Project Structure

```text
src/lib/attachments.ts (pasteAction, PASTE_KEYS_MAX)
src/server/storage.ts (copyFile), src/server/attachments.ts (classifyPasted, reuploadAttachment)
src/app/write/actions.ts (classifyPastedAttachments, reuploadPastedAttachment)
src/components/editor/useUpload.ts (pasteHtml), RichEditor.tsx, src/components/post/PostForm.tsx (postId)
tests/unit/attachments.test.ts, tests/e2e/paste-reupload.spec.ts, tests/e2e/attachment-access.spec.ts
```
