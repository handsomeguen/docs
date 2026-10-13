# Feature Specification: 임시 저장

**Feature Branch**: `009-drafts`

**Created**: 2026-10-11

**Status**: Implemented

**Input**: User description: "merger권한을 줬으니 너가 알아서 쭉 진행해줘" (008 다음 순서)

**다루는 명세 요구사항**: POST-08 (임시 저장)

**명세와 다른 점**: 발행 안내(`PublishNotice`)가 아직 없어서, 발행이 성공해 글쓰기 화면을 떠날 때(화면이 사라질 때) 임시 글을 지운다. 로그아웃은 버튼을 누를 때(`onClick`) 지운다.

## User Scenarios & Testing *(mandatory)*

### User Story 1 - 쓰던 새 글이 브라우저에 남는다 (Priority: P1) 🎯

`/write`에서 입력이 2초 멈추면 이 브라우저에 회원마다 1개 저장하고, 다시 열면 불러올지 묻는다.

**Acceptance Scenarios**:

1. **Given** 새 글 화면, **Then** 아래에 `작성 중인 글은 이 브라우저에 자동 저장됩니다.`, 입력 뒤 2초면 `임시 저장됨 HH:mm`(한국 시간). 키는 `blogville:draft:{회원 ID}` 1개.
2. **Given** 임시 글이 있음, **When** `/write`, **Then** `작성 중이던 글이 있어요. 불러올까요?` → [확인]이면 제목·본문(서식)·공개 설정·태그·카테고리가 채워진다. 지워진 카테고리는 비운다.
3. **Given** [취소], **Then** 빈 화면, 임시 글은 다음 저장 때까지 남는다.
4. **Given** 제목·본문이 비고 태그만, **Then** 저장하지 않는다.
5. **Given** 발행 실패, **Then** 남는다. **Given** 2초 안에 발행 성공, **Then** 지워지고 다시 묻지 않는다.
6. **Given** 수정 화면, **Then** 저장도 상태 줄도 없다.
7. **Given** [로그아웃], **Then** 그 회원의 임시 글이 지워진다. 다른 회원으로 로그인하면 묻지 않는다.
8. **Given** 브라우저 저장소가 막힘, **Then** 글쓰기·발행이 그대로 되고 화면 오류가 없다.

## Requirements *(mandatory)*

- **FR-001**: DB에 쓰지 않는다. `src/lib/draft.ts`의 `readDraft`·`writeDraft`·`clearDraft`는 모두 `try/catch`.
- **FR-002**: 불러온 값의 형식이 틀리면 버린다 (번호는 양의 정수, 공개 설정은 `private`가 아니면 `public`, 글자 칸은 문자열). 발행할 때는 서버가 POST-01 규칙과 정화를 다시 적용한다.

## Success Criteria *(mandatory)*

- **SC-001**: 수용 시나리오가 모두 자동 테스트로 통과한다.
