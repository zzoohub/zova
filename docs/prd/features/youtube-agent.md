# YouTube Agent

## Overview

원본 아티클에서 유튜브 콘텐츠 풀세트를 제작하는 에이전트. 텍스트(스크립트, 설명문), 오디오(보이스오버 TTS), 이미지(썸네일), 영상(씬 구성)까지 멀티모달 AI를 활용한다. MVP에서 가장 복잡한 채널 에이전트이자, 이 에이전트가 동작하면 나머지 텍스트 전용 채널은 부분집합.

---

## Requirements

- REQ-001: agent-spec 형식의 YAML로 정의되며, agent-runtime 위에서 실행
- REQ-002: 입력은 article-schema를 따르는 원본 아티클
- REQ-003: 다음 스텝으로 구성 (각 스텝의 mode는 사용자가 설정):
  - `script` — 원본 아티클 기반 유튜브 스크립트 생성. 자연스러운 구어체, 지정 분량
  - `scenes` — 스크립트를 씬 단위로 분할. 각 씬의 비주얼 디렉션 포함
  - `voiceover` — 스크립트를 TTS 오디오로 변환
  - `thumbnail` — 썸네일 이미지 생성. 텍스트 오버레이 + 비주얼 프롬프트
  - `description` — 유튜브 설명문 생성. 요약, 챕터 타임스탬프, 관련 링크
  - `tags` — SEO 태그/키워드 생성
- REQ-004: 각 스텝의 출력 형식:
  - script → 텍스트 (마크다운)
  - scenes → 씬 배열 (각 씬: 타임코드, 스크립트 구간, 비주얼 디렉션)
  - voiceover → 오디오 파일 (MP3/WAV)
  - thumbnail → 이미지 파일 (PNG/JPG)
  - description → 텍스트
  - tags → 문자열 배열
- REQ-005: script 스텝의 프롬프트에서 `{{article.sections}}`, `{{article.keyPoints}}` 등 원본 필드를 참조
- REQ-006: scenes 스텝은 `{{steps.script.output}}`을 참조하여 씬 분할
- REQ-007: voiceover 스텝은 TTS API를 호출. 음성 스타일(속도, 톤)은 에이전트 context에서 설정
- REQ-008: thumbnail 스텝은 이미지 생성 API를 호출. `{{article.hook}}`과 비주얼 스타일을 기반으로 프롬프트 구성
- REQ-009: description 스텝은 scenes의 타임코드를 활용하여 자동 챕터 생성
- REQ-010: 최종 출력은 모든 스텝 결과를 포함하는 디렉토리 구조 (텍스트 + 오디오 + 이미지)

---

## User Journeys

### 전체 AI 모드
1. 원본 아티클 입력
2. script: 아티클 기반 10분 분량 스크립트 자동 생성
3. scenes: 스크립트를 5-8개 씬으로 분할, 각 씬 비주얼 디렉션 생성
4. voiceover: 스크립트를 TTS로 오디오 변환
5. thumbnail: 아티클 hook + 비주얼 스타일로 썸네일 이미지 생성
6. description: 챕터 타임스탬프 포함 설명문 생성
7. tags: SEO 키워드 배열 생성
8. 결과: 출력 디렉토리에 모든 파일 저장

### 스크립트만 검수 (script: ai_approve, 나머지: ai)
1. script 스텝에서 생성된 스크립트를 사용자에게 표시
2. 사용자가 톤/길이/구조를 확인하고 approve 또는 편집
3. 승인된 스크립트를 기반으로 나머지 스텝 자동 실행

### 스크립트 직접 작성 (script: manual, 나머지: ai)
1. 사용자가 스크립트를 직접 작성
2. scenes부터 자동 실행 — 사용자 스크립트를 씬 분할하고 나머지 생성

### 썸네일만 커스텀 (thumbnail: manual, 나머지: ai)
1. script → scenes → voiceover → description → tags 자동 실행
2. thumbnail 스텝에서 사용자가 직접 이미지를 업로드하거나 프롬프트를 수정하여 재생성

---

## Edge Cases

- 원본 아티클이 매우 짧음 (sections 1개) → script가 짧은 영상(2-3분) 스크립트 생성. warning 표시
- TTS API 실패 → 재시도 후 실패 시 스크립트 텍스트만 출력하고 voiceover 스킵 가능
- 이미지 생성 API 실패 → 재시도 후 실패 시 썸네일 텍스트 프롬프트만 출력
- 스크립트가 너무 길어서 TTS 제한 초과 → 분할 처리 또는 사용자에게 축약 요청
- 원본 아티클에 dataPoints가 풍부 → scenes 비주얼 디렉션에 차트/그래프 제안 포함

---

## Dependencies

- article-schema (입력 포맷)
- agent-spec (에이전트 정의 구조)
- agent-runtime (실행 엔진)

## Depends On This

- pipeline-runner (리서치 → 유튜브 연결)
