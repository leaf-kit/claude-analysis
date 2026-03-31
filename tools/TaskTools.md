---
tags:
  - tool
  - task
  - background
  - management
created: 2026-03-31
type: tool-analysis
status: complete
---

# Task Tools

> 태스크 관리 도구 모음. 생성, 업데이트, 조회, 중지

## 도구 목록

| 도구 | 파일 | 읽기전용 | 용도 |
|------|------|----------|------|
| TaskCreateTool | `TaskCreateTool/TaskCreateTool.ts` | ❌ | 태스크 생성 |
| TaskUpdateTool | `TaskUpdateTool/TaskUpdateTool.ts` | ❌ | 상태/메타 업데이트 |
| TaskListTool | `TaskListTool/TaskListTool.ts` | ✅ | 태스크 목록 |
| TaskGetTool | `TaskGetTool/` | ✅ | 태스크 상세 조회 |
| TaskStopTool | `TaskStopTool/` | ❌ | 태스크 중지 |
| TaskOutputTool | `TaskOutputTool/` | ✅ | 태스크 출력 관리 |

## Task.ts — 태스크 베이스

**태스크 타입:**
- `local_bash` — 로컬 셸 태스크
- `local_agent` — 로컬 에이전트 태스크
- `remote_agent` — 원격 에이전트 태스크
- `in_process_teammate` — 프로세스 내 팀메이트
- `dream` — 드림 태스크

**태스크 상태 워크플로우:**
```
pending → running → completed / failed / killed
```

**핵심 유틸리티:**
- `generateTaskId()` — 고유 ID 생성 (타입별 프리픽스)
- `createTaskStateBase()` — 태스크 상태 팩토리
- `isTerminalTaskStatus()` — 종료 상태 확인

## 태스크 의존성

```typescript
// TaskUpdateTool로 의존성 설정
{
  taskId: "2",
  addBlockedBy: ["1"]  // 태스크 1이 완료되어야 2 시작
}
```

## 의존 모듈

- `src/Task.ts` — 태스크 인터페이스
- `src/tasks.ts` — 태스크 레지스트리
- [[utils/Task_Utils|utils/task/]] — 태스크 상태 관리
- [[Hooks_Overview|hooks/useTasksV2.ts]] — 태스크 리스트 훅

## 관련 문서
- [[Tools_Overview]] — 도구 전체 개요
- [[Hooks_Overview]] — 훅 시스템
