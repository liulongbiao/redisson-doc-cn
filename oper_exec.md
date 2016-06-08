# 操作执行

Redisson 支持对每个操作自动重试的策略并且在每次尝试期会尝试发送命令。
重试策略由设置项 `retryAttempts` (默认为 `3`) 和 `retryInterval` (默认为 `1000` ms) 来控制。
每次尝试会在 `retryInterval` 时间间隔后执行。

Redisson 实例和 Redisson 对象都是完全线程安全的。

带有同步/异步方法的 Redisson 对象可通过 `RedissonClient` 接口获得。
替代的带有 [Reactive Streams](http://reactive-streams.org/) 方法的 Redisson 对象
可通过 `RedissonReactiveClient` 接口获得。

以下是 `RAtomicLong` 对象的示例：

```java
RedissonClient client = Redisson.create(config);
RAtomicLong longObject = client.getAtomicLong('myLong');
// sync way
longObject.compareAndSet(3, 401);
// async way
longObject.compareAndSetAsync(3, 401);

RedissonReactiveClient client = Redisson.createReactive(config);
RAtomicLongReactive longObject = client.getAtomicLong('myLong');
// reactive way
longObject.compareAndSet(3, 401);
```

## 异步方式

几乎每个 Redisson 对象都扩展了具有和同步方法的镜像的异步方法的一个异步接口。如:

```java
// RAtomicLong extends RAtomicLongAsync
RAtomicLongAsync longObject = client.getAtomicLong("myLong");
Future<Boolean> future = longObject.compareAndSetAsync(1, 401);
```

异步方法返回一个扩展的具有可添加监听器的 `Future` 对象。
这样你可以以完全非阻塞的方式来获取结果。

```java
future.addListener(new FutureListener<Boolean>() {
    @Override
    public void operationComplete(Future<Boolean> future) throws Exception {
         if (future.isSuccess()) {
            // get result
            Boolean result = future.getNow();
            // ...
         } else {
            // an error has occurred
            Throwable cause = future.cause();
         }
    }
});
```

## Reactive 方式

Redisson 通过对 Java 9 的 [Reactive Streams](http://reactive-streams.org/) 标准来支持 Reactive 方式。
基于著名的 [Reactor](http://projectreactor.io/) 项目。
针对 Java 的 Reactive 对象可通过独立的 `RedissonReactiveClient` 接口来获取：

```java
RedissonReactiveClient client = Redisson.createReactive(config);
RAtomicLongReactive longObject = client.getAtomicLong("myLong");

Publisher<Boolean> csPublisher = longObject.compareAndSet(10, 91);

Publisher<Long> getPublisher = longObject.get();
```
