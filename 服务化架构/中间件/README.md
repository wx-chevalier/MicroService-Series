# 中间件

介于操作系统和应用程序之间的产品，中间件简单解释，你可以理解为面向信息系统交互，集成过程中的通用部分的集合，屏蔽了底层的通讯，交互，连接等复杂又通用化的功能，以产品的形式提供出来，系统在交互时，直接采用中间件进行连接和交互即可，避免了大量的代码开发和人工成本。其实，理论上来讲，中间件所提供的功能通过代码编写都可以实现，只不过开发的周期和需要考虑的问题太多，逐渐的，这些部分，以中间件产品的形式进行了替代。比如常见的消息中间件，即系统之间的通讯与交互的专用通道，类似于邮局，系统只需要把传输的消息交给中间件，由中间件负责传递，并保证传输过程中的各类问题，如网络问题，协议问题，两端的开发接口问题等均由消息中间件屏蔽了，出现了网络故障时，消息中间件会负责缓存消息，以避免信息丢失。中间件就是 非业务的技术类组件。

其实从广义来说 操作系统上，业务系统下与业务无关的，都是中间件，包括数据库，离线等。当然 实际上不会这么分。不过利用这个讲法应该能够更容易的去理解中间件是什么。

阿里的中间件主要就包含这么几个: [分布式关系型数据库 DRDS\_ 水平拆分 \*\*](https://www.aliyun.com/product/drds) 做数据库扩展性的 [消息队列 \_ 云消息 \*\*](https://www.aliyun.com/product/ons/) 做消息的 MOM [企业级分布式应用服务 EDAS\_ 企业云计算解决方案 \*\*](https://www.aliyun.com/product/edas) 做分布式服务的还有一些其他的中间件，比如 JstormT, 配置服务 缓存 等等，也都会放在中间件里

# 架构风格

完整的软件体系结构由不同的模式或者风格构成，常见的风格有架构设计模式、框架设计模式与驱动模式等。

Architectural styles tell us, in very broad strokes, how to organise our code. It’s the highest level of granularity and it specifies layers, high-level modules of the application and how those modules and layers interact with each other, the relations between them. Examples of Architectural Styles:

Component-based
Monolithic application
Layered
Pipes and filters
Event-driven
Publish-subscribe
Plug-ins
Client-server
Service-oriented
An Architectural Style can be implemented in various ways, with a specific technical environment, specific policies, frameworks or practices.

A pattern is a recurring solution to a recurring problem. In the case of Architectural Patterns, they solve the problems related to the Architectural Style. For example, “what classes will we have and how will they interact, in order to implement a system with a specific set of layers“, or “what high-level modules will have in our Service-Oriented Architecture and how will they communicate“, or “how many tiers will our Client-server Architecture have“.

Architectural Patterns have an extensive impact on the code base, most often impacting the whole application either horizontally (ie. how to structure the code inside a layer) or vertically (ie. how a request is processed from the outer layers into the inner layers and back). Examples of Architectural Patterns:

Three-tier
Microkernel
Model-View-Controller
Model-View-ViewModel

# 微服务架构

## 服务网关

### 负载均衡

负载均衡是我们对服务如何消化流量的通用设计，通常分为物理层的底层协议分流的硬负载均衡和软件层的软负载。负载均衡解决方案已经是业界成熟的方案，我们通常会针对特定业务在不同环境进行优化，常用有如下的负载均衡解决方案

交换机
F5
LVS/ALI-LVS
Nginx/Tengine
VIPServer/ConfigServer

负载均衡首当其冲的就是网关，因为中心化集群流量最先打到的地方就是网关了，如果网关扛不住压力的话，那么整个系统将不可用。

高性能
网关设计第一需要考虑的是高性能的流量转发，网关单节点通常能达到上百万的并发流量
分布式
出于流量压力分担和灾备考虑，网关设计同样需要分布式
业务筛选
网关同设计简单的规则，排除掉大部分的恶意流量

### 流量管理与控制

请求校验
请求鉴权可以把多少非法请求拦截，清洗
数据缓存
多数无状态的请求存在数据热点，所以采用 CDN 可以把相当大一部分的流量消费掉

剩下的真实流量我们采用不同的算法来分流请求

流量分配
计数器
队列
漏斗
令牌桶
动态流控
流量限制

在流量激增的时候，通常我们需要有限流措施来防止系统出现雪崩，那么就需要预估系统的流量上限，然后设定好上限数，但流量增加到一定阈值后，多出来的流量则不会进入系统，通过牺牲部分流量来保全系统的可用性。

QPS 粒度
线程数粒度
RT 阈值
限流策略
限流工具 - Sentinel

## 服务发现

服务编排的定义是：通过消息的交互序列来控制各个部分资源的交互。参与交互的资源都是对等的，没有集中的控制。微服务环境下服务众多我们需要有一个总的协调器来协议服务之间的依赖，调用关系

网关是流量的集散地，而注册中心则是服务的根据地。通过自动化脚本就可以启动应用服务，启动后服务则需要发现注册中心，并且把自身的服务信息注册到服务网关，也即是网关接入。注册中心则会监控服务的不同状态，做健康检查，把不可用的服务归类标记。

状态类型
第一好应用服务的状态，通过注册中心就可以检测服务是否可用
生命周期
应用服务不同的状态组成了应用的生命周期

集群版本
集群不用应用有自身对应的版本号，由不同服务组成的集群也需要定义大的版本号
版本回滚
在部署异常的时候可以根据大的集群版本进行回滚管理

## 服务容错

降级
当用户激增的时候，我们首先是在流量端做手脚，也就是限流。当我们发现限流后系统响应变慢了，有可能导致更多的问题时，我们也需要对服务本身做一些操作。服务降级就是把当前不是很核心的功能关闭掉，或者不是很要紧的准确性放宽范围，事后再做一些人工补救。
降低一致性约束
关闭非核心服务
简化功能
熔断
当我们都做了以上的操作后，还是觉得不放心，那么就需要再进一步操心。熔断是对过载的一种自身保护，犹如我们开关跳闸一样。比如当我们服务不断对数据库进行查询的时候，如果业务问题造成查询问题，这是数据库本身需要熔断来保证不会被应用拖垮，并且访问友好的信息，告诉服务不要再盲目调用了。
闭合状态
半开状态
断开状态
熔断工具- Hystrix
幂等
我们知道，一个幂等操作的特点是其任意多次执行所产生的影响均与一次执行的影响相同。那么久需要对单次操作赋予一个全局的 id 来做标识，这样多次请求后我们可以判断来源于同个客户端，避免出现脏数据。
全局一致性 ID
Snowflake

既然我们知道分布式系统故障时家常便饭的事情，那么应对故障的方案也是不可或缺的环节。通常我们有主动和被动的方式来处理，主动是在错误出现的时候，我们试图再试试几次，说不定就成功了，成功的话就可以避免了该次错误。被动方式是错误的事情已经发生了，为了挽回，我们只是做时候处理，把负面影响降到最小。既然我们知道分布式系统故障时家常便饭的事情，那么应对故障的方案也是不可或缺的环节。通常我们有主动和被动的方式来处理，主动是在错误出现的时候，我们试图再试试几次，说不定就成功了，成功的话就可以避免了该次错误。被动方式是错误的事情已经发生了，为了挽回，我们只是做时候处理，把负面影响降到最小。

## 协调与配置管理

全局配置中心按环境来区分，统一管理，减少了多处配置的混乱局面

switch
diamend

# 性能与可用性

性能表示系统在给定时间间隔内执行任何操作的响应性，而可伸缩性是系统处理负载增加而不影响性能或容易增加可用资源的能力。分布式系统通常会遇到变化的负载和活动高峰，特别是在多租户场景中，几乎是不可能预测的。相反，应用应该能够在限制范围内扩展以满足需求高峰，并在需求减少时进行扩展。可伸缩性不仅涉及计算实例，还涉及其他元素，如数据存储，消息队列等。

## 可用性

可用性是系统运行和工作的时间比例，通常以正常运行时间的百分比来衡量。它可能受系统错误，基础架构问题，恶意攻击和系统负载的影响。分布式系统通常为用户提供服务级别协议（SLA），因此应用程序必须设计为最大化可用性。

健康检查：系统实现全链路功能检查，外部工具定期通过公开端点访问系统
负载均衡：使用队列起到削峰作用，作为请求和服务之间的缓冲区，以平滑间歇性的重负载
节流：限制应用级别、租户或整个服务所消耗资源的范围

## 安全

安全性是系统能够防止在设计使用之外的恶意或意外行为，并防止泄露或丢失信息。分布式系统在受信任的本地边界之外的 Internet 上运行，通常向公众开放，并且可以为不受信任的用户提供服务。必须以保护应用程序免受恶意攻击，限制仅允许对已批准用户的访问，并保护敏感数据。

联合身份：将身份验证委派给外部身份提供商
看门人：通过使用专用主机实例来保护应用程序和服务，该实例充当客户端与应用程序或服务之间的代理，验证和清理请求，并在它们之间传递请求和数据
代客钥匙：使用为客户端提供对特定资源或服务的受限直接访问的令牌或密钥。

## 弹性调度

弹性是指系统能够优雅地处理故障并从故障中恢复。分布式系统通常是多租户，使用共享平台服务，竞争资源和带宽，通过 Internet 进行通信，以及在商用硬件上运行，意味着出现瞬态和更永久性故障的可能性增加。为了保持弹性，必须快速有效地检测故障并进行恢复。

隔离：将应用程序的元素隔离到池中，以便在其中一个失败时，其他元素将继续运行。
断路器：处理连接到远程服务或资源时可能需要不同时间修复的故障。
补偿交易：撤消一系列步骤执行的工作，这些步骤共同定义最终一致的操作
健康检查：系统实现全链路功能检查，外部工具定期通过公开端点访问系统
重试：通过透明地重试先前失败的操作，使应用程序在尝试连接到服务或网络资源时处理预期的临时故障

过去软件系统随着用户量增加需要增加机器资源的话，传统的方式就是找运维申请机器，然后部署好软件服务接入集群，整个过程依赖的是运维人员的人肉经验，效率低下而且容易出错。微服务分布式则无需人肉增加物理机器，在容器化技术的支撑下，我们只需要申请云资源，然后执行容器脚本即可。

应用扩容
用户激增需要对服务进行扩展，包括自动化扩容，峰值过后的自动缩容
机器下线
对于过时应用，进行应用下线，云平台收回容器宿主资源
机器置换
对于故障机器，可供置换容器宿主资源，服务自动启动，无缝切换

# DevOps

## 部署

微服务分布式部署是家常便饭，如何让我们的服务更好的支撑业务发展，稳健的部署策略是我们首先需要考虑的，如下的部署策略适合不同业务和不同的阶段。

停机部署
滚动部署
蓝绿部署
灰度部署
A/B 测试

## 监控

分布式系统在远程数据中心中运行，无法完全控制基础结构，这使管理和监视比单机部署更困难。应用必须公开运行时信息，管理员可以使用这些信息来管理和监视系统，以及支持不断变化的业务需求和自定义，而无需停止或重新部署应用。

由于分布式系统是由众多机器共同协作的系统，而且网络也无法保证完全可用，所以我们需要建设一套对各个环节都能监控的系统，这样我们才能从底层到业务各个层面进行监控，出现意外的时候可以及时修复故障，避免更多的问题出现。

基础层

基础层面是对容器资源的监测，包含各个硬件指标的负载情况

CPU，IO，内存，线程，吞吐
中间件

分布式系统接入了大量的中间件平台，中间件本身的健康情况也需要监控

应用层

性能监控
应用层面的需要对每个应用服务的实时指标（qps，rt），上下游依赖等进行监控
业务监控
除了应用本身的监控程度，业务监控也是保证系统正常的一个环节，通过设计合理的业务规则，对异常的情况做报警设置
监控链路

zipkin/eagleeye
sls
goc
Alimonitor

## 故障恢复

当故障已经发生后，我们第一要做的是马上消除故障，确保系统服务正常可用，这个时候通常的做回滚操作。

应用回滚

应用回滚之前需要保存好故障现场，以便排查原因。

基线回退

应用服务回滚后，代码基线也需要 revert 到前一版本。

版本回滚

整体回滚需要服务编排，通过大版本号对集群进行回滚。

# Links

- [ ] [pragmatic-java-engineer](https://github.com/superhj1987/pragmatic-java-engineer/blob/master/book/chapter1-servertech/server-basic.md)
