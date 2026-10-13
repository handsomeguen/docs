# Data Model: 블로그 기본 기능

PostgreSQL 16. 모든 엔티티는 `id` 정수 PK(identity), 시각은 `timestamptz`. 인증 표 4개는 better-auth 형식(문자열 ID)을 따른다. 마이그레이션은 `drizzle/0000_init.sql` 하나로 시작한다.

## 관계 한눈에

```text
users 1 ─── 1 profiles
users 1 ─── 1 blogs 1 ─── N categories 1 ─── N subcategories
users 1 ─── N accounts            blogs 1 ─── N posts
users 1 ─── N sessions            posts N ─── 0..1 categories
login_attempts (FK 없음)           posts N ─── 0..1 subcategories
```

## 인증 (better-auth)

### users

| 컬럼 | 타입 | 제약 | 설명 |
|---|---|---|---|
| id | text | PK | better-auth가 만드는 ID (32자) |
| username | text | NOT NULL, UNIQUE, CHECK `username ~ '^[a-z0-9_]{4,20}$'` | 아이디 (소문자로 저장) |
| display_username | text | | 라이브러리 칸 (아이디와 같게 둔다) |
| email | text | NOT NULL, UNIQUE | `{아이디}@users.blogville.invalid` (라이브러리 필수 칸) |
| email_verified | boolean | NOT NULL default false | |
| name | text | NOT NULL | 라이브러리 칸 (아이디) |
| role | text | NOT NULL default `user`, CHECK `role in ('user','admin')` | `input: false` |
| created_at, updated_at | timestamptz | NOT NULL default now() | |

### accounts

| 컬럼 | 타입 | 제약 |
|---|---|---|
| id | text | PK |
| user_id | text | NOT NULL FK → users.id ON DELETE CASCADE |
| account_id | text | NOT NULL (아이디 로그인은 user_id와 같음) |
| provider_id | text | NOT NULL (`credential`) |
| password | text | 해시 (scrypt). 원문 저장 금지 |
| created_at, updated_at | timestamptz | |
| | | UNIQUE (provider_id, account_id) |

### sessions

| 컬럼 | 타입 | 제약 |
|---|---|---|
| id | text | PK |
| user_id | text | NOT NULL FK → users.id ON DELETE CASCADE |
| token | text | NOT NULL UNIQUE |
| expires_at | timestamptz | NOT NULL |
| remember_me | boolean | NOT NULL default false (추가 칸, `input: false`) |
| ip_address, user_agent | text | |
| created_at, updated_at | timestamptz | `updated_at` = 마지막 사용 (5분에 한 번 갱신) |

### verifications

better-auth 기본 표(id, identifier, value, expires_at, created_at, updated_at). 이번 기능에서는 쓰지 않지만 라이브러리가 요구한다.

### login_attempts

| 컬럼 | 타입 | 제약 | 설명 |
|---|---|---|---|
| username | text | PK | 정리한 아이디 (없는 아이디도 기록, FK 없음) |
| failures | integer | NOT NULL default 0, CHECK `failures >= 0` | 연속 실패 수 |
| locked_until | timestamptz | | 이 시각 전에는 로그인 거부 |
| updated_at | timestamptz | NOT NULL default now() | |

**규칙**: 로그인 시도 전에 failures + 1(행 잠금). 5가 되면 `locked_until = now() + interval '5 minutes'`, failures = 0. 성공하면 행 삭제. 가입하면 그 아이디의 행 삭제.

## 도메인

### profiles

| 컬럼 | 타입 | 제약 | 설명 |
|---|---|---|---|
| user_id | text | PK, FK → users.id ON DELETE CASCADE | 회원당 1개 |
| nickname | text | NOT NULL, UNIQUE, CHECK `char_length(btrim(nickname)) between 2 and 20` | 가입 때 = 아이디 |
| created_at, updated_at | timestamptz | | |

### blogs

| 컬럼 | 타입 | 제약 | 설명 |
|---|---|---|---|
| id | integer | PK identity | |
| owner_id | text | NOT NULL, UNIQUE (`blogs_owner_id_unique`), FK → users.id ON DELETE CASCADE | 회원당 블로그 1개 |
| slug | text | NOT NULL, UNIQUE, CHECK `slug ~ '^[a-z0-9_]{3,20}$'` | 블로그 주소 |
| title | text | NOT NULL, CHECK `char_length(title) between 1 and 40 and title = btrim(title)` (`blogs_title_check`) | 블로그 이름 |
| intro | text | NOT NULL default `''`, CHECK `char_length(intro) <= 160` | 소개 |
| created_at, updated_at | timestamptz | | |

### categories (대분류)

| 컬럼 | 타입 | 제약 |
|---|---|---|
| id | integer | PK identity |
| blog_id | integer | NOT NULL FK → blogs.id ON DELETE CASCADE |
| name | text | NOT NULL, CHECK `char_length(name) between 1 and 20` |
| position | integer | NOT NULL |
| created_at | timestamptz | |
| | | UNIQUE (blog_id, name), UNIQUE (id, blog_id) ← posts 복합 FK용 |

### subcategories (소분류)

| 컬럼 | 타입 | 제약 |
|---|---|---|
| id | integer | PK identity |
| category_id | integer | NOT NULL FK → categories.id ON DELETE CASCADE |
| name | text | NOT NULL, CHECK `char_length(name) between 1 and 20` |
| position | integer | NOT NULL |
| created_at | timestamptz | |
| | | UNIQUE (category_id, name), UNIQUE (id, category_id) ← posts 복합 FK용 |

### posts

| 컬럼 | 타입 | 제약 | 설명 |
|---|---|---|---|
| id | integer | PK identity | 1~2147483647 |
| blog_id | integer | NOT NULL FK → blogs.id ON DELETE CASCADE | |
| category_id | integer | NULL, FK (category_id, blog_id) → categories(id, blog_id) ON DELETE SET NULL (category_id) | 내 블로그 대분류만 |
| subcategory_id | integer | NULL, FK (subcategory_id, category_id) → subcategories(id, category_id) ON DELETE SET NULL (subcategory_id) | 그 대분류의 소분류만 |
| title | text | NOT NULL, CHECK `char_length(title) between 1 and 100` (`posts_title_check`) | |
| content_html | text | NOT NULL, CHECK `char_length(content_html) <= 200000` | 정화된 HTML |
| content_text | text | NOT NULL, CHECK `char_length(btrim(content_text)) >= 1` | 태그 뺀 글자 (요약) |
| is_public | boolean | NOT NULL default true | |
| created_at | timestamptz | NOT NULL default now() | 목록 정렬 기준 |
| updated_at | timestamptz | NOT NULL default now(), Drizzle `$onUpdate` | |
| | | CHECK `subcategory_id is null or category_id is not null` | 소분류만 있는 글 없음 |

**인덱스**: `posts (blog_id, created_at desc, id desc)` (블로그 홈 목록), `posts (category_id)`, `posts (subcategory_id)`.

**지울 때 동작** (BLOG-05):
- 소분류 삭제 → `posts.subcategory_id`만 NULL (글은 대분류에 남음).
- 대분류 삭제 → 소분류 CASCADE 삭제, `posts.category_id`·`subcategory_id` NULL. PostgreSQL 15+의 `ON DELETE SET NULL (컬럼)` 문법으로 복합 FK에서 한 칸만 비운다. 대분류 삭제 때 소분류 FK가 먼저 끊기므로 서비스 코드에서도 한 트랜잭션으로 `UPDATE posts SET category_id = NULL, subcategory_id = NULL WHERE category_id = $1` 뒤 삭제한다.

## 가입 트랜잭션이 만드는 행

| 순서 | 표 | 값 |
|---|---|---|
| 1 | users | username = 정리한 아이디, email = `{아이디}@users.blogville.invalid`, role = `user` |
| 2 | accounts | provider_id = `credential`, password = 해시 |
| 3 | login_attempts | 그 아이디 행 삭제 |
| 4 | profiles | nickname = 아이디 |
| 5 | blogs | slug = 아이디, title = 적은 블로그 이름(trim), intro = `''` |
| 6 | categories | name = `일상`, position = 1 |

## 예약어 (아이디·블로그 주소)

`admin`, `api`, `town`, `feed`, `shop`, `closet`, `write`, `settings`, `blog`, `onboarding`, `farm`, `attendance`, `tags`, `wallet`, `files`, `notice`, `notifications`, `fishing`, `house`, `salon`, `clothes` (명세 BLOG-02의 21개를 그대로 둔다. 나중 기능과 주소가 부딪치지 않게)

## 상태

글의 상태는 `is_public` 하나다 (공개 ↔ 비공개 자유롭게 전환). 임시 저장(POST-08)은 이번 범위 밖.
