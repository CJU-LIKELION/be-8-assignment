# 🦁 8주차: JPA 기초 & 영속성 컨텍스트

7주차에서는 REST API의 CRUD 기능을 구현했습니다. 하지만 데이터가 메모리(ArrayList)에 저장되는 구조였기 때문에 서버를 재시작하면 모든 데이터가 사라지는 한계가 있었습니다. 이번 8주차에서는 이러한 문제를 해결하기 위해 JPA(Java Persistence API)를 도입하여 데이터를 MySQL 데이터베이스에 영구적으로 저장하는 구조로 확장합니다.

자세한 내용은 PBL 사이트를 참고 하세요. [8주차 - JPA 기초 & 영속성 컨텍스트](https://likelion-pbl-five.vercel.app/springboot/2f044860a4f481fb9dd3ea247d83d657)

## 🎯 과제 목표
* **@Entity 및 매핑 어노테이션 학습**: `@Entity`, `@Id`, `@GeneratedValue(strategy = IDENTITY)`의 필요성과 역할을 설명할 수 있다.
* **@Enumerated 활용**: 안전한 상수 관리를 위해 `EnumType.STRING`을 적용하는 이유를 설명할 수 있다.
* **JpaRepository 전환**: 인터페이스 상속으로 자동 제공되는 CRUD 메서드와 `findByName` 같은 쿼리 메서드 네이밍 규칙을 이해한다.
* **영속성 컨텍스트 파악**: 영속성 컨텍스트의 개념을 이해하고, `save()` 호출 시 엔티티에 `id`가 채워지는 이유를 설명할 수 있다.
* **환경 설정 및 SQL 추적**: `ddl-auto=create` 옵션을 이해하고, 콘솔에 출력되는 Hibernate SQL과 MySQL Workbench 조회를 통해 데이터를 검증할 수 있다.

## 🛠 기술 스택
* **Language**: Java (JDK 17 이상)
* **Framework**: Spring Boot 3.x
* **Database**: MySQL
* **Dependencies**: Spring Web, Spring Data JPA, MySQL Driver
* **Tool**: MySQL Workbench

## 📋 주요 기능
1. **단일 테이블 및 Enum 기반 설계**: 7주차의 상속 구조를 버리고 `Member` 단일 엔티티와 `RoleType` Enum으로 간소화하여 RDB 매핑 최적화
2. **기본키(ID) 기반 자원 식별**: 단건 조회, 수정, 삭제의 식별자를 기존 `{name}`에서 DB 기본키인 `{id}`(Auto Increment) 기반으로 변경
3. **영속성 기반 CRUD 및 응답 통합**: `JpaRepository` 기반으로 전환하고, 응답 데이터 구조 다형성을 하나의 `MemberResponse` DTO로 통합

## 🚀 실행 흐름 (Implementation Logic)
본 미션은 메모리 기반의 저장소와 순수 자바 객체 구조를 걷어내고, JPA 영속성 계층을 통해 MySQL 데이터베이스와 연동을 완성합니다.

| 단계 | 주요 작업 내용 |
| :--- | :--- |
| **Step 1** | `build.gradle`에 JPA 및 MySQL 의존성을 추가하고, `application.properties`에 `ddl-auto=create`, `show-sql` 설정을 구성 및 불필요한 기존 메모리 파일 삭제 |
| **Step 2** | `LION`과 `STAFF`를 구분하는 `RoleType` Enum을 생성하고, 테이블과 매핑될 `Member` 엔티티 클래스(기본 생성자 필수) 정의 |
| **Step 3** | `MemberRepository`가 `JpaRepository<Member, Long>`을 상속하도록 변경하고 이름 존재 여부 검증을 위한 쿼리 메서드(`existsByName`) 추가 |
| **Step 4** | `MemberService`를 엔티티 및 `id` 기반으로 수정하고, `save()` 호출 후 영속화되어 `id`가 채워진 엔티티 비즈니스 로직 적용 |
| **Step 5** | 요청 DTO 4종의 분리는 유지하되 응답 DTO를 `MemberResponse` 하나로 통합하고, `MemberController`의 엔드포인트를 `{id}` 기반으로 전면 리팩토링 |

### 🔍 API Endpoints 명세 표준화

| 기능 | HTTP Method | URI 경로 | Request Body | HTTP Status (성공 / 실패) |
| :--- | :--- | :--- | :--- | :--- |
| **Lion 등록** | `POST` | `/members/lions` | `LionCreateRequest` | `201 Created` / `409 Conflict` |
| **Staff 등록** | `POST` | `/members/staffs` | `StaffCreateRequest` | `201 Created` / `409 Conflict` |
| **전체 멤버 조회** | `GET` | `/members` | *None* | `200 OK` |
| **ID로 단일 조회** | `GET` | `/members/{id}` | *None* | `200 OK` / `404 Not Found` |
| **Lion 정보 수정** | `PUT` | `/members/lions/{id}` | `LionUpdateRequest` | `200 OK` / `404 Not Found` |
| **Staff 정보 수정** | `PUT` | `/members/staffs/{id}` | `StaffUpdateRequest` | `200 OK` / `404 Not Found` |
| **멤버 데이터 삭제** | `DELETE` | `/members/{id}` | *None* | `204 No Content` / `404 Not Found` |

## ⚠️ 제약 사항 및 유의 사항
- [ ] **엔티티 무인자 생성자 확보**: JPA의 프록시 및 리플렉션 동작을 위해 엔티티 클래스는 반드시 `protected Member() {}` 스펙의 기본 생성자를 포함해야 합니다.
- [ ] **EnumType.STRING 필수 적용**: 역할 구분을 위한 `@Enumerated` 어노테이션 속성에는 순서 변경 문제를 방지하기 위해 반드시 `STRING` 옵션을 명시합니다.
- [ ] **요청/응답 DTO 분리 정책**: 명확한 API 스펙 제한을 위해 클라이언트가 전송하는 `Request DTO` 4종은 기존 분리 상태를 유지하고, 서버가 내보내는 `Response DTO`만 `MemberResponse`로 통합합니다.
- [ ] **식별자 패러다임 수정**: 7주차의 이름(`name`) 기반 자원 탐색 경로를 중단하고, 데이터베이스 기본키 필드인 **`Long id`**를 `@PathVariable`로 바인딩하여 제어해야 합니다.
- [ ] **하이버네이트 및 DB 검증**: 애플리케이션 실행 시 테이블 생성 DDL 로그를 확인하고, API 호출 후 MySQL Workbench에서 `SELECT * FROM member;` 쿼리를 통해 영속성 적재 결과를 직접 검증해야 합니다.

---
*이 프로젝트는 [멋쟁이사자처럼(LIKELION) PBL](https://likelion-pbl-five.vercel.app/) 과정을 바탕으로 제작되었습니다.*
