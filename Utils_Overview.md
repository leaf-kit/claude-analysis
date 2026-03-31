---
tags:
  - utils
  - infrastructure
  - security
created: 2026-03-31
type: module-analysis
status: complete
---

# Utils Overview

> Claude Code의 핵심 유틸리티 라이브러리 (564 files). 권한, 설정, 모델, 보안, Git, 셸 등

## 유틸리티 모듈 맵

```
utils/
├── [[utils/Permissions_System|permissions/]]  # 26 files — 권한 시스템
├── [[utils/Settings_Management|settings/]]    # 19 files — 설정 관리
├── [[utils/Model_Management|model/]]          # 18 files — 모델 관리
├── [[utils/Bash_Utils|bash/]]                 # 15+ files — Bash 파싱/보안
├── [[utils/Telemetry|telemetry/]]             # 14 files — 텔레메트리
├── [[utils/Shell_Utils|shell/]]               # 12 files — 셸 추상화
├── [[utils/Git_Utils|git/]]                   # 5 files — Git 상태 읽기
├── task/                                      # 5 files — 태스크 관리
├── messages/                                  # 4 files — 메시지 매핑
├── sandbox/                                   # 2 files — 샌드박싱
├── memory/                                    # 2 files — 메모리 타입
├── plugins/                                   # 46+ files — 플러그인 유틸
├── swarm/                                     # 16+ files — 분산 에이전트
├── hooks/                                     # 훅 유틸리티
├── skills/                                    # 스킬 유틸리티
├── github/                                    # GitHub 통합
├── computerUse/                               # 컴퓨터 사용
├── processUserInput/                          # 사용자 입력 처리
├── secureStorage/                             # 보안 저장소
└── background/                                # 백그라운드 처리
```

## 핵심 유틸리티 상세

### 1. [[utils/Permissions_System]] — 권한 시스템 (26 files)

**핵심 아키텍처:**

```
도구 실행 요청
  │
  ▼
checkPermissions() ← permissions.ts (메인 결정 엔진)
  │
  ├── PermissionMode 확인 (default/plan/yolo/...)
  ├── PermissionRule 평가 (allow/deny/ask)
  ├── classifierDecision (AI 기반 안전 분류)
  ├── bashClassifier (위험 패턴 감지)
  ├── yoloClassifier (빠른 허용/거부)
  └── PermissionResult 반환
```

**핵심 파일:**

| 파일 | 용도 |
|------|------|
| `permissions.ts` | 메인 권한 결정 엔진 |
| `PermissionMode.ts` | 모드 설정 (default, plan, yolo 등) |
| `PermissionRule.ts` | 규칙 타입 (allow/deny/ask) |
| `PermissionResult.ts` | 결정 결과 |
| `classifierDecision.ts` | AI 기반 안전 분류 |
| `bashClassifier.ts` | Bash 위험 패턴 감지 |
| `dangerousPatterns.ts` | 위험 셸 패턴 |
| `pathValidation.ts` | 파일 경로 안전 검사 |
| `denialTracking.ts` | 거부 추적 (UX) |
| `permissionExplainer.ts` | 사람 읽기 가능 설명 |
| `PermissionUpdate.ts` | 설정 영속화 |

**권한 모드:**
- `default` — 기본 (도구별 확인)
- `plan` — 계획 모드 (실행 전 승인)
- `acceptEdits` — 편집 자동 수락
- `autoApprove` — 자동 승인
- `yolo` — 모두 허용
- `autoReject` — 모두 거부

---

### 2. [[utils/Settings_Management]] — 설정 관리 (19 files)

**계층적 설정 로딩:**

```
우선순위: Managed(최고) > Remote-Managed > Env > User(최저)
```

**핵심 파일:**

| 파일 | 용도 |
|------|------|
| `settings.ts` | 메인 설정 로더 (멀티소스) |
| `types.ts` | SettingsJson 스키마 |
| `validation.ts` | Zod 기반 스키마 검증 |
| `changeDetector.ts` | 설정 변경 감지 |
| `managedPath.ts` | 관리 설정 경로 해석 |
| `mdm/settings.ts` | Windows MDM 통합 |
| `mdm/rawRead.ts` | 레지스트리 읽기 |

---

### 3. [[utils/Model_Management]] — 모델 관리 (18 files)

**모델 선택 우선순위:**
```
세션 오버라이드 > 시작 플래그 > 환경변수 > 설정 > 기본값
```

**핵심 파일:**

| 파일 | 용도 |
|------|------|
| `model.ts` | 모델 선택/해석 |
| `modelStrings.ts` | 모델 버전 매핑/코드네임 |
| `modelCapabilities.ts` | 모델별 기능 지원 |
| `modelAllowlist.ts` | 모델 가용성 제한 |
| `aliases.ts` | 사용자 모델 별칭 |
| `providers.ts` | API 프로바이더 설정 |
| `modelCost.ts` | 토큰 가격/비용 계산 |
| `contextWindowUpgradeCheck.ts` | 1M 컨텍스트 윈도우 체크 |

**구독 인지 모델 가용성:** Claude AI, Pro, Team Premium

---

### 4. [[utils/Bash_Utils]] — Bash 파싱/보안 (15+ files)

**핵심 파일:**

| 파일 | 용도 |
|------|------|
| `commands.ts` | 명령 파싱 + 보안 (인젝션 방지) |
| `parser.ts` | Tree-sitter AST 파싱 |
| `registry.ts` | @withfig/autocomplete 명령 스펙 |
| `ast.ts` | AST 유틸리티 |
| `heredoc.ts` | Here-doc 처리 |
| `shellQuoting.ts` | 셸 인용 처리 |
| `treeSitterAnalysis.ts` | Tree-sitter 분석 |

**보안 메커니즘:**
- 랜덤 솔트 플레이스홀더로 인젝션 방지
- 정적 리다이렉트 타겟 검증
- Tree-sitter AST 파싱 (레거시 fallback 포함)
- 리다이렉션 추출 (권한 체크용)

---

### 5. [[utils/Git_Utils]] — Git 상태 읽기 (5 files)

| 파일 | 용도 |
|------|------|
| `gitFilesystem.ts` | 파일시스템 기반 Git 상태 읽기 (subprocess 미사용) |
| `gitConfigParser.ts` | .git/config 경량 파서 |
| `gitignore.ts` | gitignore 처리 |

**특징:** Git 서브프로세스 대신 파일시스템 직접 읽기로 성능 최적화

---

### 6. [[utils/Shell_Utils]] — 셸 추상화 (12 files)

| 파일 | 용도 |
|------|------|
| `shellProvider.ts` | 셸 프로바이더 인터페이스 |
| `bashProvider.ts` | Bash 구현 |
| `powershellProvider.ts` | PowerShell 구현 |
| `readOnlyCommandValidation.ts` | 읽기전용 명령 검증 |
| `resolveDefaultShell.ts` | 기본 셸 탐색 |

**읽기전용 명령 레지스트리:**
- `GIT_READ_ONLY_COMMANDS` — 안전한 git 명령
- `GH_READ_ONLY_COMMANDS` — 안전한 gh 명령
- `EXTERNAL_READONLY_COMMANDS` — 크로스셸 안전 명령

---

### 7. [[utils/Telemetry]] — 텔레메트리 (14 files)

| 파일 | 용도 |
|------|------|
| `events.ts` | OTel 이벤트 로깅 |
| `instrumentation.ts` | OpenTelemetry 설정 |
| `logger.ts` | 구조화 로깅 |
| `sessionTracing.ts` | 세션 트레이싱 |
| `perfettoTracing.ts` | Chrome Perfetto 포맷 |
| `bigqueryExporter.ts` | BigQuery 내보내기 |

**특징:** 사용자 프롬프트는 `OTEL_LOG_USER_PROMPTS` 활성화 시에만 포함

---

### 8. 기타 유틸리티

| 모듈 | 파일 수 | 용도 |
|------|---------|------|
| `swarm/` | 16+ | 분산 에이전트 코디네이션 |
| `plugins/` | 46+ | 플러그인 시스템 아키텍처 |
| `sandbox/` | 2 | @anthropic-ai/sandbox-runtime 통합 |
| `task/` | 5 | 태스크 상태 관리/디스크 영속화 |
| `messages/` | 4 | SDK 메시지 포맷 변환 |
| `memory/` | 2 | 메모리 타입 (User, Project, Local, Managed 등) |
| `skills/` | 1 | 스킬 디렉터리 변경 감지 |
| `github/` | - | GitHub 통합 |
| `computerUse/` | - | 컴퓨터 사용 기능 |
| `processUserInput/` | - | 사용자 입력 처리 |
| `secureStorage/` | - | 보안 저장소 |
| `background/` | - | 백그라운드 처리 |

## 통계

| 항목 | 값 |
|------|-----|
| 총 파일 수 | 564 |
| 권한 시스템 | 26 files |
| 설정 관리 | 19 files |
| 모델 관리 | 18 files |
| Bash 유틸 | 15+ files |
| 텔레메트리 | 14 files |
| 셸 추상화 | 12 files |

## 관련 문서
- [[Index]] — 전체 MOC
- [[Tools_Overview]] — 도구 시스템
- [[Hooks_Overview]] — 훅 시스템
