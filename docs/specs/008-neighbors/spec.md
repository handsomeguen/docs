# Feature Specification: 이웃·마을 소식

**Feature Branch**: `008-neighbors`

**Created**: 2026-10-11

**Status**: Implemented

**Input**: User description: "merger권한을 줬으니 너가 알아서 쭉 진행해줘" (007 다음 순서)

**다루는 명세 요구사항**: SOC-04 (이웃 추가·취소, 이웃 새 글), SOC-05 (마을 소식), POST-05 (페이지 번호 `…` 생략)

**이번에 하지 않는 것**: 즐겨찾는 이웃(⭐, `is_favorite`)과 친구 목록 — 광장·마을 집(TOWN-04·TOWN-12)이 아직 없다. 그래서 이웃 새 글에 "즐겨찾는 이웃 글 먼저" 묶음도 없다. 광장 게시판·출석 창(`/feed?board=1`), 휴대폰 아래 탭, [🏘 마을 구경] 버튼, 작성자 줄의 캐릭터 그림.

**명세와 다른 점**: 광장이 없어 마을 소식으로 들어갈 곳이 없으므로 헤더에 `📋 마을 소식` 링크를 둔다 (휴대폰에서는 📋만).

## User Scenarios & Testing *(mandatory)*

### User Story 1 - 다른 블로그를 이웃으로 추가하고 취소한다 (Priority: P1) 🎯

회원이 남의 블로그 홈 정보 줄 오른쪽 [+ 이웃 추가](하늘색)를 누르면 [✓ 이웃](흰 바탕)이 되고 `이웃 N`이 1 는다. 다시 누르면 확인 창 없이 취소된다.

**Acceptance Scenarios**:

1. **Given** 회원이 남의 블로그 홈, **When** [+ 이웃 추가], **Then** 서버 처리 뒤 [✓ 이웃]·`aria-pressed="true"`·`이웃 1`. 버튼 높이 44px 이상.
2. **Given** 다시 누름, **Then** [+ 이웃 추가]·`이웃 0`, 확인 창 없음.
3. **Given** 내 블로그·방문자, **Then** 이웃 버튼이 없고 `이웃 N`만 보인다.
4. **Given** 나·없는 회원·65자·빈 값으로 조작, **Then** HTTP 200, 행이 생기지 않는다. 같은 요청 10번 동시에도 행은 1개 이하.
5. **Given** 회원 탈퇴(행 삭제), **Then** 그 회원의 이웃 관계가 함께 지워진다.

### User Story 2 - 이웃 새 글을 모아 본다 (Priority: P1)

`/feed/following`에서 내가 이웃 추가한 블로그의 공개 글을 최신순 8개씩 본다.

**Acceptance Scenarios**:

1. **Given** 이웃 추가, **Then** 추가 전에 쓴 공개 글도 나오고 비공개 글·남의 글은 안 나온다. 취소하면 빠진다.
2. **Given** 이웃이 없음 또는 `?page=999`, **Then** 두 줄 문구, "마을 소식"은 `/feed` 링크.
3. **Given** 방문자, **Then** `/`로 간다.

### User Story 3 - 마을 소식에서 모든 블로그의 새 글을 본다 (Priority: P2)

`/feed`는 모든 블로그의 공개 글을 최신순 8개씩, 카드 위 작성자 줄(닉네임 · 블로그 이름)과 함께 보여 준다. 오른쪽(휴대폰은 아래)에 인기 태그.

**Acceptance Scenarios**:

1. **Given** 9개 이상, **Then** 8개와 페이지 번호, 이상한 `?page=`는 1페이지.
2. **Given** 방문자, **Then** [🏘 마을 전체] 탭만, 회원은 [💛 이웃 새 글] 탭도.
3. **Given** 375px, **Then** 가로 스크롤 없음.

## Requirements *(mandatory)*

- **FR-001**: `follows` 기본 키 `(follower_id, followee_id)`, CHECK `follows_not_self_check`, 두 외래 키 `ON DELETE CASCADE`, 인덱스 `follows_followee_idx`.
- **FR-002**: 추가·취소는 서버가 정한다: 내 행을 지우고, 지운 게 없으면 `INSERT … SELECT … FROM blogs WHERE owner_id = 상대 ON CONFLICT DO NOTHING`.
- **FR-003**: 페이지 번호는 첫·끝·현재 앞뒤 2개, 사이가 비면 `…` (POST-05). 블로그 홈·태그별 글에도 같이 적용된다.

## Success Criteria *(mandatory)*

- **SC-001**: 수용 시나리오가 모두 자동 테스트로 통과한다.
