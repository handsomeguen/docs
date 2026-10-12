# Feature Specification: 블로그 둘러보기

**Feature Branch**: `025-blog-browse`

**Created**: 2026-10-12

**Status**: Implemented

**Input**: User description: "계속 진행해줘" (024 다음 순서, `plans/gap-survey-2026-10-12.md` 12번)

**다루는 명세 요구사항**: SOC-05 열린 질문(제안) "마을 소식 1페이지 맨 위에만 블로그 둘러보기 줄을 둘까? 공개 글이 있는 블로그를 최근 공개 글 순으로 12개, 칸마다 캐릭터 · 블로그 이름 · `닉네임 · 글 N`".

**명세와 다른 점**: 캐릭터(게임)가 아직 없어 칸에는 블로그 이름과 `닉네임 · 글 N`만 보인다.

## User Scenarios & Testing *(mandatory)*

### User Story 1 - 마을 소식에서 블로그 찾아가기 (Priority: P1) 🎯

**Acceptance Scenarios**:

1. **Given** 마을 소식 1페이지(`/feed`), **Then** 탭 아래·글 카드 위에 `🏡 블로그 둘러보기`, 공개 글이 있는 블로그를 최근 공개 글 순으로 12개까지. 비공개 글만 있는 블로그는 없다.
2. **Given** 칸, **Then** 블로그 이름과 `{닉네임} · 글 N`(공개 글 수), 누르면 그 블로그 홈. 누르는 영역 44px 이상.
3. **Given** 2페이지나 이웃 새 글 탭, **Then** 이 줄이 없다.
4. **Given** 375px, **Then** 두 칸씩 감기고 가로 스크롤이 없다(넓은 화면은 세 칸).

## Success Criteria *(mandatory)*

- **SC-001**: 수용 시나리오가 자동 테스트(`tests/e2e/feed-browse.spec.ts`)로 통과한다.
