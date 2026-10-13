# Implementation Plan: 친구 마을 구경

**Branch**: `047-friend-town` | **Date**: 2026-10-12 | **Spec**: [spec.md](./spec.md)

## Summary

DB 변경 없음. `getTownHost(주소)`(공지 블로그·없는 주소면 null)와 `getMemberHouses(주인, 보는 사람)`(보는 사람의 집만 `mine`)로 `/town/[slug]`를 그린다. 화면 틀은 내 마을과 같은 `TownGame`·`TownHud`를 쓰고, `TownHud host`가 로고 자리에 마을 이름과 [🏠 내 마을로]를 띄운다. 휴대폰은 `PhoneTown host`가 그 회원의 블로그로 옮긴다.

## Project Structure

```text
src/server/town.ts, src/app/town/[slug]/page.tsx, src/app/blog/[slug]/page.tsx
src/components/town/{TownHud,TownGame,PhoneTown}.tsx
tests/e2e/friend-town.spec.ts
```
