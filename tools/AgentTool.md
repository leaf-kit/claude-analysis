---
tags:
  - tool
  - agent
  - multi-agent
  - concurrency
created: 2026-03-31
type: tool-analysis
status: complete
---

# AgentTool

> 서브에이전트 생성 도구. 복잡한 작업을 자율적으로 처리하는 전문 에이전트 스폰

## 개요

| 항목 | 값 |
|------|-----|
| 파일 | `src/tools/AgentTool/AgentTool.tsx` |
| 읽기전용 | ❌ |
| 동시성 안전 | ✅ |
| 파괴적 | ❌ |

## 핵심 기능

1. **서브에이전트 스폰**: 독립적인 에이전트 프로세스 생성
2. **격리 모드**: 워크트리 격리, 원격 실행 지원
3. **에이전트 타입**: general-purpose, Explore, Plan 등 전문화된 에이전트
4. **모델 오버라이드**: 에이전트별 다른 모델 사용 가능
5. **백그라운드 실행**: 비동기 백그라운드 태스크 지원
6. **메모리 및 진행 추적**: 에이전트 작업 상태 추적

## 에이전트 유형

| 유형 | 용도 | 사용 가능 도구 |
|------|------|---------------|
| `general-purpose` | 범용 멀티스텝 작업 | 모든 도구 |
| `Explore` | 코드베이스 탐색 | 읽기 전용 도구 |
| `Plan` | 구현 계획 설계 | 읽기 전용 도구 |

## 내장 에이전트 정의

```
src/tools/AgentTool/built-in/
├── 사전 정의된 에이전트 타입
└── 에이전트별 시스템 프롬프트
```

## 워크트리 격리

```
isolation: "worktree"
  │
  ▼
임시 Git 워크트리 생성
  │
  ▼
격리된 저장소 복사본에서 에이전트 실행
  │
  ▼
변경사항 없으면 자동 정리 / 있으면 경로와 브랜치 반환
```

## 의존 모듈

- [[services/Tools_Service]] — 도구 실행
- [[utils/Permissions_System]] — 권한 체크
- `shared/spawnMultiAgent.js` — 멀티에이전트 스폰
- [[tools/SendMessageTool]] — 에이전트 간 통신

## 관련 문서
- [[Tools_Overview]] — 도구 전체 개요
- [[tools/SendMessageTool]] — 에이전트 간 메시지
- [[Hooks_Overview]] — 훅 시스템 (Swarm)
