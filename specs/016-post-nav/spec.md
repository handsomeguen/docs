# Feature Specification: 이전 글·다음 글과 블로그 이름 링크

**Feature Branch**: `016-post-nav`

**Created**: 2026-10-12

**Status**: Implemented

**Input**: User description: "그냥 내가말없어도 1시간동안계속진행해줘" (015 다음 순서, `plans/gap-survey-2026-10-12.md` 3번)

**다루는 명세 요구사항**: POST-05 글 상세 아래 이전 글·다음 글, BLOG-02 글 상세 위 블로그 이름 링크·블로그 홈 제목 링크, POST-02 비공개 글 탭 제목.

**명세와 다른 점**: 공개 글 탭 제목은 1단계에서 정한 `{글 제목} | {블로그 이름}`을 그대로 쓴다(명세는 `{글 제목} | Blogville`). 블로그 이름 링크 옆 프로필 사진·캐릭터는 없다. 남이 비공개 글이나 없는 글 주소를 열면 지금처럼 404 화면이고 탭 제목은 1단계부터 쓰던 `길을 잃었어요 | Blogville`이다(명세는 `글 | Blogville`, 바깥으로 새는 정보는 같다).

## User Scenarios & Testing *(mandatory)*

### User Story 1 - 글 상세에서 옆 글로 간다 (Priority: P1) 🎯

**Acceptance Scenarios**:

1. **Given** 글 상세, **Then** 댓글 위에 `이전 글 {제목}`(바로 전에 쓴 글)과 `다음 글 {제목}`(바로 뒤에 쓴 글)이 한 줄에 하나씩. 순서는 블로그 홈 목록과 같은 (작성 시각, 글 번호).
2. **Given** 같은 시각에 쓴 글들, **Then** 다음 글은 바로 다음 글이고 자기 자신이 아니다.
3. **Given** 가장 최신 글, **Then** 다음 글 칸이 없다. 가장 오래된 글에는 이전 글 칸이 없고, 글이 하나면 칸 전체가 없다.
4. **Given** 주인이 아닌 사람, **Then** 사이의 비공개 글을 건너뛴다. 주인에게는 비공개 글도 나오고, 칸에는 제목만(배지 없음).
5. **Given** 375px와 긴 제목, **Then** 한 줄에서 …로 줄이고 가로 스크롤이 없다. 칸은 44px 이상.

### User Story 2 - 블로그 이름으로 블로그 홈에 간다 (Priority: P2)

1. **Given** 글 상세, **Then** 맨 위(카테고리 배지 위)에 `📖 {블로그 이름}` 링크(44px)가 `/@주소`로 간다.
2. **Given** 블로그 홈의 카테고리·검색·2쪽 화면, **Then** 제목(블로그 이름, 44px)을 누르면 `/@주소` 첫 화면.

### User Story 3 - 비공개 글 탭 제목 (Priority: P2)

1. **Given** 비공개 글 상세, **Then** 주인이 봐도 탭 제목은 `글 | Blogville`.

## Success Criteria *(mandatory)*

- **SC-001**: 수용 시나리오가 자동 테스트(`tests/e2e/post-nav.spec.ts`)로 통과한다.
