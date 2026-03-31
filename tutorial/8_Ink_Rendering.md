# 🖥️ 터미널 렌더링 엔진 — React로 터미널 UI 그리기

> Claude Code는 웹이 아닌 **터미널**에서 React를 사용합니다. 이 장에서는 커스텀 Ink 렌더링 파이프라인을 분석합니다.

## 🎨 렌더링 파이프라인 6단계

```mermaid
flowchart LR
    JSX["1️⃣ JSX 컴포넌트<br/>Box, Text, ScrollBox"] --> Reconciler["2️⃣ React Reconciler<br/>커스텀 렌더러"]
    Reconciler --> DOM["3️⃣ DOM 트리<br/>ink-box, ink-text"]
    DOM --> Yoga["4️⃣ Yoga Layout<br/>Flexbox 계산"]
    Yoga --> Buffer["5️⃣ Screen Buffer<br/>프레임 더블버퍼링"]
    Buffer --> ANSI["6️⃣ ANSI 출력<br/>터미널 렌더링"]

    style JSX fill:#e3f2fd,stroke:#1565c0
    style Yoga fill:#fff3e0,stroke:#ef6c00
    style ANSI fill:#e8f5e9,stroke:#2e7d32
```

## 🏗️ Yoga Layout — 터미널 Flexbox

웹에서 `display: flex`를 쓰듯, 터미널에서도 Flexbox로 레이아웃을 잡아요! Meta의 Yoga 엔진을 **순수 TypeScript로 포팅**했어요.

지원하는 속성: `flex-direction`, `justify-content`, `align-items`, `padding`, `margin`, `gap`, `width`, `height`, `flex-grow/shrink`

> 소스: [`src/native-ts/yoga-layout/index.ts`](../src/native-ts/yoga-layout/index.ts) (1,500줄)

## 📺 프레임 더블버퍼링

매 프레임마다 전체 화면을 다시 그리면 깜빡이겠죠? 그래서 **더블 버퍼링**을 써요:

```mermaid
flowchart TD
    Render["렌더링"] --> Back["Back Buffer<br/>(새 프레임)"]
    Front["Front Buffer<br/>(현재 화면)"]
    Back --> Diff["차이점만<br/>계산 (LogUpdate)"]
    Front --> Diff
    Diff --> Patch["변경된 부분만<br/>터미널에 쓰기"]
    Patch --> Swap["버퍼 교체"]

    style Back fill:#e3f2fd,stroke:#1565c0
    style Front fill:#fff3e0,stroke:#ef6c00
    style Diff fill:#f3e5f5,stroke:#7b1fa2
```

프레임 간격: ~16ms (60fps), `FRAME_INTERVAL_MS`

> 소스: [`src/ink/ink.tsx`](../src/ink/ink.tsx) (1,722줄)

## 🧩 핵심 컴포넌트

| 컴포넌트 | 용도 | 소스 |
|:---------|:-----|:-----|
| `Box` | Flex 컨테이너 | [`src/ink/components/Box.tsx`](../src/ink/components/Box.tsx) |
| `Text` | 텍스트 노드 | [`src/ink/components/Text.tsx`](../src/ink/components/Text.tsx) |
| `ScrollBox` | 스크롤 뷰포트 | [`src/ink/components/ScrollBox.tsx`](../src/ink/components/ScrollBox.tsx) |
| `Link` | 터미널 하이퍼링크 (OSC 8) | [`src/ink/components/Link.tsx`](../src/ink/components/Link.tsx) |

---

## 💡 엔지니어를 위한 팁

<details>
<summary><b>기술 심화</b></summary>

### 성능 최적화
- **단일 슬롯 Yoga 캐시**: 반복 `calculateLayout()` 방지
- **Blit 패스트 패스**: 레이아웃 변경 없으면 좁은 영역만 업데이트
- **스타일 풀**: ANSI 스타일 전환 중복 제거
- **스크롤 최적화**: DECSTBM blit+shift (하드웨어 스크롤)

### DOM → Output 파이프라인
```
DOMElement → renderNodeToOutput → Screen Buffer (Cell[][]) → LogUpdate diff → ANSI patches → stdout.write()
```

</details>

---

👉 다음 장: [**9장: 상태 관리와 글로벌 스토어**](./9_State_Management.md) ⚙️
