---
layout:     post
title:   【译】使用Weave Scope探索微服务通信和服务网格（OpenShift和Istio）
subtitle:   Using Weave Scope to explore Microservices Communication and Service Mesh (OpenShift and Istio)
date:       2018-01-15 21:05:00
author:     "Kaelzhang"
header-img: "img/post-concept-bg.jpg"
catalog:    true
tags:
    - istio
    - service mesh
---

# 【译】使用Weave Scope探索微服务通信和服务网格（OpenShift和Istio）

本文是我给service mesh社区翻译的一篇文章。
原文地址：[https://holisticsecurity.io/2018/01/12/using-weave-scope-to-explore-microservices-communication-and-service-mesh-openshift-and-istio/?utm_source=tuicool&utm_medium=referral](https://holisticsecurity.io/2018/01/12/using-weave-scope-to-explore-microservices-communication-and-service-mesh-openshift-and-istio/?utm_source=tuicool&utm_medium=referral)

如果你正从事ESB（企业服务总线）、消息代理、BPMS（业务流程管理套件）、SOA（面向服务架构）或微服务的工作，你会注意到你正在使用不同的方式解决应用同样的一些问题，因为这些应用都是不同类型的分布式应用。这些问题包括：

* 用户管理、认证和授权
* 日志、调试、监控及告警
* 集群、高可用、负载均衡
* ...

## 什么是服务网格？
服务网格是另一种分布式应用，其中服务/微服务/API一直相互连接。

![](/img/in-post/Weave Scope/api-mesh-security-3-istio-bookinfo-app.png)

## 服务网格在容器化和编排平台
通常使用Kubernetes托管和编排的多个容器来部署基于微服务和（或）API的服务网格。 在这种情况下，我们需要面临新的挑战如：脆弱的基础设施，不可信网络，网络分段或采用新的方法来测试，监控，部署，运维等。我们称之为DevOps。

## 容器部署模式
与EIP（企业集成模式）和软件设计模式一样，在容器平台中部署服务网格也具有模式。谷歌、微软、Netflix等推荐使用一些模式来解决上述问题。

例如，Google很好地阐述了三种模式：

* 边车模式
* 大使模式
* 适配器模式

他们都支持基于容器的服务网格的构建。 欲了解更多详情，请阅读
[基于容器的分布式系统设计模式](https://static.googleusercontent.com/media/research.google.com/en//pubs/archive/45406.pdf)。

## 什么是Istio？
以下摘自[Istio.io](https://istio.io/about/intro.html)：

Istio是一个用来连接、管理和保护微服务的开放平台。Istio支持管理微服务之间的流量，执行访问策略以及汇总遥测数据，而无需更改微服务代码。Istio提供：

* HTTP，gRPC，WebSocket及TCP流量的自动负载平衡
* 通过富路由规则、重试、故障转移及注入，对流量行为进行细粒度控制。
* 可拔插的策略层和配置API，支持访问控制、限流和配额的。
* 自动的指标、日志及集群内所有流量的跟踪，包括集群出入口。
* 通过群集内服务间强大的身份断言来保证服务到服务的身份验证。

![Istio架构](/img/in-post/Weave Scope/istio-architecture.png)

## 为什么使用Istio？

以下摘自[Istio.io](https://istio.io/docs/concepts/what-is-istio/overview.html)：

在从单体应用程序向分布式微服务架构的转型过程中，开发人员和运维人员面临诸多挑战， 使用Istio可以解决这些问题。术语服务网格（Service Mesh）通常用于描述构成这些应用程 序的微服务网络以及它们之间的交互。随着规模和复杂性的增长，服务网格越来越难以理解 和管理。它的需求包括服务发现、负载均衡、故障恢复、指标收集和监控以及通常更加复杂 的运维需求，例如A/B测试、金丝雀发布、限流、访问控制和端到端认证等。

Istio提供了一个完整的解决方案，通过为整个服务网格提供行为洞察和操作控制来满足微服务 应用程序的多样化需求。它在服务网络中统一提供了许多关键功能：

流量管理。控制服务之间的流量和API调用的流向，使得调用更可靠，并使网络在恶劣情 况下更加健壮。

可观察性。了解服务之间的依赖关系，以及它们之间流量的本质和流向，从而提供快速 识别问题的能力。

策略执行。将组织策略应用于服务之间的互动，确保访问策略得以执行，资源在消费者 之间良好分配。策略的更改是通过配置网格而不是修改应用程序代码。

服务身份和安全。为网格中的服务提供可验证身份，并提供保护服务流量的能力，使其可以在不同可信度的网络上流转。

## 获取基于容器的服务网格Istio
我已经创建了3个Ansible角色，通过使用Minishift，Weave Scope（https://github.com/weaveworks/scope），Istio和BookInfo应用程序（容器上的API和微服务）轻松获得最小化的OpenShift集群，以了解 我们必须面对的挑战。 3个Ansible的角色是：

1. [Minishift Ansible角色](https://github.com/chilcano/ansible-role-minishift)
使用[Minishift](https://github.com/minishift/minishift)在虚拟机中获得OpenShift集群
2. [Weave Scope Ansible角色](https://github.com/chilcano/ansible-role-weave-scope)
在本地运行的OpenShift中部署Weave Scope应用
3. Istio Ansible角色
在本地运行的OpenShift中，部署和配置Istio，部署BookInfo应用并注入边车代理（Envoy代理）

[我还创建了一些示例，以便你能拥有一个可测试环境，并能快速尝试所有这些内容。](https://github.com/chilcano/ansible-minishift-istio-security)

Weave Scope应用在这里扮演一个重要角色，它使我们能够轻松监控，可视，故障排除和管理整个基于容器的服务网格。

一旦完成[分步指南](https://github.com/chilcano/ansible-minishift-istio-security)，你将看到：

![](/img/in-post/Weave Scope/api-mesh-security-7-weave-scope-istio-system.png)

![](/img/in-post/Weave Scope/api-mesh-security-8-weave-scope-bookinfo.png)

![](/img/in-post/Weave Scope/api-mesh-security-9-weave-scope-bookinfo-mesh.png)

![](/img/in-post/Weave Scope/api-mesh-security-1-openshift.png)

![](/img/in-post/Weave Scope/api-mesh-security-3-istio-bookinfo-app.png)

![](/img/in-post/Weave Scope/api-mesh-security-4-istio-zipkin.png)

![](/img/in-post/Weave Scope/api-mesh-security-5-istio-grafana.png)

![](/img/in-post/Weave Scope/api-mesh-security-6-istio-servicegraph.png)

希望本文对你有帮助。


