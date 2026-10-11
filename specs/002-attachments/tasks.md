# Tasks: 사진·파일 첨부

**Input**: [spec.md](./spec.md), [plan.md](./plan.md), [data-model.md](./data-model.md), [contracts/routes.md](./contracts/routes.md)

## Phase 1: Setup

- [x] T001 `@tiptap/extension-image` 설치, `.gitignore`에 `/storage/`, `.env.example`에 `UPLOAD_DIR`

## Phase 2: Foundational

- [x] T002 `src/db/schema.ts`에 `attachments` + `drizzle/0002_attachments.sql`(CHECK·인덱스·트리거)
- [x] T003 [P] `src/lib/attachments.ts`: 형식 표, 한도, 문구, `attachmentProblem`, `sniffImage`, `cleanFileName`, `formatBytes`, `contentDisposition`, `attachmentAccess`, `extractFileKeys`
- [x] T004 [P] `tests/unit/attachments.test.ts`: 위 함수들
- [x] T005 [P] `src/server/storage.ts`: `saveFile`, `openFile`, `deleteFile`, `listFiles`
- [x] T006 `src/server/attachments.ts`: `createAttachment`, `findReadableAttachment`, `lockLinkable`, `syncPostAttachments`

## Phase 3: US1 사진 넣기 (P1) 🎯

- [x] T007 [US1] `src/app/api/uploads/route.ts`: Origin → 로그인 → 크기 머리글 → 형식·크기·매직 넘버 → 저장 → 201
- [x] T008 [US1] `src/app/files/[key]/route.ts`: 권한, 304, 머리글
- [x] T009 [US1] `src/lib/sanitize.ts`: `sanitizePostHtml(html, known)`에 `img` 허용 + `tests/unit/sanitize.test.ts`
- [x] T010 [US1] `src/server/post.ts` savePost: 잠그기 → 정화 → 저장 → 붙이기
- [x] T011 [US1] 에디터: Image 확장, [🖼 사진] 버튼, 붙여 넣기·끌어다 놓기, `useUpload`(차례로 올리기, 진행 표시, 오류 목록 [✕]), 올리는 중 발행 막기
- [x] T012 [US1] `globals.css`: 본문 사진 폭 맞춤, 에디터에서 고른 사진 노란 테두리
- [x] T013 [US1] `tests/e2e/attachments.spec.ts`: 두 장 고르기, 붙여 넣기, 상세·수정 화면, 거부 문구, 직접 요청 403/401/411/415, `data:` 없음, 375px

## Phase 4: US3 누가 여나 (P1)

- [x] T014 [US3] `tests/e2e/attachment-access.spec.ts`: 붙지 않음/공개/비공개, 304·404, 섞어 보낸 키 빠짐, 떨어짐

## Phase 5: US2 파일 카드 (P2)

- [x] T015 [US2] `src/components/editor/FileCard.ts` 노드 + [📎 파일] 버튼
- [x] T016 [US2] 정화: `a[data-file]` 허용, DB 값으로 다시 쓰기 + 단위 테스트
- [x] T017 [US2] 카드 CSS (아이콘·이름·크기, 375px)
- [x] T018 [US2] E2E: 두 개 끌어다 놓기, 원래 이름 내려받기, EXE·HTML 415, 30MB 413, 조작한 이름, 사진을 [📎]로

## Phase 6: US4 정리 (P3)

- [x] T019 [US4] `cleanupAttachments` + `scripts/cleanup-posts.ts` + `npm run posts:cleanup`
- [x] T020 [US4] E2E 또는 통합 테스트: 하루 지난 것만 지움, `--dry-run`

## Phase 7: Polish

- [x] T021 README·quickstart, 요구사항 문서 메모(다시 올리기 미룸)
- [x] T022 typecheck·lint·unit·E2E·build 통과
