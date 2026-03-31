---
tags:
  - tool
  - bash
  - shell
  - security
created: 2026-03-31
type: tool-analysis
status: complete
---

# BashTool

> 셸 명령 실행 도구. 보안 검증, 샌드박싱, 명령 의미 분류 포함

## 개요

| 항목 | 값 |
|------|-----|
| 파일 | `src/tools/BashTool/BashTool.tsx` |
| 읽기전용 | ❌ (명령에 따라 가변) |
| 동시성 안전 | ❌ |
| 파괴적 | ⚠️ (명령에 따라 가변) |

## 핵심 기능

1. **Bash 명령 실행**: 사용자/모델이 요청한 셸 명령 실행
2. **보안 검증**: 위험 명령 패턴 감지 및 차단
3. **샌드박싱**: 격리된 환경에서 실행 가능
4. **명령 의미 분류**: search, read, write로 분류
5. **백그라운드 태스크 지원**: 장시간 명령 백그라운드 실행

## 보안 아키텍처

```
명령 입력
  │
  ▼
[[utils/Bash_Utils|Tree-sitter AST 파싱]]
  │
  ▼
[[utils/Permissions_System|권한 체크]]
  ├── bashClassifier (위험 패턴 감지)
  ├── dangerousPatterns (위험 셸 패턴)
  └── pathValidation (경로 안전 검사)
  │
  ▼
샌드박싱 적용 (선택적)
  │
  ▼
명령 실행
```

## 명령 분류

| 분류 | 설명 | 예시 |
|------|------|------|
| `search` | 검색/탐색 | `find`, `grep`, `ls` |
| `read` | 읽기 | `cat`, `head`, `git log` |
| `write` | 쓰기/실행 | `rm`, `mv`, `npm install` |

## 의존 모듈

- [[utils/Bash_Utils]] — 명령 파싱 및 AST 분석
- [[utils/Permissions_System]] — 권한 체크
- [[utils/Shell_Utils]] — 셸 프로바이더
- [[utils/Settings_Management]] — 샌드박스 설정

## 관련 문서
- [[Tools_Overview]] — 도구 전체 개요
- [[utils/Permissions_System]] — 권한 시스템
- [[utils/Bash_Utils]] — Bash 유틸리티
