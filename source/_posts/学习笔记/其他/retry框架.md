---
title: Retry框架对 SpringRetry与GuavaRetry
date: 2022-08-10
categories:
    - 学习
tags:
    - SpringRetry
    - GuavaRetry
---
### 重试框架之SpringRetry
SpringRetry 为 Spring 应用程序提供了声明性重试支持。它用于Spring批处理、Spring集成、Apache Hadoop(等等)。它主要是针对可能抛出异常的一些调用操作，进行有策略的重试

<!-- more -->

#### SpringRetry的普通使用方式

* 添加依赖
```xml
<dependency>
  <groupId>org.springframework.retry</groupId>
  <artifactId>spring-retry</artifactId>
  <version>1.3.3</version>
</dependency>
<dependency>
  <groupId>org.aspectj</groupId>
  <artifactId>aspectjweaver</artifactId>
</dependency>
```

* 添加任务方法,我这里是采用一个随机整数，根据不同的条件返回不同的值，或者抛出异常
```java
@Slf4j
public class RetryDemoTask {
  /**
   * 重试方法
   * @return
   */
  public static boolean retryTask(String param) {
    log.info("收到请求参数:{}",param);
    int i = 0;
    try {
      i = SecureRandom.getInstanceStrong().nextInt(11);
    } catch (NoSuchAlgorithmException e) {
      throw new RemoteAccessException("获取随机数失败");
    }
    log.info("随机生成的数:{}",i);
    if (i == 0) {
      log.info("为0,抛出参数异常.");
      throw new IllegalArgumentException("参数异常");
    }else if (i  == 1){
      log.info("为1,返回true.");
      return true;
    }else if (i == 2){
      log.info("为2,返回false.");
      return false;
    }else{
      //为其他
        log.info("大于2,抛出自定义异常.");
        throw new RemoteAccessException("大于2,抛出远程访问异常");
      }
    }
}
```

* 使用SpringRetryTemplate
```java
@Slf4j
public class SpringRetryTemplateTest {

  /**
   * 重试间隔时间ms,默认1000ms
   * */
  private long fixedPeriodTime = 1000L;
  /**
   * 最大重试次数,默认为3
   */
  private int maxRetryTimes = 3;
  /**
   * 表示哪些异常需要重试,key表示异常的字节码,value为true表示需要重试
   */
  private Map<Class<? extends Throwable>, Boolean> exceptionMap = new HashMap<>();

  @Test
  public void test() {
    exceptionMap.put(RemoteAccessException.class,true);

    // 构建重试模板实例
    RetryTemplate retryTemplate = new RetryTemplate();

    // 设置重试回退操作策略，主要设置重试间隔时间
    FixedBackOffPolicy backOffPolicy = new FixedBackOffPolicy();
    backOffPolicy.setBackOffPeriod(fixedPeriodTime);

    // 设置重试策略，主要设置重试次数
    SimpleRetryPolicy retryPolicy = new SimpleRetryPolicy(maxRetryTimes, exceptionMap);

    retryTemplate.setRetryPolicy(retryPolicy);
    retryTemplate.setBackOffPolicy(backOffPolicy);

    Boolean execute = retryTemplate.execute(
            //RetryCallback
            retryContext -> {
              boolean b = RetryDemoTask.retryTask("abc");
              log.info("调用的结果:{}", b);
              return b;
            },
            retryContext -> {
              //RecoveryCallback
              log.info("已达到最大重试次数或抛出了不重试的异常~~~");
              return false;
            }
      );
    log.info("执行结果:{}",execute);
  }

}
```

retryTemplate 承担了重试执行者的角色，它可以设置SimpleRetryPolicy(重试策略，设置重试上限，重试的根源实体)，FixedBackOffPolicy（固定的回退策略，设置执行重试回退的时间间隔）。

RetryTemplate通过execute提交执行操作，需要准备RetryCallback 和RecoveryCallback 两个类实例，RetryCallback对应的就是重试回调逻辑实例，包装正常的功能操作，RecoveryCallback实现的是整个执行操作结束的恢复操作实例.只有在调用的时候抛出了异常，并且异常是在exceptionMap中配置的异常，才会执行重试操作，否则就调用到excute方法的第二个执行方法RecoveryCallback中。

当然,重试策略还有很多种,回退策略也是:

**重试策略**
* **NeverRetryPolicy：**    只允许调用RetryCallback一次，不允许重试
* **AlwaysRetryPolicy：**   允许无限重试，直到成功，此方式逻辑不当会导致死循环
* **SimpleRetryPolicy：**   固定次数重试策略，默认重试最大次数为3次，**RetryTemplate默认使用的策略**
* **TimeoutRetryPolicy：**  超时时间重试策略，默认超时时间为1秒，在指定的超时时间内允许重试
* **ExceptionClassifierRetryPolicy：**  设置不同异常的重试策略，类似组合重试策略，区别在于这里只区分不同异常的重试
* **CircuitBreakerRetryPolicy：** 有熔断功能的重试策略，需设置3个参数openTimeout、resetTimeout和delegate
* **CompositeRetryPolicy：**  组合重试策略，有两种组合方式，乐观组合重试策略是指只要有一个策略允许即可以重试，悲观组合重试策略是指只要有一个策略不允许即可以重试，但不管哪种组合方式，组合中的每一个策略都会执行

**重试回退策略**  重试回退策略，指的是每次重试是立即重试还是等待一段时间后重试。**默认情况下是立即重试**，如果需要配置等待一段时间后重试则需要指定回退策略BackoffRetryPolicy。
* **NoBackOffPolicy：** 无退避算法策略，每次重试时立即重试
* **FixedBackOffPolicy：**  固定时间的退避策略，需设置参数sleeper和backOffPeriod，sleeper指定等待策略，默认是Thread.sleep，即线程休眠，backOffPeriod指定休眠时间，默认1秒
* **UniformRandomBackOffPolicy：**  随机时间退避策略，需设置sleeper、minBackOffPeriod和maxBackOffPeriod，该策略在minBackOffPeriod,maxBackOffPeriod之间取一个随机休眠时间，minBackOffPeriod默认500毫秒，maxBackOffPeriod默认1500毫秒
* **ExponentialBackOffPolicy：**  指数退避策略，需设置参数sleeper、initialInterval、maxInterval和multiplier，initialInterval指定初始休眠时间，默认100毫秒，maxInterval指定最大休眠时间，默认30秒，multiplier指定乘数，即下一次休眠时间为当前休眠时间*multiplier
* **xponentialRandomBackOffPolicy：**   随机指数退避策略，引入随机乘数可以实现随机乘数回退

#### SpringRetry的注解使用方式

* 添加依赖
```xml
<dependency>
  <groupId>org.springframework.retry</groupId>
  <artifactId>spring-retry</artifactId>
  <version>1.3.3</version>
</dependency>
<dependency>
  <groupId>org.aspectj</groupId>
  <artifactId>aspectjweaver</artifactId>
</dependency>
```

* 在application启动类上加上@EnableRetry的注解
```java
@EnableRetry
public class Application {
 ...
}
```

* 为了方便测试，我这里写了一个SpringBootTest的测试基类，需要使用SpringBootTest的只要继承这个类就好了
```java
@RunWith(SpringRunner.class)
@SpringBootTest(classes = SpringRetryApplication.class)
@Slf4j
public class MyBaseTest {


  @Before
  public void init() {
    log.info("----------------测试开始---------------");
  }

  @After
  public void after() {
    log.info("----------------测试结束---------------");
  }

}
```

* 我们只要在需要重试的方法上加@Retryable，在重试失败的回调方法上加@Recover，下面是这些注解的属性

* **@EnableRetry** 是否开启重试
  * proxyTargetClass： 默认值false, 表示是否创建基于子类的cglib代理，而不是创建标准的基于的基于java接口的代理
* **@Retryable** 标注此注解的方法会在发生异常时进行重试
  * interceptor： 将interceptor的bean名称应用到retryable()
  * value: 可重试的异常类型
  * label: 统计报告的唯一标签
  * maxAttempts: 尝试的最大次数（包括第一次失败），默认为3次
  * backoff: @BackOff 指定用于重试次操作的backoff属性。默认为空
    * delay: 重试等待，默认使用1000ms
    * maxDelay: 最大重试等待时间
    * multiplier: 延迟的乘数，用于计算下一个延迟（大于0生效）
    * random： 随机重试等待时间，默认false
  
建立一个serviced类
```java
@Slf4j
@Service
public class SpringRetryDemo   {

 /**
   * 重试所调用方法
   * @param param
   * @return
   */
  @Retryable(value = {RemoteAccessException.class},maxAttempts = 3,backoff = @Backoff(delay = 2000L,multiplier = 2))
  public boolean call(String param){
      return RetryDemoTask.retryTask(param);
  }

  /**
   * 达到最大重试次数,或抛出了一个没有指定进行重试的异常
   * recover 机制
   * @param e 异常
   */
  @Recover
  public boolean recover(Exception e,String param) {
    log.error("达到最大重试次数,或抛出了一个没有指定进行重试的异常:",e);
    return false;
  }

}
```

* 然后我们调用这个service里面的call方法
```java
@Slf4j
public class SpringRetryDemoTest extends MyBaseTest {

  @Autowired
  private SpringRetryDemo springRetryDemo;

  @Test
  public void retry(){
    boolean abc = springRetryDemo.call("abc");
    log.info("--结果是:{}--",abc);
  }

}
```

这里我依然是RemoteAccessException的异常才重试，@Backoff(delay = 2000L,multiplier = 2))表示第一次间隔2秒，以后都是次数的2倍,也就是第二次4秒，第三次6秒.


### 重试框架之GuavaRetry

GuavaRetry工具与SpringRetry类似，都是通过定义重试者角色来包装正常逻辑重试，但是Guava retry有更优的策略定义，在支持重试次数和重试频度控制基础上，能够兼容支持多个异常或者自定义实体对象的重试源定义，让重试功能有更多的灵活性。

GuavaRetry也是线程安全的，入口调用逻辑采用的是Java.util.concurrent.Callable的call方法，示例代码如下：

* 添加依赖
```xml
<!-- https://mvnrepository.com/artifact/com.github.rholder/guava-retrying -->
<dependency>
  <groupId>com.github.rholder</groupId>
  <artifactId>guava-retrying</artifactId>
  <version>2.0.0</version>
</dependency>
```

* 添加测试任务
```java
@Slf4j
public class GuavaRetryDemoTask {

  public static boolean retryTask(String param)  {
    log.info("收到请求参数:{}",param);
    int i = 0;
    try {
      i = SecureRandom.getInstanceStrong().nextInt(11);
    } catch (NoSuchAlgorithmException e) {
      throw new RemoteAccessException("获取随机数失败");
    }
    log.info("随机生成的数:{}",i);
    if (i < 2) {
      log.info("为0,抛出参数异常.");
      throw new IllegalArgumentException("参数异常");
    }else if (i  < 5){
      log.info("为1,返回true.");
      return true;
    }else if (i < 7){
      log.info("为2,返回false.");
      return false;
    }else{
      //为其他
      log.info("大于2,抛出自定义异常.");
      throw new RemoteAccessException("大于2,抛出自定义异常");
    }
  }

}
```

* GuavaRetry 这里设定跟SpringRetry不一样，我们可以根据返回的结果来判断是否重试，比如返回false我们就重试
```java
@Test
public void fun01(){
  // RetryerBuilder 构建重试实例 retryer,可以设置重试源且可以支持多个重试源，可以配置重试次数或重试超时时间，以及可以配置等待时间间隔
  Retryer<Boolean> retryer = RetryerBuilder.<Boolean> newBuilder()
      //设置异常重试源
          .retryIfExceptionOfType(RemoteAccessException.class)
      //设置根据结果重试
          .retryIfResult(res-> !res)
      //设置等待间隔时间
          .withWaitStrategy(WaitStrategies.fixedWait(3, TimeUnit.SECONDS))
      //设置最大重试次数
          .withStopStrategy(StopStrategies.stopAfterAttempt(3))
          .build();
  try {
    retryer.call(() -> RetryDemoTask.retryTask("abc"));
  } catch (Exception e) {
    e.printStackTrace();
  }
}
```

我们可以更灵活的配置重试策略，比如:
* **retryIfException:** 抛出 runtime 异常、checked 异常时都会重试，但是抛出 error 不会重试。
* **retryIfRuntimeException:** 只会在抛 runtime 异常的时候才重试，checked 异常和error 都不重试。
* **retryIfExceptionOfType:** 允许我们只在发生特定异常的时候才重试，比如NullPointerException 和 IllegalStateException 都属于 runtime 异常，也包括自定义的error。
* **retryIfResult:** 可以指定你的 Callable 方法在返回值的时候进行重试
* **RetryListener:** 当发生重试之后，假如我们需要做一些额外的处理动作，比如log一下异常，那么可以使用RetryListener。每次重试之后，guava-retrying 会自动回调我们注册的监听。可以注册多个RetryListener，会按照注册顺序依次调用。

主要的配置：
* **AttemptTimeLimiter：** 单次任务执行时间限制，如果超过配置则终止执行当前任务
* **BlockStrategy：** 任务阻塞策略，通俗来讲就是当前任务执行完，下次任务还没开始这段时间做什么，默认策略BlockStrategies.THREAD_SLEEP_STRATEGY
* **RetryListener：** 自定义重试监听器
* **StopStrategy：** 重试停止策略
  * **StopAfterDelayStrategy** 设置一个最长允许的执行时间，比如设定最长执行10秒，无论任务执行次数，只要重试的时候超过了最长时间，则任务终止，并返回重试异常RetryException
  * **StopAfterAttemptStrategy** 设置最大重试次数，如果超过最大重试次数则停止重试，并返回重试异常
  * **NeverStopStrategy** 不停止，用于需要一直轮训直到返回期望结果的情况
* **WaitStrategy：** 重试等待时长
  * **FixedWaitStrategy**         固定等待时长策略
  * **RandomWaitStrategy**        随机等待时长策略（可以提供一个最小和最大时长，等待时长为其区间随机值）
  * **IncrementingWaitStrategy**  递增等待时长策略（提供一个初始值和步长，等待时间随着重试次数增加而增加）
  * **ExponentialWaitStrategy**   指数等待时长策略
  * **FibonacciWaitStrategy**   斐波那契等待时长策略
  * **ExceptionWaitStrategy**   异常时长等待策略
  * **CompositeWaitStrategy**   复合时长等待策略

### 总结
SpringRetry 和 GuavaRetry 工具都是线程安全的重试，能够支持并发业务场景的重试逻辑正确性。两者都很好的将正常方法和重试方法进行了解耦，可以设置超时时间、重试次数、间隔时间、监听结果、都是不错的框架。

但是明显感觉得到，GuavaRetry在使用上更便捷，更灵活，能根据方法返回值来判断是否重试，而SpringRetry只能根据抛出的异常来进行重试。