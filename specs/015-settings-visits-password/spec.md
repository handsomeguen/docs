# Feature Specification: 방문자 그래프·카테고리 글 수·비밀번호 바꾸기

**Feature Branch**: `015-settings-visits-password`

**Created**: 2026-10-12

**Status**: Implemented

**Input**: User description: "그냥 내가말없어도 1시간동안계속진행해줘" (014 다음 순서, `plans/gap-survey-2026-10-12.md` 1·2번)

**다루는 명세 요구사항**: BLOG-06 블로그 관리의 최근 7일 방문 그래프, BLOG-05 관리 화면의 (글 수). 명세에 없던 **비밀번호 바꾸기**(내 정보)와 헤더의 `👤 내 정보`·방문자 `로그인` 링크를 더했다.

**명세와 다른 점**: 명세의 헤더 캐릭터 배지(게임) 대신 `👤 내 정보` 링크를 둔다.

## User Scenarios & Testing *(mandatory)*

### User Story 1 - 블로그 관리에서 방문자 흐름을 본다 (Priority: P1) 🎯

**Acceptance Scenarios**:

1. **Given** 블로그 관리(`/settings/blog`), **Then** 맨 위 `👣 방문자` 카드에 `오늘 N · 어제 N · 전체 N`과 최근 7일 막대(왼쪽이 6일 전, 마지막 칸 `오늘`)가 보인다. 막대마다 `M/D 방문 N`을 읽어 준다.
2. **Given** 방문이 없는 날, **Then** 0으로 낮은 막대. 7일보다 오래된 방문은 전체에만 들어간다.
3. **Given** 375px, **Then** 가로 스크롤이 없다.

### User Story 2 - 카테고리 줄에 글 수 (Priority: P2)

1. **Given** 블로그 관리의 카테고리, **Then** 줄마다 `(N)`. 비공개 글도 세고, 대분류는 소분류 글을 포함한다.

### User Story 3 - 비밀번호를 바꾼다 (Priority: P1)

1. **Given** 내 정보(`/settings/account`)의 `비밀번호 바꾸기`에 지금 비밀번호·새 비밀번호(8~64자)·확인, **Then** `비밀번호를 바꿨어요`. 이 브라우저는 로그인이 그대로, **다른 기기의 로그인은 끊긴다**. 다음부터 새 비밀번호로만 들어간다.
2. **Given** 빈 칸·8자 미만·64자 초과·확인 다름·지금과 같음·지금 비밀번호 틀림, **Then** 각각 `지금 비밀번호와 새 비밀번호를 적어 주세요`, `비밀번호는 8자 이상이에요`, `비밀번호는 64자까지예요`, `비밀번호가 서로 달라요`, `지금과 다른 비밀번호를 적어 주세요`, `지금 비밀번호가 맞지 않아요`. 비밀번호는 그대로.

### User Story 4 - 헤더 링크 (Priority: P3)

1. **Given** 회원, **Then** 헤더에 `👤 내 정보`(375px에서는 👤만, 44px). **Given** 방문자, **Then** 헤더에 `로그인`(첫 화면).

## Success Criteria *(mandatory)*

- **SC-001**: 수용 시나리오가 자동 테스트(`tests/e2e/account-settings.spec.ts`)와 단위 테스트(`tests/unit/settings.test.ts`: `lastDays`, `fillDays`, `checkPasswordChange`)로 통과한다.
