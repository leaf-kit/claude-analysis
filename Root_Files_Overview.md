---
tags:
  - MOC
  - root-files
  - architecture
  - core
created: 2026-03-31
type: source-analysis
status: complete
---

# 루트 레벨 소스 파일 분석

> `src/` 디렉터리 루트에 위치한 18개 핵심 파일의 역할과 관계

## 아키텍처 다이어그램

```
┌─ 진입 & 초기화 ─────────────────────────────────┐
│  main.tsx → setup.ts → replLauncher.tsx          │
│              ↓                                    │
│  interactiveHelpers.tsx ← dialogLaunchers.tsx     │
└───────────────────┬──────────────────────────────┘
                    │
┌───────────────────▼──────────────────────────────┐
│  핵심 실행 루프                                    │
│  QueryEngine.ts → query.ts → context.ts          │
│       ↓              ↓                            │
│  commands.ts    tools.ts → Tool.ts               │
└───────────────────┬──────────────────────────────┘
                    │
┌───────────────────▼──────────────────────────────┐
│  지원 모듈                                        │
│  Task.ts / tasks.ts / cost-tracker.ts / history.ts│
│  costHook.ts / ink.ts / projectOnboardingState.ts │
└──────────────────────────────────────────────────┘
```

## 파일별 상세

### 진입 & 초기화 그룹

#### [[main.tsx]] — 애플리케이션 진입점
| 항목 | 값 |
|------|-----|
| 핵심 Export | `launchRepl()`, `startDeferredPrefetches()` |
| 역할 | REPL UI 실행, 마이그레이션, 서비스 초기화 |
| 주요 의존 | [[commands]], [[context]], [[costHook]], [[state/AppStateStore]] |

#### [[setup.ts]] — 세션 설정
| 항목 | 값 |
|------|-----|
| 핵심 Export | `setup()` |
| 역할 | Git/워크트리 설정, 명령어 로드, 플러그인, 권한 |
| 주요 의존 | [[bootstrap/state]], [[services]], [[utils]] |

#### [[replLauncher.tsx]] — REPL 지연 로딩
| 항목 | 값 |
|------|-----|
| 핵심 Export | `launchRepl()` |
| 역할 | 동적 import로 REPL 컴포넌트 마운트 |
| 주요 의존 | [[components/App]], [[screens/REPL]] |

#### [[interactiveHelpers.tsx]] — 인터랙티브 UI 유틸
| 항목 | 값 |
|------|-----|
| 핵심 Export | `showDialog()`, `renderAndRun()`, `showSetupScreens()` |
| 역할 | 다이얼로그 렌더링, 온보딩 흐름 |

#### [[dialogLaunchers.tsx]] — 다이얼로그 팩토리
| 항목 | 값 |
|------|-----|
| 핵심 Export | `launchSnapshotUpdateDialog()`, `launchResumeChooser()` 등 |
| 역할 | 모달 다이얼로그 실행 함수 모음 |

---

### 핵심 실행 루프 그룹

#### [[QueryEngine.ts]] — 쿼리 오케스트레이터
| 항목 | 값 |
|------|-----|
| 핵심 Export | `QueryEngineConfig`, `ask()` |
| 역할 | 재시도, 폴백, 에이전트 조율, 메모리 로딩 |
| 주요 의존 | [[query.ts]], [[services]], [[utils/systemPrompt]] |

#### [[query.ts]] — 쿼리 실행 엔진 (generator)
| 항목 | 값 |
|------|-----|
| 핵심 Export | `query()`, `QueryParams` |
| 역할 | 멀티턴 대화 실행 (API 호출 + 도구 실행) |
| 주요 의존 | [[Tool.ts]], [[context.ts]], [[services]] |

#### [[context.ts]] — 컨텍스트 준비
| 항목 | 값 |
|------|-----|
| 핵심 Export | `getSystemContext()`, `getUserContext()`, `getGitStatus()` |
| 역할 | 시스템/사용자 컨텍스트를 모든 대화에 주입 |
| 주요 의존 | [[bootstrap/state]], [[utils/claudemd]], [[utils/git]] |

#### [[commands.ts]] — 명령어 레지스트리
| 항목 | 값 |
|------|-----|
| 핵심 Export | `getCommands()`, `Command`, `findCommand()`, `getSkillToolCommands()` |
| 역할 | 내장 명령어 + 스킬 + 플러그인 통합 (100+ 명령어) |
| 주요 의존 | [[commands/]], [[skills/]], [[plugins/]] |

#### [[tools.ts]] — 도구 레지스트리
| 항목 | 값 |
|------|-----|
| 핵심 Export | `getAllBaseTools()`, `getTools()`, `assembleToolPool()`, `getMergedTools()` |
| 역할 | 내장 도구 + MCP 도구 통합, 권한 필터링, 프리셋 |
| 주요 의존 | [[Tool.ts]], [[tools/]], [[utils/permissions]] |

#### [[Tool.ts]] — 도구 인터페이스
| 항목 | 값 |
|------|-----|
| 핵심 Export | `Tool<I,O,P>`, `ToolDef`, `buildTool()`, `ToolUseContext` |
| 역할 | 모든 도구의 기본 타입 정의 및 팩토리 |
| 주요 의존 | [[types/]], [[hooks/useCanUseTool]] |

---

### 지원 모듈 그룹

#### [[Task.ts]] — 태스크 타입 & ID
| 항목 | 값 |
|------|-----|
| 핵심 Export | `TaskType`, `TaskStatus`, `TaskHandle`, `generateTaskId()` |
| 역할 | 태스크 라이프사이클 타입 (local_bash, local_agent, remote_agent 등) |

#### [[tasks.ts]] — 태스크 팩토리
| 항목 | 값 |
|------|-----|
| 핵심 Export | `getAllTasks()`, `getTaskByType()` |
| 역할 | 태스크 타입 → 구현 매핑 디스패치 테이블 |

#### [[cost-tracker.ts]] — 비용 추적
| 항목 | 값 |
|------|-----|
| 핵심 Export | `addToTotalSessionCost()`, `formatTotalCost()`, `saveCurrentSessionCosts()` |
| 역할 | 세션 비용 추적 및 영속화 |

#### [[costHook.ts]] — 비용 React 훅
| 항목 | 값 |
|------|-----|
| 핵심 Export | `useCostSummary()` |
| 역할 | 프로세스 종료 시 비용 표시 |

#### [[history.ts]] — 프롬프트 히스토리
| 항목 | 값 |
|------|-----|
| 핵심 Export | `addToHistory()`, `getHistory()`, `expandPastedTextRefs()` |
| 역할 | 입력 히스토리 JSONL 관리, 붙여넣기 참조 |

#### [[ink.ts]] — Ink 렌더링 래퍼
| 항목 | 값 |
|------|-----|
| 핵심 Export | `render()`, `createRoot()`, Box, Text, Button 등 re-export |
| 역할 | 테마 Provider 통합 Ink 래퍼 |

#### [[projectOnboardingState.ts]] — 온보딩 상태
| 항목 | 값 |
|------|-----|
| 핵심 Export | `shouldShowProjectOnboarding()`, `isProjectOnboardingComplete()` |
| 역할 | 프로젝트 온보딩 체크리스트 관리 |

## 핵심 의존 관계 흐름

```
main.tsx
  ├→ setup.ts (초기화)
  ├→ commands.ts (명령어 로드)
  ├→ context.ts (컨텍스트)
  ├→ QueryEngine.ts
  │     └→ query.ts (실행)
  │          ├→ tools.ts → Tool.ts (도구)
  │          └→ services/ (API, MCP, Compact)
  ├→ cost-tracker.ts → costHook.ts (비용)
  ├→ history.ts (히스토리)
  └→ ink.ts (렌더링)
```
