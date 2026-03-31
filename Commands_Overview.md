---
tags:
  - commands
  - CLI
  - slash-commands
created: 2026-03-31
type: module-analysis
status: complete
---

# Commands Overview

> Claude Code의 CLI 슬래시 명령어 시스템. 100+ 내장 명령어와 스킬, 플러그인, 워크플로우

## 명령어 레지스트리 (`commands.ts`)

**핵심 함수:**
- `getCommands()` — 필터링된 명령어 목록 반환
- `findCommand()` / `hasCommand()` / `getCommand()` — 명령어 조회
- `getSkillToolCommands()` — 모델 실행 가능 명령어
- `getSlashCommandToolSkills()` — 슬래시 명령 스킬
- `getMcpSkillCommands()` — [[services/MCP_Service|MCP]] 제공 스킬

**명령어 소스:**
```
내장 명령어 (100+)
  ├── 스킬 디렉터리 명령어
  ├── 플러그인 명령어
  ├── 번들 스킬
  ├── 워크플로우
  ├── 동적 스킬 (파일 작업 중 발견)
  └── MCP 제공 명령어
```

## 명령어 카탈로그 (주요 80+)

### 세션 관리

| 명령어 | 디렉터리 | 용도 |
|--------|----------|------|
| `/session` | `session/` | 세션 관리 (저장/복원) |
| `/resume` | `resume/` | 이전 세션 재개 |
| `/clear` | `clear/` | 대화 초기화 |
| `/exit` | `exit/` | Claude Code 종료 |
| `/compact` | `compact/` | 대화 압축 |
| `/context` | `context/` | 컨텍스트 확인 |
| `/rewind` | `rewind/` | 대화 되감기 |

### 파일 및 코드 관리

| 명령어 | 용도 |
|--------|------|
| `/files` | 작업 중인 파일 목록 |
| `/add-dir` | 디렉터리 추가 |
| `/diff` | 변경사항 디프 표시 |
| `/copy` | 클립보드로 복사 |

### Git 관련

| 명령어 | 용도 |
|--------|------|
| `/branch` | 브랜치 관리 |
| `/commit-push-pr` | 커밋 + 푸시 + PR |
| `/pr_comments` | PR 코멘트 확인 |
| `/review` | 코드 리뷰 |
| `/issue` | 이슈 관리 |

### 모델 및 설정

| 명령어 | 용도 |
|--------|------|
| `/model` | 모델 변경 |
| `/fast` | 빠른 모드 토글 |
| `/effort` | 노력 수준 설정 |
| `/config` | 설정 관리 |
| `/permissions` | 권한 설정 |
| `/color` | 색상/테마 |
| `/theme` | 테마 변경 |
| `/output-style` | 출력 스타일 |
| `/keybindings` | 키바인딩 설정 |

### 도구 및 에이전트

| 명령어 | 용도 |
|--------|------|
| `/agents` | 에이전트 관리 |
| `/tasks` | 백그라운드 태스크 |
| `/mcp` | MCP 서버 관리 |
| `/plugin` | 플러그인 관리 |
| `/reload-plugins` | 플러그인 리로드 |
| `/skills` | 스킬 관리 |

### 정보 및 디버깅

| 명령어 | 용도 |
|--------|------|
| `/help` | 도움말 |
| `/version` | 버전 정보 |
| `/stats` | 세션 통계 |
| `/cost` | 비용 확인 |
| `/usage` | 사용량 확인 |
| `/status` | 상태 확인 |
| `/doctor` | 진단 도구 |
| `/env` | 환경 정보 |
| `/ctx_viz` | 컨텍스트 시각화 |

### 인증 및 공유

| 명령어 | 용도 |
|--------|------|
| `/login` | 로그인 |
| `/logout` | 로그아웃 |
| `/share` | 대화 공유 |
| `/export` | 대화 내보내기 |
| `/tag` | 세션 태깅 |
| `/feedback` | 피드백 |

### 특수 기능

| 명령어 | 용도 |
|--------|------|
| `/plan` | 플랜 모드 |
| `/vim` | Vim 모드 토글 |
| `/voice` | 음성 입력 |
| `/sandbox-toggle` | 샌드박스 토글 |
| `/memory` | 메모리 관리 |
| `/thinkback` | 사고 과정 되돌아보기 |
| `/teleport` | 텔레포트 |
| `/upgrade` | 업그레이드 |

### 내부/디버그

| 명령어 | 용도 |
|--------|------|
| `/heapdump` | 힙 덤프 |
| `/debug-tool-call` | 도구 호출 디버그 |
| `/break-cache` | 캐시 깨기 |
| `/mock-limits` | 리밋 모킹 |
| `/ant-trace` | 트레이스 |
| `/perf-issue` | 성능 이슈 |
| `/bughunter` | 버그 헌터 |

## [[Skills_Overview]] — 스킬 시스템

**구조:**
```
skills/
├── bundledSkills.ts       # 번들 스킬 레지스트리
├── mcpSkillBuilders.ts    # MCP 스킬 빌더
├── loadSkillsDir.ts       # 스킬 디렉터리 로더
└── bundled/
    ├── index.ts
    ├── claudeApi.ts       # Claude API 스킬
    ├── simplify.ts        # 코드 단순화
    ├── verify.ts          # 검증 스킬
    ├── loop.ts            # 반복 실행
    ├── debug.ts           # 디버그
    ├── stuck.ts           # 막힘 해결
    ├── batch.ts           # 배치 처리
    ├── updateConfig.ts    # 설정 업데이트
    ├── keybindings.ts     # 키바인딩
    ├── remember.ts        # 기억하기
    ├── scheduleRemoteAgents.ts # 원격 에이전트 스케줄링
    └── skillify.ts        # 스킬화
```

## 명령어 필터링

```
Feature flag 게이팅
  │
  ├── 인증 요구사항 체크 (claude.ai, console API)
  │
  ├── isEnabled() 체크
  │
  ├── 권한 기반 필터링
  │
  └── REMOTE_SAFE_COMMANDS / BRIDGE_SAFE_COMMANDS (원격/모바일 안전 집합)
```

## 통계

| 항목 | 값 |
|------|-----|
| 총 명령어 디렉터리 | 80+ |
| 총 파일 수 | 207 |
| 번들 스킬 | 15+ |
| 명령어 소스 | 6 (내장, 스킬, 플러그인, 번들, 워크플로우, MCP) |

## 관련 문서
- [[Index]] — 전체 MOC
- [[Tools_Overview]] — 도구 시스템 (모델 실행 가능)
- [[services/MCP_Service]] — MCP 제공 명령어
- [[services/Plugins_Service]] — 플러그인 명령어
