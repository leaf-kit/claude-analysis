---
tags:
  - services
  - core
  - architecture
created: 2026-03-31
type: module-analysis
status: complete
---

# Services Overview

> Claude Code의 핵심 서비스 레이어. API 통신, MCP, 컨텍스트 관리, 인증, 분석 등

## 서비스 아키텍처 맵

```
┌──────────────────────────────────────────────┐
│              서비스 레이어                      │
├──────────────┬──────────────┬────────────────┤
│ [[services/  │ [[services/  │ [[services/    │
│ API_Service]]│ MCP_Service]]│ Compact_       │
│ Claude API   │ MCP 서버     │ Service]]      │
│ 멀티프로바이더│ 연결 관리    │ 컨텍스트 압축  │
├──────────────┼──────────────┼────────────────┤
│ [[services/  │ [[services/  │ [[services/    │
│ Tools_       │ LSP_Service]]│ OAuth_         │
│ Service]]    │ 언어 서버    │ Service]]      │
│ 도구 실행    │ 관리         │ 인증           │
├──────────────┼──────────────┼────────────────┤
│ [[services/  │ [[services/  │ [[services/    │
│ Analytics_   │ SessionMem   │ ExtractMem     │
│ Service]]    │ _Service]]   │ _Service]]     │
│ 분석/이벤트  │ 세션 메모리  │ 메모리 추출    │
├──────────────┼──────────────┼────────────────┤
│ [[services/  │ tips/        │ policyLimits/  │
│ Plugins_     │ 팁 레지스트리│ 정책 제한      │
│ Service]]    │ 및 스케줄러  │ 관리           │
└──────────────┴──────────────┴────────────────┘
```

## 핵심 서비스 상세

### 1. [[services/API_Service]] — Claude API 클라이언트

**목적:** Claude API와의 통신. 멀티 프로바이더(Anthropic, AWS Bedrock, Google Vertex AI, Azure Foundry) 지원

**핵심 파일:**

| 파일 | 용도 | 크기 |
|------|------|------|
| `client.ts` | 클라이언트 생성 및 프로바이더 감지 | ~390줄 |
| `claude.ts` | 메인 API 인터랙션 (메시지 핸들링) | 대형 |
| `withRetry.ts` | 재시도 로직 및 에러 핸들링 | 중형 |
| `errors.ts` | API 에러 정의 | 중형 |
| `bootstrap.ts` | API 부트스트랩 | 소형 |
| `usage.ts` | 사용량 추적 | 소형 |
| `promptCacheBreakDetection.ts` | 프롬프트 캐시 최적화 | 소형 |

**프로바이더 지원:**
```
Direct API (api.anthropic.com)
  ├── AWS Bedrock (@anthropic-ai/bedrock-sdk)
  ├── Google Vertex AI (@anthropic-ai/vertex-sdk)
  └── Azure Foundry (@anthropic-ai/foundry-sdk)
```

---

### 2. [[services/MCP_Service]] — MCP 서버 연결 관리

**목적:** Model Context Protocol 서버의 연결, 인증, 도구/리소스 발견

**핵심 파일:**

| 파일 | 용도 |
|------|------|
| `client.ts` | MCP 클라이언트 (도구/리소스 접근) |
| `config.ts` | 서버 설정 로딩/검증 |
| `MCPConnectionManager.tsx` | React 컨텍스트 (연결 상태) |
| `useManageMCPConnections.ts` | 연결 관리 훅 |
| `auth.ts` | OAuth 및 인증 |
| `types.ts` | MCP 타입 정의 |
| `channelPermissions.ts` | 채널 권한 |
| `channelAllowlist.ts` | 채널 허용 목록 |

**전송 프로토콜:** stdio, SSE, WebSocket, HTTP

**연결 흐름:**
```
설정 로딩 → 서버 연결 → 인증 → 도구/리소스 발견 → tools.ts에 병합
```

---

### 3. [[services/Compact_Service]] — 컨텍스트 윈도우 압축

**목적:** 컨텍스트 윈도우가 한계에 근접하면 자동으로 대화 요약/압축

**핵심 파일:**

| 파일 | 용도 |
|------|------|
| `compact.ts` | 메인 압축 로직 |
| `autoCompact.ts` | 자동 압축 임계값 관리 |
| `microCompact.ts` | 시간 기반 결과 정리 |
| `sessionMemoryCompact.ts` | 압축 시 메모리 추출 통합 |
| `postCompactCleanup.ts` | 압축 후 정리 |
| `grouping.ts` | 메시지 그룹핑 |
| `apiMicrocompact.ts` | API 마이크로컴팩트 |

**압축 전략:**
1. **Full Compact**: 전체 대화 요약
2. **Micro Compact**: 시간 기반 결과 정리
3. **Session Memory**: 압축 중 메모리 추출

---

### 4. [[services/Tools_Service]] — 도구 실행 오케스트레이션

**목적:** 도구의 동시 실행, 결과 스트리밍, 비용 추적

**핵심 파일:**

| 파일 | 용도 |
|------|------|
| `toolExecution.ts` | 핵심 실행 로직 |
| `toolOrchestration.ts` | 동시성 관리 (병렬/직렬 분류) |
| `StreamingToolExecutor.ts` | 스트리밍 실행기 (~140줄) |
| `toolHooks.ts` | 도구 라이프사이클 훅 |

**실행 패턴:**
```typescript
// Async Generator로 스트리밍 결과 반환
async function* runTools(toolUses, context) {
  // 동시성 안전 도구는 병렬 실행
  // 비안전 도구는 직렬 실행
  // 권한 체크 후 실행
}
```

---

### 5. [[services/LSP_Service]] — LSP 서버 관리

**목적:** Language Server Protocol 서버 인스턴스 관리, 언어별 라우팅

**핵심 파일:**

| 파일 | 용도 |
|------|------|
| `LSPClient.ts` | LSP 클라이언트 래퍼 |
| `LSPServerManager.ts` | 서버 레지스트리/라우터 |
| `LSPServerInstance.ts` | 개별 서버 인스턴스 |
| `LSPDiagnosticRegistry.ts` | 진단 추적 |
| `config.ts` | LSP 서버 설정 |
| `passiveFeedback.ts` | 수동 피드백 |

**제공 기능:** 정의 이동, 참조 찾기, 호버, 심볼 조회, 호출 계층

---

### 6. [[services/OAuth_Service]] — OAuth 2.0 인증

**목적:** PKCE 기반 OAuth 2.0 인증 코드 흐름

**파일:** `client.ts`, `index.ts`, `auth-code-listener.ts`, `crypto.ts`, `getOauthProfile.ts`

**인증 흐름:**
```
인증 URL 구성 → 브라우저 열기/수동 복사 → 콜백 수신 → 토큰 교환 → 프로필 조회
```

---

### 7. [[services/Analytics_Service]] — 분석 및 이벤트 로깅

**목적:** 제로 의존성 이벤트 로깅. Datadog + 1P 로깅 싱크

**파일:** `index.ts`, `metadata.ts`, `datadog.ts`, `firstPartyEventLogger.ts`, `growthbook.ts`, `sink.ts`, `config.ts`

**이벤트 흐름:**
```
logEvent() → 이벤트 큐 → 싱크 연결 후 전송 → Datadog + 1P
```

---

### 8. [[services/SessionMemory_Service]] — 세션 메모리

**목적:** 대화 중 자동으로 마크다운 메모 유지 관리

**파일:** `sessionMemory.ts`, `sessionMemoryUtils.ts`, `prompts.ts`

**동작:** 포크된 서브에이전트가 백그라운드에서 메모리 파일 업데이트

---

### 9. [[services/ExtractMemories_Service]] — 메모리 추출

**목적:** 쿼리 루프 종료 시 대화에서 지속 메모리 추출

**파일:** `extractMemories.ts`, `prompts.ts`

**동작:** 프롬프트 캐시 공유를 사용하는 포크된 에이전트로 실행

---

### 10. [[services/Plugins_Service]] — 플러그인 라이프사이클

**목적:** 플러그인 설치/제거/활성화/비활성화/업데이트

**파일:** `pluginOperations.ts`, `pluginCliCommands.ts`, `PluginInstallationManager.ts`

**기능:** 마켓플레이스 통합, 의존성 해석, 정책 기반 차단

---

## 기타 서비스

| 서비스 | 용도 |
|--------|------|
| `tips/` | 팁 레지스트리 및 스케줄러 |
| `policyLimits/` | 레이트 리밋 및 정책 관리 |
| `remoteManagedSettings/` | 엔터프라이즈 원격 설정 동기화 |
| `settingsSync/` | 설정 동기화 |
| `AgentSummary/` | 에이전트 실행 요약 |
| `autoDream/` | 자동 드림 기능 |
| `PromptSuggestion/` | 프롬프트 제안 |
| `MagicDocs/` | 문서 생성 |
| `teamMemorySync/` | 팀 메모리 동기화 |
| `toolUseSummary/` | 도구 사용 요약 생성 |

## 통계

| 항목 | 값 |
|------|-----|
| 총 서비스 수 | 19 |
| 총 파일 수 | 130 |
| API 프로바이더 | 4 (Anthropic, Bedrock, Vertex, Foundry) |
| MCP 전송 프로토콜 | 4 (stdio, SSE, WebSocket, HTTP) |

## 관련 문서
- [[Index]] — 전체 MOC
- [[Tools_Overview]] — 도구 시스템
- [[Query_Engine]] — 쿼리 엔진
