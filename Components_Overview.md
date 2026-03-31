---
tags:
  - components
  - UI
  - React
created: 2026-03-31
type: module-analysis
status: complete
---

# Components Overview

> Claude Code의 UI 컴포넌트 시스템. React + [[Ink_Framework]] 기반 터미널 UI

## 컴포넌트 구조

```
components/
├── messages/        # 대화 메시지 표시 (36 files)
├── permissions/     # 권한 요청 다이얼로그 (32 files)
├── PromptInput/     # 프롬프트 입력 시스템
├── design-system/   # 디자인 시스템 (18 files)
├── diff/            # 디프 뷰어 (3 files)
├── agents/          # 에이전트 관리 UI (16 files)
├── mcp/             # MCP 서버 UI (15 files)
├── tasks/           # 백그라운드 태스크 표시 (10+ files)
├── Settings/        # 설정 패널
├── HelpV2/          # 도움말 시스템
├── ui/              # 기본 UI 유틸리티
├── hooks/           # 컴포넌트 훅
├── grove/           # Grove 연결
├── memory/          # 메모리 UI
├── skills/          # 스킬 UI
├── teams/           # 팀 UI
└── ... (기타)
```

## 핵심 컴포넌트

### 1. 최상위 앱 구조

| 컴포넌트 | 파일 | 용도 |
|----------|------|------|
| App | `App.tsx` | 최상위 래퍼 (FPS, 통계, 상태) |
| REPL | `screens/REPL.tsx` | 메인 대화형 루프 화면 |
| FullscreenLayout | `FullscreenLayout.tsx` | 풀스크린 레이아웃 (스크롤, 모달) |
| StatusLine | `StatusLine.tsx` | 상태바 (모델, 권한, 토큰) |
| MessageResponse | `MessageResponse.tsx` | 어시스턴트 응답 래퍼 (⎿ 표시) |

### 2. [[components/Messages]] — 메시지 시스템 (36 files)

**어시스턴트 메시지:**
- `AssistantTextMessage.tsx` — 텍스트 응답
- `AssistantToolUseMessage.tsx` — 도구 호출 표시 (~45KB)
- `AssistantThinkingMessage.tsx` — 사고 과정 표시
- `AssistantRedactedThinkingMessage.tsx` — 수정된 사고 표시

**사용자 메시지:**
- `UserPromptMessage.tsx` — 사용자 입력
- `UserBashInputMessage.tsx` / `UserBashOutputMessage.tsx` — 셸 명령/출력
- `UserCommandMessage.tsx` — 명령 실행
- `UserImageMessage.tsx` — 이미지 첨부
- `UserChannelMessage.tsx` — 채널 메시지

**도구 결과 메시지 (`UserToolResultMessage/`):**
- `UserToolSuccessMessage.tsx` — 성공
- `UserToolErrorMessage.tsx` — 에러
- `UserToolRejectMessage.tsx` — 거부
- `UserToolCanceledMessage.tsx` — 취소

**시스템 메시지:**
- `SystemTextMessage.tsx` — 시스템 메시지 (~79KB)
- `SystemAPIErrorMessage.tsx` — API 에러
- `RateLimitMessage.tsx` — 레이트 리밋
- `AttachmentMessage.tsx` — 첨부파일 (~71KB)

### 3. [[components/Permissions]] — 권한 시스템 (32 files)

**핵심 컴포넌트:**
- `PermissionRequest.tsx` — 권한 요청 다이얼로그 (~33KB)
- `PermissionPrompt.tsx` — 권한 프롬프트 (~37KB)
- `PermissionDialog.tsx` — 모달 래퍼

**도구별 권한 컴포넌트:**

| 컴포넌트 | 대상 도구 |
|----------|-----------|
| `BashPermissionRequest/` | [[tools/BashTool]] |
| `FilePermissionDialog/` | 파일 접근 |
| `FileWritePermissionRequest/` | [[tools/FileWriteTool]] |
| `FileEditPermissionRequest/` | [[tools/FileEditTool]] |
| `WebFetchPermissionRequest/` | [[tools/WebFetchTool]] |
| `SkillPermissionRequest/` | [[tools/SkillTool]] |
| `NotebookEditPermissionRequest/` | NotebookEditTool |
| `ComputerUseApproval/` | 컴퓨터 사용 |
| `PowerShellPermissionRequest/` | PowerShellTool |

### 4. [[components/PromptInput]] — 프롬프트 입력

**파일 구성:**
- `PromptInput.tsx` — 메인 입력 (자동완성, 제안, 음성 통합)
- `PromptInputFooter.tsx` — 하단 상태 표시
- `PromptInputHelpMenu.tsx` — 키보드 단축키 도움말
- `PromptInputModeIndicator.tsx` — 모드 표시 (normal, shell, plan)
- `Notifications.tsx` — 토스트/알림
- `VoiceIndicator.tsx` — 음성 입력 상태
- `HistorySearchInput.tsx` — 히스토리 검색

### 5. [[components/DesignSystem]] — 디자인 시스템 (18 files)

| 컴포넌트 | 용도 |
|----------|------|
| `Dialog.tsx` | 모달 다이얼로그 |
| `FuzzyPicker.tsx` | 퍼지 검색 선택기 (~40KB) |
| `Tabs.tsx` | 탭 네비게이션 (~41KB) |
| `ListItem.tsx` | 리스트 아이템 렌더링 |
| `ThemedBox.tsx` / `ThemedText.tsx` | 테마 적용 스타일링 |
| `ThemeProvider.tsx` | 테마 컨텍스트 |
| `ProgressBar.tsx` | 프로그레스 바 |
| `StatusIcon.tsx` | 상태 아이콘 |
| `KeyboardShortcutHint.tsx` | 단축키 힌트 |

### 6. 기타 주요 컴포넌트

| 카테고리 | 컴포넌트 | 용도 |
|----------|----------|------|
| 다이얼로그 | `QuickOpenDialog.tsx` | 빠른 파일/명령 열기 |
| 다이얼로그 | `BridgeDialog.tsx` | IDE 브릿지 연결 (~34KB) |
| 시각화 | `ContextVisualization.tsx` | 컨텍스트 윈도우 시각화 (~76KB) |
| 상태 | `CompactSummary.tsx` | 압축 요약 표시 |
| 진행률 | `AgentProgressLine.tsx` | 에이전트 진행 상황 |
| 업데이트 | `AutoUpdater.tsx` | 자동 업데이트 (~30KB) |
| 디프 | `DiffDialog.tsx` | 디프 뷰어 (~43KB) |

## 컴포넌트 렌더링 흐름

```
REPL.tsx (메인 화면)
  │
  ├── FullscreenLayout
  │     ├── ScrollBox (스크롤 가능 영역)
  │     │     ├── Message 컴포넌트들 (대화 내역)
  │     │     │     ├── UserPromptMessage
  │     │     │     ├── AssistantTextMessage
  │     │     │     ├── AssistantToolUseMessage
  │     │     │     └── UserToolResultMessage
  │     │     └── AgentProgressLine
  │     │
  │     ├── PromptInput (하단 고정 입력)
  │     │     ├── BaseTextInput
  │     │     ├── PromptInputFooter
  │     │     └── Notifications
  │     │
  │     └── StatusLine (상태바)
  │
  └── 모달 다이얼로그
        ├── PermissionRequest (권한)
        ├── QuickOpenDialog (빠른 열기)
        └── DiffDialog (디프)
```

## 통계

| 항목 | 값 |
|------|-----|
| 총 파일 수 | 389 |
| 메시지 컴포넌트 | 36 |
| 권한 컴포넌트 | 32 |
| 디자인 시스템 | 18 |
| 에이전트 UI | 16 |
| MCP UI | 15 |

## 관련 문서
- [[Index]] — 전체 MOC
- [[Ink_Framework]] — Ink 터미널 렌더링
- [[Tools_Overview]] — 도구 시스템
- [[Hooks_Overview]] — 훅 시스템
