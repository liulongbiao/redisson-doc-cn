# 分布式锁和同步器

## TOC

* [Lock](#81-lock)
* [Fair Lock](#82-fair-lock)
* [MultiLock](#83-multilock)
* [ReadWriteLock](#84-readwritelock)
* [Semaphore](#85-semaphore)
* [CountDownLatch](#86-countdownlatch)

## <a id="81-lock"></a> Lock

Redisson 分布式可重入锁，实现了 `java.util.concurrent.locks.Lock` 接口并支持 TTL。

```java
RLock lock = redisson.getLock("anyLock");
// Most familiar locking method
lock.lock();

// Lock time-to-live support
// releases lock automatically after 10 seconds
// if unlock method not invoked
lock.lock(10, TimeUnit.SECONDS);

// Wait for 100 seconds and automatically unlock it after 10 seconds
boolean res = lock.tryLock(100, 10, TimeUnit.SECONDS);
...
lock.unlock();
```

Redisson 也支持 Lock 对象的异步方法：

```java
RLock lock = redisson.getLock("anyLock");
lock.lockAsync();
lock.lockAsync(10, TimeUnit.SECONDS);
Future<Boolean> res = lock.tryLockAsync(100, 10, TimeUnit.SECONDS);
```

## <a id="82-fair-lock"></a> Fair Lock

Redisson 分布式可重入公平锁，实现了 `java.util.concurrent.locks.Lock` 接口并支持 TTL，
并且保证 Redisson 客户端线程将以其请求的顺序获得锁。
它和简单的 Lock 对象有相同的接口。

```java
RLock fairLock = redisson.getFairLock("anyLock");
// Most familiar locking method
fairLock.lock();

// Lock time-to-live support
// releases lock automatically after 10 seconds
// if unlock method not invoked
fairLock.lock(10, TimeUnit.SECONDS);

// Wait for 100 seconds and automatically unlock it after 10 seconds
boolean res = fairLock.tryLock(100, 10, TimeUnit.SECONDS);
...
fairLock.unlock();
```

Redisson 也支持公平锁对象的异步方法：

```java
RLock fairLock = redisson.getFairLock("anyLock");
fairLock.lockAsync();
fairLock.lockAsync(10, TimeUnit.SECONDS);
Future<Boolean> res = fairLock.tryLockAsync(100, 10, TimeUnit.SECONDS);
```

## <a id="83-multilock"></a> MultiLock

`RedissonMultiLock` 对象可用于实现 Redlock 锁算法。
它将多个 `RLock` 对象划为一组并且将它们当作一个锁来处理。
每个 `RLock` 对象可以属于不同的 Redisson 实例。

```java
RLock lock1 = redissonInstance1.getLock("lock1");
RLock lock2 = redissonInstance2.getLock("lock2");
RLock lock3 = redissonInstance3.getLock("lock3");

RedissonMultiLock lock = new RedissonMultiLock(lock1, lock2, lock3);
// locks: lock1 lock2 lock3
lock.lock();
...
lock.unlock();
```

## <a id="84-readwritelock"></a> ReadWriteLock

Redisson 分布式可重入 ReadWriteLock 对象，实现了 `java.util.concurrent.locks.ReadWriteLock` 接口并支持 TTL。
可以同时存在多个 ReadLock 拥有者，但仅允许有一个 `WriteLock`

```java
RReadWriteLock rwlock = redisson.getLock("anyRWLock");
// Most familiar locking method
rwlock.readLock().lock();
// or
rwlock.writeLock().lock();

// Lock time-to-live support
// releases lock automatically after 10 seconds
// if unlock method not invoked
rwlock.readLock().lock(10, TimeUnit.SECONDS);
// or
rwlock.writeLock().lock(10, TimeUnit.SECONDS);

// Wait for 100 seconds and automatically unlock it after 10 seconds
boolean res = rwlock.readLock().tryLock(100, 10, TimeUnit.SECONDS);
// or
boolean res = rwlock.writeLock().tryLock(100, 10, TimeUnit.SECONDS);
...
lock.unlock();
```

## <a id="85-semaphore"></a> Semaphore 

Redisson 分布式 Semaphore 对象，类似于 `java.util.concurrent.Semaphore` 对象。

```java
RSemaphore semaphore = redisson.getSemaphore("semaphore");
semaphore.acquire();
//or
semaphore.acquireAsync();
semaphore.acquire(23);
semaphore.tryAcquire();
//or
semaphore.tryAcquireAsync();
semaphore.tryAcquire(23, TimeUnit.SECONDS);
//or
semaphore.tryAcquireAsync(23, TimeUnit.SECONDS);
semaphore.release(10);
semaphore.release();
//or
semaphore.releaseAsync();
```

## <a id="86-countdownlatch"></a> CountDownLatch 

Redisson 分布式 CountDownLatch 对象，结构类似于 `java.util.concurrent.CountDownLatch` 对象。

```java
RCountDownLatch latch = redisson.getCountDownLatch("anyCountDownLatch");
latch.trySetCount(1);
latch.await();

// in other thread or other JVM
RCountDownLatch latch = redisson.getCountDownLatch("anyCountDownLatch");
latch.countDown();
```
