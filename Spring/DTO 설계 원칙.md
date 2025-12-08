# DTO 설계 원칙

프로젝트 개발 과정에서 논의된 **DTO 설계, Service 계층 패턴, JPA 최적화, 트러블슈팅, 그리고 API 테스트 가이드**를 총망라한 개발 문서.

---

## 1. DTO (Data Transfer Object) Convention

Java 17의 **`record`** 를 도입하여 불변 객체(Immutable)로 관리하며, Lombok 의존성을 줄이고 가독성을 높일 수 있음.

### 1.1 기본 원칙
1.  **Record 사용**: 모든 DTO는 `class` 대신 `record` 사용을 권장.
2.  **Entity 직접 노출 금지**: Response DTO는 Entity를 필드로 갖지 않으며, 필요한 값만 추출하여 반환.
3.  **변환 위치**: DTO ↔ Entity 변환은 **Service 계층(Transactional 범위)** 내에서 수행.

### 1.2 Request DTO (요청)
* **특징**: `toEntity()` 인스턴스 메서드를 구현하여 자기 자신(`this`)의 데이터로 Entity를 생성.
* **기본값 처리**: **Compact Constructor**를 사용하여 `null` 입력 시 안전한 기본값(`0`, `""`)을 설정.
* **검증**: `@NotNull`, `@NotBlank` 등 Validation 어노테이션 사용.

```java
public record AccommodationReviewCreateRequest(
    @NotNull(message = "숙소 ID는 필수입니다.")
    Long accommodationId,

    @NotNull
    BigDecimal ratingCleanliness,
    
    @NotNull
    BigDecimal ratingAccuracy,
    
    String comment
) {
    // ✅ Compact Constructor: 기본값(Default Value) 처리
    public AccommodationReviewCreateRequest {
        ratingCleanliness = (ratingCleanliness == null) ? BigDecimal.ZERO : ratingCleanliness;
        ratingAccuracy = (ratingAccuracy == null) ? BigDecimal.ZERO : ratingAccuracy;
        comment = (comment == null) ? "" : comment;
    }

    // ✅ toEntity: DTO(this) -> Entity 변환 (인스턴스 메서드)
    // 필요한 외부 엔티티(User, Accommodation)는 파라미터로 받는다.
    public AccommodationReview toEntity(Accommodation accommodation, User guest) {
        return AccommodationReview.of(
            accommodation,
            guest,
            this.calculateAverageRating(), // 내부 헬퍼 메서드 활용
            this.ratingCleanliness,
            this.ratingAccuracy,
            this.comment
        );
    }

    // 내부 헬퍼 메서드 (BigDecimal 연산)
    private BigDecimal calculateAverageRating() {
         return ratingCleanliness.add(ratingAccuracy)
                .divide(BigDecimal.valueOf(2), 1, RoundingMode.HALF_UP);
    }
}

```

### 1.3 Response DTO (응답)

- **특징**: `static from()` 팩토리 메서드를 구현하여 Entity를 재료로 받아 DTO를 생성한다.
- **패턴**: 필드 순서 실수를 방지하기 위해 `@Builder` 패턴 사용을 강력 권장한다.

Java

`@Builder
public record AccommodationReviewResponse(
    Long reviewId,
    String guestName,
    String guestProfileImage,
    BigDecimal ratingOverall,
    String comment,
    LocalDateTime createdAt
) {
    // ✅ static factory method: Entity -> DTO 변환
    public static AccommodationReviewResponse from(AccommodationReview review) {
        return AccommodationReviewResponse.builder()
                .reviewId(review.getId())
                .guestName(review.getGuest().getName()) // 연관관계 객체에서 필요한 값만 추출
                .guestProfileImage(review.getGuest().getProfileImage())
                .ratingOverall(review.getRatingOverall())
                .comment(review.getComment())
                .createdAt(review.getCreatedAt())
                .build();
    }
}`


---

## 2. JPA Repository & Query Optimization

### 2.1 메서드 명명 규칙 (Naming Convention)

- **연관관계 ID 조회**: `_` (언더스코어)를 사용하여 탐색 경로를 명확히 한다.
    - `findByAccommodation_Id(Long id)` (**권장**: Accommodation 객체의 ID 필드)
    - `findByAccommodationId` (비권장: AccommodationId라는 필드를 찾으려 시도함)
- **존재 여부**: `existsBy...` (단수형 `exist` 아님, 's' 필수).

### 3.2 성능 최적화 (N+1 문제 해결)

- **단건/소량 조회**: `@EntityGraph(attributePaths = {"member"})` 또는 `JOIN FETCH` 사용.
- **컬렉션(1:N) 조회**: `MultipleBagFetchException` 방지를 위해 `List` 2개 이상을 동시에 Fetch Join 하지 않는다.
    - **해결책**: `hibernate.default_batch_fetch_size: 100` (yml 설정)을 통해 `IN` 절로 묶어서 조회하도록 최적화.

### 3.3 부분 조회 (Projections)

- 전체 Entity가 필요 없을 때는 필요한 컬럼만 조회하여 성능을 높인다. (JPQL `select a.title from...`)

---


## Troubleshooting Log (트러블슈팅)

### 1 JWT 401 Unauthorized Issue

- **증상**: 토큰을 헤더에 넣었는데도 401 에러가 계속 발생.
- **원인 1**: 서버 재시작 시 Secret Key가 변경됨 (고정 키 사용 필요).
- **원인 2 (주요)**: 포스트맨에 토큰 입력 시 리다이렉트 URL 파라미터를 통째로 복사함.
    - 잘못된 예: `Bearer eyJ...access_token...&refreshToken=eyJ...` (뒤에 리프레시 토큰이 붙음)
    - **해결**: `&refreshToken=` 뒷부분을 모두 지우고 순수 Access Token만 입력해야 함.

### 2 Gradle Dependency Error

- **증상**: `NoClassDefFoundError: io/github/cdimascio/dotenv/Dotenv`
- **원인**: `spring-dotenv` 라이브러리의 내부 의존성인 `java-dotenv`가 로드되지 않음.
- **해결**: `build.gradle`에 명시적으로 추가.Groovy

  `implementation 'io.github.cdimascio:java-dotenv:5.2.2'`


### 3 Enum Mapping Error

- **증상**: `JSON parse error: Cannot deserialize value of type ... from String "APARTMENT"`
- **원인**: 요청한 문자열이 Enum 클래스에 정의된 상수(예: `ENTIRE_PLACE`, `PRIVATE_ROOM`)와 일치하지 않음.
- **해결**: API 문서를 확인하여 정확한 Enum 상수명(대문자)으로 요청 전송.

### 4 JPA Query Syntax

- **오류**: `existByTargetGuest...`
- **해결**: `existsByTargetGuest...` ('s' 포함)로 수정.

---