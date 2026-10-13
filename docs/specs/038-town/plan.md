# Implementation Plan: 마을 광장

**Branch**: `038-town` | **Date**: 2026-10-12 | **Spec**: [spec.md](./spec.md)

## Summary

DB 변경 없음. `/town` 서버 페이지가 집 목록(`getMemberHouses`·`getGuestHouses`)과 내 아바타를 읽어 클라이언트 `TownGame`(캔버스)에 넘긴다.

## 결정

1. 좌표·이동 규칙은 `src/lib/town/layout.ts` 한 곳(순수 함수: `step` 축별 이동과 미끄러짐, `nearestEntrance`, `normalize`). 단위 시험이 같은 함수로 모든 입구에 걸어서 닿는지 넓이 우선 탐색으로 본다.
2. 건물·집·나무와 입구는 `townLayout(houses, member)`(`structures.ts`)가 만든다. 나무는 고정 씨앗이라 매번 같다.
3. 그림은 `src/lib/art/town.ts`가 칸 채우기 함수로 만든 글자 그림(칸 5px). 집은 단계마다 더한다(창문·다락·꽃덤불·울타리·2층·발코니·탑·산울타리·금빛). 바닥은 칸 4px 캔버스에 한 번 굽고 늘려 그린다(부드럽게 하지 않음).
4. 시험: 캔버스 `data-feet`(발 위치), `data-ready`, 입구 안내 `[data-town-prompt]`(role=status), `bv-teleport` 이벤트.
5. 키보드 양보: 글자 칸과 버튼·링크 위의 Enter·Space는 게임에 주지 않는다. 창이 흐려지면 누른 키를 비운다.
6. `.btn`이 Tailwind 유틸리티보다 강해 휴대폰에서 숨길 링크는 `contents max-sm:hidden` 칸으로 감싼다.

## Project Structure

```text
src/lib/town/{layout,structures}.ts, src/lib/art/{town,characters}.ts
src/server/town.ts, src/app/town/page.tsx, src/components/town/{TownGame,PhoneTown,ground}.ts(x)
src/app/layout.tsx, src/components/house/HouseRoom.tsx, src/app/blog/[slug]/page.tsx
tests/unit/town.test.ts, tests/e2e/town.spec.ts
```
