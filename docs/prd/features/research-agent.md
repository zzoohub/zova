# Research Agent

## Overview

주제에서 출발하여 원본 아티클을 생성하는 에이전트. 트렌드/아이디어 수집 → 소스 리서치 → 구조화된 원본 아티클 초안 생성까지의 서브 파이프라인을 실행한다. 콘텐츠 파이프라인의 시작점이자, article-schema를 생산하는 유일한 에이전트.

---

## Requirements

- REQ-001: agent-spec 형식의 YAML로 정의되며, agent-runtime 위에서 실행
- REQ-002: 다음 스텝으로 구성 (각 스텝의 mode는 사용자가 설정):
  - `topic-ideation` — 주제/아이디어 선정. 사용자가 직접 입력하거나, AI가 트렌드 기반으로 제안
  - `source-research` — 선정된 주제에 대한 소스 수집. 웹 검색, 참고 자료 정리
  - `outline` — 아티클 구조 설계. sections, keyPoints 배열의 골격
  - `draft` — article-schema를 따르는 완성된 원본 아티클 생성
- REQ-003: ���종 출력은 article-schema를 만족하는 YAML 파일
- REQ-004: 각 스텝의 프롬프트는 사용자가 커스터마이징 가능 (도메인 지식, 톤, 타겟 독자 등)
- REQ-005: source-research 스텝은 웹 검색 결과를 컨텍스트로 주입. 사용자가 직접 자료를 넣을 수도 있음 (Manual 모드)
- REQ-006: draft 스텝 출력이 article-schema validation을 통과해야 함. 실패 시 재생성

---

## User Journeys

### 주제부터 시작 (topic-ideation: manual, 나머지: ai)
1. 사용자가 주제를 직접 입력 (예: "2026년 AI 에이전트 트렌드")
2. source-research 스텝이 웹 검색으로 관련 소스 수집
3. outline 스텝이 아티클 구조 생성
4. draft 스텝이 article-schema 형식의 원본 아티클 생성
5. 결과 YAML 파일이 다음 에이전트(유튜브)의 입력으로 전달

### 완전 자동 (모든 스텝: ai)
1. topic-ideation 스텝이 설정된 도메인/트렌드 기반으로 주제 제안
2. 이하 자동으로 소스 수집 → 구조 설계 → 아티클 생성
3. 최종 아티클 출력

### 검수 모드 (draft: ai_approve, 나머지: ai)
1. topic-ideation → source-research → outline 자동 실행
2. draft 결과를 사용자에게 표시
3. 사용자가 검토 후 approve/reject/편집
4. 승인된 아티클이 최종 출력

### 사용자가 자료를 직접 제공 (source-research: manual)
1. topic-ideation에서 주제 확정
2. source-research에서 사용자가 참고 자료(URL, 텍스트)를 직접 입력
3. outline → draft는 제공된 자료 기반으로 AI가 생성

---

## Edge Cases

- 웹 검색 결과가 없거나 부족한 주제 → source-research가 빈 결과 반환. draft가 제한된 컨텍스트로 생성하되 warning 표시
- draft 출력이 article-schema validation 실패 → 자동 재생성 (최대 3회). 실패 시 raw output 표시
- 매우 넓은 주제 (예: "AI") → outline 스텝에서 범위를 좁히는 프롬프트 필요. 사용자 컨텍스트에 스코핑 가이드 포함

---

## Dependencies

- article-schema (최종 출력 포맷)
- agent-spec (에이전트 정의 구조)
- agent-runtime (실행 엔진)

## Depends On This

- youtube-agent (리서치 에이전트의 출력을 입력으로 사용)
- pipeline-runner (리서치 → 유튜브 연결)
