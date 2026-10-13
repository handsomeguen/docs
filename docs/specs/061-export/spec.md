# Feature Specification: 내 블로그 내보내기

**Feature Branch**: `feature/export` (코드 PR #54, v1.4.0) · **Created**: 2026-10-13 · **Status**: Implemented
**Input**: 티스토리 '백업'에 해당하는 기능

## User Story
블로그 설정 `📦 내보내기` → `blogville-{주소}-{YYYYMMDD}.zip`: README·index.html·blog.json·글마다 HTML(엄격한 CSP)과 Markdown·첨부 파일·댓글·방명록(닉네임만, 회원 id 없음). 비공개·예약·블라인드 글도 표시와 함께. 내 블로그만(남은 404), 10분에 한 번.

## Success Criteria
- E2E `export.spec`(2: 내려받아 풀어서 확인, 권한), 단위 `export`(32).
