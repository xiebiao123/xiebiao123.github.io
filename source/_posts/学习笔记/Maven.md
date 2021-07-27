---
title: Maven
date: 2018-07-17
categories:
    - 学习
tags:
    - maven
---
##### 基础概念

###### 稳定RELEASE

    用户A将代码打包发布到RELEASE仓库，用户B使用时，需要在pom.xml添加JAR包的依赖坐标。如果用户A将版本从1.0升级为2.0，用户B使用时也需要同时在pom.xml中修改坐标版本。但是RELEASE是稳定版本，是经过测试以后才会发布的，通常不会频繁的升级版本
![RELEASE](https://img-blog.csdn.net/20170325212027351?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvd2FuZ2JfamF2YQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

###### 快照SNAPSHOT

    SNAPSHOT是不稳定版，可能是还在开发中的版本，在开发时用户A可能每天都会更新代码，可能会频繁的发布版本。而另一组用户B需要实时得到A的最新代码版本，以进行同步开发。如果使用RELEASE仓库需要不停的更换坐标，才能升级到最新版本。而SNAPSHOT仓库则不需要这样做，用户A和用户B都不用升级版本。用户A每次发布时会根据当时时间创建一个新的快照版本，之前的快照版本也会保留成为历史版本。用户B每次构建项目时会自动根据版本时间加载最新的JAR包，这种模式更加适合于多模块同步开发测试阶段
![RELEASE](https://img-blog.csdn.net/20170325212438654?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvd2FuZ2JfamF2YQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

###### package、install、deploy命令区别

* package命令完成了项目编译、单元测试、打包功能，但没有把打好的可执行jar包，布署到本地maven仓库和远程maven私服仓库
* install命令完成了项目编译、单元测试、打包功能，同时把打好的可执行jar包，布署到本地maven仓库，但没有布署到远程maven私服仓库
* deploy命令完成了项目编译、单元测试、打包功能，同时把打好的可执行jar包，布署到本地maven仓库和远程maven私服仓库

<!-- more -->

##### release插件

* 指定maven私服地址/git仓库地址
  
``` xml
<scm>
    <!--远程git仓库地址-->
    <connection>scm:git:http://gitlab.dafycredit.com/doms/giveu-doms.git</connection>
    <developerConnection>scm:git:http://gitlab.dafycredit.com/doms/giveu-doms.git</developerConnection>
    <url>scm:git:http://gitlab.dafycredit.com/doms/giveu-doms.git</url>
    <tag>HEAD</tag>
</scm>

<distributionManagement>
    <!-- 指定仓库地址 -->
    <repository>
        <id>releases</id>
        <name>Release</name>
        <url>http://10.10.11.197:8081/nexus/content/repositories/releases/</url>
    </repository>
    <snapshotRepository>
        <id>snapshots</id>
        <name>Repository</name>
        <url>http://10.10.11.197:8081/nexus/content/repositories/snapshots/</url>
    </snapshotRepository>
</distributionManagement>
```

* 添加插件配置
  
``` xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-release-plugin</artifactId>
    <version>2.5.3</version>
    <configuration>
        <!-- 自动更改所有子模块版本 -->
        <autoVersionSubmodules>true</autoVersionSubmodules>
        <!-- 生成的tag名字 -->
        <tagNameFormat>v@{project.version}</tagNameFormat>
    </configuration>
    <dependencies>
        <dependency>
            <groupId>org.apache.maven.scm</groupId>
            <artifactId>maven-scm-provider-jgit</artifactId>
            <version>1.9.5</version>
        </dependency>
    </dependencies>
</plugin>
```

* 常用命令
  * **mvn release:prepare** 【准备发布】相当于发布前的准备。此命令会首先去去掉版本号中的SNAPSHOT标志符，并在本地本分修改前的pox.xml并提交推送到远程。以及在svn/git服务器生成一个指定版本的tag，编译并打包项目。注意运行命令之前注意本地代码一定要与远程代码保持一致，否则可能出现该错误:Cannot prepare the release because you have local modifications。
  * **mvn release:perform [-DuseReleaseProfile=false]** 【正式发布提交】正式提交到maven私服，并删除本地备份的pom.xml和版本信息文件。
  * **mvn release:rollback** 【回滚】该命令会还原之前的所有pom.xml，删除生产的版本文件并提交到git上去。如果prepare的过程中出现了错误可以执行此命令回滚prepare的操作。注意：rollback命令只会提交pom.xml的修改到git中并不会删除git上创建的tag，需要手动删除本地tag和远程tag。删除本地tag：git tag -d ${tagname} 接着推送到远程 git push origin :refs/tags/${tagname} 。切记不要忘记手动删除tag。
  * **mvn release:clean** 【删除版本文件】删除一些插件生成的相关文件。相对release:rollback也会删除这些文件。非特殊情况不需要使用该命令。如果release:clean命令的话，无法进行release:rollback 回滚。一般是release:rollback失败的话才会使用这个命令。

##### Versions插件

* 添加插件配置

``` xml
<plugin>
    <!-- versions插件 -->
    <groupId>org.codehaus.mojo</groupId>
    <artifactId>versions-maven-plugin</artifactId>
    <version>2.5.3</version>
</plugin>
```

* 常用命名
  * **mvn versions:set -DnewVersion=1.0.0-SNAPSHOT \[-DgroupId=com.\* -DartifactId=\* -DoldVersion=0.\* ]** 【预修改】
  * **mvn versions:commit** 【确认修改】
  * **mvn versions:revert** 【回滚】
  * **mvn versions:use-releases** 【使用releases版本】
