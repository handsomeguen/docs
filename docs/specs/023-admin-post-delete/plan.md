# Implementation Plan: 관리자가 글 화면에서 지우기, 공지 블로그 주소

**Branch**: `023-admin-post-delete` | **Date**: 2026-10-12 | **Spec**: [spec.md](./spec.md)

## Summary

DB 변경 없음. 글 상세 아래쪽에 관리자 [삭제]를 두고, 지운 뒤 그 블로그 홈으로 보내는 액션을 더한다. 블로그 주소 바꾸기는 명세 순서대로 "지금과 같으면 성공"을 예약어 검사보다 먼저 보고, 관리자의 `notice`를 허용한다.

## 결정

1. 글 상세 footer: 보는 사람이 관리자이고 주인이 아니면 `AdminDeleteButton`(`leave`)을 그린다. 주인이면 지금처럼 [수정]·[삭제]만. 비공개 글은 관리자에게도 404라 버튼이 나올 일이 없다.
2. `adminDeletePostAndLeave`(`src/app/admin/actions.ts`): `requireAdmin()` → `parseId` → `DELETE posts … RETURNING blog_id` → 그 블로그 주소 → `revalidatePath("/", "layout")` → `redirect("/@주소")`. 지워진 글이 없으면(이미 지워짐) 다시 그리기만 한다. 일반 회원이 보내면 `/admin`처럼 404로 끝나고 아무것도 지우지 않는다.
3. `AdminDeleteButton`에 `leave` prop: 켜면 위 액션과 글 상세의 `btn btn-danger` 모양을 쓴다. 확인 창 문구는 `/admin`과 같은 `'{제목}' 글을 삭제할까요?`.
4. 주소 규칙은 순수 함수 `mayUseSlug(주소, 역할)`(`src/lib/names.ts`): 예약어가 아니거나, 관리자의 `notice`. `updateBlogSlug(ownerId, rawSlug, role)`은 형식 → 트랜잭션 안에서 지금과 같으면 성공 → `mayUseSlug` → 겹침 검사 순서. `settings/blog/actions.ts`가 `member.role`을 넘긴다.

## Project Structure

```text
src/app/admin/actions.ts (adminDeletePostAndLeave), src/components/admin/AdminDeleteButton.tsx (leave)
src/app/blog/[slug]/[postId]/page.tsx (관리자 [삭제])
src/lib/names.ts (mayUseSlug), src/server/blog.ts (updateBlogSlug 순서), src/app/settings/blog/actions.ts (role)
tests/e2e/admin.spec.ts, tests/unit/names.test.ts
```
