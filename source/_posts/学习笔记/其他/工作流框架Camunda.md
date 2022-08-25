---
title: 工作流框架Camunda
date: 2022-08-10
categories:
    - 学习
tags:
    - Camunda
    - 工作流
---
### 框架对比

[工作流框架对比](https://zhuanlan.zhihu.com/p/435249026)

<!-- more -->

### SpringBoot 集成Camunda

* pom文件配置

```xml
<!-- SpringBoot 2.2.0以上版本 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
</dependency>
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <scope>runtime</scope>
</dependency>
<!-- 流程引擎 -->
<dependency>
    <groupId>org.camunda.bpm.springboot</groupId>
    <artifactId>camunda-bpm-spring-boot-starter</artifactId>
    <version>3.4.4</version>
</dependency>
<!-- web界面模块 -->
<dependency>
    <groupId>org.camunda.bpm.springboot</groupId>
    <artifactId>camunda-bpm-spring-boot-starter-webapp</artifactId>
    <version>3.4.4</version>
</dependency>
<!--  rest服务接口 -->
<dependency>
    <groupId>org.camunda.bpm.springboot</groupId>
    <artifactId>camunda-bpm-spring-boot-starter-rest</artifactId>
    <version>3.4.4</version>
</dependency>
```

* yml文件配置

```yml
spring:
  application:
     name: camunda-demo
  datasource:
    url: jdbc:mysql://127.0.0.1:3306/camunda-demo?serverTimezone=Asia/Shanghai
    driver-class-name: com.mysql.cj.jdbc.Driver
    username: root
    password: 123456
camunda:
  bpm:
    admin-user:
      id: admin
      password: admin
      first-name: admin
    filter:
      create: All tasks
    #禁止自动部署resources下面的bpmn文件
    auto-deployment-enabled: false
```

* 下载bpmn建模工具：https://camunda.com/download/modeler/

*  画一个流程图另存为 apply.bpmn

* testDemo

```java
@RestController
public class TestController {

  @Autowired
  RepositoryService repositoryService;
  @Autowired
  RuntimeService runtimeService;
  @Autowired
  TaskService taskService;
  @Autowired
  HistoryService historyService;
  @Autowired
  ProcessEngine processEngine;
  @Autowired
  ProcessEngine engine;

  @PostMapping("/deploy")
  public void deploy() {
    Deployment deploy = repositoryService.createDeployment()
        .addClasspathResource("BPMN/apply.bpmn")
        .deploy();
    System.out.println(deploy.getId());
  }

  @PostMapping("/start")
  public void runProcinst(){
    Map<String,Object> params = new HashMap<>();
    params.put("money",2001);
    ProcessInstance apply = runtimeService.startProcessInstanceByKey("apply",params);
    System.out.println(apply.getProcessDefinitionId());
    System.out.println(apply.getId());
    System.out.println(apply.getProcessInstanceId());
  }

  @PostMapping("/taskquery")
  public void taskQuery() {
    List<Task> tasks = taskService.createTaskQuery()
        .processDefinitionKey("apply")
        .list();
    for (Task task : tasks) {
      System.out.println(task.getAssignee());
      System.out.println(task.getId());
      System.out.println(task.getName());
      System.out.println(task.getTenantId());
    }
  }

  @PostMapping("/mytaskquery")
  public List<HistoricTaskInstance> myTaskQuery() {
    List<HistoricTaskInstance> instances = engine.getHistoryService().createHistoricTaskInstanceQuery()
        .taskAssignee("lisi").unfinished().orderByHistoricActivityInstanceStartTime().asc().list();
    return instances;
  }

  @PostMapping("/taskComplete")
  public void taskComplete(){
    Task task = taskService.createTaskQuery()
        .taskAssignee("zhangsan")
        .singleResult();
    Map<String,Object> params = new HashMap<>();
    params.put("monge","2000");
    taskService.complete(task.getId(),params);
  }

  @PostMapping("/queryDefine")
  public void queryDefine(){
    ProcessDefinitionQuery query = repositoryService.createProcessDefinitionQuery();
    List<ProcessDefinition> definitions = query.processDefinitionKey("apply")
        .orderByProcessDefinitionVersion()
        .desc()
        .list();
    for (ProcessDefinition definition : definitions) {
      System.out.println(definition.getDeploymentId());
      System.out.println(definition.getName());
      System.out.println(definition.getVersion());
      System.out.println(definition.getId());
      System.out.println(definition.getKey());
    }
  }

  @PostMapping("/deleteDefine")
  public void deleteDefine(){
    ProcessDefinitionQuery processDefinitionQuery = repositoryService.createProcessDefinitionQuery();
    List<ProcessDefinition> definitions = processDefinitionQuery.processDefinitionKey("apply")
        .orderByProcessDefinitionVersion()
        .asc()
        .list();
    ProcessDefinition processDefinition = definitions.get(0);
    if (processDefinition != null){
      repositoryService.deleteDeployment(processDefinition.getDeploymentId(),true);
    }
  }

  @PostMapping("/queryHistory")
  public void queryHistory(){
    List<HistoricActivityInstance> list = historyService.createHistoricActivityInstanceQuery()
        .finished()
        .orderByHistoricActivityInstanceEndTime()
        .asc()
        .list();
    for (HistoricActivityInstance instance : list) {
      System.out.println(instance.getActivityId());
      System.out.println(instance.getProcessDefinitionKey());
      System.out.println(instance.getAssignee());
      System.out.println(instance.getStartTime());
      System.out.println(instance.getEndTime());
      System.out.println("=============================");
    }
  }

  public void startProcInstAddBusinessKey(){
    ProcessInstance apply = runtimeService.startProcessInstanceByKey("apply", "aaaa-scsc-89uc");
    System.out.println(apply.getBusinessKey());
  }

```
