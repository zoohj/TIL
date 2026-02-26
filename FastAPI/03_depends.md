
# FastAPI 의존성 주입과 트랜잭션 최적화


## 1. FastAPI DI

일반적인 파이썬 객체 생성은 내가 직접 `repo = Repository(db)`라고 명령해야 하지만, <br> FastAPI의 DI는 "**나 이거 필요해**"라고 인자에 적어두기만 하면 FastAPI가 알아서 객체를 조립.

### 운영 관점의 이점

* **객체 생명주기 관리**<br>DB 세션처럼 "열고 닫는" 과정이 중요한 객체를 개발자가 실수로 닫지 않는 사고를 방지.
* **단일 책임 원칙**<br>서비스 로직은 "어떻게 예약하는가"에만 집중하고, "어떻게 DB에 연결하는가"는 프레임워크가 책임.



## 2. 문제 상황과 구조적 원인 분석

기존 코드에서 리포지토리 메서드를 클래스 이름으로 직접 호출(`TimeslotRepository.find_by_id`)하거나, `__init__` 생성을 수동으로 관리하면서 다음과 같은 에러가 발생. 

* **`AttributeError`**: 인스턴스가 아닌 클래스를 호출하여 `self.db` 참조 불가.
* **`InvalidRequestError`**: `db.begin()` 중복 호출로 인한 트랜잭션 충돌.
* **`TypeError`**: 의존성 주입 미숙으로 인한 인자 누락.
* **결합도 상승**: 객체 간의 의존 관계가 복잡해져 테스트 코드 작성이 어려워짐.


---

## 3. FastAPI `Depends`를 활용한 의존성 주입(DI)

**Router → Service → Repository**로 이어지는 3계층 구조에서 DI는 체인처럼 연결.


* **Service 계층**: 메서드 인자에서 직접 리포지토리를 주입받아 결합도를 낮춤.
* **Router 계층**: 서비스 객체를 주입받아 비즈니스 로직과 통신 기능을 명확히 분리.

...

1. **Router**: `service: ReservationService = Depends()`라고 선언.
2. **FastAPI의 분석**: "음, `ReservationService`가 필요하네. 그런데 이 서비스는 `ts_repo`와 `room_repo`가 있어야 하네?"라고 판단.
3. **재귀적 주입**
   * 먼저 `db 세션`을 생성.
   * 그 `db`를 넣어 `Repository`들을 생성.
   * 그 `Repository`들을 넣어 `Service`를 생성.
   * 최종적으로 완성된 `Service`를 **Router**에 전달.

...

**`__init__`을 수동으로 관리하는 대신, FastAPI의 `Depends`를 사용하여 객체 생명주기를 프레임워크에 위임.<br> 가장 말단에 있는 `service.create_reservation()`만 실행.**






---

## 4. `Depends` vs `__init__`: 무엇이 다른가?

| 비교 항목 | 직접 객체 생성 (`__init__`) | FastAPI DI (`Depends`) |
| --- | --- | --- |
| **제어권** | 개발자가 직접 통제 (Manual) | 프레임워크가 통제 (Inversion of Control) |
| **에러 위험** | 인자 누락, 순서 바뀜 등 인적 오류 높음 | 규격만 맞으면 에러 발생 가능성 낮음 |
| **유연성** | 테스트할 때 DB 연결을 끊기 어려움 | 테스트용 가짜 DB(Mock)로 갈아끼우기 매우 쉬움 |

---

## 5. `Depends()` 안에 함수를 주입

단순 클래스 주입 외에도, 특정 로직을 거친 결과를 주입받을 수 있다.

```python
current_user: User = Depends(get_current_user)
```
* 단순히 `User` 객체를 주는 게 아니라, "**토큰을 검증하고 DB에서 유저를 찾아오는 복잡한 과정**"을 거친 결과물만 전달해줌.



### `lambda`를 활용한 싱글톤 주입

이미 만들어진 전역 객체(`reservation_service`)를 주입하고 싶을 때 사용.

```python
  service: ReservationService = Depends(lambda: reservation_service)
```




## 💡 인사이트

객체의 생성 권한을 프레임워크에 넘김으로써(DI) 인적 오류를 줄이고, 

**트랜잭션의 단계를 세분화**`(Flush vs Commit)`하여 데이터 유실 가능성을 최소화. 

시스템의 **신뢰성**(Reliability)으로 직결.
