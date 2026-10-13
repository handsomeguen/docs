# Feature Specification: 에디터 도구 모음 정리

**Feature Branch**: `019-editor-toolbar`

**Created**: 2026-10-12

**Status**: Implemented

**Input**: User description: "오후 1시까지 쉬지않고 계속 진행해줘" (018 다음 순서, `plans/gap-survey-2026-10-12.md` 6번)

**다루는 명세 요구사항**: POST-01 에디터 도구(순서, `U` 밑줄·`S` 취소선·`↶` 되돌리기·`↷` 다시 실행, 마우스를 올리면 이름, 고른 도구는 검은 배경, 링크 창 규칙), POST-01 삭제 확인 창 문구, NF-18 키보드 초점이 보일 것.

**명세와 다른 점**: 없음. 글 관리(`/settings/posts`)의 [삭제] 확인 창도 글 상세와 같은 문구로 맞췄다(명세에 따로 정한 문구가 없다). 함께 고친 것: 글·댓글·카테고리·관리자 [삭제]는 확인 창을 폼 `onSubmit`에서 물어서, 화면이 준비되기 전 클릭이나 자바스크립트 없는 제출이면 확인 없이 지워졌다(018 검토에서 찾은 같은 틈). 확인을 폼 액션 안으로 옮겼다(`confirmFirst`). 링크 창에 `naver.com`처럼 scheme 없이 넣으면 `https://`를 붙이고, 저장하면 정화가 지워 죽은 링크가 되는 주소(`tel:` 등, POST-01 저장 규칙 밖)는 `링크로 넣을 수 없는 주소예요. https://로 시작하는 주소를 넣어 주세요.`를 알리고 넣지 않는다(명세에 없는 문구).

## User Scenarios & Testing *(mandatory)*

### User Story 1 - 명세의 도구를 명세 순서로 쓴다 (Priority: P1) 🎯

**Acceptance Scenarios**:

1. **Given** 글쓰기 화면, **Then** 도구가 `H2 H3 | B I U S | • 1. ❝ </> ― 🔗 | ↶ ↷ | 🖼 사진 📎 파일` 순서이고 `|` 자리에 구분선이 있다. 마우스를 올리면 `큰 제목`·`글머리 목록` 같은 이름이 뜬다. 모두 44px 이상.
2. **Given** 커서 자리에 적용된 도구, **Then** 검은 바탕(#23201b)·흰 글씨, `aria-pressed="true"`.
3. **Given** 글자를 고르고 [U]·[S], **Then** 발행한 글에 `<u>`·`<s>`로 남는다.
4. **Given** 처음 연 에디터, **Then** [↶]·[↷]는 눌리지 않는다. 글자를 치면 [↶]로 되돌리고 [↷]로 다시 살린다.

### User Story 2 - 링크 창 (Priority: P1)

1. **Given** [🔗], **Then** 창 문구 `링크 주소 (비우면 링크 해제)`, 기본값 `https://`(이미 링크면 그 주소).
2. **Given** 비우거나 `https://`만 남기고 확인, **Then** 링크가 생기지 않는다(링크 위였으면 그 링크 전체가 풀린다).
3. **Given** `naver.com`처럼 scheme 없이 넣음, **Then** `https://naver.com` 링크. `tel:`·`ftp:`·`javascript:`처럼 저장하면 사라지는 주소는 `링크로 넣을 수 없는 주소예요. https://로 시작하는 주소를 넣어 주세요.`를 알리고 넣지 않는다.

### User Story 3 - 초점과 좁은 화면 (Priority: P2)

1. **Given** Tab으로 본문 칸에 들어간다, **Then** 에디터를 감싼 상자에 3px 초점선이 보인다.
2. **Given** 375px, **Then** 도구 모음이 여러 줄로 감기고 가로 스크롤이 없다.

### User Story 4 - 글 삭제 확인 창 (Priority: P3)

1. **Given** 내 글 상세나 글 관리의 [삭제], **Then** `이 글을 삭제할까요? 댓글과 공감도 함께 지워져요.`
2. **Given** 자바스크립트가 없거나 화면이 준비되기 전, **Then** [삭제]를 눌러도 확인 없이 지워지지 않는다.

## Success Criteria *(mandatory)*

- **SC-001**: 수용 시나리오가 자동 테스트(`tests/e2e/editor.spec.ts`, 글 관리 삭제 문구는 `tests/e2e/manage.spec.ts`)로 통과한다. 화면 준비 전 클릭은 자바스크립트를 끈 브라우저로 대신 확인한다(둘 다 서버가 그린 폼만 있는 상태).
