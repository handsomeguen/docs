# Implementation Plan: 사진·파일 첨부

**Branch**: `002-attachments` | **Date**: 2026-10-11 | **Spec**: [spec.md](./spec.md)

## Summary

글쓰기 에디터에서 사진·파일을 올리면 `POST /api/uploads`(Route Handler)가 확인·저장하고 `attachments`에 붙지 않은 행을 만든다. 글을 저장하는 트랜잭션이 본문에 남은 내 첨부를 잠그고 그 글에 붙인다. `GET /files/[키]`가 붙은 글의 공개 설정으로 권한을 매번 확인해 내려준다.

## Technical Context

001-blog-core와 같다: Next.js 16.4 App Router, TypeScript, Drizzle 0.45 + PostgreSQL 16, better-auth 1.7, Tiptap 3, sanitize-html, Vitest, Playwright. 새 의존성은 `@tiptap/extension-image`(3.31.4)뿐이다.

## Constitution Check

| 원칙 | 확인 |
|---|---|
| 명세가 기준 | POST-07·09 문구·숫자 그대로. 다르게 정한 점(다시 올리기 미룸, 프로필 사진 없음)은 spec 맨 위에 적었다 |
| 서버가 마지막 방어선 | 형식·크기·매직 넘버·Origin·로그인·붙일 수 있는지 모두 서버가 다시 본다 |
| 테스트로 증명 | 수용 시나리오마다 E2E 또는 단위 테스트 |
| 작게 | 저장소는 디스크 하나, 정리 작업은 손으로 실행 |

## 결정 (research)

1. **올리기는 Route Handler**: Server Action은 본문 1MB 상한이 있고 30MB 파일을 받아야 한다. 대신 Server Action이 해 주던 Origin 확인을 같은 기준(`X-Forwarded-Host` 또는 `Host`)으로 직접 한다.
2. **형식은 확장자로 정하고, 사진은 매직 넘버로 다시 확인**: 라이브러리 없이 앞 12바이트만 본다 (PNG `89 50 4E 47`, JPG `FF D8 FF`, GIF `GIF8`, WEBP `RIFF....WEBP`). 파일은 확장자 표의 MIME으로만 내려준다.
3. **붙지 않은 상태 → 저장할 때 붙임**: 첨부 행은 올리자마자 `post_id` NULL로 생긴다. 글 저장 트랜잭션에서 본문 키의 행을 키 순서로 `FOR UPDATE` 잠그고, 붙일 수 있는 것만 정화의 `known`으로 넘긴 뒤 `post_id`를 맞춘다.
4. **`detached_at`은 트리거로**: `post_id`가 값→NULL이면 `now()`, 다시 붙으면 NULL. 글 삭제의 `ON DELETE SET NULL`도 트리거를 지난다.
5. **파일 카드는 글자 없는 `<a>`**: `data-name`·`data-size-label`을 CSS `::before/::after`로 그려 `content_text`에 들어가지 않게 한다. `aria-label`이 화면 낭독용이다.
6. **캐시**: `private, no-cache` + `ETag: "키"`. 304도 권한을 확인한 뒤에만.

## Project Structure

```text
src/lib/attachments.ts            # 형식 표·한도·문구·attachmentProblem·sniffImage·cleanFileName·formatBytes·attachmentAccess
src/server/storage.ts             # 디스크 저장·읽기·삭제·목록
src/server/attachments.ts         # createAttachment, findReadableAttachment, lockLinkable, syncPostAttachments, cleanup
src/app/api/uploads/route.ts      # POST 올리기
src/app/files/[key]/route.ts      # GET 보기·내려받기
src/components/editor/FileCard.ts # Tiptap 노드
src/components/editor/useUpload.ts
src/lib/sanitize.ts               # img·a[data-file] 허용 (known)
src/server/post.ts                # savePost 트랜잭션에 붙이기
scripts/cleanup-posts.ts          # npm run posts:cleanup
drizzle/0002_attachments.sql
tests/unit/attachments.test.ts, tests/unit/sanitize.test.ts
tests/e2e/attachments.spec.ts, tests/e2e/attachment-access.spec.ts
```
