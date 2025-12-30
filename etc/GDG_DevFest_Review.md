# 🚀 2025 GDG DevFest: AI Agent & Workflow Engineering
**📅 일시: 2025. 11. 30 (Sun)**

## 🛠️ Part 1. AI 시대 개발자의 핵심 역량 (Meta-Learning)
### 1. 학습과 비즈니스 마인드
- 메타 학습 (Learning how to learn): 기술 습득 속도보다 내가 어떻게 배우는가에 대한 메타 인지 능력 ⭐️

- 비즈니스 가치: 기술은 도구일 뿐, "서비스가 **어떤 가치를 창출**하는가"를 고민하는 성장이 필요

### 2. 개발자가 집중해야할 핵심 역량
- 예측 불가능한 변화를 인정하고 빠르게 적응하는 학습 능력
- 구현 능력보단 문제 정의와 결과가 맞는지 확인하는 검증 능력
- 적절한 단위로 요구사항 쪼개고 검증, 다른 방법 서치, 설득

...

## 🛠️ Part 2. 프롬프트 엔지니어링에서 '프로세스 설계'로

### 1. Paradigm shift

<Prompt (Stateless)>
- 어떻게 잘 물어볼까?
- 단일 모델에 의존
- 기억이 휘발

<Process (Stateful)>
- 어떻게 시스템을 만들까?
- 상태 관리
- Multi-Model Orchestration 

<h3> Prompt to(->) Process

---

### 2. Zen MCP (Model Context Protocol) Server
>Consensus
- 여러 모델의 합의 도출(의사결정 정확도 향상)

>Clink (CLI-to-CLI)
- 서브 에이전트 소환 및 작업 위임 -> 컨텍스트 오염을 방지

> Thinkdeep
- 복잡한 논리 추론(Thinking Mode)

> Code Review
- 자동화된 코드 검증(Quality Gate)
---

### 3. BMad v6: 워크플로우 중심 접근
시스템이 다음 단계 안내 \
pworkflow 접근 \
코드위키 \
Change \
correct-course \
Update \

Scale Adaptive System \

Quick Flow: 쉬운거 - 어떤거 해야하는지 단계 알려줌 \

<u>**BMAD Method**</u>: PRD(요구사항) → ARCH(아키텍처) → STORIES(작업 단위)의 선형적 흐름을 구축

<u>**Living Documents**</u>: "**문서화**는 개발의 일부"입니다. Change 발생 시 Correct-course를 통해 전체 정합성을 즉시 업데이트

### 아키텍쳐 설계
zen-mcp-server
BMAD-METHOD
adk mcp

...

## 🏗️ Part 3. ADK/A2A 기반 멀티 에이전트 프레임워크 설계 전략

### 1. 에이전트 설계의 핵심 철학
단순한 챗봇 ❌ \
특정 목적을 가진 **이벤트 플래너Event Planner**를 만든다고 가정했을 때의 설계 원칙

> **확장성** 있는 설계
- 도구(Tools) ↔ 에이전트(Agent) ↔ 모델(Model)을 분리하여, 모델이 바뀌어도 도구는 그대로 쓸 수 있게 설계.

> 맞춤형 모임 생성 시나리오
- 친구의 관심사, 장소, 일정 등 파편화된 데이터를 수집하여 최적의 결과를 도출하는 과정.

### 2. ADK의 핵심 구성 요소 및 기능
간편한 래핑 (Wrapping)
- 복잡한 API 호출이나 비즈니스 로직을 ADK로 간단하게 감싸서 에이전트가 즉시 사용 가능한 형태로 변환.

ADK Dev UI
- 에이전트를 빌드하면서 실시간으로 테스트하고, 답변이 계속 바뀌는 상황(Non-deterministic)을 모니터링하며 즉각 수정 가능.

State (상태 관리)
- 세션 안에서 단기 메모리를 유지. 단순 로그 추적이 아닌, 에이전트의 현재 상태를 제어하는 Runner가 핵심 역할 수행.

### 3. MCP(Model Context Protocol) Server 빌드 전략
에이전트와 도구 사이의 '미들웨어' 역할을 하는 MCP 서버의 상세 구조

**구조** 
```
MCP Client(Agent) → MCP Server → Tool(Google Maps, Search 등)
```

- 장점
    - (MCP 서버가 없으면 도구를 호출할 때마다 매번 구현해야 하지만) 서버를 구축하면 list_tool, call_tool 표준 규격을 통해 누구나 통합 가능.

- 배포
    - Cloud Run을 사용하여 서버리스로 배포하며, 클라이언트가 이를 호출하여 통합하는 방식.

### 4. 멀티 에이전트 워크플로우 (Multi-agent System)
하나의 에이전트가 모든 일을 하는 것이 아니라, 역할을 쪼개어 협업하는 시스템
```
< 에이전트 종류 >

소셜 프로필 에이전트: 사용자 관심사 및 정보 수집.

요약 에이전트: 수집된 정보를 바탕으로 브리핑 생성.

체크 에이전트: 최종 결과물이 조건을 만족하는지 검증하고 루프(Loop) 종료 여부 결정.
```
```
< 워크플로우 패턴 >

Sequential: 순차적 처리.

Parallel: 여러 정보를 동시에 수집.

Loop: 조건이 만족될 때까지 반복 수정.
```
### 5. A2A(Agent-to-Agent)와 Orchestrator
<u>Agent Card</u>: 에이전트의 명함. "나는 장소 추천을 잘해", "나는 일정 조율 전문이야"라는 정보를 담아 다른 에이전트에게 홍보.

<u>Orchestrator Agent</u>: 사용 가능한 에이전트 카드를 검색/저장하고, 전체 워크플로우를 지휘하는 '지휘자' 역할.

### 6. 평가 및 데이터 전략
<u>형식 일치 확인</u>: AI가 내놓은 결과가 우리가 원하는 JSON 형식인지 체크. 형식이 깨지면 Fail로 처리.

<u>프롬프트 전략</u>: 자연어 답변보다는 "어떤 형식의 JSON으로 보여줄지"를 프롬프트에 명확히 정의하여 시스템 안정성 확보.