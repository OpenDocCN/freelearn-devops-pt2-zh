# 第一章：为开发设置 Kubernetes

欢迎来到*面向开发人员的 Kubernetes*！本章首先将帮助您安装工具，以便您可以在开发中充分利用 Kubernetes。安装完成后，我们将与这些工具进行一些交互，以验证它们是否正常工作。然后，我们将回顾一些您作为开发人员想要了解的基本概念，以有效地使用 Kubernetes。我们将介绍 Kubernetes 中的以下关键资源：

+   容器

+   Pod

+   节点

+   部署

+   副本集

# 开发所需的工具

除了您通常使用的编辑和编程工具之外，您还需要安装软件来利用 Kubernetes。本书的重点是让您可以在本地开发机器上完成所有操作，同时也可以在将来如果需要更多资源，扩展和利用远程 Kubernetes 集群。Kubernetes 的一个好处是它以相同的方式处理一个或一百台计算机，让您可以利用您软件所需的资源，并且无论它们位于何处，都可以一致地进行操作。

本书中的示例将使用本地机器上终端中的命令行工具。主要的工具将是`kubectl`，它与 Kubernetes 集群通信。我们将使用 Minikube 在您自己的开发系统上运行的单台机器的微型 Kubernetes 集群。我建议安装 Docker 的社区版，这样可以轻松地构建用于 Kubernetes 内部的容器：

+   `kubectl`：`kubectl`（如何发音是 Kubernetes 社区内一个有趣的话题）是用于与 Kubernetes 集群一起工作的主要命令行工具。要安装`kubectl`，请访问页面[`kubernetes.io/docs/tasks/tools/install-kubectl/`](https://kubernetes.io/docs/tasks/tools/install-kubectl/)并按照适用于您平台的说明进行操作。

+   `minikube`：要安装 Minikube，请访问页面[`github.com/kubernetes/minikube/releases`](https://github.com/kubernetes/minikube/releases)并按照适用于您平台的说明进行操作。

+   `docker`：要安装 Docker 的社区版，请访问网页[`www.docker.com/community-edition`](https://www.docker.com/community-edition)并按照适用于您平台的说明进行操作。

# 可选工具

除了`kubectl`、`minikube`和`docker`之外，您可能还想利用其他有用的库和命令行工具。

`jq`是一个命令行 JSON 处理器，它可以轻松解析更复杂的数据结构中的结果。我会将其描述为*更擅长处理 JSON 结果的 grep 表亲*。您可以按照[`stedolan.github.io/jq/download/`](https://stedolan.github.io/jq/download/)上的说明安装`jq`。关于`jq`的详细信息以及如何使用它也可以在[`stedolan.github.io/jq/manual/`](https://stedolan.github.io/jq/manual/)上找到。

# 启动本地集群

一旦 Minikube 和 Kubectl 安装完成，就可以启动一个集群。值得知道你正在使用的工具的版本，因为 Kubernetes 是一个发展迅速的项目，如果需要从社区获得帮助，知道这些常用工具的版本将很重要。

我在撰写本文时使用的 Minikube 和`kubectl`的版本是：

+   Minikube：版本 0.22.3

+   `kubectl`：版本 1.8.0

您可以使用以下命令检查您的副本的版本：

```
minikube version
```

这将输出一个版本：

```
minikube version: v0.22.3
```

如果在按照安装说明操作时还没有这样做，请使用 Minikube 启动 Kubernetes。最简单的方法是使用以下命令：

```
minikube start
```

这将下载一个虚拟机映像并启动它，以及在其上的 Kubernetes，作为单机集群。输出将类似于以下内容：

```
Downloading Minikube ISO
 106.36 MB / 106.36 MB [============================================] 100.00% 0s
Getting VM IP address...
Moving files into cluster...
Setting up certs...
Connecting to cluster...
Setting up kubeconfig...
Starting cluster components...
Kubectl is now configured to use the cluster.
```

Minikube 将自动创建`kubectl`访问集群和控制集群所需的文件。完成后，可以获取有关集群的信息以验证其是否正在运行。

首先，您可以直接询问`minikube`关于其状态：

```
minikube status
minikube: Running
cluster: Running
kubectl: Correctly Configured: pointing to minikube-vm at 192.168.64.2
```

如果我们询问`kubectl`关于其版本，它将报告客户端的版本和正在通信的集群的版本：

```
kubectl version
```

第一个输出是`kubectl`客户端的版本：

```
Client Version: version.Info{Major:"1", Minor:"7", GitVersion:"v1.7.5", GitCommit:"17d7182a7ccbb167074be7a87f0a68bd00d58d97", GitTreeState:"clean", BuildDate:"2017-08-31T19:32:26Z", GoVersion:"go1.9", Compiler:"gc", Platform:"darwin/amd64"}
```

立即之后，它将通信并报告集群上 Kubernetes 的版本：

```
Server Version: version.Info{Major:"1", Minor:"7", GitVersion:"v1.7.5", GitCommit:"17d7182a7ccbb167074be7a87f0a68bd00d58d97", GitTreeState:"clean", BuildDate:"2017-09-11T21:52:19Z", GoVersion:"go1.8.3", Compiler:"gc", Platform:"linux/amd64"}
```

我们也可以使用`kubectl`来请求有关集群的信息：

```
kubectl cluster-info
```

然后会看到类似以下的内容：

```
Kubernetes master is running at https://192.168.64.2:8443

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```

此命令主要让您知道您正在通信的 API 服务器是否正在运行。我们可以使用附加命令来请求关键内部组件的特定状态：

```
kubectl get componentstatuses
```

```
NAME                 STATUS    MESSAGE              ERROR
scheduler            Healthy   ok
etcd-0               Healthy   {"health": "true"}
controller-manager   Healthy   ok
```

Kubernetes 还报告并存储了许多事件，您可以请求查看这些事件。这些显示了集群内部发生的情况：

```
kubectl get events
```

```
LASTSEEN   FIRSTSEEN   COUNT     NAME       KIND      SUBOBJECT   TYPE      REASON                    SOURCE                 MESSAGE
2m         2m          1         minikube   Node                  Normal    Starting                  kubelet, minikube      Starting kubelet.
2m         2m          2         minikube   Node                  Normal    NodeHasSufficientDisk     kubelet, minikube      Node minikube status is now: NodeHasSufficientDisk
2m         2m          2         minikube   Node                  Normal    NodeHasSufficientMemory   kubelet, minikube      Node minikube status is now: NodeHasSufficientMemory
2m         2m          2         minikube   Node                  Normal    NodeHasNoDiskPressure     kubelet, minikube      Node minikube status is now: NodeHasNoDiskPressure
2m         2m          1         minikube   Node                  Normal    NodeAllocatableEnforced   kubelet, minikube      Updated Node Allocatable limit across pods
2m         2m          1         minikube   Node                  Normal    Starting                  kube-proxy, minikube   Starting kube-proxy.
2m         2m          1         minikube   Node                  Normal    RegisteredNode            controllermanager      Node minikube event: Registered Node minikube in NodeController
```

# 重置和重新启动您的集群

如果您想清除本地 Minikube 集群并重新启动，这是非常容易做到的。发出一个`delete`命令，然后`start` Minikube 将清除环境并将其重置为一个空白状态：

```
minikube delete Deleting local Kubernetes cluster...
Machine deleted.

minikube start
Starting local Kubernetes v1.7.5 cluster...
Starting VM...
Getting VM IP address...
Moving files into cluster...
Setting up certs...
Connecting to cluster...
Setting up kubeconfig...
Starting cluster components...
Kubectl is now configured to use the cluster.
```

# 查看 Minikube 内置和包含的内容

使用 Minikube，您可以通过一个命令为 Kubernetes 集群启动基于 Web 的仪表板：

```
minikube dashboard
```

这将打开一个浏览器，并显示一个 Kubernetes 集群的 Web 界面。如果您查看浏览器窗口中的 URL 地址，您会发现它指向之前从`kubectl cluster-info`命令返回的相同 IP 地址，运行在端口`30000`上。仪表板在 Kubernetes 内部运行，并且它不是唯一的运行在其中的东西。

Kubernetes 是自托管的，支持 Kubernetes 运行的支持组件，如仪表板、DNS 等，都在 Kubernetes 内部运行。您可以通过查询集群中所有 Pod 的状态来查看所有这些组件的状态：

```
kubectl get pods --all-namespaces
```

```
NAMESPACE     NAME                          READY     STATUS    RESTARTS   AGE
kube-system   kube-addon-manager-minikube   1/1       Running   0          6m
kube-system   kube-dns-910330662-6pctd      3/3       Running   0          6m
kube-system   kubernetes-dashboard-91nmv    1/1       Running   0          6m
```

请注意，在此命令中我们使用了`--all-namespaces`选项。默认情况下，`kubectl`只会显示位于默认命名空间中的 Kubernetes 资源。由于我们还没有运行任何东西，如果我们调用`kubectl get pods`，我们将只会得到一个空列表。Pod 并不是唯一的 Kubernetes 资源；您可以查询许多不同的资源，其中一些我将在本章后面描述，更多的将在后续章节中描述。

暂时，再调用一个命令来获取服务列表：

```
kubectl get services --all-namespaces
```

这将输出所有服务：

```
NAMESPACE     NAME                   CLUSTER-IP   EXTERNAL-IP   PORT(S)         AGE
default       kubernetes             10.0.0.1     <none>        443/TCP         3m
kube-system   kube-dns               10.0.0.10    <none>        53/UDP,53/TCP   2m
kube-system   kubernetes-dashboard   10.0.0.147   <nodes>       80:30000/TCP    2m
```

请注意名为`kubernetes-dashboard`的服务具有`Cluster-IP`值和端口`80:30000`。该端口配置表明，在支持`kubernetes-dashboard`服务的 Pod 中，它将把来自端口`30000`的任何请求转发到容器内的端口`80`。您可能已经注意到，Cluster IP 的 IP 地址与我们之前在`kubectl cluster-info`命令中看到的 Kubernetes 主节点报告的 IP 地址非常不同。

重要的是要知道，Kubernetes 中的所有内容都在一个私有的、隔离的网络上运行，通常无法从集群外部访问。我们将在未来的章节中更详细地介绍这一点。现在，只需知道`minikube`在其中有一些额外的特殊配置来暴露仪表板。

# 验证 Docker

Kubernetes 支持多种运行容器的方式，Docker 是最常见、最方便的方式。在本书中，我们将使用 Docker 来帮助我们创建在 Kubernetes 中运行的镜像。

您可以通过运行以下命令来查看您安装的 Docker 版本并验证其是否可操作：

```
docker  version
```

像`kubectl`一样，它将报告`docker`客户端版本以及服务器版本，您的输出可能看起来像以下内容：

```
Client:
 Version: 17.09.0-ce
 API version: 1.32
 Go version: go1.8.3
 Git commit: afdb6d4
 Built: Tue Sep 26 22:40:09 2017
 OS/Arch: darwin/amd64
```

```
Server:
 Version: 17.09.0-ce
 API version: 1.32 (minimum version 1.12)
 Go version: go1.8.3
 Git commit: afdb6d4
 Built: Tue Sep 26 22:45:38 2017
 OS/Arch: linux/amd64
 Experimental: false
```

通过使用`docker images`命令，您可以查看本地可用的容器镜像，并使用`docker pull`命令，您可以请求特定的镜像。在下一章的示例中，我们将基于 alpine 容器镜像构建我们的软件，因此让我们继续拉取该镜像以验证您的环境是否正常工作：

```
docker pull alpine 
Using default tag: latest
latest: Pulling from library/alpine
Digest: sha256:f006ecbb824d87947d0b51ab8488634bf69fe4094959d935c0c103f4820a417d
Status: Image is up to date for alpine:latest
```

然后，您可以使用以下命令查看镜像：

```
docker images
```

```
REPOSITORY TAG IMAGE ID CREATED SIZE
alpine latest 76da55c8019d 3 weeks ago 3.97MB</strong>
```

如果在尝试拉取 alpine 镜像时出现错误，这可能意味着您需要通过代理工作，或者以其他方式受限制地访问互联网以满足您的镜像需求。如果您处于这种情况，您可能需要查看 Docker 关于如何设置和使用代理的信息。

# 清除和清理 Docker 镜像

由于我们将使用 Docker 来构建容器镜像，了解如何摆脱镜像将是有用的。您已经使用`docker image`命令看到了镜像列表。还有一些由 Docker 维护的中间镜像在输出中是隐藏的。要查看 Docker 存储的所有镜像，请使用以下命令：

```
docker images -a
```

如果您只按照前文拉取了 alpine 镜像，可能不会看到任何其他镜像，但在下一章中构建镜像时，这个列表将会增长。

您可以使用`docker rmi`命令后跟镜像的名称来删除镜像。默认情况下，Docker 将尝试维护容器最近使用或引用的镜像。因此，您可能需要强制删除以清理镜像。

如果您想重置并删除所有镜像并重新开始，有一个方便的命令可以做到。通过结合 Docker 镜像和`docker rmi`，我们可以要求它强制删除所有已知的镜像：

```
docker rmi -f $(docker images -a -q)
```

# Kubernetes 概念 - 容器

Kubernetes（以及这个领域中的其他技术）都是关于管理和编排容器的。容器实际上是一个包裹了一系列 Linux 技术的名称，其中最突出的是容器镜像格式和 Linux 如何隔离进程，利用 cgroups。

在实际情况下，当有人谈论容器时，他们通常意味着有一个包含运行单个进程所需的一切的镜像。在这种情况下，容器不仅是镜像，还包括关于如何调用和运行它的信息。容器还表现得好像它们有自己的网络访问权限。实际上，这是由运行容器的 Linux 操作系统共享的。

当我们想要编写在 Kubernetes 下运行的代码时，我们总是在谈论将其打包并准备在容器内运行。本书后面更复杂的示例将利用多个容器一起工作。

在容器内运行多个进程是完全可能的，但这通常被视为不好的做法，因为容器理想上适合表示单个进程以及如何调用它，并且不应被视为完整虚拟机的同一物体。

如果你通常使用 Python 进行开发，那么你可能熟悉使用类似`pip`的工具来下载你需要的库和模块，并使用类似`python your_file`的命令来调用你的程序。如果你是 Node 开发人员，那么你更可能熟悉使用`npm`或`yarn`来安装你需要的依赖，并使用`node your_file`来运行你的代码。

如果你想把所有这些打包起来，在另一台机器上运行，你可能要重新执行所有下载库和运行代码的指令，或者可能将整个目录压缩成 ZIP 文件，然后将其移动到想要运行的地方。容器是一种将所有信息收集到单个镜像中的方式，以便可以轻松地移动、安装和在 Linux 操作系统上运行。最初由 Docker 创建，现在由**Open Container Initiative**（**OCI**）（[`www.opencontainers.org`](https://www.opencontainers.org)）维护规范。

虽然容器是 Kubernetes 中的最小构建块，但 Kubernetes 处理的最小单位是 Pod。

# Kubernetes 资源 - Pod

Pod 是 Kubernetes 管理的最小单位，也是系统其余部分构建在其上的基本单位。创建 Kubernetes 的团队发现让开发人员指定应始终在同一操作系统上运行的进程，并且应该一起运行的进程组合应该是被调度、运行和管理的单位是值得的。

在本章的前面，您看到 Kubernetes 的一个基本实例有一些软件在 Pod 中运行。Kubernetes 的许多部分都是使用这些相同的概念和抽象来运行的，这使得 Kubernetes 能够自托管其自己的软件。一些用于运行 Kubernetes 集群的软件是在集群之外管理的，但越来越多地利用了 Pod 的概念，包括 DNS 服务、仪表板和控制器管理器，它们通过 Kubernetes 协调所有的控制操作。

一个 Pod 由一个或多个容器以及与这些容器相关的信息组成。当您询问 Kubernetes 有关一个 Pod 时，它将返回一个数据结构，其中包括一个或多个容器的列表，以及 Kubernetes 用来协调 Pod 与其他 Pod 以及 Kubernetes 应该如何在程序失败时采取行动的各种元数据。元数据还可以定义诸如*亲和性*之类的东西，影响 Pod 在集群中可以被调度的位置，以及如何获取容器镜像等期望。重要的是要知道，Pod 不打算被视为持久的、长期存在的实体。

它们被创建和销毁，本质上是短暂的。这允许单独的逻辑 - 包含在控制器中 - 来管理规模和可用性等责任。正是这种分工的分离使得 Kubernetes 能够在发生故障时提供自我修复的手段，并提供一些自动扩展的能力。

由 Kubernetes 运行的 Pod 具有一些特定的保证：

+   一个 Pod 中的所有容器将在同一节点上运行。

+   在 Pod 中运行的任何容器都将与同一 Pod 中的其他容器共享节点的网络

+   Pod 中的容器可以通过挂载到容器的卷共享文件。

+   一个 Pod 有一个明确的生命周期，并且始终保留在它启动的节点上。

在实际操作中，当您想要了解 Kubernetes 集群上运行的内容时，通常会想要了解 Kubernetes 中运行的 Pod 及其状态。

Kubernetes 维护并报告 Pod 的状态，以及组成 Pod 的每个容器的状态。容器的状态包括`Running`、`Terminated`和`Waiting`。Pod 的生命周期稍微复杂一些，包括严格定义的阶段和一组 PodStatus。阶段包括`Pending`、`Running`、`Succeeded`、`Failed`或`Unknown`，阶段中包含的具体细节在[`kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#pod-phase`](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#pod-phase)中有文档记录。

Pod 还可以包含探针，用于主动检查容器的某些状态信息。Kubernetes 控制器部署和使用的两个常见探针是`livenessProbe`和`readinessProbe`。`livenessProbe`定义容器是否正在运行。如果没有运行，Kubernetes 基础设施会终止相关容器，然后应用为 Pod 定义的重启策略。`readinessProbe`用于指示容器是否准备好提供服务请求。`readinessProbe`的结果与其他 Kubernetes 机制（稍后我们将详细介绍）一起使用，以将流量转发到相关容器。通常，探针被设置为允许容器中的软件向 Kubernetes 提供反馈循环。您可以在[`kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#container-probes`](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#container-probes)找到有关探针的更多详细信息，如何定义它们以及它们的用途。我们将在未来的章节中详细讨论探针。

# 命名空间

Pod 被收集到命名空间中，用于将 Pod 组合在一起以实现各种目的。在之前使用`--all-namespaces`选项请求集群中所有 Pod 的状态时，您已经看到了命名空间的一个示例。

命名空间可用于提供资源使用配额和限制，对 Kubernetes 在集群内部创建的 DNS 名称产生影响，并且在将来可能会影响访问控制策略。如果在通过`kubectl`与 Kubernetes 交互时未指定命名空间，则命令会假定您正在使用名为`default`的默认命名空间。

# 编写您的 Pods 和容器的代码

成功使用 Kubernetes 的关键之一是考虑您希望代码如何运行，并将其结构化，使其清晰地适应 Pod 和容器的结构。通过将软件解决方案结构化为将问题分解为符合 Kubernetes 提供的约束和保证的组件，您可以轻松地利用并行性和容器编排，以像使用单台机器一样无缝地使用多台机器。

Kubernetes 提供的保证和抽象反映了 Google（和其他公司）在以大规模、可靠和冗余的方式运行其软件和服务方面多年的经验，利用水平扩展的模式来解决重大问题。

# Kubernetes 资源-节点

节点是一台机器，通常运行 Linux，已添加到 Kubernetes 集群中。它可以是物理机器或虚拟机。在`minikube`的情况下，它是一台运行 Kubernetes 所有软件的单个虚拟机。在较大的 Kubernetes 集群中，您可能有一台或多台专门用于管理集群的机器，以及单独的机器用于运行工作负载。Kubernetes 通过跟踪节点的资源使用情况、调度、启动（如果需要，重新启动）Pod，以及协调连接 Pod 或在集群外部公开它们的其他机制来管理其资源。

节点可以（而且确实）与它们关联的元数据，以便 Kubernetes 可以意识到相关的差异，并在调度和运行 Pod 时考虑这些差异。Kubernetes 可以支持各种各样的机器共同工作，并在所有这些机器上高效运行软件，或者将 Pod 的调度限制在只有所需资源的机器上（例如，GPU）。

# 网络

我们之前提到，Pod 中的所有容器共享节点的网络。此外，Kubernetes 集群中的所有节点都应该相互连接并共享一个私有的集群范围网络。当 Kubernetes 在 Pod 中运行容器时，它是在这个隔离的网络中进行的。Kubernetes 负责处理 IP 地址，创建 DNS 条目，并确保一个 Pod 可以与同一 Kubernetes 集群中的另一个 Pod 进行通信。

另一个资源是服务，我们稍后会深入研究，这是 Kubernetes 用来在私有网络中向其他 Pod 公开或处理集群内外的连接的工具。默认情况下，在这个私有的隔离网络中运行的 Pod 不会暴露在 Kubernetes 集群之外。根据您的 Kubernetes 集群是如何创建的，有多种方式可以打开从集群外部访问您的软件的途径，我们将在稍后的服务中详细介绍，包括负载均衡器、节点端口和入口。

# 控制器

Kubernetes 的构建理念是告诉它你想要什么，它知道如何去做。当您与 Kubernetes 交互时，您断言您希望一个或多个资源处于特定状态，具有特定版本等等。控制器是跟踪这些资源并尝试按照您描述的方式运行软件的大脑所在的地方。这些描述可以包括运行容器镜像的副本数量、更新 Pod 中运行的软件版本，以及处理节点故障的情况，其中您意外地失去了集群的一部分。

Kubernetes 中使用了各种控制器，它们大多隐藏在我们将进一步深入研究的两个关键资源后面：部署和 ReplicaSets。

# Kubernetes 资源 – ReplicaSet

ReplicaSet 包装了 Pod，定义了需要并行运行多少个 Pod。ReplicaSet 通常又被部署包装。ReplicaSets 不经常直接使用，但对于表示水平扩展来说至关重要——表示要运行的并行 Pod 的数量。

ReplicaSet 与 Pod 相关联，并指示集群中应该运行多少个该 Pod 的实例。ReplicaSet 还意味着 Kubernetes 有一个控制器来监视持续状态，并知道要保持运行多少个 Pod。这就是 Kubernetes 真正开始为您工作的地方，如果您在 ReplicaSet 中指定了三个 Pod，而其中一个失败了，Kubernetes 将自动为您安排并运行另一个 Pod。

# Kubernetes 资源 – 部署

在 Kubernetes 上运行代码的最常见和推荐方式是使用部署，由部署控制器管理。我们将在接下来的章节中探讨部署，直接指定它们并使用诸如 `kubectl run` 等命令隐式创建它们。

Pod 本身很有趣，但受限，特别是因为它旨在是短暂的。如果一个节点死掉（或者被关机），那个节点上的所有 Pod 都将停止运行。ReplicaSets 提供了自我修复的能力。它们在集群内工作，以识别 Pod 何时不再可用，并尝试调度另一个 Pod，通常是为了使服务恢复在线，或者继续工作。

部署控制器包装并扩展了 ReplicaSet 控制器，主要负责推出软件更新并管理更新部署资源的过程。部署控制器包括元数据设置，以了解要保持运行多少个 Pod，以便通过添加容器的新版本来启用无缝滚动更新软件，并在您请求时停止旧版本。

# 代表 Kubernetes 资源

Kubernetes 资源通常可以表示为 JSON 或 YAML 数据结构。Kubernetes 专门设计成可以保存这些文件，当您想要运行软件时，可以使用诸如`kubectl deploy`之类的命令，并提供先前创建的定义，然后使用它来运行您的软件。在我们的下一章中，我们将开始展示这些资源的具体示例，并为我们的使用构建它们。

当我们进入下一个和未来的章节中的示例时，我们将使用 YAML 来描述我们的资源，并通过`kubectl`以 JSON 格式请求数据。所有这些数据结构都是针对 Kubernetes 的每个版本进行正式定义的，以及 Kubernetes 提供的用于操作它们的 REST API。所有 Kubernetes 资源的正式定义都在源代码控制中使用 OpenAPI（也称为**Swagger**）进行维护，并可以在[`github.com/kubernetes/kubernetes/tree/master/api/swagger-spec`](https://github.com/kubernetes/kubernetes/tree/master/api/swagger-spec)上查看。

# 总结

在本章中，我们安装了`minikube`和`kubectl`，并使用它们启动了一个本地 Kubernetes 集群，并简要地与之交互。然后，我们简要介绍了一些关键概念，我们将在未来的章节中更深入地使用和探索，包括容器、Pod、节点、部署和 ReplicaSet。

在下一章中，我们将深入探讨将软件放入容器所需的步骤，以及如何在自己的项目中设置容器的技巧。我们将以 Python 和 Node.js 为例，演示如何将软件放入容器，你可以将其作为自己代码的起点。
