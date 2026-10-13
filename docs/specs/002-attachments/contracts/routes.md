# Contract: 첨부 주소

## POST /api/uploads

요청: `multipart/form-data`, 칸 `file` 하나.

| 순서 | 상황 | HTTP | error |
|---|---|---|---|
| 1 | `Origin` 없음·`null`·이 사이트와 다름 | 403 | 잘못된 요청이에요 |
| 2 | 로그인 안 함 | 401 | 로그인한 회원만 올릴 수 있어요 |
| 3 | `Content-Length` 없음 | 411 | 잘못된 요청이에요 |
| 4 | `Content-Length` > 31MB | 413 | 파일은 30MB까지 올릴 수 있어요 |
| 5 | `file` 칸 없음 | 400 | 잘못된 요청이에요 |
| 6 | 받지 않는 확장자 | 415 | 올릴 수 없는 형식이에요. 사진은 PNG·JPG·GIF·WEBP, 파일은 PDF·한글·워드·엑셀·파워포인트·키노트·ZIP·TXT·CSV·MD·JSON·MP3·MP4만 올릴 수 있어요 |
| 7 | 빈 파일 | 400 | 빈 파일은 올릴 수 없어요 |
| 8 | 사진 10MB 초과 / 파일 30MB 초과 | 413 | 사진은 10MB까지 올릴 수 있어요 / 파일은 30MB까지 올릴 수 있어요 |
| 9 | 사진 확장자인데 앞부분이 그 형식이 아님 | 415 | 사진 파일이 아니에요. PNG·JPG·GIF·WEBP 사진만 올릴 수 있어요 |

성공 201: `{ key, url: "/files/키", kind, name, size, sizeLabel, ext }`. 오류: `{ error }`.

## GET /files/[키]

| 상황 | 응답 |
|---|---|
| 키 형식이 아님·행 없음·볼 수 없음 | 404 `파일을 찾을 수 없어요` (text/plain) |
| `If-None-Match: "키"`이고 볼 수 있음 | 304 |
| 볼 수 있음 | 200, 파일 내용 |

머리글: `Content-Type`(DB mime), `Content-Length`, `X-Content-Type-Options: nosniff`, `Content-Security-Policy: default-src 'none'; img-src 'self'; media-src 'self'; sandbox`, `Cache-Control: private, no-cache`, `ETag: "키"`, `Content-Disposition`: 사진 `inline`, 파일 `attachment; filename="ASCII 대체"; filename*=UTF-8''인코딩`.

## savePost (변경)

- 정화 전에 본문의 `/files/키`를 모아 붙일 수 있는 첨부를 잠그고(`known`), 정화가 `known`에 없는 사진·카드를 뺀다.
- 저장 뒤 `syncPostAttachments(글ID, 남은 키)`.
- 사진·카드만 있는 본문은 `본문을 적어 주세요`.

## 본문 HTML

- 사진: `<img src="/files/키" alt="">`
- 파일 카드: `<a href="/files/키" data-file="" data-name="원래 이름" data-size="바이트" data-size-label="1.2MB" data-ext="pdf" aria-label="원래 이름 내려받기 (1.2MB)"></a>`
