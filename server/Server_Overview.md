---
tags:
  - module
  - server
  - websocket
  - direct-connect
  - remote
created: 2026-03-31
type: source-analysis
status: complete
---

# 서버 모듈 (Server)

> 직접 연결 WebSocket 세션 관리 (3파일)

## 개요

| 항목 | 값 |
|------|-----|
| 디렉터리 | `src/server/` |
| 파일 수 | 3 |
| 통신 방식 | WebSocket (양방향) |

## 파일 구조

### types.ts — 타입 & 스키마
- `connectResponseSchema` — Zod 검증 (session_id, ws_url, work_dir)
- `ServerConfig` — 서버 인스턴스 설정
- `SessionState` — `'starting' | 'running' | 'detached' | 'stopping' | 'stopped'`
- `SessionInfo` — 런타임 세션 메타데이터
- `SessionIndexEntry` / `SessionIndex` — 세션 영속 인덱스

### createDirectConnectSession.ts — 세션 생성 팩토리
```typescript
DirectConnectError     // 커스텀 에러 타입
createDirectConnectSession()  // POST /sessions → DirectConnectConfig
```

### directConnectManager.ts — WebSocket 세션 관리자
```typescript
class DirectConnectSessionManager {
  connect()           // WebSocket 연결
  sendMessage(content)      // 사용자 메시지 전송
  respondToPermissionRequest(requestId, result) // 권한 응답
  sendInterrupt()     // 인터럽트 신호
  disconnect()        // 연결 종료
  isConnected()       // 연결 상태 확인
}
```

**메시지 라우팅:**
- `control_request` → `onPermissionRequest` 콜백
- 내부 메시지 필터링 (keep_alive, streamlined_*, post_turn_summary)
- SDK 메시지 → `onMessage` 콜백

## 의존성

| 모듈 | 용도 |
|------|------|
| [[entrypoints/agentSdkTypes]] | `SDKMessage` 타입 |
| [[entrypoints/sdk/controlTypes]] | 권한 요청 타입 |
| [[remote/RemoteSessionManager]] | `RemotePermissionResponse` |
| [[utils/debug]], [[utils/slowOperations]] | 디버깅, JSON 처리 |

## 관계

- **소비자**: `src/hooks/useDirectConnect.ts` — React 훅에서 인스턴스 생성
- **연동**: [[remote/RemoteSessionManager]] — 원격 세션 관리
