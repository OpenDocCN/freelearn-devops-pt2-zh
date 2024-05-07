# 第二章：Kubernetes 简介

正如前一章末尾提到的，本章将讨论 Kubernetes。我们将讨论：

+   Kubernetes 的简要历史-它从哪里来？

+   它是如何运作的？

+   Kubernetes 的用例是什么，谁在使用它？

+   为什么要在服务器上运行无服务器？

# Kubernetes 的简要历史

在讨论 Kubernetes 的起源之前，我们应该快速讨论一下 Kubernetes 是什么。它的发音是**koo-ber-net****-eez**，有时被称为**K8s**。**Kubernetes**是希腊语，意思是船长或船舵手，考虑到 Kubernetes 的设计目的，这个名字非常贴切。该项目的网站位于[`kubernetes.io/`](https://kubernetes.io/)，描述了它是：

“用于自动化部署、扩展和管理容器化应用的开源系统。”

该项目起源于谷歌内部的一个名为**Borg**的项目。在 Docker 引起轰动之前，谷歌长期使用容器技术。

# 控制组

谷歌自己的容器之旅始于 2006 年，当时他们的两名工程师开始了**控制组**（**cgroups**）项目。这是 Linux 内核的一个功能，可以隔离诸如 RAM、CPU、网络和磁盘 I/O 等资源，以供一组进程使用。cgroups 最初是在 2007 年发布的，在 2008 年初，该功能被合并到 Linux 内核主线版本 2.6.24 中。

您可以在[`kernelnewbies.org/Linux_2_6_24`](https://kernelnewbies.org/Linux_2_6_24)找到 Linux 内核 2.6.24 版本的发布说明。您可以在*重要事项*列表中的*第 10 点*找到有关 cgroups 引入的信息，其中讨论了允许 cgroups 连接到内核的框架。

# lmctfy

几年后的 2013 年 10 月，谷歌发布了他们自己的容器系统的开源版本，名为**lmctfy**，实际上是**Let Me Contain That For You**的缩写。这个工具实际上是他们在自己的服务器上使用的，用于运行 Linux 应用容器，它被设计为 LXC 的替代品。

lmctfy、LXC 和 Docker 都占据着同样的领域。为此，谷歌实际上在 2015 年停止了 lmctfy 的所有开发。该项目的 GitHub 页面上有一则声明，谷歌一直在与 Docker 合作，他们正在将 lmctfy 的核心概念移植到 libcontainer 中。

# Borg

这就是 Borg 项目的由来。谷歌大量使用容器，我说的是*大量*。2014 年 5 月，谷歌的 Joe Beda 在 Gluecon 上做了一个名为*大规模容器*的演讲。演讲中有一些引人注目的引用，比如：

“谷歌所有的东西都在容器中运行。”

而最常谈论的一个是：

“我们每周启动超过 20 亿个容器。”

这相当于每秒大约 3000 个，在演讲中提到，这个数字不包括任何长时间运行的容器。

虽然 Joe 详细介绍了谷歌当时如何使用容器，但他并没有直接提到 Borg 项目；相反，它只是被称为一个集群调度器。

演讲的最后一个要点是题为*声明式优于命令式*的幻灯片，介绍了以下概念：

+   命令式：在那台服务器上启动这个容器

+   声明式：运行 100 个此容器的副本，目标是同时最多有 2 个任务处于停机状态

这个概念解释了谷歌是如何能够每周启动超过 20 亿个容器，而不必真正管理超过 20 亿个容器。

直到 2015 年谷歌发表了一篇名为《谷歌 Borg 的大规模集群管理》的论文，我们才真正了解到了 Joe Beda 在前一年提到的集群调度器的实践和设计决策。

论文讨论了谷歌内部的工具 Borg 是如何运行成千上万的作业的，这些作业几乎构成了谷歌所有应用程序的集群，这些集群由成千上万台机器组成。

然后它揭示了像 Google Mail、Google Docs 和 Google Search 这样的面向客户的服务也都是从 Borg 管理的集群中提供的，以及他们自己的内部工具。它详细介绍了用户可以使用的作业规范语言，以声明他们期望的状态，使用户能够轻松部署他们的应用程序，而不必担心在谷歌基础设施中部署应用程序所需的所有步骤。

我建议阅读这篇论文，因为它很好地概述了谷歌是如何处理自己的容器服务的。

另外，如果你在想，Borg 是以《星际迷航：下一代》电视剧中的外星种族命名的。

# 项目七

2014 年，Joe Beda、Brendan Burns 和 Craig McLuckie 加入了 Brian Grant 和 Tim Hockin 参与了第七号项目。

这个项目以《星际迷航》中的角色“第七号九”命名，旨在制作一个更友好的 Borg 版本。在第一次提交时，该项目已经有了一个外部名称，即 Kubernetes。

你可以在[`github.com/kubernetes/kubernetes/commit/2c4b3a562ce34cddc3f8218a2c4d11c7310e6d56`](https://github.com/kubernetes/kubernetes/commit/2c4b3a562ce34cddc3f8218a2c4d11c7310e6d56)看到第一次提交，第一个真正稳定的版本是在四个月后发布的，可以在[`github.com/kubernetes/kubernetes/releases/tag/v0.4`](https://github.com/kubernetes/kubernetes/releases/tag/v0.4)找到。

最初，Kubernetes 的目标是将谷歌从 Borg 和运行其大型容器集群中学到的一切开源化，作为吸引客户使用谷歌自己的公共云平台的一种方式——这就是为什么你可能仍然会在项目的原始 GitHub 页面上找到对该项目的引用[`github.com/GoogleCloudPlatform/kubernetes/`](https://github.com/GoogleCloudPlatform/kubernetes/)。

然而，到了 2015 年 7 月的 1.0 版本发布时，谷歌已经意识到它已经远远超出了这个范畴，他们加入了 Linux 基金会、Twitter、英特尔、Docker 和 VMware（举几个例子），共同组建了云原生计算基金会。作为这一新合作的一部分，谷歌将 Kubernetes 项目捐赠为新组织的基础。

此后，其他项目也加入了 Kubernetes，比如：

+   Prometheus（[`prometheus.io/`](https://prometheus.io/)），最初由 SoundCloud 开发，是一个可用于存储指标的时间序列数据库。

+   Fluentd（[`www.fluentd.org/`](https://www.fluentd.org/)）是一个数据收集器，允许你从许多不同的来源获取数据，对其进行过滤或规范化，然后将其路由到诸如 Elasticsearch、MongoDB 或 Hadoop（举几个例子）这样的存储引擎。

+   containerd（[`containerd.io/`](http://containerd.io/)）是一个由 Docker 最初开发的开源容器运行时，用于实现 Open Container Initiative 标准。

+   CoreDNS（[`coredns.io/`](https://coredns.io/)）是一个完全基于插件构建的 DNS 服务，这意味着你可以创建传统上配置极其复杂的 DNS 服务。

除此之外，像 AWS、微软、红帽和甲骨文这样的新成员都在支持和为基金会的项目提供资源。

# Kubernetes 概述

现在我们对 Kubernetes 的起源有了一个概念，我们应该逐步了解构成典型 Kubernetes 集群的所有不同组件。

Kubernetes 本身是用 Go 编写的。虽然项目的 GitHub 页面显示该项目目前 84.9%是 Go，其余的 5.8%是 HTML，4.7%是 Python，3.3%是 Shell（其余是配置/规范文件等），都是文档和辅助脚本。

Go 是一种由 Google 开发并开源的编程语言，谷歌将其描述为*一种快速、静态类型、编译语言，感觉像一种动态类型、解释语言*。更多信息，请参见[`golang.org/`](https://golang.org/)。

# 组件

Kubernetes 有两个主要的服务器角色：主服务器和节点；每个角色都由多个组件组成。

主服务器是集群的大脑，它们决定 pod（在下一节中更多介绍）在集群内部部署的位置，并且对集群的健康状况以及 pod 本身的健康状况进行操作和查看。

主服务器的核心组件包括：

+   `kube-apiserver`：这是您的 Kubernetes 控制面板的前端；无论您使用什么来管理您的集群，它都将直接与此 API 服务通信。

+   `etcd`：`etcd`是 Kubernetes 用来存储集群状态的分布式键值存储。

+   `kube-controller-manager`：此服务在后台工作，以维护您的集群。它查找加入和离开集群的节点，确保正在运行正确数量的 pod，并且它们健康等等。

+   `cloud-controller-manager`：这项服务是 Kubernetes 的新功能。它与`kube-controller-manager`一起工作，其目的是与 AWS、Google Cloud 和 Microsoft Azure 等云提供商的 API 进行交互。它执行的任务示例可能是，如果要从集群中删除一个节点，它将检查您的云服务 API，看看节点是否仍然存在。如果存在，则可能会出现问题；如果不存在，则很可能是因为缩放事件而删除了节点。

+   `kube-scheduler`：根据一系列规则、利用率和可用性选择 pod 应该在哪里启动。

接下来我们有节点。一旦部署，主节点与安装在节点上的组件进行交互，以在集群内实现变化；这些是您的 pod 运行的地方。

组成节点的组件有：

+   `kubelet`：这是在节点上运行的主要组件。它负责接受来自主服务器的指令并报告回去。

+   `kube-proxy`：这项服务有助于集群通信。它充当节点上所有网络流量的基本代理，并能够配置 TCP/UDP 转发或充当 TCP/UDP 轮询负载均衡器到多个后端。

+   `docker`或`rkt`：这些是节点上实际的容器引擎。`kubelet`服务与它们交互，以启动和管理运行在集群节点上的容器。在接下来的章节中，我们将看到运行这两种节点的示例。

+   `supervisord`：这个进程管理器和监视器维护着节点上其他服务的可用性，比如`kubelet`、`docker`和`rkt`。

+   `fluentd`：这项服务有助于集群级别的日志记录。

你可能已经注意到，这些服务中唯一提到容器的是`docker`和`rkt`。Kubernetes 实际上并不直接与您的容器交互；相反，它与一个 pod 通信。

# Pods 和服务

正如前面提到的，Kubernetes 不部署容器；相反，它启动 pod。在其最简单的形式中，一个 pod 实际上可以是一个单一的容器；然而，通常一个 pod 由多个容器、存储和网络组成。

以下内容仅供说明，不是一个实际的例子；我们将在下一章中通过一个实际的例子来进行讲解。

把一个 pod 想象成一个完整的应用程序；例如，如果你运行一个简单的 web 应用程序，它可能只运行一个 NGINX 容器——这个 pod 的定义文件将看起来像下面这样：

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx:latest
    ports:
    - containerPort: 8080
```

正如你所看到的，我们提供了关于我们的 pod 的一些简单元数据，这种情况下只是名称，这样我们就可以识别它。然后我们定义了一个单一的容器，它正在运行来自 Docker hub 的最新 NGINX 镜像，并且端口`8080`是开放的。

就目前而言，这个 pod 是相当无用的，因为我们只打算显示一个欢迎页面。接下来，我们需要添加一个卷来存储我们网站的数据。为了做到这一点，我们的 pod 定义文件将如下所示：

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx:latest
    volumeMounts:
    - mountPath: /srv/www
      name: web-data
      readOnly: true
    ports:
    - containerPort: 8080
 volumes:
 - name: web-data
 emptyDir: {} 
```

正如你所看到的，我们现在正在创建一个名为`web-data`的卷，并将其以只读方式挂载到`/srv/www`，这是我们 NGINX 容器上的默认网站根目录。但这还是有点毫无意义，因为我们的卷是空的，这意味着我们的所有访问者将只看到一个 404 页面。

让我们添加第二个容器，它将从 Amazon S3 存储桶同步我们网站的 HTML：

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx:latest
    volumeMounts:
    - mountPath: /srv/www
      name: web-data
      readOnly: true
    ports:
    - containerPort: 8080
  - name: sync
    image: ocasta/sync-s3:latest
    volumeMounts:
    - mountPath: /data
      name: web-data
      readOnly: false
    env:
    - ACCESS_KEY: "awskey"
      SECRET_KEY: "aws_secret"
      S3_PATH: "s3://my-awesome-website/"
      SYNC_FROM_S3: "true"
  volumes:
  - name: web-data
    emptyDir: {}
```

现在我们有两个容器：一个是 NGINX，现在还有一个运行`s3 sync`命令的容器（[`github.com/ocastastudios/docker-sync-s3/`](https://github.com/ocastastudios/docker-sync-s3/)）。这将把我们网站的所有数据从名为`my-awesome-website`的 Amazon S3 存储桶复制到与 NGINX 容器共享的卷中。这意味着我们现在有一个网站；请注意，这一次，因为我们想要写入卷，我们不会将其挂载为只读。

到目前为止，你可能会想到自己；我们有一个从 Amazon S3 存储桶部署的网站服务的 pod，这一切都是真实的。然而，我们还没有完全完成。我们有一个正在运行的 pod，但我们需要将该 pod 暴露给网络，以便在浏览器中访问它。

为了做到这一点，我们需要启动一个服务。对于我们的示例，服务文件看起来可能是这样的：

```
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080
```

正如你所看到的，服务定义看起来与 pod 的定义类似。我们使用元数据部分设置名称。然后我们选择我们的 NGINX pod，并将端口`80`映射到端口`8080`，这是我们的 pod 正在侦听的端口。

如前所述，当我们启动第一个 Kubernetes 集群时，我们将在下一章更详细地讨论这个问题，但现在，这应该让你对 Kubernetes 的运作方式有一个很好的了解。

# 工作负载

在上一节中，我们看了 pod 和服务。虽然这些可以手动启动，但你也可以使用控制器来管理你的 pod。这些控制器允许执行不同类型的工作负载。我们将快速看一下不同类型的控制器，还讨论何时使用它们。

# 副本集

ReplicaSet 可用于启动和维护相同 pod 的多个副本。例如，使用我们在上一节中讨论的 NGINX pod，我们可以创建一个 ReplicaSet，启动三个相同 pod 的副本。然后可以在这三个 pod 之间进行负载均衡。

我们的三个 pod 可以分布在多个主机上，这意味着，如果一个主机因任何原因消失，将导致我们的一个 pod 停止服务，它将自动在健康节点上被替换。你还可以使用 ReplicaSet 来自动和手动地添加和删除 pod。

# 部署

您可能会认为使用 ReplicaSet 可以进行滚动升级和回滚。不幸的是，ReplicaSets 只能复制相同版本的 pod；幸运的是，这就是部署的用武之地。

部署控制器旨在更新 ReplicaSet 或 pod。让我们以 NGINX 为例。正如您从以下定义中所看到的，我们有 `3` 个副本都在运行 NGINX 版本 `1.9.14`：

```
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.9.14
        ports:
        - containerPort: 80
```

`kubectl` 是 Kubernetes 的命令行客户端；我们将在下一章中更详细地讨论这个问题。

我们可以使用以下命令进行部署：

```
$ kubectl create -f nginx-deployment.yaml
```

现在假设我们想要更新使用的 NGINX 图像的版本。我们只需要运行以下命令：

```
$ kubectl set image deployment/nginx-deployment nginx=nginx:1.13.5 deployment "nginx-deployment" image updated
```

这将逐个更新每个 pod，直到所有的 pod 都运行新版本的 NGINX。

# StatefulSets

这个控制器是 Kubernetes 中的新功能，旨在取代 PetSets。正如您可能从名称中猜到的那样，pod 作为部署的一部分维护其状态。它们被设计为具有：

+   在整个 pod 生命周期中保持一致的唯一网络标识符

+   持久存储

+   按照您定义的顺序执行的优雅部署和扩展

+   用户定义和控制的自动滚动更新

因此，虽然名称有所变化，但您应该将 StatefulSets 视为宠物，将 ReplicaSets 视为牲畜。

# Kubernetes 使用案例

正如我们在本章中已经提到的，Kubernetes 几乎可以在任何地方运行，从您的本地机器（我们将在下一章中介绍），到您的本地硬件或虚拟机基础设施，甚至可能跨越 AWS、Microsoft Azure 或 Google Cloud 的数百个公共云实例。事实上，您甚至可以在 Kubernetes 集群中跨多个环境。

这意味着无论您在何处运行应用程序，都会获得一致的体验，但也可以利用底层平台的功能，比如负载平衡、持久存储和自动扩展，而无需真正设计应用程序以意识到它是在运行，比如在 AWS 或 Microsoft Azure 上。

阅读成功案例时你会注意到的一个共同点是，人们谈论的是不被锁定在一个特定的供应商上。由于 Kubernetes 是开源的，他们不会被任何许可成本所限制。如果他们遇到问题或想要添加功能，他们可以直接深入源代码进行更改；他们也可以通过拉取请求将他们所做的任何更改贡献回项目中。

另外，正如前面讨论的，使用 Kubernetes 使他们不会被锁定在任何一个特定的平台供应商或架构上。这是因为可以合理地假设 Kubernetes 在安装在其他平台时会以完全相同的方式运行。因此，突然之间，您可以相对轻松地将您的应用程序在不同提供商之间移动。

另一个常见的用例是运维团队将 Kubernetes 用作基础设施即服务（IaaS）平台。这使他们能够通过 API、Web 和 CLI 向开发人员提供资源，这意味着他们可以轻松地融入自己的工作流程。它还为本地开发提供了一个一致的环境，从暂存或用户验收测试（UAT）到最终在生产环境中运行他们的应用程序。

这也是为什么使用 Kubernetes 执行无服务器工作负载是一个好主意的部分原因。您不会被任何一个提供商锁定，比如 AWS 或 Microsoft Azure。事实上，您应该把 Kubernetes 看作是一个云平台，就像我们在第一章中看到的那些；它有一个基于 Web 的控制台，一个 API 和一个命令行客户端。

# 参考资料

关于 Kubernetes 的几个案例研究，用户详细介绍了他们在使用 Kubernetes 过程中的经历：

+   **Wink**：[`kubernetes.io/case-studies/wink/`](https://kubernetes.io/case-studies/wink/)

+   **Buffer**：[`kubernetes.io/case-studies/buffer/`](https://kubernetes.io/case-studies/buffer/)

+   **Ancestry**：[`kubernetes.io/case-studies/ancestry/`](https://kubernetes.io/case-studies/ancestry/)

+   **Wikimedia 基金会**：[`kubernetes.io/case-studies/wikimedia/`](https://kubernetes.io/case-studies/wikimedia/)

还有来自以下内容的讨论、采访和演示：

+   **The New Times**：[`www.youtube.com/watch?v=P5qfyv_zGcU`](https://www.youtube.com/watch?v=P5qfyv_zGcU)

+   蒙佐：[`www.youtube.com/watch?v=YkOY7DgXKyw`](https://www.youtube.com/watch?v=YkOY7DgXKyw)

+   高盛：[`blogs.wsj.com/cio/2016/02/24/big-changes-in-goldmans-software-emerge-from-small-containers/`](https://blogs.wsj.com/cio/2016/02/24/big-changes-in-goldmans-software-emerge-from-small-containers/)

最后，您可以在[`www.cncf.io/`](https://www.cncf.io/)了解更多关于 Cloud Native Computing Foundation 的信息。

# 总结

在这一章中，我们谈到了 Kubernetes 的起源，并介绍了一些其使用案例。我们还了解了一些基本功能。

在下一章中，我们将通过在本地安装 Minikube 来亲自体验 Kubernetes。一旦我们安装好了本地的 Kubernetes，我们就可以继续进行第四章，“介绍 Kubeless 功能”，在那里我们将开始在 Kubernetes 上部署我们的第一个无服务器函数。
