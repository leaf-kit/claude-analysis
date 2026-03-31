---
tags:
  - entrypoints
  - bootstrap
  - initialization
created: 2026-03-31
type: module-analysis
status: complete
---

# Entrypoints Overview

> 앱 시작부터 대화형 루프 진입까지의 부트스트랩 흐름

## 초기화 시퀀스

```
cli.tsx → init.ts → main.tsx → setup.ts → REPL
```

## 파일별 분석

### 1. `entrypoints/cli.tsx` — CLI 부트스트랩

**목적:** 최소한의 모듈 로딩으로 빠른 시작을 보장하는 CLI 진입점

**빠른 경로(Fast-path) 처리:**
- `--version`: 모듈 로딩 없이 즉시 버전 출력
- `--dump-system-prompt`: 시스템 프롬프트 렌더링
- MCP 서버 경로 (Chrome, Computer Use)
- 데몬 워커 스포닝

**핵심 로직:**
```typescript
export async function main() {
  // 프로파일링 체크포인트로 시작 성능 측정
  // Feature flag 기반 분기
  // 동적 임포트로 지연 로딩
}
```

**연결 모듈:**
- → [[Query_Engine]] (메인 실행 루프)
- → [[Ink_Framework]] (터미널 렌더링)

---

### 2. `entrypoints/init.ts` — 종합 초기화

**목적:** 설정 검증, 환경 설정, 서비스 초기화

**초기화 단계:**

| 순서 | 단계 | 설명 |
|------|------|------|
| 1 | 설정 검증 | Config 시스템 활성화 및 유효성 검사 |
| 2 | 환경 변수 | 안전한 환경 변수 적용 (Trust 다이얼로그 이전) |
| 3 | CA 인증서 | TLS 인증서 설정 |
| 4 | 종료 핸들러 | Graceful shutdown 핸들러 등록 |
| 5 | OAuth | 계정 정보 및 인증 초기화 |
| 6 | 정책 | [[services/API_Service|원격 관리 설정]] 및 정책 제한 |
| 7 | mTLS | HTTP 에이전트 설정 |
| 8 | API 사전연결 | API 서버와의 프리커넥션 |
| 9 | 업스트림 프록시 | CCR 모드 프록시 설정 |
| 10 | LSP 정리 | [[services/LSP_Service|LSP 매니저]] 클린업 등록 |

**핵심 의존성:**
- [[services/OAuth_Service]] — 인증
- [[services/Analytics_Service]] — 텔레메트리
- [[utils/Settings_Management]] — 설정 관리
- [[utils/Permissions_System]] — 정책 제한

---

### 3. `main.tsx` — 메인 부트스트랩

**목적:** 전체 앱의 핵심 초기화 (텔레메트리, 서비스)

**주요 export:**
- `init()` — 메모이즈된 비동기 초기화 함수
- `initializeTelemetryAfterTrust()` — Trust 확인 후 텔레메트리 설정
- `setMeterState()` — OpenTelemetry 미터 설정

**텔레메트리 설정:**
```typescript
// OpenTelemetry SDK는 지연 로딩
// Trust 다이얼로그 이후에만 초기화
async function doInitializeTelemetry() {
  // Lazy-load OpenTelemetry SDK
  // 메트릭 수집 설정
}
```

---

### 4. `setup.ts` — 세션 셋업

**목적:** 초기화 이후, 첫 쿼리 이전에 수행하는 세션 수준 설정

**셋업 단계:**

| 순서 | 단계 | 설명 |
|------|------|------|
| 1 | Node 버전 검증 | 호환 버전 확인 |
| 2 | 세션 ID | 커스텀 세션 ID 처리 |
| 3 | UDS 서버 | 유닉스 도메인 소켓 메시지 서버 |
| 4 | 워크트리 | `--worktree` 옵션 시 Git 워크트리 생성 |
| 5 | 백그라운드 작업 | [[services/SessionMemory_Service|세션 메모리]], 컨텍스트 축소 |
| 6 | 프리페치 | 명령어/플러그인/훅 사전 로딩 |
| 7 | 분석 싱크 | [[services/Analytics_Service|분석 싱크]] 연결 |
| 8 | 권한 검증 | Root/sudo 체크, Docker/sandbox 검증 |
| 9 | 세션 메트릭 | 마지막 세션 메트릭 로깅 |

**워크트리 생성 흐름:**
```
Git 루트 해석 → Tmux 세션 생성 → 훅 위임 → 워크트리 초기화
```

## 통계

| 항목 | 값 |
|------|-----|
| 총 파일 수 | 8 |
| 진입점 유형 | CLI, SDK, MCP, sandbox |
| 초기화 단계 | ~15단계 |
| 지연 로딩 모듈 | OpenTelemetry, 1P 이벤트 로깅 |

## 관련 문서
- [[Index]] — 전체 MOC
- [[Query_Engine]] — 쿼리 엔진
- [[REPL_Screen]] — REPL 화면
- [[services/OAuth_Service]] — OAuth 서비스
