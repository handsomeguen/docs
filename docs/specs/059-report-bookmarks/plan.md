# Plan: 신고하기 · 스크랩
- 마이그레이션 `0034_report_bookmarks`: `reports`(대상 종류·사유·상태 CHECK, 중복 unique, 대상 id는 FK 없음), `bookmarks`(PK 회원+글).
- `src/server/reports.ts`, `src/app/blog/report-actions.ts`, `src/app/admin/reports-actions.ts`, `/admin/reports`, `/bookmarks`.
