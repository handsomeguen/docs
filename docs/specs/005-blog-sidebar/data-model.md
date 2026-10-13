# Data Model: 블로그 옆 칸·조회수·방문자

| 표 | 컬럼 | 규칙 |
|---|---|---|
| posts (추가) | view_count integer NOT NULL DEFAULT 0 | CHECK `view_count >= 0` · 인덱스 (blog_id, view_count DESC) — 인기 글 |
| post_views | post_id → posts CASCADE, date (한국 날짜), visitor_id uuid, created_at | PK (post_id, date, visitor_id) |
| blog_visits | blog_id → blogs CASCADE, date, visitor_id uuid | PK (blog_id, date, visitor_id) |

- `view_count`는 `post_views`에서 셀 수 있는 합계를 일부러 저장한 값이다. 오래된 `post_views`는 정리 작업이 지워도 합계는 남는다.
- 방문자 오늘·어제·전체는 `blog_visits`를 `WHERE blog_id` 한 번으로 센다 (`count(*) filter`).
- 최근 댓글은 댓글과 답글을 `UNION ALL`로 합쳐 지운 것을 빼고 최근 5개.
