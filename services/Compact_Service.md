---
tags:
  - service
  - compact
  - context-window
  - memory
created: 2026-03-31
type: service-analysis
status: complete
---

# Compact Service

> 컨텍스트 윈도우 압축 서비스. 대화가 길어질 때 자동으로 요약/정리

## 아키텍처

```
토큰 예산 추적
  │
  ▼
autoCompact() — 임계값 체크
  │
  ├── 임계값 미달 → 계속 진행
  │
  └── 임계값 초과 →
      ├── compactConversation() — 전체 대화 요약
      ├── microCompact() — 시간 기반 결과 정리
      └── trySessionMemoryCompaction() — 메모리 추출 통합
```

## 파일 구조

| 파일 | 용도 |
|------|------|
| `compact.ts` | 메인 압축 로직 |
| `autoCompact.ts` | 자동 압축 임계값 |
| `microCompact.ts` | 시간 기반 컴팩트 |
| `sessionMemoryCompact.ts` | 세션 메모리 통합 |
| `postCompactCleanup.ts` | 압축 후 정리 |
| `grouping.ts` | 메시지 그룹핑 |
| `apiMicrocompact.ts` | API 마이크로컴팩트 |
| `prompt.ts` | 압축 프롬프트 |
| `timeBasedMCConfig.ts` | 시간 기반 설정 |
| `compactWarningHook.ts` | 압축 경고 훅 |
| `compactWarningState.ts` | 경고 상태 |

## 압축 전략

### 1. Full Compact
전체 대화를 요약하여 하나의 시스템 메시지로 축소

### 2. Micro Compact
시간 기준으로 오래된 도구 결과를 정리

### 3. Session Memory Compact
압축 과정에서 [[services/SessionMemory_Service|세션 메모리]]로 중요 정보 추출

## 의존 모듈

- [[services/API_Service]] — 요약 생성용 API 호출
- [[services/SessionMemory_Service]] — 메모리 통합
- [[Query_Engine|query/tokenBudget.ts]] — 토큰 예산

## 관련 문서
- [[Services_Overview]] — 서비스 전체
- [[Query_Engine]] — 쿼리 엔진
