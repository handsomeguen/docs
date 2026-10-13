# Implementation Plan: 마을 ☰ 메뉴

**Branch**: `039-town-menu` | **Date**: 2026-10-12 | **Spec**: [spec.md](./spec.md)

## Summary

DB 변경 없음. `/town`이 `getHudMember`(지갑·안 읽은 수·최근 알림·이웃 목록)를 읽어 `TownHud`에 넘긴다. 로그인 액션과 `/`의 회원 이동을 `/town`으로 바꾼다.

## 결정

1. 마을에는 헤더 막대가 없다: `body:has([data-town]) > header { display: none }`(640px 이상). 레벨업 팝업·자동 출석 감시·로그인 유지는 헤더 밖(형제)으로 옮겨 숨긴 헤더와 상관없이 동작한다. 마을 페이지는 헤더가 없으니 자동 출석을 먼저 해서 ☰ 코인에 들어가게 한다.
2. `TownHud`의 창은 `<dialog open>`(모달 아님): 게임의 `yieldsKey`가 `dialog[open]`을 보고 키보드를 놓는다. Esc는 창이 직접 받는다.
3. 텔레포트는 캔버스에 `bv-teleport` 이벤트(시험 훅과 같은 길). 목록은 `townSpots(houses, member)`(게임과 같은 입구 좌표, 집이면 블로그 주소).
4. ⭐를 바꾸면 `revalidatePath`로 집 목록이 다시 와 게임을 다시 만든다. 시작 자리가 같으면 마지막 발 위치를 이어 쓴다(`lastPos`).
5. 첫 마을 환영은 `TownWelcome`이 처음 그릴 때 정한 상태로 닫을 때까지 남고, 주소의 `welcome`은 `router.replace`로 지운다(`history.replaceState`만 쓰면 ⭐ 같은 서버 액션 뒤 Next가 옛 주소를 되살린다). 게임은 `startKey`(`?at` 값)가 같으면 다시 만들어도 서 있던 자리를 이어 쓴다.
6. 마을에는 헤더가 없어 ☰ 메뉴에 바로 가기(글쓰기·마을 소식·상점·출석) 링크를 둔다. 방문자 메뉴에도 마을 소식. 메뉴의 Esc는 레벨업 같은 모달이 열려 있으면 양보한다.

## Project Structure

```text
src/app/(auth)/actions.ts, src/app/page.tsx, src/app/layout.tsx, src/app/globals.css
src/app/town/page.tsx, src/components/town/{TownHud,TownWelcome,TownGame}.tsx, src/lib/town/structures.ts, src/server/town.ts
src/app/blog/[slug]/page.tsx, src/components/house/HouseRoom.tsx
tests/unit/town.test.ts, tests/e2e/{town-menu,town,auth,login-next,account-settings,admin-reset,drafts,session-keep}.spec.ts
```
