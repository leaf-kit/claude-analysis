---
tags:
  - stats
  - report
  - dependencies
  - metrics
created: 2026-03-31
type: report
status: complete
---

# Stats Report — 프로젝트 통계 리포트

> Claude Code CLI 소스코드의 의존성 통계 및 모듈 분포 분석

## 1. 프로젝트 규모 요약

| 지표 | 값 |
|------|-----|
| **총 파일 수** | 1,902 |
| **총 디렉터리 수** | ~160 |
| **주요 언어** | TypeScript (.ts, .tsx) |
| **UI 프레임워크** | React (Ink 터미널 렌더링) |
| **빌드 도구** | Bun |
| **런타임** | Node.js / Bun |

## 2. 모듈별 파일 분포

### 상위 레벨 모듈

| 순위 | 모듈 | 파일 수 | 비율 | 유형 |
|------|------|---------|------|------|
| 1 | [[Utils_Overview\|utils/]] | 564 | 29.7% | 인프라 유틸리티 |
| 2 | [[Components_Overview\|components/]] | 389 | 20.5% | UI 컴포넌트 |
| 3 | [[Commands_Overview\|commands/]] | 207 | 10.9% | CLI 명령어 |
| 4 | [[Tools_Overview\|tools/]] | 184 | 9.7% | 도구 시스템 |
| 5 | [[Services_Overview\|services/]] | 130 | 6.8% | 서비스 레이어 |
| 6 | [[Hooks_Overview\|hooks/]] | 104 | 5.5% | React 훅 |
| 7 | [[Ink_Framework\|ink/]] | 96 | 5.0% | 터미널 렌더링 |
| 8 | bridge/ | 31 | 1.6% | IDE 브릿지 |
| 9 | constants/ | 21 | 1.1% | 상수 정의 |
| 10 | [[Skills_Overview\|skills/]] | 20 | 1.1% | 스킬 시스템 |
| 11 | [[cli]] | 19 | 1.0% | CLI 출력/파싱 |
| 12 | [[Keybindings_Overview\|keybindings/]] | 14 | 0.7% | 키바인딩 시스템 |
| 13 | [[tasks]] | 12 | 0.6% | 태스크 실행 |
| 14 | [[Migrations_Overview\|migrations/]] | 11 | 0.6% | 설정 마이그레이션 |
| 15 | [[types]] | 11 | 0.6% | 타입 정의 |
| 16 | [[Context_Module\|context/]] | 9 | 0.5% | 컨텍스트 관리 |
| 17 | [[Memdir_Overview\|memdir/]] | 8 | 0.4% | 메모리 시스템 |
| 18 | [[Entrypoints_Overview\|entrypoints/]] | 8 | 0.4% | 진입점 |
| 19 | [[Buddy_Overview\|buddy/]] | 6 | 0.3% | 컴패니언 캐릭터 |
| 20 | [[state]] | 6 | 0.3% | 앱 상태 |
| 21 | [[vim]] | 5 | 0.3% | Vim 모드 |
| 22 | [[NativeTS_Overview\|native-ts/]] | 4 | 0.2% | TS 네이티브 포트 |
| — | 기타 12개 | 24 | 1.3% | 나머지 |

### 모듈 분포 시각화

```
utils       ████████████████████████████████████████ 564
components  ██████████████████████████████           389
commands    ████████████████                         207
tools       █████████████                            184
services    █████████                                130
hooks       ███████                                  104
ink         ██████                                    96
bridge      ██                                        31
constants   █                                         21
skills      █                                         20
```

## 3. 유틸리티 하위 모듈 분포

| 모듈 | 파일 수 | 용도 |
|------|---------|------|
| [[utils/Permissions_System\|permissions/]] | 26 | 권한 시스템 |
| [[utils/Settings_Management\|settings/]] | 19 | 설정 관리 |
| [[utils/Model_Management\|model/]] | 18 | 모델 관리 |
| [[utils/Bash_Utils\|bash/]] | 15+ | Bash 파싱/보안 |
| [[utils/Telemetry\|telemetry/]] | 14 | 텔레메트리 |
| [[utils/Shell_Utils\|shell/]] | 12 | 셸 추상화 |
| plugins/ | 46+ | 플러그인 |
| swarm/ | 16+ | 분산 에이전트 |
| [[utils/Git_Utils\|git/]] | 5 | Git 유틸 |
| task/ | 5 | 태스크 관리 |

## 4. 도구 시스템 통계

### 도구 카테고리 분포

| 카테고리 | 도구 수 | 비율 |
|----------|---------|------|
| 태스크 관리 | 6 | 15.0% |
| 파일 시스템 | 5 | 12.5% |
| MCP | 4 | 10.0% |
| 에이전트/멀티에이전트 | 3 | 7.5% |
| 스케줄링 | 3 | 7.5% |
| 웹 | 2 | 5.0% |
| 셸 실행 | 2 | 5.0% |
| 워크트리/플랜 | 4 | 10.0% |
| 코드 인텔리전스 | 1 | 2.5% |
| 기타 특수 | 10 | 25.0% |
| **합계** | **40** | **100%** |

### 도구 속성 분포

| 속성 | 읽기전용 | 쓰기 가능 |
|------|----------|-----------|
| 파일 시스템 | 3 (Glob, Grep, Read) | 2 (Edit, Write) |
| 태스크 | 3 (List, Get, Output) | 3 (Create, Update, Stop) |
| 웹 | 2 (Fetch, Search) | 0 |
| 코드 인텔리전스 | 1 (LSP) | 0 |

## 5. 서비스 레이어 통계

| 서비스 | 파일 수 | 복잡도 | 핵심 의존성 |
|--------|---------|--------|------------|
| [[services/API_Service\|api/]] | 16 | 높음 | SDK 4종 |
| [[services/MCP_Service\|mcp/]] | 23 | 높음 | MCP SDK |
| [[services/Compact_Service\|compact/]] | 11 | 중간 | API, 메시지 |
| tools/ | 4 | 중간 | 도구 시스템 |
| [[services/LSP_Service\|lsp/]] | 7 | 중간 | LSP 프로토콜 |
| [[services/OAuth_Service\|oauth/]] | 5 | 중간 | HTTP |
| [[services/Analytics_Service\|analytics/]] | 9 | 중간 | DD, GrowthBook |
| SessionMemory/ | 3 | 낮음 | 포크 에이전트 |
| extractMemories/ | 2 | 낮음 | 포크 에이전트 |
| plugins/ | 3 | 중간 | 마켓플레이스 |
| tips/ | 3 | 낮음 | 파일 히스토리 |

## 6. UI 컴포넌트 통계

| 카테고리 | 파일 수 | 복잡도 |
|----------|---------|--------|
| 메시지 | 36 | 높음 |
| 권한 다이얼로그 | 32 | 높음 |
| 디자인 시스템 | 18 | 중간 |
| 에이전트 UI | 16 | 중간 |
| MCP UI | 15 | 중간 |
| 태스크 UI | 10+ | 중간 |
| 프롬프트 입력 | 8 | 높음 |
| 디프 뷰어 | 3 | 중간 |

## 7. 의존성 그래프 (모듈 간 연결)

### 핵심 의존 관계

```
Entrypoints ──→ QueryEngine ──→ query.ts
                    │              │
                    ▼              ▼
               tools.ts ◄──── Tool.ts
                    │
    ┌───────────────┼───────────────┐
    ▼               ▼               ▼
services/api   services/mcp   services/tools
    │               │               │
    ▼               ▼               ▼
utils/model   utils/settings  utils/permissions
```

### 연결 밀도 (위키링크 기준)

| 모듈 | 들어오는 링크 | 나가는 링크 | 총 연결 |
|------|--------------|------------|---------|
| [[Tools_Overview]] | 12 | 8 | 20 |
| [[Query_Engine]] | 8 | 10 | 18 |
| [[utils/Permissions_System]] | 10 | 4 | 14 |
| [[Services_Overview]] | 8 | 6 | 14 |
| [[Index]] | 0 | 25 | 25 |
| [[services/API_Service]] | 6 | 4 | 10 |
| [[services/MCP_Service]] | 5 | 3 | 8 |
| [[Ink_Framework]] | 4 | 3 | 7 |

## 8. 기술 스택 분석

### 핵심 외부 의존성

| 패키지 | 용도 | 사용 모듈 |
|--------|------|-----------|
| `@anthropic-ai/sdk` | Claude API | [[services/API_Service]] |
| `@anthropic-ai/bedrock-sdk` | AWS Bedrock | [[services/API_Service]] |
| `@anthropic-ai/vertex-sdk` | Google Vertex | [[services/API_Service]] |
| `@anthropic-ai/foundry-sdk` | Azure Foundry | [[services/API_Service]] |
| `@modelcontextprotocol/sdk` | MCP 프로토콜 | [[services/MCP_Service]] |
| `react` | UI 프레임워크 | [[Components_Overview]], [[Ink_Framework]] |
| `zod` (v4) | 스키마 검증 | [[Tools_Overview]] 전체 |
| `yoga-layout` | Flexbox 레이아웃 | [[Ink_Framework]] |
| `vscode-languageserver-protocol` | LSP | [[services/LSP_Service]], [[tools/LSPTool]] |
| `vscode-jsonrpc` | JSON-RPC | [[services/MCP_Service]], [[services/LSP_Service]] |
| `tree-sitter` | AST 파싱 | [[utils/Bash_Utils]] |
| `@withfig/autocomplete` | 셸 자동완성 | [[utils/Bash_Utils]] |

### 설계 패턴 분포

| 패턴 | 사용 빈도 | 주요 위치 |
|------|-----------|-----------|
| Factory | 높음 | Tool.ts (`buildTool()`), LSP (`createLSPServerManager()`) |
| Async Generator | 높음 | [[services/Tools_Service]] (`runTools()`) |
| React Hooks | 높음 | [[Hooks_Overview]], [[Components_Overview]] |
| Singleton Store | 중간 | `useTasksV2`, 캐시 |
| Observer/Event | 중간 | [[Ink_Framework]], 파일 감시 |
| Strategy | 중간 | 권한 핸들러, 셸 프로바이더 |
| Memoization | 높음 | `context.ts`, `commands.ts` |
| Race Condition | 중간 | 권한 훅 (훅 vs 사용자) |
| Forked Agent | 낮음 | SessionMemory, ExtractMemories |

## 9. 보안 아키텍처 통계

| 보안 계층 | 파일 수 | 기법 |
|-----------|---------|------|
| 권한 시스템 | 26 | 규칙 기반 + AI 분류 |
| Bash 보안 | 15+ | AST 파싱, 인젝션 방지 |
| 셸 검증 | 12 | 읽기전용 명령 레지스트리 |
| 경로 검증 | 5+ | 안전 경로 체크 |
| 샌드박싱 | 2 | @anthropic-ai/sandbox-runtime |

## 10. 아키텍처 품질 지표

### 모듈 응집도

| 모듈 | 응집도 | 이유 |
|------|--------|------|
| [[Ink_Framework]] | ★★★★★ | 독립적 프레임워크, 명확한 경계 |
| [[Tools_Overview]] | ★★★★☆ | 일관된 인터페이스, 도구별 격리 |
| [[Services_Overview]] | ★★★★☆ | 서비스별 명확한 책임 |
| [[Utils_Overview]] | ★★★☆☆ | 다양한 유틸, 일부 크로스커팅 |
| [[Components_Overview]] | ★★★☆☆ | 대형 컴포넌트 존재 |

### 계층 분리도

```
진입점 (Entrypoints) ← 최상위
  │
  ▼
엔진 (QueryEngine, Query) ← 오케스트레이션
  │
  ▼
도구 (Tools) ← 실행 단위
  │
  ▼
서비스 (Services) ← 비즈니스 로직
  │
  ▼
유틸리티 (Utils) ← 기반 라이브러리
```

**계층 위반:** 최소 — 대체로 단방향 의존성 유지

## 11. 신규 분석 모듈 요약 (2차 분석)

| 모듈 | 파일 수 | 줄 수 | 핵심 역할 |
|------|---------|-------|-----------|
| [[bootstrap/state\|bootstrap]] | 1 | 1,758 | 글로벌 상태 싱글톤 (209 getter/setter) |
| [[sessionHistory\|assistant]] | 1 | 87 | 원격 세션 히스토리 페이지네이션 |
| [[Buddy_Overview\|buddy]] | 6 | 1,298 | 컴패니언 캐릭터 절차적 생성 |
| [[coordinatorMode\|coordinator]] | 1 | 369 | 멀티워커 에이전트 오케스트레이션 |
| [[Keybindings_Overview\|keybindings]] | 14 | 3,159 | 커스텀 키바인딩 + 코드 시퀀스 |
| [[Memdir_Overview\|memdir]] | 8 | 1,736 | 파일 기반 영속 메모리 (AI 선택) |
| [[Migrations_Overview\|migrations]] | 11 | 603 | 설정/모델 마이그레이션 |
| [[useMoreRight\|moreright]] | 1 | 25 | 외부 빌드용 REPL 스텁 |
| [[NativeTS_Overview\|native-ts]] | 4 | 4,081 | Rust/C++ 네이티브 모듈 TS 포트 |
| [[loadOutputStylesDir\|outputStyles]] | 1 | 98 | 출력 스타일 로더 |
| [[Plugins_Module\|plugins]] | 2 | 182 | 내장 플러그인 레지스트리 |
| [[hooks_schema\|schemas]] | 1 | 222 | 훅 Zod 스키마 (순환 해결) |
| [[Screens_Overview\|screens]] | 3 | 5,977 | REPL, 세션재개, 진단 화면 |
| [[Server_Overview\|server]] | 3 | 358 | WebSocket 직접 연결 |
| [[UpstreamProxy_Overview\|upstreamproxy]] | 2 | 740 | CCR용 CONNECT-over-WS 릴레이 |
| [[voiceModeEnabled\|voice]] | 1 | 54 | 음성 모드 피처 게이트 |

### 루트 레벨 파일 (18개, 11,968줄)
→ [[Root_Files_Overview]] 참조

## 관련 문서
- [[Index]] — 전체 MOC
- [[Directory_Structure]] — 디렉터리 구조
- [[Root_Files_Overview]] — 루트 파일 분석
