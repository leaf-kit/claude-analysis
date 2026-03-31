---
tags:
  - module
  - coordinator
  - multi-agent
  - orchestration
created: 2026-03-31
type: source-analysis
status: complete
---

# coordinatorMode.ts

> 멀티 워커 에이전트 오케스트레이션 모드의 중심 모듈

## 개요

| 항목 | 값 |
|------|-----|
| 파일 경로 | `src/coordinator/coordinatorMode.ts` |
| 주요 역할 | 코디네이터 모드 감지, 시스템 프롬프트, 워커 컨텍스트 |
| 파일 크기 | 19,020 bytes |
| 의존 모듈 수 | 12 (내부) |

코디네이터 모드에서 Claude는 **오케스트레이터** 역할을 하여 여러 워커 에이전트에게 작업을 분배하고 결과를 종합합니다.

## 핵심 Export

### 모드 감지
```typescript
isCoordinatorMode(): boolean
// COORDINATOR_MODE 피처 플래그 + CLAUDE_CODE_COORDINATOR_MODE 환경변수
```

### 세션 동기화
```typescript
matchSessionMode(sessionMode: 'coordinator' | 'normal' | undefined): string | undefined
// 세션 재개 시 모드 불일치 감지 및 동기화
// tengu_coordinator_mode_switched 이벤트 로깅
```

### 워커 컨텍스트
```typescript
getCoordinatorUserContext(mcpClients, scratchpadDir?): { [k: string]: string }
// 워커에게 사용 가능한 도구 목록 제공
// INTERNAL_WORKER_TOOLS 제외, MCP 서버명 포함
```

### 시스템 프롬프트 (368줄)
```typescript
getCoordinatorSystemPrompt(): string
// 6개 섹션: 역할, 도구, 워커, 워크플로, 프롬프트 작성 가이드, 예시
```

## 내부 도구 분리

| 코디네이터 전용 도구 | 설명 |
|---------------------|------|
| `TEAM_CREATE_TOOL_NAME` | 팀 생성 |
| `TEAM_DELETE_TOOL_NAME` | 팀 삭제 |
| `SEND_MESSAGE_TOOL_NAME` | 에이전트 간 통신 |
| `SYNTHETIC_OUTPUT_TOOL_NAME` | 합성 출력 |

## 워크플로 단계

```
1. Research → 워커들이 병렬로 코드 탐색
2. Synthesis → 코디네이터가 결과 종합
3. Implementation → 워커들이 변경 수행
4. Verification → 워커가 테스트/검증
```

## 의존성

### 도구 상수 (10개)
- [[tools/AgentTool]], [[tools/BashTool]], [[tools/FileEditTool]], [[tools/FileReadTool]]
- [[tools/SendMessageTool]], [[tools/SyntheticOutputTool]], [[tools/TaskStopTool]]
- [[tools/TeamCreateTool]], [[tools/TeamDeleteTool]]

### 서비스 & 유틸리티
- [[services/analytics/growthbook]] — 피처 게이트
- [[services/analytics/index]] — 이벤트 로깅
- [[utils/envUtils]] — 환경변수 읽기

## 소비자 (이 모듈을 import하는 곳)

| 모듈 | 용도 |
|------|------|
| [[tools/AgentTool/resumeAgent]] | 워커 재개 로직 |
| [[tools/AgentTool/forkSubagent]] | fork 비활성화 (상호 배타) |
| [[utils/systemPrompt]] | 코디네이터 시스템 프롬프트 주입 |
| [[utils/toolPool]] | 코디네이터/워커 도구 필터링 |
| [[cli/print]] | 세션 재개 시 모드 표시 |
| [[QueryEngine]] | 워커 도구 컨텍스트 주입 |
| [[tools.ts]] | 조건부 도구 추가 |

## 관계

- **상위**: [[Query_Engine]] → 코디네이터 컨텍스트 병합
- **하위**: [[tools/AgentTool]] → 워커 생성/통신
- **보안**: [[hooks/toolPermission/coordinatorHandler]] → 워커 권한 처리
