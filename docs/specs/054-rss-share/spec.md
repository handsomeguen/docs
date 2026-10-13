# Feature Specification: 구독·공유·검색 노출

**Feature Branch**: `054-rss-share` (코드: handsomeguen/blogville `054-rss-share`)

**Created**: 2026-10-13

**Status**: Implemented

**Input**: User description: "너가 서버를 구동하면서 블로그로서 아쉬운거나 없는기능이있으면 추가해줘"

**다루는 것**: 명세에 없던 블로그 기본 기능. 블로그를 직접 써 보니 티스토리 같은 블로그라면 당연히 있는 구독(RSS), 공유 미리보기, 검색엔진 노출이 없었다.

## User Scenarios & Testing *(mandatory)*

### User Story 1 - RSS로 구독하기 (Priority: P1) 🎯

1. 블로그 옆 칸 맨 아래 `📡 RSS로 구독하기`(새 탭)를 누르면 `/@주소/rss`가 열린다. RSS 리더에 블로그 주소만 넣어도 찾을 수 있게 블로그 홈·글에 `<link rel="alternate" type="application/rss+xml">`를 둔다.
2. RSS 2.0, 최신 **공개** 글 20개. 글마다 제목·주소·작성 시각·글쓴이 닉네임·카테고리·요약(300자)·본문 전체. 본문의 `/files/…` 같은 사이트 안 주소는 절대 주소로 바꾼다.
3. 없는 블로그는 404.

### User Story 2 - 공유 미리보기 (Priority: P1)

1. 공개 글 주소를 카카오톡·SNS에 붙이면 제목, 요약(본문 앞 150자), 첫 사진(있으면)이 미리보기로 보인다 (Open Graph `article`, 트위터 카드).
2. 블로그 홈은 블로그 이름과 소개(없으면 `{닉네임}님의 블로그`).
3. **비공개 글**은 주인이 열어도 미리보기를 내놓지 않고 `noindex`.

### User Story 3 - 검색엔진 노출 (Priority: P2)

- `/sitemap.xml`: 첫 화면, 마을 소식, 모든 블로그 홈, 공개 글(최신 49,000개).
- `/robots.txt`: 전체 허용, 회원 전용 화면(`/settings`, `/write`, `/admin`, 상점·마을 시설 등)과 `/api/`는 막고 sitemap 주소를 알린다.
- 글·블로그 홈에 canonical 주소.

## Success Criteria *(mandatory)*

- **SC-001**: `tests/unit/feed.test.ts`(요약, 대표 사진, 절대 주소, XML 처리, RSS 모양) 6개와 `tests/e2e/rss.spec.ts` 3개가 통과하고, 옆 칸·글 시험이 그대로 통과한다.
- 사이트 주소는 `BETTER_AUTH_URL`(배포의 `APP_URL`)을 쓴다. 도메인을 붙이면 `APP_URL` 시크릿을 그 주소로 바꿔야 RSS·미리보기 주소가 맞는다.
