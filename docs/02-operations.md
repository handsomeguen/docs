# Blogville 운영 안내

운영 사이트: https://unqiuepcg.java21.net · 소스 코드: [handsomeguen/blogville](https://github.com/handsomeguen/blogville) · 버튼은 모두 그 저장소의 **Actions** 탭에 있어요.

## 한눈에

| 하고 싶은 것 | 누르는 곳 | 걸리는 시간 |
|---|---|---|
| 새 버전 배포 | `release/<버전>` PR을 `main`에 merge (자동 **deploy**) | 5~8분, 무중단 |
| 방금 배포한 버전 되돌리기 | Actions → **rollback** → Run workflow | 1분 안팎 |
| 서버·관리자 계정 상태 보기 | Actions → **status** → Run workflow | 30초 |
| 지금 DB 백업 | Actions → **backup** → Run workflow (매일 03:00 KST 자동) | 1분 안팎 |
| 사이트가 죽었는지 | 이슈 `🚨 사이트 응답 없음` (15분마다 **uptime**이 열고 닫음) | — |
| 관리자 비밀번호 바꾸기 | 시크릿 `ADMIN_PASSWORD` 바꾸고 **deploy** Run workflow | 5~8분 |

## 서버 구성

- 앞단: 도메인 nginx(학원 관리) → 우리 서버 8400번 포트.
- 8400: `bv8400-proxy` (scripts/proxy.mjs). `~/blogville/state/active-port`에 적힌 포트로 넘긴다.
- 앱: `bv8400-blue`(8401)·`bv8400-green`(8402) 중 하나가 서비스, 다른 하나는 멈춘 이전 버전(되돌리기용).
- DB: 학원 발급 PostgreSQL의 스키마 하나(`DB_SCHEMA`). 앱이 시작할 때 `scripts/migrate.mjs`로 마이그레이션.
- 서버의 도커는 다른 사람도 같이 쓴다. 우리 컨테이너·이미지는 모두 `bv8400-`로 시작하고, 스크립트는 그것만 건드린다.

## 일이 생겼을 때

1. **사이트가 안 열린다**: Actions → status. `서버 안에서 열기`가 200이면 앱은 정상이고 앞단 nginx(도메인) 문제 → 도메인 관리자에게. 200이 아니면 최근 로그를 보고, 방금 배포 뒤라면 rollback.
2. **배포 뒤 기능이 깨졌다**: rollback으로 이전 버전으로 돌린 뒤, `hotfix/<이름>` 브랜치로 고쳐 `main`에 PR (CONTRIBUTING.md).
3. **관리자 로그인이 안 된다**: status의 `관리자 계정 점검`에서 비밀번호 일치·잠금(5번 틀리면 5분)을 본다.
4. **데이터를 되살려야 한다**: 서버 `~/blogville/backups/<시각>/`의 `manifest.json`(표별 행 수)과 표마다 `.jsonl.gz`. 같은 마이그레이션까지 적용한 빈 스키마에 외래 키 순서대로 넣는다. 실제로 되살릴 일이 생기면 개발자에게 맡긴다.

## 시크릿 (blogville 저장소 Settings → Secrets and variables → Actions)

| 이름 | 쓰는 곳 |
|---|---|
| `SSH_ADDRESS`·`SSH_PORT`·`SSH_ID`·`SSH_PASSWORD` | 서버 접속 (deploy·status·rollback·backup) |
| `DB_ADDRESS`·`DB_PORT`·`DB_NAME`·`DB_SCHEMA`·`DB_USERNAME`·`DB_PASSWORD` | 앱의 DB 접속 |
| `APP_URL` (선택) | 사이트 주소. RSS·공유 미리보기·로그인 주소에 쓴다 (예: `https://unqiuepcg.java21.net`) |
| `ADMIN_USERNAME`·`ADMIN_PASSWORD` (선택) | 배포 때 관리자 계정·공지 블로그를 만들거나 비밀번호 갱신 |
| `FEEDBACK_BOT_TOKEN` (선택) | 버그·문의 자동 처리 API 열쇠. 클라우드 환경의 Network secret과 같은 값 |
| 변수 `SITE_URL` (선택) | uptime이 확인할 주소 |
