# Implementation Plan: 오류 화면 [다시 시도]와 루트 오류 화면

**Branch**: `020-error-retry` | **Date**: 2026-10-12 | **Spec**: [spec.md](./spec.md)

## Summary

DB·서버 변경 없음. `src/app/error.tsx`가 `reset` 대신 `retry`를 쓰고, `src/app/global-error.tsx`를 새로 만든다.

## 결정

1. Next 16.4의 오류 화면은 `reset`(상태만 지우고 받아 둔 화면을 다시 그림)과 `retry`(`router.refresh()` 뒤 다시 그림)를 받는다(`node_modules/next/dist/client/components/error-boundary.js`). 문서(`error.md`)도 대부분 `retry`를 쓰라고 한다.
2. `error.js`는 같은 단계의 `layout.js`를 감싸지 않는다. 루트 레이아웃의 헤더는 모든 요청에서 로그인 확인과 내 블로그를 DB에서 읽으므로 그 실패는 `global-error.tsx`가 받는다. 이 파일은 레이아웃을 대신하므로 `<html lang="ko"><body>`를 직접 그리고 `globals.css`를 불러온다.
3. 클라이언트 컴포넌트라 `metadata`를 쓸 수 없어 React `<title>`로 탭 제목을 단다(두 파일 모두).
4. 헤더가 실패한 뒤의 [처음으로]는 `<Link>`가 아니라 `<a href="/">`: 레이아웃부터 새로 불러온다.
5. 시험은 운영 데이터로 생길 수 없는 상태를 SQL로 잠깐 만든다: 날짜가 `-infinity`인 비공개 글(글 상세의 날짜 표시가 실패), 블로그 행이 없는 회원(헤더의 내 블로그 찾기가 실패). 원인을 SQL로 되돌린 뒤 [다시 시도]가 화면을 되살리는지 본다. 예전 `reset`이면 같은 오류가 남아 실패한다.

## Project Structure

```text
src/app/error.tsx, src/app/global-error.tsx
tests/e2e/error-page.spec.ts
```
