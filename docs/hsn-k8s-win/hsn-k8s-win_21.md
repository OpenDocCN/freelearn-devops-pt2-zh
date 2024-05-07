# 第十七章：评估

# 第一章：创建容器

1.  对象命名空间、进程表、作业对象和 Windows 容器隔离文件系统。此外，在这些低级功能之上，**主机计算服务**（**HCS**）和**主机网络服务**（**HNS**）抽象了运行和管理容器的公共接口。

1.  Windows Server 容器要求主机操作系统版本与容器基础镜像操作系统版本匹配。此外，在 Windows 上，您可以使用 Hyper-V 隔离，这使得可以使用与基础镜像操作系统版本不匹配的容器运行。

1.  在 Hyper-V 隔离中，每个容器都在一个专用的、最小的 Hyper-V 虚拟机中运行。容器不与主机操作系统共享内核；主机操作系统版本与容器基础操作系统版本之间没有兼容性限制。如果需要在非匹配的基础镜像操作系统版本和不受信任的代码执行场景中运行容器，则使用 Hyper-V 隔离。

1.  要在 Docker Desktop（版本 18.02 或更高版本）中启用 LCOW 支持，必须在 Docker 设置|守护程序中启用实验性功能选项。创建一个 LCOW 容器需要为`docker run`命令指定`--platform linux`参数。

1.  `docker logs <containerId>`

1.  对于已安装 Powershell 的 Windows 容器，可以使用以下命令：`docker exec -it <containerId> powershell.exe`。

# 第二章：在容器中管理状态

1.  容器层是每个 Docker 容器文件系统中的可写层的顶层。

1.  绑定挂载提供了一个简单的功能，可以将容器主机中的任何文件或目录挂载到给定的容器中。卷提供了类似的功能，但它们完全由 Docker 管理，因此您不必担心容器主机文件系统中的物理路径。

1.  可写容器层与容器主机耦合在一起，这意味着不可能轻松地将数据移动到不同的主机。层文件系统的性能比直接访问主机文件系统（例如使用卷）差。您不能在不同的容器之间共享可写层。

1.  在 Windows 主机上使用 SMB 全局映射功能，可以将 SMB 共享挂载到容器中可见。然后，您可以将 SMB 共享在容器中作为主机机器上的常规目录挂载。

1.  不行。要持久保存 Hyper-V 容器的存储数据，您必须使用 Docker 卷。如果您需要使用绑定挂载（例如，用于 SMB 全局映射），您必须使用进程隔离。

1.  `docker volume prune`

1.  Docker 中的卷驱动程序可用于管理托管在远程计算机或云服务上的卷。

# 第三章：使用容器映像

1.  Docker 注册表是一个有组织的、分层的系统，用于存储 Docker 映像，提供可伸缩的映像分发。Docker Hub 是由 Docker，Inc.托管和管理的官方公共 Docker 注册表。

1.  标签是存储库中单个图像的版本标签。

1.  `<dockerId>/<repositoryName>:<tag>`

1.  **Azure 容器注册表**（**ACR**）是由 Azure 云提供的完全托管的私有 Docker 注册表。在 ACR 的情况下，您可以使用自己的 Azure 存储帐户存储图像，并且可以使注册表完全私有，以满足您自己的基础设施需求。

1.  `latest`是在拉取或构建图像时使用的默认标签（如果您没有指定显式标签）。一般来说，除了在开发场景中，您不应该使用`latest`标签。在生产环境中，始终为您的 Kubernetes 清单或 Dockerfile 指令指定显式标签。

1.  Semver 建议使用三个数字的以下方案，即主要版本、次要版本和修补版本，用点分隔：`<major>.<minor>.<patch>`，根据需要递增每个数字。

1.  **Docker 内容信任**（**DCT**）提供了一种验证数据数字签名的方法，该数据在 Docker 引擎和 Docker 注册表之间传输。此验证允许发布者对其图像进行签名，并且消费者（Docker 引擎）验证签名以确保图像的完整性和来源。

# 第四章：Kubernetes 概念和 Windows 支持

1.  控制平面（主控）由一组组件组成，负责关于集群的全局决策，例如将应用实例的调度和部署到工作节点以及管理集群事件。数据平面由负责运行主控安排的容器工作负载的工作节点组成。

1.  集群管理使用声明性模型执行，这使得 Kubernetes 非常强大 - 您描述所需的状态，Kubernetes 会完成所有繁重的工作，将集群的当前状态转换为所需的状态。

1.  Kubernetes Pod 由一个或多个共享内核命名空间、IPC、网络堆栈（因此您可以通过相同的集群 IP 地址对其进行寻址，并且它们可以通过本地主机进行通信）和存储的容器组成。换句话说，Pod 可以包含共享某些资源的多个容器。

1.  Deployment API 对象用于声明式管理 ReplicaSet 的部署和扩展。这是确保新版本应用平稳部署的关键 API 对象。

1.  Windows 机器只能作为工作节点加入集群。无法在 Windows 上运行主组件，也没有在混合 Linux/Windows 集群的本地 Kubernetes 开发环境的设置，目前没有标准解决方案，例如 Minikube 或 Docker Desktop for Windows 支持这样的配置。

1.  Minikube 旨在为 Kubernetes 的本地开发提供稳定的环境。它可用于 Windows、Linux 和 macOS，但只能提供 Linux 集群。

1.  **AKS**（**Azure Kubernetes Service**的缩写）是 Azure 提供的完全托管的 Kubernetes 集群。AKS Engine 是 Azure 官方的开源工具，用于在 Azure 上提供自管理的 Kubernetes 集群。在内部，AKS 使用 AKS Engine，但它们不能管理彼此创建的集群。

# 第五章：Kubernetes 网络

1.  运行在节点上的 Pod 必须能够与所有节点上的所有 Pod（包括 Pod 的节点）进行通信，而无需 NAT 和显式端口映射。例如，运行在节点上的所有 Kubernetes 组件，如 kubelet 或系统守护程序/服务，必须能够与该节点上的所有 Pod 进行通信。

1.  您可以在集群节点之间存在二层（L2）连接时，仅使用 host-gw 来使用 Flannel。换句话说，在节点之间不能有任何 L3 路由器。

1.  NodePort 服务是作为 ClusterIP 服务实现的，具有使用任何集群节点 IP 地址和指定端口可达的额外功能。为实现这一点，kube-proxy 在 30000-32767 范围内（可配置）的每个节点上公开相同的端口，并设置转发，以便将对该端口的任何连接转发到 ClusterIP。

1.  降低成本（您只使用一个云负载均衡器来提供传入流量）和 L7 负载均衡功能

1.  容器运行时使用 CNI 插件将容器连接到网络，并在需要时从网络中移除它们。

1.  内部 vSwitch 未连接到容器主机上的网络适配器，而外部 vSwitch 连接并提供与外部网络的连接。

1.  Docker 网络模式（驱动程序）是来自 Docker 的概念，是**容器网络模型**（**CNM**）的一部分。这个规范是 Docker 提出的，旨在以模块化、可插拔的方式解决容器网络设置和管理挑战。CNI 是一个 CNCF 项目，旨在为任何容器运行时和网络实现提供一个简单明了的接口。它们以不同的方式解决了几乎相同的问题。在 Windows 上，Docker 网络模式和 CNI 插件的实现是相同的——它们都是 HNS 的轻量级适配器。

1.  在 Windows 上，覆盖网络模式使用外部 Hyper-V vSwitch 创建一个 VXLAN 覆盖网络。每个覆盖网络都有自己的 IP 子网，由可定制的 IP 前缀确定。

# 第六章：与 Kubernetes 集群交互

1.  kubectl 使用位于`~\.kube\config`的 kubeconfig 文件。这个 YAML 配置文件包含了 kubectl 连接到 Kubernetes API 所需的所有参数。

1.  您可以使用`KUBECONFIG`环境变量或`--kubeconfig`标志来强制 kubectl 在个别命令中使用不同的 kubeconfig。

1.  上下文用于组织和协调对多个 Kubernetes 集群的访问。

1.  `kubectl create`是一个命令，用于创建新的 API 资源，而`kubectl apply`是一个声明性管理命令，用于管理 API 资源。

1.  `kubectl patch`通过合并当前资源状态和仅包含修改属性的补丁来更新资源。补丁的常见用例是在混合 Linux/Windows 集群中需要强制执行现有 DaemonSet 的节点选择器时。

1.  `kubectl logs <podName>`

1.  `kubectl cp <podName>:<sourceRemotePath> <destinationLocalPath>`

# 第七章：部署混合本地 Kubernetes 集群

1.  如果您只计划将集群用于本地开发，请使用内部 NAT Hyper-V vSwitch。任何外部入站通信（除了您的 Hyper-V 主机机器）都需要 NAT。如果您的网络有 DHCP 和 DNS 服务器，您（或网络管理员）可以管理，那么请使用外部 Hyper-V vSwitch。这在大多数生产部署中都是这样的情况。

1.  简而言之，更改操作系统配置，比如禁用交换空间，安装 Docker 容器运行时，安装 Kubernetes 软件包，执行`kubeadm init`。

1.  服务子网是一个虚拟子网（不可路由），用于 Pod 访问服务。虚拟 IP 的可路由地址转换由运行在节点上的 kube-proxy 执行。Pod 子网是集群中所有 Pod 使用的全局子网。

1.  `kubeadm token create --print-join-command`

1.  `kubectl taint nodes --all node-role.kubernetes.io/master-`

1.  Flannel 网络使用 host-gw 后端（在 Windows 节点上使用 win-bridge CNI 插件）：host-gw 后端更可取，因为它处于稳定的功能状态，而 overlay 后端对于 Windows 节点仍处于 alpha 功能状态。

1.  简而言之，下载`sig-windows-tools`脚本，安装 Docker 和 Kubernetes 软件包；为脚本准备 JSON 配置文件；并执行它们。

1.  `kubectl logs <podName>`

# 第八章：部署混合 Azure Kubernetes 服务引擎集群

1.  AKS 是 Azure 提供的一个完全托管的 Kubernetes 集群。AKS Engine 是 Azure 官方的开源工具，用于在 Azure 上为自管理的 Kubernetes 集群进行配置。在内部，AKS 使用 AKS Engine，但它们不能管理彼此创建的集群。

1.  AKS Engine 根据提供的配置文件（集群 apimodel）生成**Azure 资源管理器**（ARM）模板。然后，您可以使用此 ARM 模板在 Azure 基础架构上部署一个完全功能的自管理 Kubernetes 集群。

1.  不可以。即使 AKS 在内部使用 AKS Engine，也不能使用 AKS Engine 来管理 AKS，反之亦然。

1.  Azure CLI，Azure Cloud Shell，kubectl，以及如果您想要使用 SSH 连接到节点，则还需要 Windows 的 SSH 客户端。

1.  AKS Engine 使用 apimodel（或集群定义）JSON 文件生成 ARM 模板，可用于直接部署 Kubernetes 集群到 Azure。

1.  使用 SSH 并执行以下命令：`ssh azureuser@<dnsPrefix>.<azureLocation>.cloudapp.azure.com`。

1.  假设`10.240.0.4`是 Windows 节点的私有 IP，创建一个 SSH 连接到主节点，将 RDP 端口转发到 Windows 节点，使用`ssh -L 5500:10.240.0.4:3389 azureuser@<dnsPrefix>.<azureLocation>.cloudapp.azure.com`命令。在一个新的命令行窗口中，使用`mstsc /v:localhost:5500`命令启动一个 RDP 会话。

# 第九章：部署您的第一个应用程序

1.  命令式方法包括执行命令式的 kubectl 命令，例如`kubectl run`或`kubectl expose`。在声明性方法中，您始终修改对象配置（清单文件），并使用`kubectl apply`命令在集群中创建或更新它们（或者，您可以使用 Kustomization 文件）。

1.  命令式的`kubectl delete`命令优于声明式删除，因为它提供可预测的结果。

1.  `kubectl diff -f <file/directory>`

1.  推荐的做法是使用`nodeSelector`来可预测地调度您的 Pod，无论是 Windows 还是 Linux 容器。

1.  您可以使用`kubectl proxy`访问任何 Service API 对象。`kubectl port-forward`是一个更低级别的命令，您可以使用它来访问单个 Pod 或部署中运行的 Pod 或服务后面的 Pod。

1.  只有当您有能够运行 Ingress Controller Pods 的节点时，才可以使用 Ingress Controller。例如，对于 ingress-nginx，只有 Linux 节点才能部署 Ingress Controller，您将能够为运行在 Windows 节点上的服务创建 Ingress 对象，但所有负载均衡都将在 Linux 节点上执行。

1.  `kubectl scale deployment/<deploymentName> --replicas=<targetNumberOfReplicas>`

# 第十章：部署 Microsoft SQL Server 2019 和 ASP.NET MVC 应用程序

1.  您可以从以下选项中选择：将参数传递给容器命令，为容器定义系统环境变量，将 ConfigMaps 或 Secrets 挂载为容器卷，并可选择使用 PodPresets 将所有内容包装起来。

1.  `LogMonitor.exe`充当应用程序进程的监督者，并将日志打印到标准输出，这些日志是根据配置文件从不同来源收集的。计划进一步扩展此解决方案，以用于侧车容器模式。

1.  您需要确保迁移可以回滚，并且数据库架构与旧版本和新版本的应用程序完全兼容。换句话说，不兼容的更改（例如重命名）必须特别处理，以使各个步骤之间保持向后兼容。

1.  这可以确保在 Pod 终止时数据持久性，并确保 SQL Server 故障转移，即使新的 Pod 被调度到不同的节点上。

1.  您需要使用`ef6.exe`命令来应用迁移。这可以使用 Kubernetes Job 对象执行。

1.  如果您为资源使用低于“限制”值的“请求”值，您可能会进入资源超额分配状态。这使得 Pod 可以临时使用比它们请求的资源更多的资源，并实现了更有效的 Pod 工作负载的装箱。

1.  VS 远程调试器在容器中的`4020` TCP 端口上公开。要连接到它，而不将其公开为服务对象，您需要使用 kubectl 端口转发。

# 第十一章：配置应用程序以使用 Kubernetes 功能

1.  命名空间的一般原则是提供资源配额和对象名称的范围。您将根据集群的大小和团队的大小来组织命名空间。

1.  就绪探针用于确定给定容器是否准备好接受流量。存活探针用于检测容器是否需要重新启动。

1.  这个探针的错误配置可能导致您的服务和容器重新启动循环中的级联故障。

1.  `requests`指定系统提供的给定资源的保证数量。`limits`指定系统提供的给定资源的最大数量。

1.  避免抖动（副本计数频繁波动）。

1.  ConfigMaps 和 Secrets 可以容纳技术上任何类型的由键值对组成的数据。Secrets 的目的是保留访问依赖项的敏感信息，而 ConfigMaps 应该用于一般应用程序配置目的。

1.  `volumeClaimTemplates`用于为此 StatefulSet 中的每个 Pod 副本创建专用的 PersistentVolumeClaim。

1.  为了确保在 Kubernetes 中对部署进行真正的零停机更新，您需要配置适当的探针，特别是就绪探针。这样，用户只有在该副本能够正确响应请求时才会被重定向到副本。

1.  最小权限原则：您的应用程序应仅访问其自己的资源（建议您使用专用服务帐户运行每个应用程序，该帐户可以访问该应用程序的 Secrets 或 ConfigMaps），用户应根据其在项目中的角色拥有受限制的访问权限（例如，QA 工程师可能只需要对集群具有只读访问权限）。

# 第十二章：使用 Kubernetes 进行开发工作流程

1.  Helm 用于为您的 Kubernetes 应用程序创建可再分发的软件包。您可以使用它来部署其他人提供的应用程序，也可以将其用作您自己应用程序的内部软件包和微服务系统的依赖管理器。

1.  Helm 2 需要在 Kubernetes 上部署一个名为 Tiller 的专用服务，它负责与 Kubernetes API 的实际通信。这引起了各种问题，包括安全和 RBAC 问题。从 Helm 3.0.0 开始，不再需要 Tiller，图表管理由客户端完成。

1.  在 Helm 中使用 Kubernetes Job 对象作为安装后钩子。

1.  在 Helm 图表清单或值文件中使用新的 Docker 镜像，并执行 `helm upgrade`。

1.  快照调试器是 Azure 应用程序洞察的一个功能，它监视您的应用程序的异常遥测，包括生产场景。每当出现未处理的异常（顶部抛出），快照调试器都会收集托管内存转储，可以直接在 Azure 门户中进行分析，或者对于更高级的场景，可以使用 Visual Studio 2019 企业版。

1.  您应该更喜欢 Kubernetes 的适当声明式管理。

1.  Azure Dev Spaces 服务为使用 AKS 集群的团队提供了快速迭代的开发体验。

# 第十三章：保护 Kubernetes 集群和应用程序

1.  Kubernetes 本身不提供管理访问集群的普通外部用户的手段。这应该委托给一个可以与 Kubernetes 集成的外部身份验证提供程序，例如，通过认证代理。

1.  为了减少攻击向量，建议的做法是永远不要使用 LoadBalancer 服务公开 Kubernetes 仪表板，并始终使用 kubectl 代理来访问页面。

1.  这将为您的 API 资源和 Secrets 提供额外的安全层，否则它们将以未加密的形式保存在 etcd 中。

1.  不，此功能仅在 Linux 容器中受支持。

1.  NetworkPolicy 对象定义了 Pod 组如何相互通信以及一般网络端点的通信方式 - 将它们视为 OSI 模型第 3 层的网络分割的基本防火墙。要使用网络策略，您需要使用支持网络策略的网络提供程序之一。

1.  在 Windows 上，作为卷挂载到 Pods 的 Kubernetes Secrets 以明文形式写入节点磁盘存储（而不是 RAM）。这是因为 Windows 目前不支持将内存文件系统挂载到 Pod 容器。这可能带来安全风险，并需要额外的操作来保护集群。

1.  当您拥有根权限时，您可以从`/proc/<pid>/environ`枚举出进程的所有环境变量，包括以这种方式注入的 Secrets。对于作为卷挂载的 Secrets，由于使用了`tmpfs`，这是不可能的。

# 第十四章：使用 Prometheus 监控 Kubernetes 应用程序

1.  为您的组件提供可观察性意味着公开有关其内部状态的信息，以便您可以轻松访问数据并推断出组件的实际状态。换句话说，如果某物是可观察的，您就可以理解它。

1.  WMI Exporter 可用于监视 Windows 节点主机操作系统和硬件。要监视 Docker 引擎本身，可以使用引擎公开的实验性指标服务器。

1.  在大规模运行的生产环境中，您可以使用 Prometheus Operator 轻松部署和管理多个不同需求的 Prometheus 集群。

1.  WMI Exporter 和 Docker 引擎指标服务器在每个节点上的专用端口上公开指标。我们需要两个额外的抓取作业来单独处理它们。

1.  将 Telegraf 服务直接托管在您的容器中使用。

1.  为您的应用程序提供额外的仪器和对业务逻辑的洞察。

1.  在您的服务对象清单中，定义一个额外的注释，例如`prometheus.io/secondary-port`。之后，您必须创建一个专用的抓取作业，它将以类似的方式消耗新的注释，就像`prometheus.io/port`一样。

1.  热图是可视化直方图随时间变化的最有效方式，最近 Grafana 已扩展了对 Prometheus 直方图指标的热图本地支持。

# 第十五章：灾难恢复

1.  DR 和 BC 之间的主要区别在于，DR 侧重于在停机后使基础设施恢复运行，而 BC 涵盖了在重大事件期间保持业务场景运行。

1.  `etcd`集群由主节点和持久卷使用。

1.  快照是由 etcd 的 v3 API 提供的备份文件。

1.  Velero 可以执行`etcd`快照，将其管理在外部存储中，并在需要时进行恢复。此外，它还可以用于使用 Restic 集成执行持久卷的备份。Etcd-operator 用于在 Kubernetes 之上提供多个`etcd`集群。您可以轻松管理`etcd`集群并执行备份恢复操作。如果您计划在您的环境中管理多个 Kubernetes 集群，请使用此方法。

1.  访问所有 Kubernetes 主节点，并在所有机器上并行执行相同的步骤：下载目标快照文件，将其恢复到本地目录，停止 Kubernetes 主组件，停止`etcd`服务，停止 kubelet 服务，交换`etcd`数据目录，启动`etcd`服务，最后启动 kubelet 服务。

1.  Kubernetes CronJob 使您能够按固定计划安排 Kubernetes 作业，类似于 Linux 系统中的 cron。

1.  从集群中删除失败的成员，添加新的替代成员，如果有多个失败的成员，则依次替换成员。

# 第十六章：运行 Kubernetes 的生产考虑

1.  在不可变基础设施中，一旦机器被配置，您还不会执行任何修改。如果您需要进行配置更改或热修复，您需要构建新的机器映像并配置新的机器。

1.  Kubernetes 可以被视为管理您的不可变容器基础设施和应用程序工作负载的平台——每当您创建一个新的 Docker 映像并部署新版本时，您只是创建新的容器并丢弃旧的容器。如果您使用声明性方法来管理您的 Kubernetes 对象，您最终会得到整洁的基础设施即代码。

1.  GitOps 是一种由 WeaveWorks 提出的管理 Kubernetes 集群和应用程序的方式，其中 Git 存储库是声明性基础架构和应用程序工作负载的唯一真相来源。这种方法完全符合基础设施即代码范式。

1.  Flux 可以用于轻松实现 GitOps，用于您的 Kubernetes 集群。

1.  升级运行在主主节点上的组件，升级运行在其他主节点上的组件，以及升级工作节点。

1.  将节点标记为不可调度，并排空现有的 Pod，然后应用所需的更新并重新启动机器，然后取消节点标记，使其再次可调度。
