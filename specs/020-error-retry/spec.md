# Feature Specification: 오류 화면 [다시 시도]와 루트 오류 화면

**Feature Branch**: `020-error-retry`

**Created**: 2026-10-12

**Status**: Implemented

**Input**: User description: "오후 1시까지 쉬지않고 계속 진행해줘" (019 다음 순서, `plans/gap-survey-2026-10-12.md` 7번)

**다루는 명세 요구사항**: NF-19 오류 문구는 한국어, 6장 화면 목록의 오류 화면 `잠깐 문제가 생겼어요`, POST-01 예외 흐름의 Blogville 오류 화면.

**명세와 다른 점**: 명세(POST-01 예외 흐름)의 두 번째 단추는 [광장으로 돌아가기]지만 광장이 아직 없어 지금처럼 [처음으로]다. 헤더까지 실패한 경우의 화면은 명세에 따로 없어, 같은 🚧 화면을 헤더 없이 보인다.

## User Scenarios & Testing *(mandatory)*

### User Story 1 - [다시 시도]가 정말 다시 시도한다 (Priority: P1) 🎯

전에는 [다시 시도]가 받아 둔 화면을 다시 그릴 뿐이라(`reset`), DB가 잠깐 끊긴 것처럼 원인이 사라진 뒤에도 같은 오류 화면이 남았다.

**Acceptance Scenarios**:

1. **Given** 글 상세를 그리다 실패, **Then** 헤더는 그대로 두고 🚧 `잠깐 문제가 생겼어요`, 탭 제목 `잠깐 문제가 생겼어요 | Blogville`, [다시 시도] [처음으로](44px 이상).
2. **Given** 원인이 사라진 뒤 [다시 시도], **Then** 서버에서 화면을 다시 받아 같은 주소의 글이 보인다.

### User Story 2 - 헤더까지 실패해도 한국어 화면 (Priority: P1)

전에는 루트 레이아웃(헤더)이 실패하면 Next 기본 영어 화면이 나왔다(`global-error`가 없었다).

1. **Given** 헤더를 그리다 실패(로그인 확인·내 블로그 찾기 등), **Then** 헤더 없이 같은 🚧 화면, `<html lang="ko">`, 탭 제목, Blogville 단추 모양.
2. **Given** 원인이 사라진 뒤 [다시 시도], **Then** 헤더와 원래 화면이 돌아온다.
3. **Given** [처음으로], **Then** `/`를 새로 불러온다(레이아웃부터 다시).

## Success Criteria *(mandatory)*

- **SC-001**: 수용 시나리오가 자동 테스트(`tests/e2e/error-page.spec.ts`)로 통과한다.
