---
tags:
  - module
  - outputStyles
  - configuration
  - markdown
created: 2026-03-31
type: source-analysis
status: complete
---

# loadOutputStylesDir.ts

> 마크다운 기반 출력 스타일 설정 로더 (99줄)

## 개요

| 항목 | 값 |
|------|-----|
| 파일 경로 | `src/outputStyles/loadOutputStylesDir.ts` |
| 주요 역할 | `.claude/output-styles/` 디렉터리에서 출력 스타일 로드 |
| 라인 수 | 99 |

## 핵심 Export

```typescript
getOutputStyleDirStyles(cwd: string): Promise<OutputStyleConfig[]> // 메모이즈됨
clearOutputStyleCaches(): void
```

### OutputStyleConfig 구조
```typescript
{
  name: string                     // 스타일 식별자
  description: string              // 사용자 표시 설명
  prompt: string                   // 스타일 시스템 프롬프트
  source: SettingSource            // 'project' | 'user' | 'built-in'
  keepCodingInstructions?: boolean // 기본 코딩 지시 유지 여부
}
```

## 로딩 알고리즘

1. `.claude/output-styles/`에서 `.md` 파일 로드 (프로젝트 + 홈)
2. Frontmatter 메타데이터 추출 (`name`, `description`, `keep-coding-instructions`)
3. 파일명 (`.md` 제외)을 기본 스타일 이름으로 사용
4. Frontmatter 없으면 마크다운 본문에서 설명 파싱
5. 결과 메모이즈 + 캐시 무효화 지원

## 의존성

| 모듈 | 용도 |
|------|------|
| [[constants/outputStyles]] | `OutputStyleConfig` 타입 |
| [[utils/frontmatterParser]] | Frontmatter 파싱 |
| [[utils/markdownConfigLoader]] | 마크다운 파일 로드 |
| [[utils/plugins/loadPluginOutputStyles]] | 플러그인 캐시 클리어 |

## 관계
- **소비자**: [[constants/outputStyles]] → 커스텀 스타일 초기화 시 호출
