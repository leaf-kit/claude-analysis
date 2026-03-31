---
tags:
  - ink
  - framework
  - terminal-rendering
  - React
created: 2026-03-31
type: module-analysis
status: complete
---

# Ink Framework

> React 기반 커스텀 터미널 렌더링 프레임워크. Flexbox 레이아웃, 이벤트 시스템, DOM 모델 포함

## 아키텍처 개요

```
React 컴포넌트 트리
  │
  ▼
[[ink/Rendering_Pipeline|Reconciler]] (React 18 Concurrent)
  │
  ▼
[[ink/Layout_System|Yoga Layout]] (Flexbox)
  │
  ▼
Output Generator (셀 기반 렌더링)
  │
  ▼
Screen Buffer (더블 버퍼링)
  │
  ▼
Terminal (ANSI 출력)
```

## 핵심 모듈

### 1. 코어 렌더링 파이프라인

#### `ink.tsx` — 메인 Ink 클래스 (~600줄)

**역할:** React 리컨실레이션, 프레임 버퍼링, 터미널 I/O 관리

**핵심 기능:**
- React 18 Concurrent 렌더링
- 더블 버퍼링 (깜빡임 방지)
- 터미널 raw 모드 관리
- [[ink/Event_System|이벤트 디스패치]] (키보드, 마우스, 포커스)
- 선택 및 복사/붙여넣기
- 성능 메트릭 수집

#### `reconciler.ts` — React Reconciler

**역할:** React 컴포넌트 → 터미널 DOM 변환

```typescript
// DOM 연산
createInstance()    // DOM 엘리먼트 생성
appendChildNode()   // 자식 추가
removeChildNode()   // 자식 제거
// 이벤트 핸들러 등록
// 스타일 디핑
// Yoga 레이아웃 통합
```

#### `renderer.ts` — 프레임 렌더러

```
React 트리 → Yoga 레이아웃 계산 → 화면 버퍼 → 블리팅 (변경분만 갱신)
```

#### `output.ts` — 출력 생성

텍스트 토큰화, ANSI 파싱, 셀 기반 렌더링, 하이퍼링크

### 2. [[ink/Layout_System]] — 레이아웃 시스템

**Yoga (Flexbox) 기반 터미널 레이아웃:**

| 파일 | 용도 |
|------|------|
| `layout/engine.ts` | 레이아웃 엔진 추상화 |
| `layout/yoga.ts` | Yoga 바인딩 |
| `layout/node.ts` | 레이아웃 노드 |
| `layout/geometry.ts` | 기하학 계산 |

**지원 속성:**
- Flex direction (row/column)
- Flex grow/shrink
- Margin, padding, border
- Width/height 제약
- Text wrapping, truncation

### 3. [[ink/Event_System]] — 이벤트 시스템 (10 files)

| 파일 | 용도 |
|------|------|
| `events/dispatcher.ts` | 이벤트 라우팅 및 전파 |
| `events/input-event.ts` | 키보드 입력 파싱 |
| `events/keyboard-event.ts` | 구조화된 키보드 이벤트 |
| `events/click-event.ts` | 마우스 클릭 |
| `events/focus-event.ts` | 포커스/블러 |
| `events/terminal-event.ts` | 터미널 이벤트 (리사이즈, 포커스) |
| `events/emitter.ts` | 이벤트 발행 |
| `events/event-handlers.ts` | 핸들러 등록 |

**이벤트 전파:** Capture → Target → Bubble

**지원:** 마우스 트래킹 (mode 1003), 터미널 포커스 감지 (DECSET 1004)

### 4. DOM 모델 (`dom.ts`)

**터미널 전용 DOM:**

```
ink-root        → 루트 컨테이너
ink-box         → Flex 컨테이너 (= <div>)
ink-text        → 텍스트 콘텐츠
ink-virtual-text → 가상 스크롤 텍스트
ink-link        → 하이퍼링크
ink-progress    → 프로그레스 바
ink-raw-ansi    → Raw ANSI 출력
```

**DOMElement 속성:**
- 스타일 속성
- 이벤트 핸들러
- Yoga 레이아웃 노드
- 스크롤 상태
- Dirty 마킹 (최적화)
- Hidden/visibility

### 5. [[ink/Components]] — Ink 기본 컴포넌트 (15 files)

#### 베이스 컴포넌트

| 컴포넌트 | 용도 | 대응 HTML |
|----------|------|-----------|
| `Box.tsx` | Flexbox 컨테이너 | `<div style="display:flex">` |
| `Text.tsx` | 텍스트 콘텐츠 | `<span>` |
| `ScrollBox.tsx` | 스크롤 컨테이너 | `<div style="overflow:auto">` |
| `Button.tsx` | 클릭 가능 버튼 | `<button>` |
| `Link.tsx` | 하이퍼링크 | `<a href>` |
| `Spacer.tsx` | 유연한 여백 | `<div style="flex:1">` |
| `Newline.tsx` | 줄바꿈 | `<br>` |

**Box Props:**
```typescript
// 레이아웃
flexDirection, flexGrow, flexShrink, margin, padding, border, height, width
// 이벤트
onClick, onFocus, onKeyDown, onMouseEnter, onMouseLeave
// 포커스
tabIndex
```

**Text Props:**
```typescript
// 스타일
color, backgroundColor, bold, italic, underline, strikethrough
// 줄바꿈
wrap: 'wrap' | 'truncate-start' | 'truncate-middle' | 'truncate-end'
```

#### 컨텍스트 컴포넌트

| 컴포넌트 | 제공 값 |
|----------|---------|
| `TerminalSizeContext.tsx` | 터미널 크기 |
| `TerminalFocusContext.tsx` | 터미널 포커스 상태 |
| `StdinContext.ts` | 입력 스트림 |
| `CursorDeclarationContext.ts` | 커서 위치 |
| `ClockContext.tsx` | 애니메이션 클럭 |
| `AppContext.ts` | 앱 종료/제어 |

### 6. [[ink/Hooks]] — 커스텀 훅 (12 files)

#### 입력 & 인터랙션

| 훅 | 용도 |
|----|------|
| `use-input.ts` | 키보드 입력 처리 (raw 키, 수정자, 펑션키) |
| `use-stdin.ts` | stdin 스트림 접근 |
| `use-selection.ts` | 텍스트 선택 (마우스 드래그, Shift+화살표) |

#### 애니메이션 & 타이밍

| 훅 | 용도 |
|----|------|
| `use-animation-frame.ts` | 동기화된 애니메이션 타이밍 |
| `use-interval.ts` | 반복 타이머 |

#### 터미널 상호작용

| 훅 | 용도 |
|----|------|
| `use-terminal-focus.ts` | 터미널 윈도우 포커스 감지 |
| `use-terminal-viewport.ts` | 뷰포트 가시성 추적 |
| `use-declared-cursor.ts` | 커서 위치/모양 선언 |
| `use-terminal-title.ts` | 터미널 윈도우 타이틀 설정 |
| `use-tab-status.ts` | iTerm2 탭 상태 |

### 7. 터미널 I/O (`termio/`)

**ANSI/터미널 프로토콜 지원:**

| 파일 | 용도 |
|------|------|
| `tokenize.ts` | ANSI 시퀀스 토큰화 |
| `parser.ts` | 터미널 제어 코드 파싱 |
| `csi.ts` | Control Sequence Introducer |
| `sgr.ts` | Select Graphic Rendition (색상/스타일) |
| `osc.ts` | Operating System Command |
| `esc.ts` | Escape 시퀀스 |
| `dec.ts` | DEC 제어 코드 |

**지원 기능:** 커서 이동, 256-color/RGB, 텍스트 속성, 마우스 트래킹, 클립보드

### 8. 렌더링 최적화

| 파일 | 용도 |
|------|------|
| `optimizer.ts` | 변경 노드만 렌더링 (디프 최적화) |
| `node-cache.ts` | 노드 캐싱 |
| `line-width-cache.ts` | 문자 폭 캐싱 |
| `log-update.ts` | 효율적 터미널 갱신 |
| `render-to-screen.ts` | Blit 최적화 (변경 셀만 갱신) |

## 통계

| 항목 | 값 |
|------|-----|
| 총 파일 수 | 96 |
| 기본 컴포넌트 | 15 |
| 커스텀 훅 | 12 |
| 이벤트 타입 | 10 |
| termio 파일 | 9 |
| 레이아웃 엔진 | Yoga (Flexbox) |
| 렌더링 방식 | 더블 버퍼링 + Blit 최적화 |

## 관련 문서
- [[Index]] — 전체 MOC
- [[Components_Overview]] — UI 컴포넌트
- [[Hooks_Overview]] — 앱 훅 시스템
