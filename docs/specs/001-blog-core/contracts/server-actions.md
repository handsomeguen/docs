# Contract: Server Actions

모든 쓰기는 Server Action이다. 공통 반환 형식:

```ts
type ActionState = { ok: true; message?: string } | { ok: false; error: string; values?: Record<string, string> };
```

- 회원 전용 액션은 첫 줄에서 `requireMember()` (비로그인 → `redirect('/')`).
- FormData에서 정해진 칸만 읽는다. `ownerId`·`blogId`·`role` 같은 칸은 무시하고 주인은 세션에서 정한다.
- 오류는 아래 표의 순서대로 처음 걸린 하나만 돌려준다. 문구는 명세 그대로.
- 성공 뒤 `revalidatePath`, 필요하면 `redirect`.

---

## signUp (`src/app/(auth)/actions.ts`)

| 입력 | 규칙 |
|---|---|
| username | trim·소문자 후 `^[a-z0-9_]{4,20}$` |
| password | 8~64자 |
| passwordConfirm | password와 같음 |
| blogTitle | trim 후 1~40자 (코드 포인트) |

| 순서 | 상황 | error |
|---|---|---|
| 1 | 아이디 형식 | 아이디는 영문 소문자, 숫자, _ 로 4~20자예요 |
| 2 | 비밀번호 8자 미만 | 비밀번호는 8자 이상이에요 |
| 3 | 비밀번호 64자 초과 | 비밀번호는 64자까지예요 |
| 4 | 확인 불일치 | 비밀번호가 서로 달라요 |
| 5 | 블로그 이름 빈 값 | 블로그 이름을 적어 주세요 |
| 6 | 블로그 이름 40자 초과 | 블로그 이름은 40자까지예요 |
| 7 | 예약어 | 이 아이디는 쓸 수 없어요 |
| 8 | 아이디·주소·닉네임 겹침, UNIQUE 위반 | 이미 있는 아이디예요 |

- 오류면 `values`에 username·blogTitle을 돌려준다 (비밀번호는 비움).
- 이미 로그인했으면 `redirect('/@내주소')`.
- 성공: `createMember` 트랜잭션 → `signInUsername({ rememberMe: false })` → `redirect('/@{아이디}?welcome=1')`.

## signIn

| 입력 | 규칙 |
|---|---|
| username | trim·소문자 |
| password | |
| rememberMe | `on`이면 true |

| 상황 | error |
|---|---|
| 빈 칸 | 아이디와 비밀번호를 적어 주세요 |
| 잠금 중 (5회 실패 후 5분) | 로그인을 너무 많이 시도했어요. 5분 뒤에 다시 시도해 주세요 |
| 없는 아이디·틀린 비밀번호·64자 초과 | 아이디 또는 비밀번호가 맞지 않아요 |

- 오류면 `values`에 username·rememberMe를 돌려준다.
- 성공: `clearLoginAttempts` → `redirect('/@내주소')`.

## signOut

- 입력 없음. `auth.api.signOut` (세션 행·쿠키 삭제) → `redirect('/')`. 다른 Origin이면 Next.js가 거부.

---

## savePost (`src/app/write/actions.ts`)

| 입력 | 규칙 |
|---|---|
| postId | 빈 값(새 글) 또는 `parseId` 정수 |
| title | trim 후 1~100자 |
| contentHtml | 최대 200,000자 (보낼 때 UTF-8 900,000바이트까지) |
| categoryId | 빈 값 또는 `parseId` 정수 |
| subcategoryId | 빈 값 또는 `parseId` 정수 |
| visibility | `public`(기본, 값 없으면) / `private` |

| 순서 | 상황 | error |
|---|---|---|
| 1 | postId 형식 | 잘못된 요청이에요 |
| 2 | 제목 빈 값 | 제목을 적어 주세요 |
| 3 | 제목 100자 초과 | 제목은 100자까지예요 |
| 4 | 본문 HTML 20만 자 초과 | 글이 너무 길어요 |
| 5 | 카테고리 ID 형식 | 잘못된 요청이에요 |
| 6 | visibility 값 | 잘못된 요청이에요 |
| 7 | 정화한 본문 글자 없음 | 본문을 적어 주세요 |
| 8 | 소분류가 고른 대분류 소속이 아님, 대분류 없이 소분류만 | 잘못된 요청이에요 |

- `resolveCategory`: 대분류가 내 블로그 것이 아니거나 없으면 → 둘 다 NULL로 저장(오류 아님). 소분류가 지워졌으면 → 대분류만 저장.
- 수정인데 남의 글·없는 글: 저장하지 않고 예외 → 오류 화면.
- 성공: `revalidatePath('/@주소')`, `redirect('/@주소/글ID')`.

## deletePost

| 입력 | 규칙 |
|---|---|
| postId | `parseId` 정수 (아니면 아무것도 안 함) |

- 내 글이면 삭제. 남의 글·없는 글이면 아무것도 지우지 않는다. 둘 다 `redirect('/@내주소')`.

---

## updateBlogInfo (`src/app/settings/blog/actions.ts`)

| 입력 | 규칙 | error |
|---|---|---|
| title | trim 후 1~40자 | 블로그 이름을 적어 주세요 / 블로그 이름은 40자까지예요 |
| intro | trim 후 0~160자 | 소개는 160자까지예요 |

- 성공: `{ ok: true, message: '저장했어요 ✓' }`.

## updateBlogSlug

| 순서 | 상황 | error |
|---|---|---|
| 1 | 형식 (trim·소문자 후 `^[a-z0-9_]{3,20}$`) | 주소는 영문 소문자, 숫자, _ 로 3~20자예요 |
| 2 | 예약어 | 이 주소는 쓸 수 없어요 |
| 3 | 다른 회원의 아이디·주소와 겹침 (이름 잠금 뒤 확인, UNIQUE 위반 포함) | 이미 있는 주소예요 |
| – | 지금 주소와 같음 | (바꾸지 않고) 저장했어요 ✓ |

- 성공: 예전·새 주소 모두 revalidate, `저장했어요 ✓`.

## updateNickname (`src/app/settings/account/actions.ts`)

| 순서 | 상황 | error |
|---|---|---|
| 1 | trim 후 2자 미만 | 닉네임은 2자 이상이에요 |
| 2 | 20자 초과 | 닉네임은 20자까지예요 |
| 3 | 다른 회원의 아이디(대소문자 무시)·닉네임(글자 그대로)과 겹침 | 이미 있는 닉네임이에요 |

---

## 카테고리 (`src/app/settings/blog/actions.ts`)

모든 카테고리 액션은 `lockBlog(내 블로그)` 뒤에 실행한다. 대상 ID가 내 블로그 것이 아니거나 형식이 틀리면 **아무것도 바꾸지 않고 문구도 없다** (renameCategory만 `잘못된 요청이에요`).

| 액션 | 입력 | 동작 | error |
|---|---|---|---|
| addCategory | name | 대분류를 position = max+1로 추가 | 카테고리 이름을 적어 주세요 / 카테고리 이름은 20자까지예요 / 이미 있는 카테고리예요 |
| addSubcategory | categoryId, name | 그 대분류 맨 아래에 소분류 추가 | 위와 같음 |
| renameCategory | kind(`category`/`sub`), id, name | 이름 바꾸기 | 위와 같음 + 잘못된 요청이에요 |
| moveCategory | kind, id, direction(`-1`/`1`) | 같은 부모 안의 이웃과 position 맞바꿈. 맨 끝이면 그대로 | 없음 |
| deleteCategory | kind, id | 소분류: 그 글의 subcategory_id NULL 후 삭제. 대분류: 그 글의 category_id·subcategory_id NULL, 소분류 CASCADE 삭제 | 없음 |

- 확인 창 문구 (화면): 대분류 `'{이름}' 카테고리를 지울까요? 글은 남고 '카테고리 없음'이 돼요.` / 소분류 `'{이름}' 소분류를 지울까요? 글은 '{대분류 이름}'에 남아요.`
- 성공하면 `revalidatePath('/settings/blog')`, `revalidatePath('/@주소')`.
