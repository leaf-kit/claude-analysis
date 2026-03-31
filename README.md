<div align="center">

# Claude Code CLI — Source Code Analysis

**Anthropic의 공식 AI 코딩 어시스턴트 [Claude Code](https://docs.anthropic.com/en/docs/claude-code) 소스코드 심층 분석**

[![Markdown](https://img.shields.io/badge/Markdown-39_Documents-blue?style=flat-square&logo=markdown)](./Index.md)
[![TypeScript](https://img.shields.io/badge/Analyzed-TypeScript-3178C6?style=flat-square&logo=typescript&logoColor=white)](./Directory_Structure.md)
[![Obsidian](https://img.shields.io/badge/Obsidian-Compatible-7C3AED?style=flat-square&logo=obsidian&logoColor=white)](#usage-with-obsidian)
[![Files Analyzed](https://img.shields.io/badge/Files_Analyzed-1%2C902-orange?style=flat-square)](./Stats_Report.md)
[![Tools](https://img.shields.io/badge/Tools-40+-green?style=flat-square)](./Tools_Overview.md)
[![Services](https://img.shields.io/badge/Services-19-red?style=flat-square)](./Services_Overview.md)
[![Source Code](https://img.shields.io/badge/Source_Code-1%2C902_Files-blueviolet?style=flat-square&logo=github)](#source-code-browse)
[![License](https://img.shields.io/badge/License-Educational-lightgrey?style=flat-square)](#license)

[Index (MOC)](./Index.md) · [Directory Structure](./Directory_Structure.md) · [Stats Report](./Stats_Report.md) · [Source Code](./src/)

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
- [Source Code Browse](#source-code-browse)
- [Usage with Obsidian](#usage-with-obsidian)
- [License](#license)

---

## Overview

이 저장소는 Anthropic의 공식 CLI 도구인 **[Claude Code](https://docs.anthropic.com/en/docs/claude-code)** 의 소스코드(`src/` 디렉터리)를 정적 분석한 결과물입니다. 총 **1,902개 파일**, **~160개 디렉터리**를 분석하여 **39개의 상호 연결된 Obsidian 마크다운 문서**로 구성했습니다.

> **실제 소스코드 포함**: 이 저장소의 [`./src/`](./src/) 디렉터리에는 Claude Code의 **실제 TypeScript/TSX 소스코드 1,902개 파일**이 포함되어 있습니다. 분석 문서를 읽으면서 원본 코드를 직접 열어볼 수 있으며, GitHub에서도 모든 파일을 바로 탐색할 수 있습니다. 전체 파일 목록은 [Source Code Browse](#source-code-browse) 섹션을 참고하세요.

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

---

## Source Code Browse

> **이 저장소에는 Claude Code의 실제 소스코드가 포함되어 있습니다.**
> [`./src/`](./src/) 디렉터리에서 모든 TypeScript/TSX 파일을 직접 열어볼 수 있습니다.
> 분석 문서와 원본 코드를 나란히 비교하며 아키텍처를 이해해 보세요.

### Module Summary (36 modules, 1,902 files)

| Module | Files | Description | Browse |
|:-------|------:|:------------|:-------|
| **src/ Root** | 18 | 핵심 엔트리 파일 | [src/ Root](./src/) |
| **assistant/** | 1 | 세션 히스토리 | [assistant/](./src/assistant/) |
| **bootstrap/** | 1 | 글로벌 상태 싱글톤 | [bootstrap/](./src/bootstrap/) |
| **bridge/** | 31 | IDE 브릿지 통신 | [bridge/](./src/bridge/) |
| **buddy/** | 6 | 컴패니언 캐릭터 | [buddy/](./src/buddy/) |
| **cli/** | 19 | CLI 전송/핸들러 | [cli/](./src/cli/) |
| **commands/** | 207 | 100+ 슬래시 명령어 | [commands/](./src/commands/) |
| **components/** | 389 | React 터미널 UI | [components/](./src/components/) |
| **constants/** | 21 | 상수 정의 | [constants/](./src/constants/) |
| **context/** | 9 | 시스템/사용자 컨텍스트 | [context/](./src/context/) |
| **coordinator/** | 1 | 멀티에이전트 오케스트레이션 | [coordinator/](./src/coordinator/) |
| **entrypoints/** | 8 | 앱 진입점 (CLI, SDK, MCP) | [entrypoints/](./src/entrypoints/) |
| **hooks/** | 104 | React 커스텀 훅 | [hooks/](./src/hooks/) |
| **ink/** | 96 | 터미널 렌더링 프레임워크 | [ink/](./src/ink/) |
| **keybindings/** | 14 | 키바인딩 시스템 | [keybindings/](./src/keybindings/) |
| **memdir/** | 8 | 파일 기반 메모리 | [memdir/](./src/memdir/) |
| **migrations/** | 11 | 설정 마이그레이션 | [migrations/](./src/migrations/) |
| **moreright/** | 1 | REPL 패널 스텁 | [moreright/](./src/moreright/) |
| **native-ts/** | 4 | 네이티브 모듈 TS 포트 | [native-ts/](./src/native-ts/) |
| **outputStyles/** | 1 | 출력 스타일 로더 | [outputStyles/](./src/outputStyles/) |
| **plugins/** | 2 | 플러그인 레지스트리 | [plugins/](./src/plugins/) |
| **query/** | 4 | 쿼리 설정 | [query/](./src/query/) |
| **remote/** | 4 | 원격 세션 | [remote/](./src/remote/) |
| **schemas/** | 1 | Zod 스키마 정의 | [schemas/](./src/schemas/) |
| **screens/** | 3 | REPL/진단 화면 | [screens/](./src/screens/) |
| **server/** | 3 | WebSocket 서버 | [server/](./src/server/) |
| **services/** | 130 | 핵심 서비스 레이어 | [services/](./src/services/) |
| **skills/** | 20 | 스킬 시스템 | [skills/](./src/skills/) |
| **state/** | 6 | 앱 상태 관리 | [state/](./src/state/) |
| **tasks/** | 12 | 백그라운드 태스크 | [tasks/](./src/tasks/) |
| **tools/** | 184 | 40개 도구 구현 | [tools/](./src/tools/) |
| **types/** | 11 | TypeScript 타입 정의 | [types/](./src/types/) |
| **upstreamproxy/** | 2 | CONNECT-over-WS 릴레이 | [upstreamproxy/](./src/upstreamproxy/) |
| **utils/** | 564 | 인프라 유틸리티 | [utils/](./src/utils/) |
| **vim/** | 5 | Vim 모드 | [vim/](./src/vim/) |
| **voice/** | 1 | 음성 모드 | [voice/](./src/voice/) |
| **Total** | **1,902** | | [`./src/`](./src/) |

---

### Complete File Listing

> 아래 테이블의 모든 파일명은 클릭하면 해당 소스코드로 직접 이동합니다.

<details>
<summary><b>전체 1,902개 파일 목록 펼치기 (click to expand)</b></summary>

### src/ Root Files

| # | File | Type |
|:--|:-----|:-----|
| 1 | [QueryEngine.ts](./src/QueryEngine.ts) | `.ts` |
| 2 | [Task.ts](./src/Task.ts) | `.ts` |
| 3 | [Tool.ts](./src/Tool.ts) | `.ts` |
| 4 | [commands.ts](./src/commands.ts) | `.ts` |
| 5 | [context.ts](./src/context.ts) | `.ts` |
| 6 | [cost-tracker.ts](./src/cost-tracker.ts) | `.ts` |
| 7 | [costHook.ts](./src/costHook.ts) | `.ts` |
| 8 | [dialogLaunchers.tsx](./src/dialogLaunchers.tsx) | `.tsx` |
| 9 | [history.ts](./src/history.ts) | `.ts` |
| 10 | [ink.ts](./src/ink.ts) | `.ts` |
| 11 | [interactiveHelpers.tsx](./src/interactiveHelpers.tsx) | `.tsx` |
| 12 | [main.tsx](./src/main.tsx) | `.tsx` |
| 13 | [projectOnboardingState.ts](./src/projectOnboardingState.ts) | `.ts` |
| 14 | [query.ts](./src/query.ts) | `.ts` |
| 15 | [replLauncher.tsx](./src/replLauncher.tsx) | `.tsx` |
| 16 | [setup.ts](./src/setup.ts) | `.ts` |
| 17 | [tasks.ts](./src/tasks.ts) | `.ts` |
| 18 | [tools.ts](./src/tools.ts) | `.ts` |

### assistant/

| # | File | Type |
|:--|:-----|:-----|
| 1 | [sessionHistory.ts](./src/assistant/sessionHistory.ts) | `.ts` |

### bootstrap/

| # | File | Type |
|:--|:-----|:-----|
| 1 | [state.ts](./src/bootstrap/state.ts) | `.ts` |

### bridge/

| # | File | Type |
|:--|:-----|:-----|
| 1 | [bridgeApi.ts](./src/bridge/bridgeApi.ts) | `.ts` |
| 2 | [bridgeConfig.ts](./src/bridge/bridgeConfig.ts) | `.ts` |
| 3 | [bridgeDebug.ts](./src/bridge/bridgeDebug.ts) | `.ts` |
| 4 | [bridgeEnabled.ts](./src/bridge/bridgeEnabled.ts) | `.ts` |
| 5 | [bridgeMain.ts](./src/bridge/bridgeMain.ts) | `.ts` |
| 6 | [bridgeMessaging.ts](./src/bridge/bridgeMessaging.ts) | `.ts` |
| 7 | [bridgePermissionCallbacks.ts](./src/bridge/bridgePermissionCallbacks.ts) | `.ts` |
| 8 | [bridgePointer.ts](./src/bridge/bridgePointer.ts) | `.ts` |
| 9 | [bridgeStatusUtil.ts](./src/bridge/bridgeStatusUtil.ts) | `.ts` |
| 10 | [bridgeUI.ts](./src/bridge/bridgeUI.ts) | `.ts` |
| 11 | [capacityWake.ts](./src/bridge/capacityWake.ts) | `.ts` |
| 12 | [codeSessionApi.ts](./src/bridge/codeSessionApi.ts) | `.ts` |
| 13 | [createSession.ts](./src/bridge/createSession.ts) | `.ts` |
| 14 | [debugUtils.ts](./src/bridge/debugUtils.ts) | `.ts` |
| 15 | [envLessBridgeConfig.ts](./src/bridge/envLessBridgeConfig.ts) | `.ts` |
| 16 | [flushGate.ts](./src/bridge/flushGate.ts) | `.ts` |
| 17 | [inboundAttachments.ts](./src/bridge/inboundAttachments.ts) | `.ts` |
| 18 | [inboundMessages.ts](./src/bridge/inboundMessages.ts) | `.ts` |
| 19 | [initReplBridge.ts](./src/bridge/initReplBridge.ts) | `.ts` |
| 20 | [jwtUtils.ts](./src/bridge/jwtUtils.ts) | `.ts` |
| 21 | [pollConfig.ts](./src/bridge/pollConfig.ts) | `.ts` |
| 22 | [pollConfigDefaults.ts](./src/bridge/pollConfigDefaults.ts) | `.ts` |
| 23 | [remoteBridgeCore.ts](./src/bridge/remoteBridgeCore.ts) | `.ts` |
| 24 | [replBridge.ts](./src/bridge/replBridge.ts) | `.ts` |
| 25 | [replBridgeHandle.ts](./src/bridge/replBridgeHandle.ts) | `.ts` |
| 26 | [replBridgeTransport.ts](./src/bridge/replBridgeTransport.ts) | `.ts` |
| 27 | [sessionIdCompat.ts](./src/bridge/sessionIdCompat.ts) | `.ts` |
| 28 | [sessionRunner.ts](./src/bridge/sessionRunner.ts) | `.ts` |
| 29 | [trustedDevice.ts](./src/bridge/trustedDevice.ts) | `.ts` |
| 30 | [types.ts](./src/bridge/types.ts) | `.ts` |
| 31 | [workSecret.ts](./src/bridge/workSecret.ts) | `.ts` |

### buddy/

| # | File | Type |
|:--|:-----|:-----|
| 1 | [CompanionSprite.tsx](./src/buddy/CompanionSprite.tsx) | `.tsx` |
| 2 | [companion.ts](./src/buddy/companion.ts) | `.ts` |
| 3 | [prompt.ts](./src/buddy/prompt.ts) | `.ts` |
| 4 | [sprites.ts](./src/buddy/sprites.ts) | `.ts` |
| 5 | [types.ts](./src/buddy/types.ts) | `.ts` |
| 6 | [useBuddyNotification.tsx](./src/buddy/useBuddyNotification.tsx) | `.tsx` |

### cli/

| # | File | Type |
|:--|:-----|:-----|
| 1 | [exit.ts](./src/cli/exit.ts) | `.ts` |
| 2 | [handlers/agents.ts](./src/cli/handlers/agents.ts) | `.ts` |
| 3 | [handlers/auth.ts](./src/cli/handlers/auth.ts) | `.ts` |
| 4 | [handlers/autoMode.ts](./src/cli/handlers/autoMode.ts) | `.ts` |
| 5 | [handlers/mcp.tsx](./src/cli/handlers/mcp.tsx) | `.tsx` |
| 6 | [handlers/plugins.ts](./src/cli/handlers/plugins.ts) | `.ts` |
| 7 | [handlers/util.tsx](./src/cli/handlers/util.tsx) | `.tsx` |
| 8 | [ndjsonSafeStringify.ts](./src/cli/ndjsonSafeStringify.ts) | `.ts` |
| 9 | [print.ts](./src/cli/print.ts) | `.ts` |
| 10 | [remoteIO.ts](./src/cli/remoteIO.ts) | `.ts` |
| 11 | [structuredIO.ts](./src/cli/structuredIO.ts) | `.ts` |
| 12 | [transports/HybridTransport.ts](./src/cli/transports/HybridTransport.ts) | `.ts` |
| 13 | [transports/SSETransport.ts](./src/cli/transports/SSETransport.ts) | `.ts` |
| 14 | [transports/SerialBatchEventUploader.ts](./src/cli/transports/SerialBatchEventUploader.ts) | `.ts` |
| 15 | [transports/WebSocketTransport.ts](./src/cli/transports/WebSocketTransport.ts) | `.ts` |
| 16 | [transports/WorkerStateUploader.ts](./src/cli/transports/WorkerStateUploader.ts) | `.ts` |
| 17 | [transports/ccrClient.ts](./src/cli/transports/ccrClient.ts) | `.ts` |
| 18 | [transports/transportUtils.ts](./src/cli/transports/transportUtils.ts) | `.ts` |
| 19 | [update.ts](./src/cli/update.ts) | `.ts` |

### commands/

| # | File | Type |
|:--|:-----|:-----|
| 1 | [add-dir/add-dir.tsx](./src/commands/add-dir/add-dir.tsx) | `.tsx` |
| 2 | [add-dir/index.ts](./src/commands/add-dir/index.ts) | `.ts` |
| 3 | [add-dir/validation.ts](./src/commands/add-dir/validation.ts) | `.ts` |
| 4 | [advisor.ts](./src/commands/advisor.ts) | `.ts` |
| 5 | [agents/agents.tsx](./src/commands/agents/agents.tsx) | `.tsx` |
| 6 | [agents/index.ts](./src/commands/agents/index.ts) | `.ts` |
| 7 | [ant-trace/index.js](./src/commands/ant-trace/index.js) | `.js` |
| 8 | [autofix-pr/index.js](./src/commands/autofix-pr/index.js) | `.js` |
| 9 | [backfill-sessions/index.js](./src/commands/backfill-sessions/index.js) | `.js` |
| 10 | [branch/branch.ts](./src/commands/branch/branch.ts) | `.ts` |
| 11 | [branch/index.ts](./src/commands/branch/index.ts) | `.ts` |
| 12 | [break-cache/index.js](./src/commands/break-cache/index.js) | `.js` |
| 13 | [bridge-kick.ts](./src/commands/bridge-kick.ts) | `.ts` |
| 14 | [bridge/bridge.tsx](./src/commands/bridge/bridge.tsx) | `.tsx` |
| 15 | [bridge/index.ts](./src/commands/bridge/index.ts) | `.ts` |
| 16 | [brief.ts](./src/commands/brief.ts) | `.ts` |
| 17 | [btw/btw.tsx](./src/commands/btw/btw.tsx) | `.tsx` |
| 18 | [btw/index.ts](./src/commands/btw/index.ts) | `.ts` |
| 19 | [bughunter/index.js](./src/commands/bughunter/index.js) | `.js` |
| 20 | [chrome/chrome.tsx](./src/commands/chrome/chrome.tsx) | `.tsx` |
| 21 | [chrome/index.ts](./src/commands/chrome/index.ts) | `.ts` |
| 22 | [clear/caches.ts](./src/commands/clear/caches.ts) | `.ts` |
| 23 | [clear/clear.ts](./src/commands/clear/clear.ts) | `.ts` |
| 24 | [clear/conversation.ts](./src/commands/clear/conversation.ts) | `.ts` |
| 25 | [clear/index.ts](./src/commands/clear/index.ts) | `.ts` |
| 26 | [color/color.ts](./src/commands/color/color.ts) | `.ts` |
| 27 | [color/index.ts](./src/commands/color/index.ts) | `.ts` |
| 28 | [commit-push-pr.ts](./src/commands/commit-push-pr.ts) | `.ts` |
| 29 | [commit.ts](./src/commands/commit.ts) | `.ts` |
| 30 | [compact/compact.ts](./src/commands/compact/compact.ts) | `.ts` |
| 31 | [compact/index.ts](./src/commands/compact/index.ts) | `.ts` |
| 32 | [config/config.tsx](./src/commands/config/config.tsx) | `.tsx` |
| 33 | [config/index.ts](./src/commands/config/index.ts) | `.ts` |
| 34 | [context/context-noninteractive.ts](./src/commands/context/context-noninteractive.ts) | `.ts` |
| 35 | [context/context.tsx](./src/commands/context/context.tsx) | `.tsx` |
| 36 | [context/index.ts](./src/commands/context/index.ts) | `.ts` |
| 37 | [copy/copy.tsx](./src/commands/copy/copy.tsx) | `.tsx` |
| 38 | [copy/index.ts](./src/commands/copy/index.ts) | `.ts` |
| 39 | [cost/cost.ts](./src/commands/cost/cost.ts) | `.ts` |
| 40 | [cost/index.ts](./src/commands/cost/index.ts) | `.ts` |
| 41 | [createMovedToPluginCommand.ts](./src/commands/createMovedToPluginCommand.ts) | `.ts` |
| 42 | [ctx_viz/index.js](./src/commands/ctx_viz/index.js) | `.js` |
| 43 | [debug-tool-call/index.js](./src/commands/debug-tool-call/index.js) | `.js` |
| 44 | [desktop/desktop.tsx](./src/commands/desktop/desktop.tsx) | `.tsx` |
| 45 | [desktop/index.ts](./src/commands/desktop/index.ts) | `.ts` |
| 46 | [diff/diff.tsx](./src/commands/diff/diff.tsx) | `.tsx` |
| 47 | [diff/index.ts](./src/commands/diff/index.ts) | `.ts` |
| 48 | [doctor/doctor.tsx](./src/commands/doctor/doctor.tsx) | `.tsx` |
| 49 | [doctor/index.ts](./src/commands/doctor/index.ts) | `.ts` |
| 50 | [effort/effort.tsx](./src/commands/effort/effort.tsx) | `.tsx` |
| 51 | [effort/index.ts](./src/commands/effort/index.ts) | `.ts` |
| 52 | [env/index.js](./src/commands/env/index.js) | `.js` |
| 53 | [exit/exit.tsx](./src/commands/exit/exit.tsx) | `.tsx` |
| 54 | [exit/index.ts](./src/commands/exit/index.ts) | `.ts` |
| 55 | [export/export.tsx](./src/commands/export/export.tsx) | `.tsx` |
| 56 | [export/index.ts](./src/commands/export/index.ts) | `.ts` |
| 57 | [extra-usage/extra-usage-core.ts](./src/commands/extra-usage/extra-usage-core.ts) | `.ts` |
| 58 | [extra-usage/extra-usage-noninteractive.ts](./src/commands/extra-usage/extra-usage-noninteractive.ts) | `.ts` |
| 59 | [extra-usage/extra-usage.tsx](./src/commands/extra-usage/extra-usage.tsx) | `.tsx` |
| 60 | [extra-usage/index.ts](./src/commands/extra-usage/index.ts) | `.ts` |
| 61 | [fast/fast.tsx](./src/commands/fast/fast.tsx) | `.tsx` |
| 62 | [fast/index.ts](./src/commands/fast/index.ts) | `.ts` |
| 63 | [feedback/feedback.tsx](./src/commands/feedback/feedback.tsx) | `.tsx` |
| 64 | [feedback/index.ts](./src/commands/feedback/index.ts) | `.ts` |
| 65 | [files/files.ts](./src/commands/files/files.ts) | `.ts` |
| 66 | [files/index.ts](./src/commands/files/index.ts) | `.ts` |
| 67 | [good-claude/index.js](./src/commands/good-claude/index.js) | `.js` |
| 68 | [heapdump/heapdump.ts](./src/commands/heapdump/heapdump.ts) | `.ts` |
| 69 | [heapdump/index.ts](./src/commands/heapdump/index.ts) | `.ts` |
| 70 | [help/help.tsx](./src/commands/help/help.tsx) | `.tsx` |
| 71 | [help/index.ts](./src/commands/help/index.ts) | `.ts` |
| 72 | [hooks/hooks.tsx](./src/commands/hooks/hooks.tsx) | `.tsx` |
| 73 | [hooks/index.ts](./src/commands/hooks/index.ts) | `.ts` |
| 74 | [ide/ide.tsx](./src/commands/ide/ide.tsx) | `.tsx` |
| 75 | [ide/index.ts](./src/commands/ide/index.ts) | `.ts` |
| 76 | [init-verifiers.ts](./src/commands/init-verifiers.ts) | `.ts` |
| 77 | [init.ts](./src/commands/init.ts) | `.ts` |
| 78 | [insights.ts](./src/commands/insights.ts) | `.ts` |
| 79 | [install-github-app/ApiKeyStep.tsx](./src/commands/install-github-app/ApiKeyStep.tsx) | `.tsx` |
| 80 | [install-github-app/CheckExistingSecretStep.tsx](./src/commands/install-github-app/CheckExistingSecretStep.tsx) | `.tsx` |
| 81 | [install-github-app/CheckGitHubStep.tsx](./src/commands/install-github-app/CheckGitHubStep.tsx) | `.tsx` |
| 82 | [install-github-app/ChooseRepoStep.tsx](./src/commands/install-github-app/ChooseRepoStep.tsx) | `.tsx` |
| 83 | [install-github-app/CreatingStep.tsx](./src/commands/install-github-app/CreatingStep.tsx) | `.tsx` |
| 84 | [install-github-app/ErrorStep.tsx](./src/commands/install-github-app/ErrorStep.tsx) | `.tsx` |
| 85 | [install-github-app/ExistingWorkflowStep.tsx](./src/commands/install-github-app/ExistingWorkflowStep.tsx) | `.tsx` |
| 86 | [install-github-app/InstallAppStep.tsx](./src/commands/install-github-app/InstallAppStep.tsx) | `.tsx` |
| 87 | [install-github-app/OAuthFlowStep.tsx](./src/commands/install-github-app/OAuthFlowStep.tsx) | `.tsx` |
| 88 | [install-github-app/SuccessStep.tsx](./src/commands/install-github-app/SuccessStep.tsx) | `.tsx` |
| 89 | [install-github-app/WarningsStep.tsx](./src/commands/install-github-app/WarningsStep.tsx) | `.tsx` |
| 90 | [install-github-app/index.ts](./src/commands/install-github-app/index.ts) | `.ts` |
| 91 | [install-github-app/install-github-app.tsx](./src/commands/install-github-app/install-github-app.tsx) | `.tsx` |
| 92 | [install-github-app/setupGitHubActions.ts](./src/commands/install-github-app/setupGitHubActions.ts) | `.ts` |
| 93 | [install-slack-app/index.ts](./src/commands/install-slack-app/index.ts) | `.ts` |
| 94 | [install-slack-app/install-slack-app.ts](./src/commands/install-slack-app/install-slack-app.ts) | `.ts` |
| 95 | [install.tsx](./src/commands/install.tsx) | `.tsx` |
| 96 | [issue/index.js](./src/commands/issue/index.js) | `.js` |
| 97 | [keybindings/index.ts](./src/commands/keybindings/index.ts) | `.ts` |
| 98 | [keybindings/keybindings.ts](./src/commands/keybindings/keybindings.ts) | `.ts` |
| 99 | [login/index.ts](./src/commands/login/index.ts) | `.ts` |
| 100 | [login/login.tsx](./src/commands/login/login.tsx) | `.tsx` |
| 101 | [logout/index.ts](./src/commands/logout/index.ts) | `.ts` |
| 102 | [logout/logout.tsx](./src/commands/logout/logout.tsx) | `.tsx` |
| 103 | [mcp/addCommand.ts](./src/commands/mcp/addCommand.ts) | `.ts` |
| 104 | [mcp/index.ts](./src/commands/mcp/index.ts) | `.ts` |
| 105 | [mcp/mcp.tsx](./src/commands/mcp/mcp.tsx) | `.tsx` |
| 106 | [mcp/xaaIdpCommand.ts](./src/commands/mcp/xaaIdpCommand.ts) | `.ts` |
| 107 | [memory/index.ts](./src/commands/memory/index.ts) | `.ts` |
| 108 | [memory/memory.tsx](./src/commands/memory/memory.tsx) | `.tsx` |
| 109 | [mobile/index.ts](./src/commands/mobile/index.ts) | `.ts` |
| 110 | [mobile/mobile.tsx](./src/commands/mobile/mobile.tsx) | `.tsx` |
| 111 | [mock-limits/index.js](./src/commands/mock-limits/index.js) | `.js` |
| 112 | [model/index.ts](./src/commands/model/index.ts) | `.ts` |
| 113 | [model/model.tsx](./src/commands/model/model.tsx) | `.tsx` |
| 114 | [oauth-refresh/index.js](./src/commands/oauth-refresh/index.js) | `.js` |
| 115 | [onboarding/index.js](./src/commands/onboarding/index.js) | `.js` |
| 116 | [output-style/index.ts](./src/commands/output-style/index.ts) | `.ts` |
| 117 | [output-style/output-style.tsx](./src/commands/output-style/output-style.tsx) | `.tsx` |
| 118 | [passes/index.ts](./src/commands/passes/index.ts) | `.ts` |
| 119 | [passes/passes.tsx](./src/commands/passes/passes.tsx) | `.tsx` |
| 120 | [perf-issue/index.js](./src/commands/perf-issue/index.js) | `.js` |
| 121 | [permissions/index.ts](./src/commands/permissions/index.ts) | `.ts` |
| 122 | [permissions/permissions.tsx](./src/commands/permissions/permissions.tsx) | `.tsx` |
| 123 | [plan/index.ts](./src/commands/plan/index.ts) | `.ts` |
| 124 | [plan/plan.tsx](./src/commands/plan/plan.tsx) | `.tsx` |
| 125 | [plugin/AddMarketplace.tsx](./src/commands/plugin/AddMarketplace.tsx) | `.tsx` |
| 126 | [plugin/BrowseMarketplace.tsx](./src/commands/plugin/BrowseMarketplace.tsx) | `.tsx` |
| 127 | [plugin/DiscoverPlugins.tsx](./src/commands/plugin/DiscoverPlugins.tsx) | `.tsx` |
| 128 | [plugin/ManageMarketplaces.tsx](./src/commands/plugin/ManageMarketplaces.tsx) | `.tsx` |
| 129 | [plugin/ManagePlugins.tsx](./src/commands/plugin/ManagePlugins.tsx) | `.tsx` |
| 130 | [plugin/PluginErrors.tsx](./src/commands/plugin/PluginErrors.tsx) | `.tsx` |
| 131 | [plugin/PluginOptionsDialog.tsx](./src/commands/plugin/PluginOptionsDialog.tsx) | `.tsx` |
| 132 | [plugin/PluginOptionsFlow.tsx](./src/commands/plugin/PluginOptionsFlow.tsx) | `.tsx` |
| 133 | [plugin/PluginSettings.tsx](./src/commands/plugin/PluginSettings.tsx) | `.tsx` |
| 134 | [plugin/PluginTrustWarning.tsx](./src/commands/plugin/PluginTrustWarning.tsx) | `.tsx` |
| 135 | [plugin/UnifiedInstalledCell.tsx](./src/commands/plugin/UnifiedInstalledCell.tsx) | `.tsx` |
| 136 | [plugin/ValidatePlugin.tsx](./src/commands/plugin/ValidatePlugin.tsx) | `.tsx` |
| 137 | [plugin/index.tsx](./src/commands/plugin/index.tsx) | `.tsx` |
| 138 | [plugin/parseArgs.ts](./src/commands/plugin/parseArgs.ts) | `.ts` |
| 139 | [plugin/plugin.tsx](./src/commands/plugin/plugin.tsx) | `.tsx` |
| 140 | [plugin/pluginDetailsHelpers.tsx](./src/commands/plugin/pluginDetailsHelpers.tsx) | `.tsx` |
| 141 | [plugin/usePagination.ts](./src/commands/plugin/usePagination.ts) | `.ts` |
| 142 | [pr_comments/index.ts](./src/commands/pr_comments/index.ts) | `.ts` |
| 143 | [privacy-settings/index.ts](./src/commands/privacy-settings/index.ts) | `.ts` |
| 144 | [privacy-settings/privacy-settings.tsx](./src/commands/privacy-settings/privacy-settings.tsx) | `.tsx` |
| 145 | [rate-limit-options/index.ts](./src/commands/rate-limit-options/index.ts) | `.ts` |
| 146 | [rate-limit-options/rate-limit-options.tsx](./src/commands/rate-limit-options/rate-limit-options.tsx) | `.tsx` |
| 147 | [release-notes/index.ts](./src/commands/release-notes/index.ts) | `.ts` |
| 148 | [release-notes/release-notes.ts](./src/commands/release-notes/release-notes.ts) | `.ts` |
| 149 | [reload-plugins/index.ts](./src/commands/reload-plugins/index.ts) | `.ts` |
| 150 | [reload-plugins/reload-plugins.ts](./src/commands/reload-plugins/reload-plugins.ts) | `.ts` |
| 151 | [remote-env/index.ts](./src/commands/remote-env/index.ts) | `.ts` |
| 152 | [remote-env/remote-env.tsx](./src/commands/remote-env/remote-env.tsx) | `.tsx` |
| 153 | [remote-setup/api.ts](./src/commands/remote-setup/api.ts) | `.ts` |
| 154 | [remote-setup/index.ts](./src/commands/remote-setup/index.ts) | `.ts` |
| 155 | [remote-setup/remote-setup.tsx](./src/commands/remote-setup/remote-setup.tsx) | `.tsx` |
| 156 | [rename/generateSessionName.ts](./src/commands/rename/generateSessionName.ts) | `.ts` |
| 157 | [rename/index.ts](./src/commands/rename/index.ts) | `.ts` |
| 158 | [rename/rename.ts](./src/commands/rename/rename.ts) | `.ts` |
| 159 | [reset-limits/index.js](./src/commands/reset-limits/index.js) | `.js` |
| 160 | [resume/index.ts](./src/commands/resume/index.ts) | `.ts` |
| 161 | [resume/resume.tsx](./src/commands/resume/resume.tsx) | `.tsx` |
| 162 | [review.ts](./src/commands/review.ts) | `.ts` |
| 163 | [review/UltrareviewOverageDialog.tsx](./src/commands/review/UltrareviewOverageDialog.tsx) | `.tsx` |
| 164 | [review/reviewRemote.ts](./src/commands/review/reviewRemote.ts) | `.ts` |
| 165 | [review/ultrareviewCommand.tsx](./src/commands/review/ultrareviewCommand.tsx) | `.tsx` |
| 166 | [review/ultrareviewEnabled.ts](./src/commands/review/ultrareviewEnabled.ts) | `.ts` |
| 167 | [rewind/index.ts](./src/commands/rewind/index.ts) | `.ts` |
| 168 | [rewind/rewind.ts](./src/commands/rewind/rewind.ts) | `.ts` |
| 169 | [sandbox-toggle/index.ts](./src/commands/sandbox-toggle/index.ts) | `.ts` |
| 170 | [sandbox-toggle/sandbox-toggle.tsx](./src/commands/sandbox-toggle/sandbox-toggle.tsx) | `.tsx` |
| 171 | [security-review.ts](./src/commands/security-review.ts) | `.ts` |
| 172 | [session/index.ts](./src/commands/session/index.ts) | `.ts` |
| 173 | [session/session.tsx](./src/commands/session/session.tsx) | `.tsx` |
| 174 | [share/index.js](./src/commands/share/index.js) | `.js` |
| 175 | [skills/index.ts](./src/commands/skills/index.ts) | `.ts` |
| 176 | [skills/skills.tsx](./src/commands/skills/skills.tsx) | `.tsx` |
| 177 | [stats/index.ts](./src/commands/stats/index.ts) | `.ts` |
| 178 | [stats/stats.tsx](./src/commands/stats/stats.tsx) | `.tsx` |
| 179 | [status/index.ts](./src/commands/status/index.ts) | `.ts` |
| 180 | [status/status.tsx](./src/commands/status/status.tsx) | `.tsx` |
| 181 | [statusline.tsx](./src/commands/statusline.tsx) | `.tsx` |
| 182 | [stickers/index.ts](./src/commands/stickers/index.ts) | `.ts` |
| 183 | [stickers/stickers.ts](./src/commands/stickers/stickers.ts) | `.ts` |
| 184 | [summary/index.js](./src/commands/summary/index.js) | `.js` |
| 185 | [tag/index.ts](./src/commands/tag/index.ts) | `.ts` |
| 186 | [tag/tag.tsx](./src/commands/tag/tag.tsx) | `.tsx` |
| 187 | [tasks/index.ts](./src/commands/tasks/index.ts) | `.ts` |
| 188 | [tasks/tasks.tsx](./src/commands/tasks/tasks.tsx) | `.tsx` |
| 189 | [teleport/index.js](./src/commands/teleport/index.js) | `.js` |
| 190 | [terminalSetup/index.ts](./src/commands/terminalSetup/index.ts) | `.ts` |
| 191 | [terminalSetup/terminalSetup.tsx](./src/commands/terminalSetup/terminalSetup.tsx) | `.tsx` |
| 192 | [theme/index.ts](./src/commands/theme/index.ts) | `.ts` |
| 193 | [theme/theme.tsx](./src/commands/theme/theme.tsx) | `.tsx` |
| 194 | [thinkback-play/index.ts](./src/commands/thinkback-play/index.ts) | `.ts` |
| 195 | [thinkback-play/thinkback-play.ts](./src/commands/thinkback-play/thinkback-play.ts) | `.ts` |
| 196 | [thinkback/index.ts](./src/commands/thinkback/index.ts) | `.ts` |
| 197 | [thinkback/thinkback.tsx](./src/commands/thinkback/thinkback.tsx) | `.tsx` |
| 198 | [ultraplan.tsx](./src/commands/ultraplan.tsx) | `.tsx` |
| 199 | [upgrade/index.ts](./src/commands/upgrade/index.ts) | `.ts` |
| 200 | [upgrade/upgrade.tsx](./src/commands/upgrade/upgrade.tsx) | `.tsx` |
| 201 | [usage/index.ts](./src/commands/usage/index.ts) | `.ts` |
| 202 | [usage/usage.tsx](./src/commands/usage/usage.tsx) | `.tsx` |
| 203 | [version.ts](./src/commands/version.ts) | `.ts` |
| 204 | [vim/index.ts](./src/commands/vim/index.ts) | `.ts` |
| 205 | [vim/vim.ts](./src/commands/vim/vim.ts) | `.ts` |
| 206 | [voice/index.ts](./src/commands/voice/index.ts) | `.ts` |
| 207 | [voice/voice.ts](./src/commands/voice/voice.ts) | `.ts` |

### components/

| # | File | Type |
|:--|:-----|:-----|
| 1 | [AgentProgressLine.tsx](./src/components/AgentProgressLine.tsx) | `.tsx` |
| 2 | [App.tsx](./src/components/App.tsx) | `.tsx` |
| 3 | [ApproveApiKey.tsx](./src/components/ApproveApiKey.tsx) | `.tsx` |
| 4 | [AutoModeOptInDialog.tsx](./src/components/AutoModeOptInDialog.tsx) | `.tsx` |
| 5 | [AutoUpdater.tsx](./src/components/AutoUpdater.tsx) | `.tsx` |
| 6 | [AutoUpdaterWrapper.tsx](./src/components/AutoUpdaterWrapper.tsx) | `.tsx` |
| 7 | [AwsAuthStatusBox.tsx](./src/components/AwsAuthStatusBox.tsx) | `.tsx` |
| 8 | [BaseTextInput.tsx](./src/components/BaseTextInput.tsx) | `.tsx` |
| 9 | [BashModeProgress.tsx](./src/components/BashModeProgress.tsx) | `.tsx` |
| 10 | [BridgeDialog.tsx](./src/components/BridgeDialog.tsx) | `.tsx` |
| 11 | [BypassPermissionsModeDialog.tsx](./src/components/BypassPermissionsModeDialog.tsx) | `.tsx` |
| 12 | [ChannelDowngradeDialog.tsx](./src/components/ChannelDowngradeDialog.tsx) | `.tsx` |
| 13 | [ClaudeCodeHint/PluginHintMenu.tsx](./src/components/ClaudeCodeHint/PluginHintMenu.tsx) | `.tsx` |
| 14 | [ClaudeInChromeOnboarding.tsx](./src/components/ClaudeInChromeOnboarding.tsx) | `.tsx` |
| 15 | [ClaudeMdExternalIncludesDialog.tsx](./src/components/ClaudeMdExternalIncludesDialog.tsx) | `.tsx` |
| 16 | [ClickableImageRef.tsx](./src/components/ClickableImageRef.tsx) | `.tsx` |
| 17 | [CompactSummary.tsx](./src/components/CompactSummary.tsx) | `.tsx` |
| 18 | [ConfigurableShortcutHint.tsx](./src/components/ConfigurableShortcutHint.tsx) | `.tsx` |
| 19 | [ConsoleOAuthFlow.tsx](./src/components/ConsoleOAuthFlow.tsx) | `.tsx` |
| 20 | [ContextSuggestions.tsx](./src/components/ContextSuggestions.tsx) | `.tsx` |
| 21 | [ContextVisualization.tsx](./src/components/ContextVisualization.tsx) | `.tsx` |
| 22 | [CoordinatorAgentStatus.tsx](./src/components/CoordinatorAgentStatus.tsx) | `.tsx` |
| 23 | [CostThresholdDialog.tsx](./src/components/CostThresholdDialog.tsx) | `.tsx` |
| 24 | [CtrlOToExpand.tsx](./src/components/CtrlOToExpand.tsx) | `.tsx` |
| 25 | [CustomSelect/SelectMulti.tsx](./src/components/CustomSelect/SelectMulti.tsx) | `.tsx` |
| 26 | [CustomSelect/index.ts](./src/components/CustomSelect/index.ts) | `.ts` |
| 27 | [CustomSelect/option-map.ts](./src/components/CustomSelect/option-map.ts) | `.ts` |
| 28 | [CustomSelect/select-input-option.tsx](./src/components/CustomSelect/select-input-option.tsx) | `.tsx` |
| 29 | [CustomSelect/select-option.tsx](./src/components/CustomSelect/select-option.tsx) | `.tsx` |
| 30 | [CustomSelect/select.tsx](./src/components/CustomSelect/select.tsx) | `.tsx` |
| 31 | [CustomSelect/use-multi-select-state.ts](./src/components/CustomSelect/use-multi-select-state.ts) | `.ts` |
| 32 | [CustomSelect/use-select-input.ts](./src/components/CustomSelect/use-select-input.ts) | `.ts` |
| 33 | [CustomSelect/use-select-navigation.ts](./src/components/CustomSelect/use-select-navigation.ts) | `.ts` |
| 34 | [CustomSelect/use-select-state.ts](./src/components/CustomSelect/use-select-state.ts) | `.ts` |
| 35 | [DesktopHandoff.tsx](./src/components/DesktopHandoff.tsx) | `.tsx` |
| 36 | [DesktopUpsell/DesktopUpsellStartup.tsx](./src/components/DesktopUpsell/DesktopUpsellStartup.tsx) | `.tsx` |
| 37 | [DevBar.tsx](./src/components/DevBar.tsx) | `.tsx` |
| 38 | [DevChannelsDialog.tsx](./src/components/DevChannelsDialog.tsx) | `.tsx` |
| 39 | [DiagnosticsDisplay.tsx](./src/components/DiagnosticsDisplay.tsx) | `.tsx` |
| 40 | [EffortCallout.tsx](./src/components/EffortCallout.tsx) | `.tsx` |
| 41 | [EffortIndicator.ts](./src/components/EffortIndicator.ts) | `.ts` |
| 42 | [ExitFlow.tsx](./src/components/ExitFlow.tsx) | `.tsx` |
| 43 | [ExportDialog.tsx](./src/components/ExportDialog.tsx) | `.tsx` |
| 44 | [FallbackToolUseErrorMessage.tsx](./src/components/FallbackToolUseErrorMessage.tsx) | `.tsx` |
| 45 | [FallbackToolUseRejectedMessage.tsx](./src/components/FallbackToolUseRejectedMessage.tsx) | `.tsx` |
| 46 | [FastIcon.tsx](./src/components/FastIcon.tsx) | `.tsx` |
| 47 | [Feedback.tsx](./src/components/Feedback.tsx) | `.tsx` |
| 48 | [FeedbackSurvey/FeedbackSurvey.tsx](./src/components/FeedbackSurvey/FeedbackSurvey.tsx) | `.tsx` |
| 49 | [FeedbackSurvey/FeedbackSurveyView.tsx](./src/components/FeedbackSurvey/FeedbackSurveyView.tsx) | `.tsx` |
| 50 | [FeedbackSurvey/TranscriptSharePrompt.tsx](./src/components/FeedbackSurvey/TranscriptSharePrompt.tsx) | `.tsx` |
| 51 | [FeedbackSurvey/submitTranscriptShare.ts](./src/components/FeedbackSurvey/submitTranscriptShare.ts) | `.ts` |
| 52 | [FeedbackSurvey/useDebouncedDigitInput.ts](./src/components/FeedbackSurvey/useDebouncedDigitInput.ts) | `.ts` |
| 53 | [FeedbackSurvey/useFeedbackSurvey.tsx](./src/components/FeedbackSurvey/useFeedbackSurvey.tsx) | `.tsx` |
| 54 | [FeedbackSurvey/useMemorySurvey.tsx](./src/components/FeedbackSurvey/useMemorySurvey.tsx) | `.tsx` |
| 55 | [FeedbackSurvey/usePostCompactSurvey.tsx](./src/components/FeedbackSurvey/usePostCompactSurvey.tsx) | `.tsx` |
| 56 | [FeedbackSurvey/useSurveyState.tsx](./src/components/FeedbackSurvey/useSurveyState.tsx) | `.tsx` |
| 57 | [FileEditToolDiff.tsx](./src/components/FileEditToolDiff.tsx) | `.tsx` |
| 58 | [FileEditToolUpdatedMessage.tsx](./src/components/FileEditToolUpdatedMessage.tsx) | `.tsx` |
| 59 | [FileEditToolUseRejectedMessage.tsx](./src/components/FileEditToolUseRejectedMessage.tsx) | `.tsx` |
| 60 | [FilePathLink.tsx](./src/components/FilePathLink.tsx) | `.tsx` |
| 61 | [FullscreenLayout.tsx](./src/components/FullscreenLayout.tsx) | `.tsx` |
| 62 | [GlobalSearchDialog.tsx](./src/components/GlobalSearchDialog.tsx) | `.tsx` |
| 63 | [HelpV2/Commands.tsx](./src/components/HelpV2/Commands.tsx) | `.tsx` |
| 64 | [HelpV2/General.tsx](./src/components/HelpV2/General.tsx) | `.tsx` |
| 65 | [HelpV2/HelpV2.tsx](./src/components/HelpV2/HelpV2.tsx) | `.tsx` |
| 66 | [HighlightedCode.tsx](./src/components/HighlightedCode.tsx) | `.tsx` |
| 67 | [HighlightedCode/Fallback.tsx](./src/components/HighlightedCode/Fallback.tsx) | `.tsx` |
| 68 | [HistorySearchDialog.tsx](./src/components/HistorySearchDialog.tsx) | `.tsx` |
| 69 | [IdeAutoConnectDialog.tsx](./src/components/IdeAutoConnectDialog.tsx) | `.tsx` |
| 70 | [IdeOnboardingDialog.tsx](./src/components/IdeOnboardingDialog.tsx) | `.tsx` |
| 71 | [IdeStatusIndicator.tsx](./src/components/IdeStatusIndicator.tsx) | `.tsx` |
| 72 | [IdleReturnDialog.tsx](./src/components/IdleReturnDialog.tsx) | `.tsx` |
| 73 | [InterruptedByUser.tsx](./src/components/InterruptedByUser.tsx) | `.tsx` |
| 74 | [InvalidConfigDialog.tsx](./src/components/InvalidConfigDialog.tsx) | `.tsx` |
| 75 | [InvalidSettingsDialog.tsx](./src/components/InvalidSettingsDialog.tsx) | `.tsx` |
| 76 | [KeybindingWarnings.tsx](./src/components/KeybindingWarnings.tsx) | `.tsx` |
| 77 | [LanguagePicker.tsx](./src/components/LanguagePicker.tsx) | `.tsx` |
| 78 | [LogSelector.tsx](./src/components/LogSelector.tsx) | `.tsx` |
| 79 | [LogoV2/AnimatedAsterisk.tsx](./src/components/LogoV2/AnimatedAsterisk.tsx) | `.tsx` |
| 80 | [LogoV2/AnimatedClawd.tsx](./src/components/LogoV2/AnimatedClawd.tsx) | `.tsx` |
| 81 | [LogoV2/ChannelsNotice.tsx](./src/components/LogoV2/ChannelsNotice.tsx) | `.tsx` |
| 82 | [LogoV2/Clawd.tsx](./src/components/LogoV2/Clawd.tsx) | `.tsx` |
| 83 | [LogoV2/CondensedLogo.tsx](./src/components/LogoV2/CondensedLogo.tsx) | `.tsx` |
| 84 | [LogoV2/EmergencyTip.tsx](./src/components/LogoV2/EmergencyTip.tsx) | `.tsx` |
| 85 | [LogoV2/Feed.tsx](./src/components/LogoV2/Feed.tsx) | `.tsx` |
| 86 | [LogoV2/FeedColumn.tsx](./src/components/LogoV2/FeedColumn.tsx) | `.tsx` |
| 87 | [LogoV2/GuestPassesUpsell.tsx](./src/components/LogoV2/GuestPassesUpsell.tsx) | `.tsx` |
| 88 | [LogoV2/LogoV2.tsx](./src/components/LogoV2/LogoV2.tsx) | `.tsx` |
| 89 | [LogoV2/Opus1mMergeNotice.tsx](./src/components/LogoV2/Opus1mMergeNotice.tsx) | `.tsx` |
| 90 | [LogoV2/OverageCreditUpsell.tsx](./src/components/LogoV2/OverageCreditUpsell.tsx) | `.tsx` |
| 91 | [LogoV2/VoiceModeNotice.tsx](./src/components/LogoV2/VoiceModeNotice.tsx) | `.tsx` |
| 92 | [LogoV2/WelcomeV2.tsx](./src/components/LogoV2/WelcomeV2.tsx) | `.tsx` |
| 93 | [LogoV2/feedConfigs.tsx](./src/components/LogoV2/feedConfigs.tsx) | `.tsx` |
| 94 | [LspRecommendation/LspRecommendationMenu.tsx](./src/components/LspRecommendation/LspRecommendationMenu.tsx) | `.tsx` |
| 95 | [MCPServerApprovalDialog.tsx](./src/components/MCPServerApprovalDialog.tsx) | `.tsx` |
| 96 | [MCPServerDesktopImportDialog.tsx](./src/components/MCPServerDesktopImportDialog.tsx) | `.tsx` |
| 97 | [MCPServerDialogCopy.tsx](./src/components/MCPServerDialogCopy.tsx) | `.tsx` |
| 98 | [MCPServerMultiselectDialog.tsx](./src/components/MCPServerMultiselectDialog.tsx) | `.tsx` |
| 99 | [ManagedSettingsSecurityDialog/ManagedSettingsSecurityDialog.tsx](./src/components/ManagedSettingsSecurityDialog/ManagedSettingsSecurityDialog.tsx) | `.tsx` |
| 100 | [ManagedSettingsSecurityDialog/utils.ts](./src/components/ManagedSettingsSecurityDialog/utils.ts) | `.ts` |
| 101 | [Markdown.tsx](./src/components/Markdown.tsx) | `.tsx` |
| 102 | [MarkdownTable.tsx](./src/components/MarkdownTable.tsx) | `.tsx` |
| 103 | [MemoryUsageIndicator.tsx](./src/components/MemoryUsageIndicator.tsx) | `.tsx` |
| 104 | [Message.tsx](./src/components/Message.tsx) | `.tsx` |
| 105 | [MessageModel.tsx](./src/components/MessageModel.tsx) | `.tsx` |
| 106 | [MessageResponse.tsx](./src/components/MessageResponse.tsx) | `.tsx` |
| 107 | [MessageRow.tsx](./src/components/MessageRow.tsx) | `.tsx` |
| 108 | [MessageSelector.tsx](./src/components/MessageSelector.tsx) | `.tsx` |
| 109 | [MessageTimestamp.tsx](./src/components/MessageTimestamp.tsx) | `.tsx` |
| 110 | [Messages.tsx](./src/components/Messages.tsx) | `.tsx` |
| 111 | [ModelPicker.tsx](./src/components/ModelPicker.tsx) | `.tsx` |
| 112 | [NativeAutoUpdater.tsx](./src/components/NativeAutoUpdater.tsx) | `.tsx` |
| 113 | [NotebookEditToolUseRejectedMessage.tsx](./src/components/NotebookEditToolUseRejectedMessage.tsx) | `.tsx` |
| 114 | [OffscreenFreeze.tsx](./src/components/OffscreenFreeze.tsx) | `.tsx` |
| 115 | [Onboarding.tsx](./src/components/Onboarding.tsx) | `.tsx` |
| 116 | [OutputStylePicker.tsx](./src/components/OutputStylePicker.tsx) | `.tsx` |
| 117 | [PackageManagerAutoUpdater.tsx](./src/components/PackageManagerAutoUpdater.tsx) | `.tsx` |
| 118 | [Passes/Passes.tsx](./src/components/Passes/Passes.tsx) | `.tsx` |
| 119 | [PrBadge.tsx](./src/components/PrBadge.tsx) | `.tsx` |
| 120 | [PressEnterToContinue.tsx](./src/components/PressEnterToContinue.tsx) | `.tsx` |
| 121 | [PromptInput/HistorySearchInput.tsx](./src/components/PromptInput/HistorySearchInput.tsx) | `.tsx` |
| 122 | [PromptInput/IssueFlagBanner.tsx](./src/components/PromptInput/IssueFlagBanner.tsx) | `.tsx` |
| 123 | [PromptInput/Notifications.tsx](./src/components/PromptInput/Notifications.tsx) | `.tsx` |
| 124 | [PromptInput/PromptInput.tsx](./src/components/PromptInput/PromptInput.tsx) | `.tsx` |
| 125 | [PromptInput/PromptInputFooter.tsx](./src/components/PromptInput/PromptInputFooter.tsx) | `.tsx` |
| 126 | [PromptInput/PromptInputFooterLeftSide.tsx](./src/components/PromptInput/PromptInputFooterLeftSide.tsx) | `.tsx` |
| 127 | [PromptInput/PromptInputFooterSuggestions.tsx](./src/components/PromptInput/PromptInputFooterSuggestions.tsx) | `.tsx` |
| 128 | [PromptInput/PromptInputHelpMenu.tsx](./src/components/PromptInput/PromptInputHelpMenu.tsx) | `.tsx` |
| 129 | [PromptInput/PromptInputModeIndicator.tsx](./src/components/PromptInput/PromptInputModeIndicator.tsx) | `.tsx` |
| 130 | [PromptInput/PromptInputQueuedCommands.tsx](./src/components/PromptInput/PromptInputQueuedCommands.tsx) | `.tsx` |
| 131 | [PromptInput/PromptInputStashNotice.tsx](./src/components/PromptInput/PromptInputStashNotice.tsx) | `.tsx` |
| 132 | [PromptInput/SandboxPromptFooterHint.tsx](./src/components/PromptInput/SandboxPromptFooterHint.tsx) | `.tsx` |
| 133 | [PromptInput/ShimmeredInput.tsx](./src/components/PromptInput/ShimmeredInput.tsx) | `.tsx` |
| 134 | [PromptInput/VoiceIndicator.tsx](./src/components/PromptInput/VoiceIndicator.tsx) | `.tsx` |
| 135 | [PromptInput/inputModes.ts](./src/components/PromptInput/inputModes.ts) | `.ts` |
| 136 | [PromptInput/inputPaste.ts](./src/components/PromptInput/inputPaste.ts) | `.ts` |
| 137 | [PromptInput/useMaybeTruncateInput.ts](./src/components/PromptInput/useMaybeTruncateInput.ts) | `.ts` |
| 138 | [PromptInput/usePromptInputPlaceholder.ts](./src/components/PromptInput/usePromptInputPlaceholder.ts) | `.ts` |
| 139 | [PromptInput/useShowFastIconHint.ts](./src/components/PromptInput/useShowFastIconHint.ts) | `.ts` |
| 140 | [PromptInput/useSwarmBanner.ts](./src/components/PromptInput/useSwarmBanner.ts) | `.ts` |
| 141 | [PromptInput/utils.ts](./src/components/PromptInput/utils.ts) | `.ts` |
| 142 | [QuickOpenDialog.tsx](./src/components/QuickOpenDialog.tsx) | `.tsx` |
| 143 | [RemoteCallout.tsx](./src/components/RemoteCallout.tsx) | `.tsx` |
| 144 | [RemoteEnvironmentDialog.tsx](./src/components/RemoteEnvironmentDialog.tsx) | `.tsx` |
| 145 | [ResumeTask.tsx](./src/components/ResumeTask.tsx) | `.tsx` |
| 146 | [SandboxViolationExpandedView.tsx](./src/components/SandboxViolationExpandedView.tsx) | `.tsx` |
| 147 | [ScrollKeybindingHandler.tsx](./src/components/ScrollKeybindingHandler.tsx) | `.tsx` |
| 148 | [SearchBox.tsx](./src/components/SearchBox.tsx) | `.tsx` |
| 149 | [SentryErrorBoundary.ts](./src/components/SentryErrorBoundary.ts) | `.ts` |
| 150 | [SessionBackgroundHint.tsx](./src/components/SessionBackgroundHint.tsx) | `.tsx` |
| 151 | [SessionPreview.tsx](./src/components/SessionPreview.tsx) | `.tsx` |
| 152 | [Settings/Config.tsx](./src/components/Settings/Config.tsx) | `.tsx` |
| 153 | [Settings/Settings.tsx](./src/components/Settings/Settings.tsx) | `.tsx` |
| 154 | [Settings/Status.tsx](./src/components/Settings/Status.tsx) | `.tsx` |
| 155 | [Settings/Usage.tsx](./src/components/Settings/Usage.tsx) | `.tsx` |
| 156 | [ShowInIDEPrompt.tsx](./src/components/ShowInIDEPrompt.tsx) | `.tsx` |
| 157 | [SkillImprovementSurvey.tsx](./src/components/SkillImprovementSurvey.tsx) | `.tsx` |
| 158 | [Spinner.tsx](./src/components/Spinner.tsx) | `.tsx` |
| 159 | [Spinner/FlashingChar.tsx](./src/components/Spinner/FlashingChar.tsx) | `.tsx` |
| 160 | [Spinner/GlimmerMessage.tsx](./src/components/Spinner/GlimmerMessage.tsx) | `.tsx` |
| 161 | [Spinner/ShimmerChar.tsx](./src/components/Spinner/ShimmerChar.tsx) | `.tsx` |
| 162 | [Spinner/SpinnerAnimationRow.tsx](./src/components/Spinner/SpinnerAnimationRow.tsx) | `.tsx` |
| 163 | [Spinner/SpinnerGlyph.tsx](./src/components/Spinner/SpinnerGlyph.tsx) | `.tsx` |
| 164 | [Spinner/TeammateSpinnerLine.tsx](./src/components/Spinner/TeammateSpinnerLine.tsx) | `.tsx` |
| 165 | [Spinner/TeammateSpinnerTree.tsx](./src/components/Spinner/TeammateSpinnerTree.tsx) | `.tsx` |
| 166 | [Spinner/index.ts](./src/components/Spinner/index.ts) | `.ts` |
| 167 | [Spinner/teammateSelectHint.ts](./src/components/Spinner/teammateSelectHint.ts) | `.ts` |
| 168 | [Spinner/useShimmerAnimation.ts](./src/components/Spinner/useShimmerAnimation.ts) | `.ts` |
| 169 | [Spinner/useStalledAnimation.ts](./src/components/Spinner/useStalledAnimation.ts) | `.ts` |
| 170 | [Spinner/utils.ts](./src/components/Spinner/utils.ts) | `.ts` |
| 171 | [Stats.tsx](./src/components/Stats.tsx) | `.tsx` |
| 172 | [StatusLine.tsx](./src/components/StatusLine.tsx) | `.tsx` |
| 173 | [StatusNotices.tsx](./src/components/StatusNotices.tsx) | `.tsx` |
| 174 | [StructuredDiff.tsx](./src/components/StructuredDiff.tsx) | `.tsx` |
| 175 | [StructuredDiff/Fallback.tsx](./src/components/StructuredDiff/Fallback.tsx) | `.tsx` |
| 176 | [StructuredDiff/colorDiff.ts](./src/components/StructuredDiff/colorDiff.ts) | `.ts` |
| 177 | [StructuredDiffList.tsx](./src/components/StructuredDiffList.tsx) | `.tsx` |
| 178 | [TagTabs.tsx](./src/components/TagTabs.tsx) | `.tsx` |
| 179 | [TaskListV2.tsx](./src/components/TaskListV2.tsx) | `.tsx` |
| 180 | [TeammateViewHeader.tsx](./src/components/TeammateViewHeader.tsx) | `.tsx` |
| 181 | [TeleportError.tsx](./src/components/TeleportError.tsx) | `.tsx` |
| 182 | [TeleportProgress.tsx](./src/components/TeleportProgress.tsx) | `.tsx` |
| 183 | [TeleportRepoMismatchDialog.tsx](./src/components/TeleportRepoMismatchDialog.tsx) | `.tsx` |
| 184 | [TeleportResumeWrapper.tsx](./src/components/TeleportResumeWrapper.tsx) | `.tsx` |
| 185 | [TeleportStash.tsx](./src/components/TeleportStash.tsx) | `.tsx` |
| 186 | [TextInput.tsx](./src/components/TextInput.tsx) | `.tsx` |
| 187 | [ThemePicker.tsx](./src/components/ThemePicker.tsx) | `.tsx` |
| 188 | [ThinkingToggle.tsx](./src/components/ThinkingToggle.tsx) | `.tsx` |
| 189 | [TokenWarning.tsx](./src/components/TokenWarning.tsx) | `.tsx` |
| 190 | [ToolUseLoader.tsx](./src/components/ToolUseLoader.tsx) | `.tsx` |
| 191 | [TrustDialog/TrustDialog.tsx](./src/components/TrustDialog/TrustDialog.tsx) | `.tsx` |
| 192 | [TrustDialog/utils.ts](./src/components/TrustDialog/utils.ts) | `.ts` |
| 193 | [ValidationErrorsList.tsx](./src/components/ValidationErrorsList.tsx) | `.tsx` |
| 194 | [VimTextInput.tsx](./src/components/VimTextInput.tsx) | `.tsx` |
| 195 | [VirtualMessageList.tsx](./src/components/VirtualMessageList.tsx) | `.tsx` |
| 196 | [WorkflowMultiselectDialog.tsx](./src/components/WorkflowMultiselectDialog.tsx) | `.tsx` |
| 197 | [WorktreeExitDialog.tsx](./src/components/WorktreeExitDialog.tsx) | `.tsx` |
| 198 | [agents/AgentDetail.tsx](./src/components/agents/AgentDetail.tsx) | `.tsx` |
| 199 | [agents/AgentEditor.tsx](./src/components/agents/AgentEditor.tsx) | `.tsx` |
| 200 | [agents/AgentNavigationFooter.tsx](./src/components/agents/AgentNavigationFooter.tsx) | `.tsx` |
| 201 | [agents/AgentsList.tsx](./src/components/agents/AgentsList.tsx) | `.tsx` |
| 202 | [agents/AgentsMenu.tsx](./src/components/agents/AgentsMenu.tsx) | `.tsx` |
| 203 | [agents/ColorPicker.tsx](./src/components/agents/ColorPicker.tsx) | `.tsx` |
| 204 | [agents/ModelSelector.tsx](./src/components/agents/ModelSelector.tsx) | `.tsx` |
| 205 | [agents/ToolSelector.tsx](./src/components/agents/ToolSelector.tsx) | `.tsx` |
| 206 | [agents/agentFileUtils.ts](./src/components/agents/agentFileUtils.ts) | `.ts` |
| 207 | [agents/generateAgent.ts](./src/components/agents/generateAgent.ts) | `.ts` |
| 208 | [agents/new-agent-creation/CreateAgentWizard.tsx](./src/components/agents/new-agent-creation/CreateAgentWizard.tsx) | `.tsx` |
| 209 | [agents/new-agent-creation/wizard-steps/ColorStep.tsx](./src/components/agents/new-agent-creation/wizard-steps/ColorStep.tsx) | `.tsx` |
| 210 | [agents/new-agent-creation/wizard-steps/ConfirmStep.tsx](./src/components/agents/new-agent-creation/wizard-steps/ConfirmStep.tsx) | `.tsx` |
| 211 | [agents/new-agent-creation/wizard-steps/ConfirmStepWrapper.tsx](./src/components/agents/new-agent-creation/wizard-steps/ConfirmStepWrapper.tsx) | `.tsx` |
| 212 | [agents/new-agent-creation/wizard-steps/DescriptionStep.tsx](./src/components/agents/new-agent-creation/wizard-steps/DescriptionStep.tsx) | `.tsx` |
| 213 | [agents/new-agent-creation/wizard-steps/GenerateStep.tsx](./src/components/agents/new-agent-creation/wizard-steps/GenerateStep.tsx) | `.tsx` |
| 214 | [agents/new-agent-creation/wizard-steps/LocationStep.tsx](./src/components/agents/new-agent-creation/wizard-steps/LocationStep.tsx) | `.tsx` |
| 215 | [agents/new-agent-creation/wizard-steps/MemoryStep.tsx](./src/components/agents/new-agent-creation/wizard-steps/MemoryStep.tsx) | `.tsx` |
| 216 | [agents/new-agent-creation/wizard-steps/MethodStep.tsx](./src/components/agents/new-agent-creation/wizard-steps/MethodStep.tsx) | `.tsx` |
| 217 | [agents/new-agent-creation/wizard-steps/ModelStep.tsx](./src/components/agents/new-agent-creation/wizard-steps/ModelStep.tsx) | `.tsx` |
| 218 | [agents/new-agent-creation/wizard-steps/PromptStep.tsx](./src/components/agents/new-agent-creation/wizard-steps/PromptStep.tsx) | `.tsx` |
| 219 | [agents/new-agent-creation/wizard-steps/ToolsStep.tsx](./src/components/agents/new-agent-creation/wizard-steps/ToolsStep.tsx) | `.tsx` |
| 220 | [agents/new-agent-creation/wizard-steps/TypeStep.tsx](./src/components/agents/new-agent-creation/wizard-steps/TypeStep.tsx) | `.tsx` |
| 221 | [agents/types.ts](./src/components/agents/types.ts) | `.ts` |
| 222 | [agents/utils.ts](./src/components/agents/utils.ts) | `.ts` |
| 223 | [agents/validateAgent.ts](./src/components/agents/validateAgent.ts) | `.ts` |
| 224 | [design-system/Byline.tsx](./src/components/design-system/Byline.tsx) | `.tsx` |
| 225 | [design-system/Dialog.tsx](./src/components/design-system/Dialog.tsx) | `.tsx` |
| 226 | [design-system/Divider.tsx](./src/components/design-system/Divider.tsx) | `.tsx` |
| 227 | [design-system/FuzzyPicker.tsx](./src/components/design-system/FuzzyPicker.tsx) | `.tsx` |
| 228 | [design-system/KeyboardShortcutHint.tsx](./src/components/design-system/KeyboardShortcutHint.tsx) | `.tsx` |
| 229 | [design-system/ListItem.tsx](./src/components/design-system/ListItem.tsx) | `.tsx` |
| 230 | [design-system/LoadingState.tsx](./src/components/design-system/LoadingState.tsx) | `.tsx` |
| 231 | [design-system/Pane.tsx](./src/components/design-system/Pane.tsx) | `.tsx` |
| 232 | [design-system/ProgressBar.tsx](./src/components/design-system/ProgressBar.tsx) | `.tsx` |
| 233 | [design-system/Ratchet.tsx](./src/components/design-system/Ratchet.tsx) | `.tsx` |
| 234 | [design-system/StatusIcon.tsx](./src/components/design-system/StatusIcon.tsx) | `.tsx` |
| 235 | [design-system/Tabs.tsx](./src/components/design-system/Tabs.tsx) | `.tsx` |
| 236 | [design-system/ThemeProvider.tsx](./src/components/design-system/ThemeProvider.tsx) | `.tsx` |
| 237 | [design-system/ThemedBox.tsx](./src/components/design-system/ThemedBox.tsx) | `.tsx` |
| 238 | [design-system/ThemedText.tsx](./src/components/design-system/ThemedText.tsx) | `.tsx` |
| 239 | [design-system/color.ts](./src/components/design-system/color.ts) | `.ts` |
| 240 | [diff/DiffDetailView.tsx](./src/components/diff/DiffDetailView.tsx) | `.tsx` |
| 241 | [diff/DiffDialog.tsx](./src/components/diff/DiffDialog.tsx) | `.tsx` |
| 242 | [diff/DiffFileList.tsx](./src/components/diff/DiffFileList.tsx) | `.tsx` |
| 243 | [grove/Grove.tsx](./src/components/grove/Grove.tsx) | `.tsx` |
| 244 | [hooks/HooksConfigMenu.tsx](./src/components/hooks/HooksConfigMenu.tsx) | `.tsx` |
| 245 | [hooks/PromptDialog.tsx](./src/components/hooks/PromptDialog.tsx) | `.tsx` |
| 246 | [hooks/SelectEventMode.tsx](./src/components/hooks/SelectEventMode.tsx) | `.tsx` |
| 247 | [hooks/SelectHookMode.tsx](./src/components/hooks/SelectHookMode.tsx) | `.tsx` |
| 248 | [hooks/SelectMatcherMode.tsx](./src/components/hooks/SelectMatcherMode.tsx) | `.tsx` |
| 249 | [hooks/ViewHookMode.tsx](./src/components/hooks/ViewHookMode.tsx) | `.tsx` |
| 250 | [mcp/CapabilitiesSection.tsx](./src/components/mcp/CapabilitiesSection.tsx) | `.tsx` |
| 251 | [mcp/ElicitationDialog.tsx](./src/components/mcp/ElicitationDialog.tsx) | `.tsx` |
| 252 | [mcp/MCPAgentServerMenu.tsx](./src/components/mcp/MCPAgentServerMenu.tsx) | `.tsx` |
| 253 | [mcp/MCPListPanel.tsx](./src/components/mcp/MCPListPanel.tsx) | `.tsx` |
| 254 | [mcp/MCPReconnect.tsx](./src/components/mcp/MCPReconnect.tsx) | `.tsx` |
| 255 | [mcp/MCPRemoteServerMenu.tsx](./src/components/mcp/MCPRemoteServerMenu.tsx) | `.tsx` |
| 256 | [mcp/MCPSettings.tsx](./src/components/mcp/MCPSettings.tsx) | `.tsx` |
| 257 | [mcp/MCPStdioServerMenu.tsx](./src/components/mcp/MCPStdioServerMenu.tsx) | `.tsx` |
| 258 | [mcp/MCPToolDetailView.tsx](./src/components/mcp/MCPToolDetailView.tsx) | `.tsx` |
| 259 | [mcp/MCPToolListView.tsx](./src/components/mcp/MCPToolListView.tsx) | `.tsx` |
| 260 | [mcp/McpParsingWarnings.tsx](./src/components/mcp/McpParsingWarnings.tsx) | `.tsx` |
| 261 | [mcp/index.ts](./src/components/mcp/index.ts) | `.ts` |
| 262 | [mcp/utils/reconnectHelpers.tsx](./src/components/mcp/utils/reconnectHelpers.tsx) | `.tsx` |
| 263 | [memory/MemoryFileSelector.tsx](./src/components/memory/MemoryFileSelector.tsx) | `.tsx` |
| 264 | [memory/MemoryUpdateNotification.tsx](./src/components/memory/MemoryUpdateNotification.tsx) | `.tsx` |
| 265 | [messageActions.tsx](./src/components/messageActions.tsx) | `.tsx` |
| 266 | [messages/AdvisorMessage.tsx](./src/components/messages/AdvisorMessage.tsx) | `.tsx` |
| 267 | [messages/AssistantRedactedThinkingMessage.tsx](./src/components/messages/AssistantRedactedThinkingMessage.tsx) | `.tsx` |
| 268 | [messages/AssistantTextMessage.tsx](./src/components/messages/AssistantTextMessage.tsx) | `.tsx` |
| 269 | [messages/AssistantThinkingMessage.tsx](./src/components/messages/AssistantThinkingMessage.tsx) | `.tsx` |
| 270 | [messages/AssistantToolUseMessage.tsx](./src/components/messages/AssistantToolUseMessage.tsx) | `.tsx` |
| 271 | [messages/AttachmentMessage.tsx](./src/components/messages/AttachmentMessage.tsx) | `.tsx` |
| 272 | [messages/CollapsedReadSearchContent.tsx](./src/components/messages/CollapsedReadSearchContent.tsx) | `.tsx` |
| 273 | [messages/CompactBoundaryMessage.tsx](./src/components/messages/CompactBoundaryMessage.tsx) | `.tsx` |
| 274 | [messages/GroupedToolUseContent.tsx](./src/components/messages/GroupedToolUseContent.tsx) | `.tsx` |
| 275 | [messages/HighlightedThinkingText.tsx](./src/components/messages/HighlightedThinkingText.tsx) | `.tsx` |
| 276 | [messages/HookProgressMessage.tsx](./src/components/messages/HookProgressMessage.tsx) | `.tsx` |
| 277 | [messages/PlanApprovalMessage.tsx](./src/components/messages/PlanApprovalMessage.tsx) | `.tsx` |
| 278 | [messages/RateLimitMessage.tsx](./src/components/messages/RateLimitMessage.tsx) | `.tsx` |
| 279 | [messages/ShutdownMessage.tsx](./src/components/messages/ShutdownMessage.tsx) | `.tsx` |
| 280 | [messages/SystemAPIErrorMessage.tsx](./src/components/messages/SystemAPIErrorMessage.tsx) | `.tsx` |
| 281 | [messages/SystemTextMessage.tsx](./src/components/messages/SystemTextMessage.tsx) | `.tsx` |
| 282 | [messages/TaskAssignmentMessage.tsx](./src/components/messages/TaskAssignmentMessage.tsx) | `.tsx` |
| 283 | [messages/UserAgentNotificationMessage.tsx](./src/components/messages/UserAgentNotificationMessage.tsx) | `.tsx` |
| 284 | [messages/UserBashInputMessage.tsx](./src/components/messages/UserBashInputMessage.tsx) | `.tsx` |
| 285 | [messages/UserBashOutputMessage.tsx](./src/components/messages/UserBashOutputMessage.tsx) | `.tsx` |
| 286 | [messages/UserChannelMessage.tsx](./src/components/messages/UserChannelMessage.tsx) | `.tsx` |
| 287 | [messages/UserCommandMessage.tsx](./src/components/messages/UserCommandMessage.tsx) | `.tsx` |
| 288 | [messages/UserImageMessage.tsx](./src/components/messages/UserImageMessage.tsx) | `.tsx` |
| 289 | [messages/UserLocalCommandOutputMessage.tsx](./src/components/messages/UserLocalCommandOutputMessage.tsx) | `.tsx` |
| 290 | [messages/UserMemoryInputMessage.tsx](./src/components/messages/UserMemoryInputMessage.tsx) | `.tsx` |
| 291 | [messages/UserPlanMessage.tsx](./src/components/messages/UserPlanMessage.tsx) | `.tsx` |
| 292 | [messages/UserPromptMessage.tsx](./src/components/messages/UserPromptMessage.tsx) | `.tsx` |
| 293 | [messages/UserResourceUpdateMessage.tsx](./src/components/messages/UserResourceUpdateMessage.tsx) | `.tsx` |
| 294 | [messages/UserTeammateMessage.tsx](./src/components/messages/UserTeammateMessage.tsx) | `.tsx` |
| 295 | [messages/UserTextMessage.tsx](./src/components/messages/UserTextMessage.tsx) | `.tsx` |
| 296 | [messages/UserToolResultMessage/RejectedPlanMessage.tsx](./src/components/messages/UserToolResultMessage/RejectedPlanMessage.tsx) | `.tsx` |
| 297 | [messages/UserToolResultMessage/RejectedToolUseMessage.tsx](./src/components/messages/UserToolResultMessage/RejectedToolUseMessage.tsx) | `.tsx` |
| 298 | [messages/UserToolResultMessage/UserToolCanceledMessage.tsx](./src/components/messages/UserToolResultMessage/UserToolCanceledMessage.tsx) | `.tsx` |
| 299 | [messages/UserToolResultMessage/UserToolErrorMessage.tsx](./src/components/messages/UserToolResultMessage/UserToolErrorMessage.tsx) | `.tsx` |
| 300 | [messages/UserToolResultMessage/UserToolRejectMessage.tsx](./src/components/messages/UserToolResultMessage/UserToolRejectMessage.tsx) | `.tsx` |
| 301 | [messages/UserToolResultMessage/UserToolResultMessage.tsx](./src/components/messages/UserToolResultMessage/UserToolResultMessage.tsx) | `.tsx` |
| 302 | [messages/UserToolResultMessage/UserToolSuccessMessage.tsx](./src/components/messages/UserToolResultMessage/UserToolSuccessMessage.tsx) | `.tsx` |
| 303 | [messages/UserToolResultMessage/utils.tsx](./src/components/messages/UserToolResultMessage/utils.tsx) | `.tsx` |
| 304 | [messages/nullRenderingAttachments.ts](./src/components/messages/nullRenderingAttachments.ts) | `.ts` |
| 305 | [messages/teamMemCollapsed.tsx](./src/components/messages/teamMemCollapsed.tsx) | `.tsx` |
| 306 | [messages/teamMemSaved.ts](./src/components/messages/teamMemSaved.ts) | `.ts` |
| 307 | [permissions/AskUserQuestionPermissionRequest/AskUserQuestionPermissionRequest.tsx](./src/components/permissions/AskUserQuestionPermissionRequest/AskUserQuestionPermissionRequest.tsx) | `.tsx` |
| 308 | [permissions/AskUserQuestionPermissionRequest/PreviewBox.tsx](./src/components/permissions/AskUserQuestionPermissionRequest/PreviewBox.tsx) | `.tsx` |
| 309 | [permissions/AskUserQuestionPermissionRequest/PreviewQuestionView.tsx](./src/components/permissions/AskUserQuestionPermissionRequest/PreviewQuestionView.tsx) | `.tsx` |
| 310 | [permissions/AskUserQuestionPermissionRequest/QuestionNavigationBar.tsx](./src/components/permissions/AskUserQuestionPermissionRequest/QuestionNavigationBar.tsx) | `.tsx` |
| 311 | [permissions/AskUserQuestionPermissionRequest/QuestionView.tsx](./src/components/permissions/AskUserQuestionPermissionRequest/QuestionView.tsx) | `.tsx` |
| 312 | [permissions/AskUserQuestionPermissionRequest/SubmitQuestionsView.tsx](./src/components/permissions/AskUserQuestionPermissionRequest/SubmitQuestionsView.tsx) | `.tsx` |
| 313 | [permissions/AskUserQuestionPermissionRequest/use-multiple-choice-state.ts](./src/components/permissions/AskUserQuestionPermissionRequest/use-multiple-choice-state.ts) | `.ts` |
| 314 | [permissions/BashPermissionRequest/BashPermissionRequest.tsx](./src/components/permissions/BashPermissionRequest/BashPermissionRequest.tsx) | `.tsx` |
| 315 | [permissions/BashPermissionRequest/bashToolUseOptions.tsx](./src/components/permissions/BashPermissionRequest/bashToolUseOptions.tsx) | `.tsx` |
| 316 | [permissions/ComputerUseApproval/ComputerUseApproval.tsx](./src/components/permissions/ComputerUseApproval/ComputerUseApproval.tsx) | `.tsx` |
| 317 | [permissions/EnterPlanModePermissionRequest/EnterPlanModePermissionRequest.tsx](./src/components/permissions/EnterPlanModePermissionRequest/EnterPlanModePermissionRequest.tsx) | `.tsx` |
| 318 | [permissions/ExitPlanModePermissionRequest/ExitPlanModePermissionRequest.tsx](./src/components/permissions/ExitPlanModePermissionRequest/ExitPlanModePermissionRequest.tsx) | `.tsx` |
| 319 | [permissions/FallbackPermissionRequest.tsx](./src/components/permissions/FallbackPermissionRequest.tsx) | `.tsx` |
| 320 | [permissions/FileEditPermissionRequest/FileEditPermissionRequest.tsx](./src/components/permissions/FileEditPermissionRequest/FileEditPermissionRequest.tsx) | `.tsx` |
| 321 | [permissions/FilePermissionDialog/FilePermissionDialog.tsx](./src/components/permissions/FilePermissionDialog/FilePermissionDialog.tsx) | `.tsx` |
| 322 | [permissions/FilePermissionDialog/ideDiffConfig.ts](./src/components/permissions/FilePermissionDialog/ideDiffConfig.ts) | `.ts` |
| 323 | [permissions/FilePermissionDialog/permissionOptions.tsx](./src/components/permissions/FilePermissionDialog/permissionOptions.tsx) | `.tsx` |
| 324 | [permissions/FilePermissionDialog/useFilePermissionDialog.ts](./src/components/permissions/FilePermissionDialog/useFilePermissionDialog.ts) | `.ts` |
| 325 | [permissions/FilePermissionDialog/usePermissionHandler.ts](./src/components/permissions/FilePermissionDialog/usePermissionHandler.ts) | `.ts` |
| 326 | [permissions/FileWritePermissionRequest/FileWritePermissionRequest.tsx](./src/components/permissions/FileWritePermissionRequest/FileWritePermissionRequest.tsx) | `.tsx` |
| 327 | [permissions/FileWritePermissionRequest/FileWriteToolDiff.tsx](./src/components/permissions/FileWritePermissionRequest/FileWriteToolDiff.tsx) | `.tsx` |
| 328 | [permissions/FilesystemPermissionRequest/FilesystemPermissionRequest.tsx](./src/components/permissions/FilesystemPermissionRequest/FilesystemPermissionRequest.tsx) | `.tsx` |
| 329 | [permissions/NotebookEditPermissionRequest/NotebookEditPermissionRequest.tsx](./src/components/permissions/NotebookEditPermissionRequest/NotebookEditPermissionRequest.tsx) | `.tsx` |
| 330 | [permissions/NotebookEditPermissionRequest/NotebookEditToolDiff.tsx](./src/components/permissions/NotebookEditPermissionRequest/NotebookEditToolDiff.tsx) | `.tsx` |
| 331 | [permissions/PermissionDecisionDebugInfo.tsx](./src/components/permissions/PermissionDecisionDebugInfo.tsx) | `.tsx` |
| 332 | [permissions/PermissionDialog.tsx](./src/components/permissions/PermissionDialog.tsx) | `.tsx` |
| 333 | [permissions/PermissionExplanation.tsx](./src/components/permissions/PermissionExplanation.tsx) | `.tsx` |
| 334 | [permissions/PermissionPrompt.tsx](./src/components/permissions/PermissionPrompt.tsx) | `.tsx` |
| 335 | [permissions/PermissionRequest.tsx](./src/components/permissions/PermissionRequest.tsx) | `.tsx` |
| 336 | [permissions/PermissionRequestTitle.tsx](./src/components/permissions/PermissionRequestTitle.tsx) | `.tsx` |
| 337 | [permissions/PermissionRuleExplanation.tsx](./src/components/permissions/PermissionRuleExplanation.tsx) | `.tsx` |
| 338 | [permissions/PowerShellPermissionRequest/PowerShellPermissionRequest.tsx](./src/components/permissions/PowerShellPermissionRequest/PowerShellPermissionRequest.tsx) | `.tsx` |
| 339 | [permissions/PowerShellPermissionRequest/powershellToolUseOptions.tsx](./src/components/permissions/PowerShellPermissionRequest/powershellToolUseOptions.tsx) | `.tsx` |
| 340 | [permissions/SandboxPermissionRequest.tsx](./src/components/permissions/SandboxPermissionRequest.tsx) | `.tsx` |
| 341 | [permissions/SedEditPermissionRequest/SedEditPermissionRequest.tsx](./src/components/permissions/SedEditPermissionRequest/SedEditPermissionRequest.tsx) | `.tsx` |
| 342 | [permissions/SkillPermissionRequest/SkillPermissionRequest.tsx](./src/components/permissions/SkillPermissionRequest/SkillPermissionRequest.tsx) | `.tsx` |
| 343 | [permissions/WebFetchPermissionRequest/WebFetchPermissionRequest.tsx](./src/components/permissions/WebFetchPermissionRequest/WebFetchPermissionRequest.tsx) | `.tsx` |
| 344 | [permissions/WorkerBadge.tsx](./src/components/permissions/WorkerBadge.tsx) | `.tsx` |
| 345 | [permissions/WorkerPendingPermission.tsx](./src/components/permissions/WorkerPendingPermission.tsx) | `.tsx` |
| 346 | [permissions/hooks.ts](./src/components/permissions/hooks.ts) | `.ts` |
| 347 | [permissions/rules/AddPermissionRules.tsx](./src/components/permissions/rules/AddPermissionRules.tsx) | `.tsx` |
| 348 | [permissions/rules/AddWorkspaceDirectory.tsx](./src/components/permissions/rules/AddWorkspaceDirectory.tsx) | `.tsx` |
| 349 | [permissions/rules/PermissionRuleDescription.tsx](./src/components/permissions/rules/PermissionRuleDescription.tsx) | `.tsx` |
| 350 | [permissions/rules/PermissionRuleInput.tsx](./src/components/permissions/rules/PermissionRuleInput.tsx) | `.tsx` |
| 351 | [permissions/rules/PermissionRuleList.tsx](./src/components/permissions/rules/PermissionRuleList.tsx) | `.tsx` |
| 352 | [permissions/rules/RecentDenialsTab.tsx](./src/components/permissions/rules/RecentDenialsTab.tsx) | `.tsx` |
| 353 | [permissions/rules/RemoveWorkspaceDirectory.tsx](./src/components/permissions/rules/RemoveWorkspaceDirectory.tsx) | `.tsx` |
| 354 | [permissions/rules/WorkspaceTab.tsx](./src/components/permissions/rules/WorkspaceTab.tsx) | `.tsx` |
| 355 | [permissions/shellPermissionHelpers.tsx](./src/components/permissions/shellPermissionHelpers.tsx) | `.tsx` |
| 356 | [permissions/useShellPermissionFeedback.ts](./src/components/permissions/useShellPermissionFeedback.ts) | `.ts` |
| 357 | [permissions/utils.ts](./src/components/permissions/utils.ts) | `.ts` |
| 358 | [sandbox/SandboxConfigTab.tsx](./src/components/sandbox/SandboxConfigTab.tsx) | `.tsx` |
| 359 | [sandbox/SandboxDependenciesTab.tsx](./src/components/sandbox/SandboxDependenciesTab.tsx) | `.tsx` |
| 360 | [sandbox/SandboxDoctorSection.tsx](./src/components/sandbox/SandboxDoctorSection.tsx) | `.tsx` |
| 361 | [sandbox/SandboxOverridesTab.tsx](./src/components/sandbox/SandboxOverridesTab.tsx) | `.tsx` |
| 362 | [sandbox/SandboxSettings.tsx](./src/components/sandbox/SandboxSettings.tsx) | `.tsx` |
| 363 | [shell/ExpandShellOutputContext.tsx](./src/components/shell/ExpandShellOutputContext.tsx) | `.tsx` |
| 364 | [shell/OutputLine.tsx](./src/components/shell/OutputLine.tsx) | `.tsx` |
| 365 | [shell/ShellProgressMessage.tsx](./src/components/shell/ShellProgressMessage.tsx) | `.tsx` |
| 366 | [shell/ShellTimeDisplay.tsx](./src/components/shell/ShellTimeDisplay.tsx) | `.tsx` |
| 367 | [skills/SkillsMenu.tsx](./src/components/skills/SkillsMenu.tsx) | `.tsx` |
| 368 | [tasks/AsyncAgentDetailDialog.tsx](./src/components/tasks/AsyncAgentDetailDialog.tsx) | `.tsx` |
| 369 | [tasks/BackgroundTask.tsx](./src/components/tasks/BackgroundTask.tsx) | `.tsx` |
| 370 | [tasks/BackgroundTaskStatus.tsx](./src/components/tasks/BackgroundTaskStatus.tsx) | `.tsx` |
| 371 | [tasks/BackgroundTasksDialog.tsx](./src/components/tasks/BackgroundTasksDialog.tsx) | `.tsx` |
| 372 | [tasks/DreamDetailDialog.tsx](./src/components/tasks/DreamDetailDialog.tsx) | `.tsx` |
| 373 | [tasks/InProcessTeammateDetailDialog.tsx](./src/components/tasks/InProcessTeammateDetailDialog.tsx) | `.tsx` |
| 374 | [tasks/RemoteSessionDetailDialog.tsx](./src/components/tasks/RemoteSessionDetailDialog.tsx) | `.tsx` |
| 375 | [tasks/RemoteSessionProgress.tsx](./src/components/tasks/RemoteSessionProgress.tsx) | `.tsx` |
| 376 | [tasks/ShellDetailDialog.tsx](./src/components/tasks/ShellDetailDialog.tsx) | `.tsx` |
| 377 | [tasks/ShellProgress.tsx](./src/components/tasks/ShellProgress.tsx) | `.tsx` |
| 378 | [tasks/renderToolActivity.tsx](./src/components/tasks/renderToolActivity.tsx) | `.tsx` |
| 379 | [tasks/taskStatusUtils.tsx](./src/components/tasks/taskStatusUtils.tsx) | `.tsx` |
| 380 | [teams/TeamStatus.tsx](./src/components/teams/TeamStatus.tsx) | `.tsx` |
| 381 | [teams/TeamsDialog.tsx](./src/components/teams/TeamsDialog.tsx) | `.tsx` |
| 382 | [ui/OrderedList.tsx](./src/components/ui/OrderedList.tsx) | `.tsx` |
| 383 | [ui/OrderedListItem.tsx](./src/components/ui/OrderedListItem.tsx) | `.tsx` |
| 384 | [ui/TreeSelect.tsx](./src/components/ui/TreeSelect.tsx) | `.tsx` |
| 385 | [wizard/WizardDialogLayout.tsx](./src/components/wizard/WizardDialogLayout.tsx) | `.tsx` |
| 386 | [wizard/WizardNavigationFooter.tsx](./src/components/wizard/WizardNavigationFooter.tsx) | `.tsx` |
| 387 | [wizard/WizardProvider.tsx](./src/components/wizard/WizardProvider.tsx) | `.tsx` |
| 388 | [wizard/index.ts](./src/components/wizard/index.ts) | `.ts` |
| 389 | [wizard/useWizard.ts](./src/components/wizard/useWizard.ts) | `.ts` |

### constants/

| # | File | Type |
|:--|:-----|:-----|
| 1 | [apiLimits.ts](./src/constants/apiLimits.ts) | `.ts` |
| 2 | [betas.ts](./src/constants/betas.ts) | `.ts` |
| 3 | [common.ts](./src/constants/common.ts) | `.ts` |
| 4 | [cyberRiskInstruction.ts](./src/constants/cyberRiskInstruction.ts) | `.ts` |
| 5 | [errorIds.ts](./src/constants/errorIds.ts) | `.ts` |
| 6 | [figures.ts](./src/constants/figures.ts) | `.ts` |
| 7 | [files.ts](./src/constants/files.ts) | `.ts` |
| 8 | [github-app.ts](./src/constants/github-app.ts) | `.ts` |
| 9 | [keys.ts](./src/constants/keys.ts) | `.ts` |
| 10 | [messages.ts](./src/constants/messages.ts) | `.ts` |
| 11 | [oauth.ts](./src/constants/oauth.ts) | `.ts` |
| 12 | [outputStyles.ts](./src/constants/outputStyles.ts) | `.ts` |
| 13 | [product.ts](./src/constants/product.ts) | `.ts` |
| 14 | [prompts.ts](./src/constants/prompts.ts) | `.ts` |
| 15 | [spinnerVerbs.ts](./src/constants/spinnerVerbs.ts) | `.ts` |
| 16 | [system.ts](./src/constants/system.ts) | `.ts` |
| 17 | [systemPromptSections.ts](./src/constants/systemPromptSections.ts) | `.ts` |
| 18 | [toolLimits.ts](./src/constants/toolLimits.ts) | `.ts` |
| 19 | [tools.ts](./src/constants/tools.ts) | `.ts` |
| 20 | [turnCompletionVerbs.ts](./src/constants/turnCompletionVerbs.ts) | `.ts` |
| 21 | [xml.ts](./src/constants/xml.ts) | `.ts` |

### context/

| # | File | Type |
|:--|:-----|:-----|
| 1 | [QueuedMessageContext.tsx](./src/context/QueuedMessageContext.tsx) | `.tsx` |
| 2 | [fpsMetrics.tsx](./src/context/fpsMetrics.tsx) | `.tsx` |
| 3 | [mailbox.tsx](./src/context/mailbox.tsx) | `.tsx` |
| 4 | [modalContext.tsx](./src/context/modalContext.tsx) | `.tsx` |
| 5 | [notifications.tsx](./src/context/notifications.tsx) | `.tsx` |
| 6 | [overlayContext.tsx](./src/context/overlayContext.tsx) | `.tsx` |
| 7 | [promptOverlayContext.tsx](./src/context/promptOverlayContext.tsx) | `.tsx` |
| 8 | [stats.tsx](./src/context/stats.tsx) | `.tsx` |
| 9 | [voice.tsx](./src/context/voice.tsx) | `.tsx` |

### coordinator/

| # | File | Type |
|:--|:-----|:-----|
| 1 | [coordinatorMode.ts](./src/coordinator/coordinatorMode.ts) | `.ts` |

### entrypoints/

| # | File | Type |
|:--|:-----|:-----|
| 1 | [agentSdkTypes.ts](./src/entrypoints/agentSdkTypes.ts) | `.ts` |
| 2 | [cli.tsx](./src/entrypoints/cli.tsx) | `.tsx` |
| 3 | [init.ts](./src/entrypoints/init.ts) | `.ts` |
| 4 | [mcp.ts](./src/entrypoints/mcp.ts) | `.ts` |
| 5 | [sandboxTypes.ts](./src/entrypoints/sandboxTypes.ts) | `.ts` |
| 6 | [sdk/controlSchemas.ts](./src/entrypoints/sdk/controlSchemas.ts) | `.ts` |
| 7 | [sdk/coreSchemas.ts](./src/entrypoints/sdk/coreSchemas.ts) | `.ts` |
| 8 | [sdk/coreTypes.ts](./src/entrypoints/sdk/coreTypes.ts) | `.ts` |

### hooks/

| # | File | Type |
|:--|:-----|:-----|
| 1 | [fileSuggestions.ts](./src/hooks/fileSuggestions.ts) | `.ts` |
| 2 | [notifs/useAutoModeUnavailableNotification.ts](./src/hooks/notifs/useAutoModeUnavailableNotification.ts) | `.ts` |
| 3 | [notifs/useCanSwitchToExistingSubscription.tsx](./src/hooks/notifs/useCanSwitchToExistingSubscription.tsx) | `.tsx` |
| 4 | [notifs/useDeprecationWarningNotification.tsx](./src/hooks/notifs/useDeprecationWarningNotification.tsx) | `.tsx` |
| 5 | [notifs/useFastModeNotification.tsx](./src/hooks/notifs/useFastModeNotification.tsx) | `.tsx` |
| 6 | [notifs/useIDEStatusIndicator.tsx](./src/hooks/notifs/useIDEStatusIndicator.tsx) | `.tsx` |
| 7 | [notifs/useInstallMessages.tsx](./src/hooks/notifs/useInstallMessages.tsx) | `.tsx` |
| 8 | [notifs/useLspInitializationNotification.tsx](./src/hooks/notifs/useLspInitializationNotification.tsx) | `.tsx` |
| 9 | [notifs/useMcpConnectivityStatus.tsx](./src/hooks/notifs/useMcpConnectivityStatus.tsx) | `.tsx` |
| 10 | [notifs/useModelMigrationNotifications.tsx](./src/hooks/notifs/useModelMigrationNotifications.tsx) | `.tsx` |
| 11 | [notifs/useNpmDeprecationNotification.tsx](./src/hooks/notifs/useNpmDeprecationNotification.tsx) | `.tsx` |
| 12 | [notifs/usePluginAutoupdateNotification.tsx](./src/hooks/notifs/usePluginAutoupdateNotification.tsx) | `.tsx` |
| 13 | [notifs/usePluginInstallationStatus.tsx](./src/hooks/notifs/usePluginInstallationStatus.tsx) | `.tsx` |
| 14 | [notifs/useRateLimitWarningNotification.tsx](./src/hooks/notifs/useRateLimitWarningNotification.tsx) | `.tsx` |
| 15 | [notifs/useSettingsErrors.tsx](./src/hooks/notifs/useSettingsErrors.tsx) | `.tsx` |
| 16 | [notifs/useStartupNotification.ts](./src/hooks/notifs/useStartupNotification.ts) | `.ts` |
| 17 | [notifs/useTeammateShutdownNotification.ts](./src/hooks/notifs/useTeammateShutdownNotification.ts) | `.ts` |
| 18 | [renderPlaceholder.ts](./src/hooks/renderPlaceholder.ts) | `.ts` |
| 19 | [toolPermission/PermissionContext.ts](./src/hooks/toolPermission/PermissionContext.ts) | `.ts` |
| 20 | [toolPermission/handlers/coordinatorHandler.ts](./src/hooks/toolPermission/handlers/coordinatorHandler.ts) | `.ts` |
| 21 | [toolPermission/handlers/interactiveHandler.ts](./src/hooks/toolPermission/handlers/interactiveHandler.ts) | `.ts` |
| 22 | [toolPermission/handlers/swarmWorkerHandler.ts](./src/hooks/toolPermission/handlers/swarmWorkerHandler.ts) | `.ts` |
| 23 | [toolPermission/permissionLogging.ts](./src/hooks/toolPermission/permissionLogging.ts) | `.ts` |
| 24 | [unifiedSuggestions.ts](./src/hooks/unifiedSuggestions.ts) | `.ts` |
| 25 | [useAfterFirstRender.ts](./src/hooks/useAfterFirstRender.ts) | `.ts` |
| 26 | [useApiKeyVerification.ts](./src/hooks/useApiKeyVerification.ts) | `.ts` |
| 27 | [useArrowKeyHistory.tsx](./src/hooks/useArrowKeyHistory.tsx) | `.tsx` |
| 28 | [useAssistantHistory.ts](./src/hooks/useAssistantHistory.ts) | `.ts` |
| 29 | [useAwaySummary.ts](./src/hooks/useAwaySummary.ts) | `.ts` |
| 30 | [useBackgroundTaskNavigation.ts](./src/hooks/useBackgroundTaskNavigation.ts) | `.ts` |
| 31 | [useBlink.ts](./src/hooks/useBlink.ts) | `.ts` |
| 32 | [useCanUseTool.tsx](./src/hooks/useCanUseTool.tsx) | `.tsx` |
| 33 | [useCancelRequest.ts](./src/hooks/useCancelRequest.ts) | `.ts` |
| 34 | [useChromeExtensionNotification.tsx](./src/hooks/useChromeExtensionNotification.tsx) | `.tsx` |
| 35 | [useClaudeCodeHintRecommendation.tsx](./src/hooks/useClaudeCodeHintRecommendation.tsx) | `.tsx` |
| 36 | [useClipboardImageHint.ts](./src/hooks/useClipboardImageHint.ts) | `.ts` |
| 37 | [useCommandKeybindings.tsx](./src/hooks/useCommandKeybindings.tsx) | `.tsx` |
| 38 | [useCommandQueue.ts](./src/hooks/useCommandQueue.ts) | `.ts` |
| 39 | [useCopyOnSelect.ts](./src/hooks/useCopyOnSelect.ts) | `.ts` |
| 40 | [useDeferredHookMessages.ts](./src/hooks/useDeferredHookMessages.ts) | `.ts` |
| 41 | [useDiffData.ts](./src/hooks/useDiffData.ts) | `.ts` |
| 42 | [useDiffInIDE.ts](./src/hooks/useDiffInIDE.ts) | `.ts` |
| 43 | [useDirectConnect.ts](./src/hooks/useDirectConnect.ts) | `.ts` |
| 44 | [useDoublePress.ts](./src/hooks/useDoublePress.ts) | `.ts` |
| 45 | [useDynamicConfig.ts](./src/hooks/useDynamicConfig.ts) | `.ts` |
| 46 | [useElapsedTime.ts](./src/hooks/useElapsedTime.ts) | `.ts` |
| 47 | [useExitOnCtrlCD.ts](./src/hooks/useExitOnCtrlCD.ts) | `.ts` |
| 48 | [useExitOnCtrlCDWithKeybindings.ts](./src/hooks/useExitOnCtrlCDWithKeybindings.ts) | `.ts` |
| 49 | [useFileHistorySnapshotInit.ts](./src/hooks/useFileHistorySnapshotInit.ts) | `.ts` |
| 50 | [useGlobalKeybindings.tsx](./src/hooks/useGlobalKeybindings.tsx) | `.tsx` |
| 51 | [useHistorySearch.ts](./src/hooks/useHistorySearch.ts) | `.ts` |
| 52 | [useIDEIntegration.tsx](./src/hooks/useIDEIntegration.tsx) | `.tsx` |
| 53 | [useIdeAtMentioned.ts](./src/hooks/useIdeAtMentioned.ts) | `.ts` |
| 54 | [useIdeConnectionStatus.ts](./src/hooks/useIdeConnectionStatus.ts) | `.ts` |
| 55 | [useIdeLogging.ts](./src/hooks/useIdeLogging.ts) | `.ts` |
| 56 | [useIdeSelection.ts](./src/hooks/useIdeSelection.ts) | `.ts` |
| 57 | [useInboxPoller.ts](./src/hooks/useInboxPoller.ts) | `.ts` |
| 58 | [useInputBuffer.ts](./src/hooks/useInputBuffer.ts) | `.ts` |
| 59 | [useIssueFlagBanner.ts](./src/hooks/useIssueFlagBanner.ts) | `.ts` |
| 60 | [useLogMessages.ts](./src/hooks/useLogMessages.ts) | `.ts` |
| 61 | [useLspPluginRecommendation.tsx](./src/hooks/useLspPluginRecommendation.tsx) | `.tsx` |
| 62 | [useMailboxBridge.ts](./src/hooks/useMailboxBridge.ts) | `.ts` |
| 63 | [useMainLoopModel.ts](./src/hooks/useMainLoopModel.ts) | `.ts` |
| 64 | [useManagePlugins.ts](./src/hooks/useManagePlugins.ts) | `.ts` |
| 65 | [useMemoryUsage.ts](./src/hooks/useMemoryUsage.ts) | `.ts` |
| 66 | [useMergedClients.ts](./src/hooks/useMergedClients.ts) | `.ts` |
| 67 | [useMergedCommands.ts](./src/hooks/useMergedCommands.ts) | `.ts` |
| 68 | [useMergedTools.ts](./src/hooks/useMergedTools.ts) | `.ts` |
| 69 | [useMinDisplayTime.ts](./src/hooks/useMinDisplayTime.ts) | `.ts` |
| 70 | [useNotifyAfterTimeout.ts](./src/hooks/useNotifyAfterTimeout.ts) | `.ts` |
| 71 | [useOfficialMarketplaceNotification.tsx](./src/hooks/useOfficialMarketplaceNotification.tsx) | `.tsx` |
| 72 | [usePasteHandler.ts](./src/hooks/usePasteHandler.ts) | `.ts` |
| 73 | [usePluginRecommendationBase.tsx](./src/hooks/usePluginRecommendationBase.tsx) | `.tsx` |
| 74 | [usePrStatus.ts](./src/hooks/usePrStatus.ts) | `.ts` |
| 75 | [usePromptSuggestion.ts](./src/hooks/usePromptSuggestion.ts) | `.ts` |
| 76 | [usePromptsFromClaudeInChrome.tsx](./src/hooks/usePromptsFromClaudeInChrome.tsx) | `.tsx` |
| 77 | [useQueueProcessor.ts](./src/hooks/useQueueProcessor.ts) | `.ts` |
| 78 | [useRemoteSession.ts](./src/hooks/useRemoteSession.ts) | `.ts` |
| 79 | [useReplBridge.tsx](./src/hooks/useReplBridge.tsx) | `.tsx` |
| 80 | [useSSHSession.ts](./src/hooks/useSSHSession.ts) | `.ts` |
| 81 | [useScheduledTasks.ts](./src/hooks/useScheduledTasks.ts) | `.ts` |
| 82 | [useSearchInput.ts](./src/hooks/useSearchInput.ts) | `.ts` |
| 83 | [useSessionBackgrounding.ts](./src/hooks/useSessionBackgrounding.ts) | `.ts` |
| 84 | [useSettings.ts](./src/hooks/useSettings.ts) | `.ts` |
| 85 | [useSettingsChange.ts](./src/hooks/useSettingsChange.ts) | `.ts` |
| 86 | [useSkillImprovementSurvey.ts](./src/hooks/useSkillImprovementSurvey.ts) | `.ts` |
| 87 | [useSkillsChange.ts](./src/hooks/useSkillsChange.ts) | `.ts` |
| 88 | [useSwarmInitialization.ts](./src/hooks/useSwarmInitialization.ts) | `.ts` |
| 89 | [useSwarmPermissionPoller.ts](./src/hooks/useSwarmPermissionPoller.ts) | `.ts` |
| 90 | [useTaskListWatcher.ts](./src/hooks/useTaskListWatcher.ts) | `.ts` |
| 91 | [useTasksV2.ts](./src/hooks/useTasksV2.ts) | `.ts` |
| 92 | [useTeammateViewAutoExit.ts](./src/hooks/useTeammateViewAutoExit.ts) | `.ts` |
| 93 | [useTeleportResume.tsx](./src/hooks/useTeleportResume.tsx) | `.tsx` |
| 94 | [useTerminalSize.ts](./src/hooks/useTerminalSize.ts) | `.ts` |
| 95 | [useTextInput.ts](./src/hooks/useTextInput.ts) | `.ts` |
| 96 | [useTimeout.ts](./src/hooks/useTimeout.ts) | `.ts` |
| 97 | [useTurnDiffs.ts](./src/hooks/useTurnDiffs.ts) | `.ts` |
| 98 | [useTypeahead.tsx](./src/hooks/useTypeahead.tsx) | `.tsx` |
| 99 | [useUpdateNotification.ts](./src/hooks/useUpdateNotification.ts) | `.ts` |
| 100 | [useVimInput.ts](./src/hooks/useVimInput.ts) | `.ts` |
| 101 | [useVirtualScroll.ts](./src/hooks/useVirtualScroll.ts) | `.ts` |
| 102 | [useVoice.ts](./src/hooks/useVoice.ts) | `.ts` |
| 103 | [useVoiceEnabled.ts](./src/hooks/useVoiceEnabled.ts) | `.ts` |
| 104 | [useVoiceIntegration.tsx](./src/hooks/useVoiceIntegration.tsx) | `.tsx` |

### ink/

| # | File | Type |
|:--|:-----|:-----|
| 1 | [Ansi.tsx](./src/ink/Ansi.tsx) | `.tsx` |
| 2 | [bidi.ts](./src/ink/bidi.ts) | `.ts` |
| 3 | [clearTerminal.ts](./src/ink/clearTerminal.ts) | `.ts` |
| 4 | [colorize.ts](./src/ink/colorize.ts) | `.ts` |
| 5 | [components/AlternateScreen.tsx](./src/ink/components/AlternateScreen.tsx) | `.tsx` |
| 6 | [components/App.tsx](./src/ink/components/App.tsx) | `.tsx` |
| 7 | [components/AppContext.ts](./src/ink/components/AppContext.ts) | `.ts` |
| 8 | [components/Box.tsx](./src/ink/components/Box.tsx) | `.tsx` |
| 9 | [components/Button.tsx](./src/ink/components/Button.tsx) | `.tsx` |
| 10 | [components/ClockContext.tsx](./src/ink/components/ClockContext.tsx) | `.tsx` |
| 11 | [components/CursorDeclarationContext.ts](./src/ink/components/CursorDeclarationContext.ts) | `.ts` |
| 12 | [components/ErrorOverview.tsx](./src/ink/components/ErrorOverview.tsx) | `.tsx` |
| 13 | [components/Link.tsx](./src/ink/components/Link.tsx) | `.tsx` |
| 14 | [components/Newline.tsx](./src/ink/components/Newline.tsx) | `.tsx` |
| 15 | [components/NoSelect.tsx](./src/ink/components/NoSelect.tsx) | `.tsx` |
| 16 | [components/RawAnsi.tsx](./src/ink/components/RawAnsi.tsx) | `.tsx` |
| 17 | [components/ScrollBox.tsx](./src/ink/components/ScrollBox.tsx) | `.tsx` |
| 18 | [components/Spacer.tsx](./src/ink/components/Spacer.tsx) | `.tsx` |
| 19 | [components/StdinContext.ts](./src/ink/components/StdinContext.ts) | `.ts` |
| 20 | [components/TerminalFocusContext.tsx](./src/ink/components/TerminalFocusContext.tsx) | `.tsx` |
| 21 | [components/TerminalSizeContext.tsx](./src/ink/components/TerminalSizeContext.tsx) | `.tsx` |
| 22 | [components/Text.tsx](./src/ink/components/Text.tsx) | `.tsx` |
| 23 | [constants.ts](./src/ink/constants.ts) | `.ts` |
| 24 | [dom.ts](./src/ink/dom.ts) | `.ts` |
| 25 | [events/click-event.ts](./src/ink/events/click-event.ts) | `.ts` |
| 26 | [events/dispatcher.ts](./src/ink/events/dispatcher.ts) | `.ts` |
| 27 | [events/emitter.ts](./src/ink/events/emitter.ts) | `.ts` |
| 28 | [events/event-handlers.ts](./src/ink/events/event-handlers.ts) | `.ts` |
| 29 | [events/event.ts](./src/ink/events/event.ts) | `.ts` |
| 30 | [events/focus-event.ts](./src/ink/events/focus-event.ts) | `.ts` |
| 31 | [events/input-event.ts](./src/ink/events/input-event.ts) | `.ts` |
| 32 | [events/keyboard-event.ts](./src/ink/events/keyboard-event.ts) | `.ts` |
| 33 | [events/terminal-event.ts](./src/ink/events/terminal-event.ts) | `.ts` |
| 34 | [events/terminal-focus-event.ts](./src/ink/events/terminal-focus-event.ts) | `.ts` |
| 35 | [focus.ts](./src/ink/focus.ts) | `.ts` |
| 36 | [frame.ts](./src/ink/frame.ts) | `.ts` |
| 37 | [get-max-width.ts](./src/ink/get-max-width.ts) | `.ts` |
| 38 | [hit-test.ts](./src/ink/hit-test.ts) | `.ts` |
| 39 | [hooks/use-animation-frame.ts](./src/ink/hooks/use-animation-frame.ts) | `.ts` |
| 40 | [hooks/use-app.ts](./src/ink/hooks/use-app.ts) | `.ts` |
| 41 | [hooks/use-declared-cursor.ts](./src/ink/hooks/use-declared-cursor.ts) | `.ts` |
| 42 | [hooks/use-input.ts](./src/ink/hooks/use-input.ts) | `.ts` |
| 43 | [hooks/use-interval.ts](./src/ink/hooks/use-interval.ts) | `.ts` |
| 44 | [hooks/use-search-highlight.ts](./src/ink/hooks/use-search-highlight.ts) | `.ts` |
| 45 | [hooks/use-selection.ts](./src/ink/hooks/use-selection.ts) | `.ts` |
| 46 | [hooks/use-stdin.ts](./src/ink/hooks/use-stdin.ts) | `.ts` |
| 47 | [hooks/use-tab-status.ts](./src/ink/hooks/use-tab-status.ts) | `.ts` |
| 48 | [hooks/use-terminal-focus.ts](./src/ink/hooks/use-terminal-focus.ts) | `.ts` |
| 49 | [hooks/use-terminal-title.ts](./src/ink/hooks/use-terminal-title.ts) | `.ts` |
| 50 | [hooks/use-terminal-viewport.ts](./src/ink/hooks/use-terminal-viewport.ts) | `.ts` |
| 51 | [ink.tsx](./src/ink/ink.tsx) | `.tsx` |
| 52 | [instances.ts](./src/ink/instances.ts) | `.ts` |
| 53 | [layout/engine.ts](./src/ink/layout/engine.ts) | `.ts` |
| 54 | [layout/geometry.ts](./src/ink/layout/geometry.ts) | `.ts` |
| 55 | [layout/node.ts](./src/ink/layout/node.ts) | `.ts` |
| 56 | [layout/yoga.ts](./src/ink/layout/yoga.ts) | `.ts` |
| 57 | [line-width-cache.ts](./src/ink/line-width-cache.ts) | `.ts` |
| 58 | [log-update.ts](./src/ink/log-update.ts) | `.ts` |
| 59 | [measure-element.ts](./src/ink/measure-element.ts) | `.ts` |
| 60 | [measure-text.ts](./src/ink/measure-text.ts) | `.ts` |
| 61 | [node-cache.ts](./src/ink/node-cache.ts) | `.ts` |
| 62 | [optimizer.ts](./src/ink/optimizer.ts) | `.ts` |
| 63 | [output.ts](./src/ink/output.ts) | `.ts` |
| 64 | [parse-keypress.ts](./src/ink/parse-keypress.ts) | `.ts` |
| 65 | [reconciler.ts](./src/ink/reconciler.ts) | `.ts` |
| 66 | [render-border.ts](./src/ink/render-border.ts) | `.ts` |
| 67 | [render-node-to-output.ts](./src/ink/render-node-to-output.ts) | `.ts` |
| 68 | [render-to-screen.ts](./src/ink/render-to-screen.ts) | `.ts` |
| 69 | [renderer.ts](./src/ink/renderer.ts) | `.ts` |
| 70 | [root.ts](./src/ink/root.ts) | `.ts` |
| 71 | [screen.ts](./src/ink/screen.ts) | `.ts` |
| 72 | [searchHighlight.ts](./src/ink/searchHighlight.ts) | `.ts` |
| 73 | [selection.ts](./src/ink/selection.ts) | `.ts` |
| 74 | [squash-text-nodes.ts](./src/ink/squash-text-nodes.ts) | `.ts` |
| 75 | [stringWidth.ts](./src/ink/stringWidth.ts) | `.ts` |
| 76 | [styles.ts](./src/ink/styles.ts) | `.ts` |
| 77 | [supports-hyperlinks.ts](./src/ink/supports-hyperlinks.ts) | `.ts` |
| 78 | [tabstops.ts](./src/ink/tabstops.ts) | `.ts` |
| 79 | [terminal-focus-state.ts](./src/ink/terminal-focus-state.ts) | `.ts` |
| 80 | [terminal-querier.ts](./src/ink/terminal-querier.ts) | `.ts` |
| 81 | [terminal.ts](./src/ink/terminal.ts) | `.ts` |
| 82 | [termio.ts](./src/ink/termio.ts) | `.ts` |
| 83 | [termio/ansi.ts](./src/ink/termio/ansi.ts) | `.ts` |
| 84 | [termio/csi.ts](./src/ink/termio/csi.ts) | `.ts` |
| 85 | [termio/dec.ts](./src/ink/termio/dec.ts) | `.ts` |
| 86 | [termio/esc.ts](./src/ink/termio/esc.ts) | `.ts` |
| 87 | [termio/osc.ts](./src/ink/termio/osc.ts) | `.ts` |
| 88 | [termio/parser.ts](./src/ink/termio/parser.ts) | `.ts` |
| 89 | [termio/sgr.ts](./src/ink/termio/sgr.ts) | `.ts` |
| 90 | [termio/tokenize.ts](./src/ink/termio/tokenize.ts) | `.ts` |
| 91 | [termio/types.ts](./src/ink/termio/types.ts) | `.ts` |
| 92 | [useTerminalNotification.ts](./src/ink/useTerminalNotification.ts) | `.ts` |
| 93 | [warn.ts](./src/ink/warn.ts) | `.ts` |
| 94 | [widest-line.ts](./src/ink/widest-line.ts) | `.ts` |
| 95 | [wrap-text.ts](./src/ink/wrap-text.ts) | `.ts` |
| 96 | [wrapAnsi.ts](./src/ink/wrapAnsi.ts) | `.ts` |

### keybindings/

| # | File | Type |
|:--|:-----|:-----|
| 1 | [KeybindingContext.tsx](./src/keybindings/KeybindingContext.tsx) | `.tsx` |
| 2 | [KeybindingProviderSetup.tsx](./src/keybindings/KeybindingProviderSetup.tsx) | `.tsx` |
| 3 | [defaultBindings.ts](./src/keybindings/defaultBindings.ts) | `.ts` |
| 4 | [loadUserBindings.ts](./src/keybindings/loadUserBindings.ts) | `.ts` |
| 5 | [match.ts](./src/keybindings/match.ts) | `.ts` |
| 6 | [parser.ts](./src/keybindings/parser.ts) | `.ts` |
| 7 | [reservedShortcuts.ts](./src/keybindings/reservedShortcuts.ts) | `.ts` |
| 8 | [resolver.ts](./src/keybindings/resolver.ts) | `.ts` |
| 9 | [schema.ts](./src/keybindings/schema.ts) | `.ts` |
| 10 | [shortcutFormat.ts](./src/keybindings/shortcutFormat.ts) | `.ts` |
| 11 | [template.ts](./src/keybindings/template.ts) | `.ts` |
| 12 | [useKeybinding.ts](./src/keybindings/useKeybinding.ts) | `.ts` |
| 13 | [useShortcutDisplay.ts](./src/keybindings/useShortcutDisplay.ts) | `.ts` |
| 14 | [validate.ts](./src/keybindings/validate.ts) | `.ts` |

### memdir/

| # | File | Type |
|:--|:-----|:-----|
| 1 | [findRelevantMemories.ts](./src/memdir/findRelevantMemories.ts) | `.ts` |
| 2 | [memdir.ts](./src/memdir/memdir.ts) | `.ts` |
| 3 | [memoryAge.ts](./src/memdir/memoryAge.ts) | `.ts` |
| 4 | [memoryScan.ts](./src/memdir/memoryScan.ts) | `.ts` |
| 5 | [memoryTypes.ts](./src/memdir/memoryTypes.ts) | `.ts` |
| 6 | [paths.ts](./src/memdir/paths.ts) | `.ts` |
| 7 | [teamMemPaths.ts](./src/memdir/teamMemPaths.ts) | `.ts` |
| 8 | [teamMemPrompts.ts](./src/memdir/teamMemPrompts.ts) | `.ts` |

### migrations/

| # | File | Type |
|:--|:-----|:-----|
| 1 | [migrateAutoUpdatesToSettings.ts](./src/migrations/migrateAutoUpdatesToSettings.ts) | `.ts` |
| 2 | [migrateBypassPermissionsAcceptedToSettings.ts](./src/migrations/migrateBypassPermissionsAcceptedToSettings.ts) | `.ts` |
| 3 | [migrateEnableAllProjectMcpServersToSettings.ts](./src/migrations/migrateEnableAllProjectMcpServersToSettings.ts) | `.ts` |
| 4 | [migrateFennecToOpus.ts](./src/migrations/migrateFennecToOpus.ts) | `.ts` |
| 5 | [migrateLegacyOpusToCurrent.ts](./src/migrations/migrateLegacyOpusToCurrent.ts) | `.ts` |
| 6 | [migrateOpusToOpus1m.ts](./src/migrations/migrateOpusToOpus1m.ts) | `.ts` |
| 7 | [migrateReplBridgeEnabledToRemoteControlAtStartup.ts](./src/migrations/migrateReplBridgeEnabledToRemoteControlAtStartup.ts) | `.ts` |
| 8 | [migrateSonnet1mToSonnet45.ts](./src/migrations/migrateSonnet1mToSonnet45.ts) | `.ts` |
| 9 | [migrateSonnet45ToSonnet46.ts](./src/migrations/migrateSonnet45ToSonnet46.ts) | `.ts` |
| 10 | [resetAutoModeOptInForDefaultOffer.ts](./src/migrations/resetAutoModeOptInForDefaultOffer.ts) | `.ts` |
| 11 | [resetProToOpusDefault.ts](./src/migrations/resetProToOpusDefault.ts) | `.ts` |

### moreright/

| # | File | Type |
|:--|:-----|:-----|
| 1 | [useMoreRight.tsx](./src/moreright/useMoreRight.tsx) | `.tsx` |

### native-ts/

| # | File | Type |
|:--|:-----|:-----|
| 1 | [color-diff/index.ts](./src/native-ts/color-diff/index.ts) | `.ts` |
| 2 | [file-index/index.ts](./src/native-ts/file-index/index.ts) | `.ts` |
| 3 | [yoga-layout/enums.ts](./src/native-ts/yoga-layout/enums.ts) | `.ts` |
| 4 | [yoga-layout/index.ts](./src/native-ts/yoga-layout/index.ts) | `.ts` |

### outputStyles/

| # | File | Type |
|:--|:-----|:-----|
| 1 | [loadOutputStylesDir.ts](./src/outputStyles/loadOutputStylesDir.ts) | `.ts` |

### plugins/

| # | File | Type |
|:--|:-----|:-----|
| 1 | [builtinPlugins.ts](./src/plugins/builtinPlugins.ts) | `.ts` |
| 2 | [bundled/index.ts](./src/plugins/bundled/index.ts) | `.ts` |

### query/

| # | File | Type |
|:--|:-----|:-----|
| 1 | [config.ts](./src/query/config.ts) | `.ts` |
| 2 | [deps.ts](./src/query/deps.ts) | `.ts` |
| 3 | [stopHooks.ts](./src/query/stopHooks.ts) | `.ts` |
| 4 | [tokenBudget.ts](./src/query/tokenBudget.ts) | `.ts` |

### remote/

| # | File | Type |
|:--|:-----|:-----|
| 1 | [RemoteSessionManager.ts](./src/remote/RemoteSessionManager.ts) | `.ts` |
| 2 | [SessionsWebSocket.ts](./src/remote/SessionsWebSocket.ts) | `.ts` |
| 3 | [remotePermissionBridge.ts](./src/remote/remotePermissionBridge.ts) | `.ts` |
| 4 | [sdkMessageAdapter.ts](./src/remote/sdkMessageAdapter.ts) | `.ts` |

### schemas/

| # | File | Type |
|:--|:-----|:-----|
| 1 | [hooks.ts](./src/schemas/hooks.ts) | `.ts` |

### screens/

| # | File | Type |
|:--|:-----|:-----|
| 1 | [Doctor.tsx](./src/screens/Doctor.tsx) | `.tsx` |
| 2 | [REPL.tsx](./src/screens/REPL.tsx) | `.tsx` |
| 3 | [ResumeConversation.tsx](./src/screens/ResumeConversation.tsx) | `.tsx` |

### server/

| # | File | Type |
|:--|:-----|:-----|
| 1 | [createDirectConnectSession.ts](./src/server/createDirectConnectSession.ts) | `.ts` |
| 2 | [directConnectManager.ts](./src/server/directConnectManager.ts) | `.ts` |
| 3 | [types.ts](./src/server/types.ts) | `.ts` |

### services/

| # | File | Type |
|:--|:-----|:-----|
| 1 | [AgentSummary/agentSummary.ts](./src/services/AgentSummary/agentSummary.ts) | `.ts` |
| 2 | [MagicDocs/magicDocs.ts](./src/services/MagicDocs/magicDocs.ts) | `.ts` |
| 3 | [MagicDocs/prompts.ts](./src/services/MagicDocs/prompts.ts) | `.ts` |
| 4 | [PromptSuggestion/promptSuggestion.ts](./src/services/PromptSuggestion/promptSuggestion.ts) | `.ts` |
| 5 | [PromptSuggestion/speculation.ts](./src/services/PromptSuggestion/speculation.ts) | `.ts` |
| 6 | [SessionMemory/prompts.ts](./src/services/SessionMemory/prompts.ts) | `.ts` |
| 7 | [SessionMemory/sessionMemory.ts](./src/services/SessionMemory/sessionMemory.ts) | `.ts` |
| 8 | [SessionMemory/sessionMemoryUtils.ts](./src/services/SessionMemory/sessionMemoryUtils.ts) | `.ts` |
| 9 | [analytics/config.ts](./src/services/analytics/config.ts) | `.ts` |
| 10 | [analytics/datadog.ts](./src/services/analytics/datadog.ts) | `.ts` |
| 11 | [analytics/firstPartyEventLogger.ts](./src/services/analytics/firstPartyEventLogger.ts) | `.ts` |
| 12 | [analytics/firstPartyEventLoggingExporter.ts](./src/services/analytics/firstPartyEventLoggingExporter.ts) | `.ts` |
| 13 | [analytics/growthbook.ts](./src/services/analytics/growthbook.ts) | `.ts` |
| 14 | [analytics/index.ts](./src/services/analytics/index.ts) | `.ts` |
| 15 | [analytics/metadata.ts](./src/services/analytics/metadata.ts) | `.ts` |
| 16 | [analytics/sink.ts](./src/services/analytics/sink.ts) | `.ts` |
| 17 | [analytics/sinkKillswitch.ts](./src/services/analytics/sinkKillswitch.ts) | `.ts` |
| 18 | [api/adminRequests.ts](./src/services/api/adminRequests.ts) | `.ts` |
| 19 | [api/bootstrap.ts](./src/services/api/bootstrap.ts) | `.ts` |
| 20 | [api/claude.ts](./src/services/api/claude.ts) | `.ts` |
| 21 | [api/client.ts](./src/services/api/client.ts) | `.ts` |
| 22 | [api/dumpPrompts.ts](./src/services/api/dumpPrompts.ts) | `.ts` |
| 23 | [api/emptyUsage.ts](./src/services/api/emptyUsage.ts) | `.ts` |
| 24 | [api/errorUtils.ts](./src/services/api/errorUtils.ts) | `.ts` |
| 25 | [api/errors.ts](./src/services/api/errors.ts) | `.ts` |
| 26 | [api/filesApi.ts](./src/services/api/filesApi.ts) | `.ts` |
| 27 | [api/firstTokenDate.ts](./src/services/api/firstTokenDate.ts) | `.ts` |
| 28 | [api/grove.ts](./src/services/api/grove.ts) | `.ts` |
| 29 | [api/logging.ts](./src/services/api/logging.ts) | `.ts` |
| 30 | [api/metricsOptOut.ts](./src/services/api/metricsOptOut.ts) | `.ts` |
| 31 | [api/overageCreditGrant.ts](./src/services/api/overageCreditGrant.ts) | `.ts` |
| 32 | [api/promptCacheBreakDetection.ts](./src/services/api/promptCacheBreakDetection.ts) | `.ts` |
| 33 | [api/referral.ts](./src/services/api/referral.ts) | `.ts` |
| 34 | [api/sessionIngress.ts](./src/services/api/sessionIngress.ts) | `.ts` |
| 35 | [api/ultrareviewQuota.ts](./src/services/api/ultrareviewQuota.ts) | `.ts` |
| 36 | [api/usage.ts](./src/services/api/usage.ts) | `.ts` |
| 37 | [api/withRetry.ts](./src/services/api/withRetry.ts) | `.ts` |
| 38 | [autoDream/autoDream.ts](./src/services/autoDream/autoDream.ts) | `.ts` |
| 39 | [autoDream/config.ts](./src/services/autoDream/config.ts) | `.ts` |
| 40 | [autoDream/consolidationLock.ts](./src/services/autoDream/consolidationLock.ts) | `.ts` |
| 41 | [autoDream/consolidationPrompt.ts](./src/services/autoDream/consolidationPrompt.ts) | `.ts` |
| 42 | [awaySummary.ts](./src/services/awaySummary.ts) | `.ts` |
| 43 | [claudeAiLimits.ts](./src/services/claudeAiLimits.ts) | `.ts` |
| 44 | [claudeAiLimitsHook.ts](./src/services/claudeAiLimitsHook.ts) | `.ts` |
| 45 | [compact/apiMicrocompact.ts](./src/services/compact/apiMicrocompact.ts) | `.ts` |
| 46 | [compact/autoCompact.ts](./src/services/compact/autoCompact.ts) | `.ts` |
| 47 | [compact/compact.ts](./src/services/compact/compact.ts) | `.ts` |
| 48 | [compact/compactWarningHook.ts](./src/services/compact/compactWarningHook.ts) | `.ts` |
| 49 | [compact/compactWarningState.ts](./src/services/compact/compactWarningState.ts) | `.ts` |
| 50 | [compact/grouping.ts](./src/services/compact/grouping.ts) | `.ts` |
| 51 | [compact/microCompact.ts](./src/services/compact/microCompact.ts) | `.ts` |
| 52 | [compact/postCompactCleanup.ts](./src/services/compact/postCompactCleanup.ts) | `.ts` |
| 53 | [compact/prompt.ts](./src/services/compact/prompt.ts) | `.ts` |
| 54 | [compact/sessionMemoryCompact.ts](./src/services/compact/sessionMemoryCompact.ts) | `.ts` |
| 55 | [compact/timeBasedMCConfig.ts](./src/services/compact/timeBasedMCConfig.ts) | `.ts` |
| 56 | [diagnosticTracking.ts](./src/services/diagnosticTracking.ts) | `.ts` |
| 57 | [extractMemories/extractMemories.ts](./src/services/extractMemories/extractMemories.ts) | `.ts` |
| 58 | [extractMemories/prompts.ts](./src/services/extractMemories/prompts.ts) | `.ts` |
| 59 | [internalLogging.ts](./src/services/internalLogging.ts) | `.ts` |
| 60 | [lsp/LSPClient.ts](./src/services/lsp/LSPClient.ts) | `.ts` |
| 61 | [lsp/LSPDiagnosticRegistry.ts](./src/services/lsp/LSPDiagnosticRegistry.ts) | `.ts` |
| 62 | [lsp/LSPServerInstance.ts](./src/services/lsp/LSPServerInstance.ts) | `.ts` |
| 63 | [lsp/LSPServerManager.ts](./src/services/lsp/LSPServerManager.ts) | `.ts` |
| 64 | [lsp/config.ts](./src/services/lsp/config.ts) | `.ts` |
| 65 | [lsp/manager.ts](./src/services/lsp/manager.ts) | `.ts` |
| 66 | [lsp/passiveFeedback.ts](./src/services/lsp/passiveFeedback.ts) | `.ts` |
| 67 | [mcp/InProcessTransport.ts](./src/services/mcp/InProcessTransport.ts) | `.ts` |
| 68 | [mcp/MCPConnectionManager.tsx](./src/services/mcp/MCPConnectionManager.tsx) | `.tsx` |
| 69 | [mcp/SdkControlTransport.ts](./src/services/mcp/SdkControlTransport.ts) | `.ts` |
| 70 | [mcp/auth.ts](./src/services/mcp/auth.ts) | `.ts` |
| 71 | [mcp/channelAllowlist.ts](./src/services/mcp/channelAllowlist.ts) | `.ts` |
| 72 | [mcp/channelNotification.ts](./src/services/mcp/channelNotification.ts) | `.ts` |
| 73 | [mcp/channelPermissions.ts](./src/services/mcp/channelPermissions.ts) | `.ts` |
| 74 | [mcp/claudeai.ts](./src/services/mcp/claudeai.ts) | `.ts` |
| 75 | [mcp/client.ts](./src/services/mcp/client.ts) | `.ts` |
| 76 | [mcp/config.ts](./src/services/mcp/config.ts) | `.ts` |
| 77 | [mcp/elicitationHandler.ts](./src/services/mcp/elicitationHandler.ts) | `.ts` |
| 78 | [mcp/envExpansion.ts](./src/services/mcp/envExpansion.ts) | `.ts` |
| 79 | [mcp/headersHelper.ts](./src/services/mcp/headersHelper.ts) | `.ts` |
| 80 | [mcp/mcpStringUtils.ts](./src/services/mcp/mcpStringUtils.ts) | `.ts` |
| 81 | [mcp/normalization.ts](./src/services/mcp/normalization.ts) | `.ts` |
| 82 | [mcp/oauthPort.ts](./src/services/mcp/oauthPort.ts) | `.ts` |
| 83 | [mcp/officialRegistry.ts](./src/services/mcp/officialRegistry.ts) | `.ts` |
| 84 | [mcp/types.ts](./src/services/mcp/types.ts) | `.ts` |
| 85 | [mcp/useManageMCPConnections.ts](./src/services/mcp/useManageMCPConnections.ts) | `.ts` |
| 86 | [mcp/utils.ts](./src/services/mcp/utils.ts) | `.ts` |
| 87 | [mcp/vscodeSdkMcp.ts](./src/services/mcp/vscodeSdkMcp.ts) | `.ts` |
| 88 | [mcp/xaa.ts](./src/services/mcp/xaa.ts) | `.ts` |
| 89 | [mcp/xaaIdpLogin.ts](./src/services/mcp/xaaIdpLogin.ts) | `.ts` |
| 90 | [mcpServerApproval.tsx](./src/services/mcpServerApproval.tsx) | `.tsx` |
| 91 | [mockRateLimits.ts](./src/services/mockRateLimits.ts) | `.ts` |
| 92 | [notifier.ts](./src/services/notifier.ts) | `.ts` |
| 93 | [oauth/auth-code-listener.ts](./src/services/oauth/auth-code-listener.ts) | `.ts` |
| 94 | [oauth/client.ts](./src/services/oauth/client.ts) | `.ts` |
| 95 | [oauth/crypto.ts](./src/services/oauth/crypto.ts) | `.ts` |
| 96 | [oauth/getOauthProfile.ts](./src/services/oauth/getOauthProfile.ts) | `.ts` |
| 97 | [oauth/index.ts](./src/services/oauth/index.ts) | `.ts` |
| 98 | [plugins/PluginInstallationManager.ts](./src/services/plugins/PluginInstallationManager.ts) | `.ts` |
| 99 | [plugins/pluginCliCommands.ts](./src/services/plugins/pluginCliCommands.ts) | `.ts` |
| 100 | [plugins/pluginOperations.ts](./src/services/plugins/pluginOperations.ts) | `.ts` |
| 101 | [policyLimits/index.ts](./src/services/policyLimits/index.ts) | `.ts` |
| 102 | [policyLimits/types.ts](./src/services/policyLimits/types.ts) | `.ts` |
| 103 | [preventSleep.ts](./src/services/preventSleep.ts) | `.ts` |
| 104 | [rateLimitMessages.ts](./src/services/rateLimitMessages.ts) | `.ts` |
| 105 | [rateLimitMocking.ts](./src/services/rateLimitMocking.ts) | `.ts` |
| 106 | [remoteManagedSettings/index.ts](./src/services/remoteManagedSettings/index.ts) | `.ts` |
| 107 | [remoteManagedSettings/securityCheck.tsx](./src/services/remoteManagedSettings/securityCheck.tsx) | `.tsx` |
| 108 | [remoteManagedSettings/syncCache.ts](./src/services/remoteManagedSettings/syncCache.ts) | `.ts` |
| 109 | [remoteManagedSettings/syncCacheState.ts](./src/services/remoteManagedSettings/syncCacheState.ts) | `.ts` |
| 110 | [remoteManagedSettings/types.ts](./src/services/remoteManagedSettings/types.ts) | `.ts` |
| 111 | [settingsSync/index.ts](./src/services/settingsSync/index.ts) | `.ts` |
| 112 | [settingsSync/types.ts](./src/services/settingsSync/types.ts) | `.ts` |
| 113 | [teamMemorySync/index.ts](./src/services/teamMemorySync/index.ts) | `.ts` |
| 114 | [teamMemorySync/secretScanner.ts](./src/services/teamMemorySync/secretScanner.ts) | `.ts` |
| 115 | [teamMemorySync/teamMemSecretGuard.ts](./src/services/teamMemorySync/teamMemSecretGuard.ts) | `.ts` |
| 116 | [teamMemorySync/types.ts](./src/services/teamMemorySync/types.ts) | `.ts` |
| 117 | [teamMemorySync/watcher.ts](./src/services/teamMemorySync/watcher.ts) | `.ts` |
| 118 | [tips/tipHistory.ts](./src/services/tips/tipHistory.ts) | `.ts` |
| 119 | [tips/tipRegistry.ts](./src/services/tips/tipRegistry.ts) | `.ts` |
| 120 | [tips/tipScheduler.ts](./src/services/tips/tipScheduler.ts) | `.ts` |
| 121 | [tokenEstimation.ts](./src/services/tokenEstimation.ts) | `.ts` |
| 122 | [toolUseSummary/toolUseSummaryGenerator.ts](./src/services/toolUseSummary/toolUseSummaryGenerator.ts) | `.ts` |
| 123 | [tools/StreamingToolExecutor.ts](./src/services/tools/StreamingToolExecutor.ts) | `.ts` |
| 124 | [tools/toolExecution.ts](./src/services/tools/toolExecution.ts) | `.ts` |
| 125 | [tools/toolHooks.ts](./src/services/tools/toolHooks.ts) | `.ts` |
| 126 | [tools/toolOrchestration.ts](./src/services/tools/toolOrchestration.ts) | `.ts` |
| 127 | [vcr.ts](./src/services/vcr.ts) | `.ts` |
| 128 | [voice.ts](./src/services/voice.ts) | `.ts` |
| 129 | [voiceKeyterms.ts](./src/services/voiceKeyterms.ts) | `.ts` |
| 130 | [voiceStreamSTT.ts](./src/services/voiceStreamSTT.ts) | `.ts` |

### skills/

| # | File | Type |
|:--|:-----|:-----|
| 1 | [bundled/batch.ts](./src/skills/bundled/batch.ts) | `.ts` |
| 2 | [bundled/claudeApi.ts](./src/skills/bundled/claudeApi.ts) | `.ts` |
| 3 | [bundled/claudeApiContent.ts](./src/skills/bundled/claudeApiContent.ts) | `.ts` |
| 4 | [bundled/claudeInChrome.ts](./src/skills/bundled/claudeInChrome.ts) | `.ts` |
| 5 | [bundled/debug.ts](./src/skills/bundled/debug.ts) | `.ts` |
| 6 | [bundled/index.ts](./src/skills/bundled/index.ts) | `.ts` |
| 7 | [bundled/keybindings.ts](./src/skills/bundled/keybindings.ts) | `.ts` |
| 8 | [bundled/loop.ts](./src/skills/bundled/loop.ts) | `.ts` |
| 9 | [bundled/loremIpsum.ts](./src/skills/bundled/loremIpsum.ts) | `.ts` |
| 10 | [bundled/remember.ts](./src/skills/bundled/remember.ts) | `.ts` |
| 11 | [bundled/scheduleRemoteAgents.ts](./src/skills/bundled/scheduleRemoteAgents.ts) | `.ts` |
| 12 | [bundled/simplify.ts](./src/skills/bundled/simplify.ts) | `.ts` |
| 13 | [bundled/skillify.ts](./src/skills/bundled/skillify.ts) | `.ts` |
| 14 | [bundled/stuck.ts](./src/skills/bundled/stuck.ts) | `.ts` |
| 15 | [bundled/updateConfig.ts](./src/skills/bundled/updateConfig.ts) | `.ts` |
| 16 | [bundled/verify.ts](./src/skills/bundled/verify.ts) | `.ts` |
| 17 | [bundled/verifyContent.ts](./src/skills/bundled/verifyContent.ts) | `.ts` |
| 18 | [bundledSkills.ts](./src/skills/bundledSkills.ts) | `.ts` |
| 19 | [loadSkillsDir.ts](./src/skills/loadSkillsDir.ts) | `.ts` |
| 20 | [mcpSkillBuilders.ts](./src/skills/mcpSkillBuilders.ts) | `.ts` |

### state/

| # | File | Type |
|:--|:-----|:-----|
| 1 | [AppState.tsx](./src/state/AppState.tsx) | `.tsx` |
| 2 | [AppStateStore.ts](./src/state/AppStateStore.ts) | `.ts` |
| 3 | [onChangeAppState.ts](./src/state/onChangeAppState.ts) | `.ts` |
| 4 | [selectors.ts](./src/state/selectors.ts) | `.ts` |
| 5 | [store.ts](./src/state/store.ts) | `.ts` |
| 6 | [teammateViewHelpers.ts](./src/state/teammateViewHelpers.ts) | `.ts` |

### tasks/

| # | File | Type |
|:--|:-----|:-----|
| 1 | [DreamTask/DreamTask.ts](./src/tasks/DreamTask/DreamTask.ts) | `.ts` |
| 2 | [InProcessTeammateTask/InProcessTeammateTask.tsx](./src/tasks/InProcessTeammateTask/InProcessTeammateTask.tsx) | `.tsx` |
| 3 | [InProcessTeammateTask/types.ts](./src/tasks/InProcessTeammateTask/types.ts) | `.ts` |
| 4 | [LocalAgentTask/LocalAgentTask.tsx](./src/tasks/LocalAgentTask/LocalAgentTask.tsx) | `.tsx` |
| 5 | [LocalMainSessionTask.ts](./src/tasks/LocalMainSessionTask.ts) | `.ts` |
| 6 | [LocalShellTask/LocalShellTask.tsx](./src/tasks/LocalShellTask/LocalShellTask.tsx) | `.tsx` |
| 7 | [LocalShellTask/guards.ts](./src/tasks/LocalShellTask/guards.ts) | `.ts` |
| 8 | [LocalShellTask/killShellTasks.ts](./src/tasks/LocalShellTask/killShellTasks.ts) | `.ts` |
| 9 | [RemoteAgentTask/RemoteAgentTask.tsx](./src/tasks/RemoteAgentTask/RemoteAgentTask.tsx) | `.tsx` |
| 10 | [pillLabel.ts](./src/tasks/pillLabel.ts) | `.ts` |
| 11 | [stopTask.ts](./src/tasks/stopTask.ts) | `.ts` |
| 12 | [types.ts](./src/tasks/types.ts) | `.ts` |

### tools/

| # | File | Type |
|:--|:-----|:-----|
| 1 | [AgentTool/AgentTool.tsx](./src/tools/AgentTool/AgentTool.tsx) | `.tsx` |
| 2 | [AgentTool/UI.tsx](./src/tools/AgentTool/UI.tsx) | `.tsx` |
| 3 | [AgentTool/agentColorManager.ts](./src/tools/AgentTool/agentColorManager.ts) | `.ts` |
| 4 | [AgentTool/agentDisplay.ts](./src/tools/AgentTool/agentDisplay.ts) | `.ts` |
| 5 | [AgentTool/agentMemory.ts](./src/tools/AgentTool/agentMemory.ts) | `.ts` |
| 6 | [AgentTool/agentMemorySnapshot.ts](./src/tools/AgentTool/agentMemorySnapshot.ts) | `.ts` |
| 7 | [AgentTool/agentToolUtils.ts](./src/tools/AgentTool/agentToolUtils.ts) | `.ts` |
| 8 | [AgentTool/built-in/claudeCodeGuideAgent.ts](./src/tools/AgentTool/built-in/claudeCodeGuideAgent.ts) | `.ts` |
| 9 | [AgentTool/built-in/exploreAgent.ts](./src/tools/AgentTool/built-in/exploreAgent.ts) | `.ts` |
| 10 | [AgentTool/built-in/generalPurposeAgent.ts](./src/tools/AgentTool/built-in/generalPurposeAgent.ts) | `.ts` |
| 11 | [AgentTool/built-in/planAgent.ts](./src/tools/AgentTool/built-in/planAgent.ts) | `.ts` |
| 12 | [AgentTool/built-in/statuslineSetup.ts](./src/tools/AgentTool/built-in/statuslineSetup.ts) | `.ts` |
| 13 | [AgentTool/built-in/verificationAgent.ts](./src/tools/AgentTool/built-in/verificationAgent.ts) | `.ts` |
| 14 | [AgentTool/builtInAgents.ts](./src/tools/AgentTool/builtInAgents.ts) | `.ts` |
| 15 | [AgentTool/constants.ts](./src/tools/AgentTool/constants.ts) | `.ts` |
| 16 | [AgentTool/forkSubagent.ts](./src/tools/AgentTool/forkSubagent.ts) | `.ts` |
| 17 | [AgentTool/loadAgentsDir.ts](./src/tools/AgentTool/loadAgentsDir.ts) | `.ts` |
| 18 | [AgentTool/prompt.ts](./src/tools/AgentTool/prompt.ts) | `.ts` |
| 19 | [AgentTool/resumeAgent.ts](./src/tools/AgentTool/resumeAgent.ts) | `.ts` |
| 20 | [AgentTool/runAgent.ts](./src/tools/AgentTool/runAgent.ts) | `.ts` |
| 21 | [AskUserQuestionTool/AskUserQuestionTool.tsx](./src/tools/AskUserQuestionTool/AskUserQuestionTool.tsx) | `.tsx` |
| 22 | [AskUserQuestionTool/prompt.ts](./src/tools/AskUserQuestionTool/prompt.ts) | `.ts` |
| 23 | [BashTool/BashTool.tsx](./src/tools/BashTool/BashTool.tsx) | `.tsx` |
| 24 | [BashTool/BashToolResultMessage.tsx](./src/tools/BashTool/BashToolResultMessage.tsx) | `.tsx` |
| 25 | [BashTool/UI.tsx](./src/tools/BashTool/UI.tsx) | `.tsx` |
| 26 | [BashTool/bashCommandHelpers.ts](./src/tools/BashTool/bashCommandHelpers.ts) | `.ts` |
| 27 | [BashTool/bashPermissions.ts](./src/tools/BashTool/bashPermissions.ts) | `.ts` |
| 28 | [BashTool/bashSecurity.ts](./src/tools/BashTool/bashSecurity.ts) | `.ts` |
| 29 | [BashTool/commandSemantics.ts](./src/tools/BashTool/commandSemantics.ts) | `.ts` |
| 30 | [BashTool/commentLabel.ts](./src/tools/BashTool/commentLabel.ts) | `.ts` |
| 31 | [BashTool/destructiveCommandWarning.ts](./src/tools/BashTool/destructiveCommandWarning.ts) | `.ts` |
| 32 | [BashTool/modeValidation.ts](./src/tools/BashTool/modeValidation.ts) | `.ts` |
| 33 | [BashTool/pathValidation.ts](./src/tools/BashTool/pathValidation.ts) | `.ts` |
| 34 | [BashTool/prompt.ts](./src/tools/BashTool/prompt.ts) | `.ts` |
| 35 | [BashTool/readOnlyValidation.ts](./src/tools/BashTool/readOnlyValidation.ts) | `.ts` |
| 36 | [BashTool/sedEditParser.ts](./src/tools/BashTool/sedEditParser.ts) | `.ts` |
| 37 | [BashTool/sedValidation.ts](./src/tools/BashTool/sedValidation.ts) | `.ts` |
| 38 | [BashTool/shouldUseSandbox.ts](./src/tools/BashTool/shouldUseSandbox.ts) | `.ts` |
| 39 | [BashTool/toolName.ts](./src/tools/BashTool/toolName.ts) | `.ts` |
| 40 | [BashTool/utils.ts](./src/tools/BashTool/utils.ts) | `.ts` |
| 41 | [BriefTool/BriefTool.ts](./src/tools/BriefTool/BriefTool.ts) | `.ts` |
| 42 | [BriefTool/UI.tsx](./src/tools/BriefTool/UI.tsx) | `.tsx` |
| 43 | [BriefTool/attachments.ts](./src/tools/BriefTool/attachments.ts) | `.ts` |
| 44 | [BriefTool/prompt.ts](./src/tools/BriefTool/prompt.ts) | `.ts` |
| 45 | [BriefTool/upload.ts](./src/tools/BriefTool/upload.ts) | `.ts` |
| 46 | [ConfigTool/ConfigTool.ts](./src/tools/ConfigTool/ConfigTool.ts) | `.ts` |
| 47 | [ConfigTool/UI.tsx](./src/tools/ConfigTool/UI.tsx) | `.tsx` |
| 48 | [ConfigTool/constants.ts](./src/tools/ConfigTool/constants.ts) | `.ts` |
| 49 | [ConfigTool/prompt.ts](./src/tools/ConfigTool/prompt.ts) | `.ts` |
| 50 | [ConfigTool/supportedSettings.ts](./src/tools/ConfigTool/supportedSettings.ts) | `.ts` |
| 51 | [EnterPlanModeTool/EnterPlanModeTool.ts](./src/tools/EnterPlanModeTool/EnterPlanModeTool.ts) | `.ts` |
| 52 | [EnterPlanModeTool/UI.tsx](./src/tools/EnterPlanModeTool/UI.tsx) | `.tsx` |
| 53 | [EnterPlanModeTool/constants.ts](./src/tools/EnterPlanModeTool/constants.ts) | `.ts` |
| 54 | [EnterPlanModeTool/prompt.ts](./src/tools/EnterPlanModeTool/prompt.ts) | `.ts` |
| 55 | [EnterWorktreeTool/EnterWorktreeTool.ts](./src/tools/EnterWorktreeTool/EnterWorktreeTool.ts) | `.ts` |
| 56 | [EnterWorktreeTool/UI.tsx](./src/tools/EnterWorktreeTool/UI.tsx) | `.tsx` |
| 57 | [EnterWorktreeTool/constants.ts](./src/tools/EnterWorktreeTool/constants.ts) | `.ts` |
| 58 | [EnterWorktreeTool/prompt.ts](./src/tools/EnterWorktreeTool/prompt.ts) | `.ts` |
| 59 | [ExitPlanModeTool/ExitPlanModeV2Tool.ts](./src/tools/ExitPlanModeTool/ExitPlanModeV2Tool.ts) | `.ts` |
| 60 | [ExitPlanModeTool/UI.tsx](./src/tools/ExitPlanModeTool/UI.tsx) | `.tsx` |
| 61 | [ExitPlanModeTool/constants.ts](./src/tools/ExitPlanModeTool/constants.ts) | `.ts` |
| 62 | [ExitPlanModeTool/prompt.ts](./src/tools/ExitPlanModeTool/prompt.ts) | `.ts` |
| 63 | [ExitWorktreeTool/ExitWorktreeTool.ts](./src/tools/ExitWorktreeTool/ExitWorktreeTool.ts) | `.ts` |
| 64 | [ExitWorktreeTool/UI.tsx](./src/tools/ExitWorktreeTool/UI.tsx) | `.tsx` |
| 65 | [ExitWorktreeTool/constants.ts](./src/tools/ExitWorktreeTool/constants.ts) | `.ts` |
| 66 | [ExitWorktreeTool/prompt.ts](./src/tools/ExitWorktreeTool/prompt.ts) | `.ts` |
| 67 | [FileEditTool/FileEditTool.ts](./src/tools/FileEditTool/FileEditTool.ts) | `.ts` |
| 68 | [FileEditTool/UI.tsx](./src/tools/FileEditTool/UI.tsx) | `.tsx` |
| 69 | [FileEditTool/constants.ts](./src/tools/FileEditTool/constants.ts) | `.ts` |
| 70 | [FileEditTool/prompt.ts](./src/tools/FileEditTool/prompt.ts) | `.ts` |
| 71 | [FileEditTool/types.ts](./src/tools/FileEditTool/types.ts) | `.ts` |
| 72 | [FileEditTool/utils.ts](./src/tools/FileEditTool/utils.ts) | `.ts` |
| 73 | [FileReadTool/FileReadTool.ts](./src/tools/FileReadTool/FileReadTool.ts) | `.ts` |
| 74 | [FileReadTool/UI.tsx](./src/tools/FileReadTool/UI.tsx) | `.tsx` |
| 75 | [FileReadTool/imageProcessor.ts](./src/tools/FileReadTool/imageProcessor.ts) | `.ts` |
| 76 | [FileReadTool/limits.ts](./src/tools/FileReadTool/limits.ts) | `.ts` |
| 77 | [FileReadTool/prompt.ts](./src/tools/FileReadTool/prompt.ts) | `.ts` |
| 78 | [FileWriteTool/FileWriteTool.ts](./src/tools/FileWriteTool/FileWriteTool.ts) | `.ts` |
| 79 | [FileWriteTool/UI.tsx](./src/tools/FileWriteTool/UI.tsx) | `.tsx` |
| 80 | [FileWriteTool/prompt.ts](./src/tools/FileWriteTool/prompt.ts) | `.ts` |
| 81 | [GlobTool/GlobTool.ts](./src/tools/GlobTool/GlobTool.ts) | `.ts` |
| 82 | [GlobTool/UI.tsx](./src/tools/GlobTool/UI.tsx) | `.tsx` |
| 83 | [GlobTool/prompt.ts](./src/tools/GlobTool/prompt.ts) | `.ts` |
| 84 | [GrepTool/GrepTool.ts](./src/tools/GrepTool/GrepTool.ts) | `.ts` |
| 85 | [GrepTool/UI.tsx](./src/tools/GrepTool/UI.tsx) | `.tsx` |
| 86 | [GrepTool/prompt.ts](./src/tools/GrepTool/prompt.ts) | `.ts` |
| 87 | [LSPTool/LSPTool.ts](./src/tools/LSPTool/LSPTool.ts) | `.ts` |
| 88 | [LSPTool/UI.tsx](./src/tools/LSPTool/UI.tsx) | `.tsx` |
| 89 | [LSPTool/formatters.ts](./src/tools/LSPTool/formatters.ts) | `.ts` |
| 90 | [LSPTool/prompt.ts](./src/tools/LSPTool/prompt.ts) | `.ts` |
| 91 | [LSPTool/schemas.ts](./src/tools/LSPTool/schemas.ts) | `.ts` |
| 92 | [LSPTool/symbolContext.ts](./src/tools/LSPTool/symbolContext.ts) | `.ts` |
| 93 | [ListMcpResourcesTool/ListMcpResourcesTool.ts](./src/tools/ListMcpResourcesTool/ListMcpResourcesTool.ts) | `.ts` |
| 94 | [ListMcpResourcesTool/UI.tsx](./src/tools/ListMcpResourcesTool/UI.tsx) | `.tsx` |
| 95 | [ListMcpResourcesTool/prompt.ts](./src/tools/ListMcpResourcesTool/prompt.ts) | `.ts` |
| 96 | [MCPTool/MCPTool.ts](./src/tools/MCPTool/MCPTool.ts) | `.ts` |
| 97 | [MCPTool/UI.tsx](./src/tools/MCPTool/UI.tsx) | `.tsx` |
| 98 | [MCPTool/classifyForCollapse.ts](./src/tools/MCPTool/classifyForCollapse.ts) | `.ts` |
| 99 | [MCPTool/prompt.ts](./src/tools/MCPTool/prompt.ts) | `.ts` |
| 100 | [McpAuthTool/McpAuthTool.ts](./src/tools/McpAuthTool/McpAuthTool.ts) | `.ts` |
| 101 | [NotebookEditTool/NotebookEditTool.ts](./src/tools/NotebookEditTool/NotebookEditTool.ts) | `.ts` |
| 102 | [NotebookEditTool/UI.tsx](./src/tools/NotebookEditTool/UI.tsx) | `.tsx` |
| 103 | [NotebookEditTool/constants.ts](./src/tools/NotebookEditTool/constants.ts) | `.ts` |
| 104 | [NotebookEditTool/prompt.ts](./src/tools/NotebookEditTool/prompt.ts) | `.ts` |
| 105 | [PowerShellTool/PowerShellTool.tsx](./src/tools/PowerShellTool/PowerShellTool.tsx) | `.tsx` |
| 106 | [PowerShellTool/UI.tsx](./src/tools/PowerShellTool/UI.tsx) | `.tsx` |
| 107 | [PowerShellTool/clmTypes.ts](./src/tools/PowerShellTool/clmTypes.ts) | `.ts` |
| 108 | [PowerShellTool/commandSemantics.ts](./src/tools/PowerShellTool/commandSemantics.ts) | `.ts` |
| 109 | [PowerShellTool/commonParameters.ts](./src/tools/PowerShellTool/commonParameters.ts) | `.ts` |
| 110 | [PowerShellTool/destructiveCommandWarning.ts](./src/tools/PowerShellTool/destructiveCommandWarning.ts) | `.ts` |
| 111 | [PowerShellTool/gitSafety.ts](./src/tools/PowerShellTool/gitSafety.ts) | `.ts` |
| 112 | [PowerShellTool/modeValidation.ts](./src/tools/PowerShellTool/modeValidation.ts) | `.ts` |
| 113 | [PowerShellTool/pathValidation.ts](./src/tools/PowerShellTool/pathValidation.ts) | `.ts` |
| 114 | [PowerShellTool/powershellPermissions.ts](./src/tools/PowerShellTool/powershellPermissions.ts) | `.ts` |
| 115 | [PowerShellTool/powershellSecurity.ts](./src/tools/PowerShellTool/powershellSecurity.ts) | `.ts` |
| 116 | [PowerShellTool/prompt.ts](./src/tools/PowerShellTool/prompt.ts) | `.ts` |
| 117 | [PowerShellTool/readOnlyValidation.ts](./src/tools/PowerShellTool/readOnlyValidation.ts) | `.ts` |
| 118 | [PowerShellTool/toolName.ts](./src/tools/PowerShellTool/toolName.ts) | `.ts` |
| 119 | [REPLTool/constants.ts](./src/tools/REPLTool/constants.ts) | `.ts` |
| 120 | [REPLTool/primitiveTools.ts](./src/tools/REPLTool/primitiveTools.ts) | `.ts` |
| 121 | [ReadMcpResourceTool/ReadMcpResourceTool.ts](./src/tools/ReadMcpResourceTool/ReadMcpResourceTool.ts) | `.ts` |
| 122 | [ReadMcpResourceTool/UI.tsx](./src/tools/ReadMcpResourceTool/UI.tsx) | `.tsx` |
| 123 | [ReadMcpResourceTool/prompt.ts](./src/tools/ReadMcpResourceTool/prompt.ts) | `.ts` |
| 124 | [RemoteTriggerTool/RemoteTriggerTool.ts](./src/tools/RemoteTriggerTool/RemoteTriggerTool.ts) | `.ts` |
| 125 | [RemoteTriggerTool/UI.tsx](./src/tools/RemoteTriggerTool/UI.tsx) | `.tsx` |
| 126 | [RemoteTriggerTool/prompt.ts](./src/tools/RemoteTriggerTool/prompt.ts) | `.ts` |
| 127 | [ScheduleCronTool/CronCreateTool.ts](./src/tools/ScheduleCronTool/CronCreateTool.ts) | `.ts` |
| 128 | [ScheduleCronTool/CronDeleteTool.ts](./src/tools/ScheduleCronTool/CronDeleteTool.ts) | `.ts` |
| 129 | [ScheduleCronTool/CronListTool.ts](./src/tools/ScheduleCronTool/CronListTool.ts) | `.ts` |
| 130 | [ScheduleCronTool/UI.tsx](./src/tools/ScheduleCronTool/UI.tsx) | `.tsx` |
| 131 | [ScheduleCronTool/prompt.ts](./src/tools/ScheduleCronTool/prompt.ts) | `.ts` |
| 132 | [SendMessageTool/SendMessageTool.ts](./src/tools/SendMessageTool/SendMessageTool.ts) | `.ts` |
| 133 | [SendMessageTool/UI.tsx](./src/tools/SendMessageTool/UI.tsx) | `.tsx` |
| 134 | [SendMessageTool/constants.ts](./src/tools/SendMessageTool/constants.ts) | `.ts` |
| 135 | [SendMessageTool/prompt.ts](./src/tools/SendMessageTool/prompt.ts) | `.ts` |
| 136 | [SkillTool/SkillTool.ts](./src/tools/SkillTool/SkillTool.ts) | `.ts` |
| 137 | [SkillTool/UI.tsx](./src/tools/SkillTool/UI.tsx) | `.tsx` |
| 138 | [SkillTool/constants.ts](./src/tools/SkillTool/constants.ts) | `.ts` |
| 139 | [SkillTool/prompt.ts](./src/tools/SkillTool/prompt.ts) | `.ts` |
| 140 | [SleepTool/prompt.ts](./src/tools/SleepTool/prompt.ts) | `.ts` |
| 141 | [SyntheticOutputTool/SyntheticOutputTool.ts](./src/tools/SyntheticOutputTool/SyntheticOutputTool.ts) | `.ts` |
| 142 | [TaskCreateTool/TaskCreateTool.ts](./src/tools/TaskCreateTool/TaskCreateTool.ts) | `.ts` |
| 143 | [TaskCreateTool/constants.ts](./src/tools/TaskCreateTool/constants.ts) | `.ts` |
| 144 | [TaskCreateTool/prompt.ts](./src/tools/TaskCreateTool/prompt.ts) | `.ts` |
| 145 | [TaskGetTool/TaskGetTool.ts](./src/tools/TaskGetTool/TaskGetTool.ts) | `.ts` |
| 146 | [TaskGetTool/constants.ts](./src/tools/TaskGetTool/constants.ts) | `.ts` |
| 147 | [TaskGetTool/prompt.ts](./src/tools/TaskGetTool/prompt.ts) | `.ts` |
| 148 | [TaskListTool/TaskListTool.ts](./src/tools/TaskListTool/TaskListTool.ts) | `.ts` |
| 149 | [TaskListTool/constants.ts](./src/tools/TaskListTool/constants.ts) | `.ts` |
| 150 | [TaskListTool/prompt.ts](./src/tools/TaskListTool/prompt.ts) | `.ts` |
| 151 | [TaskOutputTool/TaskOutputTool.tsx](./src/tools/TaskOutputTool/TaskOutputTool.tsx) | `.tsx` |
| 152 | [TaskOutputTool/constants.ts](./src/tools/TaskOutputTool/constants.ts) | `.ts` |
| 153 | [TaskStopTool/TaskStopTool.ts](./src/tools/TaskStopTool/TaskStopTool.ts) | `.ts` |
| 154 | [TaskStopTool/UI.tsx](./src/tools/TaskStopTool/UI.tsx) | `.tsx` |
| 155 | [TaskStopTool/prompt.ts](./src/tools/TaskStopTool/prompt.ts) | `.ts` |
| 156 | [TaskUpdateTool/TaskUpdateTool.ts](./src/tools/TaskUpdateTool/TaskUpdateTool.ts) | `.ts` |
| 157 | [TaskUpdateTool/constants.ts](./src/tools/TaskUpdateTool/constants.ts) | `.ts` |
| 158 | [TaskUpdateTool/prompt.ts](./src/tools/TaskUpdateTool/prompt.ts) | `.ts` |
| 159 | [TeamCreateTool/TeamCreateTool.ts](./src/tools/TeamCreateTool/TeamCreateTool.ts) | `.ts` |
| 160 | [TeamCreateTool/UI.tsx](./src/tools/TeamCreateTool/UI.tsx) | `.tsx` |
| 161 | [TeamCreateTool/constants.ts](./src/tools/TeamCreateTool/constants.ts) | `.ts` |
| 162 | [TeamCreateTool/prompt.ts](./src/tools/TeamCreateTool/prompt.ts) | `.ts` |
| 163 | [TeamDeleteTool/TeamDeleteTool.ts](./src/tools/TeamDeleteTool/TeamDeleteTool.ts) | `.ts` |
| 164 | [TeamDeleteTool/UI.tsx](./src/tools/TeamDeleteTool/UI.tsx) | `.tsx` |
| 165 | [TeamDeleteTool/constants.ts](./src/tools/TeamDeleteTool/constants.ts) | `.ts` |
| 166 | [TeamDeleteTool/prompt.ts](./src/tools/TeamDeleteTool/prompt.ts) | `.ts` |
| 167 | [TodoWriteTool/TodoWriteTool.ts](./src/tools/TodoWriteTool/TodoWriteTool.ts) | `.ts` |
| 168 | [TodoWriteTool/constants.ts](./src/tools/TodoWriteTool/constants.ts) | `.ts` |
| 169 | [TodoWriteTool/prompt.ts](./src/tools/TodoWriteTool/prompt.ts) | `.ts` |
| 170 | [ToolSearchTool/ToolSearchTool.ts](./src/tools/ToolSearchTool/ToolSearchTool.ts) | `.ts` |
| 171 | [ToolSearchTool/constants.ts](./src/tools/ToolSearchTool/constants.ts) | `.ts` |
| 172 | [ToolSearchTool/prompt.ts](./src/tools/ToolSearchTool/prompt.ts) | `.ts` |
| 173 | [WebFetchTool/UI.tsx](./src/tools/WebFetchTool/UI.tsx) | `.tsx` |
| 174 | [WebFetchTool/WebFetchTool.ts](./src/tools/WebFetchTool/WebFetchTool.ts) | `.ts` |
| 175 | [WebFetchTool/preapproved.ts](./src/tools/WebFetchTool/preapproved.ts) | `.ts` |
| 176 | [WebFetchTool/prompt.ts](./src/tools/WebFetchTool/prompt.ts) | `.ts` |
| 177 | [WebFetchTool/utils.ts](./src/tools/WebFetchTool/utils.ts) | `.ts` |
| 178 | [WebSearchTool/UI.tsx](./src/tools/WebSearchTool/UI.tsx) | `.tsx` |
| 179 | [WebSearchTool/WebSearchTool.ts](./src/tools/WebSearchTool/WebSearchTool.ts) | `.ts` |
| 180 | [WebSearchTool/prompt.ts](./src/tools/WebSearchTool/prompt.ts) | `.ts` |
| 181 | [shared/gitOperationTracking.ts](./src/tools/shared/gitOperationTracking.ts) | `.ts` |
| 182 | [shared/spawnMultiAgent.ts](./src/tools/shared/spawnMultiAgent.ts) | `.ts` |
| 183 | [testing/TestingPermissionTool.tsx](./src/tools/testing/TestingPermissionTool.tsx) | `.tsx` |
| 184 | [utils.ts](./src/tools/utils.ts) | `.ts` |

### types/

| # | File | Type |
|:--|:-----|:-----|
| 1 | [command.ts](./src/types/command.ts) | `.ts` |
| 2 | [generated/events_mono/claude_code/v1/claude_code_internal_event.ts](./src/types/generated/events_mono/claude_code/v1/claude_code_internal_event.ts) | `.ts` |
| 3 | [generated/events_mono/common/v1/auth.ts](./src/types/generated/events_mono/common/v1/auth.ts) | `.ts` |
| 4 | [generated/events_mono/growthbook/v1/growthbook_experiment_event.ts](./src/types/generated/events_mono/growthbook/v1/growthbook_experiment_event.ts) | `.ts` |
| 5 | [generated/google/protobuf/timestamp.ts](./src/types/generated/google/protobuf/timestamp.ts) | `.ts` |
| 6 | [hooks.ts](./src/types/hooks.ts) | `.ts` |
| 7 | [ids.ts](./src/types/ids.ts) | `.ts` |
| 8 | [logs.ts](./src/types/logs.ts) | `.ts` |
| 9 | [permissions.ts](./src/types/permissions.ts) | `.ts` |
| 10 | [plugin.ts](./src/types/plugin.ts) | `.ts` |
| 11 | [textInputTypes.ts](./src/types/textInputTypes.ts) | `.ts` |

### upstreamproxy/

| # | File | Type |
|:--|:-----|:-----|
| 1 | [relay.ts](./src/upstreamproxy/relay.ts) | `.ts` |
| 2 | [upstreamproxy.ts](./src/upstreamproxy/upstreamproxy.ts) | `.ts` |

### utils/

| # | File | Type |
|:--|:-----|:-----|
| 1 | [CircularBuffer.ts](./src/utils/CircularBuffer.ts) | `.ts` |
| 2 | [Cursor.ts](./src/utils/Cursor.ts) | `.ts` |
| 3 | [QueryGuard.ts](./src/utils/QueryGuard.ts) | `.ts` |
| 4 | [Shell.ts](./src/utils/Shell.ts) | `.ts` |
| 5 | [ShellCommand.ts](./src/utils/ShellCommand.ts) | `.ts` |
| 6 | [abortController.ts](./src/utils/abortController.ts) | `.ts` |
| 7 | [activityManager.ts](./src/utils/activityManager.ts) | `.ts` |
| 8 | [advisor.ts](./src/utils/advisor.ts) | `.ts` |
| 9 | [agentContext.ts](./src/utils/agentContext.ts) | `.ts` |
| 10 | [agentId.ts](./src/utils/agentId.ts) | `.ts` |
| 11 | [agentSwarmsEnabled.ts](./src/utils/agentSwarmsEnabled.ts) | `.ts` |
| 12 | [agenticSessionSearch.ts](./src/utils/agenticSessionSearch.ts) | `.ts` |
| 13 | [analyzeContext.ts](./src/utils/analyzeContext.ts) | `.ts` |
| 14 | [ansiToPng.ts](./src/utils/ansiToPng.ts) | `.ts` |
| 15 | [ansiToSvg.ts](./src/utils/ansiToSvg.ts) | `.ts` |
| 16 | [api.ts](./src/utils/api.ts) | `.ts` |
| 17 | [apiPreconnect.ts](./src/utils/apiPreconnect.ts) | `.ts` |
| 18 | [appleTerminalBackup.ts](./src/utils/appleTerminalBackup.ts) | `.ts` |
| 19 | [argumentSubstitution.ts](./src/utils/argumentSubstitution.ts) | `.ts` |
| 20 | [array.ts](./src/utils/array.ts) | `.ts` |
| 21 | [asciicast.ts](./src/utils/asciicast.ts) | `.ts` |
| 22 | [attachments.ts](./src/utils/attachments.ts) | `.ts` |
| 23 | [attribution.ts](./src/utils/attribution.ts) | `.ts` |
| 24 | [auth.ts](./src/utils/auth.ts) | `.ts` |
| 25 | [authFileDescriptor.ts](./src/utils/authFileDescriptor.ts) | `.ts` |
| 26 | [authPortable.ts](./src/utils/authPortable.ts) | `.ts` |
| 27 | [autoModeDenials.ts](./src/utils/autoModeDenials.ts) | `.ts` |
| 28 | [autoRunIssue.tsx](./src/utils/autoRunIssue.tsx) | `.tsx` |
| 29 | [autoUpdater.ts](./src/utils/autoUpdater.ts) | `.ts` |
| 30 | [aws.ts](./src/utils/aws.ts) | `.ts` |
| 31 | [awsAuthStatusManager.ts](./src/utils/awsAuthStatusManager.ts) | `.ts` |
| 32 | [background/remote/preconditions.ts](./src/utils/background/remote/preconditions.ts) | `.ts` |
| 33 | [background/remote/remoteSession.ts](./src/utils/background/remote/remoteSession.ts) | `.ts` |
| 34 | [backgroundHousekeeping.ts](./src/utils/backgroundHousekeeping.ts) | `.ts` |
| 35 | [bash/ParsedCommand.ts](./src/utils/bash/ParsedCommand.ts) | `.ts` |
| 36 | [bash/ShellSnapshot.ts](./src/utils/bash/ShellSnapshot.ts) | `.ts` |
| 37 | [bash/ast.ts](./src/utils/bash/ast.ts) | `.ts` |
| 38 | [bash/bashParser.ts](./src/utils/bash/bashParser.ts) | `.ts` |
| 39 | [bash/bashPipeCommand.ts](./src/utils/bash/bashPipeCommand.ts) | `.ts` |
| 40 | [bash/commands.ts](./src/utils/bash/commands.ts) | `.ts` |
| 41 | [bash/heredoc.ts](./src/utils/bash/heredoc.ts) | `.ts` |
| 42 | [bash/parser.ts](./src/utils/bash/parser.ts) | `.ts` |
| 43 | [bash/prefix.ts](./src/utils/bash/prefix.ts) | `.ts` |
| 44 | [bash/registry.ts](./src/utils/bash/registry.ts) | `.ts` |
| 45 | [bash/shellCompletion.ts](./src/utils/bash/shellCompletion.ts) | `.ts` |
| 46 | [bash/shellPrefix.ts](./src/utils/bash/shellPrefix.ts) | `.ts` |
| 47 | [bash/shellQuote.ts](./src/utils/bash/shellQuote.ts) | `.ts` |
| 48 | [bash/shellQuoting.ts](./src/utils/bash/shellQuoting.ts) | `.ts` |
| 49 | [bash/specs/alias.ts](./src/utils/bash/specs/alias.ts) | `.ts` |
| 50 | [bash/specs/index.ts](./src/utils/bash/specs/index.ts) | `.ts` |
| 51 | [bash/specs/nohup.ts](./src/utils/bash/specs/nohup.ts) | `.ts` |
| 52 | [bash/specs/pyright.ts](./src/utils/bash/specs/pyright.ts) | `.ts` |
| 53 | [bash/specs/sleep.ts](./src/utils/bash/specs/sleep.ts) | `.ts` |
| 54 | [bash/specs/srun.ts](./src/utils/bash/specs/srun.ts) | `.ts` |
| 55 | [bash/specs/time.ts](./src/utils/bash/specs/time.ts) | `.ts` |
| 56 | [bash/specs/timeout.ts](./src/utils/bash/specs/timeout.ts) | `.ts` |
| 57 | [bash/treeSitterAnalysis.ts](./src/utils/bash/treeSitterAnalysis.ts) | `.ts` |
| 58 | [betas.ts](./src/utils/betas.ts) | `.ts` |
| 59 | [billing.ts](./src/utils/billing.ts) | `.ts` |
| 60 | [binaryCheck.ts](./src/utils/binaryCheck.ts) | `.ts` |
| 61 | [browser.ts](./src/utils/browser.ts) | `.ts` |
| 62 | [bufferedWriter.ts](./src/utils/bufferedWriter.ts) | `.ts` |
| 63 | [bundledMode.ts](./src/utils/bundledMode.ts) | `.ts` |
| 64 | [caCerts.ts](./src/utils/caCerts.ts) | `.ts` |
| 65 | [caCertsConfig.ts](./src/utils/caCertsConfig.ts) | `.ts` |
| 66 | [cachePaths.ts](./src/utils/cachePaths.ts) | `.ts` |
| 67 | [classifierApprovals.ts](./src/utils/classifierApprovals.ts) | `.ts` |
| 68 | [classifierApprovalsHook.ts](./src/utils/classifierApprovalsHook.ts) | `.ts` |
| 69 | [claudeCodeHints.ts](./src/utils/claudeCodeHints.ts) | `.ts` |
| 70 | [claudeDesktop.ts](./src/utils/claudeDesktop.ts) | `.ts` |
| 71 | [claudeInChrome/chromeNativeHost.ts](./src/utils/claudeInChrome/chromeNativeHost.ts) | `.ts` |
| 72 | [claudeInChrome/common.ts](./src/utils/claudeInChrome/common.ts) | `.ts` |
| 73 | [claudeInChrome/mcpServer.ts](./src/utils/claudeInChrome/mcpServer.ts) | `.ts` |
| 74 | [claudeInChrome/prompt.ts](./src/utils/claudeInChrome/prompt.ts) | `.ts` |
| 75 | [claudeInChrome/setup.ts](./src/utils/claudeInChrome/setup.ts) | `.ts` |
| 76 | [claudeInChrome/setupPortable.ts](./src/utils/claudeInChrome/setupPortable.ts) | `.ts` |
| 77 | [claudeInChrome/toolRendering.tsx](./src/utils/claudeInChrome/toolRendering.tsx) | `.tsx` |
| 78 | [claudemd.ts](./src/utils/claudemd.ts) | `.ts` |
| 79 | [cleanup.ts](./src/utils/cleanup.ts) | `.ts` |
| 80 | [cleanupRegistry.ts](./src/utils/cleanupRegistry.ts) | `.ts` |
| 81 | [cliArgs.ts](./src/utils/cliArgs.ts) | `.ts` |
| 82 | [cliHighlight.ts](./src/utils/cliHighlight.ts) | `.ts` |
| 83 | [codeIndexing.ts](./src/utils/codeIndexing.ts) | `.ts` |
| 84 | [collapseBackgroundBashNotifications.ts](./src/utils/collapseBackgroundBashNotifications.ts) | `.ts` |
| 85 | [collapseHookSummaries.ts](./src/utils/collapseHookSummaries.ts) | `.ts` |
| 86 | [collapseReadSearch.ts](./src/utils/collapseReadSearch.ts) | `.ts` |
| 87 | [collapseTeammateShutdowns.ts](./src/utils/collapseTeammateShutdowns.ts) | `.ts` |
| 88 | [combinedAbortSignal.ts](./src/utils/combinedAbortSignal.ts) | `.ts` |
| 89 | [commandLifecycle.ts](./src/utils/commandLifecycle.ts) | `.ts` |
| 90 | [commitAttribution.ts](./src/utils/commitAttribution.ts) | `.ts` |
| 91 | [completionCache.ts](./src/utils/completionCache.ts) | `.ts` |
| 92 | [computerUse/appNames.ts](./src/utils/computerUse/appNames.ts) | `.ts` |
| 93 | [computerUse/cleanup.ts](./src/utils/computerUse/cleanup.ts) | `.ts` |
| 94 | [computerUse/common.ts](./src/utils/computerUse/common.ts) | `.ts` |
| 95 | [computerUse/computerUseLock.ts](./src/utils/computerUse/computerUseLock.ts) | `.ts` |
| 96 | [computerUse/drainRunLoop.ts](./src/utils/computerUse/drainRunLoop.ts) | `.ts` |
| 97 | [computerUse/escHotkey.ts](./src/utils/computerUse/escHotkey.ts) | `.ts` |
| 98 | [computerUse/executor.ts](./src/utils/computerUse/executor.ts) | `.ts` |
| 99 | [computerUse/gates.ts](./src/utils/computerUse/gates.ts) | `.ts` |
| 100 | [computerUse/hostAdapter.ts](./src/utils/computerUse/hostAdapter.ts) | `.ts` |
| 101 | [computerUse/inputLoader.ts](./src/utils/computerUse/inputLoader.ts) | `.ts` |
| 102 | [computerUse/mcpServer.ts](./src/utils/computerUse/mcpServer.ts) | `.ts` |
| 103 | [computerUse/setup.ts](./src/utils/computerUse/setup.ts) | `.ts` |
| 104 | [computerUse/swiftLoader.ts](./src/utils/computerUse/swiftLoader.ts) | `.ts` |
| 105 | [computerUse/toolRendering.tsx](./src/utils/computerUse/toolRendering.tsx) | `.tsx` |
| 106 | [computerUse/wrapper.tsx](./src/utils/computerUse/wrapper.tsx) | `.tsx` |
| 107 | [concurrentSessions.ts](./src/utils/concurrentSessions.ts) | `.ts` |
| 108 | [config.ts](./src/utils/config.ts) | `.ts` |
| 109 | [configConstants.ts](./src/utils/configConstants.ts) | `.ts` |
| 110 | [contentArray.ts](./src/utils/contentArray.ts) | `.ts` |
| 111 | [context.ts](./src/utils/context.ts) | `.ts` |
| 112 | [contextAnalysis.ts](./src/utils/contextAnalysis.ts) | `.ts` |
| 113 | [contextSuggestions.ts](./src/utils/contextSuggestions.ts) | `.ts` |
| 114 | [controlMessageCompat.ts](./src/utils/controlMessageCompat.ts) | `.ts` |
| 115 | [conversationRecovery.ts](./src/utils/conversationRecovery.ts) | `.ts` |
| 116 | [cron.ts](./src/utils/cron.ts) | `.ts` |
| 117 | [cronJitterConfig.ts](./src/utils/cronJitterConfig.ts) | `.ts` |
| 118 | [cronScheduler.ts](./src/utils/cronScheduler.ts) | `.ts` |
| 119 | [cronTasks.ts](./src/utils/cronTasks.ts) | `.ts` |
| 120 | [cronTasksLock.ts](./src/utils/cronTasksLock.ts) | `.ts` |
| 121 | [crossProjectResume.ts](./src/utils/crossProjectResume.ts) | `.ts` |
| 122 | [crypto.ts](./src/utils/crypto.ts) | `.ts` |
| 123 | [cwd.ts](./src/utils/cwd.ts) | `.ts` |
| 124 | [debug.ts](./src/utils/debug.ts) | `.ts` |
| 125 | [debugFilter.ts](./src/utils/debugFilter.ts) | `.ts` |
| 126 | [deepLink/banner.ts](./src/utils/deepLink/banner.ts) | `.ts` |
| 127 | [deepLink/parseDeepLink.ts](./src/utils/deepLink/parseDeepLink.ts) | `.ts` |
| 128 | [deepLink/protocolHandler.ts](./src/utils/deepLink/protocolHandler.ts) | `.ts` |
| 129 | [deepLink/registerProtocol.ts](./src/utils/deepLink/registerProtocol.ts) | `.ts` |
| 130 | [deepLink/terminalLauncher.ts](./src/utils/deepLink/terminalLauncher.ts) | `.ts` |
| 131 | [deepLink/terminalPreference.ts](./src/utils/deepLink/terminalPreference.ts) | `.ts` |
| 132 | [desktopDeepLink.ts](./src/utils/desktopDeepLink.ts) | `.ts` |
| 133 | [detectRepository.ts](./src/utils/detectRepository.ts) | `.ts` |
| 134 | [diagLogs.ts](./src/utils/diagLogs.ts) | `.ts` |
| 135 | [diff.ts](./src/utils/diff.ts) | `.ts` |
| 136 | [directMemberMessage.ts](./src/utils/directMemberMessage.ts) | `.ts` |
| 137 | [displayTags.ts](./src/utils/displayTags.ts) | `.ts` |
| 138 | [doctorContextWarnings.ts](./src/utils/doctorContextWarnings.ts) | `.ts` |
| 139 | [doctorDiagnostic.ts](./src/utils/doctorDiagnostic.ts) | `.ts` |
| 140 | [dxt/helpers.ts](./src/utils/dxt/helpers.ts) | `.ts` |
| 141 | [dxt/zip.ts](./src/utils/dxt/zip.ts) | `.ts` |
| 142 | [earlyInput.ts](./src/utils/earlyInput.ts) | `.ts` |
| 143 | [editor.ts](./src/utils/editor.ts) | `.ts` |
| 144 | [effort.ts](./src/utils/effort.ts) | `.ts` |
| 145 | [embeddedTools.ts](./src/utils/embeddedTools.ts) | `.ts` |
| 146 | [env.ts](./src/utils/env.ts) | `.ts` |
| 147 | [envDynamic.ts](./src/utils/envDynamic.ts) | `.ts` |
| 148 | [envUtils.ts](./src/utils/envUtils.ts) | `.ts` |
| 149 | [envValidation.ts](./src/utils/envValidation.ts) | `.ts` |
| 150 | [errorLogSink.ts](./src/utils/errorLogSink.ts) | `.ts` |
| 151 | [errors.ts](./src/utils/errors.ts) | `.ts` |
| 152 | [exampleCommands.ts](./src/utils/exampleCommands.ts) | `.ts` |
| 153 | [execFileNoThrow.ts](./src/utils/execFileNoThrow.ts) | `.ts` |
| 154 | [execFileNoThrowPortable.ts](./src/utils/execFileNoThrowPortable.ts) | `.ts` |
| 155 | [execSyncWrapper.ts](./src/utils/execSyncWrapper.ts) | `.ts` |
| 156 | [exportRenderer.tsx](./src/utils/exportRenderer.tsx) | `.tsx` |
| 157 | [extraUsage.ts](./src/utils/extraUsage.ts) | `.ts` |
| 158 | [fastMode.ts](./src/utils/fastMode.ts) | `.ts` |
| 159 | [file.ts](./src/utils/file.ts) | `.ts` |
| 160 | [fileHistory.ts](./src/utils/fileHistory.ts) | `.ts` |
| 161 | [fileOperationAnalytics.ts](./src/utils/fileOperationAnalytics.ts) | `.ts` |
| 162 | [filePersistence/filePersistence.ts](./src/utils/filePersistence/filePersistence.ts) | `.ts` |
| 163 | [filePersistence/outputsScanner.ts](./src/utils/filePersistence/outputsScanner.ts) | `.ts` |
| 164 | [fileRead.ts](./src/utils/fileRead.ts) | `.ts` |
| 165 | [fileReadCache.ts](./src/utils/fileReadCache.ts) | `.ts` |
| 166 | [fileStateCache.ts](./src/utils/fileStateCache.ts) | `.ts` |
| 167 | [findExecutable.ts](./src/utils/findExecutable.ts) | `.ts` |
| 168 | [fingerprint.ts](./src/utils/fingerprint.ts) | `.ts` |
| 169 | [forkedAgent.ts](./src/utils/forkedAgent.ts) | `.ts` |
| 170 | [format.ts](./src/utils/format.ts) | `.ts` |
| 171 | [formatBriefTimestamp.ts](./src/utils/formatBriefTimestamp.ts) | `.ts` |
| 172 | [fpsTracker.ts](./src/utils/fpsTracker.ts) | `.ts` |
| 173 | [frontmatterParser.ts](./src/utils/frontmatterParser.ts) | `.ts` |
| 174 | [fsOperations.ts](./src/utils/fsOperations.ts) | `.ts` |
| 175 | [fullscreen.ts](./src/utils/fullscreen.ts) | `.ts` |
| 176 | [generatedFiles.ts](./src/utils/generatedFiles.ts) | `.ts` |
| 177 | [generators.ts](./src/utils/generators.ts) | `.ts` |
| 178 | [genericProcessUtils.ts](./src/utils/genericProcessUtils.ts) | `.ts` |
| 179 | [getWorktreePaths.ts](./src/utils/getWorktreePaths.ts) | `.ts` |
| 180 | [getWorktreePathsPortable.ts](./src/utils/getWorktreePathsPortable.ts) | `.ts` |
| 181 | [ghPrStatus.ts](./src/utils/ghPrStatus.ts) | `.ts` |
| 182 | [git.ts](./src/utils/git.ts) | `.ts` |
| 183 | [git/gitConfigParser.ts](./src/utils/git/gitConfigParser.ts) | `.ts` |
| 184 | [git/gitFilesystem.ts](./src/utils/git/gitFilesystem.ts) | `.ts` |
| 185 | [git/gitignore.ts](./src/utils/git/gitignore.ts) | `.ts` |
| 186 | [gitDiff.ts](./src/utils/gitDiff.ts) | `.ts` |
| 187 | [gitSettings.ts](./src/utils/gitSettings.ts) | `.ts` |
| 188 | [github/ghAuthStatus.ts](./src/utils/github/ghAuthStatus.ts) | `.ts` |
| 189 | [githubRepoPathMapping.ts](./src/utils/githubRepoPathMapping.ts) | `.ts` |
| 190 | [glob.ts](./src/utils/glob.ts) | `.ts` |
| 191 | [gracefulShutdown.ts](./src/utils/gracefulShutdown.ts) | `.ts` |
| 192 | [groupToolUses.ts](./src/utils/groupToolUses.ts) | `.ts` |
| 193 | [handlePromptSubmit.ts](./src/utils/handlePromptSubmit.ts) | `.ts` |
| 194 | [hash.ts](./src/utils/hash.ts) | `.ts` |
| 195 | [headlessProfiler.ts](./src/utils/headlessProfiler.ts) | `.ts` |
| 196 | [heapDumpService.ts](./src/utils/heapDumpService.ts) | `.ts` |
| 197 | [heatmap.ts](./src/utils/heatmap.ts) | `.ts` |
| 198 | [highlightMatch.tsx](./src/utils/highlightMatch.tsx) | `.tsx` |
| 199 | [hooks.ts](./src/utils/hooks.ts) | `.ts` |
| 200 | [hooks/AsyncHookRegistry.ts](./src/utils/hooks/AsyncHookRegistry.ts) | `.ts` |
| 201 | [hooks/apiQueryHookHelper.ts](./src/utils/hooks/apiQueryHookHelper.ts) | `.ts` |
| 202 | [hooks/execAgentHook.ts](./src/utils/hooks/execAgentHook.ts) | `.ts` |
| 203 | [hooks/execHttpHook.ts](./src/utils/hooks/execHttpHook.ts) | `.ts` |
| 204 | [hooks/execPromptHook.ts](./src/utils/hooks/execPromptHook.ts) | `.ts` |
| 205 | [hooks/fileChangedWatcher.ts](./src/utils/hooks/fileChangedWatcher.ts) | `.ts` |
| 206 | [hooks/hookEvents.ts](./src/utils/hooks/hookEvents.ts) | `.ts` |
| 207 | [hooks/hookHelpers.ts](./src/utils/hooks/hookHelpers.ts) | `.ts` |
| 208 | [hooks/hooksConfigManager.ts](./src/utils/hooks/hooksConfigManager.ts) | `.ts` |
| 209 | [hooks/hooksConfigSnapshot.ts](./src/utils/hooks/hooksConfigSnapshot.ts) | `.ts` |
| 210 | [hooks/hooksSettings.ts](./src/utils/hooks/hooksSettings.ts) | `.ts` |
| 211 | [hooks/postSamplingHooks.ts](./src/utils/hooks/postSamplingHooks.ts) | `.ts` |
| 212 | [hooks/registerFrontmatterHooks.ts](./src/utils/hooks/registerFrontmatterHooks.ts) | `.ts` |
| 213 | [hooks/registerSkillHooks.ts](./src/utils/hooks/registerSkillHooks.ts) | `.ts` |
| 214 | [hooks/sessionHooks.ts](./src/utils/hooks/sessionHooks.ts) | `.ts` |
| 215 | [hooks/skillImprovement.ts](./src/utils/hooks/skillImprovement.ts) | `.ts` |
| 216 | [hooks/ssrfGuard.ts](./src/utils/hooks/ssrfGuard.ts) | `.ts` |
| 217 | [horizontalScroll.ts](./src/utils/horizontalScroll.ts) | `.ts` |
| 218 | [http.ts](./src/utils/http.ts) | `.ts` |
| 219 | [hyperlink.ts](./src/utils/hyperlink.ts) | `.ts` |
| 220 | [iTermBackup.ts](./src/utils/iTermBackup.ts) | `.ts` |
| 221 | [ide.ts](./src/utils/ide.ts) | `.ts` |
| 222 | [idePathConversion.ts](./src/utils/idePathConversion.ts) | `.ts` |
| 223 | [idleTimeout.ts](./src/utils/idleTimeout.ts) | `.ts` |
| 224 | [imagePaste.ts](./src/utils/imagePaste.ts) | `.ts` |
| 225 | [imageResizer.ts](./src/utils/imageResizer.ts) | `.ts` |
| 226 | [imageStore.ts](./src/utils/imageStore.ts) | `.ts` |
| 227 | [imageValidation.ts](./src/utils/imageValidation.ts) | `.ts` |
| 228 | [immediateCommand.ts](./src/utils/immediateCommand.ts) | `.ts` |
| 229 | [inProcessTeammateHelpers.ts](./src/utils/inProcessTeammateHelpers.ts) | `.ts` |
| 230 | [ink.ts](./src/utils/ink.ts) | `.ts` |
| 231 | [intl.ts](./src/utils/intl.ts) | `.ts` |
| 232 | [jetbrains.ts](./src/utils/jetbrains.ts) | `.ts` |
| 233 | [json.ts](./src/utils/json.ts) | `.ts` |
| 234 | [jsonRead.ts](./src/utils/jsonRead.ts) | `.ts` |
| 235 | [keyboardShortcuts.ts](./src/utils/keyboardShortcuts.ts) | `.ts` |
| 236 | [lazySchema.ts](./src/utils/lazySchema.ts) | `.ts` |
| 237 | [listSessionsImpl.ts](./src/utils/listSessionsImpl.ts) | `.ts` |
| 238 | [localInstaller.ts](./src/utils/localInstaller.ts) | `.ts` |
| 239 | [lockfile.ts](./src/utils/lockfile.ts) | `.ts` |
| 240 | [log.ts](./src/utils/log.ts) | `.ts` |
| 241 | [logoV2Utils.ts](./src/utils/logoV2Utils.ts) | `.ts` |
| 242 | [mailbox.ts](./src/utils/mailbox.ts) | `.ts` |
| 243 | [managedEnv.ts](./src/utils/managedEnv.ts) | `.ts` |
| 244 | [managedEnvConstants.ts](./src/utils/managedEnvConstants.ts) | `.ts` |
| 245 | [markdown.ts](./src/utils/markdown.ts) | `.ts` |
| 246 | [markdownConfigLoader.ts](./src/utils/markdownConfigLoader.ts) | `.ts` |
| 247 | [mcp/dateTimeParser.ts](./src/utils/mcp/dateTimeParser.ts) | `.ts` |
| 248 | [mcp/elicitationValidation.ts](./src/utils/mcp/elicitationValidation.ts) | `.ts` |
| 249 | [mcpInstructionsDelta.ts](./src/utils/mcpInstructionsDelta.ts) | `.ts` |
| 250 | [mcpOutputStorage.ts](./src/utils/mcpOutputStorage.ts) | `.ts` |
| 251 | [mcpValidation.ts](./src/utils/mcpValidation.ts) | `.ts` |
| 252 | [mcpWebSocketTransport.ts](./src/utils/mcpWebSocketTransport.ts) | `.ts` |
| 253 | [memoize.ts](./src/utils/memoize.ts) | `.ts` |
| 254 | [memory/types.ts](./src/utils/memory/types.ts) | `.ts` |
| 255 | [memory/versions.ts](./src/utils/memory/versions.ts) | `.ts` |
| 256 | [memoryFileDetection.ts](./src/utils/memoryFileDetection.ts) | `.ts` |
| 257 | [messagePredicates.ts](./src/utils/messagePredicates.ts) | `.ts` |
| 258 | [messageQueueManager.ts](./src/utils/messageQueueManager.ts) | `.ts` |
| 259 | [messages.ts](./src/utils/messages.ts) | `.ts` |
| 260 | [messages/mappers.ts](./src/utils/messages/mappers.ts) | `.ts` |
| 261 | [messages/systemInit.ts](./src/utils/messages/systemInit.ts) | `.ts` |
| 262 | [model/agent.ts](./src/utils/model/agent.ts) | `.ts` |
| 263 | [model/aliases.ts](./src/utils/model/aliases.ts) | `.ts` |
| 264 | [model/antModels.ts](./src/utils/model/antModels.ts) | `.ts` |
| 265 | [model/bedrock.ts](./src/utils/model/bedrock.ts) | `.ts` |
| 266 | [model/check1mAccess.ts](./src/utils/model/check1mAccess.ts) | `.ts` |
| 267 | [model/configs.ts](./src/utils/model/configs.ts) | `.ts` |
| 268 | [model/contextWindowUpgradeCheck.ts](./src/utils/model/contextWindowUpgradeCheck.ts) | `.ts` |
| 269 | [model/deprecation.ts](./src/utils/model/deprecation.ts) | `.ts` |
| 270 | [model/model.ts](./src/utils/model/model.ts) | `.ts` |
| 271 | [model/modelAllowlist.ts](./src/utils/model/modelAllowlist.ts) | `.ts` |
| 272 | [model/modelCapabilities.ts](./src/utils/model/modelCapabilities.ts) | `.ts` |
| 273 | [model/modelOptions.ts](./src/utils/model/modelOptions.ts) | `.ts` |
| 274 | [model/modelStrings.ts](./src/utils/model/modelStrings.ts) | `.ts` |
| 275 | [model/modelSupportOverrides.ts](./src/utils/model/modelSupportOverrides.ts) | `.ts` |
| 276 | [model/providers.ts](./src/utils/model/providers.ts) | `.ts` |
| 277 | [model/validateModel.ts](./src/utils/model/validateModel.ts) | `.ts` |
| 278 | [modelCost.ts](./src/utils/modelCost.ts) | `.ts` |
| 279 | [modifiers.ts](./src/utils/modifiers.ts) | `.ts` |
| 280 | [mtls.ts](./src/utils/mtls.ts) | `.ts` |
| 281 | [nativeInstaller/download.ts](./src/utils/nativeInstaller/download.ts) | `.ts` |
| 282 | [nativeInstaller/index.ts](./src/utils/nativeInstaller/index.ts) | `.ts` |
| 283 | [nativeInstaller/installer.ts](./src/utils/nativeInstaller/installer.ts) | `.ts` |
| 284 | [nativeInstaller/packageManagers.ts](./src/utils/nativeInstaller/packageManagers.ts) | `.ts` |
| 285 | [nativeInstaller/pidLock.ts](./src/utils/nativeInstaller/pidLock.ts) | `.ts` |
| 286 | [notebook.ts](./src/utils/notebook.ts) | `.ts` |
| 287 | [objectGroupBy.ts](./src/utils/objectGroupBy.ts) | `.ts` |
| 288 | [pasteStore.ts](./src/utils/pasteStore.ts) | `.ts` |
| 289 | [path.ts](./src/utils/path.ts) | `.ts` |
| 290 | [pdf.ts](./src/utils/pdf.ts) | `.ts` |
| 291 | [pdfUtils.ts](./src/utils/pdfUtils.ts) | `.ts` |
| 292 | [peerAddress.ts](./src/utils/peerAddress.ts) | `.ts` |
| 293 | [permissions/PermissionMode.ts](./src/utils/permissions/PermissionMode.ts) | `.ts` |
| 294 | [permissions/PermissionPromptToolResultSchema.ts](./src/utils/permissions/PermissionPromptToolResultSchema.ts) | `.ts` |
| 295 | [permissions/PermissionResult.ts](./src/utils/permissions/PermissionResult.ts) | `.ts` |
| 296 | [permissions/PermissionRule.ts](./src/utils/permissions/PermissionRule.ts) | `.ts` |
| 297 | [permissions/PermissionUpdate.ts](./src/utils/permissions/PermissionUpdate.ts) | `.ts` |
| 298 | [permissions/PermissionUpdateSchema.ts](./src/utils/permissions/PermissionUpdateSchema.ts) | `.ts` |
| 299 | [permissions/autoModeState.ts](./src/utils/permissions/autoModeState.ts) | `.ts` |
| 300 | [permissions/bashClassifier.ts](./src/utils/permissions/bashClassifier.ts) | `.ts` |
| 301 | [permissions/bypassPermissionsKillswitch.ts](./src/utils/permissions/bypassPermissionsKillswitch.ts) | `.ts` |
| 302 | [permissions/classifierDecision.ts](./src/utils/permissions/classifierDecision.ts) | `.ts` |
| 303 | [permissions/classifierShared.ts](./src/utils/permissions/classifierShared.ts) | `.ts` |
| 304 | [permissions/dangerousPatterns.ts](./src/utils/permissions/dangerousPatterns.ts) | `.ts` |
| 305 | [permissions/denialTracking.ts](./src/utils/permissions/denialTracking.ts) | `.ts` |
| 306 | [permissions/filesystem.ts](./src/utils/permissions/filesystem.ts) | `.ts` |
| 307 | [permissions/getNextPermissionMode.ts](./src/utils/permissions/getNextPermissionMode.ts) | `.ts` |
| 308 | [permissions/pathValidation.ts](./src/utils/permissions/pathValidation.ts) | `.ts` |
| 309 | [permissions/permissionExplainer.ts](./src/utils/permissions/permissionExplainer.ts) | `.ts` |
| 310 | [permissions/permissionRuleParser.ts](./src/utils/permissions/permissionRuleParser.ts) | `.ts` |
| 311 | [permissions/permissionSetup.ts](./src/utils/permissions/permissionSetup.ts) | `.ts` |
| 312 | [permissions/permissions.ts](./src/utils/permissions/permissions.ts) | `.ts` |
| 313 | [permissions/permissionsLoader.ts](./src/utils/permissions/permissionsLoader.ts) | `.ts` |
| 314 | [permissions/shadowedRuleDetection.ts](./src/utils/permissions/shadowedRuleDetection.ts) | `.ts` |
| 315 | [permissions/shellRuleMatching.ts](./src/utils/permissions/shellRuleMatching.ts) | `.ts` |
| 316 | [permissions/yoloClassifier.ts](./src/utils/permissions/yoloClassifier.ts) | `.ts` |
| 317 | [planModeV2.ts](./src/utils/planModeV2.ts) | `.ts` |
| 318 | [plans.ts](./src/utils/plans.ts) | `.ts` |
| 319 | [platform.ts](./src/utils/platform.ts) | `.ts` |
| 320 | [plugins/addDirPluginSettings.ts](./src/utils/plugins/addDirPluginSettings.ts) | `.ts` |
| 321 | [plugins/cacheUtils.ts](./src/utils/plugins/cacheUtils.ts) | `.ts` |
| 322 | [plugins/dependencyResolver.ts](./src/utils/plugins/dependencyResolver.ts) | `.ts` |
| 323 | [plugins/fetchTelemetry.ts](./src/utils/plugins/fetchTelemetry.ts) | `.ts` |
| 324 | [plugins/gitAvailability.ts](./src/utils/plugins/gitAvailability.ts) | `.ts` |
| 325 | [plugins/headlessPluginInstall.ts](./src/utils/plugins/headlessPluginInstall.ts) | `.ts` |
| 326 | [plugins/hintRecommendation.ts](./src/utils/plugins/hintRecommendation.ts) | `.ts` |
| 327 | [plugins/installCounts.ts](./src/utils/plugins/installCounts.ts) | `.ts` |
| 328 | [plugins/installedPluginsManager.ts](./src/utils/plugins/installedPluginsManager.ts) | `.ts` |
| 329 | [plugins/loadPluginAgents.ts](./src/utils/plugins/loadPluginAgents.ts) | `.ts` |
| 330 | [plugins/loadPluginCommands.ts](./src/utils/plugins/loadPluginCommands.ts) | `.ts` |
| 331 | [plugins/loadPluginHooks.ts](./src/utils/plugins/loadPluginHooks.ts) | `.ts` |
| 332 | [plugins/loadPluginOutputStyles.ts](./src/utils/plugins/loadPluginOutputStyles.ts) | `.ts` |
| 333 | [plugins/lspPluginIntegration.ts](./src/utils/plugins/lspPluginIntegration.ts) | `.ts` |
| 334 | [plugins/lspRecommendation.ts](./src/utils/plugins/lspRecommendation.ts) | `.ts` |
| 335 | [plugins/managedPlugins.ts](./src/utils/plugins/managedPlugins.ts) | `.ts` |
| 336 | [plugins/marketplaceHelpers.ts](./src/utils/plugins/marketplaceHelpers.ts) | `.ts` |
| 337 | [plugins/marketplaceManager.ts](./src/utils/plugins/marketplaceManager.ts) | `.ts` |
| 338 | [plugins/mcpPluginIntegration.ts](./src/utils/plugins/mcpPluginIntegration.ts) | `.ts` |
| 339 | [plugins/mcpbHandler.ts](./src/utils/plugins/mcpbHandler.ts) | `.ts` |
| 340 | [plugins/officialMarketplace.ts](./src/utils/plugins/officialMarketplace.ts) | `.ts` |
| 341 | [plugins/officialMarketplaceGcs.ts](./src/utils/plugins/officialMarketplaceGcs.ts) | `.ts` |
| 342 | [plugins/officialMarketplaceStartupCheck.ts](./src/utils/plugins/officialMarketplaceStartupCheck.ts) | `.ts` |
| 343 | [plugins/orphanedPluginFilter.ts](./src/utils/plugins/orphanedPluginFilter.ts) | `.ts` |
| 344 | [plugins/parseMarketplaceInput.ts](./src/utils/plugins/parseMarketplaceInput.ts) | `.ts` |
| 345 | [plugins/performStartupChecks.tsx](./src/utils/plugins/performStartupChecks.tsx) | `.tsx` |
| 346 | [plugins/pluginAutoupdate.ts](./src/utils/plugins/pluginAutoupdate.ts) | `.ts` |
| 347 | [plugins/pluginBlocklist.ts](./src/utils/plugins/pluginBlocklist.ts) | `.ts` |
| 348 | [plugins/pluginDirectories.ts](./src/utils/plugins/pluginDirectories.ts) | `.ts` |
| 349 | [plugins/pluginFlagging.ts](./src/utils/plugins/pluginFlagging.ts) | `.ts` |
| 350 | [plugins/pluginIdentifier.ts](./src/utils/plugins/pluginIdentifier.ts) | `.ts` |
| 351 | [plugins/pluginInstallationHelpers.ts](./src/utils/plugins/pluginInstallationHelpers.ts) | `.ts` |
| 352 | [plugins/pluginLoader.ts](./src/utils/plugins/pluginLoader.ts) | `.ts` |
| 353 | [plugins/pluginOptionsStorage.ts](./src/utils/plugins/pluginOptionsStorage.ts) | `.ts` |
| 354 | [plugins/pluginPolicy.ts](./src/utils/plugins/pluginPolicy.ts) | `.ts` |
| 355 | [plugins/pluginStartupCheck.ts](./src/utils/plugins/pluginStartupCheck.ts) | `.ts` |
| 356 | [plugins/pluginVersioning.ts](./src/utils/plugins/pluginVersioning.ts) | `.ts` |
| 357 | [plugins/reconciler.ts](./src/utils/plugins/reconciler.ts) | `.ts` |
| 358 | [plugins/refresh.ts](./src/utils/plugins/refresh.ts) | `.ts` |
| 359 | [plugins/schemas.ts](./src/utils/plugins/schemas.ts) | `.ts` |
| 360 | [plugins/validatePlugin.ts](./src/utils/plugins/validatePlugin.ts) | `.ts` |
| 361 | [plugins/walkPluginMarkdown.ts](./src/utils/plugins/walkPluginMarkdown.ts) | `.ts` |
| 362 | [plugins/zipCache.ts](./src/utils/plugins/zipCache.ts) | `.ts` |
| 363 | [plugins/zipCacheAdapters.ts](./src/utils/plugins/zipCacheAdapters.ts) | `.ts` |
| 364 | [powershell/dangerousCmdlets.ts](./src/utils/powershell/dangerousCmdlets.ts) | `.ts` |
| 365 | [powershell/parser.ts](./src/utils/powershell/parser.ts) | `.ts` |
| 366 | [powershell/staticPrefix.ts](./src/utils/powershell/staticPrefix.ts) | `.ts` |
| 367 | [preflightChecks.tsx](./src/utils/preflightChecks.tsx) | `.tsx` |
| 368 | [privacyLevel.ts](./src/utils/privacyLevel.ts) | `.ts` |
| 369 | [process.ts](./src/utils/process.ts) | `.ts` |
| 370 | [processUserInput/processBashCommand.tsx](./src/utils/processUserInput/processBashCommand.tsx) | `.tsx` |
| 371 | [processUserInput/processSlashCommand.tsx](./src/utils/processUserInput/processSlashCommand.tsx) | `.tsx` |
| 372 | [processUserInput/processTextPrompt.ts](./src/utils/processUserInput/processTextPrompt.ts) | `.ts` |
| 373 | [processUserInput/processUserInput.ts](./src/utils/processUserInput/processUserInput.ts) | `.ts` |
| 374 | [profilerBase.ts](./src/utils/profilerBase.ts) | `.ts` |
| 375 | [promptCategory.ts](./src/utils/promptCategory.ts) | `.ts` |
| 376 | [promptEditor.ts](./src/utils/promptEditor.ts) | `.ts` |
| 377 | [promptShellExecution.ts](./src/utils/promptShellExecution.ts) | `.ts` |
| 378 | [proxy.ts](./src/utils/proxy.ts) | `.ts` |
| 379 | [queryContext.ts](./src/utils/queryContext.ts) | `.ts` |
| 380 | [queryHelpers.ts](./src/utils/queryHelpers.ts) | `.ts` |
| 381 | [queryProfiler.ts](./src/utils/queryProfiler.ts) | `.ts` |
| 382 | [queueProcessor.ts](./src/utils/queueProcessor.ts) | `.ts` |
| 383 | [readEditContext.ts](./src/utils/readEditContext.ts) | `.ts` |
| 384 | [readFileInRange.ts](./src/utils/readFileInRange.ts) | `.ts` |
| 385 | [releaseNotes.ts](./src/utils/releaseNotes.ts) | `.ts` |
| 386 | [renderOptions.ts](./src/utils/renderOptions.ts) | `.ts` |
| 387 | [ripgrep.ts](./src/utils/ripgrep.ts) | `.ts` |
| 388 | [sandbox/sandbox-adapter.ts](./src/utils/sandbox/sandbox-adapter.ts) | `.ts` |
| 389 | [sandbox/sandbox-ui-utils.ts](./src/utils/sandbox/sandbox-ui-utils.ts) | `.ts` |
| 390 | [sanitization.ts](./src/utils/sanitization.ts) | `.ts` |
| 391 | [screenshotClipboard.ts](./src/utils/screenshotClipboard.ts) | `.ts` |
| 392 | [sdkEventQueue.ts](./src/utils/sdkEventQueue.ts) | `.ts` |
| 393 | [secureStorage/fallbackStorage.ts](./src/utils/secureStorage/fallbackStorage.ts) | `.ts` |
| 394 | [secureStorage/index.ts](./src/utils/secureStorage/index.ts) | `.ts` |
| 395 | [secureStorage/keychainPrefetch.ts](./src/utils/secureStorage/keychainPrefetch.ts) | `.ts` |
| 396 | [secureStorage/macOsKeychainHelpers.ts](./src/utils/secureStorage/macOsKeychainHelpers.ts) | `.ts` |
| 397 | [secureStorage/macOsKeychainStorage.ts](./src/utils/secureStorage/macOsKeychainStorage.ts) | `.ts` |
| 398 | [secureStorage/plainTextStorage.ts](./src/utils/secureStorage/plainTextStorage.ts) | `.ts` |
| 399 | [semanticBoolean.ts](./src/utils/semanticBoolean.ts) | `.ts` |
| 400 | [semanticNumber.ts](./src/utils/semanticNumber.ts) | `.ts` |
| 401 | [semver.ts](./src/utils/semver.ts) | `.ts` |
| 402 | [sequential.ts](./src/utils/sequential.ts) | `.ts` |
| 403 | [sessionActivity.ts](./src/utils/sessionActivity.ts) | `.ts` |
| 404 | [sessionEnvVars.ts](./src/utils/sessionEnvVars.ts) | `.ts` |
| 405 | [sessionEnvironment.ts](./src/utils/sessionEnvironment.ts) | `.ts` |
| 406 | [sessionFileAccessHooks.ts](./src/utils/sessionFileAccessHooks.ts) | `.ts` |
| 407 | [sessionIngressAuth.ts](./src/utils/sessionIngressAuth.ts) | `.ts` |
| 408 | [sessionRestore.ts](./src/utils/sessionRestore.ts) | `.ts` |
| 409 | [sessionStart.ts](./src/utils/sessionStart.ts) | `.ts` |
| 410 | [sessionState.ts](./src/utils/sessionState.ts) | `.ts` |
| 411 | [sessionStorage.ts](./src/utils/sessionStorage.ts) | `.ts` |
| 412 | [sessionStoragePortable.ts](./src/utils/sessionStoragePortable.ts) | `.ts` |
| 413 | [sessionTitle.ts](./src/utils/sessionTitle.ts) | `.ts` |
| 414 | [sessionUrl.ts](./src/utils/sessionUrl.ts) | `.ts` |
| 415 | [set.ts](./src/utils/set.ts) | `.ts` |
| 416 | [settings/allErrors.ts](./src/utils/settings/allErrors.ts) | `.ts` |
| 417 | [settings/applySettingsChange.ts](./src/utils/settings/applySettingsChange.ts) | `.ts` |
| 418 | [settings/changeDetector.ts](./src/utils/settings/changeDetector.ts) | `.ts` |
| 419 | [settings/constants.ts](./src/utils/settings/constants.ts) | `.ts` |
| 420 | [settings/internalWrites.ts](./src/utils/settings/internalWrites.ts) | `.ts` |
| 421 | [settings/managedPath.ts](./src/utils/settings/managedPath.ts) | `.ts` |
| 422 | [settings/mdm/constants.ts](./src/utils/settings/mdm/constants.ts) | `.ts` |
| 423 | [settings/mdm/rawRead.ts](./src/utils/settings/mdm/rawRead.ts) | `.ts` |
| 424 | [settings/mdm/settings.ts](./src/utils/settings/mdm/settings.ts) | `.ts` |
| 425 | [settings/permissionValidation.ts](./src/utils/settings/permissionValidation.ts) | `.ts` |
| 426 | [settings/pluginOnlyPolicy.ts](./src/utils/settings/pluginOnlyPolicy.ts) | `.ts` |
| 427 | [settings/schemaOutput.ts](./src/utils/settings/schemaOutput.ts) | `.ts` |
| 428 | [settings/settings.ts](./src/utils/settings/settings.ts) | `.ts` |
| 429 | [settings/settingsCache.ts](./src/utils/settings/settingsCache.ts) | `.ts` |
| 430 | [settings/toolValidationConfig.ts](./src/utils/settings/toolValidationConfig.ts) | `.ts` |
| 431 | [settings/types.ts](./src/utils/settings/types.ts) | `.ts` |
| 432 | [settings/validateEditTool.ts](./src/utils/settings/validateEditTool.ts) | `.ts` |
| 433 | [settings/validation.ts](./src/utils/settings/validation.ts) | `.ts` |
| 434 | [settings/validationTips.ts](./src/utils/settings/validationTips.ts) | `.ts` |
| 435 | [shell/bashProvider.ts](./src/utils/shell/bashProvider.ts) | `.ts` |
| 436 | [shell/outputLimits.ts](./src/utils/shell/outputLimits.ts) | `.ts` |
| 437 | [shell/powershellDetection.ts](./src/utils/shell/powershellDetection.ts) | `.ts` |
| 438 | [shell/powershellProvider.ts](./src/utils/shell/powershellProvider.ts) | `.ts` |
| 439 | [shell/prefix.ts](./src/utils/shell/prefix.ts) | `.ts` |
| 440 | [shell/readOnlyCommandValidation.ts](./src/utils/shell/readOnlyCommandValidation.ts) | `.ts` |
| 441 | [shell/resolveDefaultShell.ts](./src/utils/shell/resolveDefaultShell.ts) | `.ts` |
| 442 | [shell/shellProvider.ts](./src/utils/shell/shellProvider.ts) | `.ts` |
| 443 | [shell/shellToolUtils.ts](./src/utils/shell/shellToolUtils.ts) | `.ts` |
| 444 | [shell/specPrefix.ts](./src/utils/shell/specPrefix.ts) | `.ts` |
| 445 | [shellConfig.ts](./src/utils/shellConfig.ts) | `.ts` |
| 446 | [sideQuery.ts](./src/utils/sideQuery.ts) | `.ts` |
| 447 | [sideQuestion.ts](./src/utils/sideQuestion.ts) | `.ts` |
| 448 | [signal.ts](./src/utils/signal.ts) | `.ts` |
| 449 | [sinks.ts](./src/utils/sinks.ts) | `.ts` |
| 450 | [skills/skillChangeDetector.ts](./src/utils/skills/skillChangeDetector.ts) | `.ts` |
| 451 | [slashCommandParsing.ts](./src/utils/slashCommandParsing.ts) | `.ts` |
| 452 | [sleep.ts](./src/utils/sleep.ts) | `.ts` |
| 453 | [sliceAnsi.ts](./src/utils/sliceAnsi.ts) | `.ts` |
| 454 | [slowOperations.ts](./src/utils/slowOperations.ts) | `.ts` |
| 455 | [standaloneAgent.ts](./src/utils/standaloneAgent.ts) | `.ts` |
| 456 | [startupProfiler.ts](./src/utils/startupProfiler.ts) | `.ts` |
| 457 | [staticRender.tsx](./src/utils/staticRender.tsx) | `.tsx` |
| 458 | [stats.ts](./src/utils/stats.ts) | `.ts` |
| 459 | [statsCache.ts](./src/utils/statsCache.ts) | `.ts` |
| 460 | [status.tsx](./src/utils/status.tsx) | `.tsx` |
| 461 | [statusNoticeDefinitions.tsx](./src/utils/statusNoticeDefinitions.tsx) | `.tsx` |
| 462 | [statusNoticeHelpers.ts](./src/utils/statusNoticeHelpers.ts) | `.ts` |
| 463 | [stream.ts](./src/utils/stream.ts) | `.ts` |
| 464 | [streamJsonStdoutGuard.ts](./src/utils/streamJsonStdoutGuard.ts) | `.ts` |
| 465 | [streamlinedTransform.ts](./src/utils/streamlinedTransform.ts) | `.ts` |
| 466 | [stringUtils.ts](./src/utils/stringUtils.ts) | `.ts` |
| 467 | [subprocessEnv.ts](./src/utils/subprocessEnv.ts) | `.ts` |
| 468 | [suggestions/commandSuggestions.ts](./src/utils/suggestions/commandSuggestions.ts) | `.ts` |
| 469 | [suggestions/directoryCompletion.ts](./src/utils/suggestions/directoryCompletion.ts) | `.ts` |
| 470 | [suggestions/shellHistoryCompletion.ts](./src/utils/suggestions/shellHistoryCompletion.ts) | `.ts` |
| 471 | [suggestions/skillUsageTracking.ts](./src/utils/suggestions/skillUsageTracking.ts) | `.ts` |
| 472 | [suggestions/slackChannelSuggestions.ts](./src/utils/suggestions/slackChannelSuggestions.ts) | `.ts` |
| 473 | [swarm/It2SetupPrompt.tsx](./src/utils/swarm/It2SetupPrompt.tsx) | `.tsx` |
| 474 | [swarm/backends/ITermBackend.ts](./src/utils/swarm/backends/ITermBackend.ts) | `.ts` |
| 475 | [swarm/backends/InProcessBackend.ts](./src/utils/swarm/backends/InProcessBackend.ts) | `.ts` |
| 476 | [swarm/backends/PaneBackendExecutor.ts](./src/utils/swarm/backends/PaneBackendExecutor.ts) | `.ts` |
| 477 | [swarm/backends/TmuxBackend.ts](./src/utils/swarm/backends/TmuxBackend.ts) | `.ts` |
| 478 | [swarm/backends/detection.ts](./src/utils/swarm/backends/detection.ts) | `.ts` |
| 479 | [swarm/backends/it2Setup.ts](./src/utils/swarm/backends/it2Setup.ts) | `.ts` |
| 480 | [swarm/backends/registry.ts](./src/utils/swarm/backends/registry.ts) | `.ts` |
| 481 | [swarm/backends/teammateModeSnapshot.ts](./src/utils/swarm/backends/teammateModeSnapshot.ts) | `.ts` |
| 482 | [swarm/backends/types.ts](./src/utils/swarm/backends/types.ts) | `.ts` |
| 483 | [swarm/constants.ts](./src/utils/swarm/constants.ts) | `.ts` |
| 484 | [swarm/inProcessRunner.ts](./src/utils/swarm/inProcessRunner.ts) | `.ts` |
| 485 | [swarm/leaderPermissionBridge.ts](./src/utils/swarm/leaderPermissionBridge.ts) | `.ts` |
| 486 | [swarm/permissionSync.ts](./src/utils/swarm/permissionSync.ts) | `.ts` |
| 487 | [swarm/reconnection.ts](./src/utils/swarm/reconnection.ts) | `.ts` |
| 488 | [swarm/spawnInProcess.ts](./src/utils/swarm/spawnInProcess.ts) | `.ts` |
| 489 | [swarm/spawnUtils.ts](./src/utils/swarm/spawnUtils.ts) | `.ts` |
| 490 | [swarm/teamHelpers.ts](./src/utils/swarm/teamHelpers.ts) | `.ts` |
| 491 | [swarm/teammateInit.ts](./src/utils/swarm/teammateInit.ts) | `.ts` |
| 492 | [swarm/teammateLayoutManager.ts](./src/utils/swarm/teammateLayoutManager.ts) | `.ts` |
| 493 | [swarm/teammateModel.ts](./src/utils/swarm/teammateModel.ts) | `.ts` |
| 494 | [swarm/teammatePromptAddendum.ts](./src/utils/swarm/teammatePromptAddendum.ts) | `.ts` |
| 495 | [systemDirectories.ts](./src/utils/systemDirectories.ts) | `.ts` |
| 496 | [systemPrompt.ts](./src/utils/systemPrompt.ts) | `.ts` |
| 497 | [systemPromptType.ts](./src/utils/systemPromptType.ts) | `.ts` |
| 498 | [systemTheme.ts](./src/utils/systemTheme.ts) | `.ts` |
| 499 | [taggedId.ts](./src/utils/taggedId.ts) | `.ts` |
| 500 | [task/TaskOutput.ts](./src/utils/task/TaskOutput.ts) | `.ts` |
| 501 | [task/diskOutput.ts](./src/utils/task/diskOutput.ts) | `.ts` |
| 502 | [task/framework.ts](./src/utils/task/framework.ts) | `.ts` |
| 503 | [task/outputFormatting.ts](./src/utils/task/outputFormatting.ts) | `.ts` |
| 504 | [task/sdkProgress.ts](./src/utils/task/sdkProgress.ts) | `.ts` |
| 505 | [tasks.ts](./src/utils/tasks.ts) | `.ts` |
| 506 | [teamDiscovery.ts](./src/utils/teamDiscovery.ts) | `.ts` |
| 507 | [teamMemoryOps.ts](./src/utils/teamMemoryOps.ts) | `.ts` |
| 508 | [teammate.ts](./src/utils/teammate.ts) | `.ts` |
| 509 | [teammateContext.ts](./src/utils/teammateContext.ts) | `.ts` |
| 510 | [teammateMailbox.ts](./src/utils/teammateMailbox.ts) | `.ts` |
| 511 | [telemetry/betaSessionTracing.ts](./src/utils/telemetry/betaSessionTracing.ts) | `.ts` |
| 512 | [telemetry/bigqueryExporter.ts](./src/utils/telemetry/bigqueryExporter.ts) | `.ts` |
| 513 | [telemetry/events.ts](./src/utils/telemetry/events.ts) | `.ts` |
| 514 | [telemetry/instrumentation.ts](./src/utils/telemetry/instrumentation.ts) | `.ts` |
| 515 | [telemetry/logger.ts](./src/utils/telemetry/logger.ts) | `.ts` |
| 516 | [telemetry/perfettoTracing.ts](./src/utils/telemetry/perfettoTracing.ts) | `.ts` |
| 517 | [telemetry/pluginTelemetry.ts](./src/utils/telemetry/pluginTelemetry.ts) | `.ts` |
| 518 | [telemetry/sessionTracing.ts](./src/utils/telemetry/sessionTracing.ts) | `.ts` |
| 519 | [telemetry/skillLoadedEvent.ts](./src/utils/telemetry/skillLoadedEvent.ts) | `.ts` |
| 520 | [telemetryAttributes.ts](./src/utils/telemetryAttributes.ts) | `.ts` |
| 521 | [teleport.tsx](./src/utils/teleport.tsx) | `.tsx` |
| 522 | [teleport/api.ts](./src/utils/teleport/api.ts) | `.ts` |
| 523 | [teleport/environmentSelection.ts](./src/utils/teleport/environmentSelection.ts) | `.ts` |
| 524 | [teleport/environments.ts](./src/utils/teleport/environments.ts) | `.ts` |
| 525 | [teleport/gitBundle.ts](./src/utils/teleport/gitBundle.ts) | `.ts` |
| 526 | [tempfile.ts](./src/utils/tempfile.ts) | `.ts` |
| 527 | [terminal.ts](./src/utils/terminal.ts) | `.ts` |
| 528 | [terminalPanel.ts](./src/utils/terminalPanel.ts) | `.ts` |
| 529 | [textHighlighting.ts](./src/utils/textHighlighting.ts) | `.ts` |
| 530 | [theme.ts](./src/utils/theme.ts) | `.ts` |
| 531 | [thinking.ts](./src/utils/thinking.ts) | `.ts` |
| 532 | [timeouts.ts](./src/utils/timeouts.ts) | `.ts` |
| 533 | [tmuxSocket.ts](./src/utils/tmuxSocket.ts) | `.ts` |
| 534 | [todo/types.ts](./src/utils/todo/types.ts) | `.ts` |
| 535 | [tokenBudget.ts](./src/utils/tokenBudget.ts) | `.ts` |
| 536 | [tokens.ts](./src/utils/tokens.ts) | `.ts` |
| 537 | [toolErrors.ts](./src/utils/toolErrors.ts) | `.ts` |
| 538 | [toolPool.ts](./src/utils/toolPool.ts) | `.ts` |
| 539 | [toolResultStorage.ts](./src/utils/toolResultStorage.ts) | `.ts` |
| 540 | [toolSchemaCache.ts](./src/utils/toolSchemaCache.ts) | `.ts` |
| 541 | [toolSearch.ts](./src/utils/toolSearch.ts) | `.ts` |
| 542 | [transcriptSearch.ts](./src/utils/transcriptSearch.ts) | `.ts` |
| 543 | [treeify.ts](./src/utils/treeify.ts) | `.ts` |
| 544 | [truncate.ts](./src/utils/truncate.ts) | `.ts` |
| 545 | [ultraplan/ccrSession.ts](./src/utils/ultraplan/ccrSession.ts) | `.ts` |
| 546 | [ultraplan/keyword.ts](./src/utils/ultraplan/keyword.ts) | `.ts` |
| 547 | [unaryLogging.ts](./src/utils/unaryLogging.ts) | `.ts` |
| 548 | [undercover.ts](./src/utils/undercover.ts) | `.ts` |
| 549 | [user.ts](./src/utils/user.ts) | `.ts` |
| 550 | [userAgent.ts](./src/utils/userAgent.ts) | `.ts` |
| 551 | [userPromptKeywords.ts](./src/utils/userPromptKeywords.ts) | `.ts` |
| 552 | [uuid.ts](./src/utils/uuid.ts) | `.ts` |
| 553 | [warningHandler.ts](./src/utils/warningHandler.ts) | `.ts` |
| 554 | [which.ts](./src/utils/which.ts) | `.ts` |
| 555 | [windowsPaths.ts](./src/utils/windowsPaths.ts) | `.ts` |
| 556 | [withResolvers.ts](./src/utils/withResolvers.ts) | `.ts` |
| 557 | [words.ts](./src/utils/words.ts) | `.ts` |
| 558 | [workloadContext.ts](./src/utils/workloadContext.ts) | `.ts` |
| 559 | [worktree.ts](./src/utils/worktree.ts) | `.ts` |
| 560 | [worktreeModeEnabled.ts](./src/utils/worktreeModeEnabled.ts) | `.ts` |
| 561 | [xdg.ts](./src/utils/xdg.ts) | `.ts` |
| 562 | [xml.ts](./src/utils/xml.ts) | `.ts` |
| 563 | [yaml.ts](./src/utils/yaml.ts) | `.ts` |
| 564 | [zodToJsonSchema.ts](./src/utils/zodToJsonSchema.ts) | `.ts` |

### vim/

| # | File | Type |
|:--|:-----|:-----|
| 1 | [motions.ts](./src/vim/motions.ts) | `.ts` |
| 2 | [operators.ts](./src/vim/operators.ts) | `.ts` |
| 3 | [textObjects.ts](./src/vim/textObjects.ts) | `.ts` |
| 4 | [transitions.ts](./src/vim/transitions.ts) | `.ts` |
| 5 | [types.ts](./src/vim/types.ts) | `.ts` |

### voice/

| # | File | Type |
|:--|:-----|:-----|
| 1 | [voiceModeEnabled.ts](./src/voice/voiceModeEnabled.ts) | `.ts` |


</details>

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
