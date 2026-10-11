# Implementation Plan: 블로그 기본 기능 (가입·블로그·글·카테고리)

**Branch**: `001-blog-core` | **Date**: 2026-10-11 | **Spec**: [spec.md](./spec.md)

**Input**: Feature specification from `/specs/001-blog-core/spec.md`

## Summary

아이디로 가입하면 블로그(`/@아이디`)가 함께 생기고, Tiptap 에디터로 글을 써서 공개/비공개로 발행하며, 블로그 홈에서 최신순 8개씩 목록을 넘기고 2단계 카테고리로 정리하는 웹 블로그를 만든다. Next.js 16 App Router의 서버 컴포넌트가 읽기를, Server Action이 모든 쓰기를 맡고, Drizzle + PostgreSQL에 저장한다. 로그인은 better-auth username 플러그인을 Server Action 안에서만 부른다. 기술 선택의 근거는 [research.md](./research.md).

## Technical Context

**Language/Version**: TypeScript 5.x (strict), Node.js 22 LTS

**Primary Dependencies**: Next.js 16 (App Router, Server Action), React 19, better-auth (username 플러그인, nextCookies), Drizzle ORM + drizzle-kit, `postgres`(postgres.js) 드라이버, Tiptap 3 (StarterKit·Link·Placeholder), sanitize-html, zod, Tailwind CSS 4

**Storage**: PostgreSQL 16 (로컬은 Docker Compose)

**Testing**: Vitest (입력 규칙·정화·ID 파싱 단위 테스트), Playwright (E2E, Chromium 데스크톱 + 375×812 휴대폰)

**Target Platform**: Node 서버(배포는 Vercel 또는 Docker), 최신 Chrome·Edge·Safari·모바일 브라우저

**Project Type**: web-service (Next.js 풀스택 단일 프로젝트)

**Performance Goals**: 블로그 홈 글 목록 서버 응답 200ms 이내(로컬 프로덕션 빌드), 글 1,000개에서도 화면 완성 1초 이내 (NF-07)

**Constraints**: 쓰기는 모두 Server Action(Origin 검사로 CSRF 차단), 라이브러리 HTTP 인증 경로는 `get-session` 외 404, 375px 가로 스크롤 없음·누르는 영역 44px

**Scale/Scope**: 화면 7개(첫 화면, 블로그 홈, 글 상세, 글쓰기/수정, 블로그 관리, 내 정보, 404/오류), 테이블 10개(인증 4 + 도메인 6), 회원 수천 명·블로그당 글 수천 개 규모

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

| 원칙 | 확인 | 결과 |
|---|---|---|
| I. 글쓰기가 먼저다 | 범위가 AUTH·BLOG·POST 핵심뿐이고 게임 테이블·칸을 미리 만들지 않는다. 이야기 5개가 각각 시연 가능 | ✅ |
| II. 판단은 서버가 한다 | 모든 Server Action 첫 줄 `requireMember()`, 주인은 세션에서 정함, 입력 규칙은 `src/lib/*-rules.ts` 공용, `parseId` | ✅ |
| III. 무결성은 DB로도 | 가입 트랜잭션, UNIQUE(owner_id, slug, lower(username)), CHECK(길이), FK ON DELETE 규칙, drizzle 마이그레이션만 | ✅ |
| IV. 보안은 기본값 | 해시 비밀번호, sanitize-html 저장 시 정화, 쿠키 속성, 로그인 5회/5분 잠금, 비공개 글 404 | ✅ |
| V. 헤매지 않는다 | 명세 문구 그대로, 오류 1개·입력 유지, 375px/44px, 키보드 | ✅ |
| VI. 실행해서 확인 | 수용 시나리오마다 E2E 또는 Vitest 태스크가 tasks.md에 있다 | ✅ |

Phase 1 설계 뒤 다시 확인: 위반 없음. Complexity Tracking 비움.

## Project Structure

### Documentation (this feature)

```text
specs/001-blog-core/
├── plan.md              # 이 파일
├── research.md          # Phase 0: 기술 결정과 근거
├── data-model.md        # Phase 1: 테이블·제약·상태
├── quickstart.md        # Phase 1: 로컬 실행과 시나리오 확인
├── contracts/
│   ├── routes.md        # 화면 주소와 접근 규칙
│   └── server-actions.md# Server Action 입력·출력·오류
├── checklists/
│   └── requirements.md  # spec 품질 점검
└── tasks.md             # Phase 2: 실행 태스크
```

### Source Code (repository root)

```text
src/
├── app/
│   ├── layout.tsx                    # 헤더(로고·내 블로그·글쓰기·로그아웃)
│   ├── page.tsx                      # 첫 화면: 로그인·회원가입 탭 (로그인했으면 내 블로그로)
│   ├── not-found.tsx                 # 404 "길을 잃었어요"
│   ├── error.tsx                     # "잠깐 문제가 생겼어요" + [다시 시도]
│   ├── (auth)/actions.ts             # signUp, signIn, signOut
│   ├── blog/[slug]/page.tsx          # 블로그 홈 (/@slug)
│   ├── blog/[slug]/[postId]/page.tsx # 글 상세 (/@slug/postId)
│   ├── write/page.tsx                # 새 글
│   ├── write/[postId]/page.tsx       # 글 수정
│   ├── write/actions.ts              # savePost, deletePost
│   ├── settings/blog/page.tsx        # 블로그 정보 + 카테고리
│   ├── settings/blog/actions.ts      # updateBlogInfo, updateBlogSlug, category*
│   ├── settings/account/page.tsx     # 닉네임
│   ├── settings/account/actions.ts   # updateNickname
│   └── api/auth/[...all]/route.ts    # better-auth 핸들러 (허용 목록: get-session만)
├── components/
│   ├── auth/                         # LoginForm, SignUpForm, SignOutButton
│   ├── blog/                         # BlogHeader, CategoryNav, PostCard, Pagination, WelcomeCard
│   ├── editor/                       # RichEditor(Tiptap), Toolbar
│   ├── post/                         # PostForm, PostBody, CategoryBadge, DeletePostButton
│   └── settings/                     # BlogInfoForm, SlugForm, CategoryManager, NicknameForm
├── db/
│   ├── schema.ts                     # Drizzle 스키마 (data-model.md)
│   └── index.ts                      # postgres.js 연결
├── server/                           # 서버 전용 (import "server-only")
│   ├── dal.ts                        # getSession, requireMember, getMyBlog
│   ├── signup.ts                     # createMember (가입 트랜잭션)
│   ├── names.ts                      # lockName(advisory lock), findNameConflict
│   ├── login-attempts.ts             # reserveLoginAttempt, clearLoginAttempts
│   ├── blog.ts                       # getBlogBySlug, updateBlog, lockBlog
│   ├── category.ts                   # 카테고리 CRUD·순서, getCategoryTree(글 수 포함)
│   └── post.ts                       # createPost, updatePost, deletePost, listPosts, getPost, resolveCategory
└── lib/                              # 브라우저·서버 공용 순수 함수
    ├── auth.ts                       # better-auth 설정 (세션 2h/7d 훅)
    ├── names.ts                      # normalizeName, USERNAME_RE, SLUG_RE, RESERVED_NAMES
    ├── blog-rules.ts                 # checkBlogTitle, checkBlogIntro, checkNickname
    ├── category-rules.ts             # checkCategoryName
    ├── post-rules.ts                 # postInputSchema, checkPostInput, POST_ERRORS
    ├── sanitize.ts                   # sanitizePostHtml, htmlToText
    ├── ids.ts                        # parseId, parsePage
    └── text.ts                       # codePointLength, excerpt
drizzle/                              # 마이그레이션 SQL (0000_init.sql …)
scripts/seed.ts                       # 개발용 회원·글 시드
tests/
├── unit/                             # Vitest: names, post-rules, sanitize, ids, blog-rules
└── e2e/                              # Playwright: auth, session, post, list, category, settings, mobile
next.config.ts                        # rewrites: /@:slug → /blog/:slug, /@:slug/:postId → /blog/:slug/:postId
docker-compose.yml                    # postgres:16
```

**Structure Decision**: Next.js 단일 프로젝트. 화면과 Server Action은 `src/app`, DB를 만지는 코드는 `src/server`(server-only), 브라우저와 서버가 함께 쓰는 규칙은 `src/lib`에 둔다. 명세 문서가 가리키는 경로(`src/lib/post-rules.ts`, `src/server/signup.ts`, `src/server/dal.ts` 등)와 맞춰 나중에 명세 상세를 그대로 따라갈 수 있게 했다.

## 주요 설계 결정

1. **`/@slug` 주소**: Next.js에서 `@`로 시작하는 폴더는 parallel route라 쓸 수 없다 → `next.config.ts` rewrites로 `/blog/[slug]`에 잇는다. 주소창은 `/@slug` 그대로. `/blog/...`로 직접 와도 같은 화면이지만 링크는 항상 `/@` 형식으로 만든다.
2. **가입 트랜잭션**: 비밀번호 해시는 트랜잭션 밖에서 미리 만든다(느린 작업). 트랜잭션 안에서 `pg_advisory_xact_lock(hashtext(lower(name)))`로 아이디를 잠그고, 아이디·주소·닉네임 충돌을 확인한 뒤 users → accounts → login_attempts 삭제 → profiles → blogs → categories("일상") 순으로 넣는다. UNIQUE 위반도 `이미 있는 아이디예요`로 바꾼다. 커밋 뒤 `auth.api.signInUsername({ rememberMe: false })`.
3. **better-auth 사용 범위**: 사용자·세션·계정 표와 비밀번호 해시·세션 쿠키만 쓴다. 가입은 직접 트랜잭션으로 만들고(블로그와 한 트랜잭션이어야 해서), `/api/auth/[...all]`은 `get-session`만 통과시키고 나머지는 404.
4. **세션 2시간 규칙**: 세션 생성 훅에서 `remember_me`와 만료(2h/7d)를 정하고, `getSession()`이 유지 안 함 세션의 `updated_at`이 2시간 지났으면 무효로 보고, 5분에 한 번만 `updated_at`을 갱신한다.
5. **읽기와 쓰기**: 블로그 홈·상세는 서버 컴포넌트가 `src/server/*`를 직접 부른다. 쓰기는 Server Action → `requireMember()` → zod/공용 규칙 → 트랜잭션 → `revalidatePath` → `redirect`.
6. **글 목록**: `ORDER BY created_at DESC, id DESC LIMIT 8 OFFSET (page-1)*8`, 전체 수는 같은 조건의 `count(*)`. 인덱스 `posts(blog_id, created_at desc, id desc)`. 주인이 아니면 `is_public = true` 조건.
7. **카테고리 순서**: `position` 정수. ▲▼는 `lockBlog`(블로그 행 `SELECT … FOR UPDATE`) 뒤 이웃 행과 position을 맞바꾼다. 새 항목은 `max(position)+1`.
8. **본문 저장**: 브라우저의 Tiptap이 HTML을 만들고, 서버가 `sanitizePostHtml`로 정화한 `content_html`과 태그를 뺀 `content_text`(목록 요약 120자)를 함께 저장한다. 상세는 `content_html`을 그대로 그린다(이미 정화됨).
9. **오류 화면**: `notFound()` → `not-found.tsx`, 예상 못 한 오류 → `error.tsx`. 남의 글 수정 화면은 404.

## Complexity Tracking

> 위반 없음.
