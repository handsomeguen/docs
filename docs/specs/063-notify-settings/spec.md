# Feature Specification: 이웃 새 글 알림 · 알림 설정

**Feature Branch**: `feature/notify-settings` (코드 PR #62, v1.5.0) · **Created**: 2026-10-13 · **Status**: Implemented

## User Stories
1. 이웃으로 추가한 회원이 공개 글을 새로 올리면 `📝 {닉네임}님이 새 글을 올렸어요: {제목}`(예약 글은 공개되는 시각에). 수정·비공개는 알리지 않음. 한 글에 한 번, 이웃 1000명까지.
2. 내 정보 `🔔 알림 설정`: 공감·댓글·답글·방명록·새 글·레벨업을 켜고 끈다. 문의 답변 알림은 늘 켜짐.

## Success Criteria
- E2E `notify-settings.spec`, 단위 `notify-settings`.
