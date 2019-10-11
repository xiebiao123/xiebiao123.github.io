---
title: Spring Session 共享分析
date: 2018-09-26
categories:
    - 源码
tags:
    - Spring
---

### spring-session管理session分析
* DelegatingFilterProxy 代理类

```
@Override
protected void initFilterBean() throws ServletException {
    synchronized (this.delegateMonitor) {
        if (this.delegate == null) {
            // If no target bean name specified, use filter name.
            if (this.targetBeanName == null) {
                this.targetBeanName = getFilterName();
            }
            // Fetch Spring root application context and initialize the delegate early,
            // if possible. If the root application context will be started after this
            // filter proxy, we'll have to resort to lazy initialization.
            WebApplicationContext wac = findWebApplicationContext();
            if (wac != null) {
                this.delegate = initDelegate(wac);
            }
        }
    }
}
```
DelegatingFilterProxy里没有实现过滤器的任何逻辑，具体逻辑在其指定的filter-name过滤器中。初始化过滤器，如果没有配置targetBeanName，
则直接使用filter-name，这里指定的是springSessionRepositoryFilter，这个名称是一个固定值此filter在RedisHttpSessionConfiguration中被定义。

* RedisHttpSessionConfiguration 配置类
```
@Bean
public <S extends ExpiringSession> SessionRepositoryFilter<? extends ExpiringSession> springSessionRepositoryFilter(
        SessionRepository<S> sessionRepository) {
    SessionRepositoryFilter<S> sessionRepositoryFilter = new SessionRepositoryFilter<S>(
            sessionRepository);
    sessionRepositoryFilter.setServletContext(this.servletContext);
    if (this.httpSessionStrategy instanceof MultiHttpSessionStrategy) {
        sessionRepositoryFilter.setHttpSessionStrategy(
                (MultiHttpSessionStrategy) this.httpSessionStrategy);
    }
    else {
        sessionRepositoryFilter.setHttpSessionStrategy(this.httpSessionStrategy);
    }
    return sessionRepositoryFilter;
}
``` 
在RedisHttpSessionConfiguration的父类SpringHttpSessionConfiguration中定义了springSessionRepositoryFilter,此方法返回值是
SessionRepositoryFilter，这个其实就是真实的过滤器。此方法的参数SessionRepository的实现类RedisOperationsSessionRepository
其实就是就是将session持久化到redis中

* SessionRepositoryFilter 过滤器

```
@Override
protected void doFilterInternal(HttpServletRequest request,
        HttpServletResponse response, FilterChain filterChain)
        throws ServletException, IOException {
    request.setAttribute(SESSION_REPOSITORY_ATTR, this.sessionRepository);

    SessionRepositoryRequestWrapper wrappedRequest = new SessionRepositoryRequestWrapper(
            request, response, this.servletContext);
    SessionRepositoryResponseWrapper wrappedResponse = new SessionRepositoryResponseWrapper(
            wrappedRequest, response);

    HttpServletRequest strategyRequest = this.httpSessionStrategy
            .wrapRequest(wrappedRequest, wrappedResponse);
    HttpServletResponse strategyResponse = this.httpSessionStrategy
            .wrapResponse(wrappedRequest, wrappedResponse);

    try {
        filterChain.doFilter(strategyRequest, strategyResponse);
    }
    finally {
        wrappedRequest.commitSession();
    }
}
```
所有的请求都会先经过SessionRepositoryFilter过滤器，request被包装成了SessionRepositoryRequestWrapper对象，response被包装
成了SessionRepositoryResponseWrapper对象，SessionRepositoryRequestWrapper中重写了getSession等方法；finally中执行了commitSession
方法，将session进行持久化操作

* SessionRepositoryRequestWrapper 包装类

```
@Override
public HttpSessionWrapper getSession(boolean create) {
    HttpSessionWrapper currentSession = getCurrentSession();
    if (currentSession != null) {
        return currentSession;
    }
    S requestedSession = getRequestedSession();
    if (requestedSession != null) {
        if (getAttribute(INVALID_SESSION_ID_ATTR) == null) {
            requestedSession.setLastAccessedTime(Instant.now());
            this.requestedSessionIdValid = true;
            currentSession = new HttpSessionWrapper(requestedSession, getServletContext());
            currentSession.setNew(false);
            setCurrentSession(currentSession);
            return currentSession;
        }
    }
    else {
        // This is an invalid session id. No need to ask again if
        // request.getSession is invoked for the duration of this request
        if (SESSION_LOGGER.isDebugEnabled()) {
            SESSION_LOGGER.debug(
                    "No session found by id: Caching result for getSession(false) for this HttpServletRequest.");
        }
        setAttribute(INVALID_SESSION_ID_ATTR, "true");
    }
    if (!create) {
        return null;
    }
    if (SESSION_LOGGER.isDebugEnabled()) {
        SESSION_LOGGER.debug(
                "A new session was created. To help you troubleshoot where the session was created we provided a StackTrace (this is not an error). You can prevent this from appearing by disabling DEBUG logging for "
                        + SESSION_LOGGER_NAME,
                new RuntimeException(
                        "For debugging purposes only (not an error)"));
    }
    S session = SessionRepositoryFilter.this.sessionRepository.createSession();
    session.setLastAccessedTime(Instant.now());
    currentSession = new HttpSessionWrapper(session, getServletContext());
    setCurrentSession(currentSession);
    return currentSession;
}
```

重点看一下重写的getSession方法,大致分为三步
1. 首先去本地内存中获取session，如果获取不到去指定的数据库中获取，这里其实就是去redis里面获取,sessionRepository就是上面定义的RedisOperationsSessionRepository对象
2. 如果redis里面也没有则创建一个新的session
3. 然后再将session设置到redis中

### 保存的session
每次在消息处理完之后，会执行finally中的commitSession方法，每个session被保存都会创建三组数据，
* **hash结构记录** key格式：**spring:session:sessions:[sessionId]**，对应的value保存session的所有数据包括：creationTime，maxInactiveInterval，lastAccessedTime，attribute
* **set结构记录** key格式：**spring:session:expirations:[过期时间]**，对应的value为expires:[sessionId]列表，有效期默认是30分钟，即1800秒
* **string结构记录** key格式：**spring:session:sessions:expires:[sessionId]**，对应的value为空；该数据的TTL表示sessionId过期的剩余时间

### 定时清理
除了依赖redis本身的有效期机制，spring-session提供了一个定时器，用来定期检查需要被清理的session，同样是通过roundDownMinute方法来获取key，
获取这一分钟内要被删除的session，此value是set数据结构，里面存放这需要被删除的sessionId
> 注：这里面存放的的是spring:session:sessions:expires:[sessionId]，并不是实际存储session数据的spring:session:sessions:[sessionId]

首先删除了spring:session:expirations:[过期时间]，然后遍历set执行touch方法，并没有直接执行删除操作，看touch方法的注释大致意义就是尝试访问一下key，
如果key已经过去则触发删除操作，利用了redis本身的特性

### 键空间通知
定期删除机制并没有删除实际存储session数据的spring:session:sessions:[sessionId]，这里利用了redis的keyspace notification功能，大致就是通过命令产生一个通知，具体什么命令可以配置（包括：删除，过期等）具体可以查看

[参考地址](http://www.sohu.com/a/260010804_464962)