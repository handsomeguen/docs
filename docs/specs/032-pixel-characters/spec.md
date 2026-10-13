# Feature Specification: 도트 캐릭터 고르기

**Feature Branch**: `032-pixel-characters`

**Created**: 2026-10-12

**Status**: Implemented

**Input**: User description: "계속 진행해줘" + 결정 카드 "단순 도트"(16×16쯤 도트를 코드로 직접 그림)

**다루는 명세 요구사항**: GAME-01 기본 캐릭터 받기, BLOG-04 블로그 홈 프로필(사진이 없으면 캐릭터 얼굴), GAME-02 헤더의 캐릭터, ERD 3.10 `items`·`user_items`·`profiles.character_item_id`.

**명세와 다른 점**:
- 아이템은 기본 캐릭터 2종(남자 주민 `char_boy`·여자 주민 `char_girl`)만 있다. 기본 배경(초원)·기본 가구·우리 집 칸은 마을·꾸미기 단계에서 더한다(`items.type`은 지금 `character`만 허용).
- 그림은 SVG 파일이 아니라 `src/lib/art/characters.ts`의 16×16 글자 칸을 `PixelCharacter`가 사각형으로 그린다.
- 캐릭터가 생기기 전에 가입한 회원과 관리자·데모 계정은 남자 주민(데모는 여자 주민)을 받아 장착했다(마이그레이션·스크립트).
- `캐릭터 고르기`는 진짜 라디오가 아니라 `role="radio"` 버튼이다(폼 액션이 끝나면 React가 폼을 되돌려 라디오가 처음 값으로 돌아가는 것을 피함).

## User Scenarios & Testing *(mandatory)*

### User Story 1 - 가입할 때 캐릭터 고르기 (Priority: P1) 🎯

1. 회원가입 폼의 `캐릭터 고르기`에 카드 2개(56px 도트 그림 · 이름), 처음에는 남자 주민. 고른 카드는 노란 테두리·바탕.
2. 가입하면 고른 캐릭터 1종만 인벤토리에 들어가고 장착된다. 다른 칸 오류로 막혀도 고른 캐릭터는 그대로다.
3. 기본 캐릭터가 아닌 ID·이상한 값을 보내면 `고를 수 없는 캐릭터예요`, 가입되지 않는다(서버가 트랜잭션 안에서 DB로 다시 본다).
4. 헤더 `Lv.N 🪙 N` 앞에 내 캐릭터(28px), 블로그 홈 머리글은 프로필 사진이 없으면 캐릭터 얼굴(초록 동그라미).

## Success Criteria *(mandatory)*

- **SC-001**: `tests/unit/pixel-art.test.ts`, `tests/e2e/characters.spec.ts`가 통과하고 기존 시험이 그대로 통과한다.
