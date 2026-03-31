<div align="center">

# Claude Code CLI — Source Code Analysis

**Anthropic의 공식 AI 코딩 어시스턴트 [Claude Code](https://docs.anthropic.com/en/docs/claude-code) 소스코드 심층 분석**

[![Markdown](https://img.shields.io/badge/Markdown-39_Documents-blue?style=flat-square&logo=markdown)](./Index.md)
[![TypeScript](https://img.shields.io/badge/Analyzed-TypeScript-3178C6?style=flat-square&logo=typescript&logoColor=white)](./Directory_Structure.md)
[![Obsidian](https://img.shields.io/badge/Obsidian-Compatible-7C3AED?style=flat-square&logo=obsidian&logoColor=white)](#usage-with-obsidian)
[![Files Analyzed](https://img.shields.io/badge/Files_Analyzed-1%2C902-orange?style=flat-square)](./Stats_Report.md)
[![Tools](https://img.shields.io/badge/Tools-40+-green?style=flat-square)](./Tools_Overview.md)
[![Services](https://img.shields.io/badge/Services-19-red?style=flat-square)](./Services_Overview.md)
[![License](https://img.shields.io/badge/License-Educational-lightgrey?style=flat-square)](#license)

[Index (MOC)](./Index.md) · [Directory Structure](./Directory_Structure.md) · [Stats Report](./Stats_Report.md)

</div>

---

## Table of Contents

- [Overview](#overview)
- [Project Scale](#project-scale)
- [Architecture](#architecture)
  - [Layered Architecture](#layered-architecture)
  - [Initialization Flow](#initialization-flow)
  - [Query Execution Flow](#query-execution-flow)
  - [Tool Execution Flow](#tool-execution-flow)
  - [MCP Connection Flow](#mcp-connection-flow)
- [Design Principles](#design-principles)
- [Design Patterns](#design-patterns)
- [Tech Stack](#tech-stack)
- [Module Deep Dive](#module-deep-dive)
  - [Entrypoints](#1-entrypoints)
  - [Query Engine](#2-query-engine)
  - [Tool System](#3-tool-system-40-tools)
  - [Service Layer](#4-service-layer-19-services)
  - [UI Components](#5-ui-components)
  - [Ink Framework](#6-ink-framework)
  - [Hooks](#7-hooks-system)
  - [Commands](#8-commands-system)
  - [Utilities](#9-utilities)
  - [Security Architecture](#10-security-architecture)
  - [Multi-Agent System](#11-multi-agent-system)
  - [Memory System](#12-memory-system)
  - [Additional Modules](#13-additional-modules)
- [Module Distribution](#module-distribution)
- [Document Map](#document-map)
- [Usage with Obsidian](#usage-with-obsidian)
- [License](#license)

---

## Overview

이 저장소는 Anthropic의 공식 CLI 도구인 **[Claude Code](https://docs.anthropic.com/en/docs/claude-code)** 의 소스코드(`src/` 디렉터리)를 정적 분석한 결과물입니다. 총 **1,902개 파일**, **~160개 디렉터리**를 분석하여 **39개의 상호 연결된 Obsidian 마크다운 문서**로 구성했습니다.

모든 문서는 `[[위키링크]]`로 상호 연결되어 있어, [Obsidian](https://obsidian.md/)의 **Graph View**에서 모듈 간 관계를 시각적으로 탐색할 수 있습니다.

---

## Project Scale

| 지표 | 값 | 상세 |
|:-----|:---|:-----|
| **총 파일 수** | 1,902 | TypeScript (.ts, .tsx) |
| **총 디렉터리 수** | ~160 | 34개 상위 모듈 |
| **UI 프레임워크** | [React](https://react.dev/) + [Ink](https://github.com/vadimdemedes/ink) | 터미널 선언적 UI |
| **레이아웃 엔진** | [Yoga](https://www.yogalayout.dev/) | Flexbox 기반 터미널 레이아웃 |
| **빌드/런타임** | [Bun](https://bun.sh/) / [Node.js](https://nodejs.org/) | 고속 TypeScript 실행 |
| **스키마 검증** | [Zod](https://zod.dev/) v4 | 모든 도구 입출력 검증 |
| **도구 (Tools)** | 40+ | [상세 보기](./Tools_Overview.md) |
| **명령어 (Commands)** | 100+ | [상세 보기](./Commands_Overview.md) |
| **서비스 (Services)** | 19 | [상세 보기](./Services_Overview.md) |
| **React 훅 (Hooks)** | 104 files | [상세 보기](./Hooks_Overview.md) |

---

## Architecture

### Layered Architecture

Claude Code는 **5-Layer Architecture**를 채택하여 각 계층이 단방향 의존성을 유지합니다.

```
┌─────────────────────────────────────────────────────────┐
│                    USER INPUT LAYER                      │
│               Terminal / IDE / Web / Voice               │
└────────────────────────┬────────────────────────────────┘
                         │
┌────────────────────────▼────────────────────────────────┐
│                   ENTRYPOINT LAYER                       │
│          cli.tsx → init.ts → main.tsx → setup.ts         │
│                                                          │
│  - Fast-path 처리 (--version, --dump-system-prompt)      │
│  - OAuth / TLS / 정책 초기화                              │
│  - OpenTelemetry 지연 로딩                                │
└────────────────────────┬────────────────────────────────┘
                         │
┌────────────────────────▼────────────────────────────────┐
│                    ENGINE LAYER                           │
│           QueryEngine.ts ←→ query.ts                     │
│                                                          │
│  - 쿼리 라이프사이클 관리                                  │
│  - 시스템/사용자 컨텍스트 조립 (context.ts)                │
│  - 자동 컨텍스트 압축 (Compact Service)                   │
│  - REPL 대화형 루프                                       │
└────────────┬──────────────┬─────────────────────────────┘
             │              │
┌────────────▼──────┐ ┌─────▼───────────────────────┐
│   TOOL LAYER      │ │   COMMAND LAYER             │
│   40 Tools        │ │   100+ Slash Commands       │
│                   │ │                              │
│ File, Bash, Web,  │ │ /help, /model, /config,     │
│ Agent, MCP, LSP,  │ │ /compact, /mcp, /vim, ...   │
│ Task, Cron, ...   │ │                              │
└────────────┬──────┘ └──────────────────────────────┘
             │
┌────────────▼────────────────────────────────────────────┐
│                   SERVICE LAYER                          │
│                                                          │
│  ┌─────────┐ ┌─────────┐ ┌──────────┐ ┌──────────┐     │
│  │ API     │ │  MCP    │ │ Compact  │ │  Tools   │     │
│  │ Client  │ │ Server  │ │ Service  │ │ Executor │     │
│  │ (4 SDK) │ │ Manager │ │          │ │          │     │
│  └─────────┘ └─────────┘ └──────────┘ └──────────┘     │
│  ┌─────────┐ ┌─────────┐ ┌──────────┐ ┌──────────┐     │
│  │  LSP    │ │  OAuth  │ │Analytics │ │ Plugins  │     │
│  │ Server  │ │  2.0    │ │ DD + 1P  │ │ Manager  │     │
│  └─────────┘ └─────────┘ └──────────┘ └──────────┘     │
│  ┌─────────┐ ┌──────────────┐ ┌────────────────────┐    │
│  │ Session │ │   Extract    │ │  Team Memory Sync  │    │
│  │ Memory  │ │  Memories    │ │                    │    │
│  └─────────┘ └──────────────┘ └────────────────────┘    │
└────────────────────────┬────────────────────────────────┘
                         │
┌────────────────────────▼────────────────────────────────┐
│                 INFRASTRUCTURE LAYER                     │
│                                                          │
│  Permissions(26) · Settings(19) · Model(18) · Bash(15)  │
│  Telemetry(14) · Shell(12) · Git(5) · Sandbox · Crypto  │
└─────────────────────────────────────────────────────────┘
```

> 상세 아키텍처: [Index.md (MOC)](./Index.md) | 디렉터리 구조: [Directory_Structure.md](./Directory_Structure.md)

---

### Initialization Flow

앱 시작부터 REPL 진입까지의 **4-Phase Bootstrap** 시퀀스:

```
Phase 1: CLI Bootstrap (cli.tsx)
  │  Fast-path: --version, MCP 서버, 데몬 워커
  │  동적 임포트로 지연 로딩
  ▼
Phase 2: Environment Init (init.ts)
  │  Config 검증 → CA 인증서 → OAuth → 정책 → mTLS → API 프리커넥션
  │  10단계 초기화 파이프라인
  ▼
Phase 3: Service Bootstrap (main.tsx)
  │  OpenTelemetry 지연 로딩 → 메트릭 수집 → 서비스 초기화
  ▼
Phase 4: Session Setup (setup.ts)
  │  Node 버전 검증 → 워크트리 → 세션 메모리 → 권한 검증
  ▼
  REPL (대화형 루프 시작)
```

> 상세: [Entrypoints_Overview.md](./Entrypoints_Overview.md)

---

### Query Execution Flow

사용자 입력이 처리되어 응답으로 돌아오기까지의 핵심 실행 루프:

```
User Input
  │
  ▼
QueryEngine.ts ─── 라이프사이클 관리
  │
  ├──→ context.ts ─── 시스템/사용자 컨텍스트 조립
  │
  ├──→ query.ts ─── 모델 호출 + 도구 실행 루프
  │     │
  │     ├──→ API Service ─── Claude API 스트리밍 호출
  │     │
  │     ├──→ StreamingToolExecutor ─── 도구 병렬/직렬 실행
  │     │     │
  │     │     ├──→ Permissions System ─── 허용/거부/문의
  │     │     └──→ Tool.call() ─── 실제 도구 실행
  │     │
  │     └──→ Compact Service ─── 컨텍스트 압축 판단
  │
  └──→ Response Rendering
```

> 상세: [Query_Engine.md](./Query_Engine.md)

---

### Tool Execution Flow

모든 도구 실행은 **권한 시스템**을 거칩니다:

```
모델 응답 → tool_use 블록 추출
  │
  ▼
tools.ts (findToolByName) ─── 도구 풀에서 조회
  │
  ▼
Tool.checkPermissions() ─── 권한 시스템
  ├── ALLOW  → Tool.call() 실행
  ├── DENY   → 거부 메시지 반환
  └── ASK    → 사용자 권한 요청 다이얼로그
  │
  ▼
StreamingToolExecutor
  ├── 동시성 안전(isConcurrencySafe) → 병렬 실행
  └── 비안전 → 직렬 실행
  │
  ▼
결과를 메시지에 추가 → 다음 턴
```

> 상세: [Tools_Overview.md](./Tools_Overview.md)

---

### MCP Connection Flow

[Model Context Protocol](https://modelcontextprotocol.io/) 서버 연결부터 도구 사용까지:

```
settings.json (MCP 서버 정의)
  │
  ▼
MCP Service ─── 4개 전송 프로토콜 지원
  │             (stdio / SSE / WebSocket / HTTP)
  ├──→ 서버 연결 (MCPConnectionManager)
  ├──→ OAuth 인증 (필요 시)
  ├──→ 도구/리소스 발견
  │
  ▼
tools.ts (assembleToolPool)
  └── 내장 도구 + MCP 도구 병합 (중복 제거)
```

> 상세: [services/MCP_Service.md](./services/MCP_Service.md)

---

## Design Principles

Claude Code 소스코드에서 관찰되는 **6가지 핵심 설계 원칙**:

| 원칙 | 설명 | 구현 위치 |
|:-----|:-----|:---------|
| **Layered Architecture** | 진입점 → 엔진 → 도구 → 서비스 → 유틸리티의 단방향 의존성 | [Index.md](./Index.md) |
| **Permission-First** | 모든 도구 실행 전 `checkPermissions()` 통과 필수 | [Tools_Overview.md](./Tools_Overview.md) |
| **Extensibility** | MCP, 플러그인, 스킬을 통한 기능 확장 | [Services_Overview.md](./Services_Overview.md) |
| **Declarative Terminal UI** | React + Ink를 통한 선언적 터미널 렌더링 | [Ink_Framework.md](./Ink_Framework.md) |
| **Multi-Agent** | AgentTool, Coordinator, Swarm 패턴 지원 | [tools/AgentTool.md](./tools/AgentTool.md) |
| **Multi-Provider API** | Anthropic Direct, Bedrock, Vertex AI, Azure Foundry | [services/API_Service.md](./services/API_Service.md) |

---

## Design Patterns

소스코드 전반에서 사용되는 설계 패턴 분석:

| 패턴 | 빈도 | 주요 위치 | 설명 |
|:-----|:-----|:---------|:-----|
| **Factory** | 높음 | `Tool.ts` (`buildTool()`), LSP | 도구/서버 인스턴스 생성 |
| **Async Generator** | 높음 | `StreamingToolExecutor` | 스트리밍 결과 반환 (`async function*`) |
| **React Hooks** | 높음 | [Hooks_Overview.md](./Hooks_Overview.md) | 104개 커스텀 훅 |
| **Singleton Store** | 중간 | `bootstrap/state.ts` | 209개 getter/setter 글로벌 상태 |
| **Observer/Event** | 중간 | [Ink_Framework.md](./Ink_Framework.md) | 터미널 이벤트, 파일 감시 |
| **Strategy** | 중간 | 권한 핸들러, 셸 프로바이더 | 런타임 전략 교체 |
| **Memoization** | 높음 | `context.ts`, `commands.ts` | 중복 계산 방지 |
| **Forked Agent** | 낮음 | SessionMemory, ExtractMemories | 백그라운드 서브에이전트 |

> 상세: [Stats_Report.md](./Stats_Report.md)

---

## Tech Stack

### Core Dependencies

| 패키지 | 용도 | 분석 문서 |
|:-------|:-----|:---------|
| [`@anthropic-ai/sdk`](https://github.com/anthropics/anthropic-sdk-typescript) | Claude API Direct | [API_Service.md](./services/API_Service.md) |
| [`@anthropic-ai/bedrock-sdk`](https://github.com/anthropics/anthropic-sdk-typescript) | AWS Bedrock | [API_Service.md](./services/API_Service.md) |
| [`@anthropic-ai/vertex-sdk`](https://github.com/anthropics/anthropic-sdk-typescript) | Google Vertex AI | [API_Service.md](./services/API_Service.md) |
| [`@anthropic-ai/foundry-sdk`](https://github.com/anthropics/anthropic-sdk-typescript) | Azure Foundry | [API_Service.md](./services/API_Service.md) |
| [`@modelcontextprotocol/sdk`](https://github.com/modelcontextprotocol/typescript-sdk) | MCP 프로토콜 | [MCP_Service.md](./services/MCP_Service.md) |
| [`react`](https://react.dev/) | UI 프레임워크 | [Components_Overview.md](./Components_Overview.md) |
| [`ink`](https://github.com/vadimdemedes/ink) | 터미널 React 렌더러 | [Ink_Framework.md](./Ink_Framework.md) |
| [`zod`](https://zod.dev/) (v4) | 스키마 검증 | [Tools_Overview.md](./Tools_Overview.md) |
| [`yoga-layout`](https://www.yogalayout.dev/) | Flexbox 레이아웃 | [Ink_Framework.md](./Ink_Framework.md) |
| [`tree-sitter`](https://tree-sitter.github.io/tree-sitter/) | AST 파싱 | [Utils_Overview.md](./Utils_Overview.md) |
| [`vscode-languageserver-protocol`](https://github.com/microsoft/vscode-languageserver-node) | LSP | [Services_Overview.md](./Services_Overview.md) |

---

## Module Deep Dive

### 1. Entrypoints

> 앱 시작부터 REPL 진입까지의 부트스트랩 흐름

| 파일 | 역할 | 핵심 기능 |
|:-----|:-----|:---------|
| `cli.tsx` | CLI 진입점 | Fast-path 분기, 동적 임포트 |
| `init.ts` | 환경 초기화 | OAuth, TLS, 정책, API 프리커넥션 |
| `main.tsx` | 서비스 부트스트랩 | OpenTelemetry, 메트릭 |
| `setup.ts` | 세션 셋업 | 워크트리, 세션 메모리, 권한 검증 |

> 상세: [Entrypoints_Overview.md](./Entrypoints_Overview.md)

---

### 2. Query Engine

> 핵심 실행 루프 — 사용자 입력부터 응답 렌더링까지

- **QueryEngine.ts**: 쿼리 라이프사이클 관리 (시작 → 실행 → 압축 → 종료)
- **query.ts**: Claude API 호출 + 도구 실행 루프
- **context.ts**: 시스템/사용자 컨텍스트 조립 (메모이제이션)

> 상세: [Query_Engine.md](./Query_Engine.md)

---

### 3. Tool System (40 Tools)

> 파일 시스템, 셸, 에이전트, 웹, MCP 등 40개 도구

| 카테고리 | 도구 수 | 주요 도구 | 문서 |
|:---------|:--------|:---------|:-----|
| **파일 시스템** | 5 | Read, Edit, Write, Glob, Grep | [FileReadTool.md](./tools/FileReadTool.md), [FileEditTool.md](./tools/FileEditTool.md) |
| **셸 실행** | 2 | Bash, PowerShell | [BashTool.md](./tools/BashTool.md) |
| **에이전트** | 3 | Agent, SendMessage, Skill | [AgentTool.md](./tools/AgentTool.md) |
| **웹** | 2 | WebFetch, WebSearch | [Tools_Overview.md](./Tools_Overview.md) |
| **태스크 관리** | 6 | Create, Update, List, Get, Stop, Output | [TaskTools.md](./tools/TaskTools.md) |
| **MCP** | 4 | MCPTool, ListResources, ReadResource, Auth | [MCPTool.md](./tools/MCPTool.md) |
| **코드 인텔리전스** | 1 | LSP | [Tools_Overview.md](./Tools_Overview.md) |
| **워크트리/플랜** | 4 | EnterWorktree, ExitWorktree, EnterPlan, ExitPlan | [Tools_Overview.md](./Tools_Overview.md) |
| **스케줄링** | 3 | ScheduleCron, CronDelete, CronList | [Tools_Overview.md](./Tools_Overview.md) |
| **기타** | 10 | Config, Notebook, RemoteTrigger, ... | [Tools_Overview.md](./Tools_Overview.md) |

**핵심 인터페이스:**

```typescript
interface Tool<Input, Output, Progress> {
  name: string;
  call(input: Input, context: ToolUseContext): Promise<Output>;
  checkPermissions(input: Input, context: ToolPermissionContext): PermissionResult;
  isReadOnly(): boolean;
  isConcurrencySafe(): boolean;
}
```

> 상세: [Tools_Overview.md](./Tools_Overview.md)

---

### 4. Service Layer (19 Services)

> API 통신, MCP, 컨텍스트 관리, 인증, 분석 등 핵심 비즈니스 로직

| 서비스 | 파일 수 | 복잡도 | 문서 |
|:-------|:--------|:------|:-----|
| **API Client** | 16 | 높음 | [API_Service.md](./services/API_Service.md) |
| **MCP Server Manager** | 23 | 높음 | [MCP_Service.md](./services/MCP_Service.md) |
| **Compact (Context Compression)** | 11 | 중간 | [Compact_Service.md](./services/Compact_Service.md) |
| **Tool Executor** | 4 | 중간 | [Services_Overview.md](./Services_Overview.md) |
| **LSP Server** | 7 | 중간 | [Services_Overview.md](./Services_Overview.md) |
| **OAuth 2.0** | 5 | 중간 | [Services_Overview.md](./Services_Overview.md) |
| **Analytics (DD + 1P)** | 9 | 중간 | [Services_Overview.md](./Services_Overview.md) |
| **Session Memory** | 3 | 낮음 | [Services_Overview.md](./Services_Overview.md) |
| **Extract Memories** | 2 | 낮음 | [Services_Overview.md](./Services_Overview.md) |
| **Plugins** | 3 | 중간 | [Services_Overview.md](./Services_Overview.md) |

**API 멀티프로바이더:**
```
Claude API ─┬── Direct (api.anthropic.com)
            ├── AWS Bedrock (@anthropic-ai/bedrock-sdk)
            ├── Google Vertex AI (@anthropic-ai/vertex-sdk)
            └── Azure Foundry (@anthropic-ai/foundry-sdk)
```

> 상세: [Services_Overview.md](./Services_Overview.md)

---

### 5. UI Components

> React 기반 터미널 UI — 389개 파일

| 카테고리 | 파일 수 | 설명 |
|:---------|:--------|:-----|
| 메시지 렌더링 | 36 | 스트리밍 응답, 마크다운, 코드 하이라이트 |
| 권한 다이얼로그 | 32 | 도구 실행 권한 요청/승인/거부 |
| 디자인 시스템 | 18 | 테마, 컬러, 타이포그래피 |
| 에이전트 UI | 16 | 서브에이전트 상태 표시 |
| MCP UI | 15 | MCP 서버 연결 상태 |
| 프롬프트 입력 | 8 | 멀티라인 입력, 자동완성 |

> 상세: [Components_Overview.md](./Components_Overview.md)

---

### 6. Ink Framework

> 커스텀 터미널 렌더링 프레임워크 — 96개 파일

React의 선언적 UI 모델을 터미널에 적용한 커스텀 Ink 프레임워크:

- **렌더링 파이프라인**: React Reconciler → Yoga Layout → ANSI 출력
- **레이아웃 시스템**: Yoga 기반 Flexbox (width, height, padding, margin, flex)
- **커스텀 컴포넌트**: Box, Text, ScrollBox, Spinner
- **이벤트 시스템**: 키보드 입력, 마우스, 리사이즈

> 상세: [Ink_Framework.md](./Ink_Framework.md)

---

### 7. Hooks System

> 104개 React 훅 — 상태 관리, 도구 권한, 태스크 관리

- 도구 권한 훅 (ToolPermission)
- 태스크 관리 훅 (TaskManagement)
- UI 상태 훅 (useInput, useKeyboard, useFocus)

> 상세: [Hooks_Overview.md](./Hooks_Overview.md)

---

### 8. Commands System

> 100+ CLI 슬래시 명령어

`/help`, `/model`, `/config`, `/compact`, `/mcp`, `/vim`, `/cost`, `/doctor` 등 100개 이상의 슬래시 명령어와 스킬 시스템.

> 상세: [Commands_Overview.md](./Commands_Overview.md)

---

### 9. Utilities

> 564개 파일 — 프로젝트의 29.7%를 차지하는 인프라 라이브러리

| 모듈 | 파일 수 | 설명 | 문서 |
|:-----|:--------|:-----|:-----|
| **Permissions** | 26 | 규칙 기반 + AI 분류 권한 시스템 | [Utils_Overview.md](./Utils_Overview.md) |
| **Settings** | 19 | 계층적 설정 관리 | [Utils_Overview.md](./Utils_Overview.md) |
| **Model** | 18 | 모델 선택/관리/가격 | [Utils_Overview.md](./Utils_Overview.md) |
| **Bash** | 15+ | AST 파싱, 인젝션 방지 | [Utils_Overview.md](./Utils_Overview.md) |
| **Telemetry** | 14 | OpenTelemetry 기반 | [Utils_Overview.md](./Utils_Overview.md) |
| **Shell** | 12 | 셸 추상화 계층 | [Utils_Overview.md](./Utils_Overview.md) |
| **Swarm** | 16+ | 분산 에이전트 | [Utils_Overview.md](./Utils_Overview.md) |
| **Git** | 5 | Git 상태 읽기 | [Utils_Overview.md](./Utils_Overview.md) |

> 상세: [Utils_Overview.md](./Utils_Overview.md)

---

### 10. Security Architecture

Claude Code의 다층 보안 구조:

| 보안 계층 | 파일 수 | 기법 |
|:----------|:--------|:-----|
| **권한 시스템** | 26 | 규칙 기반 + AI 분류 (허용/거부/문의) |
| **Bash 보안** | 15+ | tree-sitter AST 파싱, 명령 인젝션 방지 |
| **셸 검증** | 12 | 읽기전용 명령 레지스트리 |
| **경로 검증** | 5+ | 안전 경로 체크 (디렉터리 탈출 방지) |
| **샌드박싱** | 2 | `@anthropic-ai/sandbox-runtime` |

---

### 11. Multi-Agent System

Claude Code의 멀티에이전트 오케스트레이션:

- **[AgentTool](./tools/AgentTool.md)**: 서브에이전트 생성 (워크트리 격리 지원)
- **[Coordinator Mode](./coordinator/coordinatorMode.md)**: 멀티 워커 에이전트 오케스트레이션
- **Swarm**: 분산 에이전트 패턴 (`utils/swarm/`)
- **SendMessage**: 에이전트 간 메시지 전달

---

### 12. Memory System

파일 기반 영속 메모리 시스템:

- **[Memdir](./memdir/Memdir_Overview.md)**: AI 선택 기반 파일 메모리, 팀 메모리
- **Session Memory**: 대화 중 자동 메모 유지 (포크된 서브에이전트)
- **Extract Memories**: 쿼리 루프 종료 시 지속 메모리 추출

---

### 13. Additional Modules

| 모듈 | 설명 | 문서 |
|:-----|:-----|:-----|
| **Bootstrap / State** | 글로벌 세션 상태 싱글톤 (209 getter/setter) | [bootstrap/state.md](./bootstrap/state.md) |
| **Screens** | REPL, 세션 재개, 진단 화면 | [screens/Screens_Overview.md](./screens/Screens_Overview.md) |
| **Server** | WebSocket 직접 연결 세션 관리 | [server/Server_Overview.md](./server/Server_Overview.md) |
| **Upstream Proxy** | CCR용 CONNECT-over-WebSocket 릴레이 | [upstreamproxy/UpstreamProxy_Overview.md](./upstreamproxy/UpstreamProxy_Overview.md) |
| **Keybindings** | 커스텀 키보드 단축키 + 코드 시퀀스 | [keybindings/Keybindings_Overview.md](./keybindings/Keybindings_Overview.md) |
| **Migrations** | 설정/모델 마이그레이션 11개 | [migrations/Migrations_Overview.md](./migrations/Migrations_Overview.md) |
| **Native TS** | color-diff, file-index, yoga-layout TS 포트 | [native-ts/NativeTS_Overview.md](./native-ts/NativeTS_Overview.md) |
| **Buddy** | 컴패니언 캐릭터 절차적 생성 시스템 | [buddy/Buddy_Overview.md](./buddy/Buddy_Overview.md) |
| **Plugins** | 내장 플러그인 레지스트리 + 마켓플레이스 | [plugins/Plugins_Module.md](./plugins/Plugins_Module.md) |
| **Voice** | 음성 모드 피처 게이트 | [voice/voiceModeEnabled.md](./voice/voiceModeEnabled.md) |
| **Root Files** | src/ 루트 18개 핵심 파일 | [Root_Files_Overview.md](./Root_Files_Overview.md) |

---

## Module Distribution

```
utils       ████████████████████████████████████████  564 (29.7%)
components  ██████████████████████████████            389 (20.5%)
commands    ████████████████                          207 (10.9%)
tools       █████████████                             184  (9.7%)
services    █████████                                 130  (6.8%)
hooks       ███████                                   104  (5.5%)
ink         ██████                                     96  (5.0%)
bridge      ██                                         31  (1.6%)
constants   █                                          21  (1.1%)
skills      █                                          20  (1.1%)
others      ████                                      156  (8.2%)
```

> 상세: [Stats_Report.md](./Stats_Report.md)

---

## Document Map

### Core Documents

| 문서 | 유형 | 설명 |
|:-----|:-----|:-----|
| [Index.md](./Index.md) | MOC | 전체 프로젝트 지도 (Map of Content) |
| [Directory_Structure.md](./Directory_Structure.md) | Structure | 폴더 트리 + 모듈별 파일 수 |
| [Stats_Report.md](./Stats_Report.md) | Report | 의존성 통계, 모듈 분포, 품질 지표 |

### Layer Documents

| 문서 | 유형 | 설명 |
|:-----|:-----|:-----|
| [Entrypoints_Overview.md](./Entrypoints_Overview.md) | Module | 앱 시작 흐름 (cli → init → main → setup) |
| [Query_Engine.md](./Query_Engine.md) | Module | 핵심 실행 루프 |
| [Tools_Overview.md](./Tools_Overview.md) | Module | 40개 도구 전체 카탈로그 |
| [Services_Overview.md](./Services_Overview.md) | Module | 19개 서비스 레이어 |
| [Components_Overview.md](./Components_Overview.md) | Module | 389개 UI 컴포넌트 |
| [Ink_Framework.md](./Ink_Framework.md) | Module | 커스텀 터미널 렌더링 |
| [Hooks_Overview.md](./Hooks_Overview.md) | Module | 104개 React 훅 |
| [Commands_Overview.md](./Commands_Overview.md) | Module | 100+ 슬래시 명령어 |
| [Utils_Overview.md](./Utils_Overview.md) | Module | 564개 유틸리티 파일 |
| [Root_Files_Overview.md](./Root_Files_Overview.md) | Module | src/ 루트 18개 파일 |

### Deep Dive Documents

| 문서 | 분석 대상 |
|:-----|:---------|
| [tools/AgentTool.md](./tools/AgentTool.md) | 서브에이전트 생성 + 워크트리 격리 |
| [tools/BashTool.md](./tools/BashTool.md) | 셸 실행 + AST 보안 검증 |
| [tools/FileReadTool.md](./tools/FileReadTool.md) | 파일 읽기 (PDF, 이미지, 노트북) |
| [tools/FileEditTool.md](./tools/FileEditTool.md) | 문자열 치환 기반 파일 편집 |
| [tools/MCPTool.md](./tools/MCPTool.md) | MCP 서버 도구 실행 |
| [tools/TaskTools.md](./tools/TaskTools.md) | 태스크 CRUD 도구 |
| [services/API_Service.md](./services/API_Service.md) | 4-SDK 멀티프로바이더 API |
| [services/MCP_Service.md](./services/MCP_Service.md) | MCP 서버 연결/인증/발견 |
| [services/Compact_Service.md](./services/Compact_Service.md) | 컨텍스트 윈도우 압축 전략 |
| [bootstrap/state.md](./bootstrap/state.md) | 글로벌 상태 싱글톤 |
| [coordinator/coordinatorMode.md](./coordinator/coordinatorMode.md) | 멀티에이전트 오케스트레이션 |
| [memdir/Memdir_Overview.md](./memdir/Memdir_Overview.md) | 파일 기반 영속 메모리 |
| [screens/Screens_Overview.md](./screens/Screens_Overview.md) | REPL, 세션 재개, 진단 |
| [server/Server_Overview.md](./server/Server_Overview.md) | WebSocket 직접 연결 |
| [keybindings/Keybindings_Overview.md](./keybindings/Keybindings_Overview.md) | 키바인딩 + 코드 시퀀스 |
| [migrations/Migrations_Overview.md](./migrations/Migrations_Overview.md) | 설정 마이그레이션 11개 |
| [native-ts/NativeTS_Overview.md](./native-ts/NativeTS_Overview.md) | 네이티브 모듈 TS 포트 |
| [buddy/Buddy_Overview.md](./buddy/Buddy_Overview.md) | 컴패니언 캐릭터 시스템 |
| [upstreamproxy/UpstreamProxy_Overview.md](./upstreamproxy/UpstreamProxy_Overview.md) | CONNECT-over-WS 릴레이 |

---

## Usage with Obsidian

### Quick Start

```bash
# 1. 저장소 클론
git clone https://github.com/leaf-kit/claude-analysis.git

# 2. Obsidian에서 클론된 폴더를 Vault로 열기
#    Obsidian → Open folder as vault → claude-analysis/
```

### Features

- **Graph View**: 모든 문서가 `[[위키링크]]`로 연결되어 모듈 간 관계를 시각적으로 탐색
- **YAML Frontmatter**: `tags`, `type`, `status` 기반 필터링 및 [Dataview](https://github.com/blacksmithgu/obsidian-dataview) 쿼리 지원
- **Tag Navigation**: `#tools`, `#services`, `#architecture` 등 태그로 빠른 탐색
- **Search**: 전체 문서에 걸친 코드 패턴, 함수명, 모듈명 검색

### Recommended Plugins

| 플러그인 | 용도 |
|:---------|:-----|
| [Dataview](https://github.com/blacksmithgu/obsidian-dataview) | YAML frontmatter 기반 동적 테이블/리스트 |
| [Graph Analysis](https://github.com/SkepticMystic/graph-analysis) | 그래프 중심성, 클러스터 분석 |
| [Admonition](https://github.com/javalent/admonitions) | 정보/경고/팁 박스 렌더링 |

---

## License

이 분석 문서는 **교육 및 참고 목적**으로 작성되었습니다.

- 분석 대상인 Claude Code는 [Anthropic](https://www.anthropic.com/)의 제품입니다.
- 이 저장소의 마크다운 문서는 자유롭게 참고할 수 있습니다.
- 원본 소스코드의 저작권은 Anthropic에 있습니다.

---

<div align="center">

**[Top](#claude-code-cli--source-code-analysis)**

</div>
