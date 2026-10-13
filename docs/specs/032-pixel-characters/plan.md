# Implementation Plan: 도트 캐릭터 고르기

**Branch**: `032-pixel-characters` | **Date**: 2026-10-12 | **Spec**: [spec.md](./spec.md)

## Summary

마이그레이션 `0013_pixel_characters`: `items`(code UNIQUE, type CHECK, 가격 0 이상), `user_items`(기본 키 `(user_id, item_id)`, 회원 CASCADE), `profiles.character_item_id`와 복합 외래 키 `profiles_character_owned_fk (user_id, character_item_id) → user_items`(가진 캐릭터만 장착). 기본 캐릭터 2행을 넣고 기존 회원에게 남자 주민을 준다.

## 결정

1. 도트는 `CHARACTER_ART`(16줄 × 16글자, 글자별 색)와 `pixelRects`(같은 줄에서 이어진 같은 색을 사각형 하나로). `PixelCharacter`는 `shape-rendering: crispEdges` SVG.
2. 가입: `signUp` 액션은 ID 모양만 보고(`parseId`), `createMember`가 `findStarterCharacter`로 다시 확인한 뒤 `user_items`·`profiles.character_item_id`에 넣는다.
3. 표시: 헤더는 `getCharacter`, 블로그 홈은 `getBlogBySlug`가 `items.art`를 함께 읽어 `ProfileAvatar`의 사진 없는 경우에 그린다.

## Project Structure

```text
drizzle/0013_pixel_characters.sql, src/db/schema.ts
src/lib/art/characters.ts, src/components/game/PixelCharacter.tsx
src/server/{items,signup,blog}.ts, src/app/(auth)/actions.ts, src/lib/auth-rules.ts
src/components/auth/AuthPanel.tsx, src/app/page.tsx, src/app/layout.tsx
src/components/profile/ProfileAvatar.tsx, src/app/blog/[slug]/page.tsx
scripts/{create-admin,seed}.ts
tests/unit/pixel-art.test.ts, tests/e2e/characters.spec.ts
```
