<!--
Sync Impact Report
- Version: (none) → 1.0.0 (최초 작성)
- 근거 문서: docs/01-requirements.md (Blogville 요구사항 명세서 v1.26) 1장·5장(비기능)
- Added principles: I~VI 전부 새로 작성
- Templates: plan-template.md의 Constitution Check는 아래 6개 원칙을 게이트로 쓴다 (✅ docs/specs/001-blog-core/plan.md 반영)
- Follow-up TODO: 없음
-->

# Blogville Constitution

## Core Principles

### I. 글쓰기가 먼저다

블로그의 본질(가입 → 블로그 → 글 쓰기·발행 → 목록·상세 → 카테고리)이 먼저 완성되고, 캐릭터·코인·상점·마을 같은 게임 요소는 그 위에 얹는다 (명세 1.2 목표 1).

- 기능은 명세의 요구사항 ID(AUTH-xx, BLOG-xx, POST-xx …)로 추적한다. ID는 바꾸지 않고, 뺀 요구사항은 지우지 않고 `❌ 제외`로 둔다.
- 한 기능(spec)은 사용자 이야기 단위로 나누고, 각 이야기는 혼자 시연할 수 있어야 한다.
- 지금 단계에 필요 없는 테이블·추상화는 만들지 않는다 (YAGNI). 게임용 칸을 미리 만들지 않는다.

### II. 판단은 서버가 한다

- 모든 Server Action과 회원 전용 화면은 맨 앞에서 `requireMember()` / `requireAdmin()`으로 권한을 확인한다 (NF-02). 화면이 버튼을 숨기는 것만 믿지 않는다.
- 요청에 섞여 온 `ownerId`·`blogId`·`role` 같은 값은 읽지 않는다. 주인은 언제나 세션에서 정한다.
- 입력 규칙과 오류 문구는 `src/lib/*-rules.ts`(소스 코드 저장소) 한 곳에 두고 브라우저와 서버가 같이 쓴다.
- 번호(ID)는 숫자만 적힌 1~2147483647 정수만 받는다 (`parseId`). 그 밖의 값은 404 또는 `잘못된 요청이에요`.

### III. 무결성은 DB로도 지킨다

- 여러 행을 함께 만드는 일(가입 = 회원 + 프로필 + 블로그 + 기본 카테고리)은 **한 트랜잭션**이다. 중간에 실패하면 아무것도 남지 않는다.
- "회원당 블로그 1개", "주소 유일", "제목 1~100자" 같은 규칙은 앱 코드와 함께 **DB 제약조건**(UNIQUE·CHECK·FK)으로도 막는다 (NF-15).
- DB 구조 변경은 Drizzle **마이그레이션 파일**로만 한다 (NF-22). 쿼리는 ORM 값 바인딩만 쓰고 문자열을 이어 붙이지 않는다 (NF-13).

### IV. 보안은 기본값이다

- 비밀번호는 해시로만 저장한다 (NF-09). 비밀 값은 서버 환경 변수(`.env.local`)에만 둔다 (NF-01).
- 에디터 HTML은 **저장할 때** 허용 태그만 남기고 정화한다 (NF-03).
- 로그인 쿠키는 `HttpOnly`·`SameSite=Lax`·(https면) `Secure` (NF-11). 다른 사이트에서 온 요청은 Server Action Origin 검사로 막는다 (NF-12).
- 같은 아이디로 5번 연속 실패하면 5분 막는다 (NF-10). 없는 아이디와 틀린 비밀번호는 같은 문구다.
- 남의 비공개 글은 404로, 있다는 사실도 알리지 않는다.

### V. 쓰는 사람이 헤매지 않는다

- 오류 문구는 한국어로, **무엇을 고치면 되는지** 말한다 (NF-19). 한 번에 하나만 보이고, 오류가 나도 입력한 값은 남긴다 (비밀번호 제외).
- 휴대폰(375px)과 PC에서 모든 화면을 쓸 수 있다. 가로 스크롤 없음, 누르는 영역 44px 이상 (NF-06).
- 키보드만으로 가입·로그인·글쓰기를 할 수 있고 초점 테두리가 보인다 (NF-18).
- 글 목록 화면은 1초 안에 보인다 (NF-07).

### VI. 수용 기준은 실행해서 확인한다

- spec의 수용 시나리오(Given/When/Then)마다 확인 방법을 붙인다: E2E(Playwright), 단위 테스트, 또는 (코드 확인).
- 입력 규칙·HTML 정화처럼 순수 함수는 단위 테스트로, 화면 흐름은 E2E로 확인한다.
- merge 전에 `tsc --noEmit`, `eslint`, 단위 테스트, E2E가 통과해야 한다 (NF-23).

## 기술 제약

| 영역 | 정한 것 |
|---|---|
| 웹 | Next.js 16 App Router + TypeScript (strict), React Server Components + Server Action |
| DB | PostgreSQL 16, Drizzle ORM + drizzle-kit 마이그레이션 |
| 로그인 | better-auth (username 플러그인, 아이디·비밀번호만). 라이브러리 HTTP 가입·로그인 경로는 막고 Server Action으로만 쓴다 |
| 에디터 | Tiptap (StarterKit, 제목 H2·H3, 링크, Placeholder) + 서버 sanitize-html |
| 검증 | zod |
| 스타일 | Tailwind CSS |
| 테스트 | Vitest(단위), Playwright(E2E) |
| 블로그 주소 | `/@주소` → `next.config.ts` rewrite로 `/blog/[slug]` |

스택을 바꾸려면 아래 Governance 절차로 이 문서를 먼저 고친다.

## 개발 흐름

1. 기능마다 Spec Kit 순서를 따른다: `/speckit.specify` → `/speckit.clarify`(필요하면) → `/speckit.plan` → `/speckit.tasks` → `/speckit.implement`.
2. 브랜치 이름은 spec 폴더와 같다 (`001-blog-core`). spec 폴더는 이 저장소의 `docs/specs/`, 소스 코드는 [handsomeguen/blogville](https://github.com/handsomeguen/blogville) 저장소에 둔다. 한 기능의 spec과 코드는 두 저장소에 같은 브랜치 이름으로 올린다.
3. PR은 다른 팀원 1명 이상이 승인하면 merge한다. PR 설명에 다룬 요구사항 ID를 적는다.
4. 명세(`docs/01-requirements.md`)와 코드가 달라지면 같은 PR에서 명세의 해당 줄과 변경 이력을 고친다.

## Governance

- 이 constitution은 다른 개발 관례보다 우선한다. plan.md의 Constitution Check는 위 6개 원칙을 게이트로 쓰고, 어기는 설계는 Complexity Tracking에 이유를 적어야 한다.
- 고칠 때는 PR로 올리고 팀원 1명 이상이 승인한다. 버전은 원칙을 빼거나 뜻을 바꾸면 MAJOR, 원칙·절을 더하면 MINOR, 문구만 다듬으면 PATCH를 올린다.

**Version**: 1.0.0 | **Ratified**: 2026-10-11 | **Last Amended**: 2026-10-11
