---
tags:
  - module
  - bootstrap
  - global-state
  - singleton
  - critical
created: 2026-03-31
type: source-analysis
status: complete
---

# bootstrap/state.ts

> 전체 세션의 글로벌 상태를 관리하는 싱글톤 모듈 (1,758줄, 209개 getter/setter)

## 개요

| 항목 | 값 |
|------|-----|
| 파일 경로 | `src/bootstrap/state.ts` |
| 주요 역할 | 세션 전역 상태 관리 (싱글톤) |
| 총 라인 수 | 1,758 |
| Export 함수 수 | 209 |
| 상태 프로퍼티 수 | 100+ |
| 의존 모듈 수 | 3 (내부) + 4 (외부) |

`state.ts`는 Claude Code의 **import DAG 최하위(leaf)** 에 위치하는 핵심 인프라 모듈입니다. 순환 의존을 방지하기 위해 최소한의 외부 import만 가집니다.

## 상태 카테고리

### 세션 관리 (30+)
```typescript
getSessionId() / regenerateSessionId() / switchSession()
getParentSessionId()  // 세션 계보 (plan mode → implementation)
getOriginalCwd() / getProjectRoot()
```

### 모델 & API 메트릭 (25+)
```typescript
getTotalCostUSD() / addToTotalCostState()
getTotalInputTokens() / getTotalOutputTokens()
getTotalCacheReadInputTokens() // 프롬프트 캐시 메트릭
getUsageForModel() / getModelUsage()  // 모델별 사용량
```

### 텔레메트리 & 관측성 (20+)
```typescript
setMeter() / getMeter()  // OpenTelemetry
getSessionCounter() / getLocCounter() / getCostCounter()
```

### 인증 & 보안 (6)
```typescript
getSessionIngressToken() / setSessionIngressToken()
getOauthTokenFromFd() / getApiKeyFromFd()
```

### 플러그인 & 확장성 (10+)
```typescript
getInlinePlugins() / registerHookCallbacks()
setInitJsonSchema()  // 구조화된 출력 스키마
```

### Plan & Auto 모드 (6)
```typescript
hasExitedPlanModeInSession() / handlePlanModeTransition()
needsAutoModeExitAttachment() / handleAutoModeTransition()
```

## 의존성

### 내부 (최소화됨 — leaf 모듈)
| 모듈 | Import | 용도 |
|------|--------|------|
| [[utils/crypto]] | `randomUUID()` | 세션 ID 생성 |
| [[utils/settings/settingsCache]] | `resetSettingsCache()` | 설정 캐시 초기화 |
| [[utils/signal]] | `createSignal()` | `onSessionSwitch` 반응형 시그널 |

### 외부
- `@anthropic-ai/sdk`, `@opentelemetry/*`, `lodash-es`, `node:fs`, `node:process`

## 설계 패턴

1. **싱글톤 패턴**: 모듈 로드 시 `STATE` 객체 생성
2. **Getter/Setter 패턴**: 209개 함수로 타입 안전한 접근 (직접 변경 금지)
3. **Signal 패턴**: `onSessionSwitch` 시그널로 세션 전환 반응
4. **Bootstrap 격리**: `custom-rules/bootstrap-isolation` ESLint 규칙으로 순환 import 방지
5. **Sticky Latch**: 베타 헤더 기능은 sticky-on으로 프롬프트 캐시 무효화 방지

## 주의사항

> ⚠️ "DO NOT ADD MORE STATE HERE - BE JUDICIOUS WITH GLOBAL STATE" — 코드 내 주석

- 경로 정규화: `realpathSync().normalize('NFC')` (심볼릭 링크 대응)
- 에이전트 범위 스킬: 복합 키 `${agentId}:${skillName}` 사용
- 테스트 격리: `resetStateForTests()` 제공

## 관계

- 사실상 **모든 주요 서브시스템**이 이 모듈을 import
- [[Query_Engine]], [[Tools_Overview]], [[Services_Overview]], [[Entrypoints_Overview]] 등 전체 레이어에서 참조
- [[main.tsx]], [[setup.ts]], [[query.ts]] 등 핵심 파일의 상태 소스
