LangCahin
다양한 LLM 제공자를 통일된 인터페이스로 사용

1. 단순 api 호출
2. 체인구성(prompt+model+parser)
3. 메모리/Tool추가
4. RAG - 자체문서 검색
5. Agent 여러 도구 조합

### LangChain 생태계
| 패키지 | 역할 | 설명 |
|--------|------|------|
| `langchain-core` | 핵심 | 기본 인터페이스, LCEL, 메시지 타입 등 |
| `langchain` | 체인/메모리 | 체인 구성, 메모리, 에이전트 등 고수준 기능 |
| `langchain-openai` | OpenAI 연동 | ChatOpenAI, OpenAI Embeddings 등 |
| `langchain-google-genai` | Google 연동 | ChatGoogleGenerativeAI 등 |
| `langchain-community` | 커뮤니티 통합 | 다양한 서드파티 Tool, 벡터 DB 등 |
| `langgraph` | Agent 프레임워크 | 상태 기반 Agent 구축 (Part 3에서 학습) |

```
langchain-core (핵심)
    ├── langchain (체인/메모리)
    ├── langchain-openai (모델 연동)
    ├── langchain-google-genai (모델 연동)
    ├── langchain-community (커뮤니티)
    └── langgraph (Agent)
```


### LangSmith
LangSmith는 LangChain 공식 디버깅/모니터링 플랫폼. 체인의 실행 과정을 시각적으로 추적(트레이싱)

---


## LCEL 파이프라인 (LangChain Expression Language)

- `|` 연산자로 프롬프트, 모델, 파서를 연결하여 체인을 구성한다
- 데이터가 왼쪽에서 오른쪽으로 흘러간다

```
입력 → PromptTemplate → ChatModel → OutputParser → 출력
```


## Prompt Chaining 패턴

- 하나의 체인 출력을 다음 체인의 입력으로 연결하는 패턴
- LCEL의 `|` 파이프라인이 곧 Prompt Chaining
- 복잡한 작업을 작은 단계로 나누어 처리할 수 있다

`RunnableLambda`로 감싸는 이유: `chain1`의 출력은 `str`이지만, `chain2`의 입력은 `{"text": ...}` 딕셔너리여야 하기 때문이다. 타입 변환 어댑터 역할을 한다.

---

## RunnablePassthrough

**왜 `RunnablePassthrough`가 필요한가?**

LCEL 체인은 기본적으로 이전 단계의 출력이 다음 단계의 입력으로 들어간다. 그런데 프롬프트에 변수가 여러 개일 때 문제가 생긴다.

```python
# 프롬프트에 context와 question 두 변수가 필요한 경우
prompt = "Context: {context}\n질문: {question}"
```


이때 사용자 입력(`question`)은 그대로 전달하면서, 다른 값(`context`)은 별도로 만들어서 합쳐야 한다. `RunnablePassthrough`는 이 "그대로 전달" 역할을 한다.

- `RunnablePassthrough()` — 입력을 그대로 통과시킨다
- `RunnablePassthrough.assign(key=fn)` — 입력을 유지하면서 새 키-값을 추가한다

RAG에서 가장 많이 쓰이는 패턴이다. 사용자의 질문은 유지하면서, 검색 결과를 context로 추가해야 하기 때문이다.

```python
from langchain_core.runnables import RunnablePassthrough

# RunnablePassthrough: 입력을 그대로 통과시키면서 다른 값과 조합
# RAG에서 "검색 결과 + 원래 질문"을 함께 전달할 때 핵심적으로 사용된다

prompt = ChatPromptTemplate.from_messages([
    ("system", "다음 context를 참고하여 질문에 답해줘.\n\nContext: {context}"),
    ("human", "{question}"),
])

# RunnablePassthrough.assign(): 기존 입력을 유지하면서 새 키를 추가
chain_with_passthrough = RunnablePassthrough.assign(
    context=lambda x: f"Python은 1991년에 만들어진 프로그래밍 언어입니다. 귀도 반 로섬이 개발했습니다."
) | prompt | llm | parser

result = chain_with_passthrough.invoke({"question": "Python은 누가 만들었어?"})

print(result)
```


## 비용 모니터링
LangChain은 `get_openai_callback()`이라는 비용 추적 도구 제공


## LLM API 에러 핸들링

| 에러 | HTTP 코드 | 원인 | 대응 |
|------|-----------|------|------|
| Rate Limit | 429 | 짧은 시간에 너무 많은 요청 | 자동 재시도 (`max_retries`) |
| Timeout | 408/504 | 서버 응답이 너무 느림 | 타임아웃 설정 (`request_timeout`) |
| Server Error | 500 | OpenAI 서버 장애 | 재시도 또는 fallback 모델 |
| Auth Error | 401 | API 키가 잘못됨 | `.env` 파일 확인 |

- max_retries: 자동 재시도 횟수
- request_timeout: 응답이 이 시간(초) 안에 안 오면 에러 발생
- with_fallbacks: 메인 모델이 실패하면 백업 모델로 자동 전환

## 캐싱 가능


## batch와 stream

`invoke()`가 "하나 보내고, 하나 받기"라면:
- `batch()` — 여러 입력을 한 번에 보내고, 결과 리스트를 받는다 (병렬 처리)
- `stream()` — 하나를 보내고, 응답을 토큰 단위로 조각(chunk)씩 받는다

| 메서드 | 입력 | 출력 | 비동기 버전 |
|--------|------|------|-------------|
| `invoke()` | 하나 | 하나 | `ainvoke()` |
| `batch()` | 리스트 | 리스트 | `abatch()` |
| `stream()` | 하나 | 이터레이터 (chunk) | `astream()` |

### batch

여러 입력을 동시에 처리할 때 사용한다. 내부적으로 병렬 실행되므로 하나씩 `invoke()`를 반복하는 것보다 빠르다.
`max_concurrency`로 동시 실행수 제한 가능
### stream

ChatGPT처럼 응답이 한 글자씩 나타나게 하려면 `stream()`을 사용한다. 긴 응답을 기다리지 않고 바로바로 출력할 수 있어서 UX가 좋아진다.

`stream()`의 각 chunk는 토큰 단위의 문자열 조각이다. `end=""`로 출력하면 이어 붙여지면서 자연스러운 스트리밍 효과가 난다.

### astream (비동기 스트리밍)

FastAPI나 비동기 환경에서는 `astream()`을 사용한다. `stream()`과 동일하지만 `async for`로 받는다.


Jupyter 노트북은 이벤트 루프가 이미 실행 중이라 `async for`를 셀에서 바로 쓸 수 있다. 일반 Python 스크립트에서는 `asyncio.run()`으로 감싸야 한다.

```python
# 일반 스크립트에서 사용할 때
import asyncio

async def main():
    async for chunk in chain.astream({"question": "FastAPI가 뭐야?"}):
        print(chunk, end="", flush=True)

asyncio.run(main())
```


rag의 단점은 주어진 컨텍스트에서만 찾음
