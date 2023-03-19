## chapter8 컬렉션 API 개선

자바 8,9 에서 추가되어 우리의 삶을 편리하게 만들어 줄 새로운 컬렉션 API의 기능을 배운다.  
먼저 작은 리스트, 집합, 맵을 쉽게 만들 수 있도록 자바 9에 새로 추가된 컬렉션 팩토리를 살펴본다.  
다음으로 자바 8의 개선 사항으로 리스터와 집합에서 요소를 삭제하거나 바꾸는 관용 패턴을 적용하는 방법을 배운다.  

### 8.1 컬렉션 팩토리
자바에서는 적은 요소를 포함하는 리스트
```java
List<String> friends = Arrays.asList("Raphael", "Olivia", "Thibaut");
```
고정 크기의 리스트를 만들었으므로 요소를 갱신할 순 있지만 새 요소를 추가하거나 삭제할 수는 없다.

```
friends.set(0, "Richard"); // 문제 없음
friends.add("Tom");        //UnsupportedOperationException 발생
```

Set 활용
```java
Set<String> friends =  new HashSet<>(Arrays.asList("Raphael", "Olivia", "Thibaut")));

// 스트림 활용
Set<String> friends = Stream.of("Raphael", "Olivia", "Thibaut") 
                            .collect(Collectors.toSet());
```
하지만 두 방법 모두 매끄럽지 못하며 내부적으로 불필요한 객체 할당을 필요로 한다.

**리스트 팩토리**
- List.of
```java
List<String> friends = List.of("Raphael", "Olivia", "Thibaut");
friends.add("Tom"); // UnsupportedOperationException 발생
```
이런 예외가 나쁜건 만은 아니다. 컬렉션이 의도치 않게 변하는 것을 막을 수 있기 때문이다.

**집합 팩토리**
- Set.of
```java
Set<String> friends = Set.of("Raphael", "Olivia", "Thibaut");

// 요소가 중복되어 있다는 IllegalArgumentException 발생
Set<String> friends = Set.of("Raphael", "Olivia", "Olivia");
```

**맵 팩토리**
- Map.of : 열개이하의 키와 값쌍을 가진 맵을 만들때 유용
- Map.ofEntries: 열개이상, Map.Entry<K,V> 객체를 인자로 받으며 가변 인수로 구현시 유용
```java
Map<String, Integer> ageOfFriends =
    Map.of("Raphael", 30, "Olivia", 25, "Thibaut", 26);

Map<String, Integer> ageOfFriends = Map.ofEntries(
		entry("Raphael", 30), 
		entry("Olivia", 25),
		entry("Thibaut", 26));
```

### 8.2 리스트와 집합처리
자바 8 에서는 List, Set 인터페이스에 다음와 같은 메서드를 추가했다.
- removeIf : 프레디케이트를 만족하는 요소를 제거한다.
- replaceAll : UnaryOperator 함수를 이용해 요소를 바꾼다.
- sort : List 인터페이스에서 제공하는 기능으로 리스트를 정렬한다.

**removeIf 메서드**
```java
// ConcurrentModificationException 발생
for (Transaction transaction : transactions){
	if(Charater.isDigit(transaction.getReferenceCode().charAt(0))){
		transactions.remove(transaction);
	}
}

// for-each 내부적으로 Iterator 객체를 사용하므로 아래와 동일
for(Iterator<Transaction> iterator = transactions.iterator();
			iterator.hasNext(); ){
	Transaction transaction = iterator.next();
	if(Charater.isDigit(transaction.getReferenceCode().charAt(0))){
			// 반복하면서 별도의 두 객체를 통해 컬렉션을 바꾸고 있음
			transactions.remove(transaction);
	}
}
```
Iterator 객체 : next(), hastNext()를 이용해 소스를 질의한다.  
Collection 객체 자체 : remove()를 호출해 요소를 삭제한다.  
두개의 개별 객체가 컬력션을 관리하기 때문에 반복자의 상태와 컬렉션의 상태가 서로 동기화 되지 않는다.   

transactions.remove(transaction) 대신 Iterator를 명시적으로 사용(iterator.remove())하면 되지만 코드가 조금 복잡해졌다.

이 코드는 자바 8의 removeIf 메소드로 바꿀수 있다.

```java
transaction.removeIf(transaction ->
    Charater.isDigit(transaction.getReferenceCode().charAt(0)));
```

**replaceAll 메서드**
```java
// 첫 단어만 대문자로 바꾸는 코드
referenceCodes.stream() // [a12, C14, b13]
              .map(code -> Character.toUpperCase(code.charAt(0)) + code.substring(1))
              .collect(Collectors.toList())
              .forEach(System.out::println); // [A12, C14, B13]
```
하지만 이 코드는 새 문자열 컬렉션을 만든다.  
(Collectors.toList()) replaceAll 메서드를 쓰면 기존의 컬렉션을 바꿀수 있다.

```java
referenceCodes.replaceAll(
	code -> Charater.toUpperCase(code.charAt(0)) + code.subString(1)
);
```

### 8.3 맵 처리
자바 8부터 BiConsumer를 인수로 받는 forEach 메서드를 지원하므로 코드를 좀 더 간단하게 구현할 수 있다.
```java
for(Map.Entry<String, Integer> entry : ageOfFriends.entrySet()) {
    String friend = entry.getKey();
    Integer age = entry.getValue();
    System.out.println(friend + " is " + age + " years old");
    }
```
```java
ageOfFriends.forEach((friend, age) -> System.out.println(friend + " is " + age + " years old"));
```

**정렬 메서드**
- Entry.comparingByValue
- Entry.comparingByKey
```java
// 사람의 이름을 앞파벳순으로 정렬
ageOfFriends
    .entrySet()
    .stream()
    .sorted(Map.Entry.comparingByKey())
    .forEachOrdered(System.out::println);
```

**getOrDefault 메서드**  
기존에는 찾으려는 키가 존재하지 않으면 null이 반환되어 NullPointerException를 방지하려면 요청결과가 null인지 확인해야 했다.  
기본값을 반환하는 방식으로 NullPointerException를 방지
```java
Map<String, String> favoriteMovies = Map.ofEntries(
        Map.entry("Raphael", "Star Wars"),
        Map.entry("Cristina", "Matrix"),
        Map.entry("Olivia", "James Bond")
);

System.out.println(favoriteMovies.getOrDefault("Olivia", "Matrix")); // James Bond
System.out.println(favoriteMovies.getOrDefault("Thibaut", "Matrix")); // Matrix
```

**계산 패턴**
맵에 키 존재여부에 따라 동작을 수행 후 결과를 저장해야 하는 경우 사용되는 메서드
- computeIfAbsent : 제공된 키에 해당하는 값이 없으면, 키를 이용해 새 값을 계산하고 맵에 추가
- computeIfPresent : 제공된 키가 존재하면 새 값을 계산하고 맵에 추가
- compute : 제공된 키로 새 값을 계산하여 맵에 저장
```
Map<String, List<String>> favoriteMovies = new HashMap<>();
favoriteMovies.computeIfAbsent("Raphael", name -> new ArrayList()).add("Star Wars");
```

**교체 패턴**  
- replaceAll : BiFunction을 적용한 결과로 각 항복의 값을 교체
- Replace : 키가 존재하면 맵의 값을 바꿈
```java
Map<String, String> favoriteMovies = new HashMap<>();
favoriteMovies.put("Raphael", "Star Wars");
favoriteMovies.put("Olivia", "James Bond");
favoriteMovies.replaceAll((friend, movie) -> movie.toUpperCase());
```

**합침**  
두개의 Map을 합치는 경우에는 putAll 메서드를 사용할 수 있다.
하지만 중복된 키가 없을 경우에만 문제 없이 사용가능

for each 와 merge 메서드를 이용해 충돌을 해결 할 수 있다.

```java
Map<String, String> family = Map.ofEntries(
        Map.entry("Teo", "Star Wars"),
        Map.entry("Cristina", "James Bond")
);

Map<String, String> friends = Map.ofEntries(
        Map.entry("Raphael", "Star Wars"),
        Map.entry("Cristina", "Matrix")
);

Map<String, String> everyOne = new HashMap<>(family);
friends.forEach(
        (k, v) -> everyOne.merge(k, v, (movie1, movie2) -> movie1 + " & " + movie2))
```

### 8.4 개선된 ConcurrentHashMap
ConcurrentHashMap은 내부 자료구조의 특정 부분만 잠궈 동시 추가, 갱신 작업을 허용한다.  
동기화된 Hashtable 버전에 비해 읽기 쓰기 연산 성능이 월등하다.

**리듀스와 검색**  
ConcurrentHashMap에서는 스트림과 비슷한 종류의 세가지 새로운 연산 지원  
- forEach : 각 (키, 값) 쌍에 주어진 액션 실행
- reduce : 모든 (키, 값) 쌍을 제공된 리듀스 함수를 이용해 결과로 합침
- search : 널이 아닌 값을 반환할 때까지 각 (키, 값) 쌍에 함수를 적용  

이 연산을 사용할 수 있도록 제공하는 메서드.
- 키 값으로 연산 -> forEach, reduce, search
- 키로 연산 -> forEachKey, reduceKeys, searchKeys
- 값으로 연산 -> forEachValue, reduceValues, searchValues
- Map.Entry 객체로 연산 -> forEachEntry, reduceEntries, searchEntrie

**계수**  
int 범위를 넘어서는 상황을 대처하는 mappingCount 함수를 제공

**집합뷰**  
ConcurrentHashMap 클래스는 집합 뷰로 반환하는 keySet 이라는 새 메서드 제공  
맵을 바꾸면 집합도 바뀌고 반대로 집합을 바꾸면 맵도 영향을 받는 구조이다.  
newKeySet 이라는 새 메서드를 이용해 ConcurrentHashMap 으로 유지되는 집합을 만들 수도 있다.