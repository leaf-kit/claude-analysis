---
tags:
  - MOC
  - index
  - architecture
created: 2026-03-31
type: MOC
status: complete
---

# Claude Code CLI - Map of Content (MOC)

> Claude Code는 Anthropic의 공식 CLI 기반 AI 코딩 어시스턴트입니다.
> 이 문서는 전체 소스코드의 **지식 지도(Map of Content)** 입니다.

## 아키텍처 개요

```
┌─────────────────────────────────────────────────────┐
│                   사용자 입력                         │
│              (터미널 / IDE / 웹)                      │
└───────────────┬─────────────────────────────────────┘
                │
┌───────────────▼─────────────────────────────────────┐
│           [[Entrypoints_Overview]]                   │
│     cli.tsx → init.ts → main.tsx → setup.ts          │
└───────────────┬─────────────────────────────────────┘
                │
┌───────────────▼─────────────────────────────────────┐
│              [[REPL_Screen]]                         │
│     PromptInput → Commands → QueryEngine             │
└───────┬───────────────┬───────────────┬─────────────┘
        │               │               │
┌───────▼───────┐ ┌─────▼─────┐ ┌───────▼───────┐
│ [[Query_Engine]]│ │[[Commands]]│ │[[Ink_Framework]]│
│  query.ts     │ │commands.ts│ │ 터미널 렌더링   │
│  QueryEngine  │ │ 100+개    │ │ React 기반     │
└───────┬───────┘ └───────────┘ └───────────────┘
        │
┌───────▼──────────────────────────────────────────┐
│              [[Tools_Overview]]                    │
│  40개 도구: Bash, File*, Glob, Grep, Agent, ...   │
│  Tool.ts (인터페이스) → tools.ts (레지스트리)      │
└───────┬──────────────────────────────────────────┘
        │
┌───────▼──────────────────────────────────────────┐
│            [[Services_Overview]]                   │
│  API, MCP, Compact, LSP, OAuth, Analytics, ...    │
└───────┬──────────────────────────────────────────┘
        │
┌───────▼──────────────────────────────────────────┐
│            [[Utils_Overview]]                      │
│  permissions, settings, model, bash, git, ...     │
└──────────────────────────────────────────────────┘
```

## 핵심 문서 링크

### 구조 문서
- [[Directory_Structure]] - 전체 디렉터리 트리 구조
- [[Stats_Report]] - 의존성 통계 및 모듈 분포 분석

### 시스템 레이어

#### 1. 진입점 및 부트스트랩
- [[Entrypoints_Overview]] - 앱 시작 흐름 (cli.tsx → init.ts → main.tsx → setup.ts)

#### 2. 쿼리 엔진 (핵심 실행 루프)
- [[Query_Engine]] - QueryEngine.ts와 query.ts의 실행 흐름
- [[Context_Module]] - 시스템/사용자 컨텍스트 생성

#### 3. 도구 시스템
- [[Tools_Overview]] - 40개 도구 전체 개요
- [[tools/BashTool]] - 셸 명령 실행
- [[tools/FileReadTool]] - 파일 읽기
- [[tools/FileEditTool]] - 파일 편집 (문자열 치환)
- [[tools/FileWriteTool]] - 파일 생성/덮어쓰기
- [[tools/GlobTool]] - 파일 패턴 매칭
- [[tools/GrepTool]] - 콘텐츠 검색 (ripgrep)
- [[tools/AgentTool]] - 서브에이전트 생성
- [[tools/WebFetchTool]] - 웹 콘텐츠 가져오기
- [[tools/WebSearchTool]] - 웹 검색
- [[tools/MCPTool]] - MCP 도구 실행
- [[tools/LSPTool]] - 언어 서버 프로토콜
- [[tools/SkillTool]] - 스킬 실행
- [[tools/TaskTools]] - 태스크 관리 도구 (Create/Update/List/Get/Stop)
- [[tools/SendMessageTool]] - 에이전트 간 통신
- [[tools/ScheduleCronTool]] - 크론 스케줄링

#### 4. 서비스 레이어
- [[Services_Overview]] - 서비스 전체 개요
- [[services/API_Service]] - Claude API 클라이언트 (멀티프로바이더)
- [[services/MCP_Service]] - MCP 서버 연결 관리
- [[services/Compact_Service]] - 컨텍스트 윈도우 압축
- [[services/Tools_Service]] - 도구 실행 오케스트레이션
- [[services/LSP_Service]] - LSP 서버 관리
- [[services/OAuth_Service]] - OAuth 2.0 인증
- [[services/Analytics_Service]] - 분석 및 이벤트 로깅
- [[services/SessionMemory_Service]] - 세션 메모리 관리
- [[services/ExtractMemories_Service]] - 자동 메모리 추출
- [[services/Plugins_Service]] - 플러그인 라이프사이클

#### 5. UI 컴포넌트
- [[Components_Overview]] - 컴포넌트 전체 개요
- [[components/Messages]] - 메시지 표시 시스템
- [[components/Permissions]] - 권한 요청 다이얼로그
- [[components/PromptInput]] - 프롬프트 입력 시스템
- [[components/DesignSystem]] - 디자인 시스템
- [[components/Diff]] - 디프 뷰어

#### 6. Ink 프레임워크 (터미널 렌더링)
- [[Ink_Framework]] - Ink 프레임워크 전체 개요
- [[ink/Rendering_Pipeline]] - 렌더링 파이프라인
- [[ink/Layout_System]] - Yoga 기반 Flexbox 레이아웃
- [[ink/Event_System]] - 이벤트 시스템
- [[ink/Components]] - Ink 기본 컴포넌트 (Box, Text, ScrollBox)
- [[ink/Hooks]] - Ink 커스텀 훅

#### 7. 유틸리티
- [[Utils_Overview]] - 유틸리티 전체 개요
- [[utils/Permissions_System]] - 권한 시스템 (26 files)
- [[utils/Settings_Management]] - 설정 관리
- [[utils/Model_Management]] - 모델 선택 및 관리
- [[utils/Bash_Utils]] - Bash 명령 파싱/보안
- [[utils/Git_Utils]] - Git 상태 읽기
- [[utils/Shell_Utils]] - 셸 추상화
- [[utils/Telemetry]] - 텔레메트리 시스템

#### 8. 훅 시스템
- [[Hooks_Overview]] - 훅 전체 개요
- [[hooks/ToolPermission]] - 도구 권한 훅
- [[hooks/TaskManagement]] - 태스크 관리 훅

#### 9. 명령어 시스템
- [[Commands_Overview]] - 100+ CLI 슬래시 명령어
- [[Skills_Overview]] - 스킬 시스템

## 핵심 데이터 흐름

### 1. 초기화 흐름
```
cli.tsx (빠른 부트스트랩)
  → init.ts (환경 설정, OAuth, LSP, 정책)
    → main.tsx (텔레메트리, 서비스 초기화)
      → setup.ts (워크트리, 훅, 권한 검증)
        → REPL (대화형 루프 시작)
```

### 2. 쿼리 실행 흐름
```
사용자 입력
  → [[Query_Engine]] (QueryEngine.ts - 라이프사이클 관리)
    → [[Context_Module]] (context.ts - 시스템/사용자 컨텍스트)
    → [[Query_Module]] (query.ts - 모델 호출 + 도구 실행)
      → [[Services_Overview|API Service]] (API 호출)
      → [[Tools_Overview|StreamingToolExecutor]] (도구 병렬 실행)
        → [[Permissions_System]] (권한 체크)
        → 도구 실행 결과 반환
      → [[Compact_Service]] (컨텍스트 압축 판단)
    → 응답 렌더링
```

### 3. 도구 실행 흐름
```
모델이 도구 호출 요청
  → tools.ts (도구 풀에서 도구 찾기)
    → Tool.checkPermissions() (권한 확인)
      → [[Permissions_System]] (허용/거부/문의)
    → Tool.call() (도구 실행)
    → 결과를 메시지에 추가
```

### 4. MCP 연결 흐름
```
설정에서 MCP 서버 정의
  → [[MCP_Service]] (서버 연결/인증)
    → 도구/리소스 발견
      → tools.ts에 MCP 도구 병합
      → 사용자가 MCP 도구 사용 가능
```

#### 10. 부트스트랩 & 글로벌 상태
- [[bootstrap/state]] - 글로벌 세션 상태 싱글톤 (209 getter/setter)

#### 11. 코디네이터 (멀티에이전트)
- [[coordinatorMode]] - 멀티 워커 에이전트 오케스트레이션

#### 12. 키바인딩 시스템
- [[Keybindings_Overview]] - 커스텀 키보드 단축키 (코드 시퀀스 지원)

#### 13. 메모리 시스템
- [[Memdir_Overview]] - 파일 기반 영속 메모리 (AI 선택, 팀 메모리)

#### 14. 마이그레이션
- [[Migrations_Overview]] - 설정/모델 마이그레이션 11개

#### 15. 네이티브 TS 포트
- [[NativeTS_Overview]] - color-diff, file-index, yoga-layout 포트

#### 16. 화면 시스템
- [[Screens_Overview]] - REPL, 세션 재개, 진단 화면

#### 17. 서버 & 네트워킹
- [[Server_Overview]] - WebSocket 직접 연결 세션 관리
- [[UpstreamProxy_Overview]] - CCR용 CONNECT-over-WebSocket 릴레이

#### 18. 기타 모듈
- [[sessionHistory]] - 원격 세션 히스토리 페이지네이션
- [[Buddy_Overview]] - 컴패니언 캐릭터 시스템
- [[Plugins_Module]] - 내장 플러그인 레지스트리
- [[hooks_schema]] - 훅 Zod 스키마 정의
- [[loadOutputStylesDir]] - 출력 스타일 로더
- [[voiceModeEnabled]] - 음성 모드 피처 게이트
- [[useMoreRight]] - REPL More Right 스텁

#### 19. 루트 레벨 파일
- [[Root_Files_Overview]] - src/ 루트 18개 핵심 파일 분석

## 설계 원칙

1. **레이어드 아키텍처**: 진입점 → 엔진 → 쿼리 → 도구 → 서비스
2. **권한 우선**: 모든 도구 실행 전 권한 체크 (`checkPermissions`)
3. **확장 가능성**: MCP, 플러그인, 스킬을 통한 기능 확장
4. **React 기반 터미널 UI**: Ink 프레임워크를 통한 선언적 UI
5. **멀티에이전트 지원**: AgentTool, Swarm, Coordinator 패턴
6. **멀티프로바이더 API**: Anthropic, Bedrock, Vertex AI, Azure Foundry

## 관련 문서
- [[Directory_Structure]] - 디렉터리 트리
- [[Stats_Report]] - 통계 리포트
