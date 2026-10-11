# Feature Specification: 관리자

**Feature Branch**: `012-admin`

**Created**: 2026-10-11

**Status**: Implemented

**Input**: User description: "merger권한을 줬으니 너가 알아서 쭉 진행해줘" (011 다음 순서)

**다루는 명세 요구사항**: AUTH-08 (관리자)

**명세와 다른 점**: 출석(GAME-04)이 없어서 통계 카드는 **4개**(가입 계정 · 전체 글 · 댓글 · 오늘 새 글)다. 공지 블로그에 캐릭터·초원(게임)은 없다. 광장 ☰ 메뉴가 없어 [👑 관리자]는 헤더에만 있다. 좁은 화면에서는 헤더 메뉴가 다음 줄로 내려간다.

## User Scenarios & Testing *(mandatory)*

### User Story 1 - 관리자가 사이트를 살피고 글을 지운다 (Priority: P1) 🎯

**Acceptance Scenarios**:

1. **Given** 방문자·일반 회원, **Then** `/admin`은 404, 헤더에 👑가 없다. 삭제 요청을 일반 회원 쿠키로 보내도 글이 그대로다.
2. **Given** 관리자, **Then** 헤더 [👑 관리자] → `/admin`: 통계 카드 4개, `최근 가입 (20명)`(아이디 `(아이디)` · 닉네임 · 블로그 링크 · 권한 · 가입 시각), `최근 30일 글`(최대 30개, 공개 글만 링크, 비공개는 🔒).
3. **Given** [삭제], **Then** 확인 창 `'{제목}' 글을 삭제할까요?` → 확인하면 남의 글도 지운다.
4. **Given** 남의 댓글, **Then** 관리자는 글 화면에서 지울 수 있다 (SOC-01, 이미 있음).
5. **Given** 375px, **Then** 가로 스크롤 없음, [삭제] 44px 이상.

### User Story 2 - 관리자 계정과 공지 블로그를 만든다 (Priority: P2)

`npm run admin:create`(`.env.local`의 `ADMIN_USERNAME`, `ADMIN_PASSWORD` 12~64자)가 관리자와 공지 블로그(`/@notice`, `Blogville 공지사항`, 대분류 `공지`, 닉네임 `관리자`)를 만든다. 다시 실행하면 비밀번호와 권한만 갱신한다.

## Success Criteria *(mandatory)*

- **SC-001**: 수용 시나리오가 자동 테스트로 통과한다 (스크립트는 수동 확인).
