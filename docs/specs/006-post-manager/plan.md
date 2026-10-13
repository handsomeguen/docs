# Implementation Plan: 글 관리와 주인 도구 막대

**Branch**: `006-post-manager` | **Date**: 2026-10-11 | **Spec**: [spec.md](./spec.md)

## Summary

새 표는 없다. `listBlogPosts`에 `visibility`·`pageSize`를 더하고 `countMyPosts`·`setPostVisibility`(`src/server/post.ts`)를 만들었다. 화면은 `src/app/settings/posts/page.tsx`(서버, 거르기는 GET 폼)와 줄 버튼 `PostRowActions`(클라이언트), Action은 `changeVisibility`·`removePost`(`src/app/settings/posts/actions.ts`). 블로그 홈 머리에 주인 도구 막대, 블로그 관리에 `#categories`와 [📝 글 관리].

## 결정

1. **카테고리 거르기 값**: 한 선택 상자에서 대분류·소분류를 함께 고르려고 `?cat=c{ID}`·`s{ID}`로 받는다.
2. **공개 바꾸기는 수정이 아님**: `updated_at`을 그대로 둬서 상세에 "수정됨"이 붙지 않는다.
3. **삭제 뒤 자리**: 글 상세의 삭제는 블로그 홈으로 가지만, 글 관리의 삭제는 목록에 머문다.

## Project Structure

```text
src/server/post.ts                         # listBlogPosts(visibility, pageSize), countMyPosts, setPostVisibility
src/app/settings/posts/{page,actions}.ts(x)
src/components/settings/PostRowActions.tsx
src/app/blog/[slug]/page.tsx               # 주인 도구 막대
src/app/settings/blog/page.tsx             # #categories, [📝 글 관리]
tests/e2e/manage.spec.ts
```
