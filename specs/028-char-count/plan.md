# Implementation Plan: 글쓰기 글자 수

**Branch**: `028-char-count` | **Date**: 2026-10-12 | **Spec**: [spec.md](./spec.md)

## Summary

DB 변경 없음. 서버의 `htmlToText(sanitizePostHtml(html))`(sanitize-html)를 브라우저에서 쓰지 않고, 같은 결과를 내는 문자열 셈 `postTextLength`를 둔다.

## 결정

1. `src/lib/text-length.ts`: 서버와 같은 줄 끝 규칙(`</p|h2|h3|li|blockquote|pre>`·`<br>` 뒤 줄바꿈) → 태그 제거(따옴표 속 `>` 무시) → 엔티티 풀기(`&amp;` `&lt;` `&gt;` `&quot;` `&apos;` `&nbsp;`, 숫자 엔티티) → 줄 끝 공백 제거·앞뒤 공백 제거 → `length`. sanitize-html이 글자를 다시 이스케이프하고 서버가 푸는 과정은 서로 상쇄돼 푼 글자와 같다.
2. 단위 시험이 에디터 모양 HTML(빈 문단, br, 공백, nbsp, 엔티티, 글자로 쓴 엔티티, 이모지, 제목, 목록, 인용, 코드, 링크, 구분선, 사진, 따옴표·`>`가 든 파일 카드 등)에서 두 셈을 비교한다.
3. `PostForm`은 `contentHtml`에서 `useMemo`로 세어 제출 버튼 줄 왼쪽에 보인다(`data-char-count`).

## Project Structure

```text
src/lib/text-length.ts, src/components/post/PostForm.tsx
tests/unit/text-length.test.ts, tests/e2e/char-count.spec.ts
```
