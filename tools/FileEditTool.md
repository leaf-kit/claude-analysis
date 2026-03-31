---
tags:
  - tool
  - file
  - edit
created: 2026-03-31
type: tool-analysis
status: complete
---

# FileEditTool

> 파일 편집 도구. 문자열 찾기/바꾸기 방식으로 정확한 편집 수행

## 개요

| 항목 | 값 |
|------|-----|
| 파일 | `src/tools/FileEditTool/FileEditTool.ts` |
| 읽기전용 | ❌ |
| 동시성 안전 | ❌ |
| 파괴적 | ❌ |

## 핵심 기능

1. **문자열 찾기/바꾸기**: `old_string` → `new_string` 치환
2. **전체 치환**: `replace_all` 옵션으로 모든 인스턴스 치환
3. **Git 디프 추적**: 변경사항 자동 추적
4. **파일 히스토리**: 편집 히스토리 지원
5. **고유성 검증**: `old_string`이 파일 내 유일한지 확인

## 입출력

```typescript
interface FileEditInput {
  file_path: string;    // 절대 경로
  old_string: string;   // 찾을 문자열
  new_string: string;   // 바꿀 문자열
  replace_all?: boolean; // 전체 치환
}

interface FileEditOutput {
  filePath: string;
  patch: string;        // 디프 패치
}
```

## 의존 모듈

- [[utils/Permissions_System]] — 쓰기 권한 체크
- [[utils/Git_Utils]] — Git 디프 추적

## 관련 문서
- [[Tools_Overview]] — 도구 전체 개요
- [[tools/FileReadTool]] — 파일 읽기 도구
- [[tools/FileWriteTool]] — 파일 쓰기 도구
