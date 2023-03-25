## 11. null 대신 Optional 클래스

### 11.1. 값이 없는 상황을 어떻게 처리할까?

```java
public String getCarInsuranceName(Person person){
    return person.getCar().getInsurance().getName();    
}
```
person이 없을 경우, 차가 없을 경우, 차있는 있는데 보험이 없을 경우,  
다양한 경우에 nullPointException 가 발생할 수 있다.

**보수적인 자세로 NullPointException 줄이기**
```java
public String getCarInsuranceName (Person person) {
    if(Person == null) {
        return "Unknown";
    }
    Car car = person.getCar();
    if(car == null) {
        return "Unknown";
    }
    Insurance insurance = car.getInsurance();
    if(insurance == null) {
        return "Unknown";
    }
    return insurance.getName();
}
```
모든 변수가 null인지 의심하므로 중첩된 if문이 추가되면서 코드 구조가 엉망이 되고 가독성도 떨어진다.

**null 때문에 발생하는 문제**
- 에러의 원인이다.
- 코드를 어지럽힌다.
- 아무 의미가 없다 
- 바 철학에 위배된다 : 자바는 모든 포인터를 숨겼다. 하지만 예외가 null 포인터이다.
- 형식 시스템에 구멍을 만든다. : null은 무형식이며 정보를 포함하고 있지 않으므로 모든 참조 형식에 null을 할당할 수 있다.

**다른 언어는 null 대신 무얼 사용하나?**
- 그루비 : 안전 내비케이터 연산자 (?.)
- 하스켈 : Maybe 형식 
- 스칼라 : T 형식

### 11.2 Optional 클래스 소개
자바8 은 하스켈과 스칼라의 영향을 받아 java.util.Optional<T> 클래스를 제공한다.  
값이 있으면 Optional 클래스는 값을 감싼고, 값이 없으면 Optional.empty 메서드로 Optional 을 반환한다.  
Optional을 이용하면 값이 없는 상황이 데이터에 문제가 있는것인지 알고리즘 버그인지 명확하게 구분 가능하다.  
하지만 모든 null 참조를 Optional로 대치하는 것은 바랍직하지 않다.  

### 11.3 Optional 적용 패턴
- 빈 Optional  
`Optional<Car> optCar = Optional.empty();`


- null이 아닌 값으로 Optional 만들기  
Optional.of 로 null이 아닌 값을 포함하는 Optional  
`Optional<Car> optCar = Optional.of(car);`  
  car가 null이라면 NullPointException발생할것.


- null값으로 Optional 만들기  
null값을 저장할 수 있는 Optional  
`Optional<Car> optCar = Optional.ofNullable(car);`  
  car가 null이면 빈 Optional 객체가 반환한다.  

**맵으로 Optional의 값을 추출하고 변환하기**
```java
String name = null;
if(insurance != null) {
    name = insurance.getName();
}

Optional<Insurance> optInsurance = Optional.ofNullable(insurance);
Optional<String> name = optInsurance.map(Insurance::getName);
```
Optional이 값을 포함하면 map의 인수로 제공된 함수가 값을 바꾼다.  
Optional이 비어있으면 아무일도 일어나지 않는다.


**flatMap으로 Optional 객체 연결**  
Optional로 자동차의 보험회사 이름 찾기
```java
Optional<Person> optPerson = Optrion.of(person);
Optional<String> name = optPerson.map(Person::getCar)
                                .map(Car::getInsurance)
                                .map(Insurance::getName);
```
위 코드는 컴파일되지 않는다.  
optPerson 는 Optional<Person> 이므로 map 메서드를 호출할수 힜다.  
하지만 getCar는 Optional<Car>를 반환한다.  
즉 map 연산의 결과는 Optional<Optional<Car>> 형식이 된다.  
getInsurance은 또 다른 Optional 객체를 반환하므로 getInsurance 메서드는 지원하지 않게 된다.  

이 문제를 어떻게 해결할까? flatMap  
스트림의 flatMap은 함수를 인수로 받아서 다른 스트림을 반환하는 메서드이다.  
보통 인수로 받은 함수를 스트림의 각 요소에 적용하면 스트림의 스트림이 만들어지는 데  
flatMap은 인수로 받은 함수를 적용해서 생성된 각각의 스트림에서 콘텐츠만 남는다.  
즉 함수를 적용해서 생성된 모든 스트림이 하나의 스트림으로 병합되어 평준화한다.  

flatMap 사용
```java
public String getCarInsuranceName(Optional<Person> person) {
    return person.flatMap(Person::getCar)
                 .flatMap(Car::getInsurance)
                 .map(Insurance::getName) // string을 반환하므로 flatMap을 사용할 필요가 없다.
                 .orElse("Unknown"); // 비어있으면 기본값
```

**디폴트 액션과 Optinal 언랩**
- get()  
값을 읽는 간단한 메서드지만 안전하지 않다.  
래핑된 값이 있으면 반환하고 없으면 NoSuchElementException 발생.


- orElse(T other)
값을 포함하지 않을 때 기본값을 제공한다.


- orElseGet(Supplier<? extends T> other)
orElse 메서드의 게으른 버전이다.   
Optional에 값이 없을 때만 Supplier가 실행된다. 비어있을때만 기본값을 생성하게 된다.


- orElseThrow(Supplier<? extends X> exceptionSupplier)
Optional이 비어있을때 지정한 예외를 발생시킨다.


- ifPresent(Consumer<? super T> consumer)
값이 존재할 때 인수로 넘겨준 동작을 실행할 수 있다.


- ifPresentOrElse(Consumer<? super T> action, Runnable emptyAction)  
자바9에서 추가됨, Optional이 비었을 때 실행할 수 있는 Runnable을 인수로 받는다.

**두 Optinal 합치기**
```java
public Insurance findCheapestInsurance(Person person, Car car) {
    // 보험회사가 제공하는 서비스 조회
    // 모든 결과 데이터 비교
    return cheapestCompany;
}
```

isPresent 이용해 구현
```java
public Optional<Insurance> nullSafeFindCheapestInsurance(
            Optional<Person> person, Optional<Car> car) {
    if(person.isPresent() && car.isPresent()){
        return Optional.of(findCheapestInsurance(person.get(), car.get()));
    } else {
        return Optional.empty();
    }
}
```

map과 flapMap을 이용해서 조건문을 사용하지 않고 한줄의 코드로 구현
```
public Optional<Insurance> nullSafeFindCheapestInsurance(
            Optional<Person> person, Optional<Car> car) {
    return person.flatMap(p -> car.map(c -> findCheapestInsurance(p,c)));
}
```

**필터로 특정값 거르기**
```java
// 조건문으로 단순 null 체크
Insurance insurance = ...;
if(insurance != null && "CambridgeInsurance".equal(insurance.getName())) {
    System.out.println("ok");
}

// Optional 객체에 filter 메서드 사용
Optional<Insurance> optInsurance = ...;
optInsurance.filter(ins -> "CambridgeInsurance".equals(insurance.getName()))
            .ifPresent(x -> System.out.println("ok"););
```

### 11.4. Optional을 사용한 실용 예제
**잠재적으로 null이 될 수 있는 대상을 Optional로 감싸기**  
`Object value = map.get("key");`  
map에서 반환하는 값을 Optional로 감싸서 이를 개선할 수 있다.  
`Optional<Object> value = Optional.ofNullable(map.get("key"));`

**예외와 Optional 클래스**  
자바 API는 어떤 이유에서 값을 제공할 수 없을 때 null을 반환하는 대신 예외를 발생 시킨다.  
문자열을 정수로 바꾸지 못해 발생한 NumberFormatException 예외를 Optional 로 처리하기  
```java
public static Optional<Integer> stringToInt(String s) {
    try{
        return Optional.of(Integer.parseInt(s));
    } catch (NumberFormatException e) {
        reutn Optional.empty(); // 예외 발생시 빈 Optional 반환
    }
}
```
**기본형 Optional을 사용하지 말아야 하는 이유**  
스트림처럼 Optional도 기본형 특화 클래스 OptionalInt, OptionalLong, OptionalDouble 등을 제공한다.  
하지만 Optional의 최대 요소 수는 한개 이므로 기본형 특화 클래스로 성능을 개선할 수 없다.  
또한 기본형특화 클래스에는 map, filter 등의 메서드를 지원하지 않으므로 사용을 권장하지 않는다.  

