---
tags:
  - service
  - API
  - multi-provider
  - authentication
created: 2026-03-31
type: service-analysis
status: complete
---

# API Service

> Claude API 클라이언트. 멀티 프로바이더 지원 (Anthropic, Bedrock, Vertex AI, Azure Foundry)

## 아키텍처

```
getAnthropicClient()
  │
  ├── 프로바이더 자동 감지
  │   ├── Direct API (api.anthropic.com)
  │   ├── AWS Bedrock (@anthropic-ai/bedrock-sdk)
  │   ├── Google Vertex AI (@anthropic-ai/vertex-sdk)
  │   └── Azure Foundry (@anthropic-ai/foundry-sdk)
  │
  ├── 인증 (API Key / OAuth / IAM / AD)
  │
  └── 설정 적용 (프록시, mTLS, CA 인증서)
```

## 파일 구조

| 파일 | 용도 | 연결 |
|------|------|------|
| `client.ts` | 클라이언트 생성 + 프로바이더 감지 | [[services/OAuth_Service]] |
| `claude.ts` | 메인 API 인터랙션 | [[Query_Engine]] |
| `withRetry.ts` | 재시도 로직 | [[services/API_Service\|errors.ts]] |
| `errors.ts` | 에러 정의/핸들링 | — |
| `bootstrap.ts` | API 부트스트랩 | [[Entrypoints_Overview]] |
| `usage.ts` | 사용량 추적 | 비용 추적 |
| `promptCacheBreakDetection.ts` | 프롬프트 캐시 최적화 | — |
| `filesApi.ts` | 파일 API | — |
| `logging.ts` | API 로깅 | [[services/Analytics_Service]] |
| `dumpPrompts.ts` | 프롬프트 덤프 | 디버깅 |
| `sessionIngress.ts` | 세션 인그레스 | 원격 |
| `adminRequests.ts` | 관리자 요청 | — |
| `metricsOptOut.ts` | 메트릭 옵트아웃 | — |
| `grove.ts` | Grove 통합 | — |
| `referral.ts` | 리퍼럴 | — |
| `emptyUsage.ts` | 빈 사용량 | — |

## 의존 모듈

- `@anthropic-ai/sdk` — 코어 API
- `@anthropic-ai/bedrock-sdk` — AWS
- `@anthropic-ai/vertex-sdk` — GCP
- `@anthropic-ai/foundry-sdk` — Azure
- `google-auth-library` — GCP 인증
- `@azure/identity` — Azure AD 인증

## 관련 문서
- [[Services_Overview]] — 서비스 전체
- [[Query_Engine]] — 쿼리 엔진
- [[services/OAuth_Service]] — OAuth
