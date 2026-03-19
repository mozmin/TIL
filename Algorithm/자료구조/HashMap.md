# HashMap

`HashMap`은 **키(Key)**와 **값(Value)**을 하나의 쌍(Pair)으로 묶어서 저장하는 자료구조. Map 인터페이스를 구현한 대표적인 클래스이며, 내부적으로 해시 테이블(Hash Table)을 사용하여 데이터를 저장하기 때문에 데이터를 검색하거나 삽입할 때 속도가 빠름.



### 핵심 특징
* **빠른 접근 속도:** 해시 함수를 통해 데이터가 저장될 위치(버킷, Bucket)를 바로 계산하기 때문에, 데이터 삽입 및 탐색에 있어 평균적으로 $O(1)$의 시간복잡도를 가짐.
* **순서 미보장:** 데이터가 입력된 순서나 키의 정렬 상태를 보장하지 않는다. (순서 보장이 필요하다면 `LinkedHashMap`, 키 정렬이 필요하다면 `TreeMap`을 사용 필요.)
* **중복 키 불가, 중복 값 허용:** 동일한 키로 데이터를 다시 넣으면 기존 값이 새로운 값으로 덮어씌워짐. 반면, 값(Value)은 중복되어도 상관없음.
* **Null 허용:** 키와 값 모두 `null`을 저장할 수 있다. (단, 키가 `null`인 데이터는 하나만 존재 가능.)

---

## 2. HashMap 주요 메서드

### 🔹 객체 생성
```java
HashMap<String, Integer> map = new HashMap<>();
```

## 🔹 데이터 추가 
put(K key, V value): 맵에 주어진 키와 값을 저장. 이미 존재하는 키라면 값을 덮어씀.

putIfAbsent(K key, V value): 주어진 키가 맵에 없거나 매핑된 값이 null일 때만 데이터를 저장.

## 🔹 데이터 조회
get(Object key): 주어진 키에 매핑된 값을 반환. 키가 없으면 null을 반환.

getOrDefault(Object key, V defaultValue): 키가 존재하면 해당 값을 반환하고, 없으면 지정한 defaultValue를 반환. (코딩 테스트에서 빈도수 계산이나 누적합을 구할 때 매우 유용.)

## 🔹 데이터 삭제
remove(Object key): 특정 키와 그에 해당하는 값을 삭제.

clear(): 맵에 있는 모든 데이터를 한 번에 삭제.

## 🔹 데이터 확인
containsKey(Object key): 맵에 특정 키가 존재하는지 여부를 반환. (true/false)

containsValue(Object value): 맵에 특정 값이 존재하는지 여부를 반환. (true/false)

isEmpty(): 맵이 비어있는지 확인.

size(): 맵에 저장된 키-값 쌍의 총 개수를 반환.

## 🔹 데이터 순회 (Iteration)
```java
1. keySet()을 이용한 키 순회 (값을 찾을 때 다시 get()을 호출해야 하므로 약간 느릴 수 있음)
for (String key : map.keySet()) {
    System.out.println("Key: " + key + ", Value: " + map.get(key));
}
```

2. entrySet()을 이용한 키-값 동시 순회 (성능상 가장 권장되는 방식)
```java
for (Map.Entry<String, Integer> entry : map.entrySet()) {
    System.out.println("Key: " + entry.getKey() + ", Value: " + entry.getValue());
}
```

3. values()를 이용한 값만 순회
```java
for (Integer value : map.values()) {
    System.out.println("Value: " + value);
}
```
