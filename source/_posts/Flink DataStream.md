---
title: Flink DataStream
---


## 流的构造与转换
### union
`public final DataStream<T> union(DataStream<T>... streams)`
> transformed and merge

### spilt
`public SplitStream<T> split(OutputSelector<T> outputSelector)`
> 基于outputSelector输出被命名的SplitStream

### connect
`public <R> ConnectedStreams<T, R> connect(DataStream<R> dataStream)`
`public <R> BroadcastConnectedStream<T, R> connect(BroadcastStream<R> broadcastStream)`

>在 DataStream 上有一个 union 的转换 dataStream.union(otherStream1, otherStream2, ...)，用来合并多个流，新的流会包含所有流中的数据。union 有一个限制，就是所有合并的流的类型必须是一致的。ConnectedStreams 提供了和 union 类似的功能，用来连接两个流，但是与 union 转换有以下几个区别：
    1. ConnectedStreams 只能连接两个流，而 union 可以连接多于两个流；
    2. ConnectedStreams 连接的两个流类型可以不一致，而 union 连接的流的类型必须一致；
    3. ConnectedStreams 会对两个流的数据应用不同的处理方法，并且双流之间可以共享状态。这在第一个流的输入会影响第二个流时, 会非常有用，比如计数。

### keyBy
`public <K> KeyedStream<T, K> keyBy(KeySelector<T, K> key)`
`public <K> KeyedStream<T, K> keyBy(KeySelector<T, K> key, TypeInformation<K> keyType)`
`public KeyedStream<T, Tuple> keyBy(int... fields)`
`public KeyedStream<T, Tuple> keyBy(String... fields)`
`private KeyedStream<T, Tuple> keyBy(Keys<T> keys)`

> 基于KeySelector生成KeyedStream，可以对KeyedStream进行window、aggregate、statistics（min、max、sum）等操作。

## 流的分区
### partitionCustom
> 自己提供分区函数Partitioner\<K\>

### broadcast
> 每个分区都有一份全量的数据

### shuffle
> 使用`random.nextInt`实现随机

### forward
> 默认返回0
> 文档中描述为：the output elements are forwarded to the local subtask of the next operation.

### rebalance
> 将element轮询分配到所有分区，第一次的分区数随机生成。

### rescale
> 将element轮询分配到所有分区，第一次的分区数为0。

### global
> 同forward，默认返回0。
> 文档中描述为：Sets the partitioning of the {@link DataStream} so that the output values all go to the first instance of the next processing operator. Use this setting with care since it might cause a serious performance bottleneck in the application.



