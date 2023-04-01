## Chapter 14 자바 모듈 시스템
자바9에서 가장 많이 거론되는 새로운 기능은 모듈 시스템이다.  
모듈시스템은 Jigsaw 프로젝트 내부에서 개발된 기능으로 완성까지 10년이 걸렸다.

14장에서는 모듈 시스템이란 무엇이며, 
새로운 자바 모듈 시스템이 어디에 사용될 수 있고
개발자는 이로부터 어떤 이익을 얻을 수 있는지를 설명한다.

### 14.1 압력: 소프트웨어 유추
유지보수하기 쉬운 코드를 구현하는 부분은 저수준의 영역이다.  
궁극적으로 소프트웨어 아키첵처 즉 고수준에서 기반 코드를 바꿔 생산성을 높일 수 있는 프로젝트를 만들어야한다.  
이때 관심사분리와 정보은닉의 원칙을 사용한다.  

**관심사 분리(SoC, Separation of Concern)**  
컴퓨터 프로그램을 고유의 기능으로 나누는 동작을 권장하는 원칙이다.  
회계 애플리케이션 개발중 파싱, 분석, 레포트 기능은 서로 거의 겹치지 않는 코드 그룹으로 분리할 수 있다.  
SoC원칙은 모델, 뷰, 컨트롤러 같은 아키텍쳐 관점, 복구 기법과 비즈니스로직 분리하는 샇황에 적용된다.  

장점  
- 개별 기능을 따로 작업할 수 있고 팀이 쉽게 협업한다
- 개별 부분을 재사용하기 쉽다
- 전체 시스템을 쉽게 유지보수 할 수 있다.

**정보은닉**  
세부 구현을 숨기도록 장려하는 원칙이다.  
소프트웨어의 요구사항은 자주 바꾼다.   
세부 구현을 숨김으로 프로그램의 어떤 부분을 바꿨을 때 다른 부분까지 영향을 미칠 가능성을 줄일 수 있다.  

**자바 소트프웨어**  
자바에서는 이 두가지 원칙을 적용하기 위해  
클래스와 인터페이스를 사용하고 특정 문제와 관련된 패캐지, 클래스, 인터페이스를 그룹으로 만들어 코드를 그룹화한다.  
또한 접근 제한자 (public, protected, private)등을 이용해 패키지, 메서드, 필드의 접근을 제어한다.  

### 14.2 자바 모듈 시스템을 설계한 이유  
**모듈화의 한계**  
이전에는 클래스, 패키지, JAR 세가지 수준의 코드 그룸화를 제공하였다.  
클래스 관련해 접근제한자와 캡슐화를 지원했지만  
패키지와 JAR 수준에서는 캡슐화를 거의 지원하지 않았다.  

**제한된 가시성 제어**  
public, protected, default, private 네가지 가시성 접근자  
한 패키지의 클래스와 인터페이스를 다른 패키지로 공개하려면 public으로 선언해야 하기에  
결과적으로 클래스와 인터페이스는 모두 공개된다.  


**클래스 경로**  
클래스를 모두 컴파일 후 보통 한개의 jar 파일에 넣고 클래스 경로에 JAR 파일을 추가해 사용할 수 있다.  
그러면 JVM이 동적으로 클래스 경로에 정의된 클래스를 필요할 때 읽는다.  
하지만 클래스 경로와 JAR 조합에는 약점이 있다.  

첫째, 클래스 경로에는 같은 클래스를 구분하는 버전 개념이 없다.  
같은 라이브러리 사용시 버전 1.0을 사용하는지 2.0을 사용하는지 지정 할 수 없다.  

둘째, 클래스 경로는 명시적인 의존성을 지원하지 않는다.  
각각의 JAR 안에 있는 모든 클래스는 classes로 합쳐진다.  
한 JAR 이 다른 JAR에 포함된 클래스 집합을 사용하라고 명시적으로 의존성을 정의하는 기능을 제공하지 않는다.  

따라서 메이븐이나 그레들 같은 빌드 도구를 사용해 문제를 해결한다.  
자바, JVM 에서는 명시적 의존성 정의를 지원하지 않았다.  

**거대한 JDK**  
자바 프로그램을 만들고 실행하는데 도움을 주는 도구의 집합이다.   
자바 프로그램을 컴파일하는 javac,   
애플리케이션을 로드하고 실행하는 java,  
입출력 호함 런타임 지원을 제공하는 JDK 라이브러리, 컬렉션, 스트림 등이 있다.  

JDK도 시간이 흐르면서 발전하고 덩치가 많이 커졌다.   
많은 내부 API는 공개되지 않아야하는데 자바의 낮은 캡슐화 지원으로 외부에 공개 되었다.  
ex) Spring, Netty, Mockito  
호횐성을 깨지 않고는 관련 API를 바꾸기 아주 어려운 상황이 되었다.  
이로 JDK 자체도 모듈화할 수 있는 자바 모듈 시스템 설계의 필요성이 제기 되었다.  
즉, JDK에서 필요한 부분만 골라 사용하고 클래스 경로를 쉽게 유추할 수 있으며,  
플랫폼을 진화시킬 수 있는 강렬한 캡슐화를 제공할 구조가 필요했다.

### 14.3 자바 모듈 : 큰 그림
자바8은 모듈이라는 새로운 자바 프로그램 구조단위를 제공한다.  
모둘은 module이라는 새 키워드에 이름과 바디를 추가해서 정의한다.  
모듈 디스크립터는 moduel-info.java라는 특별한 파일에 저장된다.  

### 14.4 자바 모듈 시스템으로 애플리케이션 개발하기
1. 애플리케이션 셋업   

비용처리 애플리케이션 요구사항
- 파일이나 URL에서 비용 목록을 읽는다
- 비용의 문자열 표현을 파싱한다
- 통계를 계산한다
- 유용한 요약 정보를 표시한다
- 각 태스크의 시작, 마무리 지점을 제공한다

애플리케이션의 개념을 모델링 할 여러 클래스와 인터페이스 정의
- 다양한 소스에서 데이터를 읽음 (Reader, HttpReader, FileReader)
- 다양한 포맷으로 구성된 데이터를 파싱 (Parser, JSONParser, ExpenseJSON-Parser)
- 도메인 객체를 구체화 (Expense)
- 통계를 계산하고 반환 (SummaryCalculator, SummaryStatistics)
- 다양한 기능을 분리조정 (ExpensesApplication)

각 기능을 그룹화
- expenses.readers
- expenses.readers.http
- expenses.readers.file
- expenses.parsers
- expenses.parsers.json
- expenses.model
- expenses.statistics
- expenses.application

2. 세부적인 모듈화와 거친 모듈화  
시스템을 모듈화할때 모듈의 크기를 결정해야 한다.  
세부적 모듈화 기법은 모든 패키지가 자신의 모듈을 갖는다.  
거친 모듈화 기법은 한 모듈이 시스템의 모든 패키지를 포함한다.  


3. 자바 모듈 시스템 기초  

디렉토리 구조  
```java
expenses.application  
 |- module-info.java  
 |-com  
  |- example  
   |- expenses  
    |- application  
     |- ExpensesApplication.java
```  

모듈화 애플리케이션 실행  
모듈 소스 디렉터리에서 다음 명령을 실행한다.
```java
javac module-info.java com/example/expenses/application/ExpensesApplication.java -d target

jar cvfe expenses-application.jar com.example.expenses.application.ExpensesApplication -D target
```
생성된 JAR(expenses-application jar)을 모듈화 애플리케이션으로 실행
```java
java --module-path expenses-application jar --module expenses/com.example.expenses.application.ExpensesApplication
```

자바 .class  파일을 실행할 때 다음과 같은 두가지 옵션이 새로 추가되었다.  
--module-path: 어떤 모듈을 로드할 수 잇는지 지정.
--module: 실행할 메인 모듈과 클래스를 지정.

### 14.5 여러 모듈 활용하기
자바 9에서는 여러 모듈을 export, requires로 사용한다.  

**export 구문**  
export는 다른 모듈에서 사용할 수 있도록 특정 패키지를 공개 형식으로 만든다.
```java
module expenses.readers {
    exports com.example.expenses.readers;
    exports com.example.expenses.readers.file;
    exports com.example.expenses.readers.http;
    // 패키지명
}
```

**requires 구문**  
requires는 의존하고 있는 모듈을 지정한다.
```java
module expenses.readers {
    requires java.base; // 모듈명

    exports com.example.expenses.readers;
    exports com.example.expenses.readers.file;
    exports com.example.expenses.readers.http;
    // 패키지명
}
```

### 14.6 컴파일과 패키징
프로젝트를 설정하고 모듈을 정의 했으니  
메이븐등의 빌드 도구를 이용해 프로젝트를 컴파일 할 수 있다.  

최상위 pom.xml에는 각 모듈을 정의해 준다.  

mvn clean package 명령을 실행해 프로젝트의 모듈을 JAR로 만든다.  
./expenses.application/target/expenses.application-1.0.jar  
./expenses.reader/ target/expenses.reader-1.0.jar

두 JAR을 모듈경로에 포함해서 모듈 애플리케이션을 실행
```
java --module-path \
./expenses.application/target/expenses.application-1.0.jar \
./expenses.reader/ target/expenses.reader-1.0.jar \
--moduel \
expenses.application/com.example.expenses.application.ExpensesApplication
```

자바 모듈 시스템을 설명하려면 한장이 아니라 책 한권을 써야한다.  
자세한 내용을 알고 싶으면 The Java Module System을 살펴보자. 
