# Feature Specification: 블로그 옆 칸·조회수·방문자

**Feature Branch**: `005-blog-sidebar`

**Created**: 2026-10-11

**Status**: Implemented

**Input**: User description: "너가 알아서 계속 진행해줘" (004 다음 순서: 블로그 쪽 남은 기능)

**다루는 명세 요구사항**: POST-06 (조회수), BLOG-06 (방문자 수), BLOG-09 (옆 칸 위젯·블로그 안 검색 `?s=`·태그 `?tag=`)

**이번에 하지 않는 것**: 마을 검색(BLOG-07, 마을이 없다), 블로그 관리의 최근 7일 방문 그래프, 프로필 카드·우리 집(BLOG-04), 주인 도구 막대·글 관리(BLOG-08), 공감 수(SOC-03).

## User Scenarios & Testing *(mandatory)*

### User Story 1 - 글 조회수를 센다 (Priority: P1) 🎯

주인이 아닌 사람이 글 상세를 브라우저에서 열면 `👀 N`이 1 오른다. 같은 브라우저는 글마다 하루(한국 시간) 1번.

**Acceptance Scenarios**:

1. **Given** 처음 연 브라우저, **Then** 저장값 + 1로 시작하고 서버가 돌려준 값으로 바뀐다. 쿠키 `bv_visitor`(UUID, HttpOnly, SameSite=Lax, 1년)가 생긴다.
2. **Given** 같은 브라우저로 새로고침, **Then** 오르지 않는다. 다른 브라우저면 오른다.
3. **Given** 주인, 비공개 글, HTML만 받아 가는 요청, **Then** 오르지 않는다.
4. **Given** 조회수가 오름, **Then** 글의 수정 시각은 그대로. 목록 카드에 `👀 N`.

### User Story 2 - 블로그 방문자를 센다 (Priority: P2)

블로그 홈이나 그 블로그의 글 상세를 연 사람을 블로그마다 하루 1번 센다. 옆 칸 `👣 방문자`에 오늘·어제·전체.

**Acceptance Scenarios**:

1. **Given** 처음 온 방문자, **Then** 오늘·전체가 1 오르고 새로고침 없이 보인다.
2. **Given** 같은 날 다시 옴·페이지 이동·글 상세를 거쳐 옴, **Then** 더 오르지 않는다.
3. **Given** 주인, **Then** 세지 않는다. 숫자는 같다.

### User Story 3 - 옆 칸 위젯과 블로그 안 검색 (Priority: P2)

블로그 홈을 왼쪽 글 목록 + 오른쪽 옆 칸(768px 미만은 목록 아래)으로 나눈다. 옆 칸: `🔍 블로그 검색`, `🗂 카테고리`, `📝 최근 글` 5, `🔥 인기 글`(조회수 1 이상, 많은 순 5), `💬 최근 댓글`(댓글·답글 5, `↳`는 답글, 누르면 그 댓글 자리), `🏷 태그`(글 수 많은 순 30), `👣 방문자`.

**Acceptance Scenarios**:

1. **Given** 주인이 아님, **Then** 비공개 글, 그 글의 댓글, 비공개 글에만 붙은 태그는 없다. 주인에게는 있다.
2. **Given** `?s=검색어`, **Then** 제목·본문에서 찾고 `🔍 '{검색어}' 블로그 검색 N개`. `%`·`_`·`\`는 글자 그대로. 비었거나 공백만이면 `찾을 말을 적어 주세요`. 51자 이상은 무시. 페이지를 넘겨도 검색어가 남는다.
3. **Given** 태그 클릭(`?tag=이름`), **Then** 이 블로그의 그 태그 글만 `#{태그} N개`, [← 전체 글 보기]. 없으면 `이 블로그에서 찾은 글이 없어요`.
4. **Given** `전체 글 (N)`, **Then** 누가 보든 공개 글 수.
5. **Given** 375px, **Then** 가로 스크롤 없음.

## Requirements *(mandatory)*

- **FR-001**: 날짜는 한국 시간 0시 기준(`todayKST`).
- **FR-002**: 기본 키 `(post_id, date, visitor_id)`·`(blog_id, date, visitor_id)`로 "하루 1번"을 DB가 지킨다. IP·회원 ID는 저장하지 않는다.
- **FR-003**: 조회 기록과 `view_count + 1`은 한 트랜잭션. `view_count >= 0` CHECK.
- **FR-004**: 세기는 화면이 열린 뒤 Server Action(`recordPostView`·`recordBlogVisit`)으로만. 서버 렌더는 쿠키를 심지 않고 숫자를 바꾸지 않는다. 주인인지는 Action에서 다시 확인한다.
- **FR-005**: `npm run posts:cleanup`이 어제보다 오래된 조회 기록을 지운다 (조회수는 그대로).

## Success Criteria *(mandatory)*

- **SC-001**: 수용 시나리오가 모두 자동 테스트로 통과한다.
- **SC-002**: 같은 브라우저가 같은 날 같은 글·블로그를 두 번 세는 경우가 0건이다.
