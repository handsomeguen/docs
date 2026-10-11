---

description: "블로그 기본 기능 구현 태스크"
---

# Tasks: 블로그 기본 기능 (가입·블로그·글·카테고리)

**Input**: `/specs/001-blog-core/`의 설계 문서

**Prerequisites**: plan.md, spec.md, research.md, data-model.md, contracts/

**Tests**: constitution VI(수용 기준은 실행해서 확인)에 따라 이야기마다 E2E, 순수 함수는 Vitest를 포함한다. 테스트를 먼저 쓰고 실패를 확인한 뒤 구현한다.

**Organization**: 사용자 이야기별로 묶었다. 각 이야기는 혼자 구현·시험·시연할 수 있다.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: 다른 파일이고 의존이 없어 함께 진행 가능
- **[Story]**: US1~US5 (spec.md의 사용자 이야기)

---

## Phase 1: Setup (공통 기반)

- [x] T001 Next.js 16 + TypeScript(strict) + Tailwind 4 프로젝트 생성, `src/` 구조를 plan.md대로 만든다 (`package.json`, `tsconfig.json`)
- [x] T002 [P] ESLint·Prettier 설정, `npm run typecheck`·`lint` 스크립트 추가
- [x] T003 [P] `docker-compose.yml`(postgres:16), `.env.example`(DATABASE_URL, DATABASE_URL_TEST, BETTER_AUTH_SECRET, BETTER_AUTH_URL), `.gitignore`에 `.env*` (NF-01)
- [x] T004 [P] Vitest 설정(`vitest.config.ts`, `tests/unit/`), Playwright 설정(`playwright.config.ts`, 데스크톱 + 375×812 휴대폰 프로젝트, `tests/e2e/`)
- [x] T005 [P] `next.config.ts`에 rewrites `/@:slug` → `/blog/:slug`, `/@:slug/:postId` → `/blog/:slug/:postId`

---

## Phase 2: Foundational (모든 이야기의 전제)

**⚠️ 이 단계가 끝나야 이야기 작업을 시작한다**

- [x] T006 Drizzle 연결 `src/db/index.ts`(postgres.js), `drizzle.config.ts`, `npm run db:generate`·`db:migrate` 스크립트
- [x] T007 `src/db/schema.ts`에 data-model.md의 10개 표(users, accounts, sessions, verifications, login_attempts, profiles, blogs, categories, subcategories, posts)와 CHECK·UNIQUE·복합 FK·인덱스 작성 → `drizzle/0000_init.sql` 생성 (필요한 `ON DELETE SET NULL (컬럼)`은 SQL 직접 보정)
- [x] T008 [P] `src/lib/ids.ts`: `parseId`(숫자만, 1~2147483647, `012`·`1e1` 거부), `parsePage` + `tests/unit/ids.test.ts`
- [x] T009 [P] `src/lib/names.ts`: `normalizeName`, `USERNAME_RE`, `SLUG_RE`, `RESERVED_NAMES`(21개) + `tests/unit/names.test.ts`
- [x] T010 [P] `src/lib/text.ts`: `codePointLength`, `excerpt(120자)` + 단위 테스트
- [x] T011 `src/lib/auth.ts`: better-auth 설정(drizzle adapter, username 플러그인, nextCookies, `role`·`remember_me` 추가 칸 `input: false`, 세션 생성 훅 2시간/7일)
- [x] T012 `src/app/api/auth/[...all]/route.ts`: `GET /api/auth/get-session`만 통과, 나머지 404 (NF-12)
- [x] T013 `src/server/dal.ts`: `getSession()`(유지 안 함 세션 2시간 경과 무효, 5분마다 `updated_at` 갱신), `requireMember()`, `getMyBlog()` — `import 'server-only'`
- [x] T014 [P] `src/app/layout.tsx`(헤더: 로고·내 블로그·[✏️ 글쓰기]·로그아웃 자리, 한국어 `lang="ko"`), `src/app/not-found.tsx`(`길을 잃었어요`), `src/app/error.tsx`(`잠깐 문제가 생겼어요` + [다시 시도])
- [x] T015 [P] `tests/e2e/helpers.ts`: 테스트 DB 초기화, 고유 아이디로 회원 만들기·로그인 도우미

**Checkpoint**: DB·인증 설정·공용 규칙 준비 완료

---

## Phase 3: User Story 1 - 가입하면 바로 내 블로그가 생긴다 (P1) 🎯 MVP

**Goal**: 아이디·비밀번호·블로그 이름으로 가입하면 블로그가 함께 생기고, 로그인·로그아웃이 된다 (AUTH-07·09·04, BLOG-01, AUTH-03, NF-10)

**Independent Test**: 빈 DB에서 가입 → `/@아이디?welcome=1`, 로그아웃 → 로그인 → 내 블로그

### Tests for User Story 1

- [x] T016 [P] [US1] `tests/unit/blog-rules.test.ts`: `checkBlogTitle` 공백·40/41자·이모지 1자
- [x] T017 [P] [US1] `tests/e2e/signup.spec.ts`: 시나리오 1~5 (자동 채움, 대소문자 겹침, 주소·닉네임 겹침, 형식·예약어 문구와 입력 유지, 동시 가입 1명, 실패 시 아무것도 안 남음, `role=admin` 무시, 비밀번호 해시, 키보드만으로 가입)
- [x] T018 [P] [US1] `tests/e2e/session.spec.ts`: 시나리오 6~9 (로그인 문구, 대소문자·공백 무시, 2시간/7일 쿠키, 로그아웃 후 `/write` → `/`, 다른 Origin 로그아웃 거부, `/api/auth/sign-up/email`·`sign-in/username`·`sign-out` 404)
- [x] T019 [P] [US1] `tests/e2e/login-limit.spec.ts`: 5회 실패 → 6번째 잠금 문구, 시각 조작으로 5분 뒤 풀림

### Implementation for User Story 1

- [x] T020 [P] [US1] `src/lib/blog-rules.ts`: `checkBlogTitle`, `checkBlogIntro`, `checkNickname` (문구 상수 포함)
- [x] T021 [P] [US1] `src/lib/auth-rules.ts`: 가입·로그인 zod 스키마와 검사 순서·문구 (contracts/server-actions.md signUp·signIn 표)
- [x] T022 [US1] `src/server/names.ts`: `lockName`(advisory xact lock), `findNameConflict`(username·slug·nickname)
- [x] T023 [US1] `src/server/signup.ts`: `createMember` 트랜잭션 (해시는 밖에서, users → accounts → login_attempts 삭제 → profiles → blogs → categories "일상", UNIQUE 위반을 `이미 있는 아이디예요`로)
- [x] T024 [US1] `src/server/login-attempts.ts`: `reserveLoginAttempt`(행 잠금, 5회 → 5분), `clearLoginAttempts`
- [x] T025 [US1] `src/app/(auth)/actions.ts`: `signUp`, `signIn`, `signOut` (contracts 그대로, 성공 시 `/@아이디?welcome=1` 또는 `/@내주소`)
- [x] T026 [P] [US1] `src/components/auth/SignUpForm.tsx`: 블로그 이름 자동 채움(고치면 멈춤, 비우면 다시 따라감), `useActionState`, `가입하는 중...`, 오류 한 줄(`#c62828`), 비밀번호만 비움
- [x] T027 [P] [US1] `src/components/auth/LoginForm.tsx`(로그인 상태 유지, `들어가는 중...`), `SignOutButton.tsx`(44×44px)
- [x] T028 [US1] `src/app/page.tsx`: 서비스 소개 + [로그인]/[회원가입] 탭(처음엔 로그인), 로그인했으면 내 블로그로 redirect
- [x] T029 [US1] `src/app/blog/[slug]/page.tsx` 최소판: 블로그 이름·소개·닉네임 머리, 주인에게 `?welcome=1` 환영 카드(`WelcomeCard`), 빈 목록 문구. 없는 slug → `notFound()`

**Checkpoint**: 가입·로그인·로그아웃과 빈 블로그가 동작

---

## Phase 4: User Story 2 - 글을 쓰고 발행해서 읽는다 (P1)

**Goal**: Tiptap으로 글을 쓰고 공개/비공개로 발행, 상세 보기·수정·삭제 (POST-01·02, BLOG-02 상세, NF-03)

**Independent Test**: 발행 → 상세 → 수정 → 삭제, 비공개 글은 남에게 404

### Tests for User Story 2

- [x] T030 [P] [US2] `tests/unit/sanitize.test.ts`: `<script>`·`onerror`·`onclick`·`javascript:`·`<iframe>`·`<style>` 제거, 링크 `target`·`rel`, 허용 태그 유지, 정화 뒤 빈 글자 판정
- [x] T031 [P] [US2] `tests/unit/post-rules.test.ts`: 검사 순서 ①~⑧과 문구, visibility 기본 public, 20만 자·900,000바이트 경계
- [x] T032 [P] [US2] `tests/e2e/post.spec.ts`: 시나리오 1~9 (입력 규칙 변환, 오류 뒤 입력 유지, 수정 시각, 삭제 확인, 비공개 404·🔒, 남의 글 수정 화면 404·조작 요청 무효, 비로그인 `/write` → `/`, 상세 주소 이상한 ID 404)

### Implementation for User Story 2

- [x] T033 [P] [US2] `src/lib/sanitize.ts`: `sanitizePostHtml`(research §8 허용 목록), `htmlToText`
- [x] T034 [P] [US2] `src/lib/post-rules.ts`: `postInputSchema`, `checkPostInput`, `POST_ERRORS` (브라우저·서버 공용)
- [x] T035 [US2] `src/server/post.ts`: `createPost`, `updatePost`(내 글만, 아니면 throw), `deletePost`(내 글만), `getPostForView(slug, id, viewer)`(비공개는 주인만), `getPostForEdit(id, me)`
- [x] T036 [US2] `src/app/write/actions.ts`: `savePost`, `deletePost` (카테고리는 이 단계에서 항상 NULL, US4에서 `resolveCategory` 연결)
- [x] T037 [P] [US2] `src/components/editor/RichEditor.tsx` + `Toolbar.tsx`: StarterKit(H2·H3), Link, Placeholder `오늘 배운 것, 생각한 것, 무엇이든 적어 보세요 ✏️`, 버튼(제목·굵게·기울임·목록·번호·인용·코드 블록·링크·구분선), 44px 버튼
- [x] T038 [US2] `src/components/post/PostForm.tsx`: 제목·에디터·[🌍 공개][🔒 비공개]·[발행하기]/[수정 완료], 상태는 `useState`, `startTransition(() => action(fd))`로 보내 오류 때 값 유지, 900,000바이트 사전 검사
- [x] T039 [US2] `src/app/write/page.tsx`, `src/app/write/[postId]/page.tsx`: `requireMember()`, 수정은 `parseId` → 내 글 아니면 `notFound()`
- [x] T040 [US2] `src/app/blog/[slug]/[postId]/page.tsx`: 제목·닉네임·작성일·`· 수정됨`·🔒·본문(`PostBody`, 정화된 HTML), 주인에게 [수정] [삭제](`DeletePostButton`, `confirm`), [목록으로], `generateMetadata`
- [x] T041 [US2] 헤더·블로그 홈 주인 버튼 [✏️ 새 글 쓰기] → `/write`

**Checkpoint**: US1 + US2 = 쓸 수 있는 가장 작은 블로그 (MVP 시연 가능)

---

## Phase 5: User Story 3 - 블로그 홈에서 글 목록을 넘겨 본다 (P2)

**Goal**: 최신순 8개 카드, 페이지 번호, 주인/방문자별 공개 범위 (POST-05, BLOG-02)

**Independent Test**: 시드 9개 → 1페이지 8개·2페이지 1개, 이상한 `?page`는 1페이지

### Tests for User Story 3

- [x] T042 [P] [US3] `tests/e2e/list.spec.ts`: 시나리오 1~3·5·6 (8개·페이지 번호, `?page` 이상값, 빈 화면과 [✏️ 첫 글 쓰기], 남에게 비공개 제외, 404 경우들)

### Implementation for User Story 3

- [x] T043 [US3] `src/server/post.ts`에 `listBlogPosts(blogId, { viewerIsOwner, categoryId?, subId?, page })` → `{ items, total }` (`created_at desc, id desc`, LIMIT 8 OFFSET)
- [x] T044 [P] [US3] `src/components/blog/PostCard.tsx`(배지 자리·제목·요약·날짜·🔒, 카드 전체 링크), `Pagination.tsx`(`?page=N`, 현재 페이지 표시, 다른 쿼리 유지)
- [x] T045 [US3] `src/app/blog/[slug]/page.tsx` 완성: `parsePage`, 목록·페이지·빈 화면 문구, `generateMetadata`
- [x] T046 [P] [US3] `scripts/seed.ts` + `npm run db:seed`: 회원 `demo`, 글 20개(공개 18·비공개 2)
- [x] T047 [US3] 글 1,000개 시드로 블로그 홈 응답 시간 측정 스크립트 `tests/perf/blog-scale.ts` (SC-003, NF-07)

**Checkpoint**: 블로그를 둘러볼 수 있다

---

## Phase 6: User Story 4 - 카테고리를 2단계로 만들고 글에 붙인다 (P2)

**Goal**: 대분류·소분류 관리와 글 카테고리 지정·배지·필터 (BLOG-05, POST-03)

**Independent Test**: `여행 › 맛집` 글 → 소분류 삭제 시 `여행`, 대분류 삭제 시 배지 없음·글 유지

### Tests for User Story 4

- [x] T048 [P] [US4] `tests/unit/category-rules.test.ts`: 이름 빈 값·20/21자
- [x] T049 [P] [US4] `tests/e2e/category.spec.ts`: 시나리오 1~5 (추가·소분류·문구·입력 유지·▲▼·비활성·이름 바꾸기 반영·삭제 규칙·취소, 남의 ID 조작 무효, 동시 순서 바꾸기)
- [x] T050 [P] [US4] `tests/e2e/post-category.spec.ts`: 시나리오 6~7, US3 시나리오 4 (소분류 칸 켜짐·풀림, 조작한 소분류 `잘못된 요청이에요`, 남의 대분류 → 카테고리 없음, 왼쪽 목록 필터·글 수)

### Implementation for User Story 4

- [x] T051 [P] [US4] `src/lib/category-rules.ts`: `checkCategoryName`, 문구 상수
- [x] T052 [US4] `src/server/blog.ts`에 `lockBlog`, `src/server/category.ts`: add/addSub/rename/move/delete, `getCategoryTree(blogId, viewerIsOwner)`(글 수 포함)
- [x] T053 [US4] `src/app/settings/blog/actions.ts`: 카테고리 액션 5개 (contracts 표대로)
- [x] T054 [US4] `src/components/settings/CategoryManager.tsx`: 대분류·들여 쓴 소분류, [소분류 추가]·[이름 바꾸기]·▲▼·[삭제](확인 문구), Enter로 추가, 오류 위치
- [x] T055 [US4] `src/app/settings/blog/page.tsx`: `requireMember()`, 카테고리 카드 (US5의 정보 카드 자리 남김)
- [x] T056 [US4] `src/server/post.ts`에 `resolveCategory` 추가, `savePost`에 연결 / `PostForm`에 [대분류 ▼][소분류 ▼] (`getCategoryOptions`)
- [x] T057 [P] [US4] `src/components/post/CategoryBadge.tsx`: `대분류` / `대분류 › 소분류`, 상세에서는 `?category=`·`&sub=` 링크
- [x] T058 [US4] `src/components/blog/CategoryNav.tsx` + 블로그 홈에 `?category`·`?sub` 필터 연결 (sub 우선, 이상한 값 무시), 휴대폰은 접는 목록

**Checkpoint**: 카테고리로 글을 정리하고 거를 수 있다

---

## Phase 7: User Story 5 - 블로그 이름·소개·주소와 닉네임을 바꾼다 (P3)

**Goal**: 블로그 정보·주소·닉네임 변경 (BLOG-03, AUTH-03)

**Independent Test**: 주소 변경 → 새 주소 열림, 예전 주소 404

### Tests for User Story 5

- [x] T059 [P] [US5] `tests/e2e/settings.spec.ts`: 시나리오 1~6 (문구·값 유지, 주소 소문자화·예약어·겹침·같은 주소, 닉네임 규칙, 남의 blogId 섞어도 내 것만)

### Implementation for User Story 5

- [x] T060 [US5] `src/server/blog.ts`: `updateBlogInfo`, `updateBlogSlug`(lockName → findNameConflict), `src/server/profile.ts`: `updateNickname`
- [x] T061 [US5] `src/app/settings/blog/actions.ts`에 `updateBlogInfo`·`updateBlogSlug`, `src/app/settings/account/actions.ts`에 `updateNickname`
- [x] T062 [P] [US5] `src/components/settings/BlogInfoForm.tsx`, `SlugForm.tsx`, `NicknameForm.tsx` (`저장했어요 ✓`, 오류는 버튼 왼쪽 한 줄)
- [x] T063 [US5] `src/app/settings/blog/page.tsx`에 정보·주소 카드, `src/app/settings/account/page.tsx`, 블로그 홈 주인 버튼 [⚙️ 블로그 관리]

**Checkpoint**: 모든 이야기가 혼자 동작

---

## Phase 8: Polish & Cross-Cutting

- [x] T064 [P] `tests/e2e/mobile.spec.ts`: 375px 모든 화면 가로 스크롤 없음, 누르는 영역 44px, 버튼 글자 한 줄 (NF-06, SC-006)
- [x] T065 [P] 키보드만으로 가입·로그인·글쓰기 E2E와 초점 테두리 스타일 (NF-18)
- [x] T066 [P] 다른 Origin으로 savePost·deletePost·카테고리 액션을 보내면 거부되는지 E2E (NF-12, SC-005)
- [x] T067 코드 리뷰 점검: 모든 Server Action 첫 줄 `requireMember()`, `sql.raw`·문자열 이어 붙이기 없음 (NF-02, NF-13)
- [x] T068 [P] `README.md`: quickstart.md 내용, 스크립트 목록
- [x] T069 `docs/01-requirements.md`에서 이번에 구현한 ID의 상태와 이 spec과 다르게 정한 점(가입 뒤 가는 곳, 캐릭터 칸 없음) 메모, 변경 이력 한 줄
- [x] T070 quickstart.md 시나리오를 처음부터 따라 해 확인

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (1)** → **Foundational (2)** → 이야기들 → **Polish (8)**
- Foundational이 끝나기 전에는 어떤 이야기도 시작하지 않는다.

### User Story Dependencies

- **US1 (P1)**: Foundational 뒤 바로. 다른 이야기에 의존하지 않는다.
- **US2 (P1)**: 글 주인이 필요해 US1의 `createMember`(T023)와 로그인(T025)에 의존. 테스트는 도우미(T015)로 회원을 만든다.
- **US3 (P2)**: US2의 `posts` 쓰기(T035)에 의존하지만, 시드(T046)로 혼자 시험할 수 있다.
- **US4 (P2)**: 관리 화면은 US1 뒤 바로 가능. 글 연결(T056~T058)은 US2·US3 뒤.
- **US5 (P3)**: US1 뒤 바로 가능. US4와 같은 `settings/blog` 파일을 만지므로 T055 → T063 순서.

### Within Each User Story

- 테스트를 먼저 쓰고 실패 확인 → lib(규칙) → server(DB) → actions → components → page.

### Parallel Opportunities

- Setup의 T002~T005, Foundational의 T008~T010·T014·T015
- 이야기마다 [P] 테스트 파일과 lib·컴포넌트 파일
- US1이 끝나면 한 사람은 US2→US3, 다른 사람은 US4 관리 화면, 또 한 사람은 US5를 동시에 진행할 수 있다 (팀 3명 기준)

## Parallel Example: User Story 2

```bash
Task: "T030 tests/unit/sanitize.test.ts"
Task: "T031 tests/unit/post-rules.test.ts"
Task: "T032 tests/e2e/post.spec.ts"
Task: "T033 src/lib/sanitize.ts"
Task: "T034 src/lib/post-rules.ts"
Task: "T037 src/components/editor/RichEditor.tsx"
```

## Implementation Strategy

### MVP First

1. Phase 1~2
2. Phase 3 (US1) + Phase 4 (US2)
3. **멈추고 확인**: 가입 → 글 발행 → 상세 → 수정·삭제를 시연
4. 이어서 US3 → US4 → US5 순서로 하나씩 더하고 매번 시연

### Next Specs (이 spec 다음)

- `002-blog-extras`: 태그(POST-04), 조회수(POST-06), 방문자 수(BLOG-06), 위젯·검색·글 관리(BLOG-07~09)
- `003-attachments`: 이미지·파일 첨부, 임시 저장 (POST-07~09)
- `004-social`: 댓글·답글·공감·이웃·마을 소식 (SOC-01~05)
- `005-game-core`: 캐릭터·경험치·코인·출석 (GAME), 이후 상점·마을 (SHOP, TOWN)

## Notes

- 태스크마다 또는 논리 묶음마다 커밋한다.
- PR 설명에 다룬 요구사항 ID와 태스크 ID를 적는다.
