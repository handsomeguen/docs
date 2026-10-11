# Feature Specification: 글 관리와 주인 도구 막대

**Feature Branch**: `006-post-manager`

**Created**: 2026-10-11

**Status**: Implemented

**Input**: User description: "너가 알아서 계속 진행해줘" (005 다음 순서)

**다루는 명세 요구사항**: BLOG-08 (주인 도구 막대, 글 관리 `/settings/posts`)

**이번에 하지 않는 것**: 도구 막대의 `🪞 옷장 꾸미기`(게임이 없다), 여러 글 한꺼번에 고르기.

## User Scenarios & Testing *(mandatory)*

### User Story 1 - 주인 도구 막대 (Priority: P1) 🎯

주인이 자기 블로그 홈 위에서 [✏️ 새 글 쓰기] [📝 글 관리 N] [🗂 카테고리 관리] [⚙️ 블로그 설정]으로 바로 간다. N은 비공개를 포함한 내 글 수. 다른 사람에게는 없다.

**Acceptance Scenarios**:

1. **Given** 주인, **Then** 네 버튼이 보이고 [🗂 카테고리 관리]는 `/settings/blog#categories`의 카테고리 상자로 간다.
2. **Given** 다른 회원·방문자, **Then** 도구 막대가 없다. 방문자가 `/settings/posts`를 열면 첫 화면으로 간다.

### User Story 2 - 글 관리 (Priority: P1)

`/settings/posts`에서 내 글을 공개·비공개 모두 20개씩 최신순으로 보고, 상태·카테고리·제목·본문 검색으로 거르고, [수정] [🔒 비공개로/🌐 공개로] [삭제]를 한다.

**Acceptance Scenarios**:

1. **Given** 처음 열기, **Then** `전체 N · 공개 N · 비공개 N`과 `내 글 N개`. 줄마다 제목(상세로), 공개 상태, 카테고리, 날짜, 👀, 💬.
2. **Given** 조건으로 거르기(`?visibility=`, `?cat=c{ID}|s{ID}`, `?s=`), **Then** `찾은 글 N개`, 페이지를 넘겨도 조건이 남고 [조건 지우기]로 처음으로. 이상한 값은 무시, 빈 검색은 `찾을 말을 적어 주세요`.
3. **Given** [비공개로]/[공개로], **Then** 바로 바뀌고 숫자가 바뀐다. 글의 수정 시각은 그대로.
4. **Given** [삭제] → 확인 창에서 취소, **Then** 그대로. 확인하면 지워지고 글 관리에 머문다.
5. **Given** 남의 글 ID로 바꾸기·삭제를 조작, **Then** 아무것도 바뀌지 않는다.
6. **Given** 375px, **Then** 가로 스크롤 없음.

## Requirements *(mandatory)*

- **FR-001**: 바꾸기·삭제는 `WHERE id = ? AND blog_id = 내 블로그`로 서버가 막는다.
- **FR-002**: 목록은 블로그 홈과 같은 `listBlogPosts`(주인 보기 + `visibility`·`pageSize`)를 쓴다.

## Success Criteria *(mandatory)*

- **SC-001**: 수용 시나리오가 모두 자동 테스트로 통과한다.
