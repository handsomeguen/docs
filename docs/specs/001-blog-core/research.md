# Research: 블로그 기본 기능

Phase 0. spec과 constitution에서 정하지 않은 기술 선택을 정리한다. 기본 스택은 2026-10-11 사용자 결정(명세와 같은 스택)을 따른다.

## 1. 웹 프레임워크: Next.js 16 App Router

- **Decision**: Next.js 16 App Router, 서버 컴포넌트로 읽고 Server Action으로 쓴다. 별도 REST API 서버는 두지 않는다.
- **Rationale**: 명세(`docs/01-requirements.md`)의 구현 방식이 모두 이 구조다(`src/app/...`, Server Action, `revalidatePath`). Server Action은 Origin 검사로 CSRF를 기본 차단한다(NF-12).
- **Alternatives**: Pages Router + API Routes(명세와 어긋남), 별도 백엔드(Express/Nest, 이 규모에 과함).

## 2. `/@slug` 주소

- **Decision**: `next.config.ts` rewrites `/@:slug` → `/blog/:slug`, `/@:slug/:postId` → `/blog/:slug/:postId`.
- **Rationale**: App Router에서 `@폴더`는 parallel route 문법이라 주소로 못 쓴다 (명세 BLOG-02 처리 방식과 같음).
- **Alternatives**: middleware에서 rewrite(같은 효과, 설정이 더 복잡), 주소를 `/blog/slug`로 바꿈(명세 위반).

## 3. DB와 ORM: PostgreSQL 16 + Drizzle

- **Decision**: Drizzle ORM 스키마(`src/db/schema.ts`) + drizzle-kit `generate`로 SQL 마이그레이션, 드라이버는 postgres.js.
- **Rationale**: 명세가 Drizzle 마이그레이션(`drizzle/00xx_*.sql`, `$onUpdate`)을 기준으로 쓰였다. CHECK·부분 인덱스·식 인덱스(`lower(username)`)를 스키마에 적을 수 있다.
- **Alternatives**: Prisma(CHECK 제약·advisory lock을 raw SQL로 써야 함), SQLite(사용자가 고르지 않음).

## 4. 인증: better-auth (username 플러그인)

- **Decision**: better-auth의 `user`·`session`·`account`·`verification` 표와 비밀번호 해시(scrypt), 세션 쿠키를 쓴다. 가입 레코드는 우리 트랜잭션에서 직접 만들고, 로그인은 Server Action 안에서 `auth.api.signInUsername`을 부른다. `nextCookies()` 플러그인으로 Server Action에서 쿠키를 설정한다.
- **Rationale**: 명세 AUTH-07·09의 구현 방식과 같다. 가입을 직접 트랜잭션으로 해야 블로그·카테고리와 원자적으로 만들 수 있다. 이메일 칸은 라이브러리가 필수로 요구해 `{아이디}@users.blogville.invalid`를 넣는다.
- **해시 함수 공유**: 가입 트랜잭션에서는 better-auth가 내보내는 `hashPassword`(`better-auth/crypto`)를 써서 로그인 검증과 같은 형식으로 저장한다.
- **Alternatives**: Auth.js(아이디·비밀번호 Credentials가 세션 DB와 잘 맞지 않음), 직접 구현(보안 위험).

## 5. 세션 유지 시간

- **Decision**: `session` 표에 `remember_me boolean`을 더한다(`additionalFields`, `input: false`). 생성 훅에서 유지 안 함 = 만료 2시간·쿠키 Max-Age 없음, 유지 = 7일. `getSession()`이 유지 안 함 세션의 마지막 사용 2시간 경과를 검사하고 5분마다 `updated_at`을 갱신.
- **Rationale**: 명세 AUTH-09 표 그대로.
- **Alternatives**: 라이브러리 기본(늘 7일, 결정과 다름).

## 6. 로그인 시도 제한

- **Decision**: `login_attempts(username PK, failures, locked_until, updated_at)`. 로그인 전에 실패로 미리 세고(`reserveLoginAttempt`, 행 잠금), 성공하면 지운다. 5번째 실패에서 `locked_until = now() + 5분`. 시각은 DB `now()` 기준.
- **Rationale**: 명세 NF-10. 라이브러리 HTTP 경로를 막아 우회를 없앤다.
- **Alternatives**: IP 기준 제한(공유 IP 문제), 메모리 카운터(서버 여러 대에서 깨짐).

## 7. 이름 겹침과 동시성

- **Decision**: 아이디·블로그 주소·닉네임을 바꾸거나 만들 때 트랜잭션 안에서 `pg_advisory_xact_lock(hashtext('name:' || lower(값)))` → `findNameConflict`(users.username, blogs.slug, profiles.nickname 세 곳 확인). 각 컬럼에도 UNIQUE.
- **Rationale**: "세 값이 서로 겹칠 수 없다"(AUTH-03)는 테이블을 넘는 규칙이라 UNIQUE 하나로 못 막는다. 잠금으로 동시 가입 중 하나만 성공.
- **Alternatives**: 이름 전용 테이블 `names(name PK, kind, owner)`(더 단단하지만 이번 범위에 과함, 나중에 고려).

## 8. 에디터와 HTML 정화

- **Decision**: Tiptap 3 `StarterKit`(heading levels [2,3], link `openOnClick:false`·`autolink:true`) + `Placeholder`. 서버 `sanitize-html` 허용 목록: `p br h2 h3 strong em u s code pre blockquote ul ol li a hr`, `a[href]`는 http·https·mailto·`/` 시작만, `transformTags`로 `target`·`rel` 부여.
- **Rationale**: 명세 POST-01·NF-03. 이미지·파일 카드는 첨부 기능(POST-07·09)이 생길 때 허용 목록에 더한다.
- **Alternatives**: DOMPurify + jsdom(서버에서 무거움), 마크다운 저장(명세는 HTML).

## 9. 입력 규칙 공유

- **Decision**: `src/lib/post-rules.ts` 등에 zod 스키마 + 순서대로 검사해 첫 오류 문구를 돌려주는 함수를 두고, 클라이언트 폼과 Server Action이 같이 import한다. 길이는 `[...str].length`(코드 포인트)로 센다(블로그 이름·소개·카테고리·닉네임). 글 제목은 명세대로 앞뒤 공백 제거 뒤 100자.
- **Rationale**: 브라우저와 서버의 판단이 어긋나지 않게(명세 POST-01 입력 규칙).

## 10. 테스트

- **Decision**: Vitest로 순수 함수, Playwright로 E2E(로컬 Next 서버 + 테스트 DB, 각 테스트가 고유 아이디로 회원 생성). 휴대폰은 375×812 터치 에뮬레이션.
- **Rationale**: constitution VI. 명세의 `e2e/*.mjs` 시나리오와 1:1로 맞추기 쉽다.

## 11. 시간대

- **Decision**: DB는 `timestamptz`, 화면 표시는 `Intl.DateTimeFormat('ko-KR', { timeZone: 'Asia/Seoul' })`.
