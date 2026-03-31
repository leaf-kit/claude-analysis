---
tags:
  - service
  - MCP
  - protocol
  - extensibility
created: 2026-03-31
type: service-analysis
status: complete
---

# MCP Service

> Model Context Protocol 서버 연결 관리. 도구/리소스 발견, 인증, 연결 오케스트레이션

## 아키텍처

```
설정 파일 (mcpServers in settings.json)
  │
  ▼
getAllMcpConfigs() — 설정 로딩/검증
  │
  ▼
useManageMCPConnections — 연결 관리 훅
  │
  ├── 서버 연결 (stdio / SSE / WebSocket / HTTP)
  ├── 인증 (OAuth / 커스텀)
  ├── 도구/리소스 발견
  │
  ▼
MCPConnectionManager.tsx — React 컨텍스트
  │
  ▼
tools.ts에 MCP 도구 병합
```

## 파일 구조

| 파일 | 용도 | 크기 |
|------|------|------|
| `client.ts` | MCP 클라이언트 (도구/리소스) | 대형 |
| `config.ts` | 설정 관리 | 대형 |
| `MCPConnectionManager.tsx` | React 연결 컨텍스트 | 중형 |
| `useManageMCPConnections.ts` | 연결 관리 훅 | 대형 |
| `auth.ts` | OAuth 및 인증 | 대형 |
| `types.ts` | 타입 정의 | 소형 |
| `channelPermissions.ts` | 채널 권한 | 중형 |
| `channelAllowlist.ts` | 채널 허용목록 | 소형 |
| `channelNotification.ts` | 채널 알림 | 소형 |
| `normalization.ts` | 설정 정규화 | 소형 |
| `envExpansion.ts` | 환경변수 확장 | 소형 |
| `utils.ts` | 유틸리티 | 소형 |
| `mcpStringUtils.ts` | 문자열 유틸 | 소형 |
| `InProcessTransport.ts` | 인프로세스 전송 | 소형 |
| `SdkControlTransport.ts` | SDK 제어 전송 | 소형 |
| `officialRegistry.ts` | 공식 레지스트리 | 소형 |
| `vscodeSdkMcp.ts` | VSCode SDK MCP | 소형 |
| `headersHelper.ts` | 헤더 도우미 | 소형 |
| `elicitationHandler.ts` | 정보 수집 핸들러 | 소형 |
| `xaaIdpLogin.ts` | XAA IDP 로그인 | 소형 |
| `xaa.ts` | XAA 통합 | 소형 |
| `claudeai.ts` | Claude.ai MCP | 소형 |
| `oauthPort.ts` | OAuth 포트 | 소형 |

## 전송 프로토콜

| 프로토콜 | 용도 |
|----------|------|
| `stdio` | 로컬 프로세스 통신 |
| `SSE` | Server-Sent Events |
| `WebSocket` | 양방향 스트림 |
| `HTTP` | HTTP 요청/응답 |

## 의존 모듈

- `@modelcontextprotocol/sdk` — MCP 프로토콜
- `vscode-jsonrpc` — JSON-RPC 통신
- [[services/OAuth_Service]] — 인증
- [[Tools_Overview|tools.ts]] — 도구 병합

## 관련 문서
- [[Services_Overview]] — 서비스 전체
- [[tools/MCPTool]] — MCP 도구
- [[Tools_Overview]] — 도구 시스템
