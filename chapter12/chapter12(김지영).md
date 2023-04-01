## Chapter 12 새로운 날짜와 시간 API
자바 1.0에서는 java.util.Date 클래스 하나로 날짜와 시간 관련 기능을 제공했다.  
Date 클래스는 날짜가 아닌 밀리토 단위로 표현되어 직관적이지 않았다.  
또한 toString으로 반환되는 문자열을 추가로 활용하기 어려웠다.  
Date 클래스는 JVM의 기본 시간대인 중앙 유럽 시간을 사용했다.  


이 장에서는 새로운 날짜와 시간 API가 제공하는 기능을 알아본다.  
사람과 기기에서 사용할 수 있는 날짜와 시간을 생성하는 기본적인 방법과  
날짜 시간 객체를 조작, 파싱, 출력하고 다양한 시간대와 대안 캘린더 등을 사용하는 방법을 살펴본다.  

### 12.1 LocalDate, LocalTime, Instant, Duration, Period 클래스

**LocalDate와 LocalTime**
LocalDate 인스턴스는 시간을 제외한 날짜를 표현하는 불변 객체다.  
LocalDate 객체는 어떤 시간대 정보도 포함하지 않는다.  
정적 팩토리 메서드 of으로 LocalDate 인스턴스를 만들 수 있다.
```java
    LocalDate date = LocalDate.of(2017, 9, 21); // of
    int year = date.getYear();
    Month month = date.getMonth();
    int day = date.getDayOfMonth();
    LocalDate now = LocalDate.now(); // 현재 날짜 정보
```
시간에 대한 정보는 LocalTime 클래스로 표현할 수 있다.  
LocalTime도 정적 메서드 of로 인스턴스를 만들 수 있다.
```java
    LocalTime time = LocalTime.of(13, 45, 20); // 13:45:20
    int hour = time.getHour();
    int minute = time.getMinute();
    int second = time.getSecond();
```

**날짜와 시간 조합**  
LocalDateTime은 LocalDate와 LocalTime을 쌍으로 갖는 복합 클래스다.
```java
    LocalDateTime dt1 = LocalDateTime.of(2017, Month.SEPTEMBER, 21, 13, 45, 20);
    LocalDate date = dt1.toLocalDate();
    LocalTime time = dt1.toLocalTime();
    LocalDateTime dt2 = LocalDateTime.of(date, time); // 2017-09-21T13:45:20
    LocalDateTime dt3 = date.atTime(13,45,20); // 2017-09-21T13:45:20
    LocalDateTime dt4 = date.atTime(time); // 2017-09-21T13:45:20
    LocalDateTime dt5 = time.atDate(date); // 2017-09-21T13:45:20
```

**Instant 클래스 : 기계의 날짜와 시간**  
java.time.Instant 클래스에서는 기계적인 관점에서 시간을 표현한다.  
nstant 클래스는 유닉스 에포크 시간(Unix epoch time) (1970년 1월 1일 0시 0분 0초 UTC)을 기준으로 특정 지점까지의 시간을 초로 표현한다.  
Instant 클래스도 사람이 확인할 수 있도록 시간을 표현해주는 정적 팩토리 메서드 now를 제공한다. 하지만 사람이 읽을 수 있는 시간정보는 제공하지 않는다.   

**Duration과 Period 정의**  
두 시간 객체 사이의 지속시간을 알고 싶을 때 사용 한다.    
```java
    LocalTime time1 = LocalTime.of(12, 00, 00);
    LocalTime time2 = LocalTime.of(12, 10, 30);
    Duration duration = Duration.between(time1, time2);
    System.out.println("duration = " + duration.getSeconds()); // 630

    LocalDate start = LocalDate.of(2017, 9, 11);
    LocalDate end = LocalDate.of(2017, 9, 21);
    Period period = Period.between(start, end);
    System.out.println("period = " + period.get(ChronoUnit.DAYS)); // period = 10
    
    Duration threeMinutes = Duration.ofMinutes(3); // PT3M
    long seconds = threeMinutes.getSeconds(); // 180
    Duration threeMinutes2 = Duration.of(3, ChronoUnit.MINUTES); //PT3M

    Period thenDays = Period.ofDays(10); // P10D
    Period threeWeeks = Period.ofWeeks(3); // P21D
    Period twoYearsSixMonthsOneDay = Period.of(2,6,1); //P2Y6M1D
```

### 12.2 날짜 조정, 파싱, 포매팅
- 절대적인 방식으로 LocalDated의 속성 바꾸기
```java
    LocalDate date1 = LocalDate.of(2017,9,21); // 2017-09-21
    LocalDate date2 = date1.withYear(2011); // 2011-09-21
    LocalDate date3 = date2.withDayOfMonth(25); // 2011-09-25
    LocalDate date4 = date3.with(ChronoField.MONTH_OF_YEAR,2); // 2011-02-25

```
- 상대적인 방식으로 LocalDate 속성 바꾸기
```
    LocalDate date2_1 = date1.plusWeeks(1); // 2017-09-28
    LocalDate date3_1 = date2_1.minusYears(6); // 2011-09-28
    LocalDate date4_1 = date3_1.plus(6, ChronoUnit.MONTHS); // 2012-03-28
```

**TemporalAdjusters 사용하기**  
다음 주 일요일, 돌아오는 평일, 어떤 달의 마지막 날 등의 좀더 복잡한 날짜 조정
```java
    import static java.time.temporal.TemporalAdjusters.lastDayOfMonth;
    import static java.time.temporal.TemporalAdjusters.nextOrSame;

    LocalDate date1 = LocalDate.now(); // 2023-04-01
    LocalDate date2 = date1.with(nextOrSame(DayOfWeek.SUNDAY)); // 2023-04-02
    LocalDate date3 = date2.with(lastDayOfMonth()); // 2023-04-30
```
참조 p399 표 12-3 TemporalAdjusters 클래스의 팩토리 메서드
https://docs.oracle.com/javase/8/docs/api/java/time/temporal/TemporalAdjusters.html

TemporalAdjuster 커스텀 구현을 간단하게 만들수도 있다.  
하나의 메서드만 정의하므로 함수형 인터페이스이다.  
```java
@FunctionalInterface
public interface TemporalAdjuster {
    Temporal adjustInto(Temporal temporal);
}
```
**날짜와 시간 객체 출력과 파싱**  
포매팅과 파싱 전용 패키지 java.time.format
```java
    LocalDate date = LocalDate.of(2014,3,18);
    String d1 = date.format(DateTimeFormatter.BASIC_ISO_DATE); // 20140318
    String d2 = date.format(DateTimeFormatter.ISO_LOCAL_DATE); // 2014-03-18
```
DateTimeFormatter의 정적팩토리 메서드 이용
```
    LocalDate date1 = LocalDate.parse("20140318", DateTimeFormatter.BASIC_ISO_DATE);
    LocalDate date2 = LocalDate.parse("20140318", DateTimeFormatter.ISO_LOCAL_DATE);
```
패턴으로 DateTimeFormatter 만들기
```java
    DateTimeFormatter formatter = DateTimeFormatter.ofPattern("dd/MM/yyyy");
    LocalDate date1 = LocalDate.of(2014,3,18);
    String formattedDate = date1.format(formatter);
    LocalDate date2 = LocalDate.parse(formattedDate, formatter);
```
지역화된 DateTimeFormatter 만들기
```java
    DateTimeFormatter italianFormatter = DateTimeFormatter.ofPattern("d. MMMM yyyy", Locale.ITALIAN);
    LocalDate date3 = LocalDate.of(2014,3,18);
    String formattedDate1 = date3.format(italianFormatter); // 18. marzo 2014
    LocalDate date4 = LocalDate.parse(formattedDate1, italianFormatter);
```
DateTimeFormatter 만들기
```java
    DateTimeFormatter italianFormatter = new DateTimeFormatterBuilder()
        .appendText(ChronoField.DAY_OF_MONTH)
        .appendLiteral(". ")
        .appendText(ChronoField.MONTH_OF_YEAR)
        .appendLiteral("")
        .appendText(ChronoField.YEAR)
        .parseCaseInsensitive()
        .toFormatter(Locale.ITALIAN);
```

### 12.3 다양한 시간대와 캘린더 활용방법
**시간대 사용하기**  
표준시간이 같은 지역을 묶어 time zone 규칙 집합 정의한다.  
참고 : https://www.iana.org/time-zones   

특정 시점에 시간대 적용
```java
    ZoneId romeZone = ZoneId.of("Europe/Rome"); // {지역}/{도시}

    LocalDate date = LocalDate.of(2014, Month.MARCH, 18);
    ZonedDateTime zdt1 = date.atStartOfDay(romeZone); // 2014-03-18T00:00+01:00[Europe/Rome]
    LocalDateTime datetime = LocalDateTime.of(2014, Month.MARCH, 18, 13, 45);
    ZonedDateTime zdt2 = datetime.atZone(romeZone); // 2014-03-18T13:45+01:00[Europe/Rome]
    Instant instant = Instant.now();
    ZonedDateTime zdt3 = instant.atZone(romeZone); // 023-04-01T12:14:03.017491+02:00[Europe/Rome]
```

ZoneId를 이용해서 LocalDateTime을 Instant로 바꾸기
```java
    Instant instant1 = Instant.now();
    LocalDateTime timeFromInstant = LocalDateTime.ofInstant(instant1, romeZone); // 2023-04-01T12:14:47.218896
```

**UTC/Greenwich 기준의 고정 오프셋**  
UTC 협정 세계시 / GMT 그리니치 표준시 를 기준으로 시간대를 표현하기도 한다.  
예시일 뿐 권장하지 않는 방식이다.
```java
// 뉴욕은 런던보다 5시간 느리다.
// 실제 미국 동부 표준시의 오프셋 값 "-05:00"이다.
ZoneOffset newYorkOffset = ZoneOffset.of("-05:00");

LocalDateTime date = LocalDateTime.of(2014, Month.MARCH, 18, 13, 45);
OffsetDateTime dateTimeInNewYork = OffsetDateTime.of(date, newYorkOffset);
```

**대안 캘린더 시스템 사용하기**  
자바8에서 추가된 4개의 캘린터 시스템  
ThaiBuddhistDate, MinguoDate, JapaneseDate, HijrahDate  

프로그램 입출력을 지역화하는 상황을 제외하고는  
모든 데이터 저장, 조작, 비지니스 규칙해석 등의 작업에서 LocalDate를 사용해야 한다.  