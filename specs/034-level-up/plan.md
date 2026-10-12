# Implementation Plan: 레벨업 알림

**Branch**: `034-level-up` | **Date**: 2026-10-12 | **Spec**: [spec.md](./spec.md)

## Summary

마이그레이션 `0015_level_up`: `notifications.actor_id`·`post_id` NULL 허용, `level` 컬럼, 종류 CHECK에 `level_up`, 모양 CHECK(레벨업은 행동한 사람·글 없이 레벨 2~99, 나머지는 레벨 없이 둘 다 있음), 부분 고유 인덱스 `notifications_level_up_uq (user_id, level) WHERE kind = 'level_up'`.

## 결정

1. `addLedgerEntry(tx, entry)`: 경험치가 있으면 INSERT 전 누적 경험치를 읽고, 실제로 들어갔을 때만 `notifyLevelUps(tx, userId, 전, 후)`. 호출하는 쪽이 `lockUser`를 잡고 있어 누적이 어긋나지 않는다.
2. 목록·🔔 숫자·한 줄 열기는 글·블로그·닉네임을 LEFT JOIN하고, 볼 수 있는 조건에 `kind = 'level_up'`을 더한다.
3. `getPendingLevelUp`(안 읽은 가장 높은 레벨) → 레이아웃 헤더와 `/api/notifications/unread`(`levelUp`). `LevelUpDialog`는 처음 값과 주소 이동 때 받은 값 중 닫은 레벨보다 높은 것만 띄운다.
4. `dismissLevelUp`(Server Action): `parseLevel`(2~99 정수 글자)·`go`(`stay`/`wallet`) 확인 → `dismissLevelUps`(그 레벨 이하 안 읽은 레벨업 읽음) → `wallet`이면 `/wallet`.
5. 조사는 `withJosa`(`src/lib/josa.ts`): 한글 끝 글자 받침, 숫자는 읽는 소리(1·3·6·7·8·0, 끝이 0이면 십·백 → 받침).
6. E2E: 팝업이 화면을 막으므로 `tests/e2e/fixtures.ts`가 `addLocatorHandler`로 [확인]을 눌러 닫는다. 모든 spec이 여기서 `test`를 가져오고, `level-up.spec`만 `@playwright/test`를 그대로 쓴다.

## Project Structure

```text
drizzle/0015_level_up.sql, src/db/schema.ts
src/lib/{josa,notifications}.ts, src/server/{points,attendance,notifications}.ts
src/app/layout.tsx, src/app/api/notifications/unread/route.ts, src/app/notifications/{actions.ts,page.tsx}
src/components/game/LevelUpDialog.tsx
tests/unit/{josa,game}.test.ts, tests/e2e/level-up.spec.ts, tests/e2e/fixtures.ts
```
