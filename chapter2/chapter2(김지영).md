## chapter2. 동작 파라미터화 코드 전달하기

어떤 상황에서 일을 하든 소비자의 요구사항은 항상 바뀐다.  
동작 파라미터화를 이용하면 자주 바뀌는 요구사항에 효과적으로 대응할 수 있다.  
동작 파라미터화란 아직은 어떻게 실행할 것이지 결정하지 않은 코드 블록을 의미한다.  
이 코드 블록은 나중에 프로그램에서 호출한다. 즉, 코드 블록의 실행은 나중으로 미뤄진다.

예를 들어 나중에 실행될 메서드의 인수로 코드 블록을 전달할 수 있다.  
결과적으로 코드 블록에 따라 메서드의 동작이 파라미터화 된다.  
예를 들어 컬렉션을 처리할 때 다음과 같은 메서드를 구현한다 가정하자.
- 리스트의 모든 요소에 대해서 ‘어떤 동작'을 수행할 수 있음
- 리스트 관련 작업을 끝낸 다음에 ‘어떤 다른 동작'을 수행할 수 있음
- 에러가 발생하면 ‘정해진 어떤 다른 동작'을 수행할 수 있음

### 1. 변화하는 요구사항에 대응하기
**녹색 사과 필터링**
```
    public static List<Apple> filterGreenApples(List<Apple> inventory) {
        List<Apple> result = new ArrayList<>(); // 사과 누적 리스트
        for (Apple apple : inventory) {
            if (GREEN.equals(apple.getColor()) { // 녹색 사과만 선택
                result.add(people);
            }
        }
        return result;
    }
```
요구사항 추가 : 빨간 사과도 필터링  
→ 크게 고민하지 않는다면 메서드를 복사해서 filterRedApples라는 새로운 메서드를 만들고,  
    if 문의 조건을 빨간 사과로 바꾸는 방법이 있을 것임.  
→ 하지만 나중에 더 다양한 색으로 필터링한다면 변화에 적절하게 대응할 수 없다.

이러한 상황에서는 다음과 같은 좋은 규칙이 있다.  
→ 거의 비슷한 코드가 반복 존재한다면 그 코드를 추상화한다.

**색을 파라미터화**  
색을 파라미터화할 수 있도록 메서드에 파라미터를 추가하면 변화하는 요구사항에 좀 더 유연하게 대응하는 코드를 만들 수 있다.
```
    public static List<Apple> filterApplesByColor(List<Apple> inventory, Color color) {
        List<Apple> result = new ArrayList<>();
        for (Apple apple : inventory) {
            if (apple.getColor().equals(color)) {
                result.add(people);
            }
        }
        return result;
    }

    // 다음과 같이 호출할 수 있게 되었다.
    List<Apple> greenApples = filterApplesByColor(inventory, GREEN);
    List<Apple> redApples = filterApplesByColor(inventory, RED);
```
또 다른 요구사항 추가 :  가벼운 사과(150g 미만)와 무거운 사과(150g 이상)로 구분  
색을 파라미터화 한것처럼 무게도 파라미터화
```
    public static List<Apple> filterApplesByWeight(List<Apple> inventory, int weight) {
        List<Apple> result = new ArrayList<>();
        for (Apple apple : inventory) {
            if (apple.getWeight() >= weight) {
                result.add(people);
            }
        }
        return result;
    }
```
하지만 구현 코드를 자세히 보면 목록을 검색하고, 각 사과에 필터링 조건을 적용하는 부분의 코드가 색 필터링 코드와 대부분 중복된다.  
이는 소프트웨어 공학의 DRY(don’t repeat yourself - 같은 것을 반복하지 말 것) 원칙을 어기는 것이다.  

그래서 색과 무게를 filter라는 메서드로 합치는 방법도 있다.

**가능한 모든 속성으로 필터링**
```
    public static List<Apple> filterApples(List<Apple> inventory,
        Color color, int weight, boolean flag) {
        List<Apple> result = new ArrayList<>();
        for (Apple apple: inventory) {
            if((flag && apple.getColor().equals(color)) ||
                (!flag && apple.getWeight() > weight)){
                result.add(apple);
            }
        }
        return result;
    }

    // 위 메서드를 다음처럼 사용할 수 잇다.
    List<Apple> greenApples = filterApples(inventory, GREEN, 0, true);
    List<Apple> heavyApples = filterApples(inventory, null, 150, false);
```
다음은 모든 속성을 메서드 파라미터로 추가한 모습이다.  
(색과 무게 중) 어떤 기준으로 사과를 필터링할지 구분하기 위해 flag를 추가하였다. 

하지만 이는
형편 없는 코드다.  
true와 false가 무엇을 의미하는지 명확하지 않다.  
요구사항이 바뀔 때 유연하게 대응할 수 없다.  
(사과의 크기, 모양, 출하지 등으로 필터링해야한다면? 심지어 녹색 사과 중에 무거운 사과를 필터링하고 싶다면?)

### 2. 동적 파라미터화 
앞선 예제로 파라미터를 추가하는 방법이 아닌 변화하는 요구사항에 좀 더 유연하게 대응할 수 있는 방법이 절실하다는 것을 확인했다.  
사과의 어떤 속성에 기초해서 불리언 값(사과가 녹색인가? 150g 이상인가?)을 반환하는 방법을 통해 개선을 할 수 있을 것 같다.  
참 또는 거짓을 반환하는 함수를 프레디케이트라고 한다.


먼저, 선택 조건을 결정하는 인터페이스를 정의한다.   
그 다음 다양한 선택 조건을 대표하는 여러 버전의 ApplePredicate를 정의할 수 있을 것이다.
```
    public interface ApplePredicate {
        boolean test (Apple apple);
    }

    // 무거운 사과만 선택
    public class AppleHeavyWeightPredicate implements AppplePredicate {
        public boolean test(Apple apple) {
            return apple.getWeight() > 150;
        }
    }

    // 녹색 사과만 선택
    public class AppleColorPredicate implements AppplePredicate {
        public boolean test(Apple apple) {
            return GREEN.equals(apple.getColor());
        }
    }
```
위와 같은 패턴을 전략패턴이라고도 한다.  
전략 디자인 패턴은 각 알고리즘(전략)을 캡슐화하는 알고리즘 패밀리를 정의해둔 다음에 런타임에 알고리즘(전략)을 선택하는 기법이다.  
ApplePredicate - 알고리즘 패밀리, AppleHeavyWeightPredicate,AppleColorPredicate  - 전략  
사과를 조건에 맞게 필터링하는 filterApples() 메서드에 ApplePredicate의 구현체를 파라미터로 받아 사과를 필터링하게 한다.  
이렇게 하면 filterApples 메서드 내부에서 컬렉션을 반복하는 로직과 컬렉션의 각 요소에 적용할 동작을 분리할 수 있다는 점에서 소프트웨어 엔지니어링적으로 큰 이득을 얻는다.

```
    public static List<Apple> filterApples(List<Apple> inventory, ApplePredicate p) {
        List<Apple> result = new ArrayList<>();
        for (Apple apple : inventory) {
            if (p.test(apple)) {
                result.add(apple);
            }
        }
        return result;
    }
```
150그램이 넘는 빨간사과를 필터링 하려면?
```
    public class AppleRedAndHeavyPredicate implements ApplePredicate {
        public boolean test(Apple apple) {
            return RED.equals(apple.getColor()) && apple.weight() > 150;
        }
    }

    List<Apple> redAndHeavyApples = filterApples(inventory, new AppleRedAndHeavyPredicate());
```

> 한 개의 파라미터, 다양한 동작
> 컬렉션 탐색 로직과 각 항목에 적용할 동작을 분리할 수 있다는 것이 동작 파라미터화의 강점이다.  
> 한 메서드가 다른 동작을 수행하도록 재활용할 수 있다.   
> 따라서 유연한 API를 만들 때 동작 파라미터화가 중요한 역할을 한다.

### 3. 복잡한 과정 간소화
동작을 추상화해서 변화하는 요구사항에 대응할 수 있는 코드를 구현하는 방법을 살펴봤다.  
하지만 여러 클래스를 구현해서 인스턴스화하는 과정이 조금은 거추장스럽게 느껴질 수 있다.  
로직과 관련 없는 코드도 많이 추가되기 때문에 시간 낭비를 초래한다.  
자바는 클래스의 선언과 인스턴스화를 동시에 수행할 수 있도록 익명 클래스(anonymous class)라는 기법을 제공한다.

**익명 클래스**
익명 클래스는 말 그대로 이름이 없는 클래스다.  
익명 클래스를 이용하면 클래스 선언과 인스턴스화를 동시에 사용할 수 있다.
익명클래스로 메소드 동작을 직접 파라미터화 할 수 있다.
```
    List<Apple> redApples = filterApples(inventory, new ApplePredicate() {
        public boolean test(Apple apple) {
            return RED.equals(apple.getColor());
        }
    });
```
익명 클래스로도 아직 부족한 점이 있다.  
첫째, 굵은 글씨로 표현한 부분에서 알 수 있는 것처럼 익명 클래스는 여전히 많은 공간을 차지한다.  
둘째, 많은 프로그래머가 익명 클래스의 사용에 익숙하지 않다.  

자바 8의 람다 표현식을 이용해서 위 예제 코드를 다음처럼 간단하게 재구현할 수 있다.
```
    List<Apple> result = filterApples(inventoruy, (Apple apple) -> RED.equals(apple.getColor()));
```

리스트 형식으로 추상화
이제 바나나, 오렌지, 정수, 문자열 등의 리스트에 필터 메서드를 사용할 수 있다.  
람다 표현식으로 다음과 같이 코드를 작성할 수 있다.  
```
    public interface Predicate<T> {
        boolean test(T t);
    }

    public static <T> List<T> filter(List<T> list, Predicate<T> p) {
        List<T> result = new ArrayList<>();
        for(T e : list) {
            if(p.test(e)) {
                result.add(e);
            }
        }
        return result;
    }
    
    List<Apple> redApples = filter(inventory, (Apple apple) -> RED.equals(apple.getColor()));
    List<Apple> evenNumbers = filter(numbers, (Integer i) -> i % 2 == 0);
```
### 4. 실전 예제
자바 API의 많은 메서드를 다양한 동작으로 파라미터화할 수 있다.
- Comparator로 정렬하기
- Runnable로 코드 블록 실행하기
- Callable을 결과로 반환하기
- GUI 이벤트 처리하기

