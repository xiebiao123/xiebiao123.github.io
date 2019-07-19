---
title: Groovy
date: 2018-07-19
categories:
    - 学习
tags:
    - groovy
---

### 类型
#### 字符串
```
/*多行字符串可以使用三个连续的单引号或双引号包括*/
def multiline="""line1
line2
line3
"""

/*内插字符串,变量直接插入到字符串中,当使用内插字符串的时候，字符串字面值是Groovy的字符串类型GString。这一点需要注意。
普通的Java字符串是不变的，而GString是可变的。另外它们的哈希值也不同。因此在使用Map等数据类型的时候需要格外注意，避
免使用GString作为Map的键。*/
def name = 'Guillaume'
def greeting = "Hello ${name}"
```

<!-- more -->

#### 布尔类型
```
Groovy的布尔类型和Java类似，也有true和false两个值。不过Groovy的布尔语义更丰富。未到结尾的迭代器、非空对象引用、非零数
字都认为是真；空集合、空字符串等认为是假
```

#### 数字类型
```
// Groovy支持byte、char 、short、 int 、long和 BigInteger等几种数字类型。如果使用普通方式声明，它们和Java中的变量很相似。

// 如果使用def关键字声明，那么这些数字会自动选择可以容纳它们的类型
def a = 1
assert a instanceof Integer

// Integer.MAX_VALUE
def b = 2147483647
assert b instanceof Integer

// Integer.MAX_VALUE + 1
def c = 2147483648
assert c instanceof Long

// Long.MAX_VALUE
def d = 9223372036854775807
assert d instanceof Long

// Long.MAX_VALUE + 1
def e = 9223372036854775808
assert e instanceof BigInteger

// 数据计算

/*  数字的计算结果和Java规则类似：小于int的整数类型会被提升为int类型，计算结果也是int类型；小于long的整数类型和long计算，
结果是long类型；BigInteger和其它整数类型计算，结果是BigInteger类型；BigDecimal和其它整数类型计算，结果是BigDecimal类型；
BigDecimal和float、double等类型计算，结果是double类型。*/

```

#### 列表
```
/*使用[....]语法声明列表，默认情况下列表是ArrayList实现。我们也可以使用as运算符自己选择合适的列表底层类型。*/
def arrayList = [1, 2, 3]
assert arrayList instanceof java.util.ArrayList

def linkedList = [2, 3, 4] as LinkedList    
assert linkedList instanceof java.util.LinkedList

/*使用[索引]引用和修改列表元素。如果索引是负的，则从后往前计数。要在列表末尾添加元素，可以使用左移运算符<<。如果在方括号中指
定了多个索引，会返回由这些索引对应元素组成的新列表。使用两个点加首位索引..可以选择一个子列表。*/
def letters = ['a', 'b', 'c', 'd']

assert letters[0] == 'a'     
assert letters[1] == 'b'

assert letters[-1] == 'd'    
assert letters[-2] == 'c'

letters[2] = 'C'             
assert letters[2] == 'C'

letters << 'e'               
assert letters[ 4] == 'e'
assert letters[-1] == 'e'

assert letters[1, 3] == ['b', 'd']         
assert letters[2..4] == ['C', 'd', 'e']    

// 列表还可以组合成复合列表
def multi = [[0, 1], [2, 3]]     
assert multi[1][0] == 2  
```

#### 数组
```
/*声明数组的方式和列表一样，只不过需要显示指定数组类型。数组的使用方法也和列表类似，只不过由于数组是不可变的，所以不能像数组
末尾添加元素。*/

int[] intArray = [1, 2, 3, 4, 5]
def intArray2 = [1, 2, 3, 4, 5, 6] as int[]
```

### Map
```
/*创建Map同样使用方括号，不过这次需要同时指定键和值了。Map创建好之后，我们可以使用[键]或.键来访问对应的值。默认情况下创建的
Map是java.util.LinkedHashMap，我们可以声明变量类型或者使用as关键字改变Map的实际类型。*/

def colors = [red: '#FF0000', green: '#00FF00', blue: '#0000FF']   

assert colors['red'] == '#FF0000'    
assert colors.green  == '#00FF00' 

/*关于Map有一点需要注意。如果将一个变量直接作为Map的键的话，其实Groovy会用该变量的名称作为键，而不是实际的值。如果需要讲变量
的值作为键的话，需要在变量上添加小括号。*/

def key = 'name'
def person = [key: 'Guillaume']      //键是key而不是name

assert !person.containsKey('name')   
assert person.containsKey('key') 

//这次才正确的将key变量的值作为Map的键
person = [(key): 'Guillaume']        

assert person.containsKey('name')    
assert !person.containsKey('key') 
```

### 运算符
- Groovy的数学运算符和Java类似，只不过多了一个乘方运算**和乘方赋值**=。
- Groovy的关系运算符（大于、小于等于这些）和Java类似。
- Groovy的逻辑运算符（与或非这些）和Java类似，也支持短路计算。
- Groovy的位运算符合Java类似。
- Groovy的三元运算符条件?值1:值2和Java类似。

#### 可空运算
```
/*Groovy支持Elvis操作符，当对象非空的时候结果是值1，为空时结果是值2。或者更直接，对象非空是使用对象本身，为空时给另一个值，常用于给定某个可空变量的默认值。*/
displayName = user.name ? user.name : 'Anonymous'   
displayName = user.name ?: 'Anonymous'
```

#### 安全导航运算符
```
/*当调用一个对象上的方法或属性时，如果该对象为空，就会抛出空指针异常。这时候可以使用?.运算符，当对象为空时表达式的值也是空，不会抛出空指针异常。*/
def person = Person.find { it.id == 123 }    
def name = person?.name                      
assert name == null
```

#### 字段访问运算符
```
/*在Groovy中默认情况下使用点运算符.会引用属性的Getter或Setter。如果希望直接访问字段，需要使用.@运算符。*/
class User {
    public final String name                 
    User(String name) { this.name = name}
    String getName() { "Name: $name" }       
}
def user = new User('Bob')
assert user.name == 'Name: Bob'   
assert user.@name == 'Bob'
```
#### 方法指针运算
```
/* 我们可以将方法赋给变量，这需要使用.&运算符。然后我们就可以像调用方法那样使用变量。方法引用的实际类型是Groovy的闭包Closure。这种运算符可以将方法作为参数 */
def str = 'example of method reference'            
def fun = str.&toUpperCase                         
def upper = fun()                                  
assert upper == str.toUpperCase()
```
#### 展开运算符
```
/*展开运算符*.会调用一个列表上所有元素的相应方法或属性，然后将结果再组合成一个列表。*/
class Car {
    String make
    String model
}
def cars = [
       new Car(make: 'Peugeot', model: '508'),
       new Car(make: 'Renault', model: 'Clio')]       
def makes = cars*.make                                
assert makes == ['Peugeot', 'Renault']
// 展开运算符是空值安全的，如果遇到了null值，不会抛出空指针异常，而是返回空值。
cars = [
   new Car(make: 'Peugeot', model: '508'),
   null,                                              
   new Car(make: 'Renault', model: 'Clio')]
assert cars*.make == ['Peugeot', null, 'Renault']     
assert null*.make == null 
// 展开运算符还可以用于展开方法参数、列表和Map。
```

#### 范围运算符
```
/*使用..创建范围。默认情况下范围是闭区间，如果需要开闭区间可以在结束范围上添加<符号。范围的类型是groovy.lang.Range，它继承了List接口，也就是说我们可以将范围当做List使用。*/

def range = 0..5                                    
assert (0..5).collect() == [0, 1, 2, 3, 4, 5]       
assert (0..<5).collect() == [0, 1, 2, 3, 4]         
assert (0..5) instanceof List                       
assert (0..5).size() == 6  
```

#### 比较运算符
```
<=>运算符相当于调用compareTo方法。

def list = ['Grace','Rob','Emmy']
assert ('Emmy' in list)
```

#### 成员运算符
```
/*成员运算符in相当于调用contains或isCase方法。*/
def list = ['Grace','Rob','Emmy']
assert ('Emmy' in list)
```

#### 相等运算符
```
==运算符和Java中的不同。在Groovy中它相当于调用equals方法。如果需要比较引用，使用is。
def list1 = ['Groovy 1.8','Groovy 2.0','Groovy 2.3']        
def list2 = ['Groovy 1.8','Groovy 2.0','Groovy 2.3']        
assert list1 == list2    //比较内容相等                                   
assert !list1.is(list2)   //比较引用相等
```

#### 转化运算符
```
/*我们可以使用Java形式的(String) i来转换类型。但是假如类型不匹配的话，就会抛出ClassCastException。而使用as运算符就会避免这种情况。*/

Integer x = 123
String s = x as String 

如果希望自己的类也支持as运算符的话，需要实现asType方法。
```

### 表达式语句
#### 声明变量
```
/*Groovy支持以传统方式使用变量类型 变量名的方式声明变量，也可以使用def关键字声明变量。使用def关键字的时候，变量类型由编译器自动推断，无法推断时就是Object类型。*/

// Groovy可以同时声明多个变量。
def (a, b, c) = [10, 20, 'foo']

// 如果左边的变量数比右面的值多，那么剩余的变量就是null。
def (a, b, c) = [1, 2]
assert a == 1 && b == 2 && c == null
如果等号右面比左面多，那么多余的值会被忽略。
def (a, b) = [1, 2, 3]
assert a == 1 && b == 2


```