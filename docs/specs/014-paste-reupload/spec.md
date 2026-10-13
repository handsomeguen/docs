# Feature Specification: 내 다른 글 첨부 붙여 넣기

**Feature Branch**: `014-paste-reupload`

**Created**: 2026-10-12

**Status**: Implemented

**Input**: User description: "계속 진행해줘" (013 다음 순서)

**다루는 명세 요구사항**: POST-07 흐름 7 (POST-09 파일 카드도 같음). 002 첨부 단계에서 미뤘던 것.

**명세와 다른 점**: 프로필 사진이 아직 없어 "프로필 사진을 붙여 넣으면 다시 올리기"는 없다.

## User Scenarios & Testing *(mandatory)*

### User Story 1 - 다른 글의 사진·파일을 복사해 새 글에 붙여 넣는다 (Priority: P1) 🎯

**Acceptance Scenarios**:

1. **Given** 이 사이트의 `/files/키`(전체 주소 포함) 사진·파일 카드가 든 HTML을 붙여 넣음, **Then** 서버에 물어서 넣는다: 내 것이고 안 붙었거나 이 글 것은 **그대로**, 내 다른 글 것은 **새 키로 다시 올려**(원래 글의 첨부는 그대로), 남의 것·없는 키는 **넣지 않는다**. 글자는 함께 들어간다.
2. **Given** 서버가 확인하는 동안, **Then** `붙여 넣는 중...`, 다시 올리는 동안 `올리는 중... (1/2)`이 보이고 [🖼 사진]·[📎 파일]·발행 버튼(`첨부를 올리는 중...`)이 막힌다.
3. **Given** 다시 올리기 실패, **Then** 그 첨부만 빼고 `파일 이름: 올리지 못했어요. 다시 시도해 주세요`.
4. **Given** 다시 올린 파일 카드, **Then** 원래 이름으로 내려받는다.
5. **Given** 저장 요청을 직접 고쳐 남의 것·다른 글 것을 넣음, **Then** 저장할 때 빠진다 (002의 서버 검사 그대로).

## Success Criteria *(mandatory)*

- **SC-001**: 수용 시나리오가 자동 테스트(`tests/e2e/paste-reupload.spec.ts`, `attachment-access.spec.ts`)와 단위 테스트(`pasteAction`)로 통과한다.
