# Implementation Plan: 출석 보상

**Branch**: `033-attendance` | **Date**: 2026-10-12 | **Spec**: [spec.md](./spec.md)

## Summary

마이그레이션 `0014_attendance`: `attendance_rewards`(day 1~7, 보상표 행), `attendances`(기본 키 `(user_id, date)`, `cycle_day` → 보상표, `session_id` → `sessions` SET NULL), 원장 사유에 `attendance`, 부분 고유 인덱스 `point_ledger_attendance_uq`.

## 결정

1. `nextCycleDay`(`src/lib/game.ts`)와 `monthCalendar`(`src/lib/kst.ts`)는 순수 함수로 단위 시험.
2. `ensureTodayAttendance`(`src/server/attendance.ts`): 트랜잭션 [회원 행 KEY SHARE(탈퇴와 같은 순서) → `lockUser` → 오늘 행 있으면 읽기만 → 오늘 이전 마지막 출석 → 일차 → INSERT(세션은 행이 있을 때만) → 보상표 → 원장 INSERT ON CONFLICT DO NOTHING].
3. 레이아웃 헤더가 wallet을 읽기 전에 부르고, 실패는 `console.error`로만 남긴다. 출석 화면은 한 번 더 부르고 실패하면 오류 화면.
4. `AttendanceDayWatcher`(헤더 안 클라이언트): 주소가 바뀌거나 탭이 다시 보일 때 `todayKST()`가 바뀌었으면 `router.refresh()`.

## Project Structure

```text
drizzle/0014_attendance.sql, src/db/schema.ts
src/lib/{game,kst}.ts, src/server/{attendance,points}.ts
src/app/layout.tsx, src/app/attendance/page.tsx, src/components/game/AttendanceDayWatcher.tsx
tests/unit/game.test.ts, tests/e2e/attendance.spec.ts, tests/e2e/rewards.spec.ts(헤더 숫자에 출석 반영)
```
