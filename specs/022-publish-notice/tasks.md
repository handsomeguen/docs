# Tasks: 새 글 발행 안내

- [x] T001 [US1] 새 글만 `?new=1`, 수정은 그대로
- [x] T002 [US1·US2] `showsPublishNotice`와 단위 시험, 글 상세의 `PublishNotice`(노란 `role="status"`, 주소의 `?new` 지우기)
- [x] T003 `tests/e2e/publish-notice.spec.ts`, README·요구사항 문서 메모, 전체 검사
- [x] T004 검토: 뒤로·앞으로 가기에 안내가 다시 붙지 않게(처음 붙을 때 `useSearchParams`로 정함), 주소 정리는 한 틈 미뤄 Next가 감싼 `replaceState`로. 시험은 화면 안 링크로 옮긴 뒤 뒤로·앞으로 가기
