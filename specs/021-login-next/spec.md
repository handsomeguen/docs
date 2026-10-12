# Feature Specification: 로그인 뒤 보던 글로 돌아오기

**Feature Branch**: `021-login-next`

**Created**: 2026-10-12

**Status**: Implemented

**Input**: User description: "계속 진행해줘" (020 다음 순서, `plans/gap-survey-2026-10-12.md` 8번)

**다루는 명세 요구사항**: AUTH-09 열린 질문(원래 보던 화면 `?next=`으로 돌려보낼까?), SOC-01 열린 질문(방문자가 `로그인`을 눌러 로그인하면 보던 글로 돌아오게 할까?), SOC-03 열린 질문(제안: 로그인하면 이 글로 돌아오게).

**명세와 다른 점**: 열린 질문을 이렇게 정했다. (1) 글 상세 댓글·공감의 [로그인]과 헤더 [로그인]만 돌아갈 주소를 붙인다. 회원 전용 화면(글쓰기·블로그 관리 등)에서 첫 화면으로 보내질 때는 아직 붙지 않는다. (2) 공감 버튼은 지금처럼 막아 두고 안내 글의 `로그인`을 링크로 바꾼다. (3) 가입은 환영 카드(`?welcome=1`) 때문에 지금처럼 내 블로그로 간다. (4) 첫 화면 로그인 폼 위에 `로그인하면 보던 화면으로 돌아가요`를 보인다(명세에 없는 문구).

## User Scenarios & Testing *(mandatory)*

### User Story 1 - 댓글을 달려고 로그인하면 그 글로 돌아온다 (Priority: P1) 🎯

**Acceptance Scenarios**:

1. **Given** 방문자가 남의 공개 글을 본다, **Then** 댓글 칸 `로그인하면 댓글을 남길 수 있어요`의 `로그인`은 `/?next=/@주소/글ID#comments`(인코딩)이고 44px 이상이다.
2. **Given** 그 링크로 첫 화면에 왔다, **Then** 로그인 폼 위에 `로그인하면 보던 화면으로 돌아가요`. 비밀번호를 틀려도 돌아갈 주소는 남는다.
3. **Given** 로그인, **Then** 그 글의 댓글 자리로 돌아오고 댓글 입력칸이 보인다.

### User Story 2 - 공감 안내와 헤더의 [로그인] (Priority: P1)

1. **Given** 방문자, **Then** 공감 버튼 아래 `로그인하면 공감할 수 있어요`의 `로그인`이 `/?next=/@주소/글ID` 링크(44px)다. 버튼 설명은 그대로 `로그인하면 공감할 수 있어요`.
2. **Given** 방문자 헤더의 [로그인], **Then** 지금 보던 화면 주소가 붙고 화면을 옮기면 따라 바뀐다. 첫 화면에서는 `/` 그대로.
3. **Given** 공감 안내의 [로그인]으로 로그인, **Then** 그 글로 돌아오고 공감 버튼이 눌린다.

### User Story 3 - 다른 사이트로 보내지 않는다 (Priority: P1)

1. **Given** `next`가 `//evil.com`, `https://evil.com`, `/\evil.com`, 탭이 섞인 값, `javascript:` 등, **Then** 안내 문구·숨은 칸이 없고 로그인하면 내 블로그.
2. **Given** 숨은 칸을 다른 사이트 주소로 바꿔 보냄, **Then** 내 블로그.
3. **Given** 이미 로그인한 채 `/?next=…`, **Then** 첫 화면을 거치지 않고 그 주소로(다른 사이트 주소면 내 블로그로).

### User Story 4 - 가입은 그대로 (Priority: P3)

1. **Given** 돌아갈 주소를 들고 첫 화면에 와서 가입, **Then** 지금처럼 `/@아이디?welcome=1`.

## Success Criteria *(mandatory)*

- **SC-001**: 수용 시나리오가 자동 테스트(`tests/e2e/login-next.spec.ts`, 주소 규칙은 `tests/unit/safe-next.test.ts`)로 통과한다.
