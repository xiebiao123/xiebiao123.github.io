---
title: Markdown

date: 2018-06-24

categories:
	- 学习
tags:
	- 学习笔记
---

##### 1.标题

	# 一级标题
	## 二级标题
	### 三级标题
	#### 四级标题
	##### 五级标题
	###### 六级标题

	这是高阶标题(效果和一级标题一样)
	===
	这是次阶标题(效果和二级标题一样)
	---

<!-- more -->

# 一级标题
## 二级标题
### 三级标题
#### 四级标题
##### 五级标题
###### 六级标题
这是高阶标题(效果和一级标题一样)
===
这是次阶标题(效果和二级标题一样)
---

----------

#### 2.列表
##### 无序（*, +, -）
	* 1
		* 嵌套1
		* 嵌套2
		* 嵌套3
	+ 2
	- 3

* 1
	* 嵌套1
	* 嵌套2
	* 嵌套3
+ 2
- 3
##### 有序（1., 2., 3.）
1. 1
2. 2
3. 3

----------

##### 3.引用(>)
	> 这里是引用
	>> 这是二级引用
	>>> 这是三级引用

> 这里是引用
>> 这是二级引用
>>> 这是三级引用

----------

##### 4.粗体&斜体
<!-- Markdown 的粗体和斜体也非常简单，用两个 * 包含一段文本就是粗体的语法，用一个 * 包含一段文本就是斜体的语法 -->
	**这是粗体**
	__这是粗体__
	_这是斜体_
	*这是斜体*

**这是粗体**
__这是粗体__
_这是斜体_
*这是斜体*

----------

##### 5.表格

	| Tables        | Are           | Cool  |
	| ------------- |:-------------:| -----:|
	| col 3 is      | right-aligned | $1600 |
	| col 2 is      | centered      |   $12 |
	| zebra stripes | are neat      |    $1 |

| Tables        | Are           | Cool  |
| ------------- |:-------------:| -----:|
| col 3 is      | right-aligned | $1600 |
| col 2 is      | centered      |   $12 |
| zebra stripes | are neat      |    $1 |

----------

##### 6.代码块
```javascript
/** 
* nth element in the fibonacci series. 
* @param n >= 0 
* @return the nth element, >= 0. 
*/
function fib(n) { 
 var a = 1, b = 1; 
 var tmp; 
 while (--n >= 0) { 
     tmp = a;
     a += b; 
     b = tmp;
 }
 return a; 
}
document.write(fib(10));
```

```java
int a=1;
int b=2;
System.out.println(a+b)
```

----------

##### 7.图片&链接
<!-- 插入链接与插入图片的语法很像，区别在一个 !号 -->
	[baidu链接](http://www.baidu.com "只是一个提示")
	![picture](http://www.turingbook.com/Content/img/Turing.Gif "只是一个提示")
[baidu链接](http://www.baidu.com "只是一个提示")
![picture](http://www.turingbook.com/Content/img/Turing.Gif "只是一个提示")

###### 行内图片&链接

这就是行内链接：[百度](http://www.baidu.com "只是一个提示")
这就是行内图片：![图灵社区](http://www.turingbook.com/Content/img/Turing.Gif)

###### 参考式图片&链接
	1. [JavaScript | MDN][1]
	[1]: http://developer.mozilla.org/zh-CN/docs/Web/JavaScript
	
这里我们参考：
1. [JavaScript | MDN][1]
2. [ECMAScript 6 入门 阮一峰][2]
3. [InfoQ JavaScript][3]
4. [图灵社区][4]
5. ![图灵社区Logo][5]
[1]: http://developer.mozilla.org/zh-CN/docs/Web/JavaScript
[2]: http://es6.ruanyifeng.com
[3]: http://www.infoq.com/cn/javascript/?utm_source=infoq&utm_medium=header_graybar&utm_campaign=topic_clk
[4]: http://www.ituring.com.cn
[5]: http://www.turingbook.com/Content/img/Turing.Gif

##### 8.分割线(* - _)

	***

	---

	___

***

---

___

##### 9.删除线
	~~删除线哈哈哈哈~~

~~删除线哈哈哈哈~~

##### 10.注脚

这是一个注脚测试[^1]
[^1]:这是一个测试，用来阐释注脚。

这是一个注脚测试[^footer2]
[^footer2]:这是一个测试，用来阐释注脚。


##### 11.标签

标签: 数学 `英语` `数学`

Tags: 数学