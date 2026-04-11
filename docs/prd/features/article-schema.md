# Article Schema

## Overview

모든 에이전트가 소비/생산하는 구조화된 원본 아티클 포맷. 리서치 에이전트가 생성하고, 채널 에이전트(유튜브 등)가 이 포맷을 입력으로 받아 채널별 콘텐츠를 만든다. 에이전트 간 데이터 교환의 공유 계약(contract).

---

## Requirements

- REQ-001: 원본 아티클은 고정된 필드 스키마를 가진다 (MVP에서는 사용자 확장 불가)
- REQ-002: 스키마는 다음 필드를 포함한다:
  - `title` — 아티클 제목
  - `hook` — 한 줄 요약. 소셜/썸네일용 훅 라인
  - `tldr` — 3-5문장 요약
  - `sections[]` — 본문 섹션 배열. 각 섹션은 heading + body + (optional) dataPoints
  - `keyPoints[]` — 핵심 포인트 배열. 쓰레드/챕터 분할의 소스
  - `dataPoints[]` — 수치/통계 데이터. 차트, 카드, 인포그래픽 소스
  - `sources[]` — 출처 URL + 제목
  - `tags[]` — 분류 태그
  - `metadata` — 생성일시, 도메인, 언어, 작성 에이전트 ID
- REQ-003: 스키마는 YAML과 JSON으로 직렬화/역직렬화 가능해야 한다
- REQ-004: 필수 필드(title, sections)와 선택 필드를 구분한다. 필수 필드 누락 시 validation 에러
- REQ-005: TypeScript 타입 정의를 SSOT로 두고, YAML/JSON validation은 이 타입에서 파생

---

## User Journeys

### 리서치 에이전트가 아티클 생성
1. 리서치 에이전트의 마지막 스텝이 원본 아티클을 출력
2. 출력은 article-schema를 따르는 YAML 파일로 저장
3. validation 통과 시 다음 에이전트(유튜브 등)의 입력으로 전달
4. validation 실패 시 에러 메시지와 함께 해당 스텝 재실행 또는 Manual 모드로 전환

### 사용자가 직접 아티클 작성 (Manual 모드)
1. 사용자가 article-schema 템플릿을 복사
2. 필드를 채워서 YAML 파일로 저장
3. 파이프라인 실행 시 이 파일을 원본으로 사용

### 채널 에이전트가 아티클 소비
1. 유튜브 에이전트가 원본 아티클 YAML을 읽음
2. sections → 스크립트/씬 분할, keyPoints → 챕터, hook → 썸네일 텍스트 등 필드별로 소비
3. 필요한 필드가 비어있으면 해당 스텝을 스킵하거나 기본값 사용

---

## Edge Cases

- sections가 1개뿐인 짧은 아티클 → 유효. 채널 에이전트가 짧은 콘텐츠로 처리
- dataPoints가 없는 아티클 → 유효. 데이터 시각화 스텝 스킵
- sources가 없는 아티클 → 유효하지만 warning 표시
- 매우 긴 아티클 (sections 20개+) → 유효. 채널 에이전트가 요약/선택

---

## Dependencies

- None (최상위 피처. 다른 모든 피처가 이 스키마에 의존)

## Depends On This

- agent-runtime (스키마 validation)
- research-agent (아티클 생성)
- youtube-agent (아티클 소비)
- pipeline-runner (에이전트 간 아티클 전달)
