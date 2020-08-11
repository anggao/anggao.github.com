# Service Mesh01


## 微服务架构的特性

- 围绕业务构建团队
  - [Conway's Law](https://en.wikipedia.org/wiki/Conway%27s_law)
  - > Any organization that designs a system (defined broadly) will produce a design whose structure is a copy of the organization's communication structure.
- 去中心化的数据管理

## [Fallacies of distributed computing](https://en.wikipedia.org/wiki/Fallacies_of_distributed_computing)

- The network is reliable;
- Latency is zero;
- Bandwidth is infinite;
- The network is secure;
- Topology doesn't change;
- There is one administrator;
- Transport cost is zero;
- The network is homogeneous.

## 如何管理和控制服务间的通信

- 服务注册与发现
- 路由，流量转移
- 弹性能力 （问题发生时熔断，超时，重试)
- 安全 (授权认证)
- 可观察性 (系统状态，资源状态等)

## Service Mesh 的演进

- 控制逻辑和业务逻辑耦合
- 公共库
  - 好处：解耦，消除重复
  - 问题： 成本， 语言绑定, 仍有侵入
- 代理
  - 问题：功能简陋
- Sidecar 模式
- Service Mesh

## [Service Mesh](https://buoyant.io/2017/04/25/whats-a-service-mesh-and-why-do-i-need-one/)

> A service mesh is a dedicated **infrastructure layer** for handling service-to-service communication. It’s responsible for the reliable **delivery of requests** through the complex topology of services that comprise a modern, cloud native application. In practice, the service mesh is typically implemented as an array of lightweight **network proxies** that are deployed alongside application code, **without the application needing to be aware**.

- Service Mesh 是 Sidecar 的网络拓扑模式

## Service Mesh 的主要功能

- 流量控制
  - 路由
    - 蓝绿部署
    - 灰度发布
    - A/B 测试
  - 流量转移
  - 超时重试
  - 熔断
  - 故障注入
  - 流量镜像
- 策略
  - 流量限制
  - 黑白名单
- 网络安全
  - 授权及身份认证
- 可观察性
  - 指标收集和展示
  - 日志收集
  - 分布式追踪

## Service Mesh 和 API Gateway 的区别

- 功能有重叠但角色不同
- Service Mesh 在应用内, API 网关在应用之上(边界)

## Service Mesh 技术标准

- UDPA (Universal Data Plane API)
- SMI (Service Mesh Interface)

