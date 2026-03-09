# 스프링 DB 접근 기술과 통합 테스트 활용

> **Date:** 2026-03-09 <br>
> **Topic:** H2, Jdbc, **JPA**, 스프링 데이터 JPA
---

## 1. H2 데이터베이스 및 환경 설정

* **H2 DB**: 개발 및 테스트용으로 최적화된 가벼운 DB.

* **연결 방식**: 최초 접속 시 `jdbc:h2:~/test`로 파일을 생성하고, 이후에는 `jdbc:h2:tcp://localhost/~/test`를 통해 네트워크 모드로 접속.

* **주의사항**: `application.properties` 설정 시 `spring.datasource.username=sa`를 반드시 추가해야 하며, 오타(예: `spirng`)가 나지 않도록 주의.



## 2. 순수 JDBC에서 JdbcTemplate으로의 발전

* **순수 JDBC**: `Connection`, `PreparedStatement` 등을 직접 관리하며 반복 코드가 매우 많았던 방식.


* **JdbcTemplate**: JDBC API의 반복 코드를 대부분 제거해주지만, SQL은 직접 작성해야 함.


* **RowMapper**: 데이터베이스의 조회 결과(`ResultSet`)를 자바 객체(`Member`)로 변환해주는 번역기 역할.



## 3. 스프링 통합 테스트 (Integration Test)

* **@SpringBootTest**: 스프링 컨테이너와 테스트를 함께 실행하여 실제 의존관계가 주입된 상태로 테스트.


* **@Transactional**: 테스트 시작 전 트랜잭션을 시작하고 종료 후 **롤백(Rollback)**하여 DB에 데이터를 남기지 않음. => 다음 테스트에 영향을 주지 않고 반복 테스트 가능.


* **단위 테스트 vs 통합 테스트**: 가급적 순수 자바 코드로 테스트하는 '단위 테스트'가 훨씬 빠르고 좋은 테스트일 확률이 높음.



## 4. JPA와 스프링 데이터 JPA

* **JPA**: 반복 코드는 물론 기본 SQL까지 직접 만들어 실행해주며, 객체 중심의 설계로 패러다임을 전환.


* **스프링 데이터 JPA**: 인터페이스만으로 개발을 완료할 수 있게 도와주는 프레임워크. `findByName()`처럼 메서드 이름만으로도 조회 기능을 자동으로 제공.(* 단, JPA 이해가 선행되어야 함.)


