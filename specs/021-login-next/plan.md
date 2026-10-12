# Implementation Plan: 로그인 뒤 보던 글로 돌아오기

**Branch**: `021-login-next` | **Date**: 2026-10-12 | **Spec**: [spec.md](./spec.md)

## Summary

DB 변경 없음. 돌아갈 주소 규칙 `safeNext`를 순수 함수로 두고, 첫 화면(`/`)·로그인 액션·[로그인] 링크가 그것을 쓴다.

## 결정

1. `safeNext(value)`(`src/lib/safe-next.ts`): 문자열이고 2048자 이하, `/`로 시작하고 `//`로 시작하지 않으며 제어 문자·역슬래시가 없고, 가짜 기준 주소로 풀었을 때 다른 사이트가 아니면 정리한 `경로?검색#조각`을 돌려준다. 브라우저는 주소의 탭·줄바꿈을 지우고 역슬래시를 `/`로 읽어 `/\t/evil.com`·`/\evil.com`이 `//evil.com`이 되므로 막는다.
2. 서버가 두 번 거른다: 첫 화면이 `searchParams.next`를 걸러 로그인 폼 숨은 칸에 넣고, `signIn`이 폼 값을 다시 거른다(조작한 값). 통과하면 `redirect(next)`, 아니면 지금처럼 내 블로그.
3. 이미 로그인한 회원이 `/?next=…`를 열면 첫 화면이 바로 그 주소로 보낸다.
4. 글 상세가 `loginHref("/@주소/글ID")`를 공감·댓글에 넘긴다(댓글은 `#comments`).
5. 헤더는 레이아웃이라 주소를 모르므로 클라이언트 컴포넌트 `HeaderLoginLink`가 `usePathname()`으로 만든다(검색어는 붙이지 않는다). 글 상세는 요청마다 그리는 화면이라 `/@주소` 다시 쓰기와 서버·브라우저 주소가 같다.
6. 공감 안내에 44px 링크가 끼면 버튼의 설명이 `로그인 하면`으로 끊겨 읽혀, 버튼 설명은 숨은 `#like-login`에 따로 둔다.

## Project Structure

```text
src/lib/safe-next.ts, tests/unit/safe-next.test.ts
src/app/page.tsx, src/app/(auth)/actions.ts, src/components/auth/{AuthPanel,HeaderLoginLink}.tsx, src/app/layout.tsx
src/app/blog/[slug]/[postId]/page.tsx, src/components/post/{CommentSection,LikeButton}.tsx
tests/e2e/login-next.spec.ts (comments·likes·account-settings 시험의 링크 주소도 맞춤)
```
