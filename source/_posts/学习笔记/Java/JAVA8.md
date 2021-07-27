---
title: Java8
date: 2018-07-01
categories:
    - 学习
tags:
    - java
---

##### Lambada表达式

* **可选类型声明** - 无需声明参数的类型。编译器可以从该参数的值推断。
* **可选圆括号参数** - 无需在括号中声明参数。对于多个参数，括号是必需的。
* **可选大括号** - 表达式主体没有必要使用大括号，如果主体中含有一个单独的语句。
* **可选return关键字** - 编译器会自动返回值，如果主体有一个表达式返回的值。花括号是必需的，以表明表达式返回一个值。

<!-- more -->

###### 示例

```java
/**
 * lambda表达式
 * 定义方法骨架，实现由调用者实现
 */
public class LambdaDemo {

    //with type declaration (声明类型)
    private static final MathOperation addition = (int a, int b) -> a + b;
    //with out type declaration
    private static final MathOperation subtraction = (a, b) -> a - b;
    //with return statement along with curly braces （返回值）
    private static final MathOperation multiplication = (int a, int b) -> { return a * b; };
    //without return statement and without curly braces
    private static final MathOperation division = (int a, int b) -> a / b;

    interface MathOperation {
        int operation(int a, int b);
    }
    interface GreetingService {
        void sayMessage(String message);
    }
    private int operate(int a, int b, MathOperation mathOperation){
        return mathOperation.operation(a, b);
    }

    public static void main(String args[]){
        LambdaDemo tester = new LambdaDemo();
        System.out.println("10 + 5 = " + tester.operate(10, 5, addition));
        System.out.println("10 - 5 = " + tester.operate(10, 5, subtraction));
        System.out.println("10 x 5 = " + tester.operate(10, 5, multiplication));
        System.out.println("10 / 5 = " + tester.operate(10, 5, division));
        //with parenthesis（括号）
        GreetingService greetService1 = message -> System.out.println("Hello " + message);
        //without parenthesis
        GreetingService greetService2 = (message) -> System.out.println("Hello " + message);
        greetService1.sayMessage("Mahesh");
        greetService2.sayMessage("Suresh");
    }
}
```

###### 结果

```text
    10 + 5 = 15
    10 - 5 = 5
    10 x 5 = 50
    10 / 5 = 2
    Hello Mahesh
    Hello Suresh
```

* lambda表达式主要用于定义内联执行的功能的接口，即只有一个单一的方法接口。
* Lambda表达式消除匿名类的需求，并给出了一个非常简单但功能强大的函数式编程能力。

###### 变量的作用域

&emsp;&emsp; 在lambda表达式中，可以指任何最终的变量（final）或有效的最后一个变量

##### 方法引用

* 静态方法
* 实例方法
* 使用new运算符构造函数(TreeSet::new)

###### 示例

```java
public class MethdQuote {
    
    public static void main(String args[]){
        List<String> names = new ArrayList<>();
        names.add("Mahesh");
        names.add("Suresh");
        names.add("Ramesh");
        names.add("Naresh");
        names.add("Kalpesh");
        //names.forEach(System.out::println);
        GreetingService greetingService = new GreetingService();
        names.forEach(greetingService::sayMessage);
    }
}

class GreetingService {
    void sayMessage(String message){
        System.out.println(message);
    }
}
```

###### 结果

```text
    Mahesh
    Suresh
    Ramesh
    Naresh
    Kalpesh
```

##### 函数式接口

|序号| 接口名 | 说明 |
| --|:-------------:| -----|
| 1 | BiConsumer<T,U>   | 表示接收两个输入参数和不返回结果的操作。 |
| 2 | BiFunction<T,U,R> | 表示接受两个参数，并产生一个结果的函数。 |
| 3 | BinaryOperator\<T> | 表示在相同类型的两个操作数的操作，生产相同类型的操作数的结果。 |
| 4 | BiPredicate<T,U>  | 代表两个参数谓词（布尔值函数）。 |
| 5 | BooleanSupplier   | 代表布尔值结果的提供者。 |
| 6 | Consumer\<T> | 表示接受一个输入参数和不返回结果的操作。|
| 7 | DoubleBinaryOperator | 代表在两个double值操作数的运算，并产生一个double值结果。 |
| 8 | DoubleConsumer | 表示接受一个double值参数，不返回结果的操作。 |
| 9 | DoubleFunction\<R> | 表示接受double值参数，并产生一个结果的函数。 |
| 10 | DoublePredicate | 代表一个double值参数谓词（布尔值函数）。 |
| 11 | DoubleSupplier | 表示double值结果的提供者。 |
| 12 | DoubleToIntFunction | 表示接受double值参数，并产生一个int值结果的函数。|
| 13 | DoubleToLongFunction | 代表接受一个double值参数，并产生一个long值结果的函数。 |
| 14 | DoubleUnaryOperator | 表示上产生一个double值结果的单个double值操作数的操作。 |
| 15 | Function<T,R> | 表示接受一个参数，并产生一个结果的函数。 |
| 16 | IntBinaryOperator | 表示对两个int值操作数的运算，并产生一个int值结果。 |
| 17 | IntConsumer | 表示接受单个int值的参数并没有返回结果的操作。 |
| 18 | IntFunction\<R> | 表示接受一个int值参数，并产生一个结果的函数。 |
| 19 | IntPredicate | 表示一个整数值参数谓词（布尔值函数）。 |
| 20 | IntSupplier | 代表整型值的结果的提供者。 |
| 21 | IntToDoubleFunction | 表示接受一个int值参数，并产生一个double值结果的功能。 |
| 22 | IntToLongFunction | 表示接受一个int值参数，并产生一个long值结果的函数。 |
| 23 | IntUnaryOperator | 表示产生一个int值结果的单个int值操作数的运算。 |
| 24 | LongBinaryOperator | 表示在两个long值操作数的操作，并产生一个long值结果。 |
| 25 | LongConsumer | 表示接受一个long值参数和不返回结果的操作。 |
| 26 | LongFunction\<R>  | 表示接受long值参数，并产生一个结果的函数。 |
| 27 | LongPredicate | 代表一个long值参数谓词（布尔值函数）。|
| 28 | LongSupplier | 表示long值结果的提供者。 |
| 29 | LongToDoubleFunction | 表示接受double参数，并产生一个double值结果的函数。 |
| 30 | LongToIntFunction | 表示接受long值参数，并产生一个int值结果的函数。 |
| 31 | LongUnaryOperator | 表示上产生一个long值结果单一的long值操作数的操作。 |
| 32 | ObjDoubleConsumer\<T> | 表示接受对象值和double值参数，并且没有返回结果的操作。 |
| 33 | ObjIntConsumer\<T> | 表示接受对象值和整型值参数，并返回没有结果的操作。 |
| 34 | ObjLongConsumer\<T> | 表示接受对象的值和long值的说法，并没有返回结果的操作。 |
| 35 | Predicate\<T> | 代表一个参数谓词（布尔值函数）。 |
| 36 | Supplier\<T> | 表示一个提供者的结果。 |
| 37 | ToDoubleBiFunction<T,U> | 表示接受两个参数，并产生一个double值结果的功能。 |
| 38 | ToDoubleFunction\<T> | 代表一个产生一个double值结果的功能。 |
| 39 | ToIntBiFunction<T,U> | 表示接受两个参数，并产生一个int值结果的函数。 |
| 40 | ToIntFunction\<T> | 代表产生一个int值结果的功能。 |
| 41 | ToLongBiFunction<T,U> | 表示接受两个参数，并产生long值结果的功能。|
| 42 | ToLongFunction\<T> | 代表一个产生long值结果的功能。 |
| 43 | UnaryOperator\<T> | 表示上产生相同类型的操作数的结果的单个操作数的操作。 |

###### 示例

``` java
public class FunctionInterface {
    public static void main(String args[]) {

        List<Integer> list = Arrays.asList(1, 2, 3, 4, 5, 6);

        System.out.println("Print all numbers:");
        eval(list, n -> true);

        System.out.println("Print even numbers:");
        eval(list, n -> n % 2 == 0);

        System.out.println("Print numbers greater than 3:");
        eval(list, n -> n > 3);

        Map<Integer,Integer> map1 = new HashMap<>();
        Map<Integer,Integer> map2 = new HashMap<>();
        map1.put(1,2);
        map2.put(1,2);
        map1.put(2,2);
        map2.put(3,2);

        doubleValue(map1,map2,(x,y)-> x+y );
        doubleValue(map1,map2,(x,y)-> x-y );
        doubleValue(map1,map2,(x,y)-> x*y );
        doubleValue(map1,map2,(x,y)-> x/y );
    }

    public static Map<Integer,Integer> doubleValue(Map<Integer,Integer> map1, Map<Integer,Integer> map2, BiFunction<Integer,Integer,Integer> biFunction) {
        Map<Integer,Integer> result = new HashMap<>();
        map1.forEach((key,vaule)->{
            if(map2.containsKey(key)){
                result.put(key,biFunction.apply(map2.get(key),vaule)) ;
            }
        });
        result.forEach( (x,y)-> System.out.println("x:"+x+",y:"+y) );
        return result;
    }

    public static void eval(List<Integer> list, Predicate<Integer> predicate) {
        for (Integer n : list) {
            if (predicate.test(n)) {
                System.out.println(n + " ");
            }
        }
    }
}
```

###### 结果

``` text
    Print all numbers:
    1 
    2 
    3 
    4 
    5 
    6 
    Print even numbers:
    2 
    4 
    6 
    Print numbers greater than 3:
    4 
    5 
    6 
    x:1,y:4
    x:1,y:0
    x:1,y:4
    x:1,y:1
```

##### 默认方法

* 多重默认，使用超指定接口的默认方法
* 静态默认方法

###### 示例

``` java
public class DefaultFunction implements Vehicle,FourWheeler{

    public static void main(String args[]) {
        DefaultFunction defaultFunction = new DefaultFunction();
        defaultFunction.print();
    }
    @Override
    public void print() {
        Vehicle.super.print();
        Vehicle.blowHorn();
    }
}

interface Vehicle {
    default void print(){
        System.out.println("I am a vehicle!");
    }
    //静态默认方法
    static void blowHorn(){
        System.out.println("Blowing horn!!!");
    }

}

interface FourWheeler {
    default void print(){
        System.out.println("I am a four wheeler!");
    }
}
```

###### 结果

``` text
    I am a vehicle!
    Blowing horn!!!
```

##### 数据流

* 元素序列 - 流提供了一组特定类型的以顺序方式元素。流获取/计算需求的元素。它不存储元素。
* 源- 流使用集合，数组或I/O资源为输入源。
* 聚合操作 - 数据流支持如filter, map, limit, reduced, find, match等聚合操作。
* 管道传输 - 大多数流操作的返回流本身使他们的结果可以被管道传输。这些操作被称为中间操作以及它们的功能是利用输入，处理输入和输出返回到目标。collect()方法是终端操作，这是通常出现在管道传输操作结束标记流的结束。
* 自动迭代 - 流操作内部做了反复对比，其中明确迭代需要集合提供源元素。

1. 生成数据流
    * stream() -返回顺序流考虑集合作为其源。
    * parallelStream() - 返回并行数据流考虑集合作为其源。
2. forEach 遍历每个元素
3. map 映射每个元素对应的结果
4. filter 过滤基于标准元素
5. limit 减少流的大小
6. sorted 排序
7. collect（收集器）用来处理组合在一个数据流的元素的结果。收集器可用于返回一个列表或一个字符串。
8. summaryStatistics（统计）统计收集器引入计算所有统计数据

###### 示例

```java
public class StreamDemo {

    public static void main(String[] args){
        //ForEach
        Random random = new Random();
        random.ints(3,5,20).forEach(System.out::println);
        //map 对每个元素的操作
        List<Integer> numbers = Arrays.asList(3, 2, 2, 3, 7, 3, 5);
        List<Integer> squaresList = numbers.stream().map( i -> i*i).distinct().collect(Collectors.toList());
        squaresList.stream().forEach(System.out::println);
        //filter
        List<String>strings = Arrays.asList("abc", "", "bc", "efg", "abcd","", "jkl");
        Long count = strings.stream().filter(string -> string.isEmpty()).count();
        System.out.println(count);
        //limit方法用于减少流的大小
        random.ints(5,20).limit(3).forEach(System.out::println);
        //sorted
        random.ints(5,20).limit(3).sorted().forEach(System.out::println);
        //并行处理 parallelStream
        strings = Arrays.asList("abc", "", "bc", "efg", "abcd","", "jkl");
        count = strings.parallelStream().filter(string -> string.isEmpty()).count();
        System.out.println(count);
        //收集器collect | Collectors
        strings = Arrays.asList("abc", "", "bc", "efg", "abcd","", "jkl");
        List<String> filtered = strings.stream().filter(string -> !string.isEmpty()).collect(Collectors.toList());
        System.out.println("Filtered List: " + filtered);
        String mergedString = strings.stream().filter(string -> !string.isEmpty()).collect(Collectors.joining(", "));
        System.out.println("Merged String: " + mergedString);
        //统计
        numbers = Arrays.asList(3, 2, 2, 3, 7, 3, 5);
        IntSummaryStatistics stats = numbers.stream().mapToInt((x) -> x).summaryStatistics();
        System.out.println("Highest number in List : " + stats.getMax());
        System.out.println("Lowest  number in List : " + stats.getMin());
        System.out.println("Sum of all numbers : " + stats.getSum());
        System.out.println("Average of all  numbers : " + stats.getAverage());
        
        testStream();
    }

    public static void testStream(){
        List<String> strings = Arrays.asList("abc", "", "bc", "efg", "abcd","", "jkl");
        List<String> filtered = strings.stream().filter(string -> StringUtils.isNotEmpty(string)).collect(Collectors.toList());
        filtered.stream().forEach(System.out::println);
    }
}
```

###### 运行结果

```text
    8
    7
    7
    9
    4
    49
    25
    2
    19
    16
    8
    10
    10
    11
    2
    Filtered List: [abc, bc, efg, abcd, jkl]
    Merged String: abc, bc, efg, abcd, jkl
    Highest number in List : 7
    Lowest  number in List : 2
    Sum of all numbers : 25
    Average of all  numbers : 3.5714285714285716
    abc
    bc
    efg
    abcd
    jkl
```

##### Optional类

&emsp;&emsp; Optional用于包含非空对象的容器对象

###### 示例

```java
public class OptionalDemo {

   public static void main(String args[]){
      OptionalDemo optionalDemo = new OptionalDemo();
      Integer value1 =  null;
      Integer value2 =  new Integer(10);
      //Optional.ofNullable - allows passed parameter to be null.
      Optional<Integer> a = Optional.ofNullable(value1);
      //Optional.of - throws NullPointerException if passed parameter is null
      Optional<Integer> b = Optional.of(value2);
      System.out.println(optionalDemo.sum(a,b));
   }

   public Integer sum(Optional<Integer> a, Optional<Integer> b){
      //Optional.isPresent - checks the value is present or not
      System.out.println("First parameter is present: " + a.isPresent());
      System.out.println("Second parameter is present: " + b.isPresent());
      //Optional.orElse - returns the value if present otherwise returns
      //the default value passed.
      Integer value1 = a.orElse(new Integer(0));
      //Optional.get - gets the value, value should be present
      Integer value2 = b.get();
      return value1 + value2;
   }
}
```

###### 结果

```text
    First parameter is present: false
    Second parameter is present: true
    10
```

##### 时间

* LocalDate/LocalDateTime | 本地日期时间API
* ZonedDateTime | 时区日期时间API
* ChronoUnit | 计时单位枚举
* Period/Duration | 时间间隔
* TemporalAdjuster | 时间调节器
* toInstant() | 向后兼容性

###### LocalDate示例

```java
/**
 * 本地时间
 */
public class LocalDateApi {

   public static void main(String args[]){
      LocalDateApi localDateApi = new LocalDateApi();
      localDateApi.testLocalDateTime();
   }
   public void testLocalDateTime(){
      // Get the current date and time
      LocalDateTime currentTime = LocalDateTime.now();     
      System.out.println("Current DateTime: " + currentTime);
      System.out.println("date0: " + currentTime.toLocalTime());
      LocalDate date1 = currentTime.toLocalDate();
      System.out.println("date1: " + date1);
      Month month = currentTime.getMonth();
      int day = currentTime.getDayOfMonth();
      int seconds = currentTime.getSecond();
      System.out.println("Month: " + month
         +",day: " + day
         +",seconds: " + seconds
      );
      LocalDateTime date2 = currentTime.withDayOfMonth(10).withYear(2012);
      System.out.println("date2: " + date2);
      //12 december 2014
      LocalDate date3 = LocalDate.of(2014, Month.DECEMBER, 12); 
      System.out.println("date3: " + date3);
      //22 hour 15 minutes
      LocalTime date4 = LocalTime.of(22, 15);
      System.out.println("date4: " + date4);
      //22 hour 15 minutes 10秒
      LocalTime date5 = LocalTime.of(22, 15,10);
      System.out.println("date5: " + date5);
      //parse a string
      LocalTime date6 = LocalTime.parse("20:15:30");
      System.out.println("date6: " + date6);
   }
}
```

###### LocalDate结果

``` shell
    Current DateTime: 2018-07-01T22:23:51.761
    date0: 22:23:51.761
    date1: 2018-07-01
    Month: JULY,day: 1,seconds: 51
    date2: 2012-07-10T22:23:51.761
    date3: 2014-12-12
    date4: 22:15
    date5: 22:15:10
    date6: 20:15:30
```

###### ZonedDateTime示例

```java
/**
 * 时区时间
 */
public class ZonedDateTimeApi {

   public static void main(String args[]){
      ZonedDateTimeApi zonedDateTimeApi = new ZonedDateTimeApi();
      zonedDateTimeApi.testZonedDateTime();
   }

   public void testZonedDateTime(){
      // Get the current date and time
      ZonedDateTime date1 = ZonedDateTime.parse("2007-12-03T10:15:30+05:30[Asia/Karachi]");
      System.out.println("date1: " + date1);
      ZoneId id = ZoneId.of("Europe/Paris");
      System.out.println("ZoneId: " + id);
      ZoneId currentZone = ZoneId.systemDefault();
      System.out.println("CurrentZone: " + currentZone);
   }
}
```

###### ZonedDateTime结果

```text
    date1: 2007-12-03T10:15:30+05:00[Asia/Karachi]
    ZoneId: Europe/Paris
    CurrentZone: Asia/Shanghai
```

###### Period | Duration示例

```java
/**
 * 计算时间间隔
 */
public class PeriodAndDurationApi {

   public static void main(String args[]){
      PeriodAndDurationApi periodAndDurationApi = new PeriodAndDurationApi();
      periodAndDurationApi.testPeriod();
      periodAndDurationApi.testDuration();
   }

   public void testPeriod(){
      //Get the current date
      LocalDate date1 = LocalDate.now();
      System.out.println("Current date: " + date1);
      //add 1 month to the current date
      LocalDate date2 = date1.plus(1, ChronoUnit.YEARS)
              .plus(1, ChronoUnit.MONTHS)
              .plus(1, ChronoUnit.DAYS);
      System.out.println("Next month: " + date2);
      Period period = Period.between(date1, date2);
      System.out.println("Period: " + period);
      System.out.println("Period: " + period.getYears());
      System.out.println("Period: " + period.getMonths());
      System.out.println("Period: " + period.getDays());
   }

   public void testDuration(){
      LocalTime time1 = LocalTime.now();
      Duration twoHours = Duration.ofHours(2);
      //有隔天问题
      LocalTime time2 = time1.plus(twoHours);
      Duration duration = Duration.between(time1, time2);
      System.out.println("Duration: " + duration);
      System.out.println("Duration: " + duration.getSeconds());
   }
}
```

###### Period | Duration结果

```text
    Current date: 2018-07-01
    Next month: 2019-08-02
    Period: P1Y1M1D
    Period: 1
    Period: 1
    Period: 1
    Duration: PT1H
    Duration: 7200 
```

###### ChromoUnits示例

```java
/**
 * 时区枚举ChromoUnits
 */
public class ChromoUnitsApi {

   public static void main(String args[]){
      ChromoUnitsApi chromoUnitsApi = new ChromoUnitsApi();
      chromoUnitsApi.testChromoUnits();
   }

   public void testChromoUnits(){
      //Get the current date
      LocalDate today = LocalDate.now();
      System.out.println("Current date: " + today);
      //add 1 week to the current date
      LocalDate nextWeek = today.plus(1, ChronoUnit.WEEKS);
      System.out.println("Next week: " + nextWeek);
      //add 1 month to the current date
      LocalDate nextMonth = today.plus(1, ChronoUnit.MONTHS);
      System.out.println("Next month: " + nextMonth);
      //add 1 year to the current date
      LocalDate nextYear = today.plus(1, ChronoUnit.YEARS);
      System.out.println("Next year: " + nextYear);
      //add 10 years to the current date
      LocalDate nextDecade = today.plus(1, ChronoUnit.DECADES);
      System.out.println("Date after ten year: " + nextDecade);
   }
}
```

###### ChromoUnits结果

```text
    Current date: 2018-07-01
    Next week: 2018-07-08
    Next month: 2018-08-01
    Next year: 2019-07-01
    Date after ten year: 2028-07-01
```

###### TemporalAdjusters示例

```java
/**
 * TemporalAdjusters 是做日期数学计算
 */
public class AdjustersApi {

   public static void main(String args[]){
      AdjustersApi adjustersApi = new AdjustersApi();
      adjustersApi.testAdjusters();
   }

   public void testAdjusters(){
      //Get the current date
      LocalDate date1 = LocalDate.now();
      System.out.println("Current date: " + date1);
      //get the next tuesday
      LocalDate nextTuesday = date1.with(TemporalAdjusters.next(DayOfWeek.TUESDAY));
      System.out.println("Next Tuesday on : " + nextTuesday);
      //get the second saturday of next month
      LocalDate firstInYear = LocalDate.of(date1.getYear(),date1.getMonth(), 1);
      LocalDate secondSaturday = firstInYear.with(
              TemporalAdjusters.nextOrSame(DayOfWeek.SATURDAY)).with(
              TemporalAdjusters.next(DayOfWeek.SATURDAY));
      System.out.println("Second saturday on : " + secondSaturday);
      System.out.print(firstInYear.with(TemporalAdjusters.firstDayOfNextYear()));
   }
}
```

###### TemporalAdjusters结果

```text
    Current date: 2018-07-01
    Next Tuesday on : 2018-07-03
    Second saturday on : 2018-07-14
    2019-01-01
```

###### toInstant()示例

```java
/**
 * 新旧日期兼容 toInstant()
 */
public class BackwardCompatability {

   public static void main(String args[]){
      BackwardCompatability backwardCompatability = new BackwardCompatability();
      backwardCompatability.testBackwardCompatability();
   }

   public void testBackwardCompatability(){
      //Get the current date
      Date currentDate = new Date();
      System.out.println("Current date: " + currentDate);
      //Get the instant of current date in terms of milliseconds
      Instant now = currentDate.toInstant();
      ZoneId currentZone = ZoneId.systemDefault();
      LocalDateTime localDateTime = LocalDateTime.ofInstant(now, currentZone);
      System.out.println("Local date: " + localDateTime);
      ZonedDateTime zonedDateTime = ZonedDateTime.ofInstant(now, currentZone);
      System.out.println("Zoned date: " + zonedDateTime);
   }
}
```

###### toInstant()结果

```text
    Current date: Sun Jul 01 22:34:20 CST 2018
    Local date: 2018-07-01T22:34:20.804
    Zoned date: 2018-07-01T22:34:20.804+08:00[Asia/Shanghai]
```

##### Base64

* 简单 - 输出映射设置字符在A-ZA-Z0-9+/。编码器不添加任何换行输出和解码器拒绝在A-Za-z0-9+/以外的任何字符。
* URL - 输出映射设置字符在A-Za-z0-9+_。输出URL和文件名安全。
* MIME - 输出映射到MIME友好的格式。输出表示在每次不超过76个字符行和使用'\r'后跟一个换行符'\n'回车作为行分隔符。无行隔板的存在是为了使编码的结束输出。

###### Base64.Decoder(解码器) | Base64.Encoder(编码器) 方法

|序号| 接口名 | 说明 |
| --|:-------------:| -----|
| 1 | static Base64.Decoder getDecoder() |返回Base64.Decoder解码使用基本型base64编码方案。|
| 2 | static Base64.Encoder getEncoder() |返回Base64.Encoder编码使用的基本型base64编码方案。|
| 3 | static Base64.Decoder getMimeDecoder() |返回Base64.Decoder解码使用MIME类型的base64解码方案。|
| 4 | static Base64.Encoder getMimeEncoder() |返回Base64.Encoder编码使用MIME类型base64编码方案。|
| 5 | static Base64.Encoder getMimeEncoder(int lineLength, byte[] lineSeparator)        |返回Base64.Encoder编码使用指定的行长度和线分隔的MIME类型base64编码方案。|
| 6 | static Base64.Decoder getUrlDecoder() |返回Base64.Decoder解码使用URL和文件名安全型base64编码方案。|
| 7 | static Base64.Encoder getUrlEncoder() |返回Base64.Decoder解码使用URL和文件名安全型base64编码方案。|

###### 示例

```java
public class Base64Demo {

   public static void main(String args[]){
      try {
         // Encode using basic encoder
         String base64encodedString = Base64.getEncoder().encodeToString("YiiBai?java8".getBytes("utf-8"));
         System.out.println("Base64 Encoded String (Basic) :" + base64encodedString);
         // Decode
         byte[] base64decodedBytes = Base64.getDecoder().decode(base64encodedString);
         System.out.println("Original String: "+new String(base64decodedBytes, "utf-8"));
         
         base64encodedString = Base64.getUrlEncoder().encodeToString("YiiBai?java8".getBytes("utf-8"));
         System.out.println("Base64 Encoded String (URL) :" + base64encodedString);
         StringBuilder stringBuilder = new StringBuilder();
         for (int i = 0; i < 10; ++i) {
            stringBuilder.append(UUID.randomUUID().toString());
         }

         byte[] mimeBytes = stringBuilder.toString().getBytes("utf-8");
         String mimeEncodedString = Base64.getMimeEncoder().encodeToString(mimeBytes);
            System.out.println("Base64 Encoded String (MIME) :"+mimeEncodedString);
      }catch(UnsupportedEncodingException e){
         System.out.println("Error :"+e.getMessage());
      }
   }
}
```

###### 结果

```text
    Base64 Encoded String (Basic) :WWlpQmFpP2phdmE4
    Original String: YiiBai?java8
    Base64 Encoded String (URL) :WWlpQmFpP2phdmE4
    Base64 Encoded String (MIME) :MjdkM2ZiNGItOTVkNS00YzI5LTg5MWUtYTJhOGJlODMyMTUxNTQ1YjlmMmEtYTc3YS00NzBlLWE5
    Y2UtZjIzYzdjODI3M2ZmZGM4ODRlZmMtYWViNS00YTNiLTg5NGEtMTRjZDI4ZGY4ZDc5NzI2ZDA5
    NTktNjQyYi00NTRjLWIxY2UtYzM3MjI1YzQxNTViZTUxYjUwZDQtOTk4Yy00MGY4LWIxOGMtNWJl
    OGE0M2EzYzJjOTlmMmUwZjMtMmM0NS00YjkzLTlmYWItN2RkNTljNDZkYmU1YTU3Y2I4ODYtMDNi
    NS00YjUyLTk4ZTMtYTZmMmQ2ZTljMDg2Zjk4OGUxZGEtMDM3OC00NmY3LThjZDctODVhMDc0NzRk
    YThlY2I1YzYwZTItMGI3ZS00YTQ4LWI5YzAtMjI0MTFlMmMxYmU2MTdmMTJhZjgtMTRiZC00NGZj
    LWJmYzItODM5NTczMTVlNGVk
```
