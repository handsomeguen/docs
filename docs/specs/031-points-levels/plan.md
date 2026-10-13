# Implementation Plan: 경험치·레벨·코인과 보상

**Branch**: `031-points-levels` | **Date**: 2026-10-12 | **Spec**: [spec.md](./spec.md)

## Summary

`point_ledger`(마이그레이션 `0012_points_ledger`)에 한 줄씩 쌓고, 레벨·잔액은 합계로만 계산한다. 보상은 각 활동의 트랜잭션 안에서 `grantReward`로 넣는다.

## 결정

1. `src/lib/game.ts`: `expForLevel`·`levelFromExp`·`levelProgress`·`levelsGained`, `REWARD_RULES`, `POST_REWARD_MIN_LENGTH`, `postEarnsReward`, `REASON_LABEL`. 글쓰기 화면 안내와 서버가 같은 함수를 쓴다.
2. `grantReward(tx, 회원, 사유, ref)`(`src/server/points.ts`): `pg_advisory_xact_lock(hashtext('points:'+회원))` → 오늘(한국 0시 이후) 같은 사유 행 수 < 상한일 때만 INSERT. 각 트랜잭션의 맨 끝에서 부르고 다른 행을 잠그지 않아 교착이 생기지 않는다.
3. 지급하는 곳: `createMember`(signup), `savePost`(새 공개 글 100자 이상, ref 글 ID), `createComment`·`createReply`(남의 글, ref `comment:ID`/`reply:ID`), `toggleLike`(새 공감이고 남의 글, ref `글ID:공감한사람ID`가 없을 때).
4. 헤더는 `getWallet`으로 `Lv.N 🪙 N`. 보상이 생기는 Server Action은 모두 `revalidatePath("/", "layout")`.
5. 제약: CHECK 사유 4가지, 경험치 0 이상, 둘 다 0 금지. 부분 고유 인덱스 `point_ledger_like_received_uq`, `point_ledger_signup_uq`. 인덱스 `(user_id, reason, created_at)`, `(user_id, created_at desc, id desc)`.

## Project Structure

```text
drizzle/0012_points_ledger.sql, src/db/schema.ts
src/lib/game.ts, src/server/points.ts
src/server/{signup,post,comments,likes}.ts, src/app/write/actions.ts
src/app/layout.tsx, src/app/wallet/page.tsx
src/components/post/{PostForm,PublishNotice}.tsx, src/app/blog/[slug]/[postId]/page.tsx
tests/unit/game.test.ts, tests/e2e/rewards.spec.ts
```
