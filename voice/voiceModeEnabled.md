---
tags:
  - module
  - voice
  - feature-gate
  - auth
created: 2026-03-31
type: source-analysis
status: complete
---

# voiceModeEnabled.ts

> 음성 모드 기능 게이트 및 인증 확인 (1파일)

## 개요

| 항목 | 값 |
|------|-----|
| 파일 경로 | `src/voice/voiceModeEnabled.ts` |
| 주요 역할 | 음성 모드 활성화 조건 확인 |

## 핵심 Export

```typescript
isVoiceGrowthBookEnabled(): boolean  // GrowthBook 킬스위치 확인
hasVoiceAuth(): boolean              // Anthropic OAuth 토큰 확인
isVoiceModeEnabled(): boolean        // 두 조건 모두 충족 여부
```

## 활성화 조건

| 체크 | 조건 | 비고 |
|------|------|------|
| 빌드 타임 | `feature('VOICE_MODE')` | Bun 번들 피처 플래그 |
| 런타임 | `tengu_amber_quartz_disabled` ≠ true | GrowthBook 긴급 차단 |
| 인증 | Anthropic OAuth 활성 + 유효 토큰 | API 키, Bedrock, Vertex 불가 |

**인증 세부사항:**
- `voice_stream` 엔드포인트 (claude.ai) 사용
- `getClaudeAIOAuthTokens()` 메모이즈 (~1시간마다 갱신)
- macOS에서 첫 호출 시 `security` 프로세스 생성 (~20-50ms)

## 의존성

| 모듈 | Import |
|------|--------|
| [[services/analytics/growthbook]] | `getFeatureValue_CACHED_MAY_BE_STALE()` |
| [[utils/auth]] | `getClaudeAIOAuthTokens()`, `isAnthropicAuthEnabled()` |

## 관계

- **소비자**: `commands/voice/voice.ts` — 음성 명령 실행 전 확인
- **소비자**: `hooks/useVoiceEnabled.ts` — React 훅 래퍼
- **소비자**: `tools/ConfigTool/prompt.ts` — 음성 설정 옵션 조건부 표시
