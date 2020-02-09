# ABSTRACT 摘要

##

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

##

To illustrate the feasibility of virtual networks, we develop Oktopus, a system that implements the proposed abstractions. 

为了说明虚拟网络的可行性，我们开发了Oktopus系统来实现所提出的抽象。

Using realistic, large-scale simulations and an Oktopus deployment on a 25-node two-tier testbed, we demonstrate that the use of virtual networks yields signiﬁcantly better and more predictable tenant performance. 

通过在25个节点的两层测试台上使用真实的大型模拟和Oktopus部署，我们证明使用虚拟网络可以显著地提高和优化可预测的租户性能。

Further, using a simple pricing model, we ﬁnd that the our abstractions can reduce tenant costs by up to 74% while maintaining provider revenue neutrality. 

此外，使用一个简单的定价模型，我们发现我们的抽象可以减少高达74%的租户成本，同时保持提供者收入中立。

# 研究背景

## 1. INTRODUCTION 介绍

云提供商和租户之间接口的简单性极大地促进了云数据中心的日益流行，这些数据中心提供按需使用计算资源。租户仅仅要求他们所需的计算和存储资源的数量，并按即付即用的方式收取费用。

虽然这个接口很吸引人，也很简单，但是它忽略了一个关键资源，即(云内)网络。云提供商不向租户提供有保障的网络资源。相反，租户的计算实例(虚拟机或简称为vm)通过所有租户共享的网络进行通信。因此，租户的vm之间的流量所获得的带宽取决于租户控制范围之外的各种因素，如网络负载和租户的vm的位置，而数据中心网络拓扑[14]的超额订阅特性又进一步加剧了这种情况。不可避免地，这将导致云网络向租户提供的性能的高度可变性[13、23、24、30]，而这又会对租户和提供商产生一些负面影响。

-不可预测的应用程序性能和租户成本。可变的网络性能是导致云[30]中应用程序性能不可预测的主要原因之一，这是采用云的主要障碍[10,26]。这适用于广泛的应用程序:从面向用户的web应用程序[18,30]到处理事务的web应用程序[21]和类似于MapReduce的数据密集型应用程序[30,38]。此外，由于租户根据占用vm的时间支付费用，而这一时间又受到网络的影响，因此租户最终将为网络流量支付费用;然而，这样的交流应该是免费的(隐性成本)。

-限制云适用性。缺乏保证的网络性能严重阻碍了云支持各种依赖于可预测性能的应用程序的能力。云计算中的高性能计算和科学计算应用程序的性能差且易变，这一点已经得到了很好的证明[17,33]。这同样适用于数据并行应用程序，如MapReduce，它依靠网络以较高的速率传输大量数据。事实上，Amazon的ClusterCompute[2]解决了这个问题，它以高昂的成本为租户提供了一个专用的10Gbps网络，并且没有超额订阅。

-生产数据中心效率低下和收入损失。上面的参数不仅适用于云数据中心，还适用于具有多个租户(产品组)、应用程序(搜索、广告、MapReduce)和服务(BigTable、HDFS、GFS)的任何数据中心。例如，在运行MapReduce作业的生产数据中心中，可变的网络性能会导致作业调度性能低下，并对数据中心的吞吐量产生显著影响[7,31]。此外，这种由网络引发的应用程序不可预测性使得调度作业在质量上变得更加困难，并且妨碍了程序员的工作效率，更不用说在收入[7]方面的巨大损失了。

这些限制是由于租户期望的网络性能和实现的网络性能不匹配造成的，这会损害租户和提供商。在这些因素的驱动下，本文解决了扩展提供者和租户之间的接口以明确地考虑网络资源，同时保持其简单性的挑战。我们的首要目标是允许租户表达他们的网络需求，同时确保供应商能够灵活地考虑这些需求。为此，我们建议使用“虚拟网络”将租户需求公开给提供商。租户除了获得计算实例外，还可以获得一个连接其实例的虚拟网络。虚拟网络将租户性能与底层基础设施隔离开来。这样的解耦也有利于供应商——它们可以修改它们的物理拓扑，而不会影响租户。

虚拟网络的概念引出了一个重要的问题:虚拟网络拓扑应该是什么样的?一方面，提供给租户的抽象必须符合应用程序需求。另一方面，抽象控制底层物理网络基础设施上的多路复用的数量，从而控制并发租户的数量。在此指导下，我们提出了两个新颖的抽象概念，以满足应用程序需求，同时保持租户成本低和供应商收入有吸引力。第一个称为虚拟集群，它提供了将所有vm连接到单个非超额订阅(虚拟)交换机的假象。这适用于数据密集型应用程序，如MapReduce，它的特征是所有到所有的流量模式。第二个集群名为虚拟超额订阅集群，它模拟一个超额订阅的两层集群，该集群适合具有本地通信模式的应用程序。

本文的主要贡献是虚拟网络抽象的设计和对租户保障、租户成本和提供商收益之间的权衡的探索。我们进一步介绍了Oktopus，一个实现我们的抽象的系统。Oktopus将租户虚拟网络映射到在线设置中的物理网络，并强制这些映射。通过在25个节点的测试台上进行大量的模拟和部署，我们发现通过虚拟网络表达需求能够在租户和提供者之间建立一种共生关系;租户实现了更好的和可预测的性能，而改进的数据中心吞吐量(25-435%，取决于抽象和工作负载)增加了提供者收入。

Oktopus的一个关键要点是我们的抽象可以在今天部署:它们不需要对租户应用程序进行任何更改，也不需要对路由器和交换机进行更改。此外，向租户提供有保障的网络带宽为显式带宽收费打开了大门。使用今天的云定价数据，我们发现虚拟网络可以减少中位数租户成本高达74%，同时确保供应商的收入中立。

更一般地说，我们认为可预测的网络性能是迈向更大目标的一个小而重要的步骤，这个目标是为多租户数据中心[36]中的租户提供明确的成本-性能权衡，从而消除采用云的一个重要障碍。

