# 分布式集合

## <a id="71-map"></a> Map

Redisson 分布式的 Map 对象，实现了 `java.util.concurrent.ConcurrentMap`
和 `java.util.Map` 接口。
Map 的大小由 Redis 限制为 `4 294 967 295`。

```java
RMap<String, SomeObject> map = redisson.getMap("anyMap");
SomeObject prevObject = map.put("123", new SomeObject());
SomeObject currentObject = map.putIfAbsent("323", new SomeObject());
SomeObject obj = map.remove("123");

map.fastPut("321", new SomeObject());
map.fastRemove("321");

Future<SomeObject> putAsyncFuture = map.putAsync("321");
Future<Void> fastPutAsyncFuture = map.fastPutAsync("321");

map.fastPutAsync("321", new SomeObject());
map.fastRemoveAsync("321");
```

[Redisson PRO](http://redisson.pro/) 版本的 Map 对象
在集群模式中支持 [数据分区](./partition.md)。

## <a id="711-map-eviction"></a> Map eviction

Redisson 分布式的 Map 可通过独立的 MapCache 对象支持 eviction。
它也实现了  `java.util.concurrent.ConcurrentMap`
和 `java.util.Map` 接口。
Redisson 有一个基于 Map 和 MapCache 对象的
[Spring Cache 集成](https://github.com/mrniko/redisson/wiki/7.-additional-features#74-spring-cache-integration)。

当前 Redis 实现中没有 map 项的 eviction 功能。
因此，过期的项由 `org.redisson.EvictionScheduler` 来清理。
它一次可移除 100 条过期项。
任务的调度时间会根据上次任务中删除的过期项数量自动调整，时间在 1 秒到 2 个小时内。
因此若清理任务每次删除了 100 项数据，它将每秒钟执行一次(最小的执行延迟)。
但如果当前过期项数量比前一次少，则执行延迟将扩大为 1.5 倍。

```java
RMapCache<String, SomeObject> map = redisson.getMapCache("anyMap");
// ttl = 10 minutes, 
map.put("key1", new SomeObject(), 10, TimeUnit.MINUTES);
// ttl = 10 minutes, maxIdleTime = 10 seconds
map.put("key1", new SomeObject(), 10, TimeUnit.MINUTES, 10, TimeUnit.SECONDS);

// ttl = 3 seconds
map.putIfAbsent("key2", new SomeObject(), 3, TimeUnit.SECONDS);
// ttl = 40 seconds, maxIdleTime = 10 seconds
map.putIfAbsent("key2", new SomeObject(), 40, TimeUnit.SECONDS, 10, TimeUnit.SECONDS);
```

## <a id="72-multimap"></a> MultiMap

Redisson 分布式的 MultiMap 对象允许给每个键绑定多个值。
键的数量限制由 Redis 限制为 `4 294 967 295`。

## <a id="721-set-based-multimap"></a> 基于 Set 的 MultiMap

基于 Set 的 MultiMap 不允许每个键中的值有重复。

```java
RSetMultimap<SimpleKey, SimpleValue> map = redisson.getSetMultimap("myMultimap");
map.put(new SimpleKey("0"), new SimpleValue("1"));
map.put(new SimpleKey("0"), new SimpleValue("2"));
map.put(new SimpleKey("3"), new SimpleValue("4"));

Set<SimpleValue> allValues = map.get(new SimpleKey("0"));

List<SimpleValue> newValues = Arrays.asList(new SimpleValue("7"), new SimpleValue("6"), new SimpleValue("5"));
Set<SimpleValue> oldValues = map.replaceValues(new SimpleKey("0"), newValues);

Set<SimpleValue> removedValues = map.removeAll(new SimpleKey("0"));
```

## <a id="722-list-based-multimap"></a> 基于 List 的 MultiMap

基于 List 的 MultiMap 会存储插入的顺序且允许键对应的值中存在重复。

```java
RListMultimap<SimpleKey, SimpleValue> map = redisson.getListMultimap("test1");
map.put(new SimpleKey("0"), new SimpleValue("1"));
map.put(new SimpleKey("0"), new SimpleValue("2"));
map.put(new SimpleKey("0"), new SimpleValue("1"));
map.put(new SimpleKey("3"), new SimpleValue("4"));

List<SimpleValue> allValues = map.get(new SimpleKey("0"));

Collection<SimpleValue> newValues = Arrays.asList(new SimpleValue("7"), new SimpleValue("6"), new SimpleValue("5"));
List<SimpleValue> oldValues = map.replaceValues(new SimpleKey("0"), newValues);

List<SimpleValue> removedValues = map.removeAll(new SimpleKey("0"));
```

## <a id="723-multimap-eviction"></a> MultiMap eviction

Multimap 对象可通过独立的 MultimapCache 对象来支持 eviction。
它对基于 Set 和 List 的 MultiMap 分别有 `RSetMultimapCache` 和 `RListMultimapCache` 对象。

过期的项由 `org.redisson.EvictionScheduler` 来清理。
它一次可移除 100 条过期项。
任务的调度时间会根据上次任务中删除的过期项数量自动调整，时间在 1 秒到 2 个小时内。
因此若清理任务每次删除了 100 项数据，它将每秒钟执行一次(最小的执行延迟)。
但如果当前过期项数量比前一次少，则执行延迟将扩大为 1.5 倍。

RSetMultimapCache 示例：

```java
RSetMultimapCache<String, String> multimap = redisson.getSetMultimapCache("myMultimap");
map.put("1", "a");
map.put("1", "b");
map.put("1", "c");

map.put("2", "e");
map.put("2", "f");

map.expireKey("2", 10, TimeUnit.MINUTES);
```

## <a id="73-set"></a> Set

Redisson 分布式的 Set 对象，实现了 `java.util.Set` 接口。
它通过元素状态比较来保持元素的独立性。
Set 大小由 Redis 限制为 `4 294 967 295`。

```java
RSet<SomeObject> set = redisson.getSet("anySet");
set.add(new SomeObject());
set.remove(new SomeObject());
```

[Redisson PRO](http://redisson.pro/) 版本的 Set 对象
在集群模式中支持 [数据分区](./partition.md)。

## <a id="731-set-eviction"></a> Set eviction

Redisson 分布式的 Set 对象可通过独立的 SetCache 对象来支持 eviction。
它也实现了 `java.util.Set` 接口。

当前 Redis 实现中没有 set 值的 eviction 功能。
因此，过期的项由 `org.redisson.EvictionScheduler` 来清理。
它一次可移除 100 条过期项。
任务的调度时间会根据上次任务中删除的过期项数量自动调整，时间在 1 秒到 2 个小时内。
因此若清理任务每次删除了 100 项数据，它将每秒钟执行一次(最小的执行延迟)。
但如果当前过期项数量比前一次少，则执行延迟将扩大为 1.5 倍。

```java
RSetCache<SomeObject> set = redisson.getSetCache("anySet");
// ttl = 10 seconds
set.add(new SomeObject(), 10, TimeUnit.SECONDS);
```

## <a id="74-sortedset"></a> SortedSet

Redisson 分布式的 SortedSet 对象，实现了 `java.util.SortedSet` 接口。
它通过比较器来排列元素并保持唯一性。

```java
RSortedSet<Integer> set = redisson.getSortedSet("anySet");
set.trySetComparator(new MyComparator()); // set object comparator
set.add(3);
set.add(1);
set.add(2);

set.removeAsync(0);
set.addAsync(5);
```

## <a id="75-scoredsortedset"></a> ScoredSortedSet

Redisson 分布式的 ScoredSortedSet 对象。
它通过在元素插入时定义的分数来排列元素，
通过元素状态的比较来保持元素的唯一性。

```java
RScoredSortedSet<SomeObject> set = redisson.getScoredSortedSet("simple");

set.add(0.13, new SomeObject(a, b));
set.addAsync(0.251, new SomeObject(c, d));
set.add(0.302, new SomeObject(g, d));

set.pollFirst();
set.pollLast();

int index = set.rank(new SomeObject(g, d)); // get element index
Double score = set.getScore(new SomeObject(g, d)); // get element score
```

## <a id="76-lexsortedset"></a> LexSortedSet

Redisson 分布式的 Set 对象，它仅允许字典序的 String 对象，并实现了 `java.util.Set<String>` 接口。
它通过元素状态比较来保持元素的唯一性。

```java
RLexSortedSet set = redisson.getLexSortedSet("simple");
set.add("d");
set.addAsync("e");
set.add("f");

set.lexRangeTail("d", false);
set.lexCountHead("e");
set.lexRange("d", true, "z", false);
```

## <a id="77-list"></a> List

Redisson 分布式的 List 对象，实现了 `java.util.List` 接口。
它保留元素的插入顺序。
List 大小由 Redis 限制为 `4 294 967 295`。

```java
RList<SomeObject> list = redisson.getList("anyList");
list.add(new SomeObject());
list.get(0);
list.remove(new SomeObject());
```

## <a id="78-queue"></a> Queue

Redisson 分布式的 Queue 对象，实现了 `java.util.Queue` 接口。
Queue 大小由 Redis 限制为 `4 294 967 295`。

```java
RQueue<SomeObject> queue = redisson.getQueue("anyQueue");
queue.add(new SomeObject());
SomeObject obj = queue.peek();
SomeObject someObj = queue.poll();
```

## <a id="79-deque"></a> Deque

Redisson 分布式的 Deque 对象，实现了 `java.util.Deque` 接口。
Deque 大小由 Redis 限制为 `4 294 967 295`。

```java
RDeque<SomeObject> queue = redisson.getDeque("anyDeque");
queue.addFirst(new SomeObject());
queue.addLast(new SomeObject());
SomeObject obj = queue.removeFirst();
SomeObject someObj = queue.removeLast();
```

## <a id="710-blocking-queue"></a> BlockingQueue

Redisson 分布式的 BlockingQueue 对象，实现了 `java.util.concurrent.BlockingQueue` 接口。
BlockingQueue 大小由 Redis 限制为 `4 294 967 295`。

```java
RBlockingQueue<SomeObject> queue = redisson.getBlockingQueue("anyQueue");
queue.offer(new SomeObject());

SomeObject obj = queue.peek();
SomeObject someObj = queue.poll();
SomeObject ob = queue.poll(10, TimeUnit.MINUTES);
```

`poll`、`pollFromAny`、`pollLastAndOfferFirstTo` 和 `take` 方法
在重连到 Redis 服务器或 Redis 服务器故障恢复时会自动被重新订阅。

## <a id="711-blocking-deque"></a> BlockingDeque

Redisson 分布式的 BlockingDeque 对象，实现了 `java.util.concurrent.BlockingDeque` 接口。
BlockingDeque 大小由 Redis 限制为 `4 294 967 295`。

```java
RBlockingDeque<Integer> deque = redisson.getBlockingDeque("anyDeque");
deque.putFirst(1);
deque.putLast(2);
Integer firstValue = queue.takeFirst();
Integer lastValue = queue.takeLast();
Integer firstValue = queue.pollFirst(10, TimeUnit.MINUTES);
Integer lastValue = queue.pollLast(3, TimeUnit.MINUTES);
```

`poll`、`pollFromAny`、`pollLastAndOfferFirstTo` 和 `take` 方法
在重连到 Redis 服务器或 Redis 服务器故障恢复时会自动被重新订阅。
