---
tags:
  - core
  - query
  - execution-loop
created: 2026-03-31
type: module-analysis
status: complete
---

# Query Engine

> Claude Code의 핵심 실행 엔진. 사용자 입력을 받아 모델 호출과 도구 실행을 오케스트레이션

## 아키텍처

```
사용자 입력
  │
  ▼
QueryEngine.ts (라이프사이클 관리)
  ├── 메시지 준비 및 정규화
  ├── 메모리/첨부파일 프리페칭
  ├── Thinking 설정
  ├── 모델 선택 및 파싱
  │
  ▼
query.ts (실제 실행)
  ├── 메시지 정규화
  ├── 도구 가용성 확인
  ├── [[services/Compact_Service|자동 압축]] 판단
  ├── 모델 응답 스트리밍
  ├── [[Tools_Overview|도구 실행 오케스트레이션]]
  ├── [[utils/Permissions_System|권한 체크]]
  ├── 토큰 예산 추적
  └── [[services/SessionMemory_Service|세션 메모리]] 통합
```

## QueryEngine.ts — 상위 오케스트레이터

**역할:** 쿼리 라이프사이클 전체를 관리하는 상위 계층

**핵심 기능:**
- 에러 처리 및 분류 (재시도 가능 에러 감지)
- 사용량 추적 및 비용 계산
- 메시지 상태 관리
- 메모리/첨부파일 프리페칭
- Thinking 설정 구성
- 플러그인 캐시 관리
- 모델 선택 및 파싱

**의존 모듈:**
- [[Query_Module|query.ts]] — 실제 실행
- `cost-tracker.ts` — 비용 추적
- [[utils/Model_Management]] — 모델 선택
- [[services/Analytics_Service]] — 분석

## query.ts — 핵심 실행 모듈

**역할:** 실제 모델 호출과 도구 실행을 처리하는 핵심 모듈

**실행 흐름:**

```typescript
async function query(params) {
  // 1. 메시지 정규화 및 준비
  // 2. 도구 가용성 확인 (deferred tool search 포함)
  // 3. 자동 압축 여부 판단
  // 4. API 호출 → 모델 응답 스트리밍
  // 5. 응답에서 도구 호출 추출
  // 6. StreamingToolExecutor로 도구 병렬 실행
  // 7. 권한 체크 및 실행
  // 8. 결과를 대화에 추가
  // 9. 토큰 예산 갱신
  // 10. 세션 메모리 통합
}
```

**핵심 의존성:**

| 의존 모듈 | 용도 |
|-----------|------|
| [[Tools_Overview\|Tool.ts, tools.ts]] | 도구 인터페이스 및 레지스트리 |
| [[services/Tools_Service\|StreamingToolExecutor]] | 도구 병렬 실행 |
| [[services/Compact_Service\|compact]] | 컨텍스트 압축 |
| [[services/API_Service\|API Client]] | Claude API 호출 |
| [[utils/Permissions_System]] | 권한 확인 |
| [[utils/Model_Management]] | 모델 유틸리티 |
| [[Context_Module\|context.ts]] | 시스템/사용자 컨텍스트 |

## query/ 하위 모듈

| 파일 | 용도 |
|------|------|
| `tokenBudget.ts` | 토큰 예산 관리 및 임계값 계산 |
| `stopHooks.ts` | 쿼리 중단 훅 |
| `config.ts` | 쿼리 설정 |
| `deps.ts` | 쿼리 의존성 주입 |

## context.ts — 컨텍스트 모듈

**역할:** 각 대화에 추가되는 시스템/사용자 컨텍스트 생성

**주요 export:**
- `getGitStatus()` — Git 저장소 정보 (브랜치, 상태, 최근 커밋)
- `getSystemContext()` — 시스템 수준 컨텍스트
- `getUserContext()` — 사용자 수준 컨텍스트 (CLAUDE.md 포함)

**캐싱 전략:**
- 대화 단위로 메모이즈
- 캐시 주입 변경 시 무효화
- Git 명령/파일 읽기의 지연 로딩

## 통계

| 항목 | 값 |
|------|-----|
| 관련 파일 수 | 6 |
| 핵심 함수 | `query()`, `QueryEngine` |
| 도구 실행 방식 | 스트리밍 + 병렬 |
| 컨텍스트 소스 | Git, CLAUDE.md, 시스템 정보 |

## 관련 문서
- [[Index]] — 전체 MOC
- [[Tools_Overview]] — 도구 시스템
- [[services/Compact_Service]] — 컨텍스트 압축
- [[services/API_Service]] — API 클라이언트
