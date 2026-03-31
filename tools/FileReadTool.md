---
tags:
  - tool
  - file
  - read
created: 2026-03-31
type: tool-analysis
status: complete
---

# FileReadTool

> 파일 읽기 도구. PDF, 이미지, 노트북, 바이너리 파일 지원

## 개요

| 항목 | 값 |
|------|-----|
| 파일 | `src/tools/FileReadTool/FileReadTool.ts` |
| 읽기전용 | ✅ |
| 동시성 안전 | ✅ |
| 파괴적 | ❌ |

## 핵심 기능

1. **텍스트 파일 읽기**: 줄 번호 포함 출력
2. **PDF 읽기**: 페이지 범위 지정 가능
3. **이미지 읽기**: 멀티모달 LLM으로 시각적 표시
4. **노트북 읽기**: .ipynb 파일의 셀 + 출력 표시
5. **바이너리 감지**: 바이너리 파일 안전 처리
6. **토큰 추정**: 파일 크기에 따른 토큰 제한
7. **offset/limit**: 대형 파일 부분 읽기

## 의존 모듈

- [[utils/Permissions_System]] — 읽기 권한 체크
- 토큰 추정 유틸리티

## 관련 문서
- [[Tools_Overview]] — 도구 전체 개요
- [[tools/FileEditTool]] — 파일 편집 도구
- [[tools/FileWriteTool]] — 파일 쓰기 도구
