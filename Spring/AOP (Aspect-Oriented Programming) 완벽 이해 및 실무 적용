# AOP (Aspect-Oriented Programming) 완벽 이해 및 실무 적용

## 1. AOP란 무엇인가?
**AOP(관점 지향 프로그래밍)**는 애플리케이션의 핵심 비즈니스 로직과 공통 로직을 분리하여 모듈화하는 프로그래밍 기법이다.

AOP를 사용하면 기존 비즈니스 로직을 전혀 수정하지 않고도, 외부에서 Proxy 객체를 통해 부가 기능을 앞뒤로 동적으로 끼워 넣을 수 있다.

### 핵심 용어 정리
Aspect (관점): 공통으로 적용될 부가 기능의 모음 (예: 로깅 클래스)

Advice (조언): 부가 기능을 언제 실행할지 정의 (Before, After, Around 등)

Pointcut (지점): 부가 기능을 어느 클래스/메서드에 적용할지 필터링하는 규칙

JoinPoint (결합점): Advice가 적용될 수 있는 실행 지점 (보통 메서드 실행 시점)

## 2. AOP 구현 예시 코드 [실행 시간 측정]
핵심 비즈니스 로직인 LogisticsService를 건드리지 않고, @Aspect를 활용해 메서드 실행 시간을 측정하는 로직을 분리해 낸다.

### 1) 핵심 로직 (Target)
```Java
import org.springframework.stereotype.Service;

@Service
public class LogisticsService {
    
    // 핵심 비즈니스 로직 (AOP 관련 코드는 단 한 줄도 없다!)
    public void processDelivery() {
        System.out.println("📦 주문된 물건을 포장하고 출고를 준비합니다.");
        try {
            Thread.sleep(1000); // 비즈니스 로직 처리에 1초 소요 가정
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

### 2) AOP 공통 부가 기능 (Aspect)
```Java
import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.springframework.stereotype.Component;

@Aspect      // 이 클래스가 AOP 관점(부가 기능)임을 선언
@Component   // 스프링 빈으로 등록
public class PerformanceAspect {

    // Pointcut: LogisticsService 내의 모든 메서드 실행 시
    // Advice: 대상 메서드 실행 전후(Around)로 동작
    @Around("execution(* com.example..LogisticsService.*(..))")
    public Object measurePerformance(ProceedingJoinPoint joinPoint) throws Throwable {
        
        // --- Before: 원래 메서드 실행 전 ---
        long start = System.currentTimeMillis();
        System.out.println("⏱️ [AOP] 바코드 스캔 및 작업 시작: " + joinPoint.getSignature().getName());

        // --- 핵심 비즈니스 로직 실행 ---
        Object result = joinPoint.proceed(); 

        // --- After: 원래 메서드 실행 후 ---
        long end = System.currentTimeMillis();
        System.out.println("⏱️ [AOP] 작업 완료! 소요 시간: " + (end - start) + "ms");

        return result;
    }
}
```

## 3. 실무 트러블슈팅: 내부 호출 문제
초보 백엔드 개발자가 실무에서 AOP를 사용할 때 가장 많이 부딪히는 문제가 바로 **"어? 메서드에 AOP를 걸었는데 왜 작동을 안 하지?"**이다.

이 문제는 스프링 AOP가 프록시(Proxy, 대리인) 기반으로 동작하기 때문에 발생한다.

### 문제 상황 (AOP가 무시되는 경우)
```Java
@Service
public class LogisticsService {

    // 외부에서 이 메서드를 호출함
    public void startWork() {
        System.out.println("작업을 지시합니다.");
        processDelivery(); // 같은 클래스 내부의 다른 메서드를 직접 호출
    }

    // AOP(시간 측정)를 걸어둔 메서드
    public void processDelivery() {
        System.out.println("포장 작업을 시작합니다.");
    }
}
```
외부 컨트롤러에서 startWork()를 호출하면, 그 안에서 processDelivery()를 호출하더라도 AOP(시간 측정 로직)가 작동하지 않는다.

### 원인 파악
스프링은 AOP를 적용할 때 진짜 객체 대신 **Proxy**을 앞에 세워둔다.
외부에서 접근할 때는 대리인을 거치기 때문에 부가 기능이 정상 작동하지만, 이미 진짜 객체 내부로 들어온 상태(startWork())에서 자기 자신의 다른 메서드(processDelivery())를 호출할 때는 대리인을 거치지 않고 직접 호출(this.processDelivery())해버리기 때문이다.

### 해결 방법
1. 클래스 분리 (가장 권장): 내부 호출이 일어나지 않도록, processDelivery()를 담당하는 별도의 Service 클래스를 새로 만들어 분리한다. (객체 지향적 관점에서도 책임이 분리되어 더 좋다)

2. 자기 자신을 주입받기 (Self-Injection): (스프링 부트 2.6 이상부터는 순환 참조 에러가 발생할 수 있어 권장하지 않음)

3. AopContext 사용: AopContext.currentProxy()를 사용해 억지로 프록시를 통해 호출하게 만들 수 있지만, 코드가 스프링 프레임워크에 강하게 종속되므로 피하는 것이 좋다.

---

마치며
AOP는 단순히 코드를 줄이는 것을 넘어, 개발자가 '비즈니스 핵심 가치'에만 온전히 집중할 수 있게 만들어주는 훌륭한 패턴이다. AOP와 프록시의 동작 원리를 이해하면 스프링의 트랜잭션(@Transactional)이 왜 내부 호출 시 작동안하는지도 자연스럽게 깨달을 수 있다.
