hugging face / Ollama
<details>

### Ollama 설치

공식 사이트(https://ollama.com)에서 설치 파일을 다운로드한다.

- **macOS**: `brew install ollama` 또는 공식 사이트에서 다운로드
- **Windows**: 공식 사이트에서 설치 파일 다운로드

설치 후 터미널에서 모델을 다운로드한다:

```bash
# 모델 다운로드 (0.5b는 가장 가벼운 모델로, CPU에서도 무리 없이 실행된다)
ollama pull qwen2.5:0.5b

# 서버가 꺼져 있다면 직접 실행
ollama serve

# LangChain Ollama 연동 패키지 설치
uv add langchain-ollama
```

```python
from langchain_ollama import ChatOllama

# 모델만 교체하면 된다
local_llm = ChatOllama(model="qwen2.5:0.5b", temperature=0)

# 나머지는 동일
chain = prompt | local_llm | parser

result = chain.invoke({
    "role": "Python",
    "question": "for 문을 설명해줘",
})

print(result)
```
</details>



# RAG
## 검색(Retrieval) -> 응답(Generation)
```bash
[ 문서 준비 단계 (오프라인) ]
문서 → 로드(DocumentLoader) → 분할(TextSplitter) → 임베딩 → 벡터 DB 저장

[ 질의 응답 단계 (온라인) ]
질문 → 임베딩 → 벡터 DB 검색 → 관련 문서 추출 → LLM에 전달 → 응답 생성
```
### Document Loader

RAG에서 사용할 문서를 로드하는 도구이다. LangChain은 다양한 형식의 문서 로더를 제공한다.

| 로더 | 형식 | 패키지 |
|------|------|--------|
| `TextLoader` | `.txt` | `langchain_community` |
| `PyPDFLoader` | `.pdf` | `pypdf` |
| `CSVLoader` | `.csv` | `langchain_community` |
| `WebBaseLoader` | 웹페이지 | `langchain_community` |

### Text Splitter
chunk_size: 청크 최대 크기(문자 수)
chunk_overlap: 인접 청크 간 겹치는 부분

#### RecursiveCharacterTextSplitter


# vectorDB



# RAG pipeline