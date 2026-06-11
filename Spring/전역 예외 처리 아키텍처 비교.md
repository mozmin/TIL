# Spring Boot 전역 예외 처리(Global Exception Handling) 아키텍처 비교
Spring Boot 프로젝트에서 `@RestControllerAdvice`를 활용하여 예외를 전역으로 처리할 때, 아키텍처 설계 방식은 크게 2가지로 나뉩니다. 각 방식의 구현 형태와 장단점(Trade-off)을 정리한다.
---
## 방법 1: 도메인/상황별 커스텀 예외 클래스 생성 방식
에러가 발생하는 상황(Case)마다 각각 독립적인 예외 클래스를 만들어 던지는(Throw) 전통적인 객체지향 방식.

### 구현 예시

```java
// 1. 에러 상황별로 예외 클래스를 무수히 생성
public class UserNotFoundException extends RuntimeException {
    public UserNotFoundException(Long id) {
        super("User with id " + id + " not found");
    }
}
public class WishlistAlreadyExistsException extends RuntimeException { ... }
// 2. 비즈니스 로직에서 발생
if (user == null) {
    throw new UserNotFoundException(userId);
}
// 3. 전역 예외 처리기에서 각각 매핑하여 처리
@RestControllerAdvice
public class GlobalExceptionHandler {
    @ExceptionHandler(UserNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleUserNotFound(UserNotFoundException e) {
        return ResponseEntity.status(HttpStatus.NOT_FOUND).body(...);
    }
    
    @ExceptionHandler(WishlistAlreadyExistsException.class)
    public ResponseEntity<ErrorResponse> handleWishlist(WishlistAlreadyExistsException e) { ... }
}
```

### 장점 (Pros)
**객체지향적이고 직관적:** 코드를 읽을 때 예외 클래스 이름만(UserNotFoundException) 보아도 어떤 도메인의 무슨 에러인지 100% 명확하게 파악할 수 있다.
**세밀한 복구(Recovery) 로직:** 특정 로직에서 try-catch(UserNotFoundException e) 형태로 개별 에러만 정밀하게 잡아내어 다른 로직으로 우회시키는 등의 처리가 쉽다.
상세한 정보 포함: 예외 클래스 내부 필드에 특정 값(예: 찾지 못한 엔티티의 ID)을 담아 로깅할 수 있다.

### 단점 (Cons)
**클래스 폭발 (Class Explosion):** 프로젝트가 커질수록 ~Exception 파일이 수십~수백 개로 무한 증식하여 패키지 관리가 매우 난잡해진다.
**반복되는 Handler 코드:** @ExceptionHandler 메서드도 예외 클래스 개수만큼 무한정 늘어난다.

---

## 방법 2: 공통 에러 코드(Enum) + 단일 예외 클래스 방식 (실무 트렌드)
프로젝트 내의 모든 에러를 하나의 ErrorCode Enum 파일에 문서처럼 모아두고, 예외를 던질 때는 CustomException 단일 클래스만 사용하는 방식.

### 구현 예시
```java
// 1. 시스템의 모든 에러를 한 곳에 모아둔 Enum
public enum ErrorCode {
    USER_NOT_FOUND(HttpStatus.NOT_FOUND, "해당 유저를 찾을 수 없습니다."),
    WISHLIST_ALREADY_EXISTS(HttpStatus.CONFLICT, "이미 존재하는 위시리스트입니다."),
    INVALID_INPUT_VALUE(HttpStatus.BAD_REQUEST, "잘못된 입력값입니다.");

    private final HttpStatus status;
    private final String message;
}

// 2. 프로젝트에서 공통으로 쓸 단 하나의 예외 클래스
public class CustomException extends RuntimeException {
    private final ErrorCode errorCode;
    public CustomException(ErrorCode errorCode) { this.errorCode = errorCode; }
}

// 3. 비즈니스 로직에서 발생 (Enum만 넘겨줌)
if (user == null) {
    throw new CustomException(ErrorCode.USER_NOT_FOUND);
}

// 4. 전역 예외 처리기 (모든 비즈니스 에러를 한 번에 처리)
@RestControllerAdvice
public class GlobalExceptionHandler {
    @ExceptionHandler(CustomException.class)
    public ResponseEntity<ErrorResponse> handleCustomException(CustomException e) {
        ErrorCode code = e.getErrorCode();
        return ResponseEntity.status(code.getStatus()).body(new ErrorResponse(code.getMessage()));
    }
}
```

### 장점 (Pros)
**에러 명세 중앙 집중화 (프론트엔드 친화적)**: ErrorCode.java 파일 하나가 곧 API 에러 명세서 역할을 합니다. 프론트엔드 개발자에게 "이 파일 보고 예외 처리하세요"라고 전달하기에 압도적으로 편리.
**구조의 깔끔함:** 불필요한 예외 클래스가 생기지 않고 패키지가 아주 깔끔하게 유지.
일관성: 여러 명의 백엔드 개발자가 협업해도 에러 응답 포맷과 코드가 통일.

### 단점 (Cons)
**동적 메시지 처리의 번거로움:** "1번 유저를 찾을 수 없습니다" 처럼 런타임에 동적인 값을 메시지에 넣으려면 Enum 설계와 예외 생성자가 다소 복잡해진다.
**에러 복구 로직 작성의 불편함:** try-catch(CustomException e)로 잡은 뒤, 내부에서 if (e.getErrorCode() == ErrorCode.USER_NOT_FOUND) 형태로 분기 처리를 해야 하므로 개별 복구 로직 작성이 약간 번거롭다.

## 결론 및 실무 추천
**최신 스타트업 및 IT 기업의 백엔드 아키텍처에서는 '방법 2(Enum 기반)'를 압도적으로 선호.** 최근 백엔드 개발은 마이크로서비스(MSA)와 프론트엔드 분리 구조가 기본이므로, 에러를 서버 내부에서 스스로 복구(try-catch)하기보다는 **"프론트엔드에게 얼마나 일관되고 친절한 규격(Contract)으로 에러를 던져주느냐"**가 훨씬 중요하기 때문.
