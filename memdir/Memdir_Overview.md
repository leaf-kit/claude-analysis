---
tags:
  - module
  - memdir
  - memory
  - persistence
  - AI-selection
created: 2026-03-31
type: source-analysis
status: complete
---

# Memory 시스템 (Memdir)

> 파일 기반 영속 메모리 관리 — 4가지 타입, AI 기반 선택, 팀 메모리 지원 (8파일)

## 개요

| 항목 | 값 |
|------|-----|
| 디렉터리 | `src/memdir/` |
| 파일 수 | 8 |
| 메모리 타입 | 4 (user, feedback, project, reference) |
| 최대 메모리 파일 | 200개/디렉터리 |
| AI 선택 모델 | Sonnet (최대 5개 선택) |

## 파일 구조

```
memdir/
├── memdir.ts              — 메인 프롬프트 빌더 & 라이프사이클
├── paths.ts               — 경로 해결 & 보안 검증
├── teamMemPaths.ts        — 팀 메모리 격리 & 경로 순회 방어
├── memoryScan.ts          — 디렉터리 스캔 & 인덱싱
├── findRelevantMemories.ts — AI 기반 메모리 선택
├── memoryAge.ts           — 신선도 & 부패 경고
├── memoryTypes.ts         — 4타입 분류 체계 & 프롬프트
└── teamMemPrompts.ts      — 듀얼 디렉터리 통합 프롬프트
```

## 메모리 타입 체계

| 타입 | 범위 | 용도 |
|------|------|------|
| **user** | 항상 private | 사용자 역할, 목표, 선호 |
| **feedback** | private (기본) 또는 team | 작업 접근 방식 가이드 (교정 & 확인) |
| **project** | private 또는 team | 진행 중인 작업, 목표, 이니셔티브 |
| **reference** | 보통 team | 외부 시스템 포인터 (Linear, Grafana 등) |

## 메모리 모드

### KAIROS 모드 (어시스턴트)
```
일일 로그 추가 (append-only)
  → /dream 스킬이 야간에 요약
    → MEMORY.md 인덱스 + 주제 파일로 증류
```

### TEAMMEM 모드 (듀얼 디렉터리)
```
Private: {autoMemPath}/memory/   — 개인 메모리
Team:    {autoMemPath}/team/     — 팀 공유 메모리
```

### Auto 모드 (기본)
```
단일 디렉터리: {autoMemPath}/memory/
  → MEMORY.md 인덱스 + 주제 파일
```

## AI 기반 메모리 선택

```typescript
// findRelevantMemories.ts
findRelevantMemories(query, memoryDir, signal, recentTools?, alreadySurfaced?)
  → scanMemoryFiles()     // .md 파일 스캔 (최대 200개)
  → formatMemoryManifest() // 매니페스트 텍스트 생성
  → selectRelevantMemories() // Sonnet으로 최대 5개 선택
    → sideQuery()          // Claude API 호출
    → JSON 파싱 + 유효성 검증
```

## 보안 (PSR M22186)

`teamMemPaths.ts`의 4중 방어:
1. **Layer 1**: `path.resolve()`로 `..` 세그먼트 거부
2. **Layer 2**: `realpath()`로 심볼릭 링크 해결
3. **Layer 3**: `lstat()`로 댕글링 심볼릭 링크 감지
4. **Layer 4**: 심볼릭 링크 루프 감지

## 경로 해결 체인

```
환경변수 오버라이드
  → Settings.json 오버라이드 (프로젝트 설정 제외 - 악의적 repo 방지)
    → Git root 기반 기본 경로
      → ~/.claude/projects/{sanitized-path}/memory/
```

## 의존성

### 주요 내부 모듈
| 모듈 | 용도 |
|------|------|
| [[bootstrap/state]] | 세션 상태 (KAIROS, CWD) |
| [[services/analytics]] | 텔레메트리 |
| [[utils/git]] | Git root 탐색 |
| [[utils/settings]] | 설정 읽기 |
| [[utils/model/model]] | Sonnet 모델 선택 |
| [[utils/sideQuery]] | AI 쿼리 실행 |

## 관계

- **상위**: [[Query_Engine]] → `loadMemoryPrompt()` 호출하여 메모리 프롬프트 로드
- **도구**: [[tools/FileReadTool]] → `memoryFreshnessNote()` 표시
- **서비스**: [[services/ExtractMemories_Service]] → 자동 메모리 추출
- **MEMORY.md**: 최대 200줄, 25,000바이트 제한
