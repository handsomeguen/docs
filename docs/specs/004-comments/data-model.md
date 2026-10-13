# Data Model: 댓글과 답글

| 표 | 컬럼 | 규칙 |
|---|---|---|
| comments | id identity PK, post_id → posts CASCADE, author_id → users SET NULL, content text, deleted_at, created_at | CHECK: deleted_at IS NULL → 1~1000자, 아니면 '' · 인덱스 (post_id, created_at) |
| replies | id identity PK, comment_id → comments CASCADE, author_id → users CASCADE, content, deleted_at, created_at | 같은 CHECK · 인덱스 (comment_id, created_at) |

- `💬 N` = 그 글의 `deleted_at IS NULL` 댓글 수 + 그 댓글들의 `deleted_at IS NULL` 답글 수.
