# Feature Specification: 글 읽기 편의

**Feature Branch**: `feature/reader-ux` (코드 PR #49, v1.3.0) · **Created**: 2026-10-13 · **Status**: Implemented

## User Stories
1. 날짜 옆 `약 N분 읽기`(공백 뺀 500자/분, 최소 1분).
2. 제목(h2·h3) 3개 이상이면 `📑 목차`(휴대폰은 접힘), 제목에 안정적인 id.
3. 코드 블록 색 입히기(서버 highlight.js, 13개 언어) + `[복사]`.
4. 본문 사진 크게 보기(`<dialog>`, Esc·바깥 클릭).
5. 공개 글 공유 막대: 링크 복사·공유하기(휴대폰)·X·페이스북. 비공개 글엔 없음.
6. 긴 글 `↑` 맨 위로.
7. 접근성: 글자 대비(`--accent` #2a7348), 글 카드 링크 이름 맞춤 → Lighthouse 접근성 100.

## Success Criteria
- E2E `reader.spec`(7), 단위 `reader`(27, script·onerror·style이 통과하지 못함).
