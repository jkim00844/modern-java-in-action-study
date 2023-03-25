## chapter9. 리팩터링, 테스팅, 디버깅

람다 표현식을 이용해 가독성과 유연성을 높이려면 기존 코드를 어떻게 리팩터링해야 하는 지 설명한다.

### 9.1 가독성과 유연성을 개선하는 리팩터링

코드 가독성을 개선한다는 것은 우리가 구현한 코드를 다른 사람이 쉽게 이해하고 유지보수할 수 있게 만드는 것을 의미한다.  
코드 가독성을 높이려면 코드를 문서화를 잘하고 표준 코딩 규칭을 준수하는 등의 노력을 기울여야 한다.  

리팩토링 예제 3가지
- 익명 클래스를 람다 표현식으로 리팩터링
- 람다 표현식을 메서드 참조로 리팩터링하기
- 명령형 데이터 처리를 스트림으로 리팩터링하기

1. 익명 클래스를 람다 표현식으로 리팩터링
```java
Runnable r1 = new Runnable() { // 익명클래스
    public void run() {
        System.out.println("Hello");
    }
};

// 람다표현식의 코드
Runnable r2 = () -> System.out.println("Hello");
```
모든 익명클래스를 람다 표현식으로 변환할 수 있는것은 아님.
1) 익명 클래스에서 this는 익명클래스 자신을 가리키지만 람다에서 this는 람다를 감싸는 클래스를 가르킨다.
2) 익명 클래스는 감싸고 있는 클래스의 변수를 가릴 수 있다.
하지만 람다 표현식으로는 변수를 가릴 수 없다.
```java
int a = 10;

Runnable r1 = new Runnable() {
    public void run() {
        int a = 2; // 잘 동작.
        System.out.println(a);
    }
};

Runnable r2 = () -> {
    int a = 2; // 컴파일 에러.
    System.out.println(a);
}
```
3) 익명 클래스를 람다 표현식으로 바꾸면 콘텍스트 오버로딩에 따른 모호함이 초래될 수 있다.
```java
interface Task {
    public void execute();
}
public static void doSomething(Runnable r){ r.run(); }
public static void doSomething(Task a){ r.execute(); }

// Task를 구현하는 익명클래스 전달가능.
doSomething(new Task() {
    public void execute() {
        System.out.println("Danger danger!!");
    }
});
```

```java
// doSomething(Runnable r)과 doSomething(Task a) 중 어느 것을 가르키는 지 알 수 모호하다.
doSomething(() -> System.out.println("Danger danger!!"));

//명시적 형변환으로 모호한 제거 가능.
doSomething((Task)() -> System.out.println("Danger danger!!"));
```

2. 람다 표현식을 메서드 참조로 리팩터링하기
```java
    Map<CaloricLevel, List<Dish>> dishesByCaloricLevel = menu.stream()
        .collect(
            groupingBy(dish -> {
                if(dish.getCalories() <= 400) return CaloricLevel.DIET;
                else if (dish.getCalories() <= 700) return CaloricLevel.NORMAL;
                else return CaloricLevel.FAT;
            }));

    // 람다표현식을 별도의 메서드로 추출한 후 groupingBy에 인수로 전달
    public class Dish {
        public CaloricLevel getCaloricLevel() {
            if(dish.getCalories() <= 400) return CaloricLevel.DIET;
            else if (dish.getCalories() <= 700) return CaloricLevel.NORMAL;
            else return CaloricLevel.FAT;
        }
    }
    
    Map<CaloricLevel, List<Dish>> dishesByCaloricLevel =
    menu.stream().collect(groupingBy(Dish::getCaloricLevel));
```
comparing 과 maxBy 같은 정적 헬퍼 메서드를 활용
람다 표현식보다 메서드 참조가 코드의 의도를 더 명확하게 보여준다.

```
// 비교구현에 신경써야함.
inventory.sort(
    (Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight()));

// 코드가 문제 자체를 설명.
inventory.sort(comparing(Apple::getWeight));
```
내장 컬렉터 활용. ex) summingInt
```java
int totalCalories = 
    menu.stream().map(Dish::getCalories)
                 .reduce(0, (c1, c2) -> c1 + c2);

int totalCalories = menu.stream().collect(summingInt(Dish::getCalories));
```

3. 명령형 데이터 처리를 스트림으로 리펙터링  
스트림 API를 이용하면 문제를 더 직접적으로 기술할 수 있고 쉽게 병렬화 할 수 있다.
```java
List<String> dishNames = new ArrayList<>();
for(Dish dish: menu) {
    if(dish.getCalories() > 300) {
        dishNames.add(dish.getName());
    }
}

menu.parallelStream()
    .filter(d -> d.getCalories() > 300)
    .map(Dish::getName)
    .collect(toList());
```

**코드 유연셩 개선**  
함수형 인터페이스 적용  
람다 표현식을 이용하려면 함수형 인터페이스가 필요하다.  
자주 사용하는 람다 표현식 리팩터링 패턴  
- 조건부 연기 실행
- 실행 어라운드

1. 조건부 연기 실행

```java
if(logger.isLoggable(Log.FINER)) {
    logger.finer("Problem: " + generateDiagnostic());
}
```
위 코드의 두가지 문제점
- logger의 상태가 isLoggable이라는 메서드에 의해 클라이언트 코드로 노출된다.
- 메시지를 로깅할 뗴마다 logger 객체의 상태를 매번 확인해야 한다.

메시지를 로깅하기전에 logger객체가 적절한 수준으로 설정되었는지 내부적으로 확인하는 log메서드를 사용
```java
logger.log(Level.FINER, "Problem: " + generateDiagnostic());
```
logger가 활성화 되지 않더라도 항상 로깅 메시지를 평가하게 된다는 문제점이 남아있다.  
람드를 이용해 문제를 해결할 수 있다.  
특정 조건(예제 에서는 logger수준을 finer로 설정)에서만 메시지가 생성될 수 있도록 메시지 생성 과정을 연기 할 수 있다.  
자바 8에서 제겅화는 Supplier를 인수로 갖는 오버로드된 log 메서드 활용

```java
public void log(Level level, Supplier<String> msgSupplier)

// log 메서드는 logger의 수준이 적절하게 설정되어 있을때만 인수로 넘겨진 람다를 내부적으로 실행
public void log(Level level, Supplier<String> msgSupplier) {
    if(logger.isLoggable(level)) {
        log(level, msgSupplier.get());
    }
}

logger.log(Level.FINER, () -> "Problem: " + generateDiagnostic());
```

만약 클라이언트 코드에서 객체 상태를 자주 확인하거나(예를 들면 logger의 상태)  
객체의 일부 메서드를 호출하는 상황(예를 들면 메시지 로깅)이라면   
내부적으로 객체의 상태를 확인한 다음에 메서드를 호출하도록 구현하는 것이 좋다.  

2. 실행 어라운드  
매번 같은 준비, 종료과정을 반복적으로 수행한는 코드는 람다로 변환할 수 있다.

```java

String oneLine = 
    processFile((BufferedReader b) -> b.readLine()); // 람다 전달

String twoLines = 
    processFile((BufferedReader b) -> b.readLine() + b.readLine); // 다른 람다 전달

public static String processFile(BufferedReaderProcessor p) throws IOException {
    try(BufferedReader br = 
        new BufferedReader(new FileReader("ModernJavaInAction.txt"))) {
        return p.process(br);
    }
}

public interface BufferedReaderProcessor {
    String process(BufferedReader b) throws IOException;
}
```
람다로 BufferedReader 객체의 동작을 결정할 수 있는것은 함수형 인터페이스 BufferedReaderProcessor때문이다.

### 9.2 람다로 객체지향 디자인 패턴 리팩터링하기
1. 전략 
전략 디자인 패턴
- 알고리즘을 나타내는 인터페이스(Strategy 인터페이스)  
- 다양한 알고리즘을 나타내는 한개 이상의 인터페이스 구현(ConcreatStrategyA, ConcreatStrategyB 구체적 구현클래스)
- 전략 객체를 사용하는 한개 이상의 클라이언트

```java
public interface ValidationStrategy {
    boolean execute(String s);
}

//구현 클래스 하나이상.
public class IsAllowerCase implements ValidationStrategy {
    public boolean execute(String s) {
        return s.matches("[a-z]+");
    }
}
public class IsNumeric implements ValidationStrategy {
    public boolean execute(String s) {
        return s.matches("\\d+");
    }
}

//구현 클래스를 다양한 검증 전략으로 활용.
public class Validator {
    private final ValidationStrategy strategy;
    public Validator(ValidationStrategy v) {
        this.strategy = v;
    }
    public boolean validate(String s) {
        return strategy.execute(s);
    }
}

Validator numericValidator = new Validator(new IsNumeric());
boolean b1 = numericValicator.validate("aaaa"); // false
Validaror lowerCaseValidator = new Validator(new IsAllLowerCase());
boolean b2 = lowerCaseValidator.validate("bbbb"); // true
```
람다 표현식 사용
```java
Validator numericValidator = 
    new Validator((String s) -> s.matches("[a-z]+"));
boolean b1 = nemericValidator.validate("aaaa");
Validator lowerCaseValidator = 
    new Validator((String s) -> s.matches("\\d+"));
boolean b1 = lowerCaseValidator.validate("aaaa");
```
2. 템플릿 메서드  
알고리즘의 개요를 제시한 다은ㅁ에 알고리즘의 일부를 고칠 수 있는 유연함을 제공해야 할 때  
= 이 알고리즘을 사용하고 싶은데 그대로는 안되고 조금 고쳐야하는 상황일 때  
템플릿 메서드가 적합하다.

예를 들어 고객 계좌에 보너스를 입금한다고 가정할때
```java
abstract class OnlineBanking {
    public void processCustomer(int id) {
        Customer c = Database.getCustomerWithId(id);
        makeCustomerHappy(c);
    }
    
    abstract void makeCustomerHappy(Customer c);
}
```
OnlineBanking 클래스를 상속받아 makeCustomerhappy메서드가 원하는 동작을 수행하도록 구현할 수 있다.

람다 표현식 사용
```java
public void processCustomer(int id, Cunsumer<Customer> makeCustomerHappy) {
    Customer c = Database.getCustomerWithId(id);
    makeCustomerHappy.accept(c);
}

//onlineBanking 클래스를 상속받지 않고 직접 람다를 전달해서 다양한 동작 추가할 수 있다.
new OnlineBankingLamda().processCustomer(1337, (Customer c) -> 
    System.out.println("Hello "+ c.getName()));
```

3. 옵저버  
어떤 이벤트가 발생했을때 한 객체(subject)가 다른 객체 리스트(observer)에 자동으로 알름을 보내야 하는 상황에서  
옵저버 디자인 패턴을 사용한다.

다양한 신문 매체(뉴욕타임스, 가디언, 르몽드)가 뉴스 트윗을 구독하고 있으며  
특정 키워드를 포함하는 트윗이 등록되면 알람을 받고 싶어 한다.
```java
interface Observer {
    void notify (String tweet);
}

// 트윗에 포함된 다양한 키워드에 다른 동작을 수행할 수 있는 여러 옵저버 정의.
class NYTimes implements Observer {
    public void notify(String tweet) {
        if(tweet != null && tweet.contains("money")){
            System.out.println("Breaking news in NY! " +tweet);
        }
    }
}
class Gurdian implements Obserber {
    public void notify(String tweet) {
        if(tweet != null && tweet.contains("queen")) {
            System.out.println("Yet more news from London... " +tweet);
        }
    }
}
class LeMonde implements Obserber {
    public void notify(String tweet) {
        if(tweet != null && tweet.contains("wine")) {
            System.out.println("Today cheese, wine and news! " +tweet);
        }
    }
}
```
옵저버를 사용하는 주제 구현
```java
interface Subject {
    void registerObserver(Observer o);
    void notifyObservers(String tweet);
}

// 주제는 registerObserver 메서드로 새로운 옵저버를 등록한 다음 
// notifyObserver 메서드로 트윗의 옵저버에 이를 알림
class Feed implements Subject {
    private final List<Observer> observers = new ArrayList<>();
    public void registerObserver(Observer o) {
        this.observers.add(o);
    }
    public void notifyObservers(String tweet) {
        observers.forEach(o -> o.notify(tweet));
    }
}
```
구현
```
Feed f = new Feed();
f.registerObserer(new NYTimes());
f.registerObserer(new Guardian());
f.registerObserer(new LeMonde());
f.registerObserer("The queen said her favourite book is Harry Porter");
```

람다 표현식 사용하기  
세 개의 옵저버를 명시적으로 인스턴스화하지 않고 람다 표현식을 직접 전달해서 실행할 동작을 지정할 수 있다.
```java
f.registerObserver((String tweet) -> {
    if(tweet !=null && tweet.contains("money")) {
        System.out.println("Breaking news in NY! "+tweet);
    }
})
```
항상 람다 표현식을 사용해야 할까?  
복잡한 구조의 코드를 작성해야 할 땐 람다보다 기존의 클래스 구현방식을 고수하는 것이 나을 수 있다.

4. 의무 체인  
작업처리 객체의 체인(동작 체인 등)을 만들때는 의무 체인 패턴을 사용한다.  
한 객체가 어떤 작업을 처리한 다음에 다른 객체로 결과를 전달하고 다른 객체도 해야 할 작업을 처리한 다음에 또 다른 객체로 전달하는 식이다.  
setSuccessor 다음 handle 작업 처리
```java
public abstract class ProcessingObject<T> {
    protected ProcessingObject<T> successor;
    public void setSuccessor(ProcessingObject<T> successor) {
        this.successor = successor;
    }
    public T handle(T input) {
        T r = handleWork(input);
        if(successor != null) {
            return successor.handle(r);
        }
        return r;
    }
    abstract protected T handleWork(T input);
}
```
ProcessingObject 클래스를 상속받아 handleWork 메서드를 구현하여 다양한 종류의 작업처리 객체를 만들수 있음.  
텍스트를 처리하는 예제
```java
public class HeaderTextProcessing extends ProcessingObject<String> {
    public String handleWork(String text) {
        return "From Raoul, Mario and Alan: "+text;
    }
}

public class SpellCheckerProcessing extends ProcessingObject<String> {
    public String handleWork(String text) {
        return text.replaceAll("labda", "lambda");
    }
}

// 두 작업처리 객체를 연결해서 작업 체인
ProcessingObject<String> p1 = new HeaderTextProcessing();
ProcessingObject<String> p2 = new SpellCheckerProcessing();
p1.setSuccessor(p2);
String result = p1.handle("Aren't labdas really sexy?!!"); 
System.out.println(result); // From Raoul, Mario and Alan: Aren't lambdas really sexy?!!
```

람다 표현식 사용  
작업처리 객체를 Function<String, String>, UnaryOperator<String> 형식의 인스턴스로 표현 가능.
```java
UnaryOperator<String> headerProcessing = 
    (String text) -> "From Raoul, Mario and Alan: "+text;
UnaryOperator<String> spellCheckerProcessing = 
    (String text) -> text.replaceAll("labda", "lambda");
Function<String, String> pipeline = 
    headerProcessing.andThen(spellCheckerProcessing);
String result = pipeline.apply("Aren't labda really sexy?!!");
```

5. 팩토리  
인스턴스화 로직을 클라이언트에 노출하지 않고 객체를 만들때 팩토리 디자인 패턴을 사용한다.
```java
public class ProductFactory {
    public static Product createProduct(String name) {
        switch(name) { // 모두 Product의 서브 형식.
            case "loan": return new Loan();
            case "stock": return new Stock();
            case "bond": return new Bond();
            default: throw new RuntimeException("No such product "+ name);
        }
    }
}

// 생성자와 설정을 외부로 노출하지 않고 클라이언트가 단순하게 상품을 생산할 수 있다.
Product p = ProductFactory.createProduct("loan");
```

```java
Supplier<Product> loanSupplier = Loan::new;
Loan loan = loanSupplier.get();

// 상품명을 생성자로 연결하는 Map을 만들어 재구현
final static Map<String, Supplier<Product>> map = new HashMap<>();
static {
    map.put("loan", Loan::new);
    map.put("stock", Stock::new);
    map.put("bond", Bond::new);
}

public static Product createProduct(String name) {
    Supplier<Product> p = map.get(name);
    if(p != null) return p.get();
    throw new IllegalArgumentException("No such product "+ name);
}
```

### 9.3 람다 테스팅  
개발자의 최종 업무 목표는 깔끔한 코드를 구현하는 것이 아니라 제대로 동작하는 코드를 구현하는 것이다.  
의도대로 동작하는지 확인하는 단위 테스팅을 진행해야 한다.  
예상된 결과를 도출할 것이라 단언하는 테스트 케이스를 구현하다.  

**보이는 람다 표현식의 동작 테스팅**  
람다는 익명함수이므로 테스트 코드 이름을 호출할 수 없다.  
따라서 필요시 람다를 필드에 저장해서 람다의 로직을 테스트 할 수 있다.  
(compareByXAndThenY를 이용하면 메서드 참도로 생성한 Comparator 객체에 접근 할 수 있다.)
```java
public class Point {
    public final static Comparator<Point> compareByXAndThenY =
        comparing(Point::getX).thenComparing(Point::getY);
    ...
}

@Test
public void testComparingTwoPoints() throws Exception {
    Point p1 = new Point(10, 15);
    Point p2 = new Point(10, 20);
    
    int result = Point.compareByXAndThenY.compare(p1, p2);
    assertTrue(result < 0);
}
```

**람다를 사용하는 메서드의 동작에 집중하라**  
람다 표현식을 사용하는 메서드의 동작을 테스트 함으로써 람다를 공개하지 않으면서도 람다 표현식을 검증할 수 있다.

```java
public static List<Point> moveAllPointsRightBy(List<Point> points, int a) {
    return points.stream()
                 .map(p -> new Point(p.getX() + a, p.getY()))
                 .collect(toList());
}

// p -> new Point(p.getX() + a, p.getY()를 테스트하는 부분이 없다.
// moveAllPointsRightBy 메서드를 테스트하는 부분이 없다.

@Test
public void testMoveAllPointsRightBy() throws Exception {
    List<Point> points =
    Arrays.asList(new Point(5, 5), new Point(10,5));
    List<Point> expectedPoints =
    Arrays.asList(new Point(15, 5), new Point(20,5));
    List<Point> newPoints = Point.moveAllPointsRightBy(points, 10);
    assertEquals(expectedPoints, newPoints);
}
```

**복잡한 람다를 개별 메서드로 분할하기**  
테스트 코드에서 람다 표현식을 참조 할 수 없다.  
복잡한 람다 표현식은 람다를 메서드 참조로 바꿔서 테스트하자.  

**고차원 함수 테스팅**  
함수를 인수로 받거나 다른함수를 반환하는 메서드는 사용하기 어렵다.  
메서드가 람다를 인수로 받는다면 다른 람다로 메서드 동작 테스트할 수 있다.  
프레디케이트로 만든 filter 메서드 활용
```java
@Test
public void testFilter() throws Exception {
    List<Integer> numbers = Arrays.asList(1,2,3,4);
    List<Integer> even = filter(number, i -> i % 2 == 0);
    List<Integer> smallerThanThree = filter(number, i-> i<3);
    assertEquals(Arrays.asList(2,4), even);
    assertEquals(Arrays.asList(1,2), smallerThanThree);
}
```

### 9.4 디버깅
문제가 발 생한 코드를 디버깅 할 때 개발자가 확인해아하는 두가지
- 스택 트레이스
- 로깅

1. 스택 트레이스  
예외 발생으로 프로그램 실행이 갑자기 중단되었다면 먼저 어디에서 멈췄고 어떻게 멈추게 되었는지 살펴봐야 한다.  
스택 프레임에서 이 정보를 확인 할 수 있다.  
하지만 람다 표현식은 이름이 없기 때문에 조금 복잡한 스택 트레이스가 생성된다.  
메서드 참조를 사용해도 스택 트레이스에는 메서드 명이 나타나지 않는다.


2. 정보 로깅  
스트림 파이프라인에 적용된 각각의 연산(map, filter, limit)이 어떤 결과를 도출하는지 확인하고 싶을  때  
peek 스트림 연산을 활용
```java
numbers.stream()
  .map(x -> x + 17)
  .filter(x -> x % 2 == 0)
  .limit(3)
  .forEach(System.out::println);

numbers.stream()
    .peek(x -> System.out.println(x))
    .map(x -> x + 17)
    .peek(x -> System.out.println(x))
    .filter(x -> x % 2 == 0)
    .peek(x -> System.out.println(x))
    .limit(3)
    .peek(x -> System.out.println(x))
    .collect(toList());
```
