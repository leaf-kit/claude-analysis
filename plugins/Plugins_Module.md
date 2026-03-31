---
tags:
  - module
  - plugins
  - built-in
  - extensibility
created: 2026-03-31
type: source-analysis
status: complete
---

# 플러그인 모듈 (Plugins)

> 내장 플러그인 레지스트리 및 초기화 (2파일)

## 개요

| 항목 | 값 |
|------|-----|
| 디렉터리 | `src/plugins/` |
| 파일 수 | 2 |

## 파일 구조

### builtinPlugins.ts — 내장 플러그인 레지스트리

```typescript
BUILTIN_MARKETPLACE_NAME = 'builtin'

registerBuiltinPlugin(definition)     // 내장 플러그인 등록
isBuiltinPluginId(pluginId)           // '@builtin' 접미사 확인
getBuiltinPluginDefinition(name)      // 이름으로 조회
getBuiltinPlugins()                   // 활성/비활성 분리 반환
getBuiltinPluginSkillCommands()       // 스킬 → Command 변환
clearBuiltinPlugins()                 // 테스트용 초기화
```

- `Map<string, BuiltinPluginDefinition>` 저장소 사용
- `BundledSkillDefinition` → `Command` 변환 (source: `'bundled'`)
- 사용자 설정으로 활성/비활성 토글 가능

**의존성:** [[commands]] (Command 타입), [[skills/bundledSkills]], [[types/plugin]], [[utils/settings]]

### bundled/index.ts — 초기화 진입점

```typescript
initBuiltinPlugins()  // 현재 빈 스캐폴딩
```

**의존성:** 없음 (향후 플러그인 등록 시 확장)

## 관계

- **호출자**: [[commands.ts]] (`getBuiltinPluginSkillCommands()` 호출)
- **호출자**: [[main.tsx]] (`initBuiltinPlugins()` 호출)
- **연동**: [[Services_Overview|Plugins_Service]] — 플러그인 라이프사이클 관리
