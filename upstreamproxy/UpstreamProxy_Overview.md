---
tags:
  - module
  - upstreamproxy
  - networking
  - CCR
  - security
  - websocket
created: 2026-03-31
type: source-analysis
status: complete
---

# 업스트림 프록시 (Upstream Proxy)

> CCR 세션용 CONNECT-over-WebSocket 릴레이 (2파일)

## 개요

| 항목 | 값 |
|------|-----|
| 디렉터리 | `src/upstreamproxy/` |
| 파일 수 | 2 |
| 프로토콜 | HTTP CONNECT → WebSocket 터널링 |
| 보안 | ptrace 보호, mTLS, 세션 토큰 |

## 파일 구조

### upstreamproxy.ts — 프록시 초기화 & 환경변수 관리

```typescript
SESSION_TOKEN_PATH = '/run/ccr/session_token'

initUpstreamProxy(opts?)       // 프록시 설정 초기화
getUpstreamProxyEnv()          // 서브프로세스 환경변수 반환
resetUpstreamProxyForTests()   // 테스트용 리셋
```

**초기화 흐름:**
```
CCR 모드 감지 (CLAUDE_CODE_REMOTE + CCR_UPSTREAM_PROXY_ENABLED)
  → 세션 토큰 읽기 (/run/ccr/session_token)
  → prctl(PR_SET_DUMPABLE, 0) (Bun FFI, ptrace 방지)
  → CCR CA 인증서 다운로드 + 시스템 CA 번들 병합
  → 로컬 CONNECT→WS 릴레이 시작 (127.0.0.1:ephemeral)
  → 토큰 파일 삭제
```

**환경변수:**
- `HTTPS_PROXY` / `https_proxy` — 프록시 주소
- `SSL_CERT_FILE` / `NODE_EXTRA_CA_CERTS` — CA 번들
- `NO_PROXY` — 루프백, RFC1918, IMDS, Anthropic API, GitHub 등

### relay.ts — CONNECT-over-WebSocket 릴레이

```typescript
encodeChunk(data)              // UpstreamProxyChunk protobuf 인코딩
decodeChunk(buf)               // protobuf 디코딩
startUpstreamProxyRelay(opts)  // 릴레이 서버 시작
startNodeRelay(wsUrl, ...)     // Node.js 구현 (테스트용)
```

**릴레이 아키텍처:**
```
curl/gh/kubectl → HTTP CONNECT → 127.0.0.1:{port}
  Phase 1: CONNECT 헤더 수집 (CRLF CRLF 종료)
  Phase 2: WebSocket으로 바이트 터널링 (protobuf)
    → CCR 서버
```

**에러 처리:** 405 (비-CONNECT), 400 (8KB 초과 헤더), 502 (WS 실패)

## 의존성

| 모듈 | 용도 |
|------|------|
| [[utils/cleanupRegistry]] | 정리 콜백 등록 |
| [[utils/mtls]] | WebSocket TLS 옵션 |
| [[utils/proxy]] | WebSocket 프록시 에이전트 |
| [[utils/debug]] | 디버그 로깅 |

## 관계

- **호출자**: [[entrypoints/init]] → `initUpstreamProxy()` 호출
- **소비자**: [[utils/subprocessEnv]] → `getUpstreamProxyEnv()` 호출
- **역할**: 모든 자식 프로세스 (Bash, MCP, LSP, Hook)의 HTTPS 트래픽을 CCR 서버로 터널링
