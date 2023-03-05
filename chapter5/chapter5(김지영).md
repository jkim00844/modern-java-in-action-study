## chapter5. 스트림 활용

### 5.1 필터링
스트림의 요소를 선택하는 방법,  
즉 프레디케이트 필터링 방법과 고유 요소만 필터링 하는 방법 학습

**프레디케이트로 필터링**  
filter 메서드는 프레디케이트(불리언을 반환하는 함수)를 인수로 받아서 프레디케이트와 일치하는 모든 요소를 포함하는 스트림을 반환한다.  

**고유 요소 필터링**  
스트림은 고유 요소로 이루어진 스트림을 반환하는 distinct메서드도 지원한다.   
(고유 여부는 스트림에서 만든 객체의 hashCode, equals로 결정된다.)

### 5.2 스트림 슬라이싱
**프레디케이트를 이용한 슬라이싱**  
TAKEWHILE 활용
filer 연산을 이용하면 전체 스트림을 반복하면서 각요소에 프레디케이트를 적용하게 된다.
리스트가 정렬되어 있는 경우 takeWhile은 조건에 대해 참이 아닐경우 바로 거기서 멈추게 된다.
```
    List<Dish> sliceMenu1 = specialMenu.stream()
                            .takewhile(dish-> dish.getCalories() < 320)
                            .collect(toList());
```
320 보다 크거나 같은 요리가 나올 경우 작업을 중단한다.

DROPWHILE 활용  
dropWhile은 takeWhile과 정반대의 작업을 수행한다.  
프레디케이트가 처음으로 거짓이 되는 지점에서 작업을 중단하고, 지금까지 발견된 요소는 버린 남은 요소를 반환한다.

**스트림 축소**  
스트림은 주어진 값 이하의 크기를 갖는 새로운 스트림을 반환하는 limit(n) 메서드를 지원한다.  
최대 요소 n개를 반환할 수 있다.

**요소 건너뛰기**
스트림은 처음 n개 요소를 제외한 스트림을 반환하는 skip(n) 메서드를 지원한다. 

### 5.3 매핑

특정 객체에서 특정 데이터를 선택하는 작업은 데이터 처리 과정에서 자주 수행되는 연산이다.  
스트림 API의 map과 flatMap 메서드는 특정 데이터를 선택하는 기능을 제공한다.

스트림의 각 요소에 함수 적용하기
스트림은 함수를 인수로 받는 map 메서드를 지원한다.  
인수로 제공된 함수는 각 요소에 적용되며 함수를 적용한 결과가 새로운 요소로 매핑된다.  
(이 과정은 기존 값을 고치는 개념보다는 새로운 버전을 만든다는 개념에 가까우므로 변환에 가까운 매핑이라는 단어를 사용한다.)  

스트림 평면화  
flatMap은 각 배열을 스트림이 아니라 스트림의 콘텐츠로 매핑한다.
```
 List<String> uniqueCharacters = words.stream()
          .map(word -> word.split("")) // 각 단어를 개별 문자를 포함하는 배열로 변환 
          .map(Arrays::stream) // 각 배열을 별도의 스트림으로 생성
          .distinct()
          .collect(toList());
```

```
 List<String> uniqueCharacters = words.stream()
          .map(word -> word.split("")) // 각 단어를 개별 문자를 포함하는 배열로 변환 
          .flatMap(Arrays::stream) // 생성된 스트림을 하나의 스트림으로 평면화
          .distinct()
          .collect(toList());
```
flatMap 메서드는 스트림의 각 값을 다른 스트림으로 만든 다음에 모든 스트림을 하나의 스트림으로 연결하는 기능을 수행한다.

### 5.4 검색과 매칭
특정 속성이 데이터 집합에 있는지 여부를 검색하는 데이터 처리도 자주 사용된다.  
스트림 API는 allMatch, anyMatch, noneMatch, findFirst, findAny 등 다양한 유틸리티 메서드를 제공한다.
- anyMatch : 프레디케이트가 주어진 스트림에서 적어도 한 요소와 일치하는지 확인할 때  
- allMatch : 스트림의 모든 요소가 주어진 프레디케이트와 일치하는지 검사.  
- noneMatch : noneMatch는 allMatch와 반대 연산을 수행한다. 즉, 주어진 프레디케이트와 일치하는 요소가 없는지 확인한다.
- findAny : 현재 스트림에서 임의의 요소를 반환, 다른 스트림연산과 연결해서 사용할 수 있다.
- findFirst : 첫 번째 요소를 찾는데 사용  
- findFirst vs findAny    
병렬 실행에서는 첫 번째 요소를 찾기 어렵다.     
따라서 요소의 반환 순서가 상관없다면 병렬 스트림에서는 제약이 적은 findAny를 사용한다.  

Optional이란?  
findAny는 아무 요소도 반환하지 않을 수 있어 NullPointException이 발생할 수 있다.    
null은 쉽게 에러를 일으킬 수 있으므로 자바 8 라이브러리 설계자는 Optional<T>를 만들었다.  
Optional이란 값의 존재나 부재 여부를 표현하는 컨테이너 클래스다.  
값이 존재하는지 확인하고 값이 없을 때 어떻게 처리할지 강제하는 기능을 제공한다.


Optional의 기능 
- isPresent() : Optional이 값을 포함하면 true, 값을 포함하고 잇지 않으면 false 반환
- ifPresent(Consumer<T> block) : 값이 있으면 주어진 블록을 실행
- T get() : 값이 존재하면 값 반환, 없으면 NoSuchElementException 발생
- T orElse(T other) : 값이 있으면 값을 반환, 없으면 기본값 반환

### 5.5 리듀싱
모든 스트림 요소를 처리해서 값으로 도출하는 질의를 리듀싱 연산이라고 한다.  
함수형 프로그래밍 언어 용어로는 이 과정이 마치 종이를 작은 조각이 될 때까지 반복해서 접는 것과 비슷하다는 의미로 폴드(fold)라고 부른다.  

**요소의 합**
```
    int sum = 0;
    for (int x : numbers) {
            sum += x;
    }
```
reduce를 이용하면 애플리케이션의 반복된 패턴을 추상화할 수 있다.
```
    int sum = numbers.stream().reduce(0, (a, b) -> a + b);
    int sum = numbers.stream().reduce(0, Integer::sum); 
```

**최댓값과 최솟값**
```
Optional<Integer> max = numbers.stream().reduce(Integer::max);
Optional<Integer> min = numbers.stream().reduce(Integer::min);
```

**reduce 메서드의 장점과 병렬화**  
기존의 단계적 반복으로 합계를 구하는 것과 reduce를 이용해서 구하는 것은 어떤 차이가 있는가?  
반복적인 합계에서는 sum 변수를 공유해야 하므로 쉽게 병렬화하기 어렵다.  
강제적으로 동기화시키더라도 결국 병렬화로 얻어야 할 이득이 스레드 간의 소모적인 경쟁 때문에 상쇄되어 버린다.

reduce를 이용하면 내부 반복이 추상화되면서 내부 구현에서 병렬로 reduce를 실행 할 수 있게 된다.   
(포크/조인 프레임워크를 이용하는데 7장에서 깊게 살펴본다.)  

**스트림 연산 : 상태 없음과 상태 있음**  
스트림을 이용해서 원하는 모든 연산을 쉽게 구현할 수 있으며  
컬렉션으로 스트림을 만드는 stream 메서드를 parallelStream로 바꾸는 것만으로도 병렬성을 얻을 수 있다.  

map, filter 등은 입력 스트림에서 각 요소를 받아 0 또는 결과를 출력 스트림으로 보낸다.  
따라서 이들은 보통 상태가 없는, 즉 내부 상태를 갖지 않는 연산이다.

reduce, sum, max 같은 연산은 결과를 누적할 내부 상태가 필요하다.  
스트림에서 처리하는 요소 수와 관계없이 내부 상태의 크기는 한정되어 있다.

sorted나 distinct 같은 연산은 스트림의 요소를 정렬하거나 중복을 제거하려면 과거의 이력을 알고 있어야 한다.  
이러한 연산을 내부 상태를 갖는 연산이라고 한다.  
이 연산들은 어떤 요소를 출력 스트림으로 추가하려면 모든 요소가 버퍼에 추가되어 있어야 한다.  
데이터 스트림의 크기가 크거나 무한이라면 문제가 생길 수 있다.
(예를 들어 모든 소스를 포함하는 스트림을 역순으로 만들려면 첫 번째로 가장 큰 소수, 즉 세상에 존재하지 않은 수를 반환해야한다.)  

### 5.6 실전 연습
```
        // 2011년에 일어난 모든 트랜잭션을 찾아 값을 오름차순으로 정리하시오.
        List<Transaction> transactionsOf2011 =
                transactions.stream()
                            .filter(transaction -> transaction.getYear() == 2011)
                            .sorted(Comparator.comparing(Transaction::getValue))
                            .collect(toList());

        // 거래자가 근무하는 모든 도시를 중복 없이 나열하시오.
        List<String> cities =
                transactions.stream()
                            .map(Transaction::getTrader)
                            .map(Trader::getCity)
                            .distinct() 
                            .collect(toList());

        // Cambridge에서 근무하는 모든 거래자를 찾아서 이름순으로 정렬하시오.
        List<Trader> traders =
                transactions.stream()
                            .map(Transaction::getTrader)
                            .filter(trader -> trader.getCity().equals("Cambridge"))
                            .distinct()
                            .sorted(Comparator.comparing(Trader::getName))
                            .collect(toList());

        // 모든 거래자의 이름을 알파벳순으로 정렬해서 반환하시오.
        String traderStr =
                transactions.stream()
                            .map(Transaction::getTrader)
                            .map(Trader::getName)
                            .distinct()
                            .sorted()
                            .reduce("", (a, b) -> a + b); // .collect(joining()); // 6장에서 설명

        // Miilan에 거래자가 있는가?
        boolean existed =
                transactions.stream()
                            .map(Transaction::getTrader) // .anyMatch(transaction -> transaction.getTrader().getCity().equals("Milan"));
                            .map(Trader::getCity)
                            .anyMatch(city -> city.equals("Milan"));

        // Cambridge에 거주하는 거래자의 모든 트랜잭션 값을 출력하시오.
        transactions.stream()
                    .filter(transaction -> transaction.getTrader().getCity().equals("Cambridge"))
                    .map(Transaction::getValue)
                    .forEach(System.out::println);

        // 전체 트랜잭션 중 최댓값은 얼마인가?
        Optional<Integer> highestValue = transactions.stream()
                                            .map(Transaction::getValue)
                                            .reduce(Integer::max);

        // 스트림은 Comparator를 인수로 받는 min,max 메서드 제공
        // 전체 트랜잭션 중 최솟값은 얼마인가?
        Optional<Integer> smallestValue = transactions.stream()
                                            .min(Transaction::getValue);
```

### 5.7 숫자형 스트림  
```
int calories = menu.stream()
                    .map(Dish::getCalories)
                    .reduce(0, Interger::sum);
```
위 코드에는 박싱 비용이 숨어있다.    
내부적으로 합계를 계산하기 전에 Integer를 기본형으로 언박싱해야 한다.  
다음 코드처럼 직접 sum메서드를 호출할수 있다면 더 좋지 않을까?
```
int calories = menu.stream()
                    .map(Dish::getCalories)
                    .sum();
```
스트림 API에서는 숫자 스트림을 효율적으로 처리할 수 있도록 기본형 특화 스트림을 제공한다.  
박싱 비용을 피할 수 있도록 int 요소에 특화된 IntStream, double 요소에 특화된 DoubleStream, long요소에 특화된 LongStream을 제공한다.  

**기본형 특화 스트림**  
숫자 스트림으로 매핑
```
int calories = menu.stream()
                    .mapToInt(Dish::getCalories) // IntStream 반환
                    .sum();
```
**객체 스트림으로 복원하기**
```
IntStream intStream = menu.stream().mapToInt(Dish::getCalories); // 스트림을 숫자 스트림으로 변환
Stream<Integer> stream = intStream.boxed(); // 숫자 스트림을 스트림으로 변환
```

기본값 : OptionalInt  
Intstream의 반환타입은 OptionalInt이다.  
OptionalInt, OptionalDouble, OptionalLong 세 가지 기본형 특화 스트림 버전도 제공한다.  
스트림에 요소가 없는 상황과 실제 최댓값이 0인 상황을 어떻게 구별할 수 있을까?  
OptionalInt를 사용해서 최댓값이 없는 상황에서 기본값을 명시적으로 정의 할 수 있다.
```
// 컴파일 에러 -> Instream의 max()의 리턴 타입은 OptionalInt
Optional<Integer> maxCalories = menu.stream()
                              .mapToInt(Dish::getCalories)
                              .max();

OptionalInt maxCalories = menu.stream()
                              .mapToInt(Dish::getCalories)
                              .max();

// 값이 없을 때 기본 최댓값을 명시적으로 설정
int max = maxCalories.orElse(1);
```

**숫자 범위**  
IntStream과 LongStream에서는 range와 rangeClosed라는 두 가지 정적 메서드를 제공한다.  
첫 번째 인수로 시작값을, 두 번째 인수로 종료값을 갖는다.
```
IntStream evenNumber = IntStream.rangeClosed(1, 100)
                        .filter(n ->n%2==0);
System.out.println(evenNumber.count());
```

### 5.8 스트림 만들기
일련의 값, 배열, 파일, 심지어 함수를 이용한 무한 스트림 만들기 등 다양한 방식으로 스트림을 만드는 방법을 설명한다.  

**값으로 스트림 만들기**  
Stream.of를 이용
```
Stream<String> stream = Stream.of("Modern", "Java", "In", "Action");
stream.map(String::toUpperCase)
			.forEach(System.out::println);
			
// empty 메서드를 이용해서 스트림을 비울 수 있다.			
Stream<String> emptyStream = Stream.empty();
```

**null이 될 수 있는 객체로 스트림 만들기**  
```
// 자바9 이전. null을 명시적으로 확인했어야 했음
String homeValue = System.getProperty("home");
Stream<String> homeValueStream = homeValue == null ? Stream.empty() : Stream.of(homeValue);
        
// 자바9 이후
Stream<String> homeValueStream = Stream.ofNullable(System.getProperty("home"));
```

**배열로 스트림 만들기**
```
int[] numbers = {2, 3, 5, 7, 11, 13};
int sum = Arrays.stream(numbers).sum();
```

**파일로 스트림 만들기**
```
long uniqueWords = 0;

try (Stream<String> lines = 
    Files.lines(Path.get("data.txt"), Charset.defaultCharset())) {
        uniqueWords = lines.flatMap(line -> Arrays.stream(line.split(" ")))
                            .distinct() // 고유 단어 수 계산 + 중복제거
                            .count();
} catch (IOException e) {
}
```
Stream 인터페이스는 자원을 자동으로 해제할 수 있는 AutoCloseable 인터페이스를 구현하므로 finally에서 자원을 닫을 필요가 없다.  

**함수로 무한 스트림 만들기**  
Stream.iterate와 Stream.generate로 무한 스트림을 만들 수 있다.
```
Stream.iterate(0, n -> n + 2)
			.limit(10) // 10개로 제한
      .forEach(System.out::println);
```
```
Stream.generate(Math::random)
      .limit(5) // 5개로 제한
      .forEach(System.out::println);
```
무한 스트림의 경우 무한한 크기를 가지기 때문에 limit를 이용해서 명시적으로 스트림의 크기를 제한해야 한다.  
마찬가지로 무한 스트림의 요소는 무한적이므로 계산이 반복되므로 정렬하거나 리듀스할 수 없다.
