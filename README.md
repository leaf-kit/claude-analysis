# Claude Code CLI - Source Code Analysis

> **Claude Code** 소스코드를 분석하여 Obsidian 최적화 지식 베이스로 구축한 프로젝트입니다.

## Overview

이 저장소는 Anthropic의 공식 CLI 도구인 **Claude Code**의 소스코드(`src/` 디렉터리)를 정적 분석하고,
그 결과를 Obsidian에서 그래프 뷰와 함께 탐색할 수 있는 마크다운 문서 세트로 정리한 것입니다.

### 주요 분석 결과

| 지표 | 값 |
|------|-----|
| 총 파일 수 | ~1,902 |
| 주요 언어 | TypeScript (.ts, .tsx) |
| UI 프레임워크 | React (Ink 터미널 렌더링) |
| 빌드/런타임 | Bun / Node.js |
| 도구(Tools) | 40+ |
| 명령어(Commands) | 100+ |

## Architecture

```
사용자 입력 (터미널 / IDE / 웹)
        │
   Entrypoints (cli.tsx → init.ts → main.tsx)
        │
   REPL Screen (PromptInput → Commands → QueryEngine)
        │
   Tools (Bash, File, Glob, Grep, Agent, ...)
        │
   Services (API, MCP, Compact, LSP, OAuth, ...)
        │
   State & Hooks (상태 관리, 생명주기 후킹)
```

## Documents

| 문서 | 설명 |
|------|------|
| [Index.md](Index.md) | 전체 프로젝트 지도 (Map of Content) |
| [Directory_Structure.md](Directory_Structure.md) | 폴더 트리 구조 |
| [Stats_Report.md](Stats_Report.md) | 의존성 통계 및 모듈 분포 |
| [Entrypoints_Overview.md](Entrypoints_Overview.md) | 진입점 분석 |
| [Components_Overview.md](Components_Overview.md) | UI 컴포넌트 분석 |
| [Tools_Overview.md](Tools_Overview.md) | 도구 시스템 분석 |
| [Services_Overview.md](Services_Overview.md) | 서비스 레이어 분석 |
| [Commands_Overview.md](Commands_Overview.md) | 명령어 시스템 분석 |
| [Hooks_Overview.md](Hooks_Overview.md) | 훅 시스템 분석 |
| [Utils_Overview.md](Utils_Overview.md) | 유틸리티 분석 |
| [Ink_Framework.md](Ink_Framework.md) | Ink 프레임워크 분석 |
| [Query_Engine.md](Query_Engine.md) | 쿼리 엔진 분석 |
| [Root_Files_Overview.md](Root_Files_Overview.md) | 루트 파일 분석 |

각 하위 디렉터리(`tools/`, `services/`, `bootstrap/` 등)에도 개별 모듈 분석 문서가 포함되어 있습니다.

## Usage with Obsidian

1. 이 저장소를 클론합니다:
   ```bash
   git clone https://github.com/leaf-kit/claude-analysis.git
   ```
2. Obsidian에서 클론된 폴더를 **Vault**로 엽니다.
3. **Graph View**를 열면 모듈 간 연결 관계를 시각적으로 탐색할 수 있습니다.

모든 문서는 `[[위키링크]]` 형식으로 상호 연결되어 있으며, YAML Frontmatter를 통해 태그 기반 필터링이 가능합니다.

## License

이 분석 문서는 교육 및 참고 목적으로 작성되었습니다.
