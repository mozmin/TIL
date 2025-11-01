# Comparator vs Comparable

자바에서 객체를 정렬하는 두 가지 핵심 인터페이스

## 1. `Comparable` vs `Comparator`: 핵심 개념 비교

| 구분 | `Comparable` (비교 가능한) | `Comparator` (비교자, 외부 심판)                                                         |
| --- | --- |-----------------------------------------------------------------------------------|
| **핵심 아이디어** | **"내가 내 순서를 정한다"** (클래스 내장 방식) | **"외부 심판이 순서를 정해준다"** (외부 주입 방식)                                                  |
| **인터페이스** | `Comparable<T>` | `Comparator<T>`                                                                   |
| **핵심 메소드** | `int compareTo(T other)`(자기 자신과 `other` 비교) | `int compare(T o1, T o2)`(`o1`과 `o2` 비교)                                          |
| **구현 위치** | 정렬할 클래스 **내부**에 `implements` | 클래스 **외부**에 별도의 클래스나 람다식으로 구현                                                     |
| **언제 사용하는가?** | 해당 객체의 **기본/자연스러운 정렬 순서**를 정의할 때.(예: `String`은 사전순, `Integer`는 숫자 크기순) | - **다양한 정렬 기준**이 필요할 때 - **원본 클래스 수정이 불가능**할 때 - **복합 정렬**이 필요할 때 (A순, 같으면 B순...) |
| **장점** | `sort` 메소드 호출이 간결함 | 유연성, 확장성이 뛰어나고 원본 클래스를 건드리지 않음                                                    |
| **코딩테스트 추천** | - | **(강력 추천)** 람다식과 함께 사용하여 빠르고 유연하게 구현                                              |

## 2. 배열 타입별 정렬 방법

### A. 기본 타입 배열 (Primitive Type Array)

`int[]`, `double[]`, `char[]` 등 기본 타입 배열은 `Arrays.sort()`를 사용해 간단하게 **오름차순**으로 정렬할 수 있습니다.

```java
import java.util.Arrays;
import java.util.Collections;

// int[] 배열
int[] nums = {3, 1, 4, 1, 5, 9, 2};
Arrays.sort(nums);
// 결과: [1, 1, 2, 3, 4, 5, 9]

// char[] 배열
char[] chars = {'z', 'a', 'b', 'c', 'a'};
Arrays.sort(chars);
// 결과: ['a', 'a', 'b', 'c', 'z']

// 내림차순 정렬이 필요할 경우?
// -> 기본 타입은 직접 내림차순이 안되므로 Wrapper 클래스 배열로 변환해야 합니다.
Integer[] numsWrapper = {3, 1, 4, 1, 5, 9, 2};
Arrays.sort(numsWrapper, Collections.reverseOrder());
// 결과: [9, 5, 4, 3, 2, 1, 1]`
```

### B. `String` 배열 (Comparable이 이미 구현된 객체 배열)

`String` 클래스는 이미 `Comparable` 인터페이스를 구현하고 있어 사전순(오름차순)으로 기본 정렬.

```Java
import java.util.Arrays;
import java.util.Comparator;

String[] fruits = {"Banana", "Apple", "Cherry"};

// 1. 기본 정렬 (오름차순)
Arrays.sort(fruits);
// 결과: ["Apple", "Banana", "Cherry"]

// 2. 내림차순 정렬 (Comparator 활용)
Arrays.sort(fruits, (s1, s2) -> s2.compareTo(s1));
// 또는
Arrays.sort(fruits, Comparator.reverseOrder());
// 결과: ["Cherry", "Banana", "Apple"]

// 3. 글자 길이순 정렬 (Comparator + 람다)
Arrays.sort(fruits, (s1, s2) -> s1.length() - s2.length());
// 결과: ["Apple", "Cherry", "Banana"]`
```

### C. 2차원 `int[][]` 배열 (코딩테스트 단골 유형)

`Comparator`와 람다식을 활용하여 다양한 기준으로 정렬할 수 있습니다.

```Java
import java.util.Arrays;
import java.util.Comparator;

int[][] points = {{3, 4}, {1, 2}, {3, 1}, {2, 4}};

// 1. 첫 번째 원소(x좌표) 기준 오름차순
Arrays.sort(points, (a, b) -> a[0] - b[0]);
// 결과: [[1, 2], [2, 4], [3, 4], [3, 1]]

// 2. 복합 정렬: 첫 번째 원소 오름차순, 같으면 두 번째 원소(y좌표) 내림차순
Arrays.sort(points, (a, b) -> {
if (a[0] == b[0]) {
return b[1] - a[1]; // 두 번째 원소는 내림차순
}
return a[0] - b[0]; // 첫 번째 원소는 오름차순
});
// 결과: [[1, 2], [2, 4], [3, 4], [3, 1]]

// 3. 복합 정렬 (더 세련된 방식: Comparator.comparingInt)
Arrays.sort(points, Comparator.comparingInt((int[] p) -> p[0])
.thenComparing(Comparator.comparingInt((int[] p) -> p[1]).reversed()));
// 결과: [[1, 2], [2, 4], [3, 4], [3, 1]]`
```

### D. 커스텀 객체 배열

우리가 직접 만든 클래스(예: `Student`, `Node` 등)의 객체 배열을 정렬하는 방법입니다.

### 예시 `Student` 클래스

```Java
class Student {
String name;
int score;
public Student(String name, int score) { /* 생성자 */ }
// getter, toString 등...
}

Student[] students = {
new Student("모코코", 90),
new Student("지미니", 100),
new Student("자바", 90)
};
```

### 방법 1: `Comparable` 구현 (기본 정렬 기준 정의)

`Student` 클래스 자체에 기본 정렬 기준을 내장합니다. (예: 점수 내림차순)

```Java
`class Student implements Comparable<Student> {
// ... 필드, 생성자 ...

    @Override
    public int compareTo(Student other) {
        // 점수 기준 내림차순
        return other.score - this.score;
    }
}

// 사용법
Arrays.sort(students);
// 결과: [지미니(100), 모코코(90), 자바(90)] (기본 정렬 기준에 따라)`
```
### 방법 2: `Comparator` 활용 (다양한 정렬 기준 적용) - **(가장 추천)**

필요한 곳에서 `Comparator`를 람다식으로 구현하여 정렬합니다.

```Java

`// 1. 점수 기준 오름차순
Arrays.sort(students, (s1, s2) -> s1.score - s2.score);
// 결과: [모코코(90), 자바(90), 지미니(100)]

// 2. 이름 기준 오름차순
Arrays.sort(students, (s1, s2) -> s1.name.compareTo(s2.name));
// 결과: [모코코(90), 자바(90), 지미니(100)] (자바, 지미니, 모코코 순)

// 3. 복합 정렬: 점수 내림차순, 같으면 이름 오름차순
Arrays.sort(students, (s1, s2) -> {
if (s1.score == s2.score) {
return s1.name.compareTo(s2.name); // 이름 오름차순
}
return s2.score - s1.score; // 점수 내림차순
});
// 결과: [지미니(100), 자바(90), 모코코(90)] (점수 같을 때 이름순 정렬)`
```

## Q1. Arrays.sort(points, (a, b) -> a[0] - b[0]); 이거는 뭐가 생략된걸까?

(a, b) -> a[0] - b[0] 람다식은 Comparator 인터페이스를 구현한 **익명 클래스**의 축약형

**람다식으로 변환되는 과정**

**1단계: `new Comparator<int[]>() { ... }` 생략**`Arrays.sort` 메소드는 두 번째 인자로 `Comparator`가 필요하다는 것을 이미 알고 있습니다. 따라서 컴파일러는 우리가 `Comparator`를 구현하려 한다는 것을 문맥상 추론할 수 있으므로, 이 boilerplate 코드를 생략할 수 있습니다.

**남은 코드:**

```Java

(int[] a, int[] b) -> {
    return a[0] - b[0];
}
```
**2단계: 파라미터 타입 `int[]` 생략**
컴파일러는 `points` 배열이 `int[][]` 타입이라는 것을 알고 있습니다. 따라서 배열의 각 요소인 `a`와 `b`가 `int[]` 타입이라는 것도 **타입 추론(Type Inference)**을 통해 알 수 있습니다. 그래서 파라미터의 타입을 생략할 수 있습니다.

**남은 코드:**

```Java

(a, b) -> {
    return a[0] - b[0];
}
```
**3단계: 중괄호 `{}`와 `return` 키워드 생략**
메소드의 바디(body)가 **단 한 줄의 `return` 문으로만 이루어져 있을 경우**, 중괄호와 `return` 키워드를 함께 생략.

**최종 형태:**

```Java
(a, b) -> a[0] - b[0]
```
---

**결론적으로 아래와 같은 부분들이 생략.**

- `new Comparator<int[]>()` : `Comparator` 객체를 생성하는 부분
- `@Override public int compare(...)` : 오버라이드할 메소드 선언부
- `int[]` : 파라미터의 타입 선언
- `{ }` 와 `return` : 메소드의 바디를 감싸는 중괄호와 반환 키워드

## Q2. 실무에서는 어떻게 쓰일까?

2차원 배열 int[][] 정렬 (가장 흔한 유형)
```java
import java.util.Arrays;

int[][] points = {{3, 4}, {1, 2}, {3, 1}, {2, 4}};

// 1. x좌표(points[i][0]) 기준으로 오름차순 정렬
// 방식 1: 직접 비교 람다
Arrays.sort(points, (a, b) -> a[0] - b[0]);

// 방식 2: 키 추출 람다 (올바른 문법)
Arrays.sort(points, Comparator.comparingInt(a -> a[0]));
// 결과: [[1, 2], [2, 4], [3, 4], [3, 1]]

// 2. x좌표가 같으면, y좌표(points[i][1]) 기준으로 오름차순 정렬
Arrays.sort(points, (a, b) -> {
    if (a[0] == b[0]) {
        return a[1] - b[1]; // x좌표가 같으면 y좌표 비교
    }
    return a[0] - b[0]; // 기본은 x좌표 비교
});
// 결과: [[1, 2], [2, 4], [3, 1], [3, 4]]

// 3. 더 세련된 방식: Comparator.comparingInt 사용
Arrays.sort(points, Comparator.comparingInt((int[] a) -> a[0])
                             .thenComparingInt(a -> a[1]));
// 결과: [[1, 2], [2, 4], [3, 1], [3, 4]]
