# Implementation Plan: 방명록

**Branch**: `056-guestbook` | **Spec**: [spec.md](spec.md)

- DB: 새 표 `guestbook_entries`(`blog_id` → blogs, `author_id` → users 둘 다 ON DELETE CASCADE, `content` 1~1000자 CHECK, `is_secret`, `edited_at`, 주인 답글 `reply`·`replied_at`·`reply_edited_at`과 묶음 CHECK, 인덱스 `(blog_id, created_at desc, id desc)`·`author_id`). `notifications`에 `guestbook_entry_id`(→ 방명록 글 CASCADE, 인덱스), 종류 CHECK에 `guestbook`·`guestbook_reply`, 모양 CHECK에 "방명록 알림은 글 없이 방명록 글". 마이그레이션 `0027_guestbook` (표·인덱스·외래 키·CHECK만).
- 순수 함수: `src/lib/guestbook-rules.ts` — `isEntryHidden`(`canSeeSecret` 재사용), `guestbookPath`, `GUESTBOOK_PAGE_SIZE = 10`, 문구 `GUESTBOOK_ERRORS`·`GUESTBOOK_SECRET_PLACEHOLDER`. `src/lib/notifications.ts` — `GuestbookKind`, `guestbookNotificationText`, `guestbookNotificationHref`, `isGuestbookKind`.
- 서버: `src/server/guestbook.ts` — `getGuestbook`(가려지는 글·답글 내용은 빈 글자, 권한 플래그 계산), `countGuestbook`, `createEntry`(주인 회원 행 → 블로그 행 잠금 뒤 저장과 알림, `lockMemberRow`를 comments.ts에서 꺼내 씀), `editEntry`(UPDATE 조건 `author_id = 나`), `removeEntry`(관리자·작성자·주인 조건 DELETE), `setReply`(주인만, 작성자 회원 행 → 글 행 FOR UPDATE, 처음일 때만 알림), `removeReply`. 알림 `notifyGuestbook`, 알림 조회는 방명록 글을 거쳐 블로그를 잇는다.
- Action: `src/app/blog/guestbook-actions.ts` — `requireMember` → `parseId` → `checkComment` → 서버 함수 → `revalidatePath("/", "layout")`.
- 화면: `src/app/blog/[slug]/guestbook/page.tsx`, `src/components/blog/GuestbookSection.tsx`(댓글의 `CommentForm`을 꺼내 `secretLabel`로 재사용). 주소 `next.config.ts` rewrite `/@:slug/guestbook`(글 번호 주소보다 먼저). 옆 칸 맨 위 `📖 방명록 N`(`getBlogSidebar`가 수를 함께), 주인 도구 막대 `📖 방명록 N`. 알림함·마을 ☰ 메뉴 문구.
