# Agent Spec

## Overview

에이전트를 YAML 파일로 정의하는 구조. 에이전트의 이름, 스텝 목록, 각 스텝의 모드(AI/AI+Approve/Manual), 프롬프트, 컨텍스트 주입 설정을 선언적으로 기술한다. 에이전트 런타임은 이 spec 파일을 읽어서 실행한다.

---

## Requirements

- REQ-001: 에이전트는 하나의 YAML 파일로 정의된다
- REQ-002: spec 파일은 다음 구조를 가진다:
  - `name` — 에이전트 ���름
  - `description` — 에이전트 설명
  - `input` — 입력 타입 (article, topic, raw text 등)
  - `output` — 출력 타입 (article, channel-content 등)
  - `context` — 글로벌 컨텍스트 (브랜드 보이스, 도메인 지식 등). 모든 스텝에 주입
  - `steps[]` — 스텝 배열 (순서대로 실행)
- REQ-003: 각 스텝은 다음 필드를 가진다:
  - `id` — 스텝 고유 ID
  - `name` — 사람이 읽을 수 있는 이름
  - `mode` — `ai` | `ai_approve` | `manual`
  - `prompt` — AI 모드에서 사용할 프롬프트 템플릿
  - `context` — 이 스텝에만 추가 주입할 컨텍스트 (optional)
  - `output_key` — 결과를 저장할 키 이름
- REQ-004: 프롬프트에서 이전 스텝 결과를 변수로 참조 가능 (예: `{{steps.script.output}}`)
- REQ-005: 프롬프트에서 원본 아티클 필드를 변수로 참조 가능 (예: `{{article.title}}`, `{{article.sections}}`)
- REQ-006: spec 파일 로딩 시 validation — 필수 필드 누락, 잘못된 mode 값, 존재하지 않는 변수 참조 등 에러 보고
- REQ-007: spec 파일의 TypeScript 타입 정의를 SSOT로 두고, validation은 이 타입에서 파생

---

## User Journeys

### 새 에이전트 정의
1. 사용자가 `agents/` 디렉토리에 YAML 파일 생성
2. name, steps, 각 스텝의 mode와 prompt를 작성
3. 파이프라인 실행 시 런타임이 spec을 로드하고 validation
4. validation 통과 시 실행, 실패 시 에러 위치와 이유 표시

### 기존 에이전트 커스터마이징
1. 기본 제공 에이전트(research, youtube) spec 파일을 열어봄
2. 프롬프트 텍스트를 자기 도메인에 맞게 수정
3. 스텝의 mode를 변경 (예: ai → ai_approve로 검수 추가)
4. 파이프라인 재실행. 변경사항 즉시 반영

### 스텝 모드 변경
1. 특정 스텝의 mode를 `ai` → `manual`로 변경
2. 파이프라인 실행 시 해당 스텝에서 일시 정지
3. 사용자가 직접 내용을 입력/편집
4. 입력 완료 후 다음 스텝으로 진행

---

## Edge Cases

- 스텝이 1개뿐인 에이전트 → 유효
- 모든 스텝이 manual인 에이전트 → 유효 (AI 없이 구조만 사용)
- 프롬프트에서 아직 실행되지 않은 스텝 결과를 참조 → validation 에러
- context에 매우 긴 텍스트 (도메인 지식 문서 전체) → 유효하지만 토큰 제한 warning

---

## Dependencies

- article-schema (프롬프트에서 아티클 필드 참조)

## Depends On This

- agent-runtime (spec을 읽어서 실행)
- research-agent (spec 파일로 정의됨)
- youtube-agent (spec 파일로 정의됨)
