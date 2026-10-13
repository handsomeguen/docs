# Implementation Plan: 구독·공유·검색 노출

**Branch**: `054-rss-share` | **Spec**: [spec.md](spec.md)

- 순수 함수: `src/lib/feed.ts` — `siteUrl`, `excerpt`, `firstImageKey`, `absolutizeHtml`, `escapeXml`, `buildRss`.
- 조회: `src/server/post.ts` — `listFeedPosts(blogId, 20)`(공개 글만), `listSitemapEntries()`.
- 경로: `src/app/blog/[slug]/rss/route.ts`(`/@:slug/rss` rewrite를 글 번호 주소보다 먼저), `src/app/sitemap.ts`, `src/app/robots.ts`.
- 메타: 루트 레이아웃 `generateMetadata`로 `metadataBase`, 블로그 홈·글 `generateMetadata`에 description·canonical·RSS alternate·openGraph·twitter. 비공개 글은 `openGraph: null`, `robots: noindex`.
- 화면: 블로그 옆 칸 `📡 RSS로 구독하기`(`data-rss`).
- 새 DB 구조 없음.
