# 2021-基于 Netty 的长连接网关

WebSocket 是目前实现服务端推送的主流技术，恰当使用能够有效提供系统响应能力，提升用户体验。通过 WebSocket 长连接网关可以快速为系统增加数据推送能力，有效减少运维成本，提高开发效率。长连接网关的价值在于它封装了 WebSocket 通信细节，与业务系统解耦，使得长连接网关与业务系统可独立优化迭代，避免重复开发，便于开发与维护。其次，网关提供了简单易用的 HTTP 推送通道，支持多种开发语言接入，便于系统集成和使用。另外，网关采用了分布式架构，可以实现服务的水平扩容、负载均衡与高可用。最后，网关集成了监控与报警，当系统异常时能及时预警，保证服务的健康和稳定。

在众多的 WebSocket 实现中，从性能、扩展性、社区支持等方面考虑，最终选择了 Netty。Netty 是一个高性能、事件驱动、异步非阻塞的网络通信框架，在许多知名的开源软件中被广泛使用。WebSocket 是有状态的，无法像直接 HTTP 以集群方式实现负载均衡，长连接建立后即与服务端某个节点保持着会话，因此集群下想要得知会话属于哪个节点，有两种方案，一种是使用类似微服务的注册中心来维护全局的会话映射关系，一种是使用事件广播由各节点自行判断是否持有会话，两种方案对比如下表所示。

| 方案     | 优点                                   | 缺点                                     |
| -------- | -------------------------------------- | ---------------------------------------- |
| 注册中心 | 会话映射关系清晰，集群规模较大时更合适 | 实现复杂，强依赖注册中心，有额外运维成本 |
| 事件广播 | 实现简单更加轻量                       | 节点较多时，所有节点均被广播，资源浪费   |

综合考虑实现成本与集群规模，选择了轻量级的事件广播方案。实现广播可以选择基于 RocketMQ 的消息广播、基于 Redis 的 Publish/Subscribe、基于 ZooKeeper 的通知等方案，其优缺点对比如下表所示。从吞吐量、实时性、持久化、实现难易等方面考虑，最终选择了 RocketMQ。

| 方案           | 优点                       | 缺点                             |
| -------------- | -------------------------- | -------------------------------- |
| 基于 RocketMQ  | 吞吐量高、高可用、保证可靠 | 实时性不如 Redis                 |
| 基于 Redis     | 实时性高、实现简单         | 不保证可靠                       |
| 基于 ZooKeeper | 实现简单                   | 写入性能较差，不适合频繁写入场景 |

## 系统架构

![Netty & WebSocket 架构](https://img.imgdb.cn/item/6051b557524f85ce299b8bce.jpg)

网关的整体流程如下：

- 客户端与网关任一节点握手建立起长连接，节点将其加入到内存维护的长连接队列。客户端定时向服务端发送心跳消息，如果超过设定的时间仍没有收到心跳，则认为客户端与服务端的长连接已断开，服务端会关闭连接，清理内存中的会话。
- 当业务系统需要向客户端推送数据时，通过网关提供的 HTTP 接口将数据发向网关。
- 网关在接收到推送请求后，将消息写入 RocketMQ。
- 网关作为消费者，以广播模式消费消息，所有节点都会接收到消息。
- 节点接收到消息后判断推送的消息目标是否在自己内存中维护的长连接队列里，如果存在则通过长连接推送数据，否则直接忽略。

网关以多节点方式构成集群，每节点负责一部分长连接，可实现负载均衡，当面对海量连接时，也可以通过增加节点的方式分担压力，实现水平扩展。同时，当节点出现宕机时，客户端会尝试重新与其他节点握手建立长连接，保证服务整体的可用性。

## 会话管理

长连接建立起来后，会话维护在各节点的内存中。SessionManager 组件负责管理会话，内部使用了哈希表维护了 UID 与 UserSession 的关系；UserSession 代表用户维度的会话，一个用户可能会同时建立多个长连接，因此 UserSession 内部同样使用了一个哈希表维护 Channel 与 ChannelSession 的关系。为了避免用户无限制的创建长连接，UserSession 在内部的 ChannelSession 超过一定数量后，会将最早建立的 ChannelSession 关闭，减少服务器资源占用。SessionManager、UserSession、ChannelSession 的关系如下图所示。

![SessionManager 组件](https://img.imgdb.cn/item/6051b608524f85ce299bf4a7.jpg)
