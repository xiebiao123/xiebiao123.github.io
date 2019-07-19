---
title: Freemarker
date: 2018-07-19
categories:
    - 学习
tags:
    - freemarker
---

### 取值指令
- 常用${var}语法进行取值
- 对null、不存在对象取值${var!}或${var!'defaultValue'}
- 取包装对象的值,通过‘点’语法${user.name}
- 取值的时候可以进行赋值、运算
- Date类型格式化${date?String('yyyy-MM-dd')}
- 转义HTML内容${date?html}
- {data?string.currency} -- $20.00
- {data?string.percent} -- 20%
- 布尔类型${foo?string("yes","no")}

### 内置函数(使用?)
#### 对于字符串
- html 对字符串进行HTML编码
- cap_first 使字符串第一个字母大写
- lower_case 将字符串转换成小写
- trim 去掉字符串前后的空白字符

<!-- more -->

#### 示例
- ${“freeMarker”?cap_first} 

#### 对于集合(List、Map)
- size 获得集合中元素的数目

#### 示例


### 逻辑判断

#### IF
```
<#if condition>... 
<#elseif condition2>... 
<#elseif condition3>...... 
<#else>... 
```
#### SWITCH
```
<#switch value> 
<#case refValue1> 
... 
<#break> 
<#case refValue2> 
... 
<#break> 
... 
<#case refValueN> 
... 
<#break> 
<#default> 
... 
</#switch>
```
### 定义变量
<#assign x=0..100/>

### 集合读取
```
<#list students as stu> 
    ${stu}<br/>
</#list> 
```

### 宏/模板

#### macro
```
# macro语法
<#macro macro_name param1 param2 param3 paramN>
    template_code ${param1}
    <#nested/> //调用方嵌套代码
</#macro>
# macro调用
<@macro_name param1="value1" param1="value2"/>

<@macro_name param1="value1" param1="value2">
    nested_template
</@macro_name>
```

#### function
```
# function语法
<#function function_name param1 param2>
    <#return param1 + param2>
</#function>

# function调用
${function_name(param1,param2)}
```