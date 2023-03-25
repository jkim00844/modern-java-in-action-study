## chapter10. 람다를 이용한 도메인 전용 언어

애플리케이션 핵심 비즈니스를 모델링하는 소프트웨어 영역에서 읽기 쉽고, 이해하기 쉬운 코드는 특히 중요하다.

도메인 전용 언어(DSL)로 애플리케이션의 비즈니스 로직을 표현함로써  
개발팀과 도메인 전문가가 공유하고 이해할 수 있는 코드를 구현할 수 있다.  
DSL은 특정 도메인을 대상으로 만들어진 특수 프로그래밍 언어이다.  
ex) 메이븐, 엔트  

### 10.1 도메인 전용 언어
DSL이란?  
특정 비즈니스 도메인을 인터페이스로 만든 API
DSL의 역할
- 의사 소통의 왕 : 프로그래머가 아닌 사람도 이해할 수 있어야 한다.
- 한 번 코드를 구현하지만 여러 번 읽는다 : 가독성은 유지 보수의 핵심이다.
DSL의 장점
- 간결함
- 가독성
- 유지보수
- 높은 수준의 추상화
- 집중 : 도메인 규칙을 표현할 목적으로 설계되어 프로그래머가 특정 코드에 집중할 수 있다.
- 관심사 분리 : 애플리케이션의 인프라구조와 관련된 문제와 독립적으로 비즈니스 관련 코드에 집중하기 용이.

DSL의 단점
- DSL의 설계의 어려움 : 간결하게 제한적인 언어에 도메인 지식을 담는 것이 쉽지 않음.
- 개발 비용 : 초기작업의 어려움.
- 추가 우회 계층
- 새로 배워야 하는 언어
- 호스팅 언어 한계 : 일부 자바와 같은 범용 프로그램밍 언어로는 사용자 친화적인 DSL을 만들기가 힘들다.

**JVM에서 이용할 수있는 다른 DSL 해결책**  
내부 DSL  
순수 자바 코드 같은 기존 호스팅 언어를 기반으로 구현  
유연성이 떨어지기 때문에 간단하고 표현력 있는 DSL을 만드는데 한계가 있었지만  
람다 표현식이 등장하여 어느정도 해결 되었다.  

다중 DSL  
JVM으로 인해 내부 DSL와 외부 DSL의 중간 카테고리에 해당  
자바보다 제약이 줄고 간편한 문법을 지향하도록 설계된 언어(코틀린, 실론)들로 인해  
간결한 DSL을 만들 수 있게 되었다.   

외부 DSL
호스팅 언어와는 독립적으로 자체 문법을 가짐.
쉽게 만들 수 없지만 무한한 유연성을 가진다.
하지만 DSL와 호스트 언어 사이에 인공 계층이 생긴다.

### 10.2 최신 자바 API의 작은 DSL
자바의 새로은 기능의 장점을 적용한 첫 API는 네이티브 자바 API 자신이다.

**스트림 API는 컬렉션을 조작하는 DSL**  
Stream 인터페이스는 네이티브 자바 API 에 작은 내부 DSL을 적용한 좋은 예다.  
컬렉션의 항목을 필터, 정렬, 변환, 그룹화, 조작하는 작지만 강력한 DSL 이다.  
Stream 인터페이스를 이용해 함수형으로 가독성 높은 코드를 구현  
```java
Files.lines(Paths.get(fileName))
    .filter(line -> line.startWith("ERROR"))
    .limit(40)
    .collect(toList());
```

**데이터를 수집하는 DSL인 Collectors**  
Collector 인터페이스는 데이터 수집(수집, 그룹화, 파이션)을 수행하는 DSL로 간주할 수 있다.
```java
Map<String, Map<Color, List<Car>>> carsByBrandAndColor = 
    cars.stream().collect(grouping(Car::getBrand, groupingBy(Car::getColor)));
    
// 두 Comparators 연결    
Comparator<Person> comparator = 
    comparing(Person::getAge).thenComparing(Person::getName);
// 중첩
Collector<? super Car, ?, Map<Brand, Map<Color, List<Car>>>> carGroupingCollector = 
    groupingBy(Car::getBrand, groupingBy(Car::getColor));
```

### 10.3 자바로 DSL을 만드는 패턴과 기법
DSL은 특정 도메인 모델에 적용할 친화적이고 가독성 높은 API를 제공한다.

도메인 객체의 API를 직접 이용해 주식 거래 주문
```java
Order order = new Order();
order.setCustomer("BigBank");

Trade trade1 = new Trade();
trade1.setType(Trade.Type.BUY);

Stock stock1 = new Stock();
stock1.setSymbol("IBM");
stock1.setMarket("NYSE");

trade1.setStock(stock1);
trade1.setPrice(125.00);
trade1.setQuantity(80);
order.addTrade(trade1);

Trade trade2 = new Trade();
trade2.setType(Trade.Type.BUY);

Stock stock2 = new Stock();
stock2.setSymbol("GOOGLE");
stock2.setMarket("NASDAQ");

trade2.setStock(stock2);
trade2.setPrice(375.00);
trade2.setQuantity(50);
order.addTrade(trade2);
```
장황하고 비개발자인 도메인 전문가가 위 코드를 이해하고 검증하기를 기대할 수 없다.

**1. 메서드 체인**  
메서드 체인으로 거래주문 정의
```java
Order order = forCustomer("BigBank")
    .buy(80)
    .stock("IBM")
    .on("NYSE")
    .at(125.00)
    .sell(50)
    .stock("GOOGLE")
    .on("NASDAQ")
    .at(375.00)
    .end();
```
메서드 체인으로 객체를 생성하려면 도메인객체를 만드는 빌더를 구현해야 한다.

장점
- 이 접근 방식은 주문에 사용한 파라미터가 빌더 내부로 국한된다는 장점.
- 정적 메서드 사용을 최소화하고 메서드 이름이 인수의 이름을 대신하도록 만듦으로서 DSL 가독성을 개선하는 효과가 있음.

단점
- 빌더를 구현해야함.
- 상위수준의 빌러를 하위수준의 빌더와 연결할 접착 코드가 필요.
- 도메인 객체 중첩구조와 일치하게 들여쓰기를 강제하는 방법이 없다.

**2. 중첩된 함수 이용**
다름 함수 안에 함수를 이용해 도메인 모델을 만든다.
```java
Order order = order("BigBank",
                buy(80, stock("IBM", on("NYSE")), at(125.00)),
                sell(50, stock("GOOGLE", on("NASDAQ")), at(375.00))
);
```
장점
- 메서드 체인에 비해 함수의 중첩 방식이 도메인 객체 계층 구조에 그대로 반영된다는 것

단점
- 많은 괄호를 사용해야함.
- 인수 목록을 정적메서드에 넘겨줘야 한다는 제약.
- 인수의 의미가 이름이 아니라 위치에 의해 정의된다..

**3. 람다 표현식을 이용한 함수 시퀀싱**  
람다표현식으로 정의한 함수 시퀀스 사용한다.
```java
Order order = order(o -> {
    o.forCustomer("BigBank");
    o.buy(t -> {
        t.quantity(80);
        t.price(125.00);
        t.stock(s -> {
            s.symbol("IBM");
            s.market("NYSE");
        });
    });
    o.sell( t -> {
        t.quantity(50);
        t.price(375.00);
        t.stock(s -> {
            s.symbol("GOOGLE");
            s.market("NASDAQ");
        });
    });
});

```
람다 표현식을 받아 실행해 도메인 모델을 만들어내는 여러 빌더를 구현해야 한다.

장점
- 메서드 체인 패턴 처럼 프루언트 방식으로 정의 객체를 정의 할 수 있다.
- 중첩 함수 형식처럼 다양한 람다 표현식의 중첩 수준과 비슷하게 도메인 객체의 계층 구조를 유지한다.

단점 
- 많은 설정코드가 필요하다.
- 람다 표현식 문법에 의한 잡음도 영향을 받느다.

**4. 조합하기**  
여러 DSL 패턴 이용하기
```java
Order order = forCustomer("BigBank", //최상위 주문속성 지정하는 중첩함수
                        buy(t -> t.quantity(80) // 한 개의 주문을 만드는 람다
                            .stock("IBM") // 거래 객체를 만드는 람다의 바디 메서드 체인
                            .on("NYSE")
                            .at(125.00)),
                        sell(t -> t.quantity(50)
                                    .stock("GOOGLE")
                                    .on("NASDAQ")
                                    .at(125.00)));
```
여러 DSL 패턴을 횬용해 가독성을 높일 수 있었다.  
하지만 여러 기법을 혼용하고 있으므로 한 가지 기법을 적용한 DSL에 비해 사용자가 DSL을 배우는데 시간이 오래 걸린다.

**5. DSL에 메서드 참조 사용하기**  
주문의 총 합에 0개 이상의 세금을 추가해 최종값을 계산하는 기능 추가
```java
public class TaxCalculator {
    public DoubleUnaryOperator taxFunction = d -> d;
    // 주문갑에 적용된 모든 세금을 계산
    
    public TaxCalculator with(DoubleUnaryOperator f) {
        taxFunction = taxFunction.andThen(f); 
        //새로운 세금 계산함수 얻어 인수로 전달된 함수와 현재 함수 합침.
        return this;
    }
    
    public double calculate(Order order) {
        return taxFunction.applyAsDouble(order.getValue());
    } // 주문의 총 합에 세금 계산 함수를 적용해 최종 주문값 계산.
}


double value = new TaxCalculator().with(Tax::regional)
                                  .with(Tax::surcharge)
                                  .calculate(order);
```
메서드 참조를 활용하고 읽기 쉽고 간결한 코드를 구현하였다.

참고 p353, 
표 10-1 DSL 패턴의 장점과 단점

### 10.4  실생활 자바 8 DSL
**1. jOOQ**  
SQL은 DSL에 가장 흔히 사용되는 분야로   
jOOQ는 SQL을 구현하는 내부적 DSL로 자바에 직접 내장된 형식 안전 언어이다.  
```java
create.selectFrom(BOOK)
      .where(BOOK.PUBLISHED_IN.eq(2016))
      .orderBy(BOOK.TITLE)
```

**2. 큐컴버**  
동작 주도 behavior-driven develoment(BDD)은 테스트 주도 개발의 확장으로   
다양한 비즈니스 시나리오를 구조적으로 서술하는 간단한 도메인 전용 스크리팅 언어를 사용한다.  
큐컴버는 다른 BDD 프레임워크와 마찬가지로 이들 명령문을 실행할 수 있는 테스트 케이스로 변환한다.  
이는 테스트와 동시에 비즈니스 기능의 수용 기준이 되기도 한다.  
```java
Feature: Buy stock
  Scenario: Buy 10 IBM stocks
    Given the price of a "IBM" stock is 125$
    When I buy 10 "IBM"
    Then the order value should be 1250$
```

**3. 스프링 통합**  
스프링 통합은 엔터프라이즈 통합패턴을 지원할 수 있도록 의존성 주입에 기반한 스프링 프로그래밍 모델을 확장한다.  
스프링 통합의 핵심 목표는 복잡한 엔터프라이즈 통합 솔루션을 구현하는 단순한 모델을 제공하고 비동기, 메시지 주도 아키텍퍼를 쉽게 적용할 수 있게 돕는 것이다.
```java
@Bean 
public IntegrationFlow myFlow() {
    return flow -> flow.filter((Integer p) -> p%2 == 0)
                       .transform(Object::toString)
                       .handle(System.out::println);
}
```
스프링 통합 DSL에서 가장 널리 사용하는 패턴은 메서드 체인으로  
이 패턴은 전달되는 메시지 흐름을 만들고 데이터를 변환하는 기능에 적합하다.