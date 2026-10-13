# Plan: 내보내기
- `GET /settings/export`, `src/server/export.ts`, zip은 `fflate@0.8.2`로 읽는 만큼 만들고 첨부는 조각씩. 마크다운은 허용 태그만 다루는 작은 변환기. DB 변경 없음.
