---
tags:
  - module
  - buddy
  - companion
  - easter-egg
  - procedural-generation
created: 2026-03-31
type: source-analysis
status: complete
---

# Buddy 시스템 (Companion)

> 사용자 ID 기반 절차적 생성 컴패니언 캐릭터 시스템 (6파일, 1,298줄)

## 개요

| 항목 | 값 |
|------|-----|
| 디렉터리 | `src/buddy/` |
| 파일 수 | 6 |
| 총 라인 수 | 1,298 |
| 종(Species) 수 | 18 |
| 희귀도 단계 | 5 |

Buddy 시스템은 사용자 ID의 해시값을 기반으로 **결정적(deterministic)** 컴패니언 캐릭터를 생성하는 이스터에그 기능입니다. `/buddy` 명령어로 활성화됩니다.

## 파일 구조

```
buddy/
├── types.ts          — 타입, 상수 정의 (148줄)
├── companion.ts      — 절차적 생성 로직 (133줄)
├── sprites.ts        — ASCII 아트 스프라이트 (514줄)
├── prompt.ts         — 시스템 프롬프트 생성 (36줄)
├── CompanionSprite.tsx — React 렌더링 컴포넌트 (370줄)
└── useBuddyNotification.tsx — 티저 알림 훅 (97줄)
```

## 핵심 개념

### 종(Species) — 18종
duck, goose, blob, cat, dragon, octopus, owl, penguin, turtle, snail, ghost, axolotl, capybara, cactus, robot, rabbit, mushroom, chonk

### 희귀도(Rarity)
| 등급 | 확률 | 특징 |
|------|------|------|
| Common | 60% | 모자 없음 |
| Uncommon | 25% | 모자 가능 |
| Rare | 10% | 높은 스탯 |
| Epic | 4% | 높은 스탯 + 모자 |
| Legendary | 1% | 최고 스탯 |

### Bones vs Soul 분리
- **Bones** (결정적): 종, 희귀도, 눈, 모자, 스탯 — 사용자 ID 해시에서 매번 재생성
- **Soul** (영속적): 이름, 성격 — 설정 파일에 저장

## 핵심 함수

```typescript
// companion.ts
roll(userId: string): Roll          // Bones 생성 (캐시됨)
getCompanion(): Companion | undefined // 전체 컴패니언 조회

// sprites.ts
renderSprite(bones, frame?): string[] // 5줄 ASCII 스프라이트 렌더링
spriteFrameCount(species): number     // 애니메이션 프레임 수

// prompt.ts
getCompanionIntroAttachment(messages?): Attachment[] // 첫 메시지 인트로
```

## 애니메이션 시스템

- **틱 간격**: 500ms
- **유휴 시퀀스**: `[0,0,0,0,1,0,0,0,-1,0,0,2,0,0,0]` (주로 정지, 가끔 동작)
- **쓰다듬기**: `/buddy pet` → 하트 5프레임 애니메이션 (2.5초)
- **말풍선**: 10초 표시, 마지막 3초 페이드

## 의존성

| 파일 | 내부 모듈 의존 |
|------|---------------|
| companion.ts | [[utils/config]] |
| prompt.ts | [[types/message]], [[utils/attachments]], [[utils/config]] |
| CompanionSprite.tsx | [[hooks/useTerminalSize]], [[state/AppState]], [[ink]] |
| useBuddyNotification.tsx | [[context/notifications]], [[utils/thinking]] |

## 피처 게이트

- `feature('BUDDY')` — 모든 buddy 코드가 이 플래그로 보호
- 티저 윈도우: 2026년 4월 1-7일에만 알림 표시

## 관계

- **CompanionSprite** → [[screens/REPL]] 내 프롬프트 옆에 렌더링
- **prompt.ts** → [[Query_Engine]] 첫 메시지에 컴패니언 인트로 첨부
- **types.ts** → 모든 buddy 파일의 타입 기반
