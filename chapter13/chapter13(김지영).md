## Chapter13 디폴트 메서드
인터페이스를 구현하는 클래스는   
인터페이스에서 정의하는 모든 메서드 구현을 제공하거나 아니면 슈퍼클래스의 구현을 상속받아야한다.  
평소에는 이 규칙을 지키는데 아무 문제가 없지만 라이브러리 설계자 입장에서는   
인터페이스에 새로운 메서드를 추가하거나 바꾸고 싶을 때 문제가 발생한다.   
인터페이스르 바꾸면 이전에 해당 인터페이스를 구현했던 모든 클래스의 구현도 고쳐야 하기 때문이다.  

자바 8에서는 이 문제를 해결하기 위해 2가지 방법을 제공한다.  
- 인터페이스 내부에 정적 메소드 사용
- 디폴트 메서드 사용

```java
List<Integer> numbers = Arrays.asList(3, 5, 1, 2, 6);
numbers.sort(Comparator.naturalOrder());
```
naturalOrder 메서드는 알파벳요소로 정렬하는 Comparator 정적 메소드  
sort는 List인터페이스 디폴트 메서드  
### 13.1 변화하는 API 
**API 버전 1**
```java
// Resizable 인터페이스 초기버전
public interface Resizable extends Drawable {
    int getWidth();
    int getHeight();
    void setWidth(int width);
    void setHeight(int height);
    void setAbsoluteSize(int width, int height);
}

// 사용자 구현 Ellipse 클래스
public class Ellipse implements Resizable {
    ...
}

public class Game {
    public static void main (String...args) {
        List<Resizable> resizableShapes = Arrays.asList(new Square(), new Recrangle(), new Ellipse());
        Utils.paint(resizableShapes);
    }
}

public class Utils {
    public static vlid paint(List<Resizable> l) {
        l.forEach(r -> {
            r.setAbsoluteSize(42, 42);
            r.draw();
        });
    }
}
```
**API 버전 2**  
Resizable을 구현하는 Square와 Rectangle 구현을 개선해달라는 요청을 받음.
```java
public interface Resizable extends Drawable {
    int getWidth();
    int getHeight();
    void setWidth(int width);
    void setHeight(int height);
    void setAbsoluteSize(int width, int height);
    void setRelativeSize(int wFactor, int hFactor); // 추가된 메서드
}

```

**사용자가 격는 문제**  
setRelativeSize 메서드를 구현해야한다.  
인터페에스에 새로운 메서드를 추가하면 바이너리 호환성은 유지된다.  
하지만 이때 Ellipse 객체가 인수로 전달되면 setRelativeSize 메서드를 정의하지 않았으므로 런타임 에러가 발생할 것이다.

- 바이너리 호환성  
뭔가를 바꾼 이후에도 에러없이 기존 바이너리가 실행될 수 있는 상황
ex) 인터페이스에 메서드를 추가했을때 추가된 메서드를 호출하지 않는 한 문제가 일어나지 않는 것.

- 소스 호환성  
코드를 고쳐도 기존 프로그램을 성공적으로 컴파일 할 수 있는것.

- 동작 호환성  
코드를 고쳐도 같은 입력값이 주어지면 프로그램이 같은 동작을 실행한다는 것

### 13.2 디폴트 메서드란 무엇인가?
자바8에서는 호환성을 유지하면서 API를 바꿀수 있도록 디폴트 메서드를 제공한다.
```java
public interface Sized {
    int size();
    default boolean isEmpty() {
        return size() == 0;
    }
}
```
Sized 인터페이스를 구현하는 모든 클래스는 isEmpty의 구현도 상속받는다.  
즉 인터페이스에 디폴트 메서드를 추가하면 소스 호환성이 유지된다.  

추상클래스와 인터페이스의 차이  
클래스는 하나의 추상클래스만 상속받을수 있지만 인터페이스를 여러개 구현가능  
추상클래스는 필드를 공통 상태를 가질수 있음. 인터페이스는 가질 수 없음.  

### 13.3 디폴트 메서드 활용 패턴
1. 선택형 메스드  
인터페이스를 구현하는 클래스에서 메서드의 내용이 비어있는 때  
예를 들면 Iterator의 remove 메서드  
remove 기능은 잘 사용하지 않아 자바 8이전에는 remove에 빈 구현을 제공했다.  
디폴트 메서드를 이용하면 remove 같은 메서드에 기본 구현을 제공할 수 있으므로 빈 구현을 제공할 필요가 없다.
```java
interface Iterator<T> {
    boolean hasNext();
    T next();
    default void remove() {
        throw new UnsupportedOperationException();
    }
}
```

2. 다중 상속  
디폴트 메서드를 이용하면 기존에 불가능했던 동작 다중 상속 기능도 구현할 수 있다.  
디폴트 메서드를 이용하면 클래스는 다중상속을 이용해 기존코드를 재사용 할 수 있다.  
```java
public class ArrayList<E> extends AbstractList<E> 
        implements List<E>, RandomAccess, Cloneable, Serializable {
}
```
ArrayList는 1개의 클래스 상속받고 6개의 인터페이스 구현  
결과적으로 ArrayList는 AbstractList, List, Collection, Iterable, RandomAccess, Cloneable, Serializable의 서브 형식이 된다.  
(List<E> extends Collection<E>, Collection<E> extends Iterable<E> )  
디폴트 메서드를 사용하지 않아도 다중 상속을 활용할 수 있다.  

자바8 에서는 인터페이스가 구현을 포함할 수 있으므로 클래스는 여러 인터페이스에서 동작을 상속 받을수 있게 되었다.

**기능이 중복되지 않는 최소의 인터페이스**  
게임에서 다양한 특성을 같는 여러 모양을 정의한다고 가정하자.  
회전하는것, 움직이는것, 크기를 조절하는것 으로 기능을 나눠 인터페이스를 만든다.  

**인터페이스 조합**  
이들 인터페이스를 조합해서 게임에 필요한 다양한 클래스를 구현한다.
```
// 움직이고 회전하고 크기 조절할 수 있는 Monster 클래스
public class Monster implements Rotatable, Moveable, Resizable {
// 모든 추상메서드 구현을 해야하지만 디폴트메서드는 구현 x
...
}

// 움직이고 회전할 수 있는 Sun 클래스
puvlic class Sun implements Moveable, Rotatable {
...
}
```

### 13.4 해석 규칙
자바 8에는 디폴트 메서드가 추가되었으므로 같은 시그니처를 갖는 디폴트 메서드를 상속받는 상황이 생길 수 있다.  
실전에서는 자주 일어나지는 않지만 규칙은 있다.  

**알아야 할 세 가지 해결 규칙**
1. 클래스가 항상 이긴다. 클래스나 슈퍼클래스에서 정의한 메서드가 디폴트 메서드보다 우선권을 갖는다.  
2. 1번 다음(=클래스나 슈퍼클래스의 메서드 정의가 없으면) 서브인터페이스가 이긴다. B가 A를 상속받는다면 B가 이긴다.  
```java
public class B extends A{}
public class D implements A{}
public class C extends D implements B, A {
    public static void main(String... args){
        new C().hello();  // Hello from B
    } 
}
```
D는 hello를 오버라이드 하지 않았다. D는 A의 디폴트 메서드 구현을 상속받는다.  
이 슈퍼클래스 D에 hello() 메서드에 정의가 없으므로 A,B를 선택해야하는데  
여기선 B가 A를 상속받으므로 B의 hello 메서드가 출력된다.   
3. 2번에서도 결정이 안되면 여러 인터페이스를 상속받는 클래스가 명시적으로 디폴트 메서드를 오버라이드하고 호출해야한다.  
```java
public class C implements B, A {
    void hello() {
        B.super.hello(); // 명시적 선택
    }
}
```




