# Data Model: 사진·파일 첨부

## attachments (새 표)

| 컬럼 | 타입 | 규칙 |
|---|---|---|
| key | text PK | `^[0-9a-f]{32}$` (서버가 만든 무작위 이름, 저장소 파일 이름과 같다) |
| user_id | text NOT NULL | → users.id ON DELETE CASCADE (올린 회원) |
| kind | text NOT NULL | `image` / `file` |
| name | text NOT NULL | 원래 이름, 1~255자 |
| mime | text NOT NULL | 서버 표의 형식 |
| size | integer NOT NULL | > 0 바이트 |
| post_id | integer NULL | → posts.id ON DELETE SET NULL. NULL = 어느 글에도 붙지 않음 |
| detached_at | timestamptz NULL | 글에서 떨어진 시각 (트리거가 관리) |
| created_at | timestamptz NOT NULL | 기본 now() |

- 인덱스: `(user_id, created_at)`, `(post_id)`, 정리용 `(created_at) WHERE post_id IS NULL`.
- 트리거 `attachments_track_detached` (BEFORE UPDATE OF post_id): 값→NULL이면 `detached_at = now()`, NULL→값이면 `detached_at = NULL`.

## 상태

```text
올림 (post_id NULL, detached_at NULL) ──저장──▶ 붙음 (post_id = 글)
붙음 ──본문에서 빠짐·글 삭제──▶ 떨어짐 (post_id NULL, detached_at = 시각) ──다시 저장──▶ 붙음
올림/떨어짐 ──하루 지남 + 정리 작업──▶ 행·파일 삭제
```

## 누가 여나 (attachmentAccess)

| 첨부 상태 | 열 수 있는 사람 |
|---|---|
| 공개 글에 붙음 | 누구나 |
| 비공개 글에 붙음 | 그 블로그 주인 |
| 붙지 않음 | 올린 회원 |

## 붙일 수 있는 첨부 (lockLinkable)

① 행이 있음 ② `user_id` = 나 ③ `post_id`가 NULL 또는 이 글 (새 글이면 NULL만). 본문 키를 정렬해 `FOR UPDATE`로 잠근다.
