# Implementation Plan: 알림함(🔔)

**Branch**: `017-notifications` | **Date**: 2026-10-12 | **Spec**: [spec.md](./spec.md)

## Summary

새 표 `notifications(id, user_id 받는 사람, kind, actor_id, post_id, created_at, read_at)` (마이그레이션 `0009_notifications`). 공감·댓글·답글을 저장하는 트랜잭션 안에서 `notifyActivity()`가 한 줄 넣는다. 알림함은 서버 컴포넌트 `/notifications`, 읽음 처리는 Server Action, 헤더 🔔은 클라이언트 컴포넌트 `NotificationBell`.

## 결정

1. CHECK `notifications_kind_check`(like·comment·reply), `notifications_not_self_check`(받는 사람 ≠ 행동한 사람). 앱도 자기 자신이면 넣지 않는다.
2. 공감 한 번만: 부분 고유 인덱스 `notifications_like_uq (user_id, actor_id, post_id) WHERE kind = 'like'` + `ON CONFLICT DO NOTHING`. `toggleLike`는 공감 행이 실제로 들어갔을 때만(`returning`) 알림을 넣는다.
3. 받는 사람: 공감·댓글은 `lockVisiblePost`가 돌려주는 글 주인, 답글은 잠근 원댓글의 작성자.
4. 닉네임·글 제목·블로그 주소는 저장하지 않고 보여 줄 때 JOIN한다. 받는 사람이 지금 볼 수 있는 글(공개 글이거나 내 글)의 알림만 센다·보인다·연다.
5. 헤더 숫자는 10개에서 멈춰 센다(`LIMIT 10` 부분 쿼리, `9+`). 레이아웃은 화면을 옮겨도 다시 그려지지 않으므로 `NotificationBell`이 주소가 바뀔 때 `GET /api/notifications/unread`(`no-store`)로 새로 받는다. 서버가 헤더를 다시 그리면(Server Action 뒤) 숫자를 `{ count }` 새 객체로 넘겨, 값이 전과 같아도 서버 값으로 맞추고 받는 중이던 옛 숫자는 버린다.
6. 읽음 처리는 모두 `user_id = 나` 조건. 남의 알림 ID는 아무것도 바꾸지 않고 `/notifications`로 돌아간다.
7. 지움: `user_id`·`actor_id`·`post_id` 모두 `ON DELETE CASCADE` (탈퇴는 `users` 행을 지운다).
8. 잠금 순서: 알림을 넣으면 받는 회원 행에 외래 키 잠금이 생긴다. 탈퇴(`deleteMember`)는 회원 행 → 글·댓글 순서로 잠그므로, 댓글·답글·공감도 받는 회원 행(글 주인, 원댓글 작성자)을 글·원댓글보다 먼저 `FOR KEY SHARE`로 잠근다(`lockMemberRow`). 거꾸로 잠그면 공감·댓글과 받는 사람의 탈퇴가 겹칠 때 교착 상태가 된다.
9. 안 읽은 줄은 노란 배경 말고도 읽는 화면용 `안 읽음,`을 버튼 이름 앞에 둔다. 마지막보다 큰 쪽은 빈 문구(POST-05).

## Project Structure

```text
src/db/schema.ts, drizzle/0009_notifications.sql
src/lib/notifications.ts (문구·주소·시간·숫자)
src/server/notifications.ts (notifyActivity, countUnreadNotifications, listNotifications, readNotification, markAllNotificationsRead)
src/server/comments.ts, src/server/likes.ts (알림 넣기)
src/app/notifications/{page,actions}.ts(x), src/app/api/notifications/unread/route.ts
src/components/notifications/NotificationBell.tsx, src/app/layout.tsx (헤더)
tests/unit/notifications.test.ts, tests/e2e/notifications.spec.ts
```
