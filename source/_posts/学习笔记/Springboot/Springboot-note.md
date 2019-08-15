---
title: Spring boot 遇到的坑
date: 2018-07-25
categories:
    - 学习
tags:
    - Spring boot
    - maven
---
### Spring boot不使用继承,使用依赖
```
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>1.5.21.RELEASE</version>
</parent>

# 替换为

<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-dependencies</artifactId>
            <version>1.5.21.RELEASE</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```
<!-- more -->

如果你的pom是继承spring-boot-starter-parent的话，只需要下面的指定就行
```
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
    </plugins>
</build>
```
如果你的POM不是继承spring-boot-starter-parent的话，需要下面的指定
```text
<build>
    <plugins>
		<plugin>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-maven-plugin</artifactId>
			<configuration>
                 <mainClass>com.johnny.LaonongminManagerApplication</mainClass>
			</configuration>
			<executions>
				<execution>
					<goals>
						<goal>repackage</goal>
					</goals>
				</execution>
			</executions>
		</plugin>
	</plugins>
</build>
```

### springboot读取.properties配置文件中的map和list类型配置参数
```
#map 第一种方式
data.person.name=zhangsan
data.person.sex=man
data.person.age=11
data.person.url=xxxxxxxx
#map 第二种方式
data.person[name]=zhangsan
data.person[sex]=man
data.person[age]=11
data.person[url]=xxxxxxxx
#list 第一种方式
data.list[0]=apple0
data.list[1]=apple1
data.list[2]=apple2
#list 第二种方式
data.list=apple0,apple1,apple2
```
```
/**
 * data.person.name
 * 这里map名需要和application.properties中的参数一致
 */
private Map<String, String> person = new HashMap<>();
/**
 * data.list
 * 这里list名需要和application.properties中的参数一致
 */
private List<String> list = new ArrayList<>();
```
