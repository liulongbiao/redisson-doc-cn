# 其它特性

## 操作节点

Redisson NodesGroup 对象提供了对 Redis 节点的一些控制：

```java
NodesGroup nodesGroup = redisson.getNodesGroup();
nodesGroup.addConnectionListener(new ConnectionListener() {
    public void onConnect(InetSocketAddress addr) {
       // Redis server connected
    }

    public void onDisconnect(InetSocketAddress addr) {
       // Redis server disconnected
    }
});
```

允许给单个或所有 Redis 服务器发送 ping 请求：

```java
NodesGroup nodesGroup = redisson.getNodesGroup();
Collection<Node> allNodes = nodesGroup.getNodes();
for (Node n : allNodes) {
    n.ping();
}
// or
nodesGroup.pingAll();
```

## 执行批量命令

通过 `RBatch` 对象可以将多个命令汇总到一个网络调用中一次性发送并执行。
通过这个对象你可以一组命令的执行时间。
在 Redis 中这种方式称为 [Pipeling](http://redis.io/topics/pipelining)。

```java
RBatch batch = redisson.createBatch();
batch.getMap("test").fastPutAsync("1", "2");
batch.getMap("test").fastPutAsync("2", "3");
batch.getMap("test").putAsync("2", "5");
batch.getAtomicLongAsync("counter").incrementAndGetAsync();
batch.getAtomicLongAsync("counter").incrementAndGetAsync();

List<?> res = batch.execute();
```

## 脚本

```java
redisson.getBucket("foo").set("bar");
String r = redisson.getScript().eval(Mode.READ_ONLY, 
   "return redis.call('get', 'foo')", RScript.ReturnType.VALUE);

// do the same using cache
RScript s = redisson.getScript();
// load script into cache to all redis master instances
String res = s.scriptLoad("return redis.call('get', 'foo')");
// res == 282297a0228f48cd3fc6a55de6316f31422f5d17

// call script by sha digest
Future<Object> r1 = redisson.getScript().evalShaAsync(Mode.READ_ONLY, 
   "282297a0228f48cd3fc6a55de6316f31422f5d17", 
   RScript.ReturnType.VALUE, Collections.emptyList());
```

## Spring Cache 集成

Redisson 完全支持 [Spring Cache 抽象](https://github.com/mrniko/redisson/wiki/docs.spring.io/spring-framework/docs/current/spring-framework-reference/html/cache.html).
每个 Cache 实例具有两个重要参数： `ttl` 和 `maxIdleTime`，
且在它们没有定义或者等于 `0` 时会永久存储。
配置示例：

```java
@Configuration
@ComponentScan
@EnableCaching
public static class Application {

    @Bean(destroyMethod="shutdown")
    RedissonClient redisson() throws IOException {
        Config config = new Config();
        config.useClusterServers()
              .addNodeAddress("127.0.0.1:7004", "127.0.0.1:7001");
        return Redisson.create(config);
    }

    @Bean
    CacheManager cacheManager(RedissonClient redissonClient) {
        Map<String, CacheConfig> config = new HashMap<String, CacheConfig>();
        config.put("testMap", new CacheConfig(24*60*1000, 12*60*1000));
        return new RedissonSpringCacheManager(redissonClient, config);
    }

}
```

Cache 配置也可以从 JSON 或 YAML 配置文件中读取：

```java
@Configuration
@ComponentScan
@EnableCaching
public static class Application {

    @Bean(destroyMethod="shutdown")
    RedissonClient redisson(@Value("classpath:/redisson.json") Resource configFile) throws IOException {
        Config config = Config.fromJSON(configFile.getInputStream());
        return Redisson.create(config);
    }

    @Bean
    CacheManager cacheManager(RedissonClient redissonClient) throws IOException {
        return new RedissonSpringCacheManager(redissonClient, "classpath:/cache-config.json");
    }

}
```

## 底层 Redis 客户端

Redisson 使用高性能异步且无锁的 Redis 客户端。
它支持异步和同步模式。
如果 Redisson 尚不支持，你可以通过它自行执行命令。
当然，在使用底层客户端之前，你可能想在 [Redis命令映射](./cmd_mapping.md) 查询一下。

```java
RedisClient client = new RedisClient("localhost", 6379);
RedisConnection conn = client.connect();
//or 
Future<RedisConnection> connFuture = client.connectAsync();

conn.sync(StringCodec.INSTANCE, RedisCommands.SET, "test", 0);
conn.async(StringCodec.INSTANCE, RedisCommands.GET, "test");

conn.sync(RedisCommands.PING);

conn.close()
// or
conn.closeAsync()

client.shutdown();
// or
client.shutdownAsync();
```
