---
title: Spring schema 自定义扩展
date: 2018-07-19
categories:
    - 学习
tags:
    - Spring
---
### 实现步骤
Spring 2.5在2.0的基于Schema的Bean配置的基础之上，再增加了扩展XML配置的机制。通过该机制，我们可以编写自己的Schema，并根据
自定义的Schema用自定的标签配置Bean。要使用的Spring的扩展XML配置机制，也比较简单，有以下4个步骤：
* 编写自定义Schema文件
* 编写自定义NamespaceHandler
* 编写解析BeanDefinition的parser
* 在Spring中注册上述组建

<!-- more -->

#### Maven 依赖
```
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-core</artifactId>
    <version>3.2.4.RELEASE</version>
</dependency>
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-beans</artifactId>
    <version>3.2.4.RELEASE</version>
</dependency>
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-aop</artifactId>
    <version>3.2.4.RELEASE</version>
</dependency>
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context</artifactId>
    <version>3.2.4.RELEASE</version>
</dependency>
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context-support</artifactId>
    <version>3.2.4.RELEASE</version>
</dependency>
```

#### 自定义schema文件
参考：http://www.w3school.com.cn/schema/schema_elements_ref.asp , 如下people.xsd文件
```
<?xml version="1.0" encoding="UTF-8"?>
<xsd:schema xmlns="http://www.pomelo.com/schema/people"
            xmlns:xsd="http://www.w3.org/2001/XMLSchema"
            xmlns:beans="http://www.springframework.org/schema/beans"
            targetNamespace="http://www.pomelo.com/schema/people"
            elementFormDefault="qualified"
            attributeFormDefault="unqualified">
    <xsd:import namespace="http://www.springframework.org/schema/beans"/>

    <xsd:element name="student">
        <xsd:complexType>
            <xsd:complexContent>
                <xsd:extension base="beans:identifiedType">

                    <xsd:attribute name="name" type="xsd:string">
                        <xsd:annotation>
                            <xsd:documentation>姓名</xsd:documentation>
                        </xsd:annotation>
                    </xsd:attribute>

                    <xsd:attribute name="age" type="xsd:string">
                        <xsd:annotation>
                            <xsd:documentation>年龄</xsd:documentation>
                        </xsd:annotation>
                    </xsd:attribute>

                </xsd:extension>
            </xsd:complexContent>
        </xsd:complexType>
    </xsd:element>
</xsd:schema>
```

#### 自定义NamespaceHandler
```
import org.springframework.beans.factory.xml.NamespaceHandlerSupport;

public class StudentNamespaceHandler extends NamespaceHandlerSupport {
    @Override
    public void init() {
        registerBeanDefinitionParser("student", new StudentBeanDefinitionParser());
    }
}
```

#### 编写BeanDefinition
```
import org.springframework.beans.factory.support.BeanDefinitionBuilder;
import org.springframework.beans.factory.xml.AbstractSingleBeanDefinitionParser;
import org.springframework.util.StringUtils;
import org.w3c.dom.Element;

public class StudentBeanDefinitionParser extends AbstractSingleBeanDefinitionParser {

    protected Class getBeanClass(Element element) {
        return Student.class;
    }

    protected void doParse(Element element, BeanDefinitionBuilder bean) {
        String name = element.getAttribute("name");
        bean.addPropertyValue("name", name);

        String age = element.getAttribute("age");
        if (StringUtils.hasText(age)) {
            bean.addPropertyValue("age", Integer.valueOf(age));
        }
    }
}
```

实体类：
```
public class Student {

    private String name;  

    private int age;  

    public String getName() {  
        return name;  
    }  

    public void setName(String name) {  
        this.name = name;  
    }  

    public int getAge() {  
        return age;  
    }  

    public void setAge(int age) {  
        this.age = age;  
    }  

}
```

#### 注册schema组件
最后在META-INFO目录下添加两个配置文件（spring.handler 和 spring.schema）:
**spring.handler**配置文件如下：
```
http\://www.pomelo.com/schema/people=schema.StudentNamespaceHandler
```
**spring.schema**配置文件如下：
```
http\://www.pomelo.com/schema/people.xsd=META-INF/people.xsd
```

#### 测试
新建applicationContext.xml放在clasapath下面：
```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:people="http://www.pomelo.com/schema/people"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.pomelo.com/schema/people http://www.pomelo.com/schema/people.xsd">

    <people:student id="student1" name="student1" age="18"/>

    <people:student id="student2" name="student2" age="20" />


    <bean id="student3" class="schema.Student">
        <property name="name" value="student3"/>
        <property name="age" value="23"/>
    </bean>

</beans>
```
java调用：
```
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class SchemaTest {

    public static void main(String[] args) {

        ApplicationContext ctx = new ClassPathXmlApplicationContext("/applicationContext.xml");
        Student student1 = (Student) ctx.getBean("student1");
        Student student2 = (Student) ctx.getBean("student2");
        Student student3 = (Student) ctx.getBean("student3");

        System.out.println("name: " + student1.getName() + " age :" + student1.getAge());
        System.out.println("name: " + student2.getName() + " age :" + student2.getAge());
        System.out.println("name: " + student3.getName() + " age :" + student3.getAge());
    }
}
```

* [参考链接](https://blog.csdn.net/zhengyong15984285623/article/details/60876418)
* [源码地址](https://github.com/zyongjava/pomelo)