---
title: Spock
date: 2018-07-19
categories:
    - 学习
tags:
    - spock
---

### 基本概念
```
# 预先定义的几个函数
def setup() {}          // run before every feature method
def cleanup() {}        // run after every feature method
def setupSpec() {}      // run before the first feature method
def cleanupSpec() {}    // run after the last feature method

# 每个method又被划分为不同的block，不同的block处于测试执行的不同阶段，在测试运行时，各个block按照不同的顺序和规则被执行

# setup / given 在这个block中会放置与这个测试函数相关的初始化程序
setup:
    def stack = new Stack()
    def elem = "push me"
    
# when与then需要搭配使用，在when中执行待测试的函数，在then中判断是否符合预期
when:
    stack.push(elem)  
then:
    !stack.empty
    stack.size() == 1
    stack.peek() == elem

# 在then或expect中会默认assert所有返回值是boolean型的顶级语句。如果要在其它地方增加断言，需要显式增加assert关键字
def setup() {
  stack = new Stack()
  assert stack.empty
}

# 如果要验证有没有抛出异常，可以用thrown()，如果要验证没有抛出某种异常，可以用notThrown()
when:
    stack.pop()  
then:
    def e = thrown(EmptyStackException)
    e.cause == null
    
setup:
    def map = new HashMap()  
when:
    map.put(null, "elem")  
then:
    notThrown(NullPointerException)
    
# expect可以看做精简版的when+then
when:
    def x = Math.max(1, 2)   ==>>   expect: Math.max(1, 2) == 2
then:
    x == 2

# Cleanup Blocks 函数退出前做一些清理工作，如关闭资源等

# Where
class DataDriven extends Specification {
    def "maximum of two numbers"() {
        expect:
        Math.max(a, b) == c

        where:
        a | b || c
        3 | 5 || 5
        7 | 0 || 7
        0 | 0 || 0
    }
}

# 上述例子实际会跑三次测试，相当于在for循环中执行三次测试，a/b/c的值分别为3/5/5,7/0/7和0/0/0。
如果在方法前声明@Unroll，则会当成三个方法运行。更进一步，可以为标记@Unroll的方法声明动态的spec名。

class DataDriven extends Specification {
    @Unroll
    def "maximum of #a and #b should be #c"() {
        expect:
        Math.max(a, b) == c

        where:
        a | b || c
        3 | 5 || 5
        7 | 0 || 7
        0 | 0 || 0
    }
}
```

<!-- more -->

### mock
```
class PublisherSpec extends Specification {
    Publisher publisher = new Publisher()
    Subscriber subscriber = Mock()
    Subscriber subscriber2 = Mock()

    def setup() {
        publisher.subscribers.add(subscriber)
        publisher.subscribers.add(subscriber2)
    }
}
# 创建了mock对象之后就可以对它的交互做验证了
def "should send messages to all subscribers"() {
    when:
    publisher.send("hello")

    then:
    1 * subscriber.receive("hello")
    1 * subscriber2.receive("hello")
}
# 上面的例子里验证了：在publisher调用send时，两个subscriber都应该被调用一次receive(“hello”)

示例中，表达式中的次数、对象、函数和参数部分都可以灵活定义：
1 * subscriber.receive("hello")      // exactly one call
0 * subscriber.receive("hello")      // zero calls
(1..3) * subscriber.receive("hello") // between one and three calls (inclusive)
(1.._) * subscriber.receive("hello") // at least one call
(_..3) * subscriber.receive("hello") // at most three calls
_ * subscriber.receive("hello")      // any number of calls, including zero
1 * subscriber.receive("hello")     // an argument that is equal to the String "hello"
1 * subscriber.receive(!"hello")    // an argument that is unequal to the String "hello"
1 * subscriber.receive()            // the empty argument list (would never match in our example)
1 * subscriber.receive(_)           // any single argument (including null)
1 * subscriber.receive(*_)          // any argument list (including the empty argument list)
1 * subscriber.receive(!null)       // any non-null argument
1 * subscriber.receive(_ as String) // any non-null argument that is-a String
1 * subscriber.receive({ it.size() > 3 }) // an argument that satisfies the given predicate
                                          // (here: message length is greater than 3)
1 * subscriber._(*_)     // any method on subscriber, with any argument list
1 * subscriber._         // shortcut for and preferred over the above
1 * _._                  // any method call on any mock object
1 * _                    // shortcut for and preferred over the above

# 对mock对象定义函数的返回值可以用如下方法：
subscriber.receive(_) >> "ok"

# 如果要每次调用返回不同结果
subscriber.receive(_) >>> ["ok", "error", "error", "ok"]

# 如果要做额外的操作，如抛出异常，可以使用：
subscriber.receive(_) >> { throw new InternalError("ouch") }

# 如果要每次调用都有不同的结果，可以把多次的返回连接起来
subscriber.receive(_) >>> ["ok", "fail", "ok"] >> { throw new InternalError() } >> "ok"

# 注意，spock不支持两次分别设定调用和返回值，如果把上例写成这样是错的
setup:
    subscriber.receive("message1") >> "ok"
when:
    publisher.send("message1")
then:
    1 * subscriber.receive("message1")

此时spock会对subscriber执行两次设定：
* 第一次设定receive(“message1″)只能调用一次，返回值为默认值（null）。
* 第二次设定receive(“message1″)会返回ok，不限制次数。
```