## chapter1. 자바 8,9,10,11 : 무슨 일이 일어나고 있는 가?

### 1.1 역사의 흐름은 무엇인가?
자바 8은 간결한 코드, 멀티코어 프로세서의 쉬운 활용이라는 두 가지 요구사항을 기반으로 한다.
- 스트림 API
- 메서드에 코드를 전달하는 기법
- 인터페이스의 디폴트 메서드


### 1.2 왜 아직도 자바는 변화하는 가?
자바 9 설계의 밑바탕을 이루는 세 가지 프로그래밍 개념
1. 스트림 처리
- 스트림이란 한 번에 한 개씩 만들어지는 연속적인 데이터 항목들의 모임이다.
- 자동차 조립라인처럼 물리적인 순서로 한개씩 운반하지만 각각 작업장은 동시에 작업을 처리한다.
- 데이터베이스 질의처럼 고수준으로 추상화해서 일련의 스트림으로 만들어서 처리 할 수 있다.
- 스트림 파이프라인을 이용해 입력 부분을 여러 CPU 코어에 쉽게 할당 할 수 있다. 복잡한 작업 없이 병렬성의 효과를 얻을 수 있다.

2. 동작 파라미터화로 메서드에 코드 전달하기  
   동작 파라미터화: 메서드(코드)를 다른 메서드의 인수로 넘겨주는 기능  
   스트림이 연산의 동작을 파라미터화 할 수 있는 코드를 전달한다는 사상에 기초하기 때문에 중요한 기능 중 하나


3. 병렬성과 공유 가변 데이터  
   보통 다른 코드와 동시에 실행하더라도 안전하게 실행할 수 있는 코드를 만들려면 공유된 가변 데이터에 접근하지 않아야 한다.    
   하지만 공유된 변수나 객체가 있으면 병렬성에 문제가 발생한다.  
   synchronized를 이용해서 보호하는 규칙을 만들 수 있을 것이다.  
   하지만 자바 8 스트림을 이용하면 기존의 자바 스레드 API보다 쉽게 병렬성을 활용할 수 있다.

### 1.3 자바 함수
**메서드 참조 (method reference)**
- 익명 클래스를 통한 파일 리스팅
```
File[] hiddenFiles = new File(".").listFiles(new FileFilter() {
	public boolean accept(File file) {        
		return file.isHidden();    
	}
});
```
- 메서드 참조를 이용한 파일 리스팅
```
File[] hiddenFiles = new File(".").listFiles(File::isHidden);
```
new로 객체 참조를 생성할 필요없이 File::isHidden를 이용해 메서드 참조로 코드가 간결해 졌다.

**람다 : 익명 함수**  
자바 8에서는 메서드를 일급값으로 취급할 뿐 아니라 람다(anonymous function)를 포함하여 함수도 값으로 취급할 수 있다.

예제
```
    public static List<Apple> filterGreenApples(List<Apple> inventory){
        List<Apple> result = new ArrayList<>();
        for (Apple apple: inventory){
            if (GREEN.equals(apple.getColor())) {
                result.add(apple);
            }
        }
        return result;
    }
```
메서드 참조 활용
```
    public static boolean isGreenApple(Apple apple) {
        return GREEN.equals(apple.getColor()); 
    }
    
    public static List<Apple> filterApples(List<Apple> inventory, Predicate<Apple> p){
        List<Apple> result = new ArrayList<>();
        for(Apple apple : inventory){
            if(p.test(apple)){
                result.add(apple);
            }
        }
        return result;
    }    
```
```
 List<Apple> greenApples = filterApples(inventory, FilteringApples::isGreenApple);
```
람다 활용
```
List<Apple> greenApples2 = filterApples(inventory, (Apple a) -> "green".equals(a.getColor()));
```
### 1.4 스트림
스트림 API 이전 컬렉션 API를 이용하여 다양한 로직을 처리하였을 것이다.  
스트림 API를 이용하면 컬렉션 API와는 상당히 다른 방식으로 데이터를 처리할 수 있다.  
컬렉션 API를 사용하 for-each 루프를 이용하여 각 요소를 반복하면서 작업을 수행했다.  
이러한 방식의 반복을 외부 반복(external iteration)이라 한다.  
반면 스트림 API를 이용하면 루프를 신경 쓸 필요가 없다.  
스트림 API에서는 라이브러리 내부에서 모든 데이터가 처리된다.  
이와 같은 반복을 내부 반복(internal iteration)이라고 한다.


스트림 API는  데이터 필터링, 데이터 추출, 데이터 그룹화 등 컬렉션을 처리하면서  
발생하는 모호함과 반복적인 코드문제, 멀티코어 활용의 어려움이라는 문제를 해결했다.  
스트림은 스트림 내의 요소를 쉽게 병렬로 처리할 수 있는 환경을 제공하는 것이 핵심이다.  
처음에는 이상하게 들릴 수 있겠지만 컬렉션을 필터링할 수 있는 가장 빠른 방법은  
컬렉션을 스트림으로 바꾸고, 병렬로 처리한 다음에, 리스트로 다시 복원하는 것이다.

### 1.5 디폴트 메서드와 자바 모듈
자바 8에스는 인터페이스를 쉽게 바꿀 수 있도록 디폴트 메서드를 지원한다.  
예를 들어 자바 8에서는 List에 sort메서드를 직접 호출 할 수 있다.  
이는 자바 8의 List 인터페이스에 드음과 같은 디폴트 메서드 정의가 추가되었기 때문이다.
```
default void sort(Comparator<? super E> c){
    Collections.sort(this, c);
}
```
### 1.6 함수형 프로그래밍에서 가져온 다른 유용한 아이디어
자바 8에서는 NullPointer예외를 피할 수 있도록 도와주는 Optional<T> 클래스를 제공한다.  
Optional<T>는 값을 갖거나 갖지 않을 수 있는 컨터이너 객체로  
값이 없는 상황을 어떻게 처리할지 명시적으로 구현하는 메서드를 포함하고 있어 NullPointer예외를 방지 할 수 있다.

> 자바 8에서는   
> 메서드 참조와 람다로 병렬성처리와 간결한 코드를 구현 할 수 있다.  
> 디폴트 메서드를 이용해 기존 인터페이스를 구현하는 클래스를 바꾸지 않고도 인터페이스를 변경 할 수 있다.  
> Optional의 등장으로 NullPointerException 방지할 수 있다.  