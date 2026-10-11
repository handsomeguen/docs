# Feature Specification: 회원 탈퇴

**Feature Branch**: `011-withdraw`

**Created**: 2026-10-11

**Status**: Implemented

**Input**: User description: "merger권한을 줬으니 너가 알아서 쭉 진행해줘" (010 다음 순서)

**다루는 명세 요구사항**: AUTH-06 (회원 탈퇴)

**참고**: 코인·경험치·아이템·소셜 연결은 아직 없어서 지울 것도 없다. 안내 문구는 명세 그대로 둔다. 탈퇴해도 그 브라우저의 임시 글(POST-08)은 남는다 (명세의 열린 질문 그대로).

## User Scenarios & Testing *(mandatory)*

### User Story 1 - 비밀번호를 다시 적고 탈퇴한다 (Priority: P1) 🎯

`/settings/account`(블로그 관리의 [👤 내 정보]) 맨 아래 `회원 탈퇴`에서 안내를 읽고 비밀번호를 다시 적은 뒤 빨간 [회원 탈퇴]를 누른다. 확인 창은 없다.

**Acceptance Scenarios**:

1. **Given** 빈 칸, **Then** `비밀번호를 적어 주세요`. **Given** 틀린 비밀번호, **Then** `비밀번호가 맞지 않아요`. 둘 다 비밀번호 칸을 비우고 아무것도 지우지 않는다.
2. **Given** 맞는 비밀번호, **Then** 한 트랜잭션으로 지우고 로그인 쿠키를 지운 뒤 `/`로 간다.
3. **Then** 회원·로그인 수단·세션·프로필·블로그·글·이웃·공감·댓글·답글·로그인 실패 기록이 0행.
4. **Then** 혼자인 내 댓글과 내 답글은 지우고, 남의 답글이 달린 내 댓글은 작성자·내용 없이 `삭제된 댓글이에요` 자리만 남는다. 남의 답글은 그대로. 화면에 내 닉네임·원문이 없다.
5. **Then** 그 아이디로 로그인할 수 없고 블로그 주소는 404, 같은 아이디로 다시 가입할 수 있다.

## Requirements *(mandatory)*

- **FR-001**: `deleteMember`(`src/server/account.ts`): 회원 행 잠금 → 댓글·답글 정리 → `login_attempts` 삭제 → `users` 삭제(나머지는 외래 키 CASCADE).
- **FR-002**: 비밀번호는 `accounts`(provider `credential`)의 해시로 확인한다.

## Success Criteria *(mandatory)*

- **SC-001**: 수용 시나리오가 모두 자동 테스트로 통과한다.
