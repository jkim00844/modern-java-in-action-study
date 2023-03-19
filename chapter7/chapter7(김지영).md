## chapter7 병렬 데이터 처리와 성능
스트림으로 데이터 컬렉션 동작을 얼마나 쉽게 병렬로 실행할 수 있는지 설명한다.  
스트림을 이용하면 순차 스트림을 병렬 스트림으로 자연스럽게 바꿀수 있다.  

### 7.1 병렬스트림
컬렉션에 parallelStream을 추출하면 병렬 스트림(parallel stream)이 생성된다.  
병렬 스트림이란 각각의 스레드에서 처리할 수 잇도록 스트림 요소를 여러 청크로 분할한 스트림이다.
**순차 스트림을 병렬 스트림으로 변환하기**
```java
// 숫자 n을 인수로 받아서 1부터 n까지의 모든 숫자의 합계
public long sequentialSum(long n) {
        return Stream.iterate(1L, i -> i + 1)
                     .limit(n)
                     .reduce(0L, Long::sum);
    }
```
순차 스트림에 parallel 메서드를 호출하면 기존의 함수형 리듀싱 연산이 병렬로 처리된다.
```java
public long parallelSum(long n) {
        return Stream.iterate(1L, i -> i + 1)
                     .limit(n)
                     .parallel()
                     .reduce(0L, Long::sum);
    }
```

**스트림 성능 측정**
반복형(for 루프) VS 순차 스트림 VS 병렬 스트림  
성능 측정을 위해 자바 마이크로 벤치마크(Java Microbenchmark Harness, JMH)라는 라이브러리를 이용했다.  
반복형 - 순차 스트림 - 병렬 스트림 순으로 성능이 좋았다.  
병렬 스트림이 순차 버전에 비해 다섯 배나 느린 실망스러운 결과가 나왔다. 

두가지 원인
- 반복 결과로 박싱된 객체가 만들어지므로 숫자를 더하려면 언박싱을 해야한다.
- 반복 작업은 병렬로 수행할 수 있는 독립 단위로 나누기 어렵다.  

결국 순차처리 방식과 크게 다른 점이 없으므로 스레드를 할당하는 오버헤드만 증가하게 되었다.

**더 특화된 메서드 사용**  
iterate 대신 LongStream과 같은 기본형 특화 스트림을 이용해서 박싱 비용을 줄여보자.  
LongStream의 장점
- LongStream.rangeClosed는 기본형 long을 직접 사용하므로 박싱과 언박싱 오버헤드가 사라짐
- LongStream.rangeClosed는 쉽게 청크로 분할할 수 있는 숫자 범위를 생산함.     
예를들어 1-20범위의 숫자를 각각 1-5, 6-10, 11-15, 16-20으로 분할할 수 있음

```java
    @Benchmark
    public long rangedSum() {
        return LongStream.rangeClosed(1, N)
                         .reduce(0L, Long::sum);
    }
```
```
    @Benchmark
    public long rangedSum() {
        return LongStream.rangeClosed(1, N)
                         .reduce(0L, Long::sum);
    }
```
순차 실행보다 빠른 성능을 갖는 병렬 리듀싱을 만들었다.  

올바른 자료구조를 선택해야 병렬 실행도 최적의 성능을 발휘할 수 있다는 사슬을 확인할 수 있다.  
병렬화가 완전 공짜라는 아니라는 사실을 기억하자!  
스트림을 재귀적으로 분할하고, 각 서브스트림을 서로 다른 스레드의 리듀싱 연산으로 할당하고, 결과를 하나의 값으로 합치는 일련의 작업이 수반된다.

**병렬 스트림의 올바른 사용법**
```java
// n까지의 자연수를 더하면서 공유된 누적자를 바꾸는 코드
public long sideEffectParallelSum(long n) {
        Accumulator accumulator = new Accumulator();
        LongStream.rangeClosed(1, n).parallel().forEach(accumulator::add);
        return accumulator.total;
}
```
병렬 스트림으로 활용한 코드로 결과의 결과값이 제대로 나오지 않는다.  
여러 스레드에서 공유하는 객체의 상태를 바꾸는 foreach에서 add 메서드를 호출하면서 문제가 발생한다.


별렬 스트림과 병렬 계산에서는 공유된 가변 상태를 피해야 한다.

**병렬 스트림 효과적으로 사용하기**
- 확신이 서지 않으면 직접 측정하라. 무조건 병렬 스트림으로 바꾸는 것이 능사는 아니다.
- 박싱을 주의하라. 자동 박싱과 언박싱은 성능을 크게 저하시킬 수 있는 요소이다. 기본형 특화 스트림을 사용하자.
- 순차 스트림보다 병렬 스트림에서 성능이 떨어지는 연산이 있다.  
특히 limit, findFirst처럼 요소의 순서에 의존하는 연산들..
- 스트림에서 수행하는 전체 파이프라인 연산 비용을 고려하라.  
처리해야 할 요소수가 N, 하나의 요소를 처리하는데 드는 비용을 Q라고 하면 Q가 높아진다는 것은 병렬 스트림으로 성능을 개선할 수 있는 가능성이 있다.
- 소량의 데이터에서는 병렬 스트림이 도움되지 않는다.
- 스트림을 구성하는 자료구조가 적절한지 확인하라.  
예를 들어 ArrayList를 LinkedList보다 효율적으로 분할할 수 있다.
- 스트림의 특성과 파이프라인의 중간 연산이 스트림의 특성을 어떻게 바꾸는지에 따라 분해 과정의 성능이 달라질 수 있다.  
예를 들어 SIZED 스트림은 정확히 같은 크기의 두스트림으로 분할할 수 있으므로 효과적인 스트림을 병렬 처리할 수 있다.  
- 최종 연산의 병합 과정 비용을 살펴보라. 예를들면 Collector의 combine메서드  
병합 과정의 비용이 비싸다면 병렬 스트림으로 얻은 성능의 이익이 서브스트림의 부분 결과를 합치는 과정에서 상쇄할 수 있다.

### 7.2 포크/조인 프레임워크
병렬 스트림의 내부 구조 - 포크/조인 프레임워크  
포크/조인 프레임워크는 병렬화할 수 있는 작업을 재귀적으로 작은 작업으로 분할한 다음에 서브태스크 각각의 결과를 합쳐서 전체 결과를 만들도록 설계되었다.  
포크/조인 프레임워크에서는 서브태스크를 스레드 풀(ForkJoinPool)의 작업자 스레드에 분산 할당하는 ExecutorService 인터페이스를 구현한다.  

**RecursiveTask 활용**  
스레드 풀을 이용하려면 RecursiveTask<R>의 서브클래스를 만들어야 한다.  
여기서 R은 병렬화된 테스크가 생성하는 결과 형식 또는 결과가 없을 때는 RecursiveAction이다.  
RecursiveTask를 정의 하려면 추상메서드 compute 구현해야 한다.  
`protected abstract R compute();`

compute 메서드 의사코드
```java
if (태스크가 충분히 작거나 더 이상 분할할 수 없으면) {
    순차적으로 태스크 계산
} else {
    태스크를 두 서브태스크로 분할
    태스크가 다시 서브태스크로 분할되도록 이 메서드를 재귀적으로 호출함
    모든 서브태스크의 연산이 완료될 때까지 기다림
    각 서브태스크의 결과를 합침
}
```
이는 분활 후 정복(divide-and-conquer)알고리즘의 병렬화 버전이다.

**포크/조인 프레임워크를 제대로 사용하는 방법**
- join 메서드를 태스크에 호출하면 태스크가 생산하는 결과가 준비될 때 까지 호출자를 블록시킨다.  
따라서 두 서브태스크가 모두 시작된 다음에 join을 호출해야 한다.  
그렇지 않으면 각각의 서브태스크가 다른 태스크가 끝나길 기다리게 되면서 순차 알고리즘보다 느리고 복잡한 프로그램이 될 수 있다.
- RecursiveTask 내에서는 ForkJoinPool의 invoke 메서드 대신 compute나 fork 메서드를 호출한다.   
순차 코드에서 병렬 계산을 시작할 때만 invoke를 사용한다.
- 서브테스크에 fork 메서드를 호출해서 ForkJoinPool의 일정을 조절할 수 있다.
- 포크/조인 프레임워크를 이용하는 병렬 계산은 디버깅이 어렵다.
- 티코어에서 포크/조인 프레임워크를 사용하는 것이 순차처리보다 무조건 빠른 거라는 생각은 버려야 한다.

### 7.3 Spilterator 인터페이스
스트림을 자동으로 분할해주는 기능 - Spliterator
Spliterator는 분할할 수 있는 반복자라는 의미다.  
Iterator 처럼 소스의 요소 탐색 기능을 제공하지만 병렬 작업에 특화되어있다.  
자바 8은 컬렉션 프레임워크에 포함된 모든 자료구조에 사용할 수 있는 디폴트 Spliterator 구현을 제공한다.  
```
public interface Spliterator<T> {
    boolean tryAdvance(Consumer<? super T> action);
    Spliterator<T> trySplit();  
    long estimateSize();    
    int characteristics();
}
```
- tryAdvance 메서드는 Spliterator의 요소를 하나씩 순차적으로 소비하면서 탐색해야 할 요소가 남아있으면 true를 반환(iterator 동작과 같다)
- trySplit 메서드는 Spliterator의 일부 요소(자신이 반환한 요소)를 분할해서 두 번째 Spliterator를 생성하는 메서드
- estimateSize 메서드는 탐색해야 할 요소 수 정보 제공한다.
- characteristics 메서드는 Spliterator 객체의 특성에 대한 int값을 반환한다.

분할 과정
스트림을 여러 스트림으로 분할하는 과정은 재귀적으로 일어난다.
null을 반환할때 까지 반복하고 null을 반환했다는 것은 더 이상 자료구조를 분할 할 수 없음을 의미한다.

**Spliterator 특성**  
Spliterator의 characteristics 추상 메서드는 Spliterator 자체의 특성 집합을 int 타입으로 반환한다.  
Spliterator를 이용하는 프로그램들은 이들 특성을 참고해서 Spliterator를 더 잘 제어하고 최적화할 수 있다.

|특성|의미|
|----|----|
|ORDERED| 리스트처럼 요소에 정해진 순서가 있으므로 요소를 탐색하고 분할할 때 이 순서에 유의해야한다.|
|DISTINCT|	x, y 두 요소를 방문했을 때 x.equals(y)는 항상 false를 반환한다. |
|SORTED	|탐색된 요소는 미리 정의된 정렬 순서를 따른다.|
|SIZED	크|기가 알려진 소스로 Spliterator를 생성했으므로 estimateSize는 정확한 값을 반환한다.|
|NONNULL|	탐색하는 모든 요소는 null이 아니다.|
|IMMUTABLE|	이 Spliterator의 소스는 불변이다. 즉, 요소를 탐색하는 동안 요소를 추가하거나, 삭제, 수정이 불가하다.|
|CONCURRENT|	동기화 없이 Spliterator의 소스를 여러 스레드에서 동시에 고칠 수 있다.|
|SUBSIZED|	Spliterator 그리고 분할되는 모든 Spliterator는 SIZED 특성을 갖는다.|