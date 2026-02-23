# 🗄️ 데이터 영속성(Persistence) 및 트랜잭션 설계


### 1. SQLAlchemy 주요 메서드 활용 전략

세션 메서드의 차이를 명확히 구분하여 데이터의 상태를 정밀하게 제어. 

| 메서드 | 역할 및 활용 (운영 관점) |
| --- | --- |
| **`db.add()`** | 객체를 세션 관리 대상으로 등록하여 **Persistent(영속)** 상태로 전환합니다. |
| **`db.flush()`** | DB에 쿼리를 전송하여 **ID를 확정**하되, 커밋은 유보합니다. 에러 발생 시 롤백 가능성을 확보하면서도, 연관 데이터 생성을 위해 임시 ID가 필요한 경우 활용합니다. |
| **`db.commit()`** | 트랜잭션을 확정하고 모든 변경 사항을 DB에 **영구 저장**합니다. |
| **`db.refresh()`** | DB의 최신 상태(자동 생성된 ID, 기본값 등)를 파이썬 객체에 다시 **동기화**합니다. |

---

### 2. 영속성 정의 및 Cascade 옵션

데이터 간의 복잡한 관계를 관리할 때 객체의 상태 변화가 어떻게 전이될지 정의합니다.

* **`save-update` (기본값)**: 부모 객체가 세션에 추가(`add`)되거나 업데이트될 때, 연결된 자식 객체들도 자동으로 세션에 포함되어 데이터 누락을 방지합니다.
* **M:N 관계 관리**: 사용자에게 입력받는 추가 컬럼이 없는 경우, `new_post.tags.append(tag)`와 같이 관계 설정을 통해 영속성을 관리합니다.
* **1:N 관계 관리**: `relationship`으로 연결된 자식 객체들을 리스트 형태로 안전하게 호출합니다.

---

### 3. M:N 관계와 `db.flush()` 활용

복잡한 연관 관계 생성 시 `db.flush()`를 호출하여 상위 객체의 ID를 즉시 확보하면, 트랜잭션을 유지한 상태에서 하위/중간 테이블과의 관계를 안전하게 정의 가능.

**예시: 게시글(Post) 생성 시 새로운 태그(Tag) 동시 등록**

```python
def create_post_with_tags(db: Session, post_data: PostCreate, tag_names: list[str]):
    with db.begin(): # 트랜잭션 시작 (원자성 보장)
        # 1. 게시글 객체 생성 및 세션 등록
        new_post = Post(title=post_data.title, content=post_data.content)
        db.add(new_post)
        
        # 2. flush로 new_post의 ID를 미리 확보 (Commit 전이지만 ID 사용 가능)
        db.flush() 
        
        # 3. 태그 처리 및 연결
        for name in tag_names:
            tag = db.query(Tag).filter(Tag.name == name).first()
            if not tag:
                tag = Tag(name=name)
                db.add(tag)
                db.flush() # 새 태그의 ID 확보
            
            # 4. 확보된 ID들을 바탕으로 M:N 관계 연결 (중간 테이블 데이터 준비)
            new_post.tags.append(tag)
            
    # with 블록 종료 시점에 한 번에 Commit 발생 (All-or-Nothing)
    db.refresh(new_post)
    return new_post

```

---

### 💡 장애 예측 기반 설계

운영 환경에서는 네트워크 지연이나 예기치 못한 로직 에러로 인해 트랜잭션이 중단될 수 있음.

1. **원자성 보장**: <br>`flush`를 활용하면 여러 단계의 작업을 하나의 트랜잭션으로 묶을 수 있다. 중간에 에러가 발생해도 전체가 롤백되므로 데이터 오염을 방지.
2. **순서 의존성 해결**: <br>중간 테이블은 양쪽 테이블의 ID가 모두 필요. `flush`는 트랜잭션을 종료하지 않고도 이 의존성을 깔끔하게 해결하여 로직의 가독성과 안전성을 동시에 높임.
