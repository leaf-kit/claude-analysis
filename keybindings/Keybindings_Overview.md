---
tags:
  - module
  - keybindings
  - keyboard
  - react-hooks
  - chord
created: 2026-03-31
type: source-analysis
status: complete
---

# 키바인딩 시스템 (Keybindings)

> 사용자 커스터마이징 가능한 키보드 단축키 시스템 (14파일, 코드 지원, 핫 리로드)

## 개요

| 항목 | 값 |
|------|-----|
| 디렉터리 | `src/keybindings/` |
| 파일 수 | 14 |
| 컨텍스트 수 | 18 (Global, Chat, Autocomplete 등) |
| 액션 수 | 72 |
| 기본 바인딩 블록 | 13개 컨텍스트 |

## 아키텍처

```
Default Bindings (defaultBindings.ts)
    ↓
Loader (loadUserBindings.ts) ← Parser (parser.ts)
    ↓
KeybindingSetup (KeybindingProviderSetup.tsx)
    ├→ KeybindingProvider (KeybindingContext.tsx)
    │     ├→ Resolver (resolver.ts) ← Match (match.ts)
    │     └→ Handler Registry
    ├→ ChordInterceptor (useInput 래퍼)
    ├→ Validation (validate.ts)
    └→ Notifications (경고 표시)
```

## 핵심 파일

### 데이터 & 설정
| 파일 | 역할 |
|------|------|
| `schema.ts` | Zod 스키마 (18 컨텍스트, 72 액션) |
| `defaultBindings.ts` | 기본 키 바인딩 (플랫폼별) |
| `reservedShortcuts.ts` | OS/터미널 예약 단축키 |
| `template.ts` | keybindings.json 템플릿 생성 |

### 파싱 & 매칭
| 파일 | 역할 |
|------|------|
| `parser.ts` | "ctrl+shift+k" → `ParsedKeystroke` 파싱 |
| `match.ts` | Ink 입력과 키스트로크 매칭 |
| `resolver.ts` | 키스트로크 → 액션 해결 (코드 지원) |
| `validate.ts` | 중복, 예약 키, 구문 오류 검증 |

### React 통합
| 파일 | 역할 |
|------|------|
| `KeybindingContext.tsx` | React 컨텍스트 Provider |
| `KeybindingProviderSetup.tsx` | 초기화 + ChordInterceptor |
| `useKeybinding.ts` | 단일/다중 바인딩 훅 |
| `useShortcutDisplay.ts` | 단축키 표시 텍스트 훅 |
| `shortcutFormat.ts` | 비-React 단축키 표시 유틸 |
| `loadUserBindings.ts` | 파일 감시 + 핫 리로드 |

## 코드(Chord) 시스템

```typescript
// 다중 키스트로크 시퀀스 지원
"ctrl+x ctrl+k"  → 에이전트 종료
"ctrl+x ctrl+e"  → 외부 에디터

// 코드 타임아웃: 1000ms (기본)
// Escape로 코드 취소
```

## 해결(Resolution) 우선순위

1. **긴 코드** > 단일 키 매칭
2. **마지막 매칭** 우선 (사용자 오버라이드)
3. **컨텍스트 우선순위**: 등록된 활성 컨텍스트 → 현재 컨텍스트 → Global
4. `null` 액션 = 명시적 언바인딩

## 플랫폼 대응

| 기능 | macOS | Windows | Linux |
|------|-------|---------|-------|
| 이미지 붙여넣기 | ctrl+v | alt+v | ctrl+v |
| 모드 전환 | shift+tab | meta+m (VT 없을 때) | shift+tab |
| 메타 표시 | cmd | meta | meta |
| 예약 키 | cmd+c/v/q/w | - | - |

## 의존성

### 외부
- `zod/v4` — 스키마 검증
- `chokidar` — 파일 감시
- `react` — UI 프레임워크

### 내부 주요
- [[ink]] — 터미널 UI (useInput)
- [[services/analytics/growthbook]] — 피처 게이트 (내부 사용자만)
- [[context/notifications]] — 경고 표시
- [[utils/platform]] — OS 감지

## 관계

- **소비자**: [[screens/REPL]], [[components/PromptInput]], 모든 키 입력 컴포넌트
- **설정 파일**: `~/.claude/keybindings.json`
- **명령어**: `/keybindings` → 템플릿 생성
