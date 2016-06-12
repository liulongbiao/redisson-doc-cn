# 数据序列化

数据序列化在 Redisson 中广泛地用于解编排在 Redis 服务器连接的网络上接收和发送的字节。
默认有多种流行的解编码器可用：

| Codec 类名                                                                             |  描述                                                    |
|-----------------------------------------|--------------------------|
| `org.redisson.codec.JsonJacksonCodec`   | [Jackson JSON](https://github.com/FasterXML/jackson) codec. **默认 codec** |
| `org.redisson.codec.CborJacksonCodec`   | [CBOR](http://cbor.io/) 二进制 json codec |
| `org.redisson.codec.MsgPackJacksonCodec`| [MsgPack](http://msgpack.org/)  二进制 json codec |
| `org.redisson.codec.KryoCodec`          | [Kryo](https://github.com/EsotericSoftware/kryo) 二进制 codec |
| `org.redisson.codec.SerializationCodec` | JDK 序列化 codec |
| `org.redisson.codec.FstCodec`           | [FST](https://github.com/RuedigerMoeller/fast-serialization) 10 倍速且 100% JDK 序列化兼容的 codec |
| `org.redisson.codec.LZ4Codec`           | [LZ4](https://github.com/jpountz/lz4-java) 压缩 codec |
| `org.redisson.codec.SnappyCodec`        | [Snappy](https://github.com/xerial/snappy-java) 压缩 codec |
| `org.redisson.client.codec.StringCodec` | 简单 String codec |
| `org.redisson.client.codec.LongCodec`   | 简单 Long codec |
