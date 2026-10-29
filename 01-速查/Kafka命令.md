# Kafka 命令速查（3.5.1）

> ⚠️ **3.5.1 没有 `--zookeeper` 参数了**，全部用 `--bootstrap-server`。
> 下面标 🟢 的只读，🟡 的会改状态。

## 设个变量，少打字

```bash
export KAFKA_HOME=/opt/kafka          # 改成你的实际路径
export BS=10.0.0.1:9092               # 改成你的 broker 地址
```

## 🟢 健康检查

```bash
$KAFKA_HOME/bin/kafka-broker-api-versions.sh --bootstrap-server $BS | head

$KAFKA_HOME/bin/kafka-topics.sh --bootstrap-server $BS --list

$KAFKA_HOME/bin/kafka-topics.sh --bootstrap-server $BS --describe --topic <topic名>

$KAFKA_HOME/bin/kafka-topics.sh --bootstrap-server $BS --under-replicated-partitions
```

> `--describe` 重点看三列：
> - **Leader**：-1 说明这个分区没有主，等于不可用
> - **Replicas / Isr**：Isr 少了副本 = 有 broker 掉了
> - **Under-replicated** 有输出 = 副本不同步，要查

## 🟢 消费组（排查积压必用）

```bash
$KAFKA_HOME/bin/kafka-consumer-groups.sh --bootstrap-server $BS --list

$KAFKA_HOME/bin/kafka-consumer-groups.sh --bootstrap-server $BS --describe --group <组名>
```

> **LAG = 积压量**。LAG 一直涨 = 消费跟不上（消费端慢了或者挂了）。
> LAG 不涨但也不降 = 消费端卡住了。

## 🟡 改状态（生产上先问研发）

```bash
# 修改 topic 分区数（只能增不能减）
$KAFKA_HOME/bin/kafka-topics.sh --bootstrap-server $BS --alter --topic <topic> --partitions 6

# 动态改配置（不用重启）
$KAFKA_HOME/bin/kafka-configs.sh --bootstrap-server $BS --alter --entity-type topics --entity-name <topic> --add-config retention.ms=604800000
```

## 🟡 重置消费位移（危险，先 dry-run）

```bash
$KAFKA_HOME/bin/kafka-consumer-groups.sh --bootstrap-server $BS --group <组名> \
  --reset-offsets --to-earliest --topic <topic> --dry-run

# 确认无误后，把 --dry-run 换成 --execute
```

> ⚠️ **必须停掉消费者再重置**，否则会乱。
> ⚠️ 生产上**不要随便用 `--from-beginning`** 读数据，IO 压力大。

## 常见故障对照

| 现象 | 先查 |
|---|---|
| 消费积压涨 | `--describe --group` 看 LAG；再查消费端进程 |
| 分区不可用 | `--describe --topic` 看 Leader 是不是 -1 |
| 副本不同步 | `--under-replicated-partitions` |
| 生产发不进去 | 看 broker 是否 `acks` 超时、磁盘满没满 |
