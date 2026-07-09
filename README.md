# 🦁 7주차: REST API 설계 - 멤버 관리 CRUD (REST 아키텍처 원칙과 DTO)

6주차에서 다룬 스프링 컨테이너 전환에 이어, 본격적으로 웹 API 설계 표준인 **REST API 원칙**에 맞춰 멤버 관리 시스템의 CRUD(생성, 조회, 수정, 삭제) 엔진을 설계하는 과제입니다.

자세한 내용은 PBL 사이트를 참고 하세요. [7주차 - REST API 설계 (CRUD)](https://likelion-pbl-five.vercel.app/springboot/2f044860a4f481fb9dd3ea247d83d657)

## 🎯 과제 목표
* **REST 원칙 숙달**: URI로 자원(Resource)을 표현하고 HTTP 메서드로 행위를 표현하는 REST API 설계 규칙을 이해합니다.
* **DTO 도입**: 데이터 전송 객체(DTO)의 개념과 필요성을 이해하고, 클라이언트 데이터와 도메인 엔티티를 명확히 분리합니다.
* **HTTP 상태 코드 제어**: `ResponseEntity`를 활용하여 요청 결과에 따른 정확한 상태 코드(200, 201, 204, 404, 409)를 제어합니다.
* **어노테이션 활용**: 데이터 전달 방식에 따라 `@PathVariable`과 `@RequestBody`를 적절하게 바인딩합니다.
* **역할별 설계 확장**: 고유 필드가 다른 `Lion`과 `Staff` 객체의 특성을 고려하여 API 및 DTO를 분리 설계합니다.

## 🛠 기술 스택
* **Language**: Java (JDK 17 이상)
* **Framework**: Spring Boot 3.x
* **Build Tool**: Gradle
* **Dependencies**: Spring Web
* **Key Concepts**: REST API, CRUD, HTTP Methods & Status Code, DTO, ResponseEntity

## 📋 주요 기능
1. **역할별 DTO 기반 CRUD**: 아기사자(`Lion`)와 운영진(`Staff`)의 필드 차이를 반영한 맞춤형 데이터 전송 처리
2. **식별자 기반 자원 관리**: 고유한 이름(`name`) 값을 자원의 식별자(`PathVariable`)로 사용하여 조회, 수정, 삭제 수행
3. **표준 HTTP 응답 규격**: 처리 성공 및 비즈니스 예외(중복, 비존재) 상황을 HTTP 명세에 맞춰 브라우저에 반환

## 🚀 실행 흐름 (Implementation Logic)
본 미션은 설계 표준에 맞춰 API 명세를 설계하고 계층(Controller-Service-Repository) 간 데이터를 순차적으로 완성합니다.

| 단계 | 주요 작업 내용 |
| :--- | :--- |
| **Step 1** | 요청(`Request`)과 응답(`Response`) 목적 및 역할별로 분리된 **DTO 클래스** 설계 및 팩토리 메서드 구현 |
| **Step 2** | `MemberRepository`와 `MemoryMemberRepository`에 데이터 수정(`update`) 및 삭제(`delete`) 메서드 추가 |
| **Step 3** | `MemberService`에 DTO를 엔티티로 변환하고 중복/예외 검증을 처리하는 비즈니스 CRUD 로직 고도화 |
| **Step 4** | `MemberController`에서 각 자원별 엔드포인트를 열고 `ResponseEntity`로 유연한 HTTP 상태 코드 응답 완성 |

### 🔍 API Endpoints 명세 표준화

| 기능 | HTTP Method | URI 경로 | Request Body | HTTP Status (성공 / 실패) |
| :--- | :--- | :--- | :--- | :--- |
| **Lion 등록** | `POST` | `/members/lions` | `LionCreateRequest` | `201 Created` / `409 Conflict` |
| **Staff 등록** | `POST` | `/members/staffs` | `StaffCreateRequest` | `201 Created` / `409 Conflict` |
| **단일 멤버 조회** | `GET` | `/members/{name}` | *None* | `200 OK` / `404 Not Found` |
| **Lion 정보 수정** | `PUT` | `/members/lions/{name}` | `LionUpdateRequest` | `200 OK` / `404 Not Found` |
| **Staff 정보 수정** | `PUT` | `/members/staffs/{name}` | `StaffUpdateRequest` | `200 OK` / `404 Not Found` |
| **멤버 데이터 삭제** | `DELETE` | `/members/{name}` | *None* | `204 No Content` / `404 Not Found` |

## ⚠️ 제약 사항 및 유의 사항
- [ ] **REST URI 규칙 준수**: URI 경로 설계 시 명사형 태그만 사용하며, 행위를 뜻하는 동사 표현(예: `/createLion`, `/delete`)은 엄격히 금지합니다.
- [ ] **도메인 캡슐화와 DTO 분리**: 컨트롤러 계층이 도메인 객체(`Lion`, `Staff`)를 직접 반환하거나 요청받지 않도록 정의된 DTO 엔티티를 거쳐야 합니다.
- [ ] **메모리 저장소 동기화**: 다음 주차의 JPA 연동 전까지 데이터 처리는 기존의 컬렉션 기반 메모리 저장소(`MemoryMemberRepository`) 내부에서 안전하게 갱신 및 삭제(`removeIf`)되도록 구현합니다.
- [ ] **패키지 아키텍처 구조**: 제시된 패키지 구조(`controller`, `service`, `repository`, `domain`, `dto`)의 분리 정책을 철저하게 준수합니다.
- [ ] **이름 기반 식별 및 중복 통제**: 시스템 내 모든 멤버는 숫자 ID 대신 고유한 **이름(name)**으로 식별하며, 이름이 중복되어 등록되는 상황은 비즈니스 차단 후 `409 Conflict` 응답을 주어야 합니다.

---
*이 프로젝트는 [멋쟁이사자처럼(LIKELION) PBL](https://likelion-pbl-five.vercel.app/) 과정을 바탕으로 제작되었습니다.*
