## 🛠️ System Architecture & Tech Stack

**Done-it**은 서비스의 안정성과 AI의 유연함을 동시에 확보하기 위해 **Polyglot Architecture**를 지향합니다.

| 계층 (Layer) | 역할 | 적용 기술 |
| :--- | :--- | :--- |
| **API Gateway** | 클라이언트 요청 수신, 인증 및 라우팅 | **Java / Spring Boot** |
| **AI Service** | 자연어 이해(NLU) 및 업무 추출 로직 | **Python / FastAPI (LangChain)** |
| **Data Store** | 메시지 로그 및 업무 데이터, 벡터 저장 | **PostgreSQL (pgvector)** |
| **Cache/Queue** | 실시간 알림 및 비동기 작업 처리 | **Redis** |

---

## Key Technical Challenges

### 1. 실행형 AI (Function Calling)
단순 요약을 넘어 사용자의 의도를 파악하고 외부 API를 호출.
* "회의 일정 잡아줘" -> Google/Naver Calendar API 연동
* "업무 등록해줘" -> 프로젝트 관리 DB 연동

### 2. RAG 기반 히스토리 관리
과거의 대화 맥락을 기억하여 지능형 추론을 수행.
* "저번에 말했던 그 이슈 해결됐어?"와 같은 질문에 답변 가능하도록 벡터 DB 활용.

### 3. 비동기 처리 및 성능 최적화
사용자 경험을 저해하지 않기 위해 무거운 AI 로직은 비동기로 처리.
* **Redis/Celery**를 활용하여 요약 및 외부 연동 작업을 백그라운드에서 수행.

---

## 🏗️ Backend Layered Architecture
스프링 표준 계층 구조를 프로젝트에 투영. 

* **Controller**: 외부 요청 수신 및 응답 반환 
* **Service**: 중복 업무 검증 등 핵심 비즈니스 로직 
* **Repository**: 인터페이스 설계를 통한 데이터 저장소 유연성 확보 
* **Domain**: `Member`, `Task` 등 비즈니스 객체 관리