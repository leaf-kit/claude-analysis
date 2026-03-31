---
tags:
  - tool
  - MCP
  - protocol
  - extensibility
created: 2026-03-31
type: tool-analysis
status: complete
---

# MCPTool

> MCP (Model Context Protocol) 도구 실행. 외부 MCP 서버의 도구를 동적으로 호출

## 개요

| 항목 | 값 |
|------|-----|
| 파일 | `src/tools/MCPTool/MCPTool.ts` |
| 읽기전용 | 가변 (서버 도구에 따라) |
| 동시성 안전 | 가변 |
| 파괴적 | 가변 |

## 핵심 기능

1. **동적 스키마**: MCP 서버가 정의한 스키마로 도구 호출
2. **도구 이름 오버라이드**: 서버 도구 이름 매핑
3. **Pass-through 권한**: 서버 도구의 권한을 투과적으로 처리
4. **멀티 전송**: stdio, SSE, WebSocket, HTTP 전송 지원

## MCP 통합 흐름

```
설정 파일 (settings.json)
  │
  ▼
[[services/MCP_Service|MCPConnectionManager]]
  │
  ▼
서버 연결 + 인증
  │
  ▼
도구/리소스 발견
  │
  ▼
tools.ts에 MCP 도구 병합 (assembleToolPool)
  │
  ▼
MCPTool로 실행
```

## 관련 MCP 도구

| 도구 | 용도 |
|------|------|
| `MCPTool` | MCP 서버 도구 실행 |
| `ListMcpResourcesTool` | MCP 리소스 목록 |
| `ReadMcpResourceTool` | MCP 리소스 읽기 |
| `McpAuthTool` | MCP 인증 관리 |

## 의존 모듈

- [[services/MCP_Service]] — MCP 서버 연결/관리
- `@modelcontextprotocol/sdk` — MCP 프로토콜

## 관련 문서
- [[Tools_Overview]] — 도구 전체 개요
- [[services/MCP_Service]] — MCP 서비스
