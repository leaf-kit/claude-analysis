---
tags:
  - hooks
  - React
  - permission
  - state-management
created: 2026-03-31
type: module-analysis
status: complete
---

# Hooks Overview

> Claude Code의 React 훅 시스템 (104 files). 권한, 태스크, UI, 입력, 세션 관리

## 훅 구조

```
hooks/
├── toolPermission/          # 도구 권한 훅
│   ├── PermissionContext.ts  # 권한 요청 오케스트레이션
│   ├── permissionLogging.ts  # 권한 결정 분석
│   └── handlers/
│       ├── interactiveHandler.ts   # 인터랙티브 권한 흐름
│       ├── coordinatorHandler.ts   # 코디네이터 권한 흐름
│       └── swarmWorkerHandler.ts   # Swarm 워커 권한 흐름
│
├── notifs/                  # 알림 훅
│   ├── useStartupNotification.ts
│   ├── useTeammateShutdownNotification.ts
│   └── useAutoModeUnavailableNotification.ts
│
├── useQueueProcessor.ts     # 명령 큐 처리
├── useTasksV2.ts            # 태스크 목록 관리
├── useMergedTools.ts        # 도구 풀 조립
├── useMergedCommands.ts     # 명령어 병합
├── useManagePlugins.ts      # 플러그인 라이프사이클
├── useTextInput.ts          # 텍스트 입력
├── useVimInput.ts           # Vim 모드 입력
├── useHistorySearch.ts      # 히스토리 검색
├── useTerminalSize.ts       # 터미널 크기
├── useElapsedTime.ts        # 경과 시간
├── useVoice.ts              # 음성 입출력
├── useRemoteSession.ts      # 원격 세션
├── useSwarmInitialization.ts # Swarm 초기화
└── ... (기타)
```

## 핵심 훅 상세

### 1. [[hooks/ToolPermission]] — 도구 권한 훅

**핵심: `PermissionContext.ts`**

```
도구 실행 요청
  │
  ▼
createPermissionContext()
  │
  ├── 핸들러 전략 선택:
  │   ├── interactiveHandler (메인 에이전트)
  │   ├── coordinatorHandler (팀 코디네이터)
  │   └── swarmWorkerHandler (분산 워커)
  │
  ├── createResolveOnce() — 단일 해결 보장 (이중 해결 방지)
  │
  └── ToolUseConfirm 큐 관리
```

**인터랙티브 핸들러 흐름:**
```
훅 + 분류기 vs 사용자 인터랙션 경합(Race)
  ├── 훅/분류기 승리 → 자동 결정
  └── 사용자 승리 → 다이얼로그 응답
```

**Swarm 워커 핸들러:**
```
분류기 자동 승인 시도
  ├── 성공 → 자동 허용
  └── 실패 → 리더에게 메일박스로 전달
```

**권한 로깅 (`permissionLogging.ts`):**
- Statsig, OTel, code-edit 메트릭에 결정 로깅
- 승인/거부 소스 및 이유 추적
- 코드 편집 도구의 언어 감지

---

### 2. 큐 및 처리 훅

#### `useQueueProcessor.ts`

```typescript
// useSyncExternalStore로 반응적 큐 업데이트
// 우선순위: now > next > later
// 조건: 활성 쿼리 없음, 큐 항목 있음, JSX UI 차단 없음
```

#### `useTasksV2.ts`

```typescript
// 싱글톤 스토어 + fs.watch로 태스크 디렉터리 감시
// 디바운싱 (50ms) + 폴백 폴링 (5s)
// 모든 태스크 완료 시 5초 숨김 타이머
// LazyInit: 첫 구독자에서 초기화
```

---

### 3. 도구 및 명령 훅

#### `useMergedTools.ts`
- 내장 + [[services/MCP_Service|MCP]] 도구 결합
- 거부 규칙 적용 및 중복 제거
- `assembleToolPool()` 공유 함수 사용

#### `useMergedCommands.ts`
- CLI 명령어 + MCP 명령어 병합
- 이름 기반 중복 제거

#### `useManagePlugins.ts`
- 마운트 시 플러그인 초기 로드 (삭제, 플래깅, 알림)
- 명령어, 에이전트, 훅, MCP 서버, LSP 서버 로드
- 에러 집계 (Doctor UI용)

---

### 4. UI 및 인터랙션 훅

| 훅 | 용도 |
|----|------|
| `useTextInput.ts` | 텍스트 입력 처리 |
| `useVimInput.ts` | Vim 모드 입력 |
| `useHistorySearch.ts` | 히스토리 검색 (Ctrl+R) |
| `useTerminalSize.ts` | 터미널 크기 추적 |
| `useCopyOnSelect.ts` | 선택 복사 동작 |
| `useDoublePress.ts` | 더블 프레스 감지 |
| `useExitOnCtrlCD.ts` | Ctrl+C/D 종료 |

---

### 5. 세션 및 백그라운드 훅

| 훅 | 용도 |
|----|------|
| `useRemoteSession.ts` | 원격 세션 관리 |
| `useSessionBackgrounding.ts` | 세션 백그라운딩 |
| `useSwarmInitialization.ts` | Swarm 모드 초기화 |
| `useSwarmPermissionPoller.ts` | Swarm 권한 응답 폴링 |
| `useBackgroundTaskNavigation.ts` | 백그라운드 태스크 네비게이션 |

---

### 6. 상태 및 알림 훅

| 훅 | 용도 |
|----|------|
| `useElapsedTime.ts` | 경과 시간 추적 |
| `useTimeout.ts` | 타임아웃 관리 |
| `useMinDisplayTime.ts` | 최소 표시 시간 보장 |
| `useNotifyAfterTimeout.ts` | 지연 알림 |
| `usePrStatus.ts` | GitHub PR 상태 폴링 |
| `useUpdateNotification.ts` | CLI 업데이트 알림 |
| `useIssueFlagBanner.ts` | 이슈 플래그 알림 |
| `useIdeConnectionStatus.ts` | IDE 연결 상태 |

---

### 7. 초기화 훅

| 훅 | 용도 |
|----|------|
| `useFileHistorySnapshotInit.ts` | 파일 히스토리 초기화 |
| `useMainLoopModel.ts` | 메인 루프 모델 설정 |
| `useAfterFirstRender.ts` | 렌더 후 훅 |
| `useIdeSelection.ts` | IDE 선택 UI |
| `useSSHSession.ts` | SSH 세션 설정 |

## 아키텍처 패턴

### 1. 권한 흐름 아키텍처
```
모듈형 핸들러 (interactive/coordinator/swarmWorker)
  │
  ├── 훅/분류기 vs 사용자 인터랙션 경합(Race)
  │
  ├── ResolveOnce 가드 (이중 해결 방지)
  │
  └── 분석 통합 (결정 지점)
```

### 2. 싱글톤 스토어 + 파일 감시
```
useTasksV2: 싱글톤 스토어 → fs.watch → 디바운싱 → LazyInit
```

### 3. 반응적 큐
```
useQueueProcessor: useSyncExternalStore → 우선순위 기반 처리
```

## 통계

| 항목 | 값 |
|------|-----|
| 총 파일 수 | 104 |
| 권한 관련 | ~10 files |
| UI/인터랙션 | ~15 files |
| 세션/백그라운드 | ~10 files |
| 상태/알림 | ~10 files |
| 초기화 | ~8 files |

## 관련 문서
- [[Index]] — 전체 MOC
- [[Tools_Overview]] — 도구 시스템
- [[utils/Permissions_System]] — 권한 시스템
- [[Components_Overview]] — UI 컴포넌트
