# Feature Specification: 신고하기 · 스크랩

**Feature Branch**: `feature/report-bookmarks` (코드: handsomeguen/blogville PR #56, v1.4.0)
**Created**: 2026-10-13 · **Status**: Implemented
**Input**: "블로그의 부족한 점을 찾아 11시까지 업그레이드" (네이버 블로그·티스토리 참고)

## User Stories
1. **🚨 신고 (P1)**: 회원은 볼 수 있는 남의 글·댓글·답글·방명록을 사유(스팸·광고 / 욕설·비방 / 음란·불쾌 / 개인정보 노출 / 기타+설명 1~300자)와 함께 신고한다. 같은 대상은 한 번(`이미 신고했어요`), 한 시간에 10건.
2. **관리자 신고 처리 (P1)**: `/admin/reports`(탭 `🚨 신고`, 대시보드 처리 대기 수)에서 대상별로 묶어 보고 블라인드(글)·삭제·기각. 처리하면 그 대상의 신고가 모두 닫히고 관리 기록에 남는다.
3. **🔖 스크랩 (P2)**: 공개 글을 `🔖 스크랩`하고 `/bookmarks`에서 최신순으로 모아 본다. 비공개가 된 글은 목록에서 빠진다.

## Success Criteria
- E2E `reports.spec`(5)·`bookmarks.spec`(3), 단위 `report-rules`.
- v1.4.0 배포 전 점검에서 설명 없는 신고가 `기타: NULL`로 보이던 것을 고침(DB에서 NULL을 빼고 모음).
