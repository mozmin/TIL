# 기본 규칙 (매우 중요)

- `add / remove / element` → 실패 시 **예외**
- `offer / poll / peek` → 실패 시 **특수값**(`false` 또는 `null`)
- `ArrayDeque`/`PriorityQueue`는 **null 저장 불가** → `offer(null)`은 NPE
---

# Deque (큐, 스텍)

```java
Deque<Integer> d = new ArrayDeque<>();
```

- 앞쪽
    - `offerFirst(e)`, `pollFirst()`, `peekFirst()`
    - `addFirst(e)`, `removeFirst()`, `getFirst()`
- 뒤쪽
    - `offerLast(e)`, `pollLast()`, `peekLast()`
    - `addLast(e)`, `removeLast()`, `getLast()`
- 스택처럼 사용
    - `push(e)` = `addFirst(e)`
    - `pop()` = `removeFirst()`
    - `peek()` = `peekFirst()`
- 기타
    - `isEmpty()`, `size()`, `clear()`

> 코테에서 스택은 ArrayDeque로: Deque<int[]> st = new ArrayDeque<>(); st.push(x); st.pop();
>
---

```java
import java.util.ArrayDeque;
import java.util.Deque;

public class DequeAsStackQueue {
    public static void main(String[] args) {
        Deque<Integer> dq = new ArrayDeque<>();

        // 1) Queue (FIFO): 뒤에 넣고(offerLast) 앞에서 뺀다(pollFirst)
        dq.offerLast(10);
        dq.offerLast(20);
        dq.offerLast(30);
        System.out.println("Queue poll -> " + dq.pollFirst()); // 10
        System.out.println("Queue poll -> " + dq.pollFirst()); // 20

        // 2) Stack (LIFO): top을 앞쪽으로 보고 push/pop 사용
        dq.clear();
        dq.push(1);  // = addFirst(1)
        dq.push(2);  // = addFirst(2)
        dq.push(3);  // = addFirst(3)
        System.out.println("Stack peek -> " + dq.peek()); // 3
        System.out.println("Stack pop  -> " + dq.pop());  // 3
        System.out.println("Stack pop  -> " + dq.pop());  // 2

        // (대안) 뒤쪽을 top으로 쓰고 싶다면:
        dq.clear();
        dq.offerLast(100); // push 대체
        dq.offerLast(200);
        dq.offerLast(300);
        System.out.println("Alt-Stack (back as top) -> " + dq.pollLast()); // 300
    }
}

```

---

# PriorityQueue (최소 힙)

```java
PriorityQueue<Integer> pq = new PriorityQueue<>(); // 오름차순(최소힙)
PriorityQueue<int[]> pq2 = new PriorityQueue<>((a,b) -> a[0]-b[0]); // 커스텀
```

- `offer(e)` : 삽입 (O(log n))
- `poll()` : 최솟값 꺼내기 (O(log n), 비면 `null`)
- `peek()` : 최솟값 보기 (O(1), 비면 `null`)
- 기타
    - `remove(x)` : 특정 원소 제거 (O(n))
    - **최대힙** 만들기: `new PriorityQueue<>((a,b)-> b-a)` 또는 `Collections.reverseOrder()`

> 배열/튜플 비교가 필요하면 Comparator로 묶기!

```java
import java.util.*;

public class PriorityQueueExample {
    public static void main(String[] args) {
        PriorityQueue<Integer> pq = new PriorityQueue<>(); // 자연순서(오름차순)
        pq.offer(30);
        pq.offer(10);
        pq.offer(20);
        while (!pq.isEmpty()) {
            System.out.print(pq.poll() + " "); // 10 20 30
        }
    }
}
```
---

# 빈 큐 처리 패턴 (NPE/예외 방지)

```java
Integer x = q.poll();
if (x == null) { /* 비어있음 처리 */ }
```

또는

```java
if (!q.isEmpty()) { int x = q.poll(); }
```

---

# 초기화 패턴 (입출력 빠르게)

```java
Deque<int[]> dq = new ArrayDeque<>(); // 슬라이딩 윈도우/모노톤 큐
Deque<Integer> st = new ArrayDeque<>(); // 스택
Queue<Integer> q = new ArrayDeque<>();  // 일반 큐
PriorityQueue<int[]> heap = new PriorityQueue<>((a,b)->a[0]-b[0]); // 커스텀 키
```

---

# 자주 쓰는 형태

### 1) BFS (Queue)

```java
Deque<int[]> q = new ArrayDeque<>();
q.offer(new int[]{sr, sc});
while (!q.isEmpty()) {
    int[] cur = q.poll();
    // 인접 정점 처리 → q.offer(next)
}
```

### 2) 스택(괄호/단조스택)

```java
Deque<Integer> st = new ArrayDeque<>();
for (int i = 0; i < n; i++) {
    while (!st.isEmpty() && arr[st.peek()] >= arr[i]) st.pop();
    // 여기서 st.peek()는 arr[i]보다 작은 “이전 인덱스”
    st.push(i);
}
```

### 3) 슬라이딩 윈도우 최대값 (모노톤 deque)

```java
Deque<Integer> dq = new ArrayDeque<>();
for (int i = 0; i < n; i++) {
    while (!dq.isEmpty() && dq.peekFirst() <= i - k) dq.pollFirst();      // 범위 밖 제거
    while (!dq.isEmpty() && nums[dq.peekLast()] <= nums[i]) dq.pollLast(); // 값 작은 뒤쪽 제거
    dq.offerLast(i);
    if (i >= k-1) ans[i-k+1] = nums[dq.peekFirst()];
}
```

### 4) 우선순위 큐로 K번째 원소

```java
PriorityQueue<Integer> maxK = new PriorityQueue<>(); // 최소힙, size<=k 유지
for (int v : arr) {
    maxK.offer(v);
    if (maxK.size() > k) maxK.poll(); // 가장 작은 것 제거 → 큐에는 항상 큰 k개
}
// top = k번째 큰 값
```

---

# 시간 복잡도 (평균/상수 생략)

- `ArrayDeque` : 앞/뒤 `offer/poll/peek` **암묵적 O(1)**
- `PriorityQueue` : `offer/poll` O(log n), `peek` O(1)
- `contains/remove(obj)` : 보통 O(n)
---

# 실수 방지 체크리스트

- `poll()`/`peek()`은 **null**을 반환할 수 있음 → 언박싱 주의 (`Integer` vs `int`)
- `ArrayDeque`/`PriorityQueue`에 **null 금지**
- `PriorityQueue`는 전체 정렬 아님(최솟값만 보장)
- 스택은 **`ArrayDeque`로 구현** (`Stack` 지양)

## FAQ

**Q1. `add/remove/element` vs `offer/poll/peek` 무엇을 써야 하나요?**

A. 코테에선 **예외 없는 `offer/poll/peek`** 추천(로직 단순화).

**Q2. 스택은 꼭 `Stack` 써야 하나요?**

A. 아니요. **`ArrayDeque`로 `push/pop/peek`** 쓰는 것이 표준.

**Q3. 최대 힙이 필요합니다.**

A. `new PriorityQueue<>((a,b)-> b-a)` 또는 `new PriorityQueue<>(Collections.reverseOrder())`.

**Q4. 빈 컨테이너 안전 핸들링은?**

A. `poll()` 후 `null` 체크 또는 `isEmpty()` 선 체크.