# ABSTRACT 摘要

The shared nature of the network in today’s multi-tenant datacenters implies that network performance for tenants can vary signiﬁcantly.

在当今的多租户数据中心中，网络的共享特性意味着租户的网络性能可能会差异显著。

This applies to both production datacenters and cloud environments.

这适用于生产数据中心和云环境。

Network performance variability hurts application performance which makes tenant costs unpredictable and causes provider revenue loss.

网络性能的变化会损害应用程序的性能，从而使租户成本变得不可预测，并导致提供商收入损失。

Motivated by these factors, this paper makes the case for extending the tenant-provider interface to explicitly account for the network.

在这些因素的驱动下，本文提出了扩展租赁者-提供者接口来明确地考虑网络的情况。

We argue this can be achieved by providing tenants with a virtual network connecting their compute instances.

我们认为这可以通过为租户提供一个连接他们的计算实例的虚拟网络来实现。

To this eﬀect, the key contribution of this paper is the design of virtual network abstractions that capture the trade-oﬀ between the performance guarantees oﬀered to tenants, their costs and the provider revenue. 

为此，本文的关键贡献在于虚拟网络抽象的设计，该设计捕捉到租户的性能保证、他们的成本和提供商的收入之间的权衡。




To illustrate the feasibility of virtual networks, we develop Oktopus, a system that implements the proposed abstractions. 

为了说明虚拟网络的可行性，我们开发了Oktopus系统来实现所提出的抽象。

Using realistic, large-scale simulations and an Oktopus deployment on a 25-node two-tier testbed, we demonstrate that the use of virtual networks yields signiﬁcantly better and more predictable tenant performance. 


通过在25个节点的两层测试台上使用真实的大型模拟和Oktopus部署，我们证明使用虚拟网络可以显著地提高和提高可预测的租户性能。

Further, using a simple pricing model, we ﬁnd that the our abstractions can reduce tenant costs by up to 74% while maintaining provider revenue neutrality. 

此外，使用一个简单的定价模型，我们发现我们的抽象可以减少高达74%的租户成本，同时保持提供者收入中立。

# 研究背景
