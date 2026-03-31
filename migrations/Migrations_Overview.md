---
tags:
  - module
  - migrations
  - configuration
  - model-upgrade
created: 2026-03-31
type: source-analysis
status: complete
---

# 마이그레이션 시스템 (Migrations)

> 설정 마이그레이션 및 모델 버전 업그레이드 (11파일)

## 개요

| 항목 | 값 |
|------|-----|
| 디렉터리 | `src/migrations/` |
| 파일 수 | 11 |
| 설정 마이그레이션 | 4개 |
| 모델 업그레이드 | 5개 |
| 권한/기능 업데이트 | 2개 |
| 실행 시점 | `main.tsx` 부트스트랩 |

## 실행 순서

```
main.tsx → runMigrations()
  1. migrateAutoUpdatesToSettings()
  2. migrateBypassPermissionsAcceptedToSettings()
  3. migrateEnableAllProjectMcpServersToSettings()
  4. resetProToOpusDefault()
  5. migrateSonnet1mToSonnet45()
  6. migrateLegacyOpusToCurrent()
  7. migrateSonnet45ToSonnet46()
  8. migrateOpusToOpus1m()
  9. migrateReplBridgeEnabledToRemoteControlAtStartup()
  10. migrateFennecToOpus()          (ant 사용자만)
  11. resetAutoModeOptInForDefaultOffer() (피처 플래그 조건)
```

## 카테고리별 분류

### 설정 이전 (Configuration Migration)
| 파일 | 설명 |
|------|------|
| `migrateAutoUpdatesToSettings.ts` | globalConfig → settings.json (`DISABLE_AUTOUPDATER`) |
| `migrateBypassPermissionsAcceptedToSettings.ts` | globalConfig → settings.json (`skipDangerousModePermissionPrompt`) |
| `migrateEnableAllProjectMcpServersToSettings.ts` | projectConfig → localSettings (MCP 서버 승인) |
| `migrateReplBridgeEnabledToRemoteControlAtStartup.ts` | `replBridgeEnabled` → `remoteControlAtStartup` 키 이름 변경 |

### 모델 버전 업그레이드 (Model Version)
| 파일 | 변환 |
|------|------|
| `migrateLegacyOpusToCurrent.ts` | `claude-opus-4-0/4-1` → `opus` 별칭 |
| `migrateSonnet1mToSonnet45.ts` | `sonnet[1m]` → `sonnet-4-5-20250929[1m]` 고정 |
| `migrateSonnet45ToSonnet46.ts` | `sonnet-4-5-*` → `sonnet` 별칭 (4.6으로) |
| `migrateOpusToOpus1m.ts` | `opus` → `opus[1m]` (Max/Team Premium) |
| `migrateFennecToOpus.ts` | `fennec-*` → `opus` (ant 내부용) |

### 권한/기능 업데이트 (Permission/Feature)
| 파일 | 설명 |
|------|------|
| `resetProToOpusDefault.ts` | Pro 구독자 Opus 4.5 기본값 설정 |
| `resetAutoModeOptInForDefaultOffer.ts` | auto-mode 다이얼로그 재표시 |

## 설계 원칙

1. **멱등성(Idempotency)**: 완료 플래그 또는 정확한 값 비교로 1회만 실행
2. **선택적 범위**: 특정 settings source만 터치 (자동 승격 방지)
3. **사용자별 조건**: 구독 등급, API 프로바이더, 기능 플래그 확인
4. **에러 처리**: try-catch + 분석 이벤트 로깅
5. **알림 연계**: 일부 마이그레이션은 타임스탬프 설정 → UI 1회 알림

## 의존성

### 공통 유틸리티
- [[utils/config]] — GlobalConfig/ProjectConfig 읽기/쓰기
- [[utils/settings]] — userSettings/localSettings/projectSettings 관리
- [[utils/model]] — 모델 파싱, 기능 플래그
- [[utils/auth]] — 구독 등급 감지 (Pro/Max/Team Premium)
- [[services/analytics]] — 이벤트 로깅

## 관계

- **호출자**: [[main.tsx]] `runMigrations()` (line 950)
- **설정 소스**: `~/.claude.json` (GlobalConfig), `settings.json` (Settings)
- **모델 시스템**: [[utils/model/providers]] — API 프로바이더 감지
