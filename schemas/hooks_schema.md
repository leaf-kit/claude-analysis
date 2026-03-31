---
tags:
  - module
  - schemas
  - hooks
  - zod
  - validation
created: 2026-03-31
type: source-analysis
status: complete
---

# hooks.ts (Schemas)

> 훅 설정 Zod 스키마 정의 — 순환 import 해결을 위해 분리

## 개요

| 항목 | 값 |
|------|-----|
| 파일 경로 | `src/schemas/hooks.ts` |
| 주요 역할 | 훅 구성 스키마의 단일 진실 원천 |
| 훅 타입 수 | 4 (command, prompt, agent, http) |

## 훅 타입

### 1. Command (Shell)
```typescript
{ type: 'command', command: string, shell?: ShellType, async?: boolean, asyncRewake?: boolean }
```

### 2. Prompt (LLM)
```typescript
{ type: 'prompt', prompt: string, model?: string }
```

### 3. Agent (검증자)
```typescript
{ type: 'agent', prompt: string, model?: string }
```

### 4. HTTP (웹훅)
```typescript
{ type: 'http', url: string, headers?: Record<string, string>, allowedEnvVars?: string[] }
```

### 공통 속성
- `if` — 선택적 권한 규칙 필터
- `timeout` — 타임아웃 (초)
- `statusMessage` — 스피너 표시 텍스트
- `once` — 1회 실행 플래그

## 스키마 계층

```
HookCommandSchema (개별 훅)
  ↓
HookMatcherSchema (매처 + 훅 목록)
  ↓
HooksSchema (이벤트별 훅 구성)
```

## 의존성

| 모듈 | Import | 용도 |
|------|--------|------|
| [[entrypoints/agentSdkTypes]] | `HOOK_EVENTS` | 훅 이벤트 목록 |
| [[utils/lazySchema]] | `lazySchema()` | 지연 스키마 생성 |
| [[utils/shell/shellProvider]] | `SHELL_TYPES` | 셸 타입 목록 |

## 관계

- **소비자**: [[utils/settings/types]] — 설정 타입에서 훅 스키마 import
- **소비자**: [[utils/plugins/schemas]] — 플러그인 스키마에서 import
- **분리 이유**: `settings/types.ts` ↔ `plugins/schemas.ts` 간 순환 의존 방지
