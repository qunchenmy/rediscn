---
layout: commands
title: cluster-delslots 命令
permalink: commands/cluster-delslots.html
disqusIdentifier: command_cluster-delslots
disqusUrl: http://redis.cn/commands/cluster-delslots.html
commandsType: cluster
discuzTid: 920
---

In Redis Cluster, each node keeps track of which master is serving
a particular hash slot.

在Redis Cluster中，每个节点都会记录主节点服务哪些特定的哈希槽（hash slot）。

The `DELSLOTS` command asks a particular Redis Cluster node to forget which master is serving the hash slots specified as arguments.

`DELSLOTS`命令请求特定的Redis Cluster节点忘记由参数指定的正在服务的哈希槽。当节点已经接收到DELSLOTS命令之后，并因此移除指命令传入的哈希槽的关联信息，我们称这些哈希槽是未绑定的。

In the context of a node that has received a `DELSLOTS` command and has consequently removed the associations for the passed hash slots, we say those hash slots are *unbound*. Note that the existence of unbound hash slots occurs naturally when a node has not been configured to handle them (something that can be done with the `ADDSLOTS` command) and if it has not received any information about who owns those hash slots (something that it can learn from heartbeat or update messages).

当节点已经接收到`DELSLOTS`命令，并因此移除指命令传入的哈希槽的关联信息，我们称这些哈希槽是未绑定的。注意的是这些未绑定的哈希槽是自然而然就存在的，当一个节点没有被配置去处理这些哈希槽（可以通过ADDSLOTS命令完成这些）并且该节点没有收到关于谁拥它们（节点从心跳包或者更新包获取这些消息）。


If a node with unbound hash slots receives a heartbeat packet from another node that claims to be the owner of some of those hash slots, the association is established instantly. Moreover, if a heartbeat or update message is received with a configuration epoch greater than the node's own, the association is re-established.

如果有未绑定哈希槽的节点从其它接受到节点心跳包，声称是这些为绑定节点的拥有者，节点和哈希槽的关联信息理解会被建立。但是，当从其它节点接收到带有配置纪元更大的心跳包或者更新消息时，节点和哈希槽的关联信息理解会被重建。

However, note that:

但是，请注意：

1. The command only works if all the specified slots are already
  associated with some node.

1、DELSLOTS命令只有在所有指定的哈希槽已经关联到一些节点上才有做作用。



2. The command fails if the same slot is specified multiple times.

2、如果同一个哈希槽被指定多次，该命令会失败。

3. As a side effect of the command execution, the node may go into
  *down* state because not all hash slots are covered.

3、命令执行的副作用是，因为不是所有哈希槽都覆盖到，节点可能会进入下线状态。

## Example

## 例如

The following command removes the association for slots 5000 and
5001 from the node receiving the command:

以下命令会从接受命令的节点移除槽5000和槽5001的关联信息。

    > CLUSTER DELSLOTS 5000 5001
    OK

## Usage in Redis Cluster

## Redis Cluster中的用法

This command only works in cluster mode and may be useful for debugging and in order to manually orchestrate a cluster configuration when a new cluster is created. It is currently not used by `redis-trib`, and mainly exists for API completeness.

命令只在集群模式下工作，并且对调试非常有用，并且为了创建新的集群时，可以手动的安排集群的哈希槽配置。当前没有被`redis-trib`工具使用，并且主要为了API的完整性存在。

@return

@返回值

@simple-string-reply: `OK` if the command was successful. Otherwise
an error is returned.

@[simple-string-reply](http://www.redis.cn/topics/protocol.html#simple-string-reply)：如果命令成功执行返回OK，否则返回一个错误。
