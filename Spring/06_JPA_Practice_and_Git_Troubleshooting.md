# 스프링 기본 기능 구현 (CRUD)

> **Date:** 2026-03-10 <br>
> **Topic:** Member CRUD 구현

---

## 1. 스프링 아키텍처와 DTO의 역할

* **계층형 구조**: `Controller(창구) - Service(관리) - Repository(창고) - Domain(실제 물건)`의 분리.
* **DTO(Data Transfer Object)**:
  * 엔티티(Entity)를 직접 외부로 노출하면 보안에 취약하고 순환 참조 문제가 생길 수 있음.
  * API 스펙에 맞춰 필요한 데이터만 담는 **전용 가방**인 DTO를 사용하여 데이터를 전달함.

* **생성자(Constructor)**: 객체가 생성될 때 초기값을 설정하는 규칙. 상황별로(가입용, 조회용 등) 다양한 생성자를 두어 객체 생성의 유연성을 확보함.

## 2. 순수 JPA (Pure JPA) 이해

* **Spring Data JPA를 배제한 이유**: 자동화 기능 이전에 `EntityManager`의 동작 원리를 이해하여 깊이 있는 백엔드 실력을 쌓기 위함.
* **EntityManager 주요 기능**:
  * `persist(member)`: 객체를 영속성 컨텍스트에 저장.
  * `find(Class, id)`: PK 기반 단건 조회.
  * `createQuery(JPQL, Class)`: 객체 지향 쿼리 언어를 사용한 복잡한 조회.


* **변경 감지(Dirty Checking)**:
  * `@Transactional` 환경에서 엔티티의 값을 수정하면, 트랜잭션이 끝날 때 JPA가 변경 사항을 감지하여 자동으로 `UPDATE` 쿼리를 실행함. `repository.save()`를 명시적으로 호출할 필요가 없음.



## 3. REST API 설계 관례

* **URL Mapping**: API 주소 앞에 `/api/`를 붙여 프론트엔드 정적 리소스와 백엔드 데이터를 명확히 구분함.
* **Optional 처리**: 서비스에서 반환된 `Optional<T>` 박스를 `orElseThrow()`를 이용해 안전하게 처리한 뒤 DTO로 변환함.
  * service에서 반환된 데이터를 controller에서 다시 체크할 필요 없음.


## 4. 테스트 코드(Test Code) 전략

* **Given-When-Then 패턴**: 테스트 케이스를 3단계로 나누어 가독성을 높임.
* **JUnit 5 & Gradle 설정**: 인텔리제이 설정에서 `Run tests using`을 `IntelliJ IDEA`로 변경하여 상세한 테스트 결과(함수명)를 확인 가능하게 함.
* **데이터 격리**: `@Transactional`을 테스트 클래스에 붙여 테스트 후 DB 데이터를 자동으로 롤백, 기존 데이터와의 충돌을 최소화함.
