# Feature Specification: 서버 오류 기록

**Feature Branch**: `feature/error-log` (코드 PR #61, v1.5.0) · **Created**: 2026-10-13 · **Status**: Implemented

## User Story
운영에서 처리되지 못한 서버 오류가 `error_logs`에 남고, 관리자 `⚠️ 오류` 탭(`/admin/errors`)에서 24시간·7일·전체로 본다. 같은 경로+digest는 1시간 안이면 한 줄로 묶어 횟수를 센다. `해결함`은 관리 기록에 남긴다. 대시보드에 최근 24시간 오류 수. `시험 오류 만들기`(관리자만).

## Success Criteria
- E2E `errors.spec`, 단위 `error-log-rules`. 쿼리 문자열·비밀 값은 남기지 않는다. 30일 지난 기록은 1% 확률로 정리.
