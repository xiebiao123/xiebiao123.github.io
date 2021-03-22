---
title: Spring boot
date: 2020-07-14
categories:
    - 学习
tags:
    - Spring boot
    - sentinel
    - nacos
---

### 前言
sentinel 原生版本的规则管理通过API，将规则推送至客户端并直接更新到内存中，服务重启自己定义的限流规则会丢失。如何避免这一问题？
这是我们想到了使用Nacos、Apollo等配置中心来持久话配置，客户端监听配置变化并更新本地缓存。即解决上述问题

### sentinel 控制台
#### 添加依赖
```xml
<dependency>
    <groupId>com.alibaba.boot</groupId>
    <artifactId>nacos-config-spring-boot-starter</artifactId>
    <version> 0.2.1</version>
</dependency>
```

#### 指定配置
```
nacos.config.server-addr = localhost:8848
nacos.config.namespace = a2881d2a-5c21-4f9e-9c75-dd93872e9ce8
```

#### 修改代码
* AuthorityRuleController (授权规则)
```
private boolean publishRules(String app, String ip, Integer port) {
    List<AuthorityRuleEntity> rules = repository.findAllByMachine(MachineInfo.of(app, ip, port));
    // 同步规则至nacos
    sentinelNacosConsts.setAppName(app);
    String rulesStr = JSON.toJSONString(rules);
    try {
        boolean result = configService.publishConfig(sentinelNacosConsts.getAuthorityFileName(),sentinelNacosConsts.getGroupName(),rulesStr);
        log.info("发布规则：{}",result);
    } catch (NacosException e) {
        log.info("发布规则异常：{}",e);
    }
    return sentinelApiClient.setAuthorityRuleOfMachine(app, ip, port, rules);
}
```
* DegradeController
```
private boolean publishRules(String app, String ip, Integer port) {
    List<DegradeRuleEntity> rules = repository.findAllByMachine(MachineInfo.of(app, ip, port));
    // 同步规则至nacos
    sentinelNacosConsts.setAppName(app);
    String rulesStr = JSON.toJSONString(rules);
    try {
        boolean result = configService.publishConfig(sentinelNacosConsts.getDegradeRuleFileName(),sentinelNacosConsts.getGroupName(),rulesStr);
        log.info("发布规则：{}",result);
    } catch (NacosException e) {
        log.info("发布规则异常：{}",e);
    }
    return sentinelApiClient.setDegradeRuleOfMachine(app, ip, port, rules);
}
```
* FlowControllerV1
```
private boolean publishRules(String app, String ip, Integer port) {
    List<FlowRuleEntity> rules = repository.findAllByMachine(MachineInfo.of(app, ip, port));
    // 同步规则至nacos
    sentinelNacosConsts.setAppName(app);
    String rulesStr = JSON.toJSONString(rules);
    try {
        boolean result = configService.publishConfig(sentinelNacosConsts.getFlowRuleFileName(),sentinelNacosConsts.getGroupName(),rulesStr);
        log.info("发布规则：{}",result);
    } catch (NacosException e) {
        log.info("发布规则异常：{}",e);
    }
    return sentinelApiClient.setFlowRuleOfMachine(app, ip, port, rules);
}
```
* ParamFlowRuleController
```
private CompletableFuture<Void> publishRules(String app, String ip, Integer port) {
    List<ParamFlowRuleEntity> rules = repository.findAllByMachine(MachineInfo.of(app, ip, port));
    // 同步规则至nacos
    sentinelNacosConsts.setAppName(app);
    String rulesStr = JSON.toJSONString(rules);
    try {
        boolean result = configService.publishConfig(sentinelNacosConsts.getParamFlowFileName(),sentinelNacosConsts.getGroupName(),rulesStr);
        log.info("发布规则：{}",result);
    } catch (NacosException e) {
        log.info("发布规则异常：{}",e);
    }
    return sentinelApiClient.setParamFlowRuleOfMachine(app, ip, port, rules);
}
```
* SystemController
```
private boolean publishRules(String app, String ip, Integer port) {
    List<SystemRuleEntity> rules = repository.findAllByMachine(MachineInfo.of(app, ip, port));
    // 同步规则至nacos
    sentinelNacosConsts.setAppName(app);
    String rulesStr = JSON.toJSONString(rules);
    try {
        boolean result = configService.publishConfig(sentinelNacosConsts.getSystemRuleFileName(),sentinelNacosConsts.getGroupName(),rulesStr);
        log.info("发布规则：{}",result);
    } catch (NacosException e) {
        log.info("发布规则异常：{}",e);
    }
    return sentinelApiClient.setSystemRuleOfMachine(app, ip, port, rules);
}
```

### sentinel 客户端
#### 添加依赖
```
<!-- 接入sentinel、nacos -->
<dependency>
    <groupId>com.alibaba.boot</groupId>
    <artifactId>nacos-config-spring-boot-starter</artifactId>
    <version>0.2.1</version>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
    <version>0.9.0.RELEASE</version>
</dependency>
<!--dubbo和http监控限流-->
<dependency>
    <groupId>com.alibaba.csp</groupId>
    <artifactId>sentinel-dubbo-adapter</artifactId>
    <version>1.6.0</version>
</dependency>
<dependency>
    <groupId>com.alibaba.csp</groupId>
    <artifactId>sentinel-transport-simple-http</artifactId>
    <version>1.6.0</version>
</dependency>
<!--集群流控-->
<dependency>
    <groupId>com.alibaba.csp</groupId>
    <artifactId>sentinel-cluster-client-default</artifactId>
    <version>1.6.0</version>
</dependency>
<dependency>
    <groupId>com.alibaba.csp</groupId>
    <artifactId>sentinel-cluster-server-default</artifactId>
    <version>1.6.0</version>
</dependency>
<!--传输数据给dashboard-->
<dependency>
    <groupId>com.alibaba.csp</groupId>
    <artifactId>sentinel-transport-netty-http</artifactId>
    <version>1.6.0</version>
</dependency>
<!--nacos作为规则数据源-->
<dependency>
    <groupId>com.alibaba.csp</groupId>
    <artifactId>sentinel-datasource-nacos</artifactId>
    <version>1.6.0</version>
</dependency>
<!-- 热点参数限流 -->
<dependency>
    <groupId>com.alibaba.csp</groupId>
    <artifactId>sentinel-parameter-flow-control</artifactId>
    <version>1.6.0</version>
</dependency>
<!-- Reactor适配 -->
<dependency>
    <groupId>com.alibaba.csp</groupId>
    <artifactId>sentinel-reactor-adapter</artifactId>
    <version>1.6.0</version>
</dependency>
<!-- 注解配置的支持 -->
<dependency>
    <groupId>com.alibaba.csp</groupId>
    <artifactId>sentinel-annotation-aspectj</artifactId>
    <version>1.6.0</version>
</dependency>
```
#### 指定配置
```
nacos.config.server-addr = localhost:8848
nacos.config.namespace = a2881d2a-5c21-4f9e-9c75-dd93872e9ce8
spring.cloud.sentinel.transport.port = 6719
spring.cloud.sentinel.transport.dashboard = localhost:8088
```

#### 监听配置、更新本地缓存
```
@Slf4j
@Getter
public class SentinelNacosAutoReader {

    @NacosInjected
    private ConfigService configService;

    @PostConstruct
    private void post() {
        AbstractListener flowRuleListen = new AbstractListener() {
            @Override
            public void receiveConfigInfo(String configInfo) {
                try {
                    List<FlowRule> rules = JSON.parseObject(configInfo, new TypeReference<List<FlowRule>>() {
                    });
                    FlowRuleManager.loadRules(rules);
                } catch (Exception e) {
                    log.error("FlowRule loadRules error", e);
                }
            }
        };
        AbstractListener degradeRuleLisener = new AbstractListener() {
            @Override
            public void receiveConfigInfo(String configInfo) {
                try {
                    List<DegradeRule> rules = JSON.parseObject(configInfo, new TypeReference<List<DegradeRule>>() {
                    });
                    DegradeRuleManager.loadRules(rules);
                } catch (Exception e) {
                    log.error("DegradeRule loadRules error", e);
                }
                log.info(configInfo);
            }
        };
        AbstractListener systemRuleListener = new AbstractListener() {
            @Override
            public void receiveConfigInfo(String configInfo) {
                try {
                    List<SystemRule> rules = JSON.parseObject(configInfo, new TypeReference<List<SystemRule>>() {
                    });
                    SystemRuleManager.loadRules(rules);
                } catch (Exception e) {
                    log.error("SystemRule loadRules error", e);
                }
                log.info(configInfo);
            }
        };
        AbstractListener paramFlowRuleListener = new AbstractListener() {
            @Override
            public void receiveConfigInfo(String configInfo) {
                try {
                    List<ParamFlowRule> rules = JSON.parseObject(configInfo, new TypeReference<List<ParamFlowRule>>() {
                    });
                    ParamFlowRuleManager.loadRules(rules);
                } catch (Exception e) {
                    log.error("ParamFlowRule loadRules error", e);
                }
                log.info(configInfo);
            }
        };
        AbstractListener authorityRuleListener = new AbstractListener() {
            @Override
            public void receiveConfigInfo(String configInfo) {
                try {
                    List<AuthorityRule> rules = JSON.parseObject(configInfo, new TypeReference<List<AuthorityRule>>() {
                    });
                    AuthorityRuleManager.loadRules(rules);
                } catch (Exception e) {
                    log.error("AuthorityRule loadRules error", e);
                }
                log.info(configInfo);
            }
        };

        try {
            SentinelNacosConsts consts = new SentinelNacosConsts();
            configService.addListener(consts.getFlowRuleFileName(), consts.getGroupName(), flowRuleListen);
            configService.addListener(consts.getDegradeRuleFileName(), consts.getGroupName(), degradeRuleLisener);
            configService.addListener(consts.getSystemRuleFileName(), consts.getGroupName(), systemRuleListener);
            configService.addListener(consts.getAuthorityFileName(), consts.getGroupName(), authorityRuleListener);
            configService.addListener(consts.getParamFlowFileName(), consts.getGroupName(), paramFlowRuleListener);
            log.info("完成sentinel from nacos 配置的监听");
        } catch (NacosException e) {
            log.error("sentinel配置监听失败", e);
        }
    }
}
```

### 小结
Sentinel 控制台 → 配置中心（nacos、apollo） → Sentinel 数据源(mysql) → Sentinel 客户端监听到配置中心的变化更新到本地缓存