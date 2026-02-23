
# FastAPI 프로젝트 초기 설정 및 환경 구축

## 1. 패키지 매니저 설정 (uv)

- 프로젝트 초기화 및 라이브러리 설치

    ```bash
    # 프로젝트 생성
    uv init fastapi_basic
    cd fastapi_basic

    # FastAPI 표준 패키지 설치
    uv add "fastapi[standard]"

    # 데이터베이스 및 환경 변수 관련 라이브러리 추가
    uv add sqlalchemy psycopg2-binary python-dotenv
    ```

- 애플리케이션 실행 (개발 모드)
  
    ```bash
    uv run fastapi dev 
    ```

## 2. 환경 변수 관리 (.env)

    DATABASE_URL=postgresql://{db_name}:{PASSWORD}@localhost:5432/mysite
    JWT_SECRET_KEY=your-secret-key  # openssl rand -hex 32로 생성 권장
    

## 3. 데이터베이스 연결 설정 (database.py)

```python 
import os
from dotenv import load_dotenv
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker, DeclarativeBase

# 1. 환경 변수 로드
load_dotenv()
DATABASE_URL = os.getenv("DATABASE_URL")

# 2. DB 엔진 생성
engine = create_engine(DATABASE_URL)

# 3. 세션 생성기 정의 (자동 커밋/플러시 비활성화로 안정성 확보)
SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)

# 4. SQLAlchemy 2.0 스타일의 Base 클래스
class Base(DeclarativeBase):
    pass

# 5. DB 세션 의존성 주입 함수
def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close() # 작업 완료 후 세션 자동 반환
```

## 4. 핵심 아키텍처 및 흐름 정리

운영 중인 서비스의 장애를 예측하고 유지보수를 용이하게 하기 위해 **3계층 구조**를 지향.

| 계층 | 역할 | 주요 도구 |
| --- | --- | --- |
| **Schema** | 요청/응답 형식 지정, 데이터 유효성 검사 | `Pydantic` (DTO 역할) |
| **Router** | 엔드포인트 정의, 클라이언트 입력 수신 | `FastAPI APIRouter` |
| **Service** | 비즈니스 로직 처리, 트랜잭션 관리, 예외 처리 | `SQLAlchemy Session` |

1. `Schema`: 요청/응답 형식을 지정.

2. `Router`: 클라이언트 요청을 받아 `Service`로 전달.

3. `Service`: 비즈니스 로직(예외 처리 등)을 실행하고 `Repository`를 통해 DB를 조작.


* **💡 `Pydantic`의 역할**
  * **Validation**: 입력된 데이터가 올바른 형식인지 검사.
  * **Serialization**: 파이썬 객체를 JSON 등 클라이언트가 읽을 수 있는 형태로 변환.
  * **DTO(Data Transfer Object)**: 요청(Request)과 응답(Response) 스케마를 분리하여 필요한 데이터만 외부에 노출.
  * **Annotated 활용**: Annotated를 사용하여 필드에 제약 조건을 추가하거나 의존성 주입을 명확하게 선언.



#### +@ VS Code에서 cmd + d를 활용하면 같은 단어를 동시에 수정 가능.


