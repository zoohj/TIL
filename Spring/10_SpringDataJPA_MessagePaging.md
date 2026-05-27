# Spring Data JPA 도입 및 메시지 페이징 API 구현

> **Topic:** Spring Data JPA, 메시지 페이징 API


## 1. 기술적 의사결정 (JPA vs 직접 구현)
* **고민:** JDBC/EntityManager를 사용하여 직접 쿼리를 구현할 것인가, JPA를 사용할 것인가?
* **결정:** 생산성과 유지보수 효율성을 위해 **Spring Data JPA** 도입. 
* **이유:** 비즈니스 로직(채팅 서비스)에 집중하기 위해 단순 CRUD 및 페이징 처리는 프레임워크에 위임하여 개발 속도를 최적화함.

## 2. 주요 트러블슈팅 
### ① NullPointerException (NPE)과 방어 코드
* **문제:** DB에 `sender_id`가 null인 데이터가 있을 때 `getName()` 호출 시 서버 에러(500) 발생.
* **해결:** Java의 삼항 연산자를 사용하여 `sender` 객체의 null 여부를 체크하는 방어 로직 추가.
* **배경:** 연관 관계가 있는 객체 조회 시, 데이터의 유실(회원 탈퇴 등) 가능성을 항상 염두에 두어야 함.

### ② 라이브러리 임포트(Import) 충돌
* **문제:** `java.awt.print.Pageable`을 임포트하여 컨트롤러와 서비스 간 타입 불일치 및 실행 에러 발생.
* **해결:** 스프링 전용 도구인 `org.springframework.data.domain.Pageable`로 교체.
* **교훈:** 자바는 이름이 같아도 패키지가 다르면 전혀 다른 객체이므로, 자동 완성 시 패키지 경로를 반드시 확인해야 함.

### ③ JPA PropertyReferenceException
* **문제:** Swagger 테스트 시 `sort` 파라미터에 기본값 `"string"`이 들어가 엔티티 필드 매핑 에러 발생.
* **해결:** 정렬 기준을 실제 필드명인 `createdAt`으로 명시하거나 공백 처리하여 해결.

## 3. 핵심 아키텍처 및 문법
### 빌더 패턴 (Builder Pattern)
* 생성자의 파라미터가 많아질 때 발생할 수 있는 순서 혼동 문제를 해결하고, 가독성을 높이기 위해 `@Builder` 사용. 
* DTO 설계 시 `@NoArgsConstructor`, `@AllArgsConstructor`를 함께 사용하여 JPA와 Jackson 라이브러리와의 호환성 확보.

### 스트림 API와 Slice (Paging)
* **Stream & Map:** 컨베이어 벨트 공정처럼 엔티티 리스트를 DTO 리스트로 효율적으로 변환.
* **Slice vs Page:** 전체 카운트 쿼리가 필요한 `Page` 대신, 무한 스크롤에 최적화되어 "다음 페이지 존재 여부"만 체크하는 `Slice`를 사용하여 성능 최적화.

---

## 💡 정리
```
1. @PageableDefault의 활용

클라이언트(프론트)에서 페이징 정보를 보내지 않아도 서버가 터지지 않게 **기본값(size=20, 최신순 정렬)**을 설정하여 시스템 안정성을 확보함.

2. ResponseEntity를 통한 표준 응답

단순 데이터를 넘기는 것을 넘어, HTTP 상태 코드(200 OK)와 함께 페이징 메타데이터(현재 페이지, 다음 페이지 여부 등)를 캡슐화하여 전달함으로써 RESTful한 설계를 실천함.

3. 경로(Path) 설계의 분리

기존 /messages와 페이징용 /messages/page를 분리하여 기존 기능의 하위 호환성을 유지하면서 새로운 기능을 점진적으로 도입함. (나중에는 하나로 통합하는 리팩터링 과제로 남겨둘 수 있음!)```