# Implementation Plan: 프로필 사진

**Branch**: `030-profile-photo` | **Date**: 2026-10-12 | **Spec**: [spec.md](./spec.md)

## Summary

`profiles.photo_key`(UNIQUE, `attachments.key` 참조, 첨부가 지워지면 NULL)를 더하고(마이그레이션 `0011_profile_photo`), 첨부 규칙 곳곳에 "지금 프로필 사진인가"를 넣는다.

## 결정

1. 올리기는 글쓰기와 같은 `POST /api/uploads`를 쓰고, 받은 키를 Server Action `saveProfilePhoto`로 정한다. 따로 올리는 길을 만들지 않는다.
2. `setProfilePhoto`(`src/server/profile.ts`): 한 트랜잭션에서 프로필을 먼저 잠그고(글 저장 `savePost`도 본문 첨부를 잠그기 전에 프로필을 `FOR SHARE`로 잠근다, 같은 순서라 교착 없음) 첨부를 `FOR UPDATE`로 잠가 내 것·사진·글에 안 붙음·1시간 안에 올림(하루 지난 것을 지우는 정리 작업과 겹치지 않게)을 확인하고 키를 바꾸고, 전 사진의 `detached_at`을 지금으로 둔다(정리 대상이 된다).
3. `isProfilePhotoSql`(`src/server/attachments.ts`)을 보기(`findReadableAttachment`), 붙여 넣기 판정(`classifyPasted`·`reuploadAttachment`), 글에 붙이기(`lockLinkable`·`syncPostAttachments`), 정리(`cleanupAttachments`)에 쓴다. 판정은 순수 함수 `attachmentAccess`·`pasteAction`(`src/lib/attachments.ts`)에 `isProfilePhoto`를 더해 단위 시험한다.
4. `ProfileAvatar`는 사진이 있으면 `<img>`, 없으면 첫 글자 동그라미. 블로그 홈 머리글에만 쓴다.

## Project Structure

```text
drizzle/0011_profile_photo.sql, src/db/schema.ts
src/lib/attachments.ts, src/server/{attachments,profile,blog}.ts
src/app/settings/account/{page,actions}.ts(x), src/app/blog/[slug]/page.tsx
src/components/profile/ProfileAvatar.tsx, src/components/settings/ProfilePhotoForm.tsx
tests/unit/attachments.test.ts, tests/e2e/profile-photo.spec.ts
```
