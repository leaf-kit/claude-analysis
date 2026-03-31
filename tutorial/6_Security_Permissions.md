# 🛡️ 보안 아키텍처와 권한 시스템

> 이 장에서는 Claude Code가 **위험한 행동을 방지하는 다층 보안 구조**를 분석합니다.

## 🏰 3겹 성벽 — 다층 방어 구조

Claude Code의 보안은 성의 방어와 같아요. 3겹의 성벽이 있죠!

```mermaid
flowchart TD
    Request["🔧 도구 실행 요청"]

    subgraph L1["🏰 1층: 권한 시스템 (26개 파일)"]
        Rules["규칙 기반 검사"]
        Classifier["AI 분류기"]
        Hooks["훅 시스템"]
    end

    subgraph L2["🧱 2층: 입력 검증"]
        AST["Bash AST 파싱<br/>(tree-sitter)"]
        Path["경로 검증"]
        ReadOnly["읽기전용 검증"]
    end

    subgraph L3["🔒 3층: 샌드박스"]
        Sandbox["프로세스 격리"]
    end

    Request --> L1
    L1 -->|"통과"| L2
    L2 -->|"통과"| L3
    L3 --> Execute["✅ 안전한 실행"]

    L1 -->|"거부"| Blocked["❌ 차단"]
    L2 -->|"위험"| Blocked

    style L1 fill:#fff3e0,stroke:#ef6c00
    style L2 fill:#e8eaf6,stroke:#283593
    style L3 fill:#e8f5e9,stroke:#2e7d32
    style Blocked fill:#ffebee,stroke:#c62828
```

## 🚦 권한 모드 — 6가지 운전 모드

| 모드 | 비유 | 설명 |
|:-----|:-----|:-----|
| `default` | 🚗 일반 운전 | 매 행동마다 물어봄 |
| `acceptEdits` | 🚕 택시 모드 | 파일 수정은 자동 승인 |
| `plan` | 🗺️ 지도 모드 | 계획만, 실행 안함 |
| `bypassPermissions` | 🏎️ 레이싱 모드 | 모든 것 자동 승인 |
| `auto` | 🤖 자동 운전 | AI 분류기가 판단 |
| `dontAsk` | 🚫 정지 모드 | 모든 것 자동 거부 |

```mermaid
flowchart LR
    D["default"] -->|"Shift+Tab"| AE["acceptEdits"]
    AE --> P["plan"]
    P --> BP["bypass<br/>(opt-in)"]
    BP --> D

    style D fill:#e3f2fd,stroke:#1565c0
    style AE fill:#fff3e0,stroke:#ef6c00
    style BP fill:#ffebee,stroke:#c62828
```

> 소스: [`src/utils/permissions/permissions.ts`](../src/utils/permissions/permissions.ts)

## 🔍 권한 결정 12단계

`hasPermissionsToUseToolInner()` 함수는 12단계로 권한을 결정해요:

```mermaid
flowchart TD
    Start["도구 실행 요청"] --> S1{"1. 거부 규칙<br/>매칭?"}
    S1 -->|"Yes"| DENY["❌ DENY"]
    S1 -->|"No"| S2{"2. ASK 규칙<br/>매칭?"}
    S2 -->|"Yes"| ASK["❓ ASK"]
    S2 -->|"No"| S3{"3. 도구별<br/>checkPermissions()"}
    S3 -->|"deny"| DENY
    S3 -->|"ask"| S4{"4. 위험한 파일?<br/>.gitconfig, .bashrc..."}
    S4 -->|"Yes"| ASK
    S4 -->|"No"| S5{"5. bypass<br/>모드?"}
    S5 -->|"Yes"| ALLOW["✅ ALLOW"]
    S5 -->|"No"| S6{"6. 허용 규칙<br/>매칭?"}
    S6 -->|"Yes"| ALLOW
    S6 -->|"No"| ASK

    style ALLOW fill:#e8f5e9,stroke:#2e7d32
    style DENY fill:#ffebee,stroke:#c62828
    style ASK fill:#fff3e0,stroke:#ef6c00
```

**핵심 원칙:** 거부 규칙과 위험 파일 검사는 **bypass 모드에서도 우회할 수 없어요!** (bypass-immune)

## 🌳 Bash 보안 — AST 기반 명령어 검증

Bash 명령어는 특별히 위험해서, **tree-sitter로 AST(구문 트리)를 파싱**해서 검증해요:

```mermaid
flowchart TD
    Cmd["bash 명령어 입력"] --> Parse["1. tree-sitter 파싱<br/>(50ms 타임아웃, 50K 노드 제한)"]
    Parse --> Analyze["2. AST 구조 분석"]

    Analyze --> Q["인용 컨텍스트<br/>(따옴표 처리)"]
    Analyze --> C["복합 구조<br/>(&&, ||, ;, |)"]
    Analyze --> D["위험 패턴<br/>($(), ${}, <<)"]

    Q --> Validate["3. 읽기전용 검증"]
    C --> Validate
    D --> Validate

    Validate --> PathCheck["4. 경로 검증<br/>(디렉터리 탈출 방지)"]
    PathCheck --> SandboxCheck["5. 샌드박스 확인"]
    SandboxCheck --> RuleMatch["6. 규칙 매칭<br/>Bash(git *)"]
    RuleMatch --> ClassifierCheck["7. 분류기 확인<br/>(비동기)"]

    ClassifierCheck --> Decision{"최종 결정"}
    Decision --> Allow["✅ 허용"]
    Decision --> Deny["❌ 거부"]
    Decision --> Ask["❓ 문의"]

    style Parse fill:#e3f2fd,stroke:#1565c0
    style Validate fill:#fff3e0,stroke:#ef6c00
    style Decision fill:#f3e5f5,stroke:#7b1fa2
```

> 소스: [`src/utils/bash/bashParser.ts`](../src/utils/bash/bashParser.ts) · [`src/tools/BashTool/bashSecurity.ts`](../src/tools/BashTool/bashSecurity.ts)

## 🪝 훅 시스템 — 도구 실행 전후 가로채기

**훅(Hook)**은 도구가 실행되기 전이나 후에 자동으로 실행되는 커스텀 코드예요:

```mermaid
flowchart LR
    subgraph Pre["실행 전 훅"]
        PreTool["PreToolUse"]
        PermReq["PermissionRequest"]
    end

    Tool["🔧 도구 실행"]

    subgraph Post["실행 후 훅"]
        PostTool["PostToolUse"]
        PostFail["PostToolUseFailure"]
    end

    Pre --> Tool --> Post

    style Pre fill:#fff3e0,stroke:#ef6c00
    style Tool fill:#e8f5e9,stroke:#2e7d32
    style Post fill:#e3f2fd,stroke:#1565c0
```

**훅 유형 4가지:**

| 유형 | 설명 | 예시 |
|:-----|:-----|:-----|
| `command` | 셸 명령 실행 | `"lint-staged"` |
| `prompt` | LLM 프롬프트 평가 | `"이 변경이 안전한가?"` |
| `http` | HTTP POST 전송 | 외부 서비스 알림 |
| `agent` | 에이전트 검증 | 자동 코드 리뷰 |

> 소스: [`src/schemas/hooks.ts`](../src/schemas/hooks.ts) · [`src/utils/hooks.ts`](../src/utils/hooks.ts)

## 🛡️ 위험한 파일 보호

이 파일/폴더들은 **어떤 모드에서도** 자동 승인되지 않아요:

| 보호 대상 | 이유 |
|:----------|:-----|
| `.gitconfig` | Git 설정 변조 방지 |
| `.bashrc`, `.zshrc` | 셸 시작 스크립트 보호 |
| `.mcp.json` | MCP 서버 설정 보호 |
| `.claude.json` | Claude 설정 보호 |
| `.git/` 디렉터리 | Git 저장소 무결성 |

---

## 💡 엔지니어를 위한 팁

<details>
<summary><b>펼쳐서 기술 심화 내용 보기</b></summary>

### 권한 규칙 문법

```
Tool(content)         — 도구명(패턴)
Bash(git *)           — git + 아무 인자
Read(*.ts)            — TypeScript 파일
Write(src/**)         — src/ 하위 전체
Bash(npm publish:*)   — npm publish 서브커맨드
```

### Auto 모드 위험 패턴 제거

Auto 모드 진입 시 자동으로 제거되는 허용 규칙:
- 셸 인터프리터: `python:*`, `node:*`, `ruby:*`
- 패키지 러너: `npm run:*`, `npx:*`, `bunx:*`
- 셸: `bash:*`, `sh:*`, `zsh:*`
- 위험 도구: `eval:*`, `exec:*`, `sudo:*`

### PermissionResult 타입

```typescript
// 허용
{ behavior: 'allow', updatedInput?, decisionReason? }
// 문의
{ behavior: 'ask', message, suggestions?, pendingClassifierCheck? }
// 거부
{ behavior: 'deny', message, decisionReason }
```

### 핵심 파일

| 파일 | 역할 |
|:-----|:-----|
| [`permissions.ts`](../src/utils/permissions/permissions.ts) | 12단계 결정 로직 |
| [`dangerousPatterns.ts`](../src/utils/permissions/dangerousPatterns.ts) | 위험 패턴 |
| [`filesystem.ts`](../src/utils/permissions/filesystem.ts) | 경로/파일 보호 |
| [`bashParser.ts`](../src/utils/bash/bashParser.ts) | tree-sitter 파서 |
| [`treeSitterAnalysis.ts`](../src/utils/bash/treeSitterAnalysis.ts) | AST 분석 |

</details>

---

👉 다음 장: [**7장: MCP와 확장성 아키텍처**](./7_MCP_Extensibility.md) 🔌
