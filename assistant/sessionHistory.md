---
tags:
  - module
  - assistant
  - remote-session
  - pagination
created: 2026-03-31
type: source-analysis
status: complete
---

# sessionHistory.ts

> 원격 세션의 이벤트 히스토리를 페이지네이션 방식으로 가져오는 모듈

## 개요

| 항목 | 값 |
|------|-----|
| 파일 경로 | `src/assistant/sessionHistory.ts` |
| 주요 역할 | 원격 세션 트랜스크립트 페이지네이션 |
| 연결 문서 수 | 5 |
| 의존 모듈 수 | 4 (내부) + 1 (외부) |

`sessionHistory.ts`는 Claude Code 원격 세션 API에서 세션 이벤트를 페이지 단위로 가져오는 기능을 담당합니다. viewer-only 모드에서 원격 세션 트랜스크립트를 볼 때 사용됩니다.

## 핵심 Export

### 상수
- **`HISTORY_PAGE_SIZE`** (100) — 페이지네이션 기본 페이지 크기

### 타입
- **`HistoryPage`** — 페이지 결과 (`events`, `firstId`, `hasMore`)
- **`HistoryAuthCtx`** — 인증 컨텍스트 (`baseUrl`, `headers`)

### 함수
```typescript
// 인증 컨텍스트 준비 (1회 생성 후 재사용)
createHistoryAuthCtx(sessionId: string): Promise<HistoryAuthCtx>

// 최신 이벤트 페이지 가져오기 (anchor_to_latest)
fetchLatestEvents(ctx: HistoryAuthCtx, limit?: number): Promise<HistoryPage>

// 특정 커서 이전 이벤트 가져오기
fetchOlderEvents(ctx: HistoryAuthCtx, beforeId: string, limit?: number): Promise<HistoryPage>
```

## 의존성

### 내부 모듈
| 모듈 | Import | 용도 |
|------|--------|------|
| [[constants/oauth]] | `getOauthConfig()` | OAuth 설정 (API base URL, beta headers) |
| [[entrypoints/agentSdkTypes]] | `SDKMessage` | 세션 이벤트 메시지 타입 |
| [[utils/debug]] | `logForDebugging()` | 디버그 로깅 |
| [[utils/teleport/api]] | `getOAuthHeaders()`, `prepareApiRequest()` | OAuth 토큰/헤더 생성 |

### 외부 의존성
- `axios` — HTTP 클라이언트

## 데이터 흐름

```
사용자가 원격 세션 열람 (viewer-only)
  → useAssistantHistory 훅 초기화
    → createHistoryAuthCtx() (1회)
    → fetchLatestEvents() (최신 페이지 로드)
    → 스크롤 업 시 fetchOlderEvents() (이전 페이지 로드)
      → SDKMessage[] → convertSDKMessage → UI 메시지로 변환
```

## 관계

- **소비자**: [[hooks/useAssistantHistory]] — 페이지네이션 로직과 스크롤 앵커링 구현
- **기능 게이트**: `KAIROS` 피처 플래그로 제어
- **연동**: [[remote/RemoteSessionManager]], OAuth 인증 시스템

## 설계 특징

- **재사용 가능한 인증 컨텍스트**: `createHistoryAuthCtx()`를 한 번 호출하면 여러 페이지 요청에 재사용
- **양방향 페이지네이션**: 최신→과거 방향으로 증분 로딩
- **에러 처리**: HTTP 상태 검증 포함
