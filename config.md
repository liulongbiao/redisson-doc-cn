# 配置

## 编程式配置

编程式配置通过 `Config` 对象实例来执行。如：

```java
Config config = new Config();
config.setUseLinuxNativeEpoll(true);
config.useClusterServers()
      .addNodeAddress("127.0.0.1:7181");
```

## 声明式配置

Redisson 配置可从 JSON 或 YAML 格式的文件中加载。

要从 JSON 读取配置，可使用 `Config.fromJSON` 方法指向配置源来完成：

```java
Config config = Config.fromJSON(new File("config-file.json"));  
RedissonClient redisson = Redisson.create(config);
```

要将配置写出为 JSON ，可使用 `Config.toJSON` 方法：

```java
Config config = new Config();
// ... many settings are set here
String jsonFormat = config.toJSON();
```

要从 YAML 读取配置，可使用 `Config.fromYAML` 方法指向配置源来完成：

```java
Config config = Config.fromYAML(new File("config-file.yaml"));  
RedissonClient redisson = Redisson.create(config);
```

要将配置写出为 YAML ，可使用 `Config.toYAML` 方法：

```java
Config config = new Config();
// ... many settings are set here
String yamlFormat = config.toYAML();
```

## 通用设置

以下设置属于 `org.redisson.Config` 对象，且对所有模式都是通用的：

### codec

默认值： `org.redisson.codec.JsonJacksonCodec`

Redis 数据解编码器。用于读写 Redis 数据。支持多种实现：

| Codec 类名                                                                             |  描述                                                    |
|-----------------------------------------|--------------------------|
| `org.redisson.codec.JsonJacksonCodec`   | [Jackson JSON](https://github.com/FasterXML/jackson) codec |
| `org.redisson.codec.CborJacksonCodec`   | [CBOR](http://cbor.io/) 二进制 json codec |
| `org.redisson.codec.MsgPackJacksonCodec`| [MsgPack](http://msgpack.org/)  二进制 json codec |
| `org.redisson.codec.KryoCodec`          | [Kryo](https://github.com/EsotericSoftware/kryo) 二进制 codec |
| `org.redisson.codec.SerializationCodec` | JDK 序列化 codec |
| `org.redisson.codec.FstCodec`           | [FST](https://github.com/RuedigerMoeller/fast-serialization) 10 倍速且 100% JDK 序列化兼容的 codec |
| `org.redisson.codec.LZ4Codec`           | [LZ4](https://github.com/jpountz/lz4-java) 压缩 codec |
| `org.redisson.codec.SnappyCodec`        | [Snappy](https://github.com/xerial/snappy-java) 压缩 codec |
| `org.redisson.client.codec.StringCodec` | 简单 String codec |
| `org.redisson.client.codec.LongCodec`   | 简单 Long codec |

### threads

默认值： `current_processors_amount * 2`

由所有 redis 节点客户端所共享的线程数量。

### eventLoopGroup

使用外部的 `EventLoopGroup`。EventLoopGroup 通过自己的线程来处理所有和 Redis 服务器相连的 Netty 链接。
每个 Redisson 客户端会默认创建一个自己的 EventLoopGroup。
因此若相同的 JVM 中存在多个 Redisson 实例，它会在它们之间共享一个 EventLoopGroup。

只允许使用 `io.netty.channel.epoll.EpollEventLoopGroup` 或 `io.netty.channel.nio.NioEventLoopGroup`。

### useLinuxNativeEpoll

默认值： `false`

当服务器绑定到 loopback 接口时激活一个 unix socket。
同样也用于激活 epoll 协议。
`netty-transport-native-epoll` 类库需包含在 classpath 中。

## 集群模式

编程式配置示例：

```java
Config config = new Config();
config.useClusterServers()
    .setScanInterval(2000) // cluster state scan interval in milliseconds
    .addNodeAddress("127.0.0.1:7000", "127.0.0.1:7001")
    .addNodeAddress("127.0.0.1:7002");

RedissonClient redisson = Redisson.create(config);
```

### 集群设置

有关 Redis 服务器集群配置的文档在 [这里](http://redis.io/topics/cluster-tutorial)。
最小的集群配置需要至少三个主节点。
集群连接模式可通过以下代码激活：

```java
ClusterServersConfig clusterConfig = config.useClusterServers();
```

`ClusterServersConfig` 设置项如下：

#### addNodeAddress

以 `host:port` 格式添加 Redis 集群节点地址。可以一次性添加多个节点。

#### scanInterval

默认值： `1000`

以毫秒为单位的 Redis 集群扫描间隔。

#### readFromSlaves

默认值： `true`

读操作是否可使用集群从节点。

#### loadBalancer

默认值： `org.redisson.connection.balancer.RoundRobinLoadBalancer`

其它实现有： `org.redisson.connection.balancer.WeightedRoundRobinBalancer`, 
`redisson.connection.balancer.RandomLoadBalancer`

多个 Redis 从服务器间的连接负载均衡器。

#### slaveSubscriptionConnectionMinimumIdleSize

默认值： `1`

对 **每个** 从节点的 Redis '从'节点最小空闲订阅 (pub/sub) 连接量。

#### slaveSubscriptionConnectionPoolSize

默认值： `25`

对 **每个** 从节点的 Redis '从'节点最大订阅 (pub/sub) 连接池大小。

#### slaveConnectionMinimumIdleSize

默认值： `1`

对 **每个** 从节点的 Redis '从'节点最小空闲连接量。

#### slaveConnectionPoolSize

默认值： `100`

对 **每个** 从节点的 Redis '从'节点最大连接池大小。

#### masterConnectionMinimumIdleSize

默认值： `5`

对 **每个** 从节点的 Redis '主'节点最小空闲连接量。

#### masterConnectionPoolSize

默认值： `100`

Redis '主'节点最大连接池大小。

#### idleConnectionTimeout

默认值： `10000`

若池化连接在某段 `timeout` 时间内没有被使用且当前连接量超过最小空闲连接池时，
它将会被关闭并从池中移除。其值的单位是毫秒。

#### connectTimeout

默认值： `1000`

连接到任何 Redis 服务器的超时时间。

#### timeout

默认值： `1000`

Redis 服务器响应的超时时间。从 Redis 命令被成功发送时开始计算。其值的单位是毫秒。

#### retryAttempts

默认值： `3`

若 Redis 命令在超过 `retryAttempts` 次不能发送被 Redis 服务器，则将抛出一个错误。
但若成功发送，则将开始 `timeout`。

#### retryInterval

默认值： `1000`

发送 Redis 命令重试的时间间隔。其值的单位是毫秒。

#### reconnectionTimeout

默认值： `3000`

Redis 服务器重连尝试的超时时间。在每次这种超时事件发生时， Redisson 会尝试连接到失联的 Redis 服务器。
其值的单位是毫秒。

#### failedAttempts

默认值： `3`

当任何 Redis 命令的连续的未成功执行尝试到达 `failedAttempts` 时，
这个 Redis 服务器将被从一个内部的可用从节点列表中移除。

#### database

默认值： `0`

针对 Redis 连接的数据库索引。

#### password

默认值： `null`

Redis 服务器授权的密码。

#### subscriptionsPerConnection

默认值： `5`

每个 Redis 连接上的订阅的限制

#### clientName

默认值： `null`

客户端连接的名称。

### 集群 JSON 和 YAML 配置格式

以下是 JSON 格式的集群配置示例。
所有的属性名称都匹配 `ClusterServersConfig` 和 `Config` 对象的属性名称。

```json
{
   "clusterServersConfig":{
      "idleConnectionTimeout":10000,
      "pingTimeout":1000,
      "connectTimeout":1000,
      "timeout":1000,
      "retryAttempts":3,
      "retryInterval":1000,
      "reconnectionTimeout":3000,
      "failedAttempts":3,
      "password":null,
      "subscriptionsPerConnection":5,
      "clientName":null,
      "loadBalancer":{
         "class":"org.redisson.connection.balancer.RoundRobinLoadBalancer"
      },
      "slaveSubscriptionConnectionMinimumIdleSize":1,
      "slaveSubscriptionConnectionPoolSize":25,
      "slaveConnectionMinimumIdleSize":5,
      "slaveConnectionPoolSize":100,
      "masterConnectionMinimumIdleSize":5,
      "masterConnectionPoolSize":100,
      "readMode":"SLAVE",
      "nodeAddresses":[
         "//127.0.0.1:7004",
         "//127.0.0.1:7001",
         "//127.0.0.1:7000"
      ],
      "scanInterval":1000
   },
   "threads":0,
   "codec":null,
   "useLinuxNativeEpoll":false,
   "eventLoopGroup":null
}
```

以下是 YAML 格式的集群配置示例。
所有的属性名称都匹配 `ClusterServersConfig` 和 `Config` 对象的属性名称。

```yaml
---
clusterServersConfig:
  idleConnectionTimeout: 10000
  pingTimeout: 1000
  connectTimeout: 1000
  timeout: 1000
  retryAttempts: 3
  retryInterval: 1000
  reconnectionTimeout: 3000
  failedAttempts: 3
  password: null
  subscriptionsPerConnection: 5
  clientName: null
  loadBalancer: !<org.redisson.connection.balancer.RoundRobinLoadBalancer> {}
  slaveSubscriptionConnectionMinimumIdleSize: 1
  slaveSubscriptionConnectionPoolSize: 25
  slaveConnectionMinimumIdleSize: 5
  slaveConnectionPoolSize: 100
  masterConnectionMinimumIdleSize: 5
  masterConnectionPoolSize: 100
  readMode: "SLAVE"
  nodeAddresses:
  - "//127.0.0.1:7004"
  - "//127.0.0.1:7001"
  - "//127.0.0.1:7000"
  scanInterval: 1000
threads: 0
codec: !<org.redisson.codec.JsonJacksonCodec> {}
useLinuxNativeEpoll: false
eventLoopGroup: null
```

## Elasticache 模式

编程式配置示例：

```java
Config config = new Config();
config.useElasticacheServers()
    .setScanInterval(2000) // 主节点变更扫描间隔
    .addNodeAddress("127.0.0.1:7000", "127.0.0.1:7001")
    .addNodeAddress("127.0.0.1:7002");

RedissonClient redisson = Redisson.create(config);
```

### Elasticache 设置

有关 AWS Elasticache Redis 服务器配置的文档在
[这里](http://docs.aws.amazon.com/AmazonElastiCache/latest/UserGuide/GettingStarted.html)。

Elasticache 连接模式可由以下代码激活：

```java
ElasticacheServersConfig clusterConfig = config.useElasticacheServers();
```

`ElasticacheServersConfig` 设置项如下：

#### addNodeAddress

以 `host:port` 格式添加 Redis 集群节点地址。可以一次添加多个节点。
所有的节点(主节点和从节点)必须在配置时提供。

#### scanInterval

默认值： `1000`

以毫秒为单位的 Elasticache 节点扫描间隔。

#### loadBalancer

默认值： `org.redisson.connection.balancer.RoundRobinLoadBalancer`

其它实现： `org.redisson.connection.balancer.WeightedRoundRobinBalancer`,
`org.redisson.connection.balancer.RandomLoadBalancer`

多个 Redis 从服务器间的连接负载均衡器

#### slaveSubscriptionConnectionMinimumIdleSize

默认值： `1`

对 **每个** 从节点的 Redis '从'节点最小空闲订阅 (pub/sub) 连接量。

#### slaveSubscriptionConnectionPoolSize

默认值： `25`

对 **每个** 从节点的 Redis '从'节点最大订阅 (pub/sub) 连接池大小。

#### slaveConnectionMinimumIdleSize

默认值： `1`

对 **每个** 从节点的 Redis '从'节点最小空闲连接量。

#### slaveConnectionPoolSize

默认值： `100`

对 **每个** 从节点的 Redis '从'节点最大连接池大小。

#### masterConnectionMinimumIdleSize

默认值： `5`

对 **每个** 从节点的 Redis '主'节点最小空闲连接量。

#### masterConnectionPoolSize

默认值： `100`

Redis '主'节点最大连接池大小。

#### idleConnectionTimeout

默认值： `10000`

若池化连接在某段 `timeout` 时间内没有被使用且当前连接量超过最小空闲连接池时，
它将会被关闭并从池中移除。其值的单位是毫秒。

#### connectTimeout

默认值： `1000`

连接到任何 Redis 服务器的超时时间。

#### timeout

默认值： `1000`

Redis 服务器响应的超时时间。从 Redis 命令被成功发送时开始计算。其值的单位是毫秒。

#### retryAttempts

默认值： `3`

若 Redis 命令在超过 `retryAttempts` 次不能发送被 Redis 服务器，则将抛出一个错误。
但若成功发送，则将开始 `timeout`。

#### retryInterval

默认值： `1000`

发送 Redis 命令重试的时间间隔。其值的单位是毫秒。

#### reconnectionTimeout

默认值： `3000`

Redis 服务器重连尝试的超时时间。在每次这种超时事件发生时， Redisson 会尝试连接到失联的 Redis 服务器。
其值的单位是毫秒。

#### failedAttempts

默认值： `3`

当任何 Redis 命令的连续的未成功执行尝试到达 `failedAttempts` 时，
这个 Redis 服务器将被从一个内部的可用从节点列表中移除。

#### database

默认值： `0`

针对 Redis 连接的数据库索引。

#### password

默认值： `null`

Redis 服务器授权的密码。

#### subscriptionsPerConnection

默认值： `5`

每个 Redis 连接上的订阅的限制

#### clientName

默认值： `null`

客户端连接的名称。

### Elasticache JSON 和 YAML 配置格式

以下是 JSON 格式的 Elasticache 配置示例。
所有的属性名称都匹配 `ElasticacheServersConfig` 和 `Config` 对象的属性名称。

```json
{
   "elasticacheServersConfig":{
      "idleConnectionTimeout":10000,
      "pingTimeout":1000,
      "connectTimeout":1000,
      "timeout":1000,
      "retryAttempts":3,
      "retryInterval":1000,
      "reconnectionTimeout":3000,
      "failedAttempts":3,
      "password":null,
      "subscriptionsPerConnection":5,
      "clientName":null,
      "loadBalancer":{
         "class":"org.redisson.connection.balancer.RoundRobinLoadBalancer"
      },
      "slaveSubscriptionConnectionMinimumIdleSize":1,
      "slaveSubscriptionConnectionPoolSize":25,
      "slaveConnectionMinimumIdleSize":5,
      "slaveConnectionPoolSize":100,
      "masterConnectionMinimumIdleSize":5,
      "masterConnectionPoolSize":100,
      "readMode":"SLAVE",
      "nodeAddresses":[
         "//127.0.0.1:2812",
         "//127.0.0.1:2815",
         "//127.0.0.1:2813"
      ],
      "scanInterval":1000,
      "database":0
   },
   "threads":0,
   "codec":null,
   "useLinuxNativeEpoll":false,
   "eventLoopGroup":null
}
```

以下是 YAML 格式的 Elasticache 配置示例。
所有的属性名称都匹配 `ElasticacheServersConfig` 和 `Config` 对象的属性名称。

```yaml
---
elasticacheServersConfig:
  idleConnectionTimeout: 10000
  pingTimeout: 1000
  connectTimeout: 1000
  timeout: 1000
  retryAttempts: 3
  retryInterval: 1000
  reconnectionTimeout: 3000
  failedAttempts: 3
  password: null
  subscriptionsPerConnection: 5
  clientName: null
  loadBalancer: !<org.redisson.connection.balancer.RoundRobinLoadBalancer> {}
  slaveSubscriptionConnectionMinimumIdleSize: 1
  slaveSubscriptionConnectionPoolSize: 25
  slaveConnectionMinimumIdleSize: 5
  slaveConnectionPoolSize: 100
  masterConnectionMinimumIdleSize: 5
  masterConnectionPoolSize: 100
  readMode: "SLAVE"
  nodeAddresses:
  - "//127.0.0.1:2812"
  - "//127.0.0.1:2815"
  - "//127.0.0.1:2813"
  scanInterval: 1000
  database: 0
threads: 0
codec: !<org.redisson.codec.JsonJacksonCodec> {}
useLinuxNativeEpoll: false
eventLoopGroup: null
```

## 单实例模式

编程式配置示例：

```java
// connects to 127.0.0.1:6379 by default
RedissonClient redisson = Redisson.create();

Config config = new Config();
config.useSingleServer().setAddress("myredisserver:6379");
RedissonClient redisson = Redisson.create(config);
```

### 单实例设置

有关 Redis 单服务器配置的文档在
[这里](http://redis.io/topics/config)。
单服务器连接模式可通过以下代码激活：

```java
SingleServerConfig clusterConfig = config.useSingleServer();
```

`SingleServerConfig` 设置项如下：

#### address

`host:port` 格式的 Redis 服务器地址。

#### subscriptionConnectionMinimumIdleSize

默认值： `1`

最小空闲Redis订阅(pub/sub)连接量

#### subscriptionConnectionPoolSize

默认值： `25`

最大Redis订阅 (pub/sub) 连接池大小。

#### connectionMinimumIdleSize

默认值： `5`

最小Redis空闲连接量。

#### connectionPoolSize

默认值： `100`

最大Redis连接池大小。

#### dnsMonitoring

默认值： `false`

若为 `true`，服务器地址将监控 DNS 中的变更

#### dnsMonitoringInterval

默认值： `5000`

DNS 变更监控间隔

#### idleConnectionTimeout

默认值： `10000`

若池化连接在某段 `timeout` 时间内没有被使用且当前连接量超过最小空闲连接池时，
它将会被关闭并从池中移除。其值的单位是毫秒。

#### connectTimeout

默认值： `1000`

连接到任何 Redis 服务器的超时时间。

#### timeout

默认值： `1000`

Redis 服务器响应的超时时间。从 Redis 命令被成功发送时开始计算。其值的单位是毫秒。

#### retryAttempts

默认值： `3`

若 Redis 命令在超过 `retryAttempts` 次不能发送被 Redis 服务器，则将抛出一个错误。
但若成功发送，则将开始 `timeout`。

#### retryInterval

默认值： `1000`

发送 Redis 命令重试的时间间隔。其值的单位是毫秒。

#### reconnectionTimeout

默认值： `3000`

Redis 服务器重连尝试的超时时间。在每次这种超时事件发生时， Redisson 会尝试连接到失联的 Redis 服务器。
其值的单位是毫秒。

#### failedAttempts

默认值： `3`

当任何 Redis 命令的连续的未成功执行尝试到达 `failedAttempts` 时，
这个 Redis 服务器将被从一个内部的可用从节点列表中移除。

#### database

默认值： `0`

针对 Redis 连接的数据库索引。

#### password

默认值： `null`

Redis 服务器授权的密码。

#### subscriptionsPerConnection

默认值： `5`

每个 Redis 连接上的订阅的限制

#### clientName

默认值： `null`

客户端连接的名称。

### 单实例 JSON 和 YAML 配置格式

以下是 JSON 格式的单实例配置示例。
所有属性名称都匹配 `SingleServerConfig` 和 `Config` 对象的属性名称。

```json
{
   "singleServerConfig":{
      "idleConnectionTimeout":10000,
      "pingTimeout":1000,
      "connectTimeout":1000,
      "timeout":1000,
      "retryAttempts":3,
      "retryInterval":1000,
      "reconnectionTimeout":3000,
      "failedAttempts":3,
      "password":null,
      "subscriptionsPerConnection":5,
      "clientName":null,
      "address":[
         "//127.0.0.1:6379"
      ],
      "subscriptionConnectionMinimumIdleSize":1,
      "subscriptionConnectionPoolSize":25,
      "connectionMinimumIdleSize":5,
      "connectionPoolSize":100,
      "database":0,
      "dnsMonitoring":false,
      "dnsMonitoringInterval":5000
   },
   "threads":0,
   "codec":null,
   "useLinuxNativeEpoll":false,
   "eventLoopGroup":null
}
```

以下是 YAML 格式的单实例配置示例。
所有属性名称都匹配 `SingleServerConfig` 和 `Config` 对象的属性名称。

```yaml
---
singleServerConfig:
  idleConnectionTimeout: 10000
  pingTimeout: 1000
  connectTimeout: 1000
  timeout: 1000
  retryAttempts: 3
  retryInterval: 1000
  reconnectionTimeout: 3000
  failedAttempts: 3
  password: null
  subscriptionsPerConnection: 5
  clientName: null
  address:
  - "//127.0.0.1:6379"
  subscriptionConnectionMinimumIdleSize: 1
  subscriptionConnectionPoolSize: 25
  connectionMinimumIdleSize: 5
  connectionPoolSize: 100
  database: 0
  dnsMonitoring: false
  dnsMonitoringInterval: 5000
threads: 0
codec: !<org.redisson.codec.JsonJacksonCodec> {}
useLinuxNativeEpoll: false
eventLoopGroup: null
```

## Sentinel 模式

编程式配置示例：

```java
Config config = new Config();
config.useSentinelServers()
    .setMasterName("mymaster")
    .addSentinelAddress("127.0.0.1:26389", "127.0.0.1:26379")
    .addSentinelAddress("127.0.0.1:26319");

RedissonClient redisson = Redisson.create(config);
```

### Sentinel 配置

有关 Redis 服务器 sentinel 配置的文档在
[这里](http://redis.io/topics/sentinel)。
集群连接模式可由以下代码激活：

```java
SentinelServersConfig clusterConfig = config.useSentinelServers();
```

`SentinelServersConfig` 配置项如下：

#### masterName

由 Redis Sentinel 服务器和主节点变更监控任务所使用的主服务器名称。

#### addSentinelAddress

添加 `host:port` 格式的 Redis Sentinel 节点地址。可一次性添加多个节点。

#### loadBalancer

默认值： `org.redisson.connection.balancer.RoundRobinLoadBalancer`

其它实现有： `org.redisson.connection.balancer.WeightedRoundRobinBalancer`, 
`redisson.connection.balancer.RandomLoadBalancer`

多个 Redis 从服务器间的连接负载均衡器。

#### slaveSubscriptionConnectionMinimumIdleSize

默认值： `1`

对 **每个** 从节点的 Redis '从'节点最小空闲订阅 (pub/sub) 连接量。

#### slaveSubscriptionConnectionPoolSize

默认值： `25`

对 **每个** 从节点的 Redis '从'节点最大订阅 (pub/sub) 连接池大小。

#### slaveConnectionMinimumIdleSize

默认值： `1`

对 **每个** 从节点的 Redis '从'节点最小空闲连接量。

#### slaveConnectionPoolSize

默认值： `100`

对 **每个** 从节点的 Redis '从'节点最大连接池大小。

#### masterConnectionMinimumIdleSize

默认值： `5`

对 **每个** 从节点的 Redis '主'节点最小空闲连接量。

#### masterConnectionPoolSize

默认值： `100`

Redis '主'节点最大连接池大小。

#### idleConnectionTimeout

默认值： `10000`

若池化连接在某段 `timeout` 时间内没有被使用且当前连接量超过最小空闲连接池时，
它将会被关闭并从池中移除。其值的单位是毫秒。

#### connectTimeout

默认值： `1000`

连接到任何 Redis 服务器的超时时间。

#### timeout

默认值： `1000`

Redis 服务器响应的超时时间。从 Redis 命令被成功发送时开始计算。其值的单位是毫秒。

#### retryAttempts

默认值： `3`

若 Redis 命令在超过 `retryAttempts` 次不能发送被 Redis 服务器，则将抛出一个错误。
但若成功发送，则将开始 `timeout`。

#### retryInterval

默认值： `1000`

发送 Redis 命令重试的时间间隔。其值的单位是毫秒。

#### reconnectionTimeout

默认值： `3000`

Redis 服务器重连尝试的超时时间。在每次这种超时事件发生时， Redisson 会尝试连接到失联的 Redis 服务器。
其值的单位是毫秒。

#### failedAttempts

默认值： `3`

当任何 Redis 命令的连续的未成功执行尝试到达 `failedAttempts` 时，
这个 Redis 服务器将被从一个内部的可用从节点列表中移除。

#### database

默认值： `0`

针对 Redis 连接的数据库索引。

#### password

默认值： `null`

Redis 服务器授权的密码。

#### subscriptionsPerConnection

默认值： `5`

每个 Redis 连接上的订阅的限制

#### clientName

默认值： `null`

客户端连接的名称。

### Sentinel JSON 和 YAML 配置格式

以下是 JSON 格式的 Sentinel 配置示例。
所有属性名称都匹配 `SentinelServerConfig` 和 `Config` 对象的属性名称。

```json
{
   "sentinelServersConfig":{
      "idleConnectionTimeout":10000,
      "pingTimeout":1000,
      "connectTimeout":1000,
      "timeout":1000,
      "retryAttempts":3,
      "retryInterval":1000,
      "reconnectionTimeout":3000,
      "failedAttempts":3,
      "password":null,
      "subscriptionsPerConnection":5,
      "clientName":null,
      "loadBalancer":{
         "class":"org.redisson.connection.balancer.RoundRobinLoadBalancer"
      },
      "slaveSubscriptionConnectionMinimumIdleSize":1,
      "slaveSubscriptionConnectionPoolSize":25,
      "slaveConnectionMinimumIdleSize":5,
      "slaveConnectionPoolSize":100,
      "masterConnectionMinimumIdleSize":5,
      "masterConnectionPoolSize":100,
      "readMode":"SLAVE",
      "sentinelAddresses":[
         "//127.0.0.1:26379",
         "//127.0.0.1:26389"
      ],
      "masterName":"mymaster",
      "database":0
   },
   "threads":0,
   "codec":null,
   "useLinuxNativeEpoll":false,
   "eventLoopGroup":null
}
```

以下是 YAML 格式的 Sentinel 配置示例。
所有属性名称都匹配 `SentinelServerConfig` 和 `Config` 对象的属性名称。

```yaml
---
sentinelServersConfig:
  idleConnectionTimeout: 10000
  pingTimeout: 1000
  connectTimeout: 1000
  timeout: 1000
  retryAttempts: 3
  retryInterval: 1000
  reconnectionTimeout: 3000
  failedAttempts: 3
  password: null
  subscriptionsPerConnection: 5
  clientName: null
  loadBalancer: !<org.redisson.connection.balancer.RoundRobinLoadBalancer> {}
  slaveSubscriptionConnectionMinimumIdleSize: 1
  slaveSubscriptionConnectionPoolSize: 25
  slaveConnectionMinimumIdleSize: 5
  slaveConnectionPoolSize: 100
  masterConnectionMinimumIdleSize: 5
  masterConnectionPoolSize: 100
  readMode: "SLAVE"
  sentinelAddresses:
  - "//127.0.0.1:26379"
  - "//127.0.0.1:26389"
  masterName: "mymaster"
  database: 0
threads: 0
codec: !<org.redisson.codec.JsonJacksonCodec> {}
useLinuxNativeEpoll: false
eventLoopGroup: null
```


























