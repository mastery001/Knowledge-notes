[TOC]

# 1. 副本集节点概念

## 1.1 主节点(Primary)
>主节点接收所有写操作

## 1.2 从节点(Secondaries)
>从节点复制主节点的数据以达到一致

## 1.10 Arbiter（仲裁者）
>Arbiter是mongod的实例，虽然Arbiter不存储数据，但是它实际上与其他mongod的实例一样，都会写数据文件和记录journal日志。

**备注：journal：用于在硬关机的情况下使数据库进入有效状态的顺序二进制事务日志。**

使用注意：

1. 如果副本集只存在偶数个成员，则最好添加
2. 官方文档指出：不要将Arbiter部署在副本集的节点机器上

### 1.10.1 添加Arbiter

1. 给Arbiter创建数据目录

```shell
mkdir /data/arb
```

2. 启动

```shell
mongod --port 19000 --dbpath /data/arb --replSet rsName
```

3. 连接到副本集主节点添加Arbiter

```shell
rs.addArb("host:19000")
```