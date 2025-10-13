# String 메소드 정리

### String의 불변성
Java에서 String 객체는 한번 생성되면 변경 X.
replace(), substring() 같은 메소드들은 문자열을 새로 생성 함.

### 1. 문자(열) 조회 및 추출 (⭐⭐⭐)

| 메소드 | 설명 | 코드 예시 | 코테 Tip |
| :--- | :--- | :--- | :--- |
| `char charAt(int index)` | 특정 인덱스에 있는 `char` 한 글자를 반환합니다. (0부터 시작) | `String s = "hello"; char c = s.charAt(1); // 'e'` | `for`문으로 문자열을 한 글자씩 순회할 때 가장 기본적으로 사용됩니다. |
| `int length()` | 문자열의 총 길이를 반환합니다. | `String s = "hello"; int len = s.length(); // 5` | `for`문의 반복 조건 (`i < s.length()`)을 설정할 때 필수입니다. |
| `String substring(int begin, int end)` | `begin` 인덱스부터 `end-1` 인덱스까지의 부분 문자열을 잘라내어 **새로운 문자열로 반환**합니다. | `String s = "hello"; String sub = s.substring(1, 4); // "ell"` | `end` 인덱스는 포함되지 않는다는 점을 꼭 기억하세요! |
| `char[] toCharArray()` | 문자열을 `char` 타입의 배열로 변환하여 반환합니다. | `String s = "hello"; char[] arr = s.toCharArray(); // ['h','e','l','l','o']` | 문자열의 글자 순서를 바꿔야 할 때 (e.g., 정렬) 매우 유용합니다. `Arrays.sort(arr);` |

### 2. 문자열 검색 및 확인

| 메소드 | 설명 | 코드 예시 | 코테 Tip |
| :--- | :--- | :--- | :--- |
| `boolean contains(CharSequence s)` | 특정 문자열을 포함하고 있는지 `true`/`false`로 반환합니다. | `String s = "hello world"; boolean b = s.contains("world"); // true` | 특정 조건의 문자열을 필터링할 때 간단하게 사용할 수 있습니다. |
| `int indexOf(String str)` | 특정 문자열이 처음으로 나타나는 위치의 시작 인덱스를 반환합니다. 없으면 `-1`을 반환. | `String s = "hello"; int idx = s.indexOf("l"); // 2` | 문자열 포함 여부를 확인(`-1`인지 아닌지)하거나, 특정 문자 기준으로 파싱할 때 사용합니다. |
| `boolean startsWith(String prefix)` | 문자열이 특정 문자열로 시작하는지 확인합니다. | `String s = "https://google.com"; boolean b = s.startsWith("https"); // true` | 접두사 관련 규칙이 있는 문제에서 유용합니다. |
| `boolean endsWith(String suffix)` | 문자열이 특정 문자열로 끝나는지 확인합니다. | `String s = "photo.jpg"; boolean b = s.endsWith(".jpg"); // true` | 파일 확장자나 특정 형식의 꼬리 문자열을 확인할 때 좋습니다. |
| `boolean equals(Object anObject)` | 두 문자열의 내용이 완전히 같은지 비교합니다. (대소문자 구분) | `String a = "hello"; String b = new String("hello"); boolean eq = a.equals(b); // true` | **문자열 내용 비교는 `==`가 아닌 반드시 `equals()`를 사용해야 합니다!** (매우 중요) |

### 3. 문자열 변환 및 수정 (새로운 문자열 생성)

| 메소드 | 설명 | 코드 예시 | 코테 Tip |
| :--- | :--- | :--- | :--- |
| `String replace(CharSequence old, CharSequence new)` | 특정 문자(열)을 찾아 전부 **새로운 문자(열)로 교체**한 새 문자열을 반환합니다. | `String s = "hello"; String r = s.replace("l", "w"); // "hewwo"` | 특정 문자를 제거하고 싶을 때 `s.replace("a", "")` 와 같이 빈 문자열로 교체하는 트릭을 씁니다. |
| `String replaceAll(String regex, String replacement)` | `replace`와 유사하지만, 첫 번째 인자로 **정규식(regex)**을 받습니다. | `String s = "a1b2c3d"; String r = s.replaceAll("[0-9]", "#"); // "a#b#c#d#"` | 숫자, 특수문자 등 복잡한 패턴을 한 번에 바꾸고 싶을 때 사용합니다. |
| `String toLowerCase()` / `toUpperCase()` | 문자열 전체를 소문자 또는 대문자로 변경한 새 문자열을 반환합니다. | `String s = "Hello"; String l = s.toLowerCase(); // "hello"` | 대소문자를 구분하지 않는 비교를 할 때, 양쪽 모두 소문자(또는 대문자)로 바꿔서 비교합니다. |
| `String trim()` | 문자열의 **앞뒤에 있는 공백(whitespace)을 모두 제거**한 새 문자열을 반환합니다. | `String s = "  hello  "; String t = s.trim(); // "hello"` | 사용자 입력을 받을 때 의도치 않은 공백이 포함될 수 있으므로, 처리 전에 `trim()`을 해주는 것이 안전합니다. |

### 4. 문자열 분리 및 결합

| 메소드 | 설명 | 코드 예시 | 코테 Tip |
| :--- | :--- | :--- | :--- |
| `String[] split(String regex)` | 특정 구분자(정규식)를 기준으로 문자열을 잘라 **문자열 배열**로 반환합니다. | `String s = "a,b,c"; String[] arr = s.split(","); // ["a", "b", "c"]` | 공백을 기준으로 입력을 나눌 때 (`br.readLine().split(" ")`) 가장 많이 사용됩니다. |
| `static String join(CharSequence d, Char... e)` | 여러 문자열을 특정 구분자(`d`)로 이어 붙여 **하나의 문자열**로 만들어 반환합니다. (Java 8+) | `String[] arr = {"a", "b", "c"}; String j = String.join("-", arr); // "a-b-c"` | 배열이나 리스트에 담긴 문자열들을 출력 형식에 맞게 합칠 때 매우 편리합니다. |

---

### 🚀 성능 보너스 Tip: `+` 연산 대신 `StringBuilder` 사용하기

`for`문 안에서 `String`을 `+` 연산자로 계속 이어 붙이면, 매번 새로운 `String` 객체가 생성되어 메모리와 속도에 매우 비효율적입니다. 문자열을 반복적으로 조립해야 할 때는 반드시 `StringBuilder`를 사용하는 습관을 들이는 것이 좋습니다.

#### 나쁜 예 ❌
```java
String result = "";
for (int i = 0; i < 10000; i++) {
    result += i; // 매번 새로운 String 객체가 생성되어 매우 느림
}
```

좋은 예 ✅:
```java
StringBuilder sb = new StringBuilder();
for (int i = 0; i < 10000; i++) {
    sb.append(i); // 기존 버퍼에 내용을 추가하므로 훨씬 빠름
}
String result = sb.toString(); // 마지막에 한 번만 String으로 변환
```