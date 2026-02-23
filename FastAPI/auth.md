# 🔐 FastAPI JWT 인증 시스템 구축 요약

FastAPI에서 보안(Security)과 유저 인증을 처리하기 위해 PyJWT와 bcrypt를 조합하여 표준적인 OAuth2 워크플로우 구현

## 1. 환경 설정 및 라이브러리 

- 민감한 정보는 .env 파일로 관리
- `SECRET_KEY` 유출 주의

```bash
uv add PyJWT bcrypt email-validators
```
  - PyJWT: JWT 토큰 생성 및 검증
  - bcrypt: 비밀번호 해싱 (단방향 암호화)
  - email-validators: 이메일 형식 유효성 검사


- 보안키(secret_key) 생성
  - ```bash openssl rand -hex 32 ```

`.env`
```
JWT_SECRET_KEY=your-secret-key
JWT_ALGORITHM=HS256
JWT_EXPIRE_MINUTES=30
```

## 2. 의존성 주입(Dependency Injection)과 유저 식별

- `Depends`를 활용하여 현재 로그인한 유저를 식별하는 로직을 모듈화

- `get_current_user`의 작동 원리
  - `OAuth2PasswordBearer`: 클라이언트가 보낸 요청의 `Authorization: Bearer <TOKEN>` 헤더에서 자동으로 토큰을 추출
  - 유저 반환: 추출된 토큰을 `auth_service`로 전달해 유효성을 검사한 뒤, DB에서 해당 유저 객체를 찾아 반환


<details>
<summary> 상세 코드
</summary>

```python

# mysite4.dependencies.py

from fastapi import Depends
from fastapi.security import OAuth2PasswordBearer
from sqlalchemy.orm import Session
from database import get_db
from mysite4.services.auth_service import auth_service
from mysite4.models.user import User

# Authorization 헤더에서 Bearer 토큰을 자동으로 추출한다.
oauth2_scheme = OAuth2PasswordBearer(tokenUrl="/auth/login")


def get_current_user(
    token: str = Depends(oauth2_scheme),
    db: Session = Depends(get_db),
) -> User:
    return auth_service.get_current_user(db, token)

```

</details>


## 3. 주의사항 
- ⛔️ **DB 세션 중복 관리 유의**
  - get_current_user 함수 내에서 이미 get_db를 통해 세션이 주입되고 실행됨 
  - 따라서 이 함수를 의존성으로 사용하는 엔드포인트에서 추가적인 세션을 생성하거나 종료할 때, 기존 세션과의 생명주기 충돌이 발생하지 않도록 주의해야 함

- **단일 책임**: 인증 로직은 `auth_service`에, 의존성 주입은 `dependencies.py`에 분리하여 유지보수성을 높임

- **토큰 보안**: .env에 정의된 JWT_EXPIRE_MINUTES를 통해 토큰의 유효 기간 관리
