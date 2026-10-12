# Implementation Plan: 지붕 색 고르기

**Branch**: `041-roof-color` | **Date**: 2026-10-12 | **Spec**: [spec.md](./spec.md)

## Summary

`blogs.roof_color`(NULL 허용, CHECK 10개 코드값, `0019_roof_color`)를 더한다. 순서는 저장하지 않고 블로그 번호를 씨앗으로 섞는다(`src/lib/roof.ts` `roofOrder`, mulberry32).

## 결정

1. 순수 함수 `src/lib/roof.ts`: `ROOF_CODES`·`ROOF_LABELS`·`ROOF_HEX`·`roofOrder`·`unlockedRoofCount`(= 집 단계)·`roofOpenLevel`·`currentRoof`·`canChooseRoof`.
2. `chooseRoof`(src/server/house.ts): 가구 놓기와 같은 잠금 순서[회원 행 KEY SHARE → 회원 잠금] 뒤 지갑 레벨로 집 단계를 다시 계산해 열린 색인지 확인하고 저장. 서버 액션 `chooseRoofColor` → `revalidatePath("/", "layout")`.
3. 마을 `toHouse`가 `ROOF_HEX[currentRoof(blogId, roofColor)]`. 시험 훅으로 캔버스 `data-roofs="자리:색 …"`.
4. `HouseRoom`에 주인만 `roof` 값을 넘긴다. 가구 놓기와 지붕 색 창은 하나만 열린다.

## Project Structure

```text
src/lib/roof.ts, src/lib/art/town.ts, src/db/schema.ts, drizzle/0019_roof_color.sql
src/server/house.ts, src/server/town.ts, src/app/house/actions.ts, src/components/house/HouseRoom.tsx, src/app/blog/[slug]/page.tsx, src/components/town/TownGame.tsx
tests/unit/{roof,town}.test.ts, tests/e2e/roof.spec.ts
```
