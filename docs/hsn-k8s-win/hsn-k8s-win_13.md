# 部署您的第一个应用程序

在前几章中，我们介绍了 Kubernetes 的关键操作原则和 Windows/Linux 混合集群的部署策略。现在是时候更专注于部署和使用 Kubernetes 应用程序了。为了演示 Kubernetes 应用程序的基本操作，我们将使用在第八章中创建的 AKS Engine 混合 Kubernetes 集群，*部署混合 Azure Kubernetes 服务引擎集群*。您也可以使用本地混合集群，但您应该期望功能有限；例如，LoadBalancer 类型的服务将不可用。

本章涵盖以下主题：

+   命令式部署应用程序

+   使用 Kubernetes 清单文件

+   在 Windows 节点上调度 Pods

+   访问您的应用程序

+   扩展应用程序

# 技术要求

本章，您将需要以下内容：

+   已安装 Windows 10 Pro、企业版或教育版（1903 版本或更高版本，64 位）

+   Azure 帐户

+   使用 AKS Engine 部署的 Windows/Linux Kubernetes 集群

要跟着做，您需要自己的 Azure 帐户来为 Kubernetes 集群创建 Azure 资源。如果您之前还没有为前几章创建帐户，您可以在这里阅读更多关于如何获得个人使用的有限免费帐户：[`azure.microsoft.com/en-us/free/`](https://azure.microsoft.com/en-us/free/)。

使用 AKS Engine 部署 Kubernetes 集群已在第八章中介绍过，*部署混合 Azure Kubernetes 服务引擎集群*。

您可以从官方 GitHub 存储库下载本章的最新代码示例：[`github.com/PacktPublishing/Hands-On-Kubernetes-on-Windows/tree/master/Chapter09`](https://github.com/PacktPublishing/Hands-On-Kubernetes-on-Windows/tree/master/Chapter09)。

# 命令式部署应用程序

在 Kubernetes 世界中，管理应用程序时可以选择两种方法：命令式管理和声明式管理。命令式方法包括执行命令式 kubectl 命令，例如`kubectl run`或`kubectl expose`，以及命令式对象配置管理，其中您使用命令，如`kubectl create`或`kubectl replace`。简而言之，您通过执行临时命令来管理集群，这些命令修改 Kubernetes 对象并导致集群的期望状态发生变化 - 有时，您甚至可能不知道命令式命令之后期望状态的确切变化。相比之下，在声明式方法中，您修改对象配置（清单文件），并使用`kubectl apply`命令在集群中创建或更新它们（或者，您可以使用 Kustomization 文件）。

使用声明性管理通常更接近 Kubernetes 的精神 - 整个架构都专注于保持期望的集群状态，并不断执行操作，将当前集群状态更改为期望的状态。一个经验法则是，在生产环境中，您应该始终使用声明性管理，无论是使用标准清单文件还是 Kustomization 文件。您可以轻松为对象配置提供源代码控制，并将其集成到持续集成/部署流水线中。命令式管理对于开发和概念验证场景非常有用 - 操作直接在活动集群上执行。

请记住，对于这种方法，您将无法轻松地了解先前配置的历史！

现在，让我们首先尝试使用命令式方法部署一个简单的 Web 应用程序。我们将执行以下操作：

1.  创建一个单独的裸 Pod 或 ReplicationController。

1.  使用 Service（LoadBalancer 类型）来公开它。

要命令式地创建一个 pod 或 ReplicationController，我们将使用`kubectl run`命令。此命令允许您使用生成器创建不同的容器管理对象。您可以在官方文档中找到生成器的完整列表：[`kubernetes.io/docs/reference/kubectl/conventions/#generators`](https://kubernetes.io/docs/reference/kubectl/conventions/#generators)——自 Kubernetes 1.12 以来，除了`run-pod/v1`之外的所有生成器都已被弃用。这样做的主要原因是`kubectl run`命令的相对复杂性，以及鼓励在高级场景中采用适当的声明性方法。

要部署基于`mcr.microsoft.com/dotnet/core/samples:aspnetapp` Docker 映像的示例应用程序，请执行以下步骤：

1.  打开 PowerShell 窗口，并确保您正在使用`kubeconfig`文件，该文件允许您连接到您的 AKS Engine 混合集群。

1.  确定集群中的节点上可用的 Windows Server 操作系统的版本。例如，对于 Windows Server 2019 Datacenter 节点，您需要使用具有基础层版本 1809 的容器映像。这意味着在我们的示例中，我们必须使用`mcr.microsoft.com/dotnet/core/samples:aspnetapp-nanoserver-1809` Docker 映像：

```
kubectl get nodes -o wide
```

1.  使用`run-pod/v1`生成器执行`kubectl run`命令以运行单个 pod，`windows-example`，用于具有节点选择器和操作系统类型以及`windows`的示例应用程序：

```
kubectl run `
 --generator=run-pod/v1 `
 --image=mcr.microsoft.com/dotnet/core/samples:aspnetapp-nanoserver-1809 `
 --overrides='{\"apiVersion\": \"v1\", \"spec\": {\"nodeSelector\": { \"beta.kubernetes.io/os\": \"windows\" }}}' `
 windows-example
```

1.  pod 将被调度到 Windows 节点之一，并且您可以使用以下命令监视 pod 创建的进度：

```
PS C:\src> kubectl get pods -w
NAME              READY   STATUS              RESTARTS   AGE
windows-example   0/1     ContainerCreating   0          7s
```

1.  当 pod 将其状态更改为`Running`时，您可以继续使用 LoadBalancer 服务暴露 pod：

```
kubectl expose pod windows-example `
 --name windows-example-service `
 --type LoadBalancer `
 --port 8080 `
 --target-port 80
```

1.  等待服务的`EXTERNAL-IP`可用：

```
PS C:\src> kubectl get service -w
NAME                      TYPE           CLUSTER-IP     EXTERNAL-IP      PORT(S)          AGE
kubernetes                ClusterIP      10.0.0.1       <none>           443/TCP          24h
windows-example-service   LoadBalancer   10.0.192.180   213.199.135.14   8080:30746/TCP   5m10s
```

1.  现在，您可以使用服务的外部 IP 和端口`8080`来访问在 pod 中运行的应用程序。例如，在 Web 浏览器中，导航到`http://213.199.135.14:8080/`。

或者，前面的步骤可以在一个`kubectl run`命令中完成，该命令将创建 pod 并立即使用 LoadBalancer 服务进行暴露：

```
kubectl run `
 --generator=run-pod/v1 `
 --image=mcr.microsoft.com/dotnet/core/samples:aspnetapp-nanoserver-1809 `
 --overrides='{\"apiVersion\": \"v1\", \"spec\": {\"nodeSelector\": { \"beta.kubernetes.io/os\": \"windows\" }}}' `
 --expose `
 --port 80 `
 --service-overrides='{ \"spec\": { \"type\": \"LoadBalancer\" }}' `
 windows-example
```

请注意，此命令使用端口`80`而不是`8080`来暴露服务。使用服务端口`80`和目标端口`8080`需要在`--service-overrides`标志中增加另一层复杂性。

为了完整起见，让我们在 Kubernetes ReplicationController 对象后面运行我们的`mcr.microsoft.com/dotnet/core/samples:aspnetapp-nanoserver-1809`容器。您可以在第四章中了解有关 ReplicationControllers、ReplicaSets 和 Deployments 的更多信息，*Kubernetes 概念和 Windows 支持*——通常情况下，在集群中运行裸 Pods 并不明智；您应该始终使用至少 ReplicaSets 或更好地使用 Deployments 来管理 Pods。在 Kubernetes 1.17 中，仍然可以使用`kubectl run`创建 ReplicationController——生成器已被弃用，但尚未删除。使用命令来声明式地创建 ReplicationController 需要使用不同的`--generator`标志，其值为`run/v1`：

```
kubectl run `
 --generator=run/v1  `
 --image=mcr.microsoft.com/dotnet/core/samples:aspnetapp-nanoserver-1809 `
 --overrides='{\"apiVersion\": \"v1\", \"spec\": {\"nodeSelector\": { \"beta.kubernetes.io/os\": \"windows\" }}}' `
 --expose `
 --port 80 `
 --service-overrides='{ \"spec\": { \"type\": \"LoadBalancer\" }}' `
 windows-example
```

即使这种方法快捷且不需要任何配置文件，你可以清楚地看到，除了简单操作之外，使用`kubectl run`变得复杂且容易出错。在大多数情况下，您将使用命令来执行以下操作：

+   在开发集群中快速创建 Pods

+   为了调试目的创建临时交互式 Pods

+   可预测地删除 Kubernetes 资源——在下一节中会详细介绍

现在让我们通过使用 Kubernetes 清单文件和声明式管理方法来执行类似的部署。

# 使用 Kubernetes 清单文件

Kubernetes 对象的声明式管理更接近于 Kubernetes 的精神——您专注于告诉 Kubernetes 您想要什么（描述所需状态），而不是直接告诉它要做什么。随着应用程序的增长和组件的增加，使用命令来管理集群变得不可能。最好使用命令来进行只读操作，例如`kubectl describe`、`kubectl get`和`kubectl logs`，并使用`kubectl apply`命令和 Kubernetes 对象配置文件（也称为清单文件）对集群的期望状态进行所有修改。

在使用清单文件时有一些推荐的做法：

+   最好使用 YAML 清单文件而不是 JSON 清单文件。 YAML 更容易管理，而且在 Kubernetes 社区中更常用。

+   将您的清单文件存储在 Git 等源代码控制中。在将任何配置更改应用到集群之前，先将更改推送到源代码控制中——这将使回滚和配置恢复变得更加容易。最终，您应该将此过程自动化为 CI/CD 流水线的一部分。

+   将多个清单文件组合成单个文件是推荐的，只要有意义。官方 Kubernetes 示例存储库提供了这种方法的很好演示：[`github.com/kubernetes/examples/blob/master/guestbook/all-in-one/guestbook-all-in-one.yaml`](https://github.com/kubernetes/examples/blob/master/guestbook/all-in-one/guestbook-all-in-one.yaml).

+   如果您的集群有多个清单文件，您可以使用 `kubectl apply` 递归地应用给定目录中的所有清单文件。

+   使用 `kubectl diff` 来了解将应用到当前集群配置的变化。

+   在删除 Kubernetes 对象时，请使用命令式的 `kubectl delete` 命令，因为它能给出可预测的结果。您可以在官方文档中了解更多关于资源声明式删除的信息，但在实践中，这是一种更具风险的方法：[`kubernetes.io/docs/tasks/manage-kubernetes-objects/declarative-config/#how-to-delete-objects`](https://kubernetes.io/docs/tasks/manage-kubernetes-objects/declarative-config/#how-to-delete-objects.).

+   尽可能使用标签来语义化地描述您的组件：[`kubernetes.io/docs/concepts/configuration/overview/#using-labels`](https://kubernetes.io/docs/concepts/configuration/overview/#using-labels.).

关于清单文件的更多最佳实践可以在官方文档中找到：[`kubernetes.io/docs/concepts/configuration/overview/`](https://kubernetes.io/docs/concepts/configuration/overview/).

现在，让我们尝试通过部署一个类似上一节中的应用程序来演示这种方法。这次，我们将使用 Deployment 和 service 对象，它们将在单独的清单文件中定义——在实际场景中，您可能会将这两个清单文件组合成一个文件，但出于演示目的，将它们分开是有意义的。按照以下步骤部署应用程序：

1.  打开 PowerShell 窗口。

1.  确保您的集群没有运行上一节中的资源——您可以使用 `kubectl get all` 命令来检查并使用 `kubectl delete` 命令来删除它们。

1.  为清单文件创建一个目录，例如`declarative-demo`：

```
md .\declarative-demo
cd .\declarative-demo
```

1.  创建包含部署定义的`windows-example-deployment.yaml`清单文件：

```
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: windows-example
  labels:
    app: sample
spec:
  replicas: 1
  selector:
    matchLabels:
      app: windows-example
  template:
    metadata:
      name: windows-example
      labels:
        app: windows-example
    spec:
      nodeSelector:
        "beta.kubernetes.io/os": windows
      containers:
      - name: windows-example
        image: mcr.microsoft.com/dotnet/core/samples:aspnetapp-nanoserver-1809
        ports:
          - containerPort: 80
```

1.  创建包含 LoadBalancer 服务定义的`windows-example-service.yaml`清单文件：

```
---
apiVersion: v1
kind: Service
metadata:
  name: windows-example
spec:
  type: LoadBalancer
  ports:
  - protocol: TCP
    port: 8080
    targetPort: 80
  selector:
    app: windows-example
```

1.  使用`kubectl apply`命令在当前目录中应用清单文件。请注意，如果您有多级目录层次结构，您可以使用`-R`标志进行递归处理：

```
PS C:\src\declarative-demo> kubectl apply -f .\
deployment.apps/windows-example created
service/windows-example created
```

1.  使用以下命令等待服务的外部 IP 可用：

```
PS C:\src\declarative-demo> kubectl get service -w
NAME              TYPE           CLUSTER-IP    EXTERNAL-IP   PORT(S)          AGE
kubernetes        ClusterIP      10.0.0.1      <none>        443/TCP          44h
windows-example   LoadBalancer   10.0.63.175   51.144.36.7   8080:30568/TCP   3m28s
```

1.  使用您的网络浏览器导航到外部 IP 和端口`8080`。

现在，让我们看看如何使用声明式方法对应用程序进行简单更改——我们想将 LoadBalancer 端口更改为`9090`：

1.  打开包含 LoadBalancer 服务定义的`windows-example-service.yaml`清单文件。

1.  将`spec.ports[0].port`值修改为`9090`。

1.  保存清单文件。

1.  （可选但建议）使用`kubectl diff`命令验证您的更改。请记住，您需要安装并在`$env:KUBECTL_EXTERNAL_DIFF`环境变量中定义适当的*diff*工具；您可以在第六章中了解更多信息，*与 Kubernetes 集群交互*：

```
kubectl diff -f .\
```

1.  再次应用清单文件：

```
PS C:\src\declarative-demo> kubectl apply -f .\
deployment.apps/windows-example unchanged
service/windows-example configured
```

1.  请注意，只有`service/windows-example`被检测为所需配置中的更改。

1.  现在，您可以在网络浏览器中导航到外部 IP 地址和端口`9090`以验证更改。

1.  如果您想删除当前目录中由清单文件创建的所有资源，可以使用以下命令：

```
kubectl delete -f .\
```

就是这样！正如您所看到的，声明式管理可能需要更多的样板配置，但最终，使用这种方法管理应用程序更加可预测和易于跟踪。

在管理在多个环境中运行的复杂应用程序时，考虑使用 Kustomization 文件（可与`kubectl apply`命令一起使用）或 Helm Charts。例如，使用 Kustomization 文件，您可以将配置文件组织在一个符合约定的目录结构中：[`kubectl.docs.kubernetes.io/pages/app_composition_and_deployment/structure_directories.html`](https://kubectl.docs.kubernetes.io/pages/app_composition_and_deployment/structure_directories.html)。

在下一节中，我们将简要介绍关于在 Windows 节点上调度 Pod 的推荐做法。

# 在 Windows 节点上调度 Pods

要在具有特定属性的节点上调度 Pods，Kubernetes 为您提供了一些可能的选项：

+   在 Pod 规范中使用`nodeName`。这是在给定节点上静态调度 Pods 的最简单形式，通常不建议使用。

+   在 Pod 规范中使用`nodeSelector`。这使您有可能仅在具有特定标签值的节点上调度您的 Pod。我们在上一节中已经使用了这种方法。

+   节点亲和性和反亲和性：这些概念扩展了`nodeSelector`方法，并提供了更丰富的语言来定义哪些节点是首选或避免为您的 Pod。您可以在官方文档中了解更多可能性：[`kubernetes.io/docs/concepts/configuration/assign-pod-node/#affinity-and-anti-affinity`](https://kubernetes.io/docs/concepts/configuration/assign-pod-node/#affinity-and-anti-affinity)。

+   节点污点和 Pod 容忍度：它们提供了与节点亲和性相反的功能-您将一个污点应用于给定节点（描述某种限制），Pod 必须具有特定的容忍度才能在被污染的节点上调度。

在混合 Windows/Linux 集群中调度 Pods 至少需要使用`nodeSelector`或一组带有`nodeSelector`的节点污点的组合。每个 Kubernetes 节点默认都带有一组标签，其中包括以下内容：

+   `kubernetes.io/arch`，描述节点的处理器架构，例如`amd64`或`arm`：这也被定义为`beta.kubernetes.io/arch`。

+   `kubernetes.io/os`，其值为`linux`或`windows`：这也被定义为`beta.kubernetes.io/os`。

您可以使用以下命令在 AKS Engine 集群中检查 Windows 节点（例如`7001k8s011`）的默认标签：

```
PS C:\src> kubectl describe node 7001k8s011
Name:               7001k8s011
Roles:              agent
Labels:             agentpool=windowspool2
 beta.kubernetes.io/arch=amd64
 beta.kubernetes.io/instance-type=Standard_D2_v3
 beta.kubernetes.io/os=windows
 failure-domain.beta.kubernetes.io/region=westeurope
 failure-domain.beta.kubernetes.io/zone=0
 kubernetes.azure.com/cluster=aks-engine-windows-resource-group
 kubernetes.azure.com/role=agent
 kubernetes.io/arch=amd64
 kubernetes.io/hostname=7001k8s011
 kubernetes.io/os=windows
 kubernetes.io/role=agent
 node-role.kubernetes.io/agent=
 storageprofile=managed
 storagetier=Standard_LRS
```

如果您的 Pod 规范中不包含`nodeSelector`，它可以在 Windows 和 Linux 节点上都可以调度-这是一个问题，因为 Windows 容器不会在 Linux 节点上启动，反之亦然。建议的做法是使用`nodeSelector`来可预测地调度您的 Pods，无论是 Windows 还是 Linux 容器。例如，在部署定义中，Pod 模板可能包含以下内容：

```
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: windows-example
spec:
...
  template:
...
    spec:
      nodeSelector:
        "beta.kubernetes.io/os": windows
...
```

或者，您可以在 Kubernetes 的最新版本中使用`"kubernetes.io/os": windows`选择器。对于 Linux 容器，您需要指定`"beta.kubernetes.io/os": linux`或`"kubernetes.io/os": linux`。

当您将 Windows 节点添加到现有的大型 Linux-only 集群中时，这种方法可能会导致问题，使用 Helm Charts 或 Kubernetes Operators - 这些工作负载可能没有默认指定 Linux 节点选择器。为了解决这个问题，您可以使用污点和容忍：使用特定的`NoSchedule`污点标记您的 Windows 节点，并为您的 Pod 使用匹配的容忍。我们将使用带有`os`键和值`Win1809`的污点来实现这个目的。

对于污点 Windows 节点，您有两种可能性：

+   在 kubelet 的注册级别上使用`--register-with-taints='os=Win1809:NoSchedule'`标志对节点进行污点。请注意，这种方法目前在 AKS Engine 中不可用，因为`--register-with-taints`不是用户可配置的 - 您可以在文档中阅读更多信息：[`github.com/Azure/aks-engine/blob/master/docs/topics/clusterdefinitions.md#kubeletconfig`](https://github.com/Azure/aks-engine/blob/master/docs/topics/clusterdefinitions.md#kubeletconfig)。

+   使用 kubectl 对节点进行污点。您可以使用以下命令添加一个污点：`kubectl taint nodes <nodeName> os=Win1809:NoSchedule`，并使用`kubectl taint nodes 7001k8s011 os:NoSchedule-`来删除它。

然后，您的部署定义将不得不为 Windows 指定适当的 Pod 节点选择器和污点容忍，以允许在 Windows 节点上调度：

```
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: windows-example
spec:
...
  template:
...
    spec:
      nodeSelector:
        "beta.kubernetes.io/os": windows
      tolerations:
      - key: "os"
        operator: "Equal"
        value: "Win1809"
        effect: "NoSchedule"
...
```

在这种方法中，对于 Linux 容器，您不需要指定任何节点选择器或污点容忍。但是，如果可能的话，建议使用节点选择器方法而不使用节点污点，特别是如果您正在构建一个新的集群。

在下一节中，我们将看看如何访问您的应用程序。

# 访问您的应用程序

要访问在 Pod 中运行的应用程序，根据您的情况，您有一些可能性。在调试和测试场景中，您可以通过以下简单的方式访问您的应用程序：

+   使用`kubectl exec`来创建一个临时的交互式 Pod。我们在之前的章节中使用了这种方法。

+   使用`kubectl proxy`来访问任何服务类型。这种方法仅适用于 HTTP(S)端点，因为它使用 Kubernetes API 服务器提供的代理功能。

+   使用`kubectl port-forward`。您可以使用这种方法来访问单个 Pod 或在部署或服务后面运行的 Pod。

如果您想要为生产环境的最终用户公开应用程序，您可以使用以下方法：

+   具有 LoadBalancer 或 NodePort 类型的服务对象：我们已经在上一节中演示了如何使用 LoadBalancer 服务。

+   使用 Ingress Controller 与 ClusterIP 类型的服务一起使用：这种方法减少了使用的云负载均衡器的数量（从而降低了运营成本），并在 Kubernetes 集群内执行负载均衡和路由。请注意，这种方法使用 L7 负载均衡，因此只能用于暴露 HTTP（S）端点。

您可以在《Kubernetes 网络》第五章中详细了解服务和 Ingress Controller。在本节的后面，我们将演示如何为演示应用程序使用 Ingress Controller。

您可以在官方文档中了解有关在集群中运行的应用程序的访问的更多信息：[`kubernetes.io/docs/tasks/administer-cluster/access-cluster-services/#accessing-services-running-on-the-cluster`](https://kubernetes.io/docs/tasks/administer-cluster/access-cluster-services/#accessing-services-running-on-the-cluster)。

让我们首先演示如何使用`kubectl proxy`和`kubectl port-forward`。执行以下步骤：

1.  打开一个 Powershell 窗口。

1.  确保之前部分中的演示应用程序已部署，并且在集群中部署了一个端口为`8080`的`windows-example`服务。

1.  运行`kubectl proxy`命令：

```
PS C:\src\declarative-demo> kubectl proxy
Starting to serve on 127.0.0.1:8001
```

1.  这将在本地主机的端口`8001`上暴露一个简单的代理服务器到远程 Kubernetes API 服务器。您可以自由地使用此端点使用 API，无需额外的身份验证。请注意，也可以使用原始 API 而不使用代理，但那样您就必须自己处理身份验证（[`kubernetes.io/docs/tasks/administer-cluster/access-cluster-api/`](https://kubernetes.io/docs/tasks/administer-cluster/access-cluster-api/)）。

1.  您的服务将在`http://<proxyEndpoint>/api/v1/namespaces/<namespaceName>/services/[https:]<serviceName>[:portName]/proxy`上可用。在我们的情况下，导航到`http://127.0.0.1:8001/api/v1/namespaces/default/services/windows-example/proxy/`。这种方法适用于任何服务类型，包括仅内部 ClusterIPs。

1.  终止`kubectl proxy`进程。

1.  现在，执行以下`kubectl port-forward`命令：

```
PS C:\src\declarative-demo> kubectl port-forward service/windows-example 5000:8080
Forwarding from 127.0.0.1:5000 -> 80
Forwarding from [::1]:5000 -> 80
```

1.  这将把来自您的本地主机`5000`端口的任何网络流量转发到`windows-example`服务的`8080`端口。例如，您可以在 Web 浏览器中导航到`http://127.0.0.1:5000/`。请注意，这种方法也适用于 HTTP(S)以外的不同协议。

1.  终止`kubectl port-forward`进程。

现在，让我们看看如何使用 Ingress Controller 来访问演示应用程序。使用 Ingress 是高度可定制的，有多个可用的 Ingress Controllers——我们将演示在 AKS Engine 混合集群上快速启动和运行`ingress-nginx`（[`www.nginx.com/products/nginx/kubernetes-ingress-controller`](https://www.nginx.com/products/nginx/kubernetes-ingress-controller)）。请注意，这种方法将 Ingress Controllers 的部署限制在 Linux 节点上——您将能够为运行在 Windows 节点上的服务创建 Ingress 对象，但所有的负载均衡将在 Linux 节点上执行。按照以下步骤：

1.  修改`windows-example-service.yaml`清单文件，使其具有`type: ClusterIP`，`port: 80`，并且没有`targetPort`：

```
apiVersion: v1
kind: Service
metadata:
  name: windows-example
spec:
  type: ClusterIP
  ports:
  - protocol: TCP
    port: 80
  selector:
    app: windows-example
```

1.  将您的修改应用到集群中：

```
kubectl apply -f .\
```

1.  应用官方的通用清单文件用于 ingress-nginx，它在 Linux 节点上创建一个具有一个副本的部署：

```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/static/mandatory.yaml
```

1.  申请官方的云特定清单文件用于 ingress-nginx。这将创建一个 LoadBalancer 类型的服务，用于 Ingress Controller：

```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/static/provider/cloud-generic.yaml
```

1.  等待 Ingress Controller 服务接收外部 IP 地址。外部 IP 地址`104.40.133.125`将用于所有配置在此 Ingress Controller 后运行的服务：

```
PS C:\src\declarative-demo> kubectl get service -n ingress-nginx -w
NAME            TYPE           CLUSTER-IP    EXTERNAL-IP      PORT(S)                      AGE
ingress-nginx   LoadBalancer   10.0.110.35   104.40.133.125   80:32090/TCP,443:32215/TCP   16m
```

1.  创建`windows-example-ingress.yaml`清单文件并定义 Ingress 对象。我们的应用程序的`windows-example`服务将在`<ingressLoadBalancerIp>/windows-example`路径下注册：

```
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: windows-example-ingress
  annotations:
    kubernetes.io/ingress.class: "nginx"
    nginx.ingress.kubernetes.io/rewrite-target: /$2
    nginx.ingress.kubernetes.io/ssl-redirect: "false"
spec:
  rules:
  - http:
      paths:
      - path: /windows-example(/|$)(.*)
        backend:
          serviceName: windows-example
          servicePort: 80
```

1.  应用更改：

```
kubectl apply -f .\
```

1.  导航到`http://104.40.133.125/windows-example`来测试 Ingress 定义。

当然，您可以为不同的服务创建多个 Ingress 对象，具有复杂的规则。一个经验法则是，尽可能使用 Ingress Controller 来公开您的 HTTP(S)端点，并为其他协议使用专用的 LoadBalancer 服务。

现在，让我们看看如何扩展您的应用程序！

# 扩展应用程序

在生产场景中，您肯定需要扩展您的应用程序 - 这就是 Kubernetes 强大之处；您可以手动扩展您的应用程序，也可以使用自动缩放。让我们首先看看如何执行部署的手动扩展。您可以通过命令或声明性地执行。要使用 PowerShell 中的命令执行扩展操作，请执行以下步骤：

1.  执行 `kubectl scale` 命令，将 `windows-example` 部署扩展到三个副本：

```
PS C:\src\declarative-demo> kubectl scale deployment/windows-example --replicas=3
deployment.extensions/windows-example scaled
```

1.  现在观察 Pods 如何被添加到您的部署中：

```
PS C:\src\declarative-demo> kubectl get pods -w
NAME READY STATUS RESTARTS AGE
windows-example-5cb7456474-5ndrm 0/1 ContainerCreating 0 8s
windows-example-5cb7456474-v7k84 1/1 Running 0 23m
windows-example-5cb7456474-xqp86 1/1 Running 0 8s
```

您也可以以声明性的方式执行类似的操作，这通常是推荐的。让我们进一步将应用程序扩展到四个副本：

1.  编辑 `windows-example-deployment.yaml` 清单文件，并将 `replicas` 修改为 `4`。

1.  保存清单文件并应用更改：

```
PS C:\src\declarative-demo> kubectl apply -f .\
deployment.apps/windows-example configured
ingress.networking.k8s.io/windows-example-ingress unchanged
service/windows-example unchanged
```

1.  再次使用 `kubectl get pods -w` 命令观察应用程序如何扩展。

Kubernetes 的真正力量在于自动缩放。我们将在第十一章 *配置应用程序使用 Kubernetes 功能* 中更详细地介绍自动缩放，因此在本节中，我们只会简要概述如何使用命令来执行它：

1.  首先，您需要为部署中的 pod 模板配置 CPU 资源限制 - 将其设置为一个小值，例如 `100m`。如果没有 CPU 资源限制，自动缩放将无法正确应用扩展策略：

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: windows-example
...
spec:
...
    spec:
...
      containers:
      - name: windows-example
...
        resources:
          limits:
            cpu: 100m
          requests:
            cpu: 100m
```

1.  应用修改：

```
kubectl apply -f .\
```

1.  执行以下 `kubectl autoscale` 命令：

```
kubectl autoscale deployment/windows-example --cpu-percent=15 --min=1 --max=5
```

1.  这将在集群中创建一个**水平 Pod 自动缩放器**（HPA）对象，采用默认算法，最少 1 个副本，最多 5 个副本，并基于目标 CPU 使用率的 15% 的限制进行配置。

1.  使用以下命令来检查 HPA 的状态：

```
kubectl describe hpa windows-example
```

1.  您可以尝试通过频繁刷新应用程序网页向您的 Pod 添加一些 CPU 负载。请注意，如果您使用 Ingress，您将命中 Ingress 控制器的缓存，因此在这种情况下 CPU 使用率可能不会增加。

1.  过一段时间，您会看到自动缩放开始并添加更多副本。当您减少负载时，部署将被缩减。您可以使用 `kubectl describe` 命令来检查时间线：

```
PS C:\src\declarative-demo> kubectl describe hpa windows-example
...
 Normal   SuccessfulRescale             11m                horizontal-pod-autoscaler  New size: 3; reason: cpu resource utilization (percentage of request) above target
 Normal   SuccessfulRescale             4m17s              horizontal-pod-autoscaler  New size: 1; reason: All metrics below target
```

1.  使用此命令删除 HPA 对象以关闭自动缩放：

```
kubectl delete hpa windows-example
```

对于托管的 AKS 实例，可以利用**节点级**自动缩放功能（[`docs.microsoft.com/en-us/azure/aks/cluster-autoscaler`](https://docs.microsoft.com/en-us/azure/aks/cluster-autoscaler)），为您的工作负载带来了另一个可伸缩性维度。此外，您可以考虑使用 Azure 容器实例（ACI）与 AKS 工作负载（[`docs.microsoft.com/en-us/azure/architecture/solution-ideas/articles/scale-using-aks-with-aci`](https://docs.microsoft.com/en-us/azure/architecture/solution-ideas/articles/scale-using-aks-with-aci)）。

恭喜！您已成功在 AKS Engine 混合 Kubernetes 集群上部署和自动缩放了您的第一个应用程序。

# 总结

本章简要介绍了如何在 AKS Engine 混合集群上部署和管理运行 Windows 容器应用程序。您学会了命令式和声明式集群配置管理的区别以及何时使用它们。我们已经使用了这两种方法来部署演示应用程序-现在您知道推荐的声明式方法比使用命令式命令更容易，更不容易出错。接下来，您将学习如何可预测地在 Windows 节点上安排 Pod，并如何处理将 Windows 容器工作负载添加到现有 Kubernetes 集群。最后，我们展示了如何访问在 Kubernetes 中运行的应用程序，供最终用户和开发人员使用，以及如何手动和自动扩展应用程序。

在下一章中，我们将利用所有这些新知识来将一个真正的.NET Framework 应用程序部署到我们的 Kubernetes 集群！

# 问题

1.  Kubernetes 对象的命令式和声明式管理有什么区别？

1.  何时推荐使用命令式命令？

1.  如何查看本地清单文件和当前集群状态之间的更改？

1.  在混合集群中安排 Pod 的推荐做法是什么？

1.  `kubectl proxy`和`kubectl port-forward`命令之间有什么区别？

1.  何时可以使用 Ingress Controller？

1.  如何手动扩展部署？

您可以在本书的*评估*中找到这些问题的答案。

# 进一步阅读

+   有关 Kubernetes 应用程序管理的更多信息，请参考以下 Packt 图书：

+   《完整的 Kubernetes 指南》（[`www.packtpub.com/virtualization-and-cloud/complete-kubernetes-guide`](https://www.packtpub.com/virtualization-and-cloud/complete-kubernetes-guide)）

+   使用 Kubernetes 入门-第三版（[`www.packtpub.com/virtualization-and-cloud/getting-started-kubernetes-third-edition`](https://www.packtpub.com/virtualization-and-cloud/getting-started-kubernetes-third-edition)）

+   *面向开发人员的 Kubernetes*（[`www.packtpub.com/virtualization-and-cloud/kubernetes-developers`](https://www.packtpub.com/virtualization-and-cloud/kubernetes-developers)）

+   目前，关于在 AKS Engine 上运行的混合 Windows/Linux 集群的大多数资源都可以在线获得。请查看 GitHub 上的官方文档以获取更多详细信息：

+   [`github.com/Azure/aks-engine/blob/master/docs/topics/windows.md`](https://github.com/Azure/aks-engine/blob/master/docs/topics/windows.md)

+   [`github.com/Azure/aks-engine/blob/master/docs/topics/windows-and-kubernetes.md`](https://github.com/Azure/aks-engine/blob/master/docs/topics/windows-and-kubernetes.md)

+   一般来说，许多关于 AKS（托管的 Kubernetes Azure 提供的内容，而不是 AKS Engine 本身）的主题都很有用，因为它们涉及如何将 Kubernetes 与 Azure 生态系统集成。您可以在以下 Packt 书籍中找到有关 AKS 本身的更多信息：

+   *使用 Kubernetes 进行 DevOps-第二版*（[`www.packtpub.com/virtualization-and-cloud/devops-kubernetes-second-edition`](https://www.packtpub.com/virtualization-and-cloud/devops-kubernetes-second-edition)）
