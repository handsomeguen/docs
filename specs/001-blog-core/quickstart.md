# Quickstart: 블로그 기본 기능

## 준비

- Node.js 22, Docker (PostgreSQL 16용)

```bash
cp .env.example .env.local     # DATABASE_URL, BETTER_AUTH_SECRET, BETTER_AUTH_URL=http://localhost:3000
docker compose up -d db        # postgres:16, 포트 5432
npm install
npm run db:migrate             # drizzle/ 마이그레이션 적용
npm run db:seed                # (선택) 회원 demo / demo1234, 글 20개
npm run dev                    # http://localhost:3000
```

## 확인 시나리오 (spec의 이야기 순서)

1. **가입 (US1)**: `/` → [회원가입] → 아이디 `tester1`, 비밀번호 `password1` ×2 → 블로그 이름이 `tester1의 블로그`로 채워짐 → [회원가입] → `/@tester1?welcome=1`에 환영 문구, 빈 글 목록, 왼쪽에 `일상 (0)`.
2. **글쓰기 (US2)**: [✏️ 새 글 쓰기] → 제목 `첫 글`, 본문에 `## 소제목`, `- 목록`, 링크 → [발행하기] → `/@tester1/1`에 그대로 보임. [수정] → 바꿔서 [수정 완료]. 비공개로 바꾸고 시크릿 창에서 같은 주소 → 404.
3. **목록 (US3)**: `npm run db:seed` 후 `/@demo` → 8개 + 페이지 `1 2 3`. `/@demo?page=abc` → 1페이지.
4. **카테고리 (US4)**: [⚙️ 블로그 관리] → `여행` 추가 → [소분류 추가] `맛집` → 글쓰기에서 `여행 › 맛집` 선택 후 발행 → 배지 확인 → `맛집` 삭제 → 배지 `여행`.
5. **설정 (US5)**: 블로그 관리에서 주소를 `my_blog`로 → `/@my_blog` 열림, `/@tester1` 404. 내 정보에서 닉네임 변경.
6. **로그아웃·로그인**: 헤더 [로그아웃] → `/`. 틀린 비밀번호 5번 → 6번째는 잠금 문구.

## 테스트

```bash
npm run typecheck   # tsc --noEmit
npm run lint
npm test            # Vitest 단위 테스트
npm run test:e2e    # Playwright (테스트 DB: DATABASE_URL_TEST)
```
