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

