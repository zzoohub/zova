# Zova — PRD

**Status:** Draft
**Author:** zzoo
**Last Updated:** 2026-04-11

---

## Table of Contents

1. [Problem / Opportunity](#1-problem--opportunity)
2. [Target Users](#2-target-users)
3. [Proposed Solution](#3-proposed-solution)
4. [Success Metrics](#4-success-metrics)
5. [Feature Overview](#5-feature-overview)
6. [Dev Order](#6-dev-order)
7. [Scope & Non-Goals](#7-scope--non-goals)
8. [Assumptions, Constraints & Risks](#8-assumptions-constraints--risks)

---

## 1. Problem / Opportunity

**The Problem**

다채널 콘텐츠 운영자는 하나의 주제로 블로그, 유튜브, X, 인스타, 뉴스레터 등 채널별 콘텐츠를 따로 만든다. 각 채널은 미디어 특성이 달라 고유한 제작 과정이 필요하다 — 유튜브는 스크립트·씬 분할·썸네일, 블로그는 SEO·본문·메타, X는 훅·쓰레드 분할 등. 같은 팩트와 논리를 채널마다 반복 작성하고, 톤과 포맷을 조정하는 데 시간 대부분을 쓴다.

AI 도구(ChatGPT, Claude 등)를 개별로 써도 채널마다 프롬프트를 따로 돌리고, 결과 일관성을 직접 확인하고, 수동으로 각 플랫폼에 올려야 하는 건 마찬가지다. 주 3회 이상 다채널 발행 시 하나의 주제당 2-4시간이 반복 변환 작업에 소요된다.

더 큰 문제는 — 도메인(크립토, 테크, 라이프스타일 등), 톤, 채널 전략이 사람마다 다른데, 기존 AI 콘텐츠 도구는 정해진 워크플로우를 강제한다. 어디까지 AI로 자동화하고 어디서 직접 개입할지 선택할 수 없다.

**현재 대안:**
- ChatGPT/Claude에 채널별로 프롬프트를 수동 반복 → 일관성 보장 안 됨, 비효율적
- n8n/Zapier로 워크플로우 구성 → 콘텐츠 특화 프리미티브 없음, 설정 복잡도 높음
- Jasper/Copy.ai 같은 AI 카피 도구 → 단편적 텍스트 생성, 채널별 파이프라인 부재

**Why Now**

LLM의 롱폼 생성 품질이 실사용 수준에 도달했지만, 콘텐츠 제작의 전체 흐름(아이디어 → 리서치 → 원본 → 채널별 제작)을 에이전트로 구조화하고 자동화 수준을 스텝 단위로 조절할 수 있는 도구가 없다.

---

## 2. Target Users

다채널 콘텐츠를 운영하는 크리에이터, 마케터, 소규모 미디어 팀. 도메인 불문.

**JTBD:**
> "주제 하나를 정하면, 블로그·X·유튜브 등 내가 운영하는 채널에 맞는 콘텐츠가 각각 나왔으면 좋겠다. 어떤 단계는 AI가 알아서 해주고, 어떤 단계는 내가 직접 쓰거나 확인하고 싶다."

---

## 3. Proposed Solution

**Elevator pitch:** Zova는 AI 에이전트 기반 콘텐츠 파이프라인이다. 구조화된 원본 아티클을 중심으로, 채널별 AI 에이전트가 각 미디어 타입에 맞는 콘텐츠를 제작한다. 모든 스텝은 AI 자동/승인 후 진행/수동 입력 중 선택 가능하며, 에이전트 설정은 spec 파일로 정의한다.

**Core value propositions:**

1. **원본 하나, 다채널 콘텐츠** — 구조화된 원본 아티클에서 채널별 에이전트가 각각의 제작 파이프라인을 실행. 팩트/논리를 한 번만 검증하면 된다.
2. **스텝별 자동화 조절** — 파이프라인의 모든 스텝을 AI/AI+Approve/Manual 모드로 설정. 완전 자동부터 거의 수동까지, 사용자 워크스타일에 맞게 조합한다.
3. **변경에 유연한 파이프라인** — 에이전트 정의, 프롬프트, 컨텍스트를 spec 파일로 관리. 도메인 지식, 브랜드 보이스, 채널 전략을 자유롭게 커스터마이징.

**Mental model:** CI/CD 파이프라인과 같다. Zova는 뼈대(스텝 흐름, 모드, 에이전트 런타임)를 제공하고, 각 스텝의 실제 동작(프롬프트, 컨텍스트, 소스)은 사용자가 spec 파일로 정의한다.

---

## 4. Success Metrics

- 원본 아티클 하나에서 유튜브 콘텐츠 풀세트(스크립트, 보이스오버, 썸네일, 설명문, 태그)가 나오고, 최소 편집으로 실제 발행 가능한 품질
- 유튜브 콘텐츠 제작 시간이 주제당 4-8시간 → 1시간 이하로 감소
- 사용자가 에이전트 프롬프트를 자기 도메인에 맞게 튜닝하고, 반복 사용

---

## 5. Feature Overview

| Feature | Description | Spec |
|---------|-------------|------|
| article-schema | 원본 아티클의 구조화된 공유 포맷 정의 | [features/article-schema.md](features/article-schema.md) |
| agent-spec | 에이전트와 스텝을 spec 파일(YAML)로 정의하는 구조 | [features/agent-spec.md](features/agent-spec.md) |
| agent-runtime | 에이전트를 실행하는 코어 엔진 (스텝 실행, 모드 처리, 컨텍스트 주입) | [features/agent-runtime.md](features/agent-runtime.md) |
| research-agent | 트렌드 수집 → 아이디어 발굴 → 소스 리서치 → 원본 초안 생성 | [features/research-agent.md](features/research-agent.md) |
| youtube-agent | 원본 아티클 → 스크립트, 씬 분할, 보이스오버, 썸네일, 설명문, 태그 | [features/youtube-agent.md](features/youtube-agent.md) |
| pipeline-runner | 리서치 에이전트 → 원본 아티클 → 채널 에이전트들의 전체 흐름 실행 | [features/pipeline-runner.md](features/pipeline-runner.md) |

---

## 6. Dev Order

### v0.1 — Core Engine (파이프라인이 돌아가는 최소 상태)
1. **article-schema** — 모든 에이전트가 이 포맷을 소비/생산. 먼저 확정해야 나머지가 동작.
2. **agent-spec** — 에이전트를 정의하는 spec 파일 구조. 런타임이 이걸 읽어서 실행.
3. **agent-runtime** — 스텝 실행, AI/Approve/Manual 모드 처리, 컨텍스트 주입.

### v0.2 — MVP 에이전트 (실제 콘텐츠가 나오는 상태)
4. **research-agent** — 원본 아티클 생성. 콘텐츠 파이프라인의 시작점.
5. **youtube-agent** — 가장 복잡한 채널(텍스트+오디오+이미지+영상). 이게 되면 나머지 채널은 부분집합.

### v0.3 — 파이프라인 연결
6. **pipeline-runner** — 리서치 → 원본 → 유튜브 에이전트의 전체 흐름 실행.

---

## 7. Scope & Non-Goals

**In scope:**
- 원본 아티클 구조화 포맷 정의
- YAML spec 기반 에이전트 정의 및 커스터마이징
- 에이전트 런타임 (스텝 실행, 3가지 모드 처리)
- MVP 에이전트 2종 (리서치, 유튜브)
- 멀티모달 AI 생성 (텍스트, 오디오/TTS, 이미지, 영상)
- 에이전트 간 전체 파이프라인 실행

**Out of scope:**
- GUI 에디터 — spec 파일 기반 MVP 이후에 고려. 초기 사용자는 YAML 편집 가능한 수준.
- 채널 배포 자동화 (API 연동 자동 발행) — 콘텐츠 생성 품질 검증이 우선. 배포는 수동.
- 블로그/X/인스타 등 추가 채널 에이전트 — 유튜브로 코어 검증 후 확장. 유튜브가 되면 텍스트 전용 채널은 부분집합.
- 팀 협업 기능 — 1인 사용자 워크플로우 검증 후.
- 수익화/과금 — 제품 가치 검증 후.

---

## 8. Assumptions, Constraints & Risks

**Assumptions:**
- LLM이 생성한 롱폼 콘텐츠가 최소 편집으로 발행 가능한 품질을 낼 수 있다. 현재 Claude/GPT-4 수준 기준.
- 사용자가 YAML spec 파일을 편집할 수 있다. MVP 타겟이 기술적 역량이 있는 크리에이터/마케터.
- 채널별 콘텐츠 품질은 프롬프트 튜닝에 크게 의존한다. 좋은 기본 프롬프트를 제공하되, 사용자 커스터마이징이 핵심.

**Constraints:**
- 에이전트 실행은 LLM API 호출 비용이 발생. 원본 + 채널 3개 기준 주제당 비용 관리 필요.
- spec 파일 기반이므로 MVP에서는 웹 UI 없이 CLI 또는 로컬 실행.

**Risks:**

| Risk | Severity | Likelihood | Mitigation |
|------|----------|------------|------------|
| 채널별 변환 품질이 발행 수준에 못 미침 | High | Medium | 채널별 기본 프롬프트를 충분히 튜닝하고, 사용자 커스터마이징으로 보완. AI+Approve 모드로 검수 가능 |
| 원본 아티클 스키마가 다양한 콘텐츠 유형을 커버 못 함 | Medium | Medium | 고정 스키마로 시작하되, 부족한 필드가 발견되면 빠르게 확장 |
| YAML spec 편집이 비개발자에게 진입장벽 | Medium | High | MVP는 기술적 사용자 대상. GUI 레이어를 이후 추가 |
| LLM API 비용이 주제당 예상 이상으로 발생 | Medium | Low | 스텝별 토큰 사용량 추적, Manual 모드로 비용 절감 가능 |
