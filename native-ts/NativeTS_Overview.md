---
tags:
  - module
  - native-ts
  - performance
  - port
  - flexbox
  - syntax-highlighting
  - fuzzy-search
created: 2026-03-31
type: source-analysis
status: complete
---

# Native TypeScript 포트 (native-ts)

> Rust/C++ 네이티브 모듈의 순수 TypeScript 구현 (3개 서브모듈)

## 개요

| 항목 | 값 |
|------|-----|
| 디렉터리 | `src/native-ts/` |
| 서브모듈 수 | 3 |
| 총 파일 수 | 4 |

네이티브 바이너리 없이도 동작하도록 Rust NAPI 및 C++ 모듈을 순수 TypeScript로 포팅한 모듈입니다.

## 서브모듈

### 1. color-diff/ — 구문 강조 디프 렌더링

| 항목 | 값 |
|------|-----|
| 파일 | `index.ts` (~1,000줄) |
| 원본 | Rust vendor/color-diff-src |
| 용도 | 단어 수준 디프 + 구문 강조 |

**핵심 클래스:**
- `ColorDiff` — 구문 강조된 디프 렌더링
- `ColorFile` — 개별 파일 구문 강조
- `getSyntaxTheme(name)` — 테마 설정

**기능:** 퍼지 단어 디핑, ANSI 컬러 출력, 스코프별 색상 (함수명, 문자열, 키워드 등)

**의존성:** `diff`, `highlight.js`, [[ink/stringWidth]]
**소비자:** [[components/StructuredDiff]], [[components/HighlightedCode]]

---

### 2. file-index/ — 퍼지 파일 검색

| 항목 | 값 |
|------|-----|
| 파일 | `index.ts` (370줄) |
| 원본 | Rust nucleo fuzzy search |
| 용도 | fzf 스타일 파일 검색 |

**핵심 클래스:**
- `FileIndex` — 검색 엔진
  - `loadFromFileList(files)` — 인덱싱
  - `search(query, limit)` — 퍼지 검색

**스코어링 알고리즘:**
- 위치 기반 (낮을수록 좋음)
- 경계 매칭 보너스 (`/`, `_`, `.`, camelCase)
- 갭 패널티
- "test" 포함 경로 1.05× 감점

**의존성:** 없음 (자체 완결)
**소비자:** [[hooks/fileSuggestions]]

---

### 3. yoga-layout/ — Flexbox 레이아웃 엔진

| 항목 | 값 |
|------|-----|
| 파일 | `enums.ts` (135줄), `index.ts` (2,578줄) |
| 원본 | Meta yoga-layout (C++) |
| 용도 | Ink 렌더러용 Flexbox 계산 |

**핵심 클래스:**
- `Node` — Flex 컨테이너/아이템
- `Config` — 레이아웃 설정

**지원 기능:** flex-direction, wrap, grow/shrink/basis, align, justify, margin/padding/border/gap, min/max, position, display, measure functions

**미구현:** aspect-ratio, content-box, RTL

**의존성:** 없음
**소비자:** [[ink/layout/yoga]], [[ink/reconciler]]

## 관계

```
native-ts/color-diff  → components/StructuredDiff.tsx (디프 렌더링)
native-ts/file-index  → hooks/fileSuggestions.ts (파일 자동완성)
native-ts/yoga-layout → ink/layout/yoga.ts (레이아웃 계산)
```
