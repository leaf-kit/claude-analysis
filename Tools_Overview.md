---
tags:
  - tools
  - core
  - architecture
created: 2026-03-31
type: module-analysis
status: complete
---

# Tools Overview

> Claude Code의 40개 도구 시스템 전체 개요

## 아키텍처

### Tool.ts — 도구 베이스 인터페이스

모든 도구가 구현해야 하는 핵심 인터페이스:

```typescript
interface Tool<Input, Output, Progress> {
  name: string;
  description(): string;
  call(input: Input, context: ToolUseContext): Promise<Output>;
  checkPermissions(input: Input, context: ToolPermissionContext): PermissionResult;
  isReadOnly(): boolean;
  isConcurrencySafe(): boolean;
  isDestructive(): boolean;
  // ... 렌더링 메서드
}
```

**팩토리 함수:**
- `buildTool(def: ToolDef)` — 도구 정의 → 완전한 도구 빌드
- `findToolByName(tools, name)` — 이름/별칭으로 도구 검색

### tools.ts — 도구 레지스트리

**핵심 함수:**
- `getAllBaseTools()` — 모든 내장 도구 반환
- `getTools()` — 권한 컨텍스트 + 모드 필터 적용
- `assembleToolPool()` — 내장 + [[services/MCP_Service|MCP]] 도구 결합 (중복 제거)
- `filterToolsByDenyRules()` — 권한 거부 규칙 필터링

**도구 프리셋:** `TOOL_PRESETS` 상수로 사전 정의된 도구 조합

## 도구 카탈로그 (40개)

### 파일 시스템 도구

| 도구 | 용도 | 읽기전용 | 동시성 |
|------|------|----------|--------|
| [[tools/FileReadTool]] | 파일 읽기 (PDF, 이미지, 노트북 지원) | ✅ | ✅ |
| [[tools/FileEditTool]] | 문자열 찾기/바꾸기로 파일 편집 | ❌ | ❌ |
| [[tools/FileWriteTool]] | 파일 생성/덮어쓰기 | ❌ | ❌ |
| [[tools/GlobTool]] | Glob 패턴으로 파일 검색 | ✅ | ✅ |
| [[tools/GrepTool]] | ripgrep 기반 콘텐츠 검색 | ✅ | ✅ |

### 셸 실행 도구

| 도구 | 용도 | 읽기전용 | 동시성 |
|------|------|----------|--------|
| [[tools/BashTool]] | Bash 명령 실행 (보안 검증 + 샌드박싱) | ❌ | ❌ |
| PowerShellTool | Windows PowerShell 실행 | ❌ | ❌ |

### 에이전트 / 멀티에이전트 도구

| 도구 | 용도 | 읽기전용 | 동시성 |
|------|------|----------|--------|
| [[tools/AgentTool]] | 서브에이전트 생성 (워크트리 격리 지원) | ❌ | ✅ |
| [[tools/SendMessageTool]] | 에이전트 간 메시지 전달 | ❌ | ❌ |
| [[tools/SkillTool]] | 스킬 (커스텀 명령/프롬프트) 실행 | ❌ | ❌ |

### 웹 도구

| 도구 | 용도 | 읽기전용 | 동시성 |
|------|------|----------|--------|
| [[tools/WebFetchTool]] | 웹 콘텐츠 가져오기 (HTML→Markdown 변환) | ✅ | ✅ |
| [[tools/WebSearchTool]] | 웹 검색 (도메인 필터링) | ✅ | ✅ |

### 태스크 관리 도구

| 도구 | 용도 | 읽기전용 | 동시성 |
|------|------|----------|--------|
| [[tools/TaskTools\|TaskCreateTool]] | 태스크 생성 | ❌ | ❌ |
| [[tools/TaskTools\|TaskUpdateTool]] | 태스크 상태/메타 업데이트 | ❌ | ❌ |
| [[tools/TaskTools\|TaskListTool]] | 태스크 목록 조회 | ✅ | ✅ |
| [[tools/TaskTools\|TaskGetTool]] | 태스크 상세 조회 | ✅ | ✅ |
| [[tools/TaskTools\|TaskStopTool]] | 태스크 중지 | ❌ | ❌ |
| [[tools/TaskTools\|TaskOutputTool]] | 태스크 출력 관리 | ✅ | ✅ |

### MCP (Model Context Protocol) 도구

| 도구 | 용도 | 읽기전용 | 동시성 |
|------|------|----------|--------|
| [[tools/MCPTool]] | MCP 서버 도구 실행 | 가변 | 가변 |
| ListMcpResourcesTool | MCP 리소스 목록 | ✅ | ✅ |
| ReadMcpResourceTool | MCP 리소스 읽기 | ✅ | ✅ |
| McpAuthTool | MCP 인증 관리 | ❌ | ❌ |

### 코드 인텔리전스 도구

| 도구 | 용도 | 읽기전용 | 동시성 |
|------|------|----------|--------|
| [[tools/LSPTool]] | 정의 이동, 참조 찾기, 호버, 심볼 | ✅ | ✅ |

### 워크트리 / 플랜 도구

| 도구 | 용도 |
|------|------|
| EnterWorktreeTool | Git 워크트리 격리 진입 |
| ExitWorktreeTool | 워크트리 종료/정리 |
| EnterPlanModeTool | 플랜(승인) 모드 진입 |
| ExitPlanModeTool | 플랜 모드 종료 |

### 스케줄링 도구

| 도구 | 용도 |
|------|------|
| [[tools/ScheduleCronTool]] | 크론 스케줄 생성 |
| CronDeleteTool | 크론 스케줄 삭제 |
| CronListTool | 크론 스케줄 목록 |

### 기타 도구

| 도구 | 용도 |
|------|------|
| ConfigTool | Claude Code 설정 관리 |
| NotebookEditTool | 주피터 노트북 셀 편집 |
| RemoteTriggerTool | 원격 에이전트 트리거 관리 |
| BriefTool | 사용자에게 간략 메시지 전송 |
| AskUserQuestionTool | 사용자에게 질문 (다중선택 지원) |
| SleepTool | 실행 일시 중지 |
| ToolSearchTool | 도구 검색 및 발견 |
| TodoWriteTool | 레거시 할 일 관리 |
| TeamCreateTool | 팀 생성 |
| TeamDeleteTool | 팀 삭제 |

## 도구 실행 흐름

```
모델 응답에서 tool_use 블록 추출
  │
  ▼
tools.ts에서 도구 조회 (findToolByName)
  │
  ▼
Tool.checkPermissions() → [[utils/Permissions_System]]
  ├── 허용 → Tool.call() 실행
  ├── 거부 → 거부 메시지 반환
  └── 문의 → 사용자에게 권한 요청 다이얼로그
  │
  ▼
[[services/Tools_Service|StreamingToolExecutor]]
  ├── 병렬 실행 가능 도구 분류
  ├── 동시성 안전 도구는 병렬 실행
  └── 결과를 메시지에 추가
```

## 공통 패턴

1. **Zod 스키마 검증**: 모든 도구 입출력에 Zod v4 스키마 적용
2. **지연 스키마 평가**: 임포트 사이클 방지를 위한 지연 로딩
3. **Feature Flag 기반 가용성**: 특정 도구는 feature flag로 제어
4. **결과 크기 제한**: `maxResultSizeChars`로 출력 크기 제한

## 통계

| 항목 | 값 |
|------|-----|
| 총 도구 수 | 40 |
| 파일 시스템 도구 | 5 |
| 실행 도구 | 2 |
| 에이전트 도구 | 3 |
| 웹 도구 | 2 |
| 태스크 도구 | 6 |
| MCP 도구 | 4 |
| 총 파일 수 | 184 |

## 관련 문서
- [[Index]] — 전체 MOC
- [[Query_Engine]] — 쿼리 엔진
- [[services/Tools_Service]] — 도구 실행 서비스
- [[utils/Permissions_System]] — 권한 시스템
