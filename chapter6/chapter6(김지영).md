## chapter6. 스트림으로 데이터 수집

통화별로 트랜잭션을 그룹화한 코드  - 명령형 버전
```
 Map<Currency, List<Transaction>> transactionByCurrencies = new HashMap<>(); //  그룹화한 트랜잭션을 저장할 맵을 생성

    for (Transactin transaction : transactions) {   //  트랜잭션 리스트를 반복
        Currency currency = transaction.getCurrency();
        List<Transaction> transactionsForCurrency = transactionsByCurrencies.get(currency);

        if (transactionForCurrency = null) {    //  현재 통화를 그룹화하는 맵에 항목이 없으면 항목을 만든다.
            transactionsForCurrency = new ArrayList<>();
            transactionsByCurrencies.put(currency, transactionsForCurrency);
        }

        transactinsForCurrency.add(transaction);    //  같은 통화를 가진 트랜잭션 리스트에 현재 탐색 중인 트랜잭션을 추가
    }
```

통화별로 트랜잭션을 그룹화한 코드  - 함수형 버전
```
Map<Currency, List<Transaction>> transactionsByCurrencies = transactions.stream().collect(groupingBy(Transaction::getCurrency));
```
명령형 코드에서는 문제를 해결하는 과정에서 다중 루프와 조건문을 추가하며 가독성과 유지보수성이 떨어진다.  
함수형 코드에서는 무엇을 원하는지 직접 명시할 수 있어서 간결하게 코드를 구현할 수 있다.    
다수준(multilevel)으로 그룹화를 수행할 때 명령형 프로그래밍과 함수형 프로그래밍의 차이점이 더욱 두드러진다.

### 6.1 컬렉터란 무엇인가?
**고급 리듀싱 기능을 수행하는 컬렉터**  
함수형 API의 또 다른 장점으로 높은 수준의 조합성과 재사용성을 꼽을 수 있다.  
ollect로 결과를 수집하는 과정을 간단하며서도 유연한 방식으로 정의할 수 있다는 점이 컬렉터의 최대 강점이다.  
스트림에 collect를 호출하면 내부적으로 스트림의 요소에 리듀싱 연산이 수행된다.     
명령형 프로그래밍에서 개발자가 직접 구현해야 했던 작업이 자동으로 수행된다.  
collect에서는 리듀싱 연산을 이용해서 스트림의 각 요소를 방문하면서 컬렉터가 작업을 처리한다.  
함수를 요소로 변환할 때는 컬렉터를 적용하며 최종 결과를 저장하는 자료구조에 값을 누적한다.  
Collector 인터페이스의 메서드를 어떻게 구현하느냐에 따라 스트림에 어떤 리듀스 연산을 수행할지 결정된다.  
Collectors 유틸리티 클래스는 자주 사용하는 컬렉터 인스턴스를 손쉽게 생성할 수 있는 정적 팩토리 메서드를 제공한다.  
ex) toList

미리 정의된 컬렉터
Collectors에서 제공하는 메서드의 기능은 크게 세 가지로 구분할 수 있다.
- 스트림 요소를 하나의 값으로 리듀스하고 요약
- 요소 그룹화
- 요소 분할

### 리듀싱과 요약
- Collectors.counting()  
다른 컬렉터와 함께 사용할 때 유용하다.

```
menu.stream().collect(Collectors.counting())
menu.stream().count()
```

- Collectors.maxBy, Collectors.minBy   
스트림값에서 최댓값과 최솟값 검색할 때 사용하는 컬렉터  
해당 컬렉터는 스트림의 요소를 비교하는데 사용할 Comparator인수를 받는다
```
Optional<Dish> mostCalorieDish = 
        menu.stream()
            .collect(Collectors.maxBy(Comparator.comparingInt(Dish::getCalories)));
```

**요약 연산**  
필드의 합계나 평균 등을 반환하는 연산  
리듀싱 기능이 자주 사용된다.   
- Collectors.summingInt  
객체를 int로 매핑하는 함수를 인수로 받는다.     
인수로 전달된 함수는 객체를 int로 매핑한 컬렉터를 반환한다.  
```
// 메뉴의 총 칼로리 합
int totalColories = menu.stream().collect(summingInt(Dish:getCalories));
```

**문자열 연결**  
- Collectors.joining()
하나의 문자열로 연결해서 반환한다.
```
// 메뉴명을 스트링으로
String shortMenu = menu.stream().map(Dish::getName).collect(Collectors.joining(", "));
```
**범용 리듀싱 요약 연산**  
모든 Collector는 reducing 팩토리 메서드로도 정의할 수 있다.  
가독성이나 편리성 측면에서 권장하지 않는다.
- Collectors.reducing()
```
// 메뉴의 총 칼로리 합
int totalCalories = menu.stream().collect(Collectors.reducing(0, Dish::getCalories, (i, j) -> i + j)
```

**collect vs reduce**  
collect와 reduce는 다른 메서드이지만 같은 기능을 구현할 수 있다.  
이 메서드들의 차이는 의미론적인 부분과 실용성 부분에서 이야기해볼 수 있다.
```
Stream<Integer> stream = Arrays.asList(1, 2, 3, 4, 5, 6).stream();

// collect로 만든 리스트
List<Integer> collectedList = stream.collect(toList());

// reduce로 만든 리스트
List<Integer> reducedList = stream.reduce(
                new ArrayList<>(), 
				(List<Integer> l, Integer e) -> { // 누적자
                   l.add(e);
                   return l;},
				(List<Integer> l1, List<Integer> l2) -> { // 결합자
                   l1.addAll(l2);
                   return l1;});
```
collect 메서드는 도출하려는 결과를 누적하는 컨테이너를 바꾸도록 설계된 메서드다.  
반면 reduce는 두 값을 하나로 도출하려는 불변형 연산이라는 점에서 의미론적인 문제가 일어난다.  
위 예제에서 reduce 메서드는 누적자로 사용된 리스트를 변환시키므로 reduce를 잘못 활용한 예에 해당한다.  
reduce를 잘못 사용하면 실용적인 문제(병렬성)도 발생한다. (7장에서 자세히)  
병렬성을 확보하려면 collect 메서드로 리듀싱연산을 구현하는 것이 바랍직하다.  

**같은 연산도 다양한 방식으로 수행 할 수 있다.**  
```
// 메뉴의 총 칼로리 합

// 1. reduce 메소드만 사용
int totalCalories = menu.stream().collect(reducing(0, Dish::getCalories, Integer::sum));

// 2. Optional<int> 사용
int totalCalories = menu.stream().map(Dish::getCalories).reduce(Integer::sum).get();

// 3. intStream 사용
int totalCalories = menu.stream().mapToInt(Dish::getCalories).sum();
```

**자신의 상황에 맞는 최적의 해법 선택**  
함수형 프로그래밍(특히 자바8의 컬렉션 프레임워크에 추가된 함수형 원칙에 기반한 새로운 API)에서는 하나의 연산을 다양한 방법으로 해결할 수 있다.   
또한 스트림 인터페이스에서 직접 제공하는 메서드를 이용하는 것에 컬렉터를 이용하는 코드가 더 복잡하다.  
코드가 복잡한 대신 재사용성과 커스터마이즈 가능성을 제공하는 높은 수준의 추상화와 일반화를 얻을 수 있다.  
→ 문제를 해결할 수 있는 다양한 해결 방법을 확인한 다음에 가장 일반적으로 문제에 특화된 해결책을 고르는 것이 바람직하다. 

### 6.3 그룹화
데이터 집합을 하나 이상의 특성으로 분류해서 그룹화하는 연산도 데이터베이스에서 많이 수행되는 작업으로  
자바 8의 함수형을 이용하면 가독성 있는 한 줄의 코드로 그룹화를 구현할 수 있다.  
- Collectors.groupingBy()
```
// 메뉴의 타입
Map<Dish.Type, List<Dish>> dishesByType =
				menu.stream().collect(groupingBy(Dish::getType));
```

**그룹화된 요소 조작**
- Collectors.filtering()   
```
// 500칼로리가 넘는 요리만 필터
Map<Dish.Type, List<Dish>> caloricDishesByType =
  menu.stream().filter(dish -> dish.getCalories() > 500)
                .collect(groupingBy(Dish::getType));

{OTHER = [french fries, pizza], MEAT = [pork, beef]}  
```
위 코드의 문제는 filter의 프레디케이트를 만족하는 요소가 없는 경우에 결과 맴에서 해당 키 자체가 사라진다.  
Collector 안으로 필터 프레디테이트를 이동하여 이 문제 해결  
```
Map<Dish.Type, List<Dish>> caloricDishesByType =
    menu.stream()
        .collect(groupingBy(Dish::getType, 
                filtering(dish -> dish.getCalories() > 500, toList())));

{OTHER = [french fries, pizza], MEAT = [pork, beef], FISH =[]}               
```
- Collectors.mapping()  
  다른 하나로 매핑 함수를 이용해 요소를 변환하는 작업  
```
//그룹의 각 요리를 관련 이름 목록으로 변환
Map<Dish.Type, List<String>> dishNamesByType = 
    menu.stream()
        .collect(groupingBy(Dish::getType, mapping(Dish::getName, toList())));
```
두 수준의 리스트를 한 수준으로 평면화하기 위해 flatMap을 수행해야 한다.  
이전처럼 각 그룹에 수행한 flatMappig 연산 결과를 수집해서 리스트가 아니라 집합으로 그룹화해 중복 태그를 제거할 수 있다.
```
Map<Dish.Type, Set<String>> dishNamesByType =
  menu.stream()
  .collect(groupingBy(Dish::getType, 
           flatMapping(dish -> dishTags.get(dish.getName()).stream(), toSet())));
```

**다수준 그룹화**
- Collectors.groupingBy   
일반적인 분류함수와 컬렉터를 인수로 받는다.
```
Map<Dish.Type, Map<CaloricLevel, List<Dish>>> dishesByTypeCaloricLevel =
    menu.stream().collect(groupingBy(Dish::getType,
                          groupingBy(dish -> {
                                    if (dish.getCalories() <= 400)
                                        return CaloricLevel.DIET;
                                    else if (dish.getCalories() <= 700)
                                        return CaloricLevel.NORMAL;
                                    else return CaloricLevel.FAT;
                          }))); 
                          
{MEAT={DIET=[chicken], NORMAL=[beef], FAT=[pork]},
FISH={DIET=[prawns], NORMAL=[salmon]}
OTHER={DIET=[rice, seasonal fruit], NORMAL=[french fries, pizza]}                          
```

**서브그룹으로 데이터 수집**  
위 에제에서는 두번째 groupingBy 컬렉터를 외부 컬렉터로 전달해서 다수준 그룹화 연산을 구현했다.  
사실 첫 번째 groupingBy로 넘겨주는 컬렉터의 형식의 제한이 없다.
```
// groupingBy의 두번째 인수로 counting 컬렉터를 전달해서 메뉴에서 요리의 수를 종류별로 계산
Map<Dish.Type, Long> typesCount = menu.stream().collect(
                groupingBy(Dish::getType, counting()));

{MEAT=3, FISH=2, OTHER=4}
```

```
// 요리의 종류를 분류하는 컬렉터로 메뉴에서 가장 높은 칼로리를 가진 요리를 찾는 코드
Map<Dish.Type, Optional<Dish>> mostCaloricByType = menu.stream()
                .collect(groupingBy(Dish::getType, maxBy(Comparator.comparing(Dish::getCalories))));
                
{FISH=Optional[salmon], OTHER=Optional[pizza], MEAT=Optional[pork]}
```
- Collectors.collectingAndThen()  
맵의 모든 값을 Optional로 감쌀 필요가 없으므로 Optional을 삭제할 수 있다.
```
Map<Dish.Type, Dish> mostCaloricByType = menu.stream().collect(
    groupingBy(Dish::getType,
        collectingAndThen(
            maxBy(comparingInt(Dish::getCalories)),
        Optional::get)));
        
{FISH=salmon], OTHER=pizza, MEAT=pork}
```

**groupingBy와 함께 사용하는 다른 컬렉터 예제**  
- summingInt()
```
// 메뉴에 들어있는 모든 요리의 칼로리으 합계
Map<Dish.Type, Integer> totalCaloriesByType= menu.stream().collect(
        groupingBy(Dish::getType,
        summingInt(Dish::getCalories)));
```

- mapping()  
mapping 메서드는로 만들어진 컬렉터도 groupingBy와 자주 사용된다.  
mapping 메서드는 스트림의 인수를 변환하는 함수와 변환 함수의 결과 객체를 누적하는 컬렉터를 인수로 받는다.    
mapping은 입력 요소를 누적하기 전에 매핑 함수를 적용해서 다양한 형식의 객체를 주어진 형식의 컬렉터에 맞게 변환하는 역할을 한다.  
```
Map<Dish.Type, Set<CaloricLevel>> dishesByTypeCaloricLevel =
    menu.stream().collect(
        groupingBy(Dish::getType,
            mapping(dish -> {
                if (dish.getCalories() <= 400) { return CaloricLevel.DIET;
                } else if (dish.getCalories() <= 700) { return CaloricLevel.NORMAL;
                } else { return CaloricLevel.FAT ;
                }},
        toSet())));
```

## 6.4 분할
분할 함수(partitioning function)라 불리는 프레디케이트를 분류 함수로 사용하는 특수한 그룹화 기능이다.  
분할 함수는 불리언을 반환하므로 맵의 키 형식은 Boolean이다.  
결과적으로 그룹화 맵은 최대(참 아니면 거짓) 두개의 그룹으로 분류된다.
```
// 모든 요리를 채식요리와 채식이 아닌 요리로 분류
Map<Boolean, List<Dish>> partitionedMenu = 
                menu.stream().collect(partitioningBy(Dish::isVegetarian));
                
{false = [pork, beef, chicken],
true = [french fries, rice, season fruit]}

// 채식요리만 
List<Dish> vegetarianDishes = partitionedMenu.get(true);

// filter 사용
List<Dish> vegetarianDishes = menu.stream().filter(Dish::isVegetarian).collect(toList());
```

분할의 장점  
분할의 장점은 분할 함수가 반환하는 참, 거짓 두 가지 요소의 스트림 리스트를 모두 유지한다.  
컬렉터를 두 번째 인수로 전달할 수 있는 오버로드된 버전의 partitioningBy도 존재한다. (다수준으로 분할 가능)

숫자를 소수와 비소수로 분할하기  
```
정수 n을 인수로 받아서 2에서 n까지의 자연수를 소수와 비소수로 나누기
public boolean isPrime(int candidate) {
    int candidateRoot = (int) Math.sqrt((double) candidate);
    return IntStream.rangeClosed(2, candidateRoot)
                    .noneMatch(i -> candidate % i == 0);
}

// 제곱근 이하의 수로 제한
public Map<Boolean, List<Integer>> partitionPrimes(int n) {
    return IntStream.rangeClosed(2, n).boxed()
                    .collect(partitioningBy(this::isPrime));
}
```

참조: p223 표 6.1 Collectors클래스의 정적 팩토리 메서드

### 6.5 Collector 인터페이스
Collector 인터페이스는 리듀싱 연산(컬렉터)을 어떻게 구현할지 제공하는 메서드 집합으로 구성된다.  
Collector 인터페이스의 인터페이스 시그니처와 다섯 개의 메서드 정의   
```
public interface Collector<T, A, R> {
    Supplier<A> supplier();
    BiConsumer<A, T> accumulator();
    BinaryOperator<A> combiner();
    Function<A, R> finisher();
    Set<Characteristics> characteristics();
}
```
- T는 수집된 스트림 항목의 제네릭 형식
- A는 누적자, 즉 수집 과정에서 중간 결과를 누적하는 객체의 형식
- R은 수집 연산 결과 객체의 형식(대게 컬렉션 형식)

**Collector 인터페이스의 메서드 살펴보기**
- supplier 메서드: 새로운 결과 컨테이너 만들기  
supplier는 수집 과정에서 빈 누적자 인스턴스를 만드는 파라미터가 없는 함수
```
public Supplier<List<T>> supplier() {
	return () -> new ArrayList<T>();
}

public Supplier<List<T>> supplier() {
	return ArrayList::new;
}
```

- accumulator 메서드: 결과 컨테이너에 요소 추가하기  
스트림에서 n번째 요소를 탐색할 때, 두 인수 즉 누적자와 n번째 요소를 함수에 적용한다.
```java
public BiConsumer<List<T>, T> accumulator() {
    return (list, item) -> list.add(item);
}

public BiConsumer<List<T>, T> accumulator() {
    return List::add;
}
```

- finisher 메서드: 최종 변환 값을 결과 컨테이너로 적용하기  
  finisher 메서드는 스트림 탐색을 끝내고 누적자 객체를 최종 결과로 변환하면서 누적 과정을 끝낼 때 호출할 함수를 반환해야 한다.
```
public Function<List<T>, List<T>> finisher() {
    return Function.identity();
}
```

- combiner 메서드 : 두 결과 컨테이너 병합  
  combiner는 스트림의 서로 다른 서브파트를 병렬로 처리할 때 누적자가 이 결과를 어떻게 처리할지 정의한다. 
```java
public BinaryOperator<List<T>> combiner() {
    return (list1, list2) -> {
        list1.addAll(list2);
        return list1;
    }
}
```
- Characteristics 메서드  
  Characteristics 메서드는 컬렉터의 연산을 정의하는 Charactieristics 형식의 불변 집합을 반환한다.  
  Characteristics는 다음 세 항목의 특성을 갖는 열거형이다.  
  - UNORDERED : 리듀싱 결과는 스트림의 방문 순서나 누적 순서에 영향을 받지 않는다.
  - CONCURRENT : 다중 스레드에서 accumulator 함수를 동시에 호출할 수 있으며 이 컬렉터는 스트림의 병렬 리듀싱을 수행할 수 있다.
  - IDENTITY_FINISH : finisher 메서드가 반환하는 함수는 identity를 적용할 뿐이므로 이를 생략할 수 있다.

**응용하기**  
지금까지 살펴본 다섯 가지 메서드를 이용해 자신만의 커스텀 ToListCollector를 구현할 수 있다.
```java
public class ToListCollector<T> implements Collector<T, List<T>, List<T>> {

    @Override
    public Supplier<List<T>> supplier() {
        return ArrayList::new;
    }

    @Override
    public BiConsumer<List<T>, T> accumulator() {
        return List::add;
    }

    @Override
    public BinaryOperator<List<T>> combiner() {
        return (left, right) -> {
            left.addAll(right);
            return left;
        };
    }

    @Override
    public Function<List<T>, List<T>> finisher() {
        return Function.identity();
    }

    @Override
    public Set<Characteristics> characteristics() {
        return Collections.unmodifiableSet(EnumSet.of(
	               Collector.Characteristics.CONCURRENT,
                 Collector.Characteristics.IDENTITY_FINISH));
        }
  }

// 다음과 같이 사용할 수 있다.
List<Dish> dishes = menu.stream()
                        .collect(new ToListCollector<>());
```
