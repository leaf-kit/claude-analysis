---
tags:
  - module
  - screens
  - react
  - REPL
  - UI
created: 2026-03-31
type: source-analysis
status: complete
---

# 화면 시스템 (Screens)

> 3개 핵심 React 화면 컴포넌트 (REPL, 세션 재개, 진단)

## 개요

| 항목 | 값 |
|------|-----|
| 디렉터리 | `src/screens/` |
| 파일 수 | 3 |

## 파일 구조

### 1. REPL.tsx — 핵심 대화형 루프 (~6,500줄)

Claude Code의 **메인 인터랙션 엔진**. 사용자 입력 → AI 응답 → 도구 실행의 전체 루프를 관리합니다.

```typescript
export type Screen = 'prompt' | 'transcript'
export function REPL(props: Props): React.ReactNode
```

**주요 기능:**
- 스트리밍 API 응답 처리
- 도구 실행 및 권한 관리
- 세션 관리 및 복구
- 비용/토큰 추적
- 키보드 입력 처리
- 메시지 히스토리 탐색
- Swarm/팀 협업

**핵심 의존성:**
- [[state/AppState]], [[Query_Engine]], [[Tools_Overview]]
- [[components/Messages]], [[components/PromptInput]], [[components/Permissions]]
- [[keybindings/Keybindings_Overview|KeybindingHandlers]]

---

### 2. ResumeConversation.tsx — 세션 재개 화면

이전 대화 세션을 로드하고 REPL로 전환합니다.

```typescript
export function ResumeConversation(props: Props): React.ReactNode
```

**흐름:**
```
LogSelector (세션 목록)
  → loadSameRepoMessageLogsProgressive()
  → checkCrossProjectResume()
  → restoreSessionMetadata()
  → restoreAgentFromSession()
    → REPL 컴포넌트로 전환
```

**핵심 의존성:**
- [[state/AppState]], [[services/SessionMemory_Service]]
- `LogSelector` 컴포넌트, REPL 컴포넌트

---

### 3. Doctor.tsx — 진단 화면

`/doctor` 명령어로 접근하는 시스템 건강 진단 화면입니다.

```typescript
export function Doctor(props: { onDone: Function }): React.ReactNode
```

**표시 정보:**
- 모델 설정 및 컨텍스트 경고
- 버전 정보 (stable/latest)
- 설정 검증 오류
- 에이전트 정의, 환경변수, 샌드박스 상태
- 키바인딩 경고, MCP 파싱 경고

**핵심 의존성:**
- [[utils/settings]], [[state/AppState]]
- `KeybindingWarnings`, `McpParsingWarnings`, `SandboxDoctorSection`

## 관계

```
replLauncher.tsx → REPL.tsx (메인 루프)
                → ResumeConversation.tsx → REPL.tsx (재개 후)
commands/doctor → Doctor.tsx (진단)
```

- **REPL**: [[replLauncher.tsx]]에서 래핑 및 실행
- **ResumeConversation**: REPL의 전환 단계
- **Doctor**: [[commands/doctor]]에서 렌더링
