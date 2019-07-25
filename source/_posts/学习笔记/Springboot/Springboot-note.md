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
