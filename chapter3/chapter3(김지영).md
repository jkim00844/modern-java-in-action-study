## chapter3. 람다 표현식

람다 표현식을 어떻게 만드는지, 어떻게 사용하는지, 어떻게 코드를 간결하게 만들 수 있는지 설명한다.  
또한 자바 8 API에 추가된 중요한 인터페이스와 형식 추론 등의 기능도 확인한다.  
마지막으로 람다 표현식과 함께 위력을 발휘하는 새로운 기능인 메서드 참조를 설명한다.


### 3.1 람다란 무엇인가?
람다 표현식은 메서드로 전달할 수 있는 익명 함수를 단순화한 것이라고 할 수 있다. 람다의 특징은 다음과 같다.
- 익명  
보통의 메서드와 달리 이름이 없다. 구현해야 할 코드에 대한 걱정거리가 줄어든다.
- 함수   
람다는 메서드처럼 특정 클래스에 종속되지 않으므로 함수라고 부른다. 하지만 메서드처럼 파라미터 리스트, 바디, 반환 형식, 가능한 예외 리스트를 표함한다.
- 전달  
람다 표현식을 메서드 인수로 전달하거나 변수로 저장할 수 있다.
- 간결성  
익명 클래스처럼 많은 자질구레한 코드를 구현할 필요가 없다.

람다를 이용해 간결하고 유연한  방식으로 코드를 전달할 수 있다.

람다는 세 부분으로 이루어진다.

`(Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight());`  
- 파라미터 리스트 : (Apple a1, Apple a2), Comparator의 compare 메서드 파라미터
- 화살표(->) : 화살표는 람다의 파라미터 리스트와 바디를 구분한다.
- 람다 바디 : 람다의 반환값에 해당하는 표현식이다.

람다 예제
- 불리언 표현식: (List<String> list) -> list.isEmpty()  
- 객체 생성: () -> new Apple(10)  
- 객체에서 소비:	(Apple a) -> { System.out.println(a.getWeight());}  
- 객체에서 선택/추출:	(String s) -> s.length()  
- 두 값을 조합: (int a, int b) -> a * b  
- 두 객체 비교: (Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight())  

### 3.2 어디에, 어떻게 람다를 사용할까?
**함수형 인터페이스**  
함수형 인터페이스라는 문맥에서 람다 표현식을 사용할 수 있다.  
함수형 인터페이스는 정확히 하나의 추상 메서드를 지정하는 인터페이스다.  
자바 API의 함수 인터페이스로 Comparator, Runable이 있다.

함수형 인터페이스로 뭘 할 수 있을까?  
람다 표현식으로 함수형 인터페이스의 추상 메서드 구현을 직접 전달할 수 있으므로  
전체 표현식을 함수형 인터페이스의 인스턴스로 취급(기술적으로 따지면 함수형 인터페이스를 구현한 클래스의 인스턴스)할 수 있다.  
함수형 인터페이스보다는 덜 깔끔하지만 익명 내부 클래스로도 같은 기능도 구현할 수 있다.

다음 예제는 Runnable이 오직 하나의 추상 메서드 run을 정의하는 함수형 인터페이스므로 올바른 코드다.
```=
// 람다 사용 
Runnable r1 = () -> System.out.println("Hello world 1");

// 익명 클래스 사용
Runnable r2 = new Runnable() {
	public void run() {
		System.out.println("Hello World 2");
	}
}

public static void process(Runnable r) {
	r.run();
}
process(r1);
process(r2);
process(() -> System.out.println("Hello Wordl 3")); // 람다 표현식으로 직접 전달도 가능
```
**함수 디스크립터**  
함수형 인터페이스의 추상 메서드 시그니처(signature)는 람다 표현식의 시그니처를 가리킨다.  
람다 표현식의 시그니처를 서술하는 메서드를 함수 디스트립터(function descriptor)라고 부른다.  
메서드 시그니처란: 자바 프로그래밍 언어에서 메서드 시그니처는 메서드 명과 파라미터의 순서, 타입, 개수를 의미

예를 들어 Runnable 인터페이스의 유일한 추상 메서드 run은 인수와 반환값이 없으므로(void 반환)  
Runnable 인터페이스는 인수와 반환값이 없는 시그니처로 생각할 수 있다.   
이는 () -> void 와 같은 표기법으로 나타낼 수 있다.  
두 개의 Apple을 인수로 받아 int를 반환하는 함수는 (Apple, Apple) -> int 로 표현한다.  

왜 함수형 인터페이스를 인수로 받는 메서드에만 람다 표현식을 사용할 수 있을까?
언어 설계자들은 언어를 더 복잡하게 만들지 않는 방법을 선택했다.  
대부분의 자바 프로그래머가 하나의 추상 메서드를 갖는 인터페이스에 이미 익숙하다.

**@FunctionalInterface**  
함수형 인터페이스임을 가리키는 어노테이션  
@FunctionalInterface로 인터페이스를 선언했지만, 실제로 함수형 인터페이스가 아니면 컴파일러 에러가 발생  

### 3.3 람다 활용 : 실행 어라운드 패턴
자원처리(예를 들면 데이터베이스의 파일 처리)에 사용되는 순환패턴은   
자원을 열고 처리한 다음에 자원을 닫는 순서로 이루어진다.   
자원을 처리하는 코드를 설정(setup)과 정리(cleanup) 두 과정이 둘러싸는 형태를 실행 어라운드 패턴(execute around pattern)이라고 부른다.

파일에서 한 행을 읽는 코드 
```
    public String processFile() throws IOException {
        try (BufferedReader br = new BufferedReader(new FileReader("data.txt"))){
            return br.readLine();
        }
    }
```

1단계 : 동작 파라미터화를 기억하라  
기존의 설정, 정리 과정은 재사용하고 processFile 메서드만 다른 동작을 수행하도록 명령할 수 있다면 좋을 것이다.  
processFile 메서드가 한 번에 두 행을 읽게 하려면 코드를 어떻게 고쳐야 할까?  
우선 BufferedReader를 인수로 받아서 String을 반환하는 람다가 필요하다.  
다음은 BufferedReader에서 두 행을 출력하는 코드다.

```
String result = processFile((BuffredReader br) -> br.readLine() + br.readLine());
```

2단계 : 함수형 인터페이스를 이용해서 동작 전달  
함수형 인터페이스 자리에 람다를 사용할 수 있다.  
따라서 BufferedReader -> String과 IOException을 던질 수 있는 시그니처와 일치하는 함수형 인터페이스를 만들어야 한다. 
```
    @FuntionalInterface
    public interface BufferedReaderProcessor {
        String process(BuffredReader b) throws IOException;
    }
    
    public String processFile(BufferedReaderProcessor p) throws IOException {
	...
}
```
3단계 : 동작 실행  
BuffredReaderProcessor에 정의된 process 메서드의 시그니처(BufferedReader -> String)와 일치하는 람다를 전달할 수 있다.
```
    public String processFile(BufferedReaderProcessor p) throws IOException {
        try (BuffredReader br = new BuffredReader(new FileReader("data.txt"))) {
            return p.process(br);
        }
    }
```

4단계 : 람다 전달  
람다를 이용해서 다양한 동작을 processFile 메서드로 전달할 수 있다.
```
// 한 행을 처리하는 코드
String oneLine = processFile((BufferedReader br) -> br.readLine());

// 두 행을 처리하는 코드
Strine twoLines = processFile((BuffredReader br) -> br.readLine() + br.readLine());
```
### 3.4 함수형 인터페이스 사용
함수형 인터페이스의 추상 메서드 시그니처를 함수 디스크립터라고 한다.
다양한 람다 표현식을 사용하려면 공통의 함수 디스크립터를 기술하는 함수형 인터페이스 집합이 필요하다.  
이미 자바 API는 Comparable, Runnable, Callable 등의 다양한 함수형 인터페이스를 포함하고 있다.  

자바 8 라이브러리 설계자들은 java.util.function 패키지로 여러 가지 새로운 함수형 인터페이스를 제공한다.  
**Predicate**
String 객체가 Empty가 아닌 경우를 필터링하는 예제
```
@FunctionalInterface
public interface Predicate<T> {
	boolean test(T t);
}

public <T> LisT<T> filter(List<T> list, Predicate<T> p) {
	List<T> results = new ArrayList<>();
	for(T t: list) {
		if(p.test(t)) {
			results.add(t);
		}
	}
}

Predicate<String> nonEmptyStringPredicate = (String) -> !s.isEmpty();
List<String> nonEmpty= filter(listOfStrings, nonEmptyStringPredicate);
```
**Consumer**  
제네릭 형식 T 객체를 받아서 void를 반환하는 accept라는 추상 메서드를 정의한다.  
T 형식의 객체를 인수로 받아서 어떤 동작을 수행하고 싶을 때 Consumer 인터페이스를 사용할 수 있다.  
forEach와 람다를 이용해서 리스트의 모든 항목을 출력하는 예제
``` 
@FunctionalInterface
public interface Consumer<T> {
	void accept(T t);
}

public <T> void forEach(List<T> list, Consumer<T> c) {
	for (T t: list) {
		c.accept(t);
	}
}

forEach(
	Arrays.asList(1, 2, 3, 4, 5),
	(Integer i) -> System.out.println(i)
);
```
**Function**  
제네릭 형식 T를 인수로 받아서 제네릭 형식 R 객체를 반환하는 추상 메서드 apply를 정의한다.  
입력을 출력으로 매핑하는 람다를 정의할 때 Function 인터페이스를 활용할 수 있다.  
(사과의 무게 정보를 추출하거나, 문자열을 길이와 매핑)
 
String 리스트를 인수로 받아 각 String의 길이를 포함하는 Integer 리스트로 변환하는 map 메서드를 정의하는 예제
```
@FunctionalInterface
public interface Function<T, R> {
	R apply(T t);
}

public <T, R> List<R> map(List<T> list, Function<T, R> f) {
	List<R> result = new ArrayList<>();
	for (T t : list) {
		result.add(f.apply(t));
	}
	return result;
}

List<Integer> l = map(
	Arrays.asList("lambdas", "in", "action"),
	(String s) -> s.length()
);
```

**기본형 특화**  
자바의 모든 형식은 참조형(reference type) 아니면 기본형(primitive type)에 해당한다.  
하지만 제네릭 내부 구현상 제네릭 파라미터에는 참조형만 사용할 수 있다.
자바에서는 기본형을 참조형으로 변환하는 기능을 박싱(boxing)이라고 하고,  
반대로 참조형을 기본형으로 변환하는 동작을 언박싱(unboxing)이라고 한다.  
또한, 박싱과 언박싱이 자동으로 이루어지는 기능을 오토 박싱(autoboxing)이라고 한다.

```
List<Integer> list = new ArrayList<>();
for (int i = 300; i < 400; i++) {
	list.add(i);
}
```
위 코드는 int가 Integer로 오토박싱되는 코드다. 이런 변환 과정은 비용이 소모된다.  
싱한 값은 메모리를 더 소비하여 기본형을 가져올 때도 메모리를 탐색하는 과정이 필요하다.  

자바 8에서는 기본형을 입출력으로 사용하는 상황에서 오토박싱 동작을 피할 수 있도록 특별한 버전의 함수형 인터페이스를 제공한다.
```
IntPredicate<Integer> evenNumbers = (int i) -> i % 2 == 0;
evenNumbers.test(1000); -> 박싱 없음

Predicate<Integer> evenNumbers = (Integer i) -> i % 2 == 0;
evenNumbers.test(1000); -> 박싱 
```

**예외, 람다, 함수형 인터페이스의 관계**  
함수형 인터페이스는 확인된 예외를 던지는 동작을 허용하지 않는다.  
예외를 던지는 람다 표현식을 만드려면    
확인된 예외를 선언하는 함수 인터페이스를 직접 정의하거나  
람다를 try/catch 블록으로 감싸야 한다.  

```
    @FunctionalInterface 
    public interface BufferedReaderProcessor { 
        String process(BufferedReader b) throws IOException;
    }
```
```
    Function<BufferedReader, String> f = (BufferedReader b) -> { 
        try { 
            return b.readLine(); 
        } catch (IOException e) { 
            throw new RuntimeException(e);
        } 
    };
```

### 3.5 형식 검사, 형식 추론, 제약
람다로 함수형 인터페이스의 인스턴스를 만들 수 있다고 했는데  
람다 표현식 자체에는 람다가 어떤 함수형 인터페이스를 구현하는지 정보가 포함되어 있지 않다.  
따라서 람다 표현식을 더 제대로 이해하려면 람다의 실제 형식을 파악해야 한다.  

**형식검사**  
람다가 사용되는 콘텍스트(context)를 이용해서 람다의 형식을 추론할 수 있다.  
어떤 콘텍스트(람다가 전달될 메서드 파라미터나 람다가 할당되는 변수 등)에서 기대되는 람다 표현의 형식을 대상 형식(target type)이라고 부른다.

`List<Apple> heavierThan150g =
filter(inventory, (Apple apple) -> apple.getWeight() > 150);`

다음과 같은 순서로 형식 확인 과정이 진행된다.
1. filter 메서드의 선언을 확인
2. filter 메서드는 두 번째 파라미터로 Predicate<Apple> 형식(대상 형식)을 기대한다.
3. Predicate<Apple>은 test라는 한 개의 추상 메서드를 정의하는 함수형 인터페이스다.
4. test 메서드는 Apple을 받아 boolean을 반환하는 함수 디스크립터를 묘사한다. 
5. filter 메서드로 전달된 인수는 이와 같은 요구사항을 만족해야한다. (Apple → boolean이므로 람다의 시그니처와 일치)

대상 형식이라는 특징 때문에 같은 람다 표현식이라도 호환되는 추상 메서드를 가진 다른 함수형 인터페이스로 사용될 수 있다.  
예를 들어 Callable과 PriviligedAction 인터페이스는 인수를 받지 않고 제네릭 형식 T를 반환하는 함수를 정의한다.
```
    Callable<Integer> c = () -> 42;  // 유효한 코드
    PriviligedActio<Integer> p = () -> 42; // 유효한 코드
```

**형식 추론**  
대상 형식이라는 특징 때문에 컴파일러는 람다의 시그니처도 추론할 수 있다.  
즉, 컴파일러는 람다 표현식의 파라미터 형식에 접근할 수 있으므로 람다 문법에서 이를 생략할 수 있다.  

```
// 형식을 추론하지 않음
Comparator<Apple> c = (Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight());
// 형식을 추론
Comparator<Apple> c = (a1, a2) -> a1.getWeight().compareTo(a2.getWeight());
```

**지역 변수 사용**  
람다 표현식에서는 익명 함수가 하는 것처럼 자유 변수(free variable, 파라미터로 넘겨진 변수가 아닌 외부에서 정의된 변수)를 활용할 수 있다.  
이와 같은 동작을 람다 캡처링(capturing lambda)이라고 부른다.

```
// portNumber 변수를 캡처하는 람다 예제
int portNumber = 1337;
Runnable r = () -> System.out.println(portNumber);
```

람다는 인스턴스 변수와 정적 변수를 자유롭게 캡처할 수 있지만,   
지역 변수는 명시적으로 final로 선언되어 있거나 실질적으로 final로 선언된 변수와 똑같이 사용되어야 한다.  
즉, 람다 표현식은 한 번만 할당할 수 있는 지역 변수를 캡처할 수 있다.
```
// 에러: portNumber는 한번만 할당가능
int portNumber = 1337;
Runnable r = () -> System.out.println(portNumber);
portNumber = 31337;
```

지역 변수 제약
인스턴스 변수는 힙에 저장, 지역 변수는 스택에 위치한다.  
스택은 스레드마다 고유한 영역이므로 다른 스레드에서 접근할 수 없다.  
람다에서 지역 변수에 바로 접근할 수 있다는 가정하에  
람다가 스레드에서 실행된다면 변수를 할당한 스레드가 사라져 변수 할당이 해제되었는데도  
람다를 실행하는 스레드에서는 해당 변수에 접근하려 할 수 있다.

따라서 자바 구현에서는 변수에 접근을 허용하는 것이 아니라 자유 지역 변수의 복사본을 제공하고  
복사본은 값이 바뀌지 않아야 하므로 지역변수에서는 한번만 값을 할당해야한다는 제약이 생겼다.  

### 3.6 메서드 참조
메서드 참조를 이용하면 기존의 메서드 정의를 재활용해서 람다처럼 전달할 수 있다.  

```
    // 기존의 람다 표현식
    inventory.sort((Apple a1, Apple a2) 
        -> a1.getWeight().compareTo(a2.getWeight()));
    
    // 메서드 참조와 java.util.Comparator.comparing을 활용한 코드
    inventory.sort(comparing(Apple::getWeight));
```

메서드 참조를 이용하면 기존 메서드 구현으로 람다 표현식을 만들 수 있다.  
이때 명시적으로 메서드명을 참조함으로써 가독성을 높일 수 있다.

**같은 람다, 다른 함수형 인터페이스**

메서드 참조 3가지
- 정적 메서드 참조   
Integer.parseInt() -> Integer::parseInt
- 다양한 형식의 인스턴스 메서드 참조   
String의 length() -> String::length
- 기존 객체의 인스턴스 메서드 참조  
(Transaction) expensiveTransaction.getValue() -> expensiveTransaction::getValue  


생성자 참조  
ClassName::new 처럼 클래스명과 new 키워드를 이용해서 기존 생성자의 참조를 만들 수 있다.
```
Supplier<Apple> c1 = Apple::new;
Apple a1 = c1.get();
```

### 3.7 람다, 메서드 참조 활용하기
사과 리스트를 다양한 정렬 기법으로 정렬하기 

**1단계: 코드 전달**  
sort에 전달할 정렬 전략을 가진 Comparator 구현 클래스
```
public class AppleComparator implements Comparator<Apple> {
	public int compare(Apple a1, Apple a2) {
		return a1.getWeight().compareTo(a2.getWeight());
	}
}

inventory.sort(new AppleComparator());
```
**2단계: 익명 클래스 사용**  
한 번만 사용할 Comparator라면, 구현하는 것보다 익명 클래스를 이용하는 것이 더 좋다.
```
    inventory.sort(new Comparator<Apple>() {
        public int compare(Apple a1, Apple a2) {
            return a1.getWeight().compareTo(a2.getWeight());
        }
    });
```

**3단계: 람다 표현식 사용**
```
    inventory.sort((Apple a1, Apple a2) -> 
        a1.getWeight().compareTo(a2.getWeight()
    );
```
자바 컴파일러는 람다 표현식이 사용된 콘텍스트를 활용해서 람다의 파라미터 형식을 추론할 수 있다.
```
inventory.sort((a1, a2) -> a1.getWeight().compareTo(a2.getWeight());
```

이 코드의 가독성을 더 향상시킬 수 없을까?  
Comparator는 Comparable 키를 추출해서 Comparator 객체로 만드는 Function 함수를 인수로 받는 정적 메서드 comparing을 포함한다.  
람다 표현식은 사과를 비교하는데 사용할 키를 어떻게 추출할 것인지 지정하는 한개의 인수만 포함한다.
```
Comparator<Apple> c = Comparator.comparing((Apple a) -> a.getWeight());
```

**4단계: 메서드 참조 사용**
```
inventory.sort(comparing(Apple::getWeight));
```

최적의 코드를 만들었다.
코드 자체로 ‘Apple을 weight별로 비교해서 inventory를 sort하라'는 의미를 전달할 수 있게 되었다.  

### 3.8 람다 표현식을 조합할 수 있는 유용한 메서드
Comparator 조합  
- comapring()  
`Comparator<Apple> = Comparator.comparing(Apple::getWeight);`  
- 역정렬: reversed()  
`Comparator<Apple> = Comparator.comparing(Apple::getWeight).reversed();`

Predicate 조합  
- negate()  
빨간색이 아닌 사과  
`Predicate<Apple> notRedApple = redApple.negate();`
- and(), or()  
빨간색이면서 무거운 사과  
`Predicate<Apple> redAndHeavyApple = redApple.and(apple -> apple.getWeight() > 150);`

Function 조합
- andThen()
```
  Function<Integer, Integer> f = x -> x + 1;
  Function<Integer, Integer> g = x -> x * 2;
  Function<Integer, Integer> h = f.andThen(g); // h(x) = g(f(x))
  int result = h.apply(1) // result = 4
```
- compose()
```
    Function<Integer, Integer> f = x -> x + 1;
    Function<Integer, Integer> g = x -> x * 2;
    Function<Integer, Integer> h = f.compose(g); // h(x) = f(g(x))
    int result = h.apply(1) // result = 3
```