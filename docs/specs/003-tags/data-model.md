# Data Model: 태그

| 표 | 컬럼 | 규칙 |
|---|---|---|
| tags | id integer identity PK, name text NOT NULL UNIQUE, created_at | 이름 길이 CHECK는 두지 않는다 (명세) |
| post_tags | post_id → posts.id CASCADE, tag_id → tags.id CASCADE | PK (post_id, tag_id), 인덱스 (tag_id) |

- 글 상세: `post_tags JOIN tags WHERE post_id ORDER BY name`.
- 태그별 글: 공개 글 중 `id IN (SELECT post_id FROM post_tags JOIN tags WHERE name = $1)`, `created_at DESC, id DESC`, 8개.
- 인기 태그: 공개 글과 JOIN, `GROUP BY tags.id`, `COUNT(*) DESC, name ASC`, 30개.
