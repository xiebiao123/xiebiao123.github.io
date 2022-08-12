---
title: Redisson-分布式锁
date: 2022-04-18
categories:
	- 学习
tags:
    - redisson
---

#### 加锁 redissonLock.lock()
##### 1.1 lock()
> 当我们进入到 Redisson 的lock方法时，会走到下面的代码逻辑。
> 1. 尝试去获取锁。
> 2. 获取锁成功的话，走1.2，去构建看门狗什么的。
> 3. 获取锁失败的话，进入自旋，并等待相应的时间去重新获取锁，知道锁获取成功。

``` java
private void lock(long leaseTime, TimeUnit unit, boolean interruptibly) throws InterruptedException {
	// 获取当前线程id
    long threadId = Thread.currentThread().getId();
    // 注意，这里传入了一个 -1L
    Long ttl = this.tryAcquire(-1L, leaseTime, unit, threadId);
    // 这里 ttl 不等于 null的时候，说明没有获取到锁
    if (ttl != null) {
        RFuture<RedissonLockEntry> future = this.subscribe(threadId);
        if (interruptibly) {
            this.commandExecutor.syncSubscriptionInterrupted(future);
        } else {
            this.commandExecutor.syncSubscription(future);
        }

        try {
        	// 进行自旋，然后再次尝试获取锁
            while(true) {
                ttl = this.tryAcquire(-1L, leaseTime, unit, threadId);
                if (ttl == null) {
                    return;
                }

                if (ttl >= 0L) {
                    try {
                        ((RedissonLockEntry)future.getNow()).getLatch().tryAcquire(ttl, TimeUnit.MILLISECONDS);
                    } catch (InterruptedException var13) {
                        if (interruptibly) {
                            throw var13;
                        }

                        ((RedissonLockEntry)future.getNow()).getLatch().tryAcquire(ttl, TimeUnit.MILLISECONDS);
                    }
                } else if (interruptibly) {
                    ((RedissonLockEntry)future.getNow()).getLatch().acquire();
                } else {
                    ((RedissonLockEntry)future.getNow()).getLatch().acquireUninterruptibly();
                }
            }
        } finally {
            this.unsubscribe(future, threadId);
        }
    }
}
```

##### 1.2 tryAcquire()
> 这个方法进来之后执行的是 tryAcquireAsync 方法
> 1. 先去尝试获取锁。
> 2. 获取锁成功的话，就去设置一个看门狗的定时任务，用于给锁续命用。
> 3. 获取锁失败的话，就退出这个方法，走1.1，进入自旋状态重新获取锁。

```java
private <T> RFuture<Long> tryAcquireAsync(long waitTime, long leaseTime, TimeUnit unit, long threadId) {
    RFuture ttlRemainingFuture;
    // 由于前面传入的值为 -1L ，所以这里不走
    if (leaseTime != -1L) {
        ttlRemainingFuture = this.tryLockInnerAsync(waitTime, leaseTime, unit, threadId, RedisCommands.EVAL_LONG);
    } else {
    	// 进入这个方法获取锁，获取锁成功的话，则会返回null
    	// 获取锁失败的话，会返回当前拥有锁的线程还有多长时间，进入 1.3
        ttlRemainingFuture = this.tryLockInnerAsync(waitTime, this.internalLockLeaseTime, TimeUnit.MILLISECONDS, threadId, RedisCommands.EVAL_LONG);
    }

	// 在这里主要就是当获取锁成功后，设置一个定时任务
    ttlRemainingFuture.onComplete((ttlRemaining, e) -> {
        if (e == null) {
        	// 剩余时间等于null，说明获取锁成功了
            if (ttlRemaining == null) {
                if (leaseTime != -1L) {
                    this.internalLockLeaseTime = unit.toMillis(leaseTime);
                } else {
                	// 设置一个定时任务，走1.4
                    this.scheduleExpirationRenewal(threadId);
                }
            }

        }
    });
    return ttlRemainingFuture;
}
```

##### 1.3 tryLockInnerAsync()
> 进来之后我们发现这里有一个lua脚本
> 1. 尝试获取锁，如果获取到了就创建定时任务，指定看门狗对相应的锁续命
> 2. 如果获取锁失败，就返回拥有锁线程的剩余时间

```java
<T> RFuture<T> tryLockInnerAsync(long waitTime, long leaseTime, TimeUnit unit, long threadId, RedisStrictCommand<T> command) {
    return this.evalWriteAsync(this.getRawName(), LongCodec.INSTANCE, command, 
    	// 这里面的 KEYS[1] 就是我们在最开始获取的Redis的那把锁，看那个属性或者说是锁是否存在
    	"if (redis.call('exists', KEYS[1]) == 0) " +
        "then " +
        	// 如果不存在，走下面逻辑，给当前线程设置值加1，也就是1
            "redis.call('hincrby', KEYS[1], ARGV[2], 1); " +
            // 设置过期时间
            "redis.call('pexpire', KEYS[1], ARGV[1]); " +
            "return nil; " +
        "end; " +
        // 这块的意思是锁存在，但是获取锁的是当前线程，支持可重入锁
        "if (redis.call('hexists', KEYS[1], ARGV[2]) == 1) " +
        "then " +
        	// 给当前线程的数值加1
            "redis.call('hincrby', KEYS[1], ARGV[2], 1); " +
            // 设置过期时间
            "redis.call('pexpire', KEYS[1], ARGV[1]); " +
            "return nil; " +
        "end; " +
        // 没有获取到锁，查看一下锁的有效期并返回
        "return redis.call('pttl', KEYS[1]);", Collections.singletonList(this.getRawName()), new Object[]{unit.toMillis(leaseTime), this.getLockName(threadId)});
}
```

##### 1.4 scheduleExpirationRenewal(long threadId)
> 主要作用就是去调用循环续命的那个方法

```java
protected void scheduleExpirationRenewal(long threadId) {
    RedissonBaseLock.ExpirationEntry entry = new RedissonBaseLock.ExpirationEntry();
    RedissonBaseLock.ExpirationEntry oldEntry = (RedissonBaseLock.ExpirationEntry)EXPIRATION_RENEWAL_MAP.putIfAbsent(this.getEntryName(), entry);
    if (oldEntry != null) {
        oldEntry.addThreadId(threadId);
    } else {
        entry.addThreadId(threadId);

        try {
        	// 走到这里，里面会递归调用这个方法续命，否则销毁，走1.5
            this.renewExpiration();
        } finally {
            if (Thread.currentThread().isInterrupted()) {
                this.cancelExpirationRenewal(threadId);
            }

        }
    }

}
```

##### 1.5 renewExpiration()
> 这个方法的主要作用就是
> 1. 设置一个定时任务，超过指定时间的 1/3 就会循环调用一次。
> 2. 判断当前线程是否存在，如果存在，根据指定的时间重置过期时间，如果不存在，说明任务已经执行完成，销毁看门狗线程。

```java
private void renewExpiration() {
    RedissonBaseLock.ExpirationEntry ee = (RedissonBaseLock.ExpirationEntry)EXPIRATION_RENEWAL_MAP.get(this.getEntryName());
    if (ee != null) {
    	// 这里设置一个定时任务
        Timeout task = this.commandExecutor.getConnectionManager().newTimeout(new TimerTask() {
            public void run(Timeout timeout) throws Exception {
                RedissonBaseLock.ExpirationEntry ent = (RedissonBaseLock.ExpirationEntry)RedissonBaseLock.EXPIRATION_RENEWAL_MAP.get(RedissonBaseLock.this.getEntryName());
                if (ent != null) {
                    Long threadId = ent.getFirstThreadId();
                    // 给线程续命，里面调用set方法充重置线程时间
                    if (threadId != null) {
                        RFuture<Boolean> future = RedissonBaseLock.this.renewExpirationAsync(threadId);
                        future.onComplete((res, e) -> {
                            if (e != null) {
                                RedissonBaseLock.log.error("Can't update lock " + RedissonBaseLock.this.getRawName() + " expiration", e);
                                RedissonBaseLock.EXPIRATION_RENEWAL_MAP.remove(RedissonBaseLock.this.getEntryName());
                            } else {
                                if (res) {
                                	// 如果线程还存在，就递归调用并续命
                                    RedissonBaseLock.this.renewExpiration();
                                } else {
                                	// 如果线程不存在，就关闭销毁线程
                                    RedissonBaseLock.this.cancelExpirationRenewal((Long)null);
                                }

                            }
                        });
                    }
                }
            }
        }, 
        // 每次过设置时间的 1/3 时，就会再次执行定时任务
		this.internalLockLeaseTime / 3L, TimeUnit.MILLISECONDS);
        ee.setTimeout(task);
    }
}
```

#### 2. 解锁 redissonLock.unlock()
##### 2.1、unlock()
> 主方法，调用 unlockAsync() 方法，走2.2

```java
public void unlock() {
    try {
        this.get(this.unlockAsync(Thread.currentThread().getId()));
    } catch (RedisException var2) {
        if (var2.getCause() instanceof IllegalMonitorStateException) {
            throw (IllegalMonitorStateException)var2.getCause();
        } else {
            throw var2;
        }
    }
}
```

##### 2.2、unlockAsync(long threadId)
> 1. 修改状态，释放锁等，走2.3
> 2. 删除线程，释放一些其它资源

```java
public RFuture<Void> unlockAsync(long threadId) {
    RPromise<Void> result = new RedissonPromise();
    // 解锁的主要操作，走2.3
    RFuture<Boolean> future = this.unlockInnerAsync(threadId);
    future.onComplete((opStatus, e) -> {
    	// 关闭这个锁的线程资源什么的
        this.cancelExpirationRenewal(threadId);
        if (e != null) {
            result.tryFailure(e);
        } else if (opStatus == null) {
            IllegalMonitorStateException cause = new IllegalMonitorStateException("attempt to unlock lock, not locked by current thread by node id: " + this.id + " thread-id: " + threadId);
            result.tryFailure(cause);
        } else {
            result.trySuccess((Object)null);
        }
    });
    return result;
}
```

##### 2.3、unlockInnerAsync()
> 1. 主要对锁的状态进行修改，lock一次就加1，unlock一次就减1。
> 2. 当减到0的时候就删除这个数据，就是释放锁，并广播其它线程可以加锁了。

```java
protected RFuture<Boolean> unlockInnerAsync(long threadId) {
    return this.evalWriteAsync(this.getRawName(), LongCodec.INSTANCE, RedisCommands.EVAL_BOOLEAN, 
    // 判断当前这个锁是否存在
    "if (redis.call('hexists', KEYS[1], ARGV[3]) == 0) " +
    "then " +
    	// 如果等于0，说明锁已经不存在了，返回null
        "return nil;" +
    "end; " +
    // 定义一个本地变量对锁的状态减去1，并存储在counter中
    "local counter = redis.call('hincrby', KEYS[1], ARGV[3], -1); " +
    // 如果counter大于0
    "if (counter > 0) " +
    "then " +
    	// 设置一个过期时间并返回0
        "redis.call('pexpire', KEYS[1], ARGV[2]); " +
        "return 0; " +
    "else " +
    	// 删除掉这个锁，并广播
        "redis.call('del', KEYS[1]); " +
        "redis.call('publish', KEYS[2], ARGV[1]); " +
        "return 1; " +
    "end; " +
    "return nil;", Arrays.asList(this.getRawName(), this.getChannelName()), new Object[]{LockPubSub.UNLOCK_MESSAGE, this.internalLockLeaseTime, this.getLockName(threadId)});
}
```

#### Redisson 实现的常见锁
1. 重入锁
2. 公平锁与非公平锁
3. 联锁 多个锁组成一个锁
4. 红锁 多数节点获取锁成功，才算获取锁成功
5. 读写锁

#### Redisson 实现延迟队列
``` java
// 延迟队列与阻塞队列绑定
RBlockingDeque<String> blockingDeque = redissonClient.getBlockingDeque(Constant.RedissonQueueName);
RDelayedQueue<String> rDelayedQueue = redissonClient.getDelayedQueue(blockingDeque);

// 循环从阻塞队列中获取数据
while(true){
    RBlockingDeque<String> blockingDeque = redissonClient.getBlockingDeque(Constant.RedissonQueueName);
    if (blockingDeque != null && !blockingDeque.isEmpty()) {
        //本汪说下，此处我们所取到的element，就是我们 rDelayedQueue.offer(value,secondTTL,TimeUnit.SECONDS); 放进去的value了，
        String element = blockingDeque.poll();   
    }
}
```