# 第八章：抱歉，我的应用程序吃掉了集群

使用 Kubernetes 运行我们的应用程序可以使我们在集群中的机器上实现更高的资源利用率。Kubernetes 调度程序非常有效地将不同的应用程序打包到您的集群中，以最大程度地利用每台机器上的资源。您可以安排一些低优先级的作业，如果需要可以重新启动，例如批处理作业，以及高优先级的作业，例如 Web 服务器或数据库。Kubernetes 将帮助您利用空闲的 CPU 周期，这些周期发生在您的 Web 服务器等待请求时。

如果您想减少在 AWS 上支付的 EC2 实例的费用来运行您的应用程序，这是个好消息。学会如何配置您的 Pod 是很重要的，这样 Kubernetes 可以核算您的应用程序的资源使用情况。如果您没有正确配置您的 Pod，那么您的应用程序的可靠性和性能可能会受到影响，因为 Kubernetes 可能需要从节点中驱逐您的 Pod，因为资源不足。

在本章中，您将首先学习如何核算 Pod 将使用的内存和 CPU。我们将学习如何配置具有不同服务质量的 Pod，以便重要的工作负载能够保证它们所需的资源，但不太重要的工作负载可以在有空闲资源时使用，而无需专用资源。您还将学习如何利用 Kubernetes 自动缩放功能，在负载增加时向您的应用程序添加额外的 Pod，并在资源不足时向您的集群添加额外的节点。

在本章中，您将学习如何执行以下操作：

+   配置容器资源请求和限制

+   为所需的**服务质量**（**QoS**）类别配置您的 Pod

+   设置每个命名空间资源使用的配额

+   使用水平 Pod 自动缩放器自动调整您的应用程序，以满足对它们的需求

+   使用集群自动缩放器根据集群随时间变化的使用情况自动提供和终止 EC2 实例

# 资源请求和限制

Kubernetes 允许我们通过将多个不同的工作负载调度到一组机器中来实现集群的高利用率。每当我们要求 Kubernetes 调度一个 Pod 时，它需要考虑将其放置在哪个节点上。如果我们可以给调度器一些关于 Pod 所需资源的信息，它就可以更好地决定在哪个节点上放置 Pod。然后它可以计算每个节点上的当前工作负载，并选择符合我们 Pod 预期资源使用的节点。我们可以选择使用资源**请求**向 Kubernetes 提供这些信息。请求在将 Pod 调度到节点时考虑。请求不会对 Pod 在特定节点上运行时可能消耗的资源量提供任何限制，它们只是代表我们，集群操作员，在要求将特定 Pod 调度到集群时所做的请求的记录。

为了防止 Pod 使用超过其应该的资源，我们可以设置资源**限制**。这些限制可以由容器运行时强制执行，以确保 Pod 不会使用超过所需资源的数量。

我们可以说，容器的 CPU 使用是可压缩的，因为如果我们限制它，可能会导致我们的进程运行更慢，但通常不会造成其他不良影响，而容器的内存使用是不可压缩的，因为如果容器使用超过其内存限制，唯一的补救措施就是杀死相关的容器。

向 Pod 规范添加资源限制和请求的配置非常简单。在我们的清单中，每个容器规范都可以有一个包含请求和限制的`resources`字段。在这个例子中，我们请求分配 250 MiB 的 RAM 和四分之一的 CPU 核心给一个 Nginx web 服务器容器。因为限制设置得比请求高，这允许 Pod 使用高达半个 CPU 核心，并且只有在内存使用超过 128 Mi 时才会杀死容器：

```
apiVersion: v1 
kind: Pod 
metadata: 
  name: webserver 
spec: 
  containers: 
  - name: nginx 
    image: nginx 
    resources: 
      limits: 
        memory: 128Mi 
        cpu: 500m 
      requests:
```

```
        memory: 64Mi 
        cpu: 250m 
```

# 资源单位

每当我们指定 CPU 请求或限制时，我们都是以 CPU 核心为单位进行指定。因为我们经常希望请求或限制 Pod 使用整个 CPU 核心的一部分，我们可以将这部分 CPU 指定为小数或毫核值。例如，值为 0.5 表示半个核心。还可以使用毫核值配置请求或限制。由于 1,000 毫核等于一个核心，我们可以将一半 CPU 指定为 500 m。可以指定的最小 CPU 量为 1 m 或 0.001。

我发现在清单中使用毫核单位更易读。当使用`kubectl`或 Kubernetes 仪表板时，您还会注意到 CPU 限制和请求以毫核值格式化。但是，如果您正在使用自动化流程创建清单，您可能会使用浮点版本。

内存的限制和请求以字节为单位。但在清单中以这种方式指定它们会非常笨拙且难以阅读。因此，Kubernetes 支持用于引用字节的倍数的标准前缀；您可以选择使用十进制乘数，如 M 或 G，或者其中一个二进制等效项，如 Mi 或 Gi，后者更常用，因为它们反映了物理 RAM 的实际大小。

这些单位的二进制版本实际上是大多数人在谈论兆字节或千兆字节时真正意味着的，尽管更正确的说法是他们在谈论兆比字节和吉比字节！

在实践中，您应该始终记住使用末尾带有**i**的单位，否则您将得到比预期少一些的内存。这种表示法是在 1998 年引入 ISO/IEC 80000 标准中的，以避免十进制和二进制单位之间的混淆。

| **十进制** | **二进制** |
| --- | --- |
| **名称** | **字节** | **后缀** | **名称** | **字节** | **后缀** |
| 千字节 | 1000 | K | 基比字节 | 1024 | Ki |
| 兆字节 | 1000² | M | 兆比字节 | 1024² | Mi |
| 吉字节 | 1000³ | G | 吉比字节 | 1024³ | Gi |
| 太字节 | 1000⁴ | T | 泰比字节 | 1024⁴ | Ti |
| 皮字节 | 1000⁵ | P | 皮比字节 | 1024⁵ | Pi |
| 艾字节 | 1000⁶ | E | 艾比字节 | 1024⁶ | Ei |

Kubernetes 支持的内存单位

# 如何管理具有资源限制的 Pods

当 Kubelet 启动容器时，CPU 和内存限制将传递给容器运行时，然后容器运行时负责管理该容器的资源使用。

如果您正在使用 Docker，CPU 限制（以毫核为单位）将乘以 100，以确定容器每 100 毫秒可以使用的 CPU 时间。如果 CPU 负载过重，一旦容器使用完其配额，它将不得不等到下一个 100 毫秒周期才能继续使用 CPU。

在 cgroups 中运行的不同进程之间共享 CPU 资源的方法称为**完全公平调度器**或**CFS**；这通过在不同的 cgroups 之间分配 CPU 时间来实现。这通常意味着为一个 cgroup 分配一定数量的时间片。如果一个 cgroup 中的进程处于空闲状态，并且没有使用其分配的 CPU 时间，这些份额将可供其他 cgroup 中的进程使用。

这意味着一个 pod 即使限制设置得太低，可能仍然表现良好，但只有在另一个 pod 开始占用其公平份额的 CPU 后，它才可能突然停止。您可能会发现，如果您在空集群上开始为您的 pod 设置 CPU 限制，并添加额外的工作负载，您的 pod 的性能会开始受到影响。

在本章的后面，我们将讨论一些基本的工具，可以让我们了解每个 pod 使用了多少 CPU。

如果内存限制达到，容器运行时将终止容器（并可能重新启动）。如果容器使用的内存超过请求的数量，那么当节点开始内存不足时，它就成为被驱逐的候选者。

# 服务质量（QoS）

当 Kubernetes 创建一个 pod 时，它被分配为三个 QoS 类别之一。这些类别用于决定 Kubernetes 如何在节点上调度和驱逐 pod。广义上讲，具有保证 QoS 类别的 pod 将受到最少的驱逐干扰，而具有 BestEffort QoS 类别的 pod 最有可能受到干扰：

+   **保证**：这适用于高优先级的工作负载，这些工作负载受益于尽可能避免从节点中被驱逐，并且对于 CPU 资源具有比较低 QoS 类别的 pod 的优先级，容器运行时保证在需要时将提供指定限制中的全部 CPU 数量。

+   **可突发**: 这适用于较不重要的工作负载，例如可以在可用时利用更多 CPU 的后台作业，但只保证 CPU 请求中指定的级别。当节点资源不足时，可突发的 pod 更有可能被从节点中驱逐，特别是如果它们使用的内存超过了请求的数量。

+   **BestEffort**: 具有此类别的 pod 在节点资源不足时最有可能被驱逐。此 QoS 类别中的 pod 也只能使用节点上空闲的 CPU 和内存，因此如果节点上运行的其他 pod 正在大量使用 CPU，这些 pod 可能会完全被资源耗尽。如果您在此类别中调度 Pods，您应确保您的应用在资源匮乏和频繁重启时表现如预期。

实际上，最好避免使用具有 BestEffort QoS 类别的 pod，因为当集群负载过重时，这些 pod 将受到非常不寻常的行为影响。

当我们在 pod 的容器中设置资源和请求限制时，这些值的组合决定了 pod 所在的 QoS 类别。

要被赋予 BestEffort 的 QoS 类别，pod 中的任何容器都不应该设置任何 CPU 或内存请求或限制：

```
apiVersion: v1 
kind: Pod 
metadata: 
  name: best-effort 
spec: 
  containers: 
  - name: nginx 
    image: nginx 
```

一个没有资源限制或请求的 pod 将被分配 BestEffort 的 QoS 类别。

要被赋予保证的 QoS 类别，pod 需要在 pod 中的每个容器上都设置 CPU 和内存请求和限制。限制和请求必须相匹配。作为快捷方式，如果一个容器只设置了其限制，Kubernetes 会自动为资源请求分配相等的值：

```
apiVersion: v1 
kind: Pod 
metadata: 
  name: guaranteed 
spec: 
  containers: 
  - name: nginx 
    image: nginx 
    resources: 
      limits: 
        memory: 256Mi 
        cpu: 500m 
```

一个将被分配保证的 QoS 类别的 pod。

任何介于这两种情况之间的情况都将被赋予可突发的 QoS 类别。这适用于任何设置了任何 pod 的 CPU 或内存限制或请求的 pod。但是如果它们不符合保证类别的标准，例如没有在每个容器上设置限制，或者请求和限制不匹配：

```
apiVersion: v1 
kind: Pod 
metadata: 
  name: burstable 
spec: 
  containers: 
  - name: nginx 
    image: nginx 
    resources: 
      limits: 
        memory: 256Mi 
        cpu: 500m 
      requests: 
        memory: 128Mi 
        cpu: 250m 
```

一个将被分配可突发的 QoS 类别的 pod。

# 资源配额

资源配额允许您限制特定命名空间可以使用多少资源。根据您在组织中选择使用命名空间的方式，它们可以为您提供一种强大的方式，限制特定团队、应用程序或一组应用程序使用的资源，同时仍然让开发人员有自由调整每个单独容器的资源限制。

资源配额是一种有用的工具，当您想要控制不同团队或应用程序的资源成本，但仍希望实现将多个工作负载调度到同一集群的利用率时。

在 Kubernetes 中，资源配额由准入控制器管理。该控制器跟踪诸如 Pod 和服务之类的资源的使用，如果超出限制，它将阻止创建新资源。

资源配额准入控制器由在命名空间中创建的一个或多个 `ResourceQuota` 对象配置。这些对象通常由集群管理员创建，但您可以将创建它们整合到您组织中用于分配资源的更广泛流程中。

让我们看一个示例，说明配额如何限制集群中 CPU 资源的使用。由于配额将影响命名空间中的所有 Pod，因此我们将从使用 `kubectl` 创建一个新的命名空间开始：

```
$ kubectl create namespace quota-example
namespace/quota-example created  
```

我们将从创建一个简单的示例开始，确保每个新创建的 Pod 都设置了 CPU 限制，并且总限制不超过两个核心：

```
apiVersion: v1 
kind: ResourceQuota 
metadata: 
  name: resource-quota 
  namespace: quota-example 
spec: 
  hard: 
    limits.cpu: 2 
```

通过使用 `kubectl` 将清单提交到集群来创建 `ResourceQuota`。

一旦在命名空间中创建了指定资源请求或限制的 `ResourceQuota`，则在创建之前，所有 Pod 必须指定相应的请求或限制。

为了看到这种行为，让我们在我们的命名空间中创建一个示例部署：

```
apiVersion: apps/v1 
kind: Deployment 
metadata: 
  name: example 
  namespace: quota-example 
spec: 
  selector: 
    matchLabels: 
      app: example 
  template: 
    metadata: 
      labels: 
        app: example 
    spec: 
      containers: 
      - name: nginx 
        image: nginx 
        resources: 
          limits: 
            cpu: 500m 
```

一旦您使用 `kubectl` 将部署清单提交给 Kubernetes，请检查 Pod 是否正在运行：

```
$ kubectl -n quota-example get pods
NAME                      READY     STATUS    RESTARTS   AGE
example-fb556779d-4bzgd   1/1       Running   0          1m  
```

现在，扩展部署并观察是否创建了额外的 Pod：

```
$ kubectl -n quota-example scale deployment/example --replicas=4$ kubectl -n quota-example get pods
NAME                      READY     STATUS    RESTARTS   AGE
example-fb556779d-4bzgd   1/1       Running   0          2m
example-fb556779d-bpxm8   1/1       Running   0          1m
example-fb556779d-gkbvc   1/1       Running   0          1m
example-fb556779d-lcrg9   1/1       Running   0          1m  
```

因为我们指定了 `500m` 的 CPU 限制，所以将部署扩展到四个副本没有问题，这使用了我们在配额中指定的两个核心。

但是，如果您现在尝试扩展部署，使其使用的资源超出配额中指定的资源，您将发现 Kubernetes 不会安排额外的 Pod：

```
$ kubectl -n quota-example scale deployment/example --replicas=5  
```

运行`kubectl get events`将显示一个消息，其中调度程序未能创建满足副本计数所需的额外 pod：

```
$ kubectl -n quota-example get events
...
Error creating: pods "example-fb556779d-xmsgv" is forbidden: exceeded quota: resource-quota, requested: limits.cpu=500m, used: limits.cpu=2, limited: limits.cpu=2  
```

# 默认限制

当您在命名空间上使用配额时，一个要求是命名空间中的每个容器必须定义资源限制和请求。有时，这个要求可能会导致复杂性，并使在 Kubernetes 上快速工作变得更加困难。正确指定资源限制是准备应用程序投入生产的重要部分，但是，例如，在使用 Kubernetes 作为开发或测试工作负载的平台时，这确实增加了额外的开销。

Kubernetes 提供了在命名空间级别提供默认请求和限制的功能。您可以使用这个功能为特定应用程序或团队使用的命名空间提供一些合理的默认值。

我们可以使用`LimitRange`对象为命名空间中的容器配置默认限制和请求。这个对象允许我们为 CPU 或内存，或两者都提供默认值。如果一个命名空间中存在一个`LimitRange`对象，那么任何没有在`LimitRange`中配置资源请求或限制的容器将从限制范围中继承这些值。

有两种情况下，当创建一个 pod 时，`LimitRange`会影响资源限制或请求：

+   没有资源限制或请求的容器将从`LimitRange`对象继承资源限制和请求

+   没有资源限制但有指定请求的容器将从`LimitRange`对象中继承资源限制

如果容器已经定义了限制和请求，那么`LimitRange`将不起作用。因为只指定了限制的容器会将请求字段默认为相同的值，它们不会从`LimitRange`继承请求值。让我们看一个快速示例。我们首先创建一个新的命名空间：

```
$ kubectl create namespace limit-example
namespace/limit-example created  
```

创建限制范围对象的清单，并使用`kubectl`将其提交到集群中：

```
apiVersion: v1 
kind: LimitRange 
metadata: 
  name: example 
  namespace: limit-example 
spec: 
  limits: 
  - default: 
      memory: 512Mi 
      cpu: 1 
    defaultRequest: 
      memory: 256Mi 
      cpu: 500m 
    type: Container 
```

如果您在此命名空间中创建一个没有资源限制的 pod，它将在创建时从限制范围对象中继承：

```
$ kubectl -n limit-example run --image=nginx example  
```

`deployment.apps/`示例已创建。

您可以通过运行`kubectl describe`来检查限制：

```
$ kubectl -n limit-example describe pods 
... 
    Limits: 
      cpu:     1 
      memory:  512Mi 
    Requests: 
      cpu:        500m 
      memory:     256Mi 
... 
```

# 水平 Pod 自动缩放

一些应用程序可以通过添加额外的副本来扩展以处理增加的负载。无状态的 Web 应用程序就是一个很好的例子，因为添加额外的副本提供了处理对应用程序的增加请求所需的额外容量。一些其他应用程序也设计成可以通过添加额外的 pod 来处理增加的负载；许多围绕从中央队列处理消息的系统也可以以这种方式处理增加的负载。

当我们使用 Kubernetes 部署来部署我们的 pod 工作负载时，使用 `kubectl scale` 命令简单地扩展应用程序使用的副本数量。然而，如果我们希望我们的应用程序自动响应其工作负载的变化并根据需求进行扩展，那么 Kubernetes 为我们提供了水平 Pod 自动缩放。

水平 Pod 自动缩放允许我们定义规则，根据 CPU 利用率和其他自定义指标，在我们的部署中扩展或缩减副本的数量。在我们的集群中使用水平 Pod 自动缩放之前，我们需要部署 Kubernetes 度量服务器；该服务器提供了用于发现应用程序生成的 CPU 利用率和其他指标的端点。

# 部署度量服务器

在我们可以使用水平 Pod 自动缩放之前，我们需要将 Kubernetes 度量服务器部署到我们的集群中。这是因为水平 Pod 自动缩放控制器使用 `metrics.k8s.io` API 提供的指标，而这些指标是由度量服务器提供的。

虽然一些 Kubernetes 的安装可能默认安装此附加组件，在我们的 EKS 集群中，我们需要自己部署它。

有许多方法可以部署附加组件到您的集群中：

+   如果您遵循了第七章中的建议，*一个生产就绪的集群*，并且正在使用 Terraform 为您的集群进行配置，您可以像我们在第七章中配置 kube2iam 时一样，使用 `kubectl` 部署所需的清单。

+   如果您正在使用 helm 管理集群上的应用程序，您可以使用 stable/metrics server 图表。

+   在这一章中，为了简单起见，我们将使用 `kubectl` 部署度量服务器清单。

+   我喜欢将部署诸如指标服务器和 kube2iam 等附加组件集成到配置集群的过程中，因为我认为它们是集群基础设施的组成部分。但是，如果您要使用类似 helm 的工具来管理在集群上运行的应用程序的部署，那么您可能更喜欢使用相同的工具来管理在集群上运行的所有内容。您所做的决定取决于您和您的团队为管理集群及其上运行的应用程序采用的流程。

+   指标服务器是在 GitHub 存储库中开发的，网址为[`github.com/kubernetes-incubator/metrics-server`](https://github.com/kubernetes-incubator/metrics-server)。您将在该存储库的 deploy 目录中找到部署所需的清单。

首先从 GitHub 克隆配置。指标服务器从版本 0.0.3 开始支持 EKS 提供的身份验证方法，因此请确保您使用的清单至少使用该版本。

您将在`deploy/1.8+`目录中找到许多清单。`auth-reader.yaml`和`auth-delegator.yaml`文件配置了指标服务器与 Kubernetes 授权基础设施的集成。`resource-reader.yaml`文件配置了一个角色，以赋予指标服务器从 API 服务器读取资源的权限，以便发现 Pod 所在的节点。基本上，`metrics-server-deployment.yaml`和`metrics-server-service.yaml`定义了用于运行服务本身的部署，以及用于访问该服务的服务。最后，`metrics-apiservice.yaml`文件定义了一个`APIService`资源，将 metrics.k8s.io API 组注册到 Kubernetes API 服务器聚合层；这意味着对于 metrics.k8s.io 组的 API 服务器请求将被代理到指标服务器服务。

使用`kubectl`部署这些清单很简单，只需使用`kubectl apply`将所有清单提交到集群：

```
$ kubectl apply -f deploy/1.8+  
```

您应该看到关于在集群上创建的每个资源的消息。

如果您使用类似 Terraform 的工具来配置集群，您可以在创建集群时使用它来提交指标服务器的清单。

# 验证指标服务器和故障排除

在我们继续之前，我们应该花一点时间检查我们的集群和指标服务器是否正确配置以共同工作。

在度量服务器在您的集群上运行并有机会从集群中收集度量数据（给它一分钟左右的时间）之后，您应该能够使用`kubectl top`命令来查看集群中 pod 和节点的资源使用情况。

首先运行`kubectl top nodes`。如果您看到像这样的输出，那么度量服务器已经正确配置，并且正在从您的节点收集度量数据：

```
$ kubectl top nodes
NAME             CPU(cores)   CPU%      MEMORY(bytes)   MEMORY%
ip-10-3-29-209   20m          1%        717Mi           19%
ip-10-3-61-119   24m          1%        1011Mi          28%  
```

如果您看到错误消息，那么有一些故障排除步骤可以遵循。

您应该从描述度量服务器部署并检查一个副本是否可用开始：

```
kubectl -n kube-system describe deployment metrics-server  
```

如果没有配置正确，您应该通过运行`kubectl -n kube-system describe pod`来调试创建的 pod。查看事件，看看服务器为什么不可用。确保您至少运行版本 0.0.3 的度量服务器，因为之前的版本不支持与 EKS API 服务器进行身份验证。

如果度量服务器正在正确运行，但在运行`kubectl top`时仍然看到错误，问题可能是聚合层注册的 APIservice 没有正确配置。在运行`kubectl describe apiservice v1beta1.metrics.k8s.io`时，检查底部返回的信息中的事件输出。

一个常见的问题是，EKS 控制平面无法连接到端口`443`上的度量服务器服务。如果您遵循了第七章中的说明，*一个生产就绪的集群*，您应该已经有一个安全组规则允许控制平面到工作节点的流量，但一些其他文档可能建议更严格的规则，这可能不允许端口`443`上的流量。

# 根据 CPU 使用率自动扩展 pod

一旦度量服务器安装到我们的集群中，我们将能够使用度量 API 来检索有关集群中 pod 和节点的 CPU 和内存使用情况的信息。使用`kubectl top`命令就是一个简单的例子。

水平 Pod 自动缩放器也可以使用相同的度量 API 来收集组成部署的 pod 的当前资源使用情况的信息。

让我们来看一个例子；我们将部署一个使用大量 CPU 的示例应用程序，然后配置一个水平 Pod 自动缩放器，在 CPU 利用率超过目标水平时扩展该 pod 的额外副本，以提供额外的容量。

我们将部署的示例应用程序是一个简单的 Ruby Web 应用程序，可以计算斐波那契数列中的第 n 个数字，该应用程序使用简单的递归算法，效率不是很高（非常适合我们进行自动缩放实验）。该应用程序的部署非常简单。设置 CPU 资源限制非常重要，因为目标 CPU 利用率是基于此限制的百分比：

```
deployment.yaml 
apiVersion: apps/v1 
kind: Deployment 
metadata: 
  name: fib 
  labels: 
    app: fib 
spec: 
  selector: 
    matchLabels: 
      app: fib 
  template: 
    metadata: 
      labels: 
        app: fib 
    spec: 
      containers: 
      - name: fib 
        image: errm/fib 
        ports: 
        - containerPort: 9292 
        resources: 
          limits: 
            cpu: 250m 
            memory: 32Mi 
```

我们没有在部署规范中指定副本的数量；因此，当我们首次将此部署提交到集群时，副本的数量将默认为 1。这是创建部署的良好实践，我们打算通过 Horizontal Pod Autoscaler 调整副本的数量，因为这意味着如果我们稍后使用`kubectl apply`来更新部署，我们不会覆盖水平 Pod Autoscaler 设置的副本值（无意中缩减或增加部署）。

让我们将这个应用程序部署到集群中：

```
kubectl apply -f deployment.yaml  
```

您可以运行`kubectl get pods -l app=fib`来检查应用程序是否正确启动。

我们将创建一个服务，以便能够访问部署中的 Pod，请求将被代理到每个副本，分散负载：

```
service.yaml 
kind: Service 
apiVersion: v1 
metadata: 
  name: fib 
spec: 
  selector: 
    app: fib 
  ports: 
  - protocol: TCP 
    port: 80 
    targetPort: 9292 
```

使用`kubectl`将服务清单提交到集群：

```
kubectl apply -f service.yaml  
```

我们将配置一个 Horizontal Pod Autoscaler 来控制部署中副本的数量。`spec`定义了我们希望自动缩放器的行为；我们在这里定义了我们希望自动缩放器维护应用程序的 1 到 10 个副本，并实现 60%的目标平均 CPU 利用率。

当 CPU 利用率低于 60%时，自动缩放器将调整目标部署的副本计数；当超过 60%时，将添加副本：

```
hpa.yaml 
kind: HorizontalPodAutoscaler 
apiVersion: autoscaling/v2beta1 
metadata: 
  name: fib 
spec: 
  maxReplicas: 10 
  minReplicas: 1 
  scaleTargetRef: 
    apiVersion: app/v1 
    kind: Deployment 
    name: fib 
  metrics: 
  - type: Resource 
    resource: 
      name: cpu 
      targetAverageUtilization: 60 
```

使用`kubectl`创建自动缩放器：

```
kubectl apply -f hpa.yaml  
```

`kubectl autoscale`命令是创建`HorizontalPodAutoscaler`的快捷方式。运行`kubectl autoscale deployment fib --min=1 --max=10 --cpu-percent=60`将创建一个等效的自动缩放器。

创建了 Horizontal Pod Autoscaler 后，您可以使用`kubectl describe`查看有关其当前状态的许多有趣信息：

```
$ kubectl describe hpa fib    
Name:              fib
Namespace:         default
CreationTimestamp: Sat, 15 Sep 2018 14:32:46 +0100
Reference:         Deployment/fib
Metrics:           ( current / target )
  resource cpu:    0% (1m) / 60%
Min replicas:      1
Max replicas:      10
Deployment pods:   1 current / 1 desired  
```

现在我们已经设置好了 Horizontal Pod Autoscaler，我们应该在部署中的 Pod 上生成一些负载，以说明它的工作原理。在这种情况下，我们将使用`ab`（Apache benchmark）工具重复要求我们的应用程序计算第 30 个斐波那契数：

```
load.yaml
apiVersion: batch/v1 
kind: Job 
metadata: 
  name: fib-load 
  labels: 
    app: fib 
    component: load 
spec: 
  template: 
    spec: 
      containers: 
      - name: fib-load 
        image: errm/ab 
        args: ["-n1000", "-c4", "fib/30"] 
      restartPolicy: OnFailure 
```

此作业使用`ab`向端点发出 1,000 个请求（并发数为 4）。将作业提交到集群，然后观察水平 Pod 自动缩放器的状态：

```
kubectl apply -f load.yaml    
watch kubectl describe hpa fib
```

一旦负载作业开始发出请求，自动缩放器将扩展部署以处理负载：

```
Name:                   fib
Namespace:              default
CreationTimestamp: Sat, 15 Sep 2018 14:32:46 +0100
Reference:         Deployment/fib
Metrics:           ( current / target )
  resource cpu:    100% (251m) / 60%
Min replicas:      1
Max replicas:      10
Deployment pods:   2 current / 2 desired  
```

# 基于其他指标自动缩放 pod

度量服务器提供了水平 Pod 自动缩放器可以使用的 API，以获取有关集群中 pod 的 CPU 和内存利用率的信息。

可以针对利用率百分比进行目标设置，就像我们对 CPU 指标所做的那样，或者可以针对绝对值进行目标设置，就像我们对内存指标所做的那样：

```
hpa.yaml 
kind: HorizontalPodAutoscaler 
apiVersion: autoscaling/v2beta1 
metadata: 
  name: fib 
spec: 
  maxReplicas: 10 
  minReplicas: 1 
  scaleTargetRef: 
    apiVersion: app/v1 
    kind: Deployment 
    name: fib 
  metrics: 
  - type: Resource 
    resource: 
      name: memory 
      targetAverageValue: 20M 
```

水平 Pod 自动缩放器还允许我们根据更全面的指标系统提供的其他指标进行缩放。Kubernetes 允许聚合自定义和外部指标的指标 API。

自定义指标是与 pod 相关的除 CPU 和内存之外的指标。例如，您可以使用适配器，使您能够使用像 Prometheus 这样的系统从您的 pod 中收集的指标。

如果您有关于应用程序利用率的更详细的指标可用，这可能非常有益，例如，一个公开忙碌工作进程计数的分叉 Web 服务器，或者一个公开有关当前排队项目数量的指标的队列处理应用程序。

外部指标适配器提供有关与 Kubernetes 中的任何对象不相关的资源的信息，例如，如果您正在使用外部排队系统，比如 AWS SQS 服务。

总的来说，如果您的应用程序可以公开有关其所依赖的资源的指标，并使用外部指标适配器，那将更简单，因为很难限制对特定指标的访问，而自定义指标与特定的 Pod 相关联，因此 Kubernetes 可以限制只有那些需要使用它们的用户和进程才能访问它们。

# 集群自动缩放

Kubernetes Horizontal Pod Autoscaler 的功能使我们能够根据应用程序随时间变化的资源使用情况添加和删除 pod 副本。然而，这对我们集群的容量没有影响。如果我们的 pod 自动缩放器正在添加 pod 来处理负载增加，那么最终我们的集群可能会用尽空间，额外的 pod 将无法被调度。如果我们的应用程序负载减少，pod 自动缩放器删除 pod，那么我们就需要为 EC2 实例支付费用，而这些实例将处于空闲状态。

当我们在第七章中创建了我们的集群，*一个生产就绪的集群*，我们使用自动缩放组部署了集群节点，因此我们应该能够利用它根据部署到集群上的应用程序的需求随时间变化而扩展和收缩集群。

自动缩放组内置支持根据实例的平均 CPU 利用率来调整集群的大小。然而，当处理 Kubernetes 集群时，这并不是很合适，因为运行在集群每个节点上的工作负载可能会有很大不同，因此平均 CPU 利用率并不是集群空闲容量的很好代理。

值得庆幸的是，为了有效地将 pod 调度到节点上，Kubernetes 会跟踪每个节点的容量和每个 pod 请求的资源。通过利用这些信息，我们可以自动调整集群的大小以匹配工作负载的大小。

Kubernetes 自动缩放器项目为一些主要的云提供商提供了集群自动缩放器组件，包括 AWS。集群自动缩放器可以很简单地部署到我们的集群。除了能够向我们的集群添加实例外，集群自动缩放器还能够从集群中清除 pod，然后在集群容量可以减少时终止实例。

# 部署集群自动缩放器

将集群自动缩放器部署到我们的集群非常简单，因为它只需要一个简单的 pod 在运行。我们只需要一个简单的 Kubernetes 部署，就像我们在之前的章节中使用过的那样。

为了让集群自动缩放器更新自动缩放组的期望容量，我们需要通过 IAM 角色授予权限。如果您正在使用 kube2iam，正如我们在第七章中讨论的那样，我们将能够通过适当的注释为集群自动缩放器 pod 指定这个角色：

```
cluster_autoscaler.tf
data "aws_iam_policy_document" "eks_node_assume_role_policy" { 
  statement { 
    actions = ["sts:AssumeRole"] 
    principals { 
      type = "AWS" 
      identifiers = ["${aws_iam_role.node.arn}"] 
    } 
  } 
} 

resource "aws_iam_role" "cluster-autoscaler" { 
  name = "EKSClusterAutoscaler" 
  assume_role_policy = "${data.aws_iam_policy_document.eks_node_assume_role_policy.json}" 
} 

data "aws_iam_policy_document" "autoscaler" { 
  statement { 
    actions = [ 
      "autoscaling:DescribeAutoScalingGroups", 
      "autoscaling:DescribeAutoScalingInstances", 
      "autoscaling:DescribeTags", 
      "autoscaling:SetDesiredCapacity", 
      "autoscaling:TerminateInstanceInAutoScalingGroup" 
    ] 
    resources = ["*"] 
  } 
} 

resource "aws_iam_role_policy" "cluster_autoscaler" { 
  name = "cluster-autoscaler" 
  role = "${aws_iam_role.cluster_autoscaler.id}" 
  policy = "${data.aws_iam_policy_document.autoscaler.json}" 
} 
```

为了将集群自动缩放器部署到我们的集群，我们将使用`kubectl`提交一个部署清单，类似于我们在第七章中部署 kube2iam 的方式，*一个生产就绪的集群*。我们将使用 Terraform 的模板系统来生成清单。

我们创建一个服务账户，用于自动扩展器连接到 Kubernetes API：

```
cluster_autoscaler.tpl
--- 
apiVersion: v1 
kind: ServiceAccount 
metadata: 
  labels: 
    k8s-addon: cluster-autoscaler.addons.k8s.io 
    k8s-app: cluster-autoscaler 
  name: cluster-autoscaler 
  namespace: kube-system 
```

集群自动缩放器需要读取有关集群当前资源使用情况的信息，并且需要能够从需要从集群中移除并终止的节点中驱逐 Pod。基本上，`cluster-autoscalerClusterRole`为这些操作提供了所需的权限。以下是`cluster_autoscaler.tpl`的代码续写：

```
--- 
apiVersion: rbac.authorization.k8s.io/v1beta1 
kind: ClusterRole 
metadata: 
  name: cluster-autoscaler 
  labels: 
    k8s-addon: cluster-autoscaler.addons.k8s.io 
    k8s-app: cluster-autoscaler 
rules: 
- apiGroups: [""] 
  resources: ["events","endpoints"] 
  verbs: ["create", "patch"] 
- apiGroups: [""] 
  resources: ["pods/eviction"] 
  verbs: ["create"] 
- apiGroups: [""] 
  resources: ["pods/status"] 
  verbs: ["update"] 
- apiGroups: [""] 
  resources: ["endpoints"] 
  resourceNames: ["cluster-autoscaler"] 
  verbs: ["get","update"] 
- apiGroups: [""] 
  resources: ["nodes"] 
  verbs: ["watch","list","get","update"] 
- apiGroups: [""] 
  resources: ["pods","services","replicationcontrollers","persistentvolumeclaims","persistentvolumes"] 
  verbs: ["watch","list","get"] 
- apiGroups: ["extensions"] 
  resources: ["replicasets","daemonsets"] 
  verbs: ["watch","list","get"] 
- apiGroups: ["policy"] 
  resources: ["poddisruptionbudgets"] 
  verbs: ["watch","list"] 
- apiGroups: ["apps"] 
  resources: ["statefulsets"] 
  verbs: ["watch","list","get"] 
- apiGroups: ["storage.k8s.io"] 
  resources: ["storageclasses"] 
  verbs: ["watch","list","get"] 
--- 
apiVersion: rbac.authorization.k8s.io/v1beta1 
kind: ClusterRoleBinding 
metadata: 
  name: cluster-autoscaler 
  labels: 
    k8s-addon: cluster-autoscaler.addons.k8s.io 
    k8s-app: cluster-autoscaler 
roleRef: 
  apiGroup: rbac.authorization.k8s.io 
  kind: ClusterRole 
  name: cluster-autoscaler 
subjects: 
  - kind: ServiceAccount 
    name: cluster-autoscaler 
    namespace: kube-system 
```

请注意，`cluster-autoscaler`在配置映射中存储状态信息，因此需要有权限能够从中读取和写入。这个角色允许了这一点。以下是`cluster_autoscaler.tpl`的代码续写：

```
--- 
apiVersion: rbac.authorization.k8s.io/v1beta1 
kind: Role 
metadata: 
  name: cluster-autoscaler 
  namespace: kube-system 
  labels: 
    k8s-addon: cluster-autoscaler.addons.k8s.io 
    k8s-app: cluster-autoscaler 
rules: 
- apiGroups: [""] 
  resources: ["configmaps"] 
  verbs: ["create"] 
- apiGroups: [""] 
  resources: ["configmaps"] 
  resourceNames: ["cluster-autoscaler-status"] 
  verbs: ["delete","get","update"] 
--- 
apiVersion: rbac.authorization.k8s.io/v1beta1 
kind: RoleBinding 
metadata: 
  name: cluster-autoscaler 
  namespace: kube-system 
  labels: 
    k8s-addon: cluster-autoscaler.addons.k8s.io 
    k8s-app: cluster-autoscaler 
roleRef: 
  apiGroup: rbac.authorization.k8s.io 
  kind: Role 
  name: cluster-autoscaler 
subjects: 
  - kind: ServiceAccount 
    name: cluster-autoscaler 
    namespace: kube-system 
```

最后，让我们考虑一下集群自动缩放器部署本身的清单。集群自动缩放器 Pod 包含一个运行集群自动缩放器控制循环的单个容器。您会注意到我们正在向集群自动缩放器传递一些配置作为命令行参数。最重要的是，`--node-group-auto-discovery`标志允许自动缩放器在具有我们在第七章中创建集群时在我们的自动缩放组上设置的`kubernetes.io/cluster/<cluster_name>`标记的自动缩放组上操作。这很方便，因为我们不必显式地配置自动缩放器与我们的集群自动缩放组。

如果您的 Kubernetes 集群在多个可用区中有节点，并且正在运行依赖于被调度到特定区域的 Pod（例如，正在使用 EBS 卷的 Pod），建议为您计划使用的每个可用区创建一个自动缩放组。如果您使用跨多个区域的一个自动缩放组，那么集群自动缩放器将无法指定它启动的实例的可用区。

这是`cluster_autoscaler.tpl`的代码续写：

```
--- 
apiVersion: extensions/v1beta1 
kind: Deployment 
metadata: 
  name: cluster-autoscaler 
  namespace: kube-system 
  labels: 
    app: cluster-autoscaler 
spec: 
  replicas: 1 
  selector: 
    matchLabels: 
      app: cluster-autoscaler 
  template: 
    metadata: 
      annotations: 
        iam.amazonaws.com/role: ${iam_role} 
      labels: 
        app: cluster-autoscaler 
    spec: 
      serviceAccountName: cluster-autoscaler 
      containers: 
        - image: k8s.gcr.io/cluster-autoscaler:v1.3.3 
          name: cluster-autoscaler 
          resources: 
            limits: 
              cpu: 100m 
              memory: 300Mi 
            requests: 
              cpu: 100m 
              memory: 300Mi 
          command: 
            - ./cluster-autoscaler 
            - --v=4 
            - --stderrthreshold=info 
            - --cloud-provider=aws 
            - --skip-nodes-with-local-storage=false 
            - --expander=least-waste 
            - --node-group-auto-discovery=asg:tag=kubernetes.io/cluster/${cluster_name} 
          env: 
            - name: AWS_REGION 
              value: ${aws_region} 
          volumeMounts: 
            - name: ssl-certs 
              mountPath: /etc/ssl/certs/ca-certificates.crt 
              readOnly: true 
          imagePullPolicy: "Always" 
      volumes: 
        - name: ssl-certs 
          hostPath: 
            path: "/etc/ssl/certs/ca-certificates.crt" 
```

最后，我们通过传递 AWS 区域、集群名称和 IAM 角色的变量来渲染模板化的清单，并使用`kubectl`将文件提交给 Kubernetes：

这是`cluster_autoscaler.tpl`的代码续写：

```
data "aws_region" "current" {} 

data "template_file" " cluster_autoscaler " { 
  template = "${file("${path.module}/cluster_autoscaler.tpl")}" 

  vars { 
    aws_region = "${data.aws_region.current.name}" 
    cluster_name = "${aws_eks_cluster.control_plane.name}" 
    iam_role = "${aws_iam_role.cluster_autoscaler.name}" 
  } 
} 

resource "null_resource" "cluster_autoscaler" { 
  trigers = { 
    manifest_sha1 = "${sha1("${data.template_file.cluster_autoscaler.rendered}")}" 
  } 

  provisioner "local-exec" { 
    command = "kubectl  
--kubeconfig=${local_file.kubeconfig.filename} apply -f -<<EOF\n${data.template_file.cluster_autoscaler.rendered}\nEOF" 
  } 
} 
```

# 总结

Kubernetes 是一个强大的工具；它非常有效地实现了比手动将应用程序调度到机器上更高的计算资源利用率。重要的是，您要学会通过设置正确的资源限制和请求来为您的 Pod 分配资源；如果不这样做，您的应用程序可能会变得不可靠或者资源匮乏。

通过了解 Kubernetes 如何根据您分配给它们的资源请求和限制为您的 pod 分配服务质量类别，您可以精确控制您的 pod 的管理方式。通过确保您的关键应用程序（如 Web 服务器和数据库）以保证类别运行，您可以确保它们将始终表现一致，并在需要重新安排 pod 时遭受最小的中断。您可以通过为较低优先级的 pod 设置限制来提高集群的效率，这将导致它们以可突发的 QoS 类别进行安排。可突发的 pod 可以在有空闲资源时使用额外的资源，但在负载增加时不需要向集群添加额外的容量。

资源配额在管理大型集群时非常有价值，该集群用于运行多个应用程序，甚至由组织中的不同团队使用，特别是如果您试图控制非生产工作负载（如测试和分段环境）的成本。

AWS 之所以称其机器为弹性，是有原因的：它们可以在几分钟内按需扩展或缩减，以满足应用程序的需求。如果您在负载变化的集群上运行工作负载，那么您应该利用这些特性和 Kubernetes 提供的工具来扩展部署，以匹配应用程序接收的负载以及需要安排的 pod 的大小。
