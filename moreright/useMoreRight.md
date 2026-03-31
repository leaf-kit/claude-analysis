---
tags:
  - module
  - moreright
  - stub
  - react-hook
created: 2026-03-31
type: source-analysis
status: complete
---

# useMoreRight.tsx

> REPL용 "More Right" 기능의 외부 빌드 스텁 (25줄)

## 개요

| 항목 | 값 |
|------|-----|
| 파일 경로 | `src/moreright/useMoreRight.tsx` |
| 주요 역할 | 외부 빌드용 플레이스홀더 훅 |
| 라인 수 | 25 |

실제 구현은 내부 전용이며, 이 파일은 외부 빌드에서 컴파일 에러를 방지하는 스텁입니다.

## 인터페이스

```typescript
export function useMoreRight(_args: {
  enabled: boolean
  setMessages: (action: M[] | ((prev: M[]) => M[])) => void
  inputValue: string
  setInputValue: (s: string) => void
  setToolJSX: (args: M) => void
}): {
  onBeforeQuery: (input: string, all: M[], n: number) => Promise<boolean>
  onTurnComplete: (all: M[], aborted: boolean) => Promise<void>
  render: () => null
}
```

## 의존성
- 없음 (자체 완결)

## 관계
- **소비자**: [[screens/REPL]] — REPL 화면에서 통합
