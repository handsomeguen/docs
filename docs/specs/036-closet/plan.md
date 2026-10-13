# Implementation Plan: 꾸미기

**Branch**: `036-closet` | **Date**: 2026-10-12 | **Spec**: [spec.md](./spec.md)

## Summary

마이그레이션 `0017_closet`: `avatar_equips`(기본 키 `(user_id, slot)`, 부위 CHECK, `user_items` 복합 외래 키 ON DELETE CASCADE), `blogs.background_item_id`(`user_items` 복합 외래 키), 기본 배경 `bg_meadow`를 모든 회원에게 주고 건다.

## 결정

1. `equipItem`(`src/server/items.ts`): 회원 행 KEY SHARE(탈퇴와 같은 순서) → 가진 아이템인지 → 배경이면 블로그에 걸고, 모자·옷·소품이면 화면이 보낸 `wear`대로 부위에 넣거나 바꾸고(ON CONFLICT DO UPDATE), `wear = false`면 그 아이템을 벗긴다(탭 두 개에서 눌러도 엇갈리지 않게).
2. `avatarLayers`(`src/lib/shop.ts`)가 그리는 순서를 정하고, `getAvatar`가 헤더·블로그 홈에 그릴 그림 목록을 준다. `ProfileAvatar`의 `character`는 그림 목록이 됐다.
3. `ClosetView`는 낙관적으로 바꾸고, 실패하면 그 클릭이 바꾼 칸만(그 사이 다른 클릭이 바꾸지 않았을 때) 되돌린다. 저장 표시는 2초 뒤 사라진다.
4. `Miniroom`: 배경 그림을 칸에 꽉 채우고(`PixelLayers stretch`) 캐릭터가 통통 튄다(`hop`, 움직임 줄이기 설정이면 멈춤).

## Project Structure

```text
drizzle/0017_closet.sql, src/db/schema.ts
src/lib/{shop.ts,art/items.ts}, src/server/{items,blog,signup}.ts, scripts/{seed,create-admin}.ts
src/app/closet/{page,actions}.ts(x), src/components/closet/ClosetView.tsx, src/components/game/{Miniroom,PixelLayers}.tsx
src/app/layout.tsx, src/app/blog/[slug]/page.tsx, src/components/profile/ProfileAvatar.tsx, src/components/shop/ShopView.tsx
tests/unit/shop.test.ts, tests/e2e/{closet,shop,manage}.spec.ts
```
