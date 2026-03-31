---
tags:
  - structure
  - overview
  - architecture
created: 2026-03-31
type: structure
status: complete
---

# Directory Structure

> Claude Code CLI 소스코드의 전체 디렉터리 트리 구조

## 프로젝트 개요

| 항목 | 값 |
|------|-----|
| 총 파일 수 | 1,902 |
| 주요 언어 | TypeScript / TSX |
| UI 프레임워크 | React (Ink 기반 터미널 렌더링) |
| 빌드 도구 | Bun |

## 최상위 구조

```
src/
├── entrypoints/          # 진입점 (CLI, SDK, MCP)
├── main.tsx              # 메인 부트스트랩
├── setup.ts              # 세션 셋업
├── Tool.ts               # 도구 베이스 인터페이스
├── tools.ts              # 도구 레지스트리
├── Task.ts               # 태스크 베이스 인터페이스
├── tasks.ts              # 태스크 레지스트리
├── query.ts              # 쿼리 실행 모듈
├── QueryEngine.ts        # 쿼리 엔진 오케스트레이션
├── context.ts            # 컨텍스트 생성
├── commands.ts           # 명령어 레지스트리
├── history.ts            # REPL 히스토리
├── ink.ts                # Ink 프레임워크 진입점
├── cost-tracker.ts       # 비용 추적
├── costHook.ts           # 비용 훅
├── replLauncher.tsx       # REPL 런처
├── interactiveHelpers.tsx # 인터랙티브 헬퍼
├── dialogLaunchers.tsx    # 다이얼로그 런처
├── projectOnboardingState.ts # 온보딩 상태
│
├── tools/                # 40개 도구 구현
├── services/             # 핵심 서비스 레이어
├── components/           # UI 컴포넌트 (389 files)
├── ink/                  # 커스텀 터미널 렌더링 프레임워크
├── hooks/                # React 훅 (104 files)
├── utils/                # 유틸리티 (564 files)
├── commands/             # CLI 명령어 (207 files)
├── skills/               # 스킬 시스템
├── context/              # 컨텍스트 관리
├── state/                # 상태 관리
├── constants/            # 상수 정의
├── types/                # 타입 정의
├── schemas/              # 스키마 정의
├── cli/                  # CLI 전송 레이어
├── bridge/               # IDE 브릿지
├── remote/               # 원격 세션
├── tasks/                # 백그라운드 태스크 구현
├── coordinator/          # 코디네이터 (멀티에이전트)
├── buddy/                # 버디 시스템
├── vim/                  # Vim 모드
├── voice/                # 음성 입력
├── plugins/              # 플러그인 시스템
├── keybindings/          # 키바인딩
├── migrations/           # 마이그레이션
├── memdir/               # 메모리 디렉터리
├── moreright/            # 모어라이트 패널
├── outputStyles/         # 출력 스타일
├── screens/              # 화면 레이아웃
├── server/               # 서버 모듈
├── native-ts/            # 네이티브 TS 바인딩
├── upstreamproxy/        # 업스트림 프록시
├── bootstrap/            # 부트스트랩
├── query/                # 쿼리 설정
└── assistant/            # 어시스턴트 세션
```

## 디렉터리별 파일 수 분포

| 디렉터리 | 파일 수 | 비율 | 설명 |
|-----------|---------|------|------|
| [[utils]] | 564 | 29.7% | 핵심 유틸리티 라이브러리 |
| [[components]] | 389 | 20.5% | UI 컴포넌트 |
| [[commands]] | 207 | 10.9% | CLI 슬래시 명령어 |
| [[tools]] | 184 | 9.7% | 도구 시스템 |
| [[services]] | 130 | 6.8% | 서비스 레이어 |
| [[hooks]] | 104 | 5.5% | React 훅 |
| [[ink]] | 96 | 5.0% | 터미널 렌더링 |
| [[bridge]] | 31 | 1.6% | IDE 브릿지 |
| [[constants]] | 21 | 1.1% | 상수 |
| [[skills]] | 20 | 1.1% | 스킬 시스템 |
| 기타 | 156 | 8.2% | 나머지 모듈 |

## 주요 하위 디렉터리 상세

### [[tools]] (40개 도구)

```
tools/
├── AgentTool/          # 서브에이전트 생성
├── BashTool/           # 셸 명령 실행
├── FileReadTool/       # 파일 읽기
├── FileEditTool/       # 파일 편집
├── FileWriteTool/      # 파일 쓰기
├── GlobTool/           # 파일 패턴 매칭
├── GrepTool/           # 콘텐츠 검색
├── WebFetchTool/       # 웹 페이지 가져오기
├── WebSearchTool/      # 웹 검색
├── MCPTool/            # MCP 도구 실행
├── LSPTool/            # 언어 서버 프로토콜
├── SkillTool/          # 스킬 실행
├── TaskCreateTool/     # 태스크 생성
├── TaskUpdateTool/     # 태스크 업데이트
├── TaskListTool/       # 태스크 목록
├── SendMessageTool/    # 에이전트 간 메시지
├── ScheduleCronTool/   # 크론 스케줄링
├── NotebookEditTool/   # 주피터 노트북 편집
├── ConfigTool/         # 설정 관리
├── EnterWorktreeTool/  # 워크트리 진입
├── ExitWorktreeTool/   # 워크트리 종료
├── EnterPlanModeTool/  # 플랜 모드 진입
├── ExitPlanModeTool/   # 플랜 모드 종료
├── RemoteTriggerTool/  # 원격 트리거
├── shared/             # 공유 유틸리티
└── ... (기타 도구)
```

### [[services]] (15개 서비스)

```
services/
├── api/                # Claude API 클라이언트
├── mcp/                # MCP 서버 연결 관리
├── compact/            # 컨텍스트 압축
├── tools/              # 도구 실행 오케스트레이션
├── lsp/                # LSP 서버 관리
├── oauth/              # OAuth 인증
├── analytics/          # 분석 및 텔레메트리
├── SessionMemory/      # 세션 메모리
├── extractMemories/    # 메모리 추출
├── plugins/            # 플러그인 관리
├── tips/               # 팁 시스템
├── policyLimits/       # 정책 제한
├── remoteManagedSettings/ # 원격 설정 관리
├── settingsSync/       # 설정 동기화
├── teamMemorySync/     # 팀 메모리 동기화
├── autoDream/          # 자동 드림
├── PromptSuggestion/   # 프롬프트 제안
├── MagicDocs/          # 매직 독스
└── toolUseSummary/     # 도구 사용 요약
```

### [[utils]] (주요 하위 모듈)

```
utils/
├── bash/               # Bash 명령 파싱 및 보안
├── git/                # Git 상태 읽기
├── permissions/        # 권한 시스템 (26 files)
├── settings/           # 설정 관리 (19 files)
├── model/              # 모델 선택 (18 files)
├── shell/              # 셸 추상화 (12 files)
├── telemetry/          # 텔레메트리 (14 files)
├── plugins/            # 플러그인 유틸리티
├── swarm/              # 분산 에이전트
├── messages/           # 메시지 처리
├── memory/             # 메모리 타입
├── sandbox/            # 샌드박싱
├── task/               # 태스크 관리
├── hooks/              # 훅 유틸리티
├── skills/             # 스킬 유틸리티
├── github/             # GitHub 통합
├── computerUse/        # 컴퓨터 사용
├── deepLink/           # 딥링크
├── dxt/                # DXT 확장
├── filePersistence/    # 파일 영속성
├── nativeInstaller/    # 네이티브 설치
├── processUserInput/   # 사용자 입력 처리
├── secureStorage/      # 보안 저장소
├── suggestions/        # 제안
├── ultraplan/          # 울트라플랜
└── background/         # 백그라운드 처리
```

## 관련 문서

- [[Index]] - 전체 프로젝트 지도(MOC)
- [[Stats_Report]] - 통계 리포트
- [[tools/Tools_Overview]] - 도구 시스템 개요
- [[services/Services_Overview]] - 서비스 레이어 개요
