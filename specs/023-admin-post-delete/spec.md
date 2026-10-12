# Feature Specification: 관리자가 글 화면에서 지우기, 공지 블로그 주소

**Feature Branch**: `023-admin-post-delete`

**Created**: 2026-10-12

**Status**: Implemented

**Input**: User description: "계속 진행해줘" (022 다음 순서, `plans/gap-survey-2026-10-12.md` 10번)

**다루는 명세 요구사항**: AUTH-08 남은 수용 기준(오래된 글, 최근 30일·30개 밖도 관리자가 지울 수 있다)과 열린 질문(글 화면에 관리자 [삭제]를 둘까?), BLOG-03 주소 바꾸기 순서(지금과 같으면 성공 → 예약어 검사, 관리자의 `notice`는 허용).

**명세와 다른 점**: 없음. 비공개 글은 지금처럼 관리자에게도 404라(관리자 비공개 글 열람은 명세 예외 표가 막는다) 글 화면에서는 지울 수 없고 `/admin` 최근 글에서만 지운다. 그래서 최근 30일·30개 밖의 비공개 글은 아직 관리자가 지울 길이 없다.

## User Scenarios & Testing *(mandatory)*

### User Story 1 - 관리자가 남의 글 화면에서 [삭제] (Priority: P1) 🎯

**Acceptance Scenarios**:

1. **Given** 관리자가 남의 공개 글(오래되어 `/admin` 최근 글에 없는 글도)을 연다, **Then** 아래쪽에 [삭제]가 있다. 누르면 확인 창 `'{제목}' 글을 삭제할까요?`, 취소하면 그대로다.
2. **Given** 확인, **Then** 글이 지워지고 그 블로그 홈(`/@주소`)으로 간다.
3. **Given** 일반 회원·방문자가 남의 글을 연다, **Then** 관리자 [삭제]가 없고, 요청을 조작해 보내도 글이 그대로다.
4. **Given** 관리자가 자기 글을 연다, **Then** 지금처럼 [수정]·[삭제](주인 확인 창)만 있다.

### User Story 2 - 공지 블로그 주소 `notice` (Priority: P2)

1. **Given** 블로그 주소가 `notice`인 관리자가 주소를 그대로 두고 [주소 바꾸기], **Then** `저장했어요 ✓`.
2. **Given** 관리자가 다른 주소로 바꿨다가 `notice`로, **Then** 바뀐다.
3. **Given** 일반 회원이 `notice`로, **Then** `이 주소는 쓸 수 없어요`. 관리자도 `notice` 말고 다른 예약어는 쓸 수 없다.

## Success Criteria *(mandatory)*

- **SC-001**: 수용 시나리오가 자동 테스트(`tests/e2e/admin.spec.ts`, 주소 규칙은 `tests/unit/names.test.ts`)로 통과한다.
