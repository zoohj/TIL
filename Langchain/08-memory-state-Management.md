checkpointer
그래프의 State스냅샷을 저장하고 복원
thread_id로 대회 세션 구분


```python 
memory = MemorySaver()

# checkpointer와 함께 컴파일
graph_with_memory = simple_builder.compile(checkpointer=memory)

config = {"configurable": {"thread_id": "session-1"}}

```


## 저장된 state 확인
state = graph.get_state(config)
![alt text](../images/get_state.png)


## Long-term Memory
유저/네임스페이스 단위
명시적 삭제까지
유저 선호도, 학습 내용

### Store
store데이터 구조:
namespace, key, value

`*, store: BaseStore`
