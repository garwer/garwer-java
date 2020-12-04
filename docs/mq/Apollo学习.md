> apollo是携程研发的分布式配置中心，能够集中管理应用不同环境

配置修改后能够实时推送到应用端，并具备用规范的权限，流程治理等特性，适用于微服务配置管理场景

可以通过命名空使不同应用共享一份配置，允许对共享的配置覆盖



优点

```
功能强大，可统一管理不同环境集群命名空间配置
配置修改实时生效，客户端能1s接收到最新的配置，通知应用程序
版本发布管理，所有配置存在版本概念，方便支持配置回滚
支持灰度发布，只对部分实例生效，等观察没问题再推给所有实例
部署简单，对外依赖少，外部依赖仅mysql。
```



### 为什么使用eureka做配置中心

而不是zk

提供完整的服务注册和服务发现

eureka支持自身容器启用，即充当了eureka角色也充当了服务提供者，提高服务可用性

开源



客户端会定时拉配置这是一个fallback机制防止推送机制失效导致配置不更新

客户端和服务端保持一个场链接从而可以第一时间获得服务更新的推送

客户端定时每5分钟去拉，可以指定apollo.refreshInterval



### 四个模块和主要功能

```
1.configService
提供配置获取接口、配置推送接口、服务于客户端
2.adminservice
服务于管理界面，配置管理接口。配置修改接口
3.client
为应用获取配置并支持实时更新
4.Portal
配置管理界面，通过metaserver获取adminservice服务列表
```

三个辅助模块

nginxlb和域名系统结合协助metaserver获取adminservice和configservice地址列表

eureka服务发现和注册和configservice一起部署、config和adminservice注册并实时报心跳

metaserver:和configservice一起部署相当于一个代理

- Portal通过域名访问MetaServer获取AdminService的地址列表

- Client通过域名访问MetaServer获取ConfigService的地址列表

  ​

  ​