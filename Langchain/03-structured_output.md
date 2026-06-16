## structured output

python 객체로 받을 수 있음
pydantic모델


✅ 구조화된 응답 — 바로 코드에서 사용 가능
SentimentResult(sentiment='부정', intensity='강함', reason="'최악'이라는 표현")

### with_structured_output()

json 생성, pydantic모델로 변환
field: llm 이 필드에 무엇을 넣어야하는지

promt | llm |parser 
>> structured_llm.invoke()

추가지시 필요한 경우 >> prompt | structured_llm

structured_llm

## structured output 실패 처리

```python
# 실패 시 최대 3번 자동 재시도
chain = prompt | structured_llm
safe_chain = chain.with_retry(stop_after_attempt=3) # 체인 재시도
```

### max_retries vs with_retry

이 둘은 재시도하는 **위치**가 다르다.

```
[LangChain 체인] → [LLM API 요청] → 네트워크/서버 → [응답 수신] → [파싱/검증]
                    ↑ max_retries                       ↑ with_retry
```

| | `max_retries` | `.with_retry()` |
|---|---|---|
| **설정 위치** | `ChatOpenAI(max_retries=3)` | `chain.with_retry(stop_after_attempt=3)` |
| **재시도 주체** | OpenAI SDK (HTTP 클라이언트) | LangChain (체인) |
| **대상 에러** | 네트워크 에러, 타임아웃, 429 Rate Limit | 파싱 실패, 스키마 불일치 |
| **동작** | 같은 HTTP 요청을 다시 보냄 | 체인 전체를 처음부터 다시 실행 (= LLM 재호출) | 

- `max_retries`: **서버가 응답을 안 줄 때** — OpenAI SDK가 같은 요청을 재전송
- `with_retry`: **응답은 왔는데 내용이 잘못됐을 때** — LangChain이 체인을 처음부터 다시 실행하므로 LLM API를 다시 호출한다






