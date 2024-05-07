# 第六章：使用关键的 Kubernetes 资源

在本章中，我们将设计一个挑战 Kubernetes 能力和可伸缩性的大规模平台。Hue 平台的目标是创建一个无所不知、无所不能的数字助手。Hue 是你的数字延伸。它将帮助你做任何事情，找到任何东西，并且在许多情况下，将代表你做很多事情。它显然需要存储大量信息，与许多外部服务集成，响应通知和事件，并且在与你的互动方面非常智能。

在本章中，我们将有机会更好地了解 Kubectl 和其他相关工具，并将详细探讨我们之前见过的资源，如 pod，以及新资源，如**jobs**。在本章结束时，你将清楚地了解 Kubernetes 有多么令人印象深刻，以及它如何可以作为极其复杂系统的基础。

# 设计 Hue 平台

在本节中，我们将为惊人的 Hue 平台设定舞台并定义范围。Hue 不是老大哥，Hue 是小弟！Hue 将做任何你允许它做的事情。它可以做很多事情，但有些人可能会担心，所以你可以选择 Hue 可以帮助你多少。准备好进行一次疯狂的旅行！

# 定义 Hue 的范围

Hue 将管理你的数字人格。它将比你自己更了解你。以下是 Hue 可以管理并帮助你的一些服务列表：

+   搜索和内容聚合

+   医疗

+   智能家居

+   金融-银行、储蓄、退休、投资

+   办公室

+   社交

+   旅行

+   健康

+   家庭

+   智能提醒和通知：让我们想想可能性。Hue 将了解你，也了解你的朋友和所有领域的其他用户的总和。Hue 将实时更新其模型。它不会被陈旧的数据所困扰。它将代表你采取行动，提供相关信息，并持续学习你的偏好。它可以推荐你可能喜欢的新节目或书籍，根据你的日程安排和家人或朋友的情况预订餐厅，并控制你的家庭自动化。

+   安全、身份和隐私：Hue 是您在线的代理。有人窃取您的 Hue 身份，甚至只是窃听您的 Hue 互动的后果是灾难性的。潜在用户甚至可能不愿意信任 Hue 组织他们的身份。让我们设计一个非信任系统，让用户随时有权终止 Hue。以下是一些朝着正确方向的想法：

+   通过专用设备和多因素授权实现强大的身份验证，包括多种生物识别原因

+   频繁更换凭证

+   快速服务暂停和所有外部服务的身份重新验证（将需要向每个提供者提供原始身份证明）

+   Hue 后端将通过短暂的令牌与所有外部服务进行交互

+   将 Hue 构建为一组松散耦合的微服务的架构。

Hue 的架构将需要支持巨大的变化和灵活性。它还需要非常可扩展，其中现有的功能和外部服务不断升级，并且新的功能和外部服务集成到平台中。这种规模需要微服务，其中每个功能或服务都与其他服务完全独立，除了通过标准和/或可发现的 API 进行定义的接口。

# Hue 组件

在着手进行微服务之旅之前，让我们回顾一下我们需要为 Hue 构建的组件类型。

+   用户资料：

用户资料是一个重要组成部分，有很多子组件。它是用户的本质，他们的偏好，跨各个领域的历史，以及 Hue 对他们了解的一切。

+   用户图：

用户图组件模拟了用户在多个领域之间的互动网络。每个用户参与多个网络：社交网络（如 Facebook 和 Twitter）、专业网络、爱好网络和志愿者社区。其中一些网络是临时的，Hue 将能够对其进行结构化以使用户受益。Hue 可以利用其对用户连接的丰富资料，即使不暴露私人信息，也能改善互动。

+   身份：

身份管理是至关重要的，正如前面提到的，因此它值得一个单独的组件。用户可能更喜欢管理具有独立身份的多个互斥配置文件。例如，也许用户不愿意将他们的健康配置文件与社交配置文件混合在一起，因为这样做可能会意外地向朋友透露个人健康信息的风险。

+   **授权者**：

授权者是一个关键组件，用户明确授权 Hue 代表其执行某些操作或收集各种数据。这包括访问物理设备、外部服务的帐户和主动程度。

+   **外部服务**：

Hue 是外部服务的聚合器。它并非旨在取代您的银行、健康提供者或社交网络。它将保留大量关于您活动的元数据，但内容将保留在您的外部服务中。每个外部服务都需要一个专用组件来与外部服务的 API 和政策进行交互。当没有 API 可用时，Hue 通过自动化浏览器或原生应用程序来模拟用户。

+   **通用传感器**：

Hue 价值主张的一个重要部分是代表用户行事。为了有效地做到这一点，Hue 需要意识到各种事件。例如，如果 Hue 为您预订了假期，但它感觉到有更便宜的航班可用，它可以自动更改您的航班或要求您确认。有无限多件事情可以感知。为了控制感知，需要一个通用传感器。通用传感器将是可扩展的，但提供一个通用接口，供 Hue 的其他部分统一使用，即使添加了越来越多的传感器。

+   **通用执行器**：

这是通用传感器的对应物。Hue 需要代表您执行操作，比如预订航班。为了做到这一点，Hue 需要一个通用执行器，可以扩展以支持特定功能，但可以以统一的方式与其他组件交互，比如身份管理器和授权者。

+   **用户学习者**：

这是 Hue 的大脑。它将不断监视您所有的互动（经您授权的），并更新对您的模型。这将使 Hue 随着时间变得越来越有用，预测您的需求和兴趣，提供更好的选择，在合适的时候提供更相关的信息，并避免让人讨厌和压抑。

# Hue 微服务

每个组件的复杂性都非常巨大。一些组件，比如外部服务、通用传感器和通用执行器，需要跨越数百、数千甚至更多不断变化的外部服务进行操作，而这些服务是在 Hue 的控制范围之外的。甚至用户学习器也需要学习用户在许多领域和领域的偏好。微服务通过允许 Hue 逐渐演变并增加更多的隔离能力来满足这种需求，而不会在自身的复杂性下崩溃。每个微服务通过标准接口与通用 Hue 基础设施服务进行交互，并且可以通过明确定义和版本化的接口与其他一些服务进行交互。每个微服务的表面积是可管理的，微服务之间的编排基于标准最佳实践：

+   **插件**：

插件是扩展 Hue 而不会产生大量接口的关键。关于插件的一件事是，通常需要跨越多个抽象层的插件链。例如，如果我们想要为 Hue 添加与 YouTube 的新集成，那么你可以收集大量特定于 YouTube 的信息：你的频道、喜欢的视频、推荐以及你观看过的视频。为了向用户显示这些信息并允许他们对其进行操作，你需要跨越多个组件并最终在用户界面中使用插件。智能设计将通过聚合各种操作类别，如推荐、选择和延迟通知，来帮助许多不同的服务。

插件的好处在于任何人都可以开发。最初，Hue 开发团队将不得不开发插件，但随着 Hue 变得更加流行，外部服务将希望与 Hue 集成并构建 Hue 插件以启用其服务。

当然，这将导致插件注册、批准和策划的整个生态系统。

+   **数据存储**：

Hue 将需要多种类型的数据存储和每种类型的多个实例来管理其数据和元数据：

+   +   关系数据库

+   图数据库

+   时间序列数据库

+   内存缓存

由于 Hue 的范围，每个数据库都将需要进行集群化和分布式处理。

+   **无状态微服务**：

微服务应该大部分是无状态的。这将允许特定实例被快速启动和关闭，并根据需要在基础设施之间迁移。状态将由存储管理，并由短暂的访问令牌访问微服务。

+   **基于队列的交互**：

所有这些微服务需要相互通信。用户将要求 Hue 代表他们执行任务。外部服务将通知 Hue 各种事件。与无状态微服务相结合的队列提供了完美的解决方案。每个微服务的多个实例将监听各种队列，并在从队列中弹出相关事件或请求时做出响应。这种安排非常健壮且易于扩展。每个组件都可以是冗余的并且高度可用。虽然每个组件都可能出现故障，但系统非常容错。

队列可以用于异步的 RPC 或请求-响应式的交互，其中调用实例提供一个私有队列名称，被调用者将响应发布到私有队列。

# 规划工作流程

Hue 经常需要支持工作流程。典型的工作流程将得到一个高层任务，比如预约牙医；它将提取用户的牙医详情和日程安排，与用户的日程安排匹配，在多个选项之间进行选择，可能与用户确认，预约，并设置提醒。我们可以将工作流程分类为完全自动和涉及人类的人工工作流程。然后还有涉及花钱的工作流程。

# 自动工作流程

自动工作流程不需要人为干预。Hue 有完全的权限来执行从开始到结束的所有步骤。用户分配给 Hue 的自主权越多，它的效果就会越好。用户应该能够查看和审计所有的工作流程，无论是过去还是现在。

# 人工工作流程

人工工作流程需要与人的交互。最常见的情况是用户需要从多个选项中进行选择或批准某项操作，但也可能涉及到另一个服务上的人员。例如，要预约牙医，您可能需要从秘书那里获取可用时间的列表。

# 预算意识工作流程

有些工作流程，比如支付账单或购买礼物，需要花钱。虽然从理论上讲，Hue 可以被授予对用户银行账户的无限访问权限，但大多数用户可能更愿意为不同的工作流程设置预算，或者只是将花钱作为经过人工批准的活动。

# 使用 Kubernetes 构建 Hue 平台

在本节中，我们将看看各种 Kubernetes 资源以及它们如何帮助我们构建 Hue。首先，我们将更好地了解多才多艺的 Kubectl，然后我们将看看在 Kubernetes 中运行长时间运行的进程，内部和外部暴露服务，使用命名空间限制访问，启动临时作业以及混合非集群组件。显然，Hue 是一个庞大的项目，所以我们将在本地 Minikube 集群上演示这些想法，而不是实际构建一个真正的 Hue Kubernetes 集群。

# 有效使用 Kubectl

Kubectl 是您的瑞士军刀。它几乎可以做任何与集群相关的事情。在幕后，Kubectl 通过 API 连接到您的集群。它读取您的`.kube/config`文件，其中包含连接到您的集群或集群所需的信息。命令分为多个类别：

+   **通用命令**：以通用方式处理资源：`create`，`get`，`delete`，`run`，`apply`，`patch`，`replace`等等

+   **集群管理命令**：处理节点和整个集群：`cluster-info`，`certificate`，`drain`等等

+   **故障排除命令**：`describe`，`logs`，`attach`，`exec`等等

+   **部署命令**：处理部署和扩展：`rollout`，`scale`，`auto-scale`等等

+   **设置命令**：处理标签和注释：`label`，`annotate`等等

```
Misc commands: help, config, and version 
```

您可以使用 Kubernetes `config view`查看配置。

这是 Minikube 集群的配置：

```
~/.minikube > k config view 
apiVersion: v1 
clusters: 
- cluster: 
    certificate-authority: /Users/gigi.sayfan/.minikube/ca.crt 
    server: https://192.168.99.100:8443 
  name: minikube 
contexts: 
- context: 
    cluster: minikube 
    user: minikube 
  name: minikube 
current-context: minikube 
kind: Config 
preferences: {} 
users: 
- name: minikube 
  user: 
    client-certificate: /Users/gigi.sayfan/.minikube/client.crt 
    client-key: /Users/gigi.sayfan/.minikube/client.key 
```

# 理解 Kubectl 资源配置文件

许多 Kubectl 操作，比如`create`，需要复杂的分层输出（因为 API 需要这种输出）。Kubectl 使用 YAML 或 JSON 配置文件。这是一个用于创建 pod 的 JSON 配置文件：

```
apiVersion: v1
kind: Pod
metadata:
  name: ""
  labels:
    name: ""
  namespace: ""
  annotations: []
  generateName: ""
spec:
     ...  
```

+   `apiVersion`：非常重要的 Kubernetes API 不断发展，并且可以通过 API 的不同版本支持相同资源的不同版本。

+   `kind`：`kind`告诉 Kubernetes 它正在处理的资源类型，在本例中是`pod`。这是必需的。

+   `metadata`：这是描述 pod 及其操作位置的大量信息：

+   `名称`：在其命名空间中唯一标识 pod

+   `标签`：可以应用多个标签

+   `命名空间`：pod 所属的命名空间

+   `注释`：可查询的注释列表

+   `规范`：`规范`是一个包含启动 pod 所需的所有信息的 pod 模板。它可能非常复杂，所以我们将在多个部分中探讨它：

```
"spec": {
  "containers": [
  ],
  "restartPolicy": "",
  "volumes": [
  ]
}  
```

+   `容器规范`：pod 规范的容器是容器规范的列表。每个容器规范都有以下结构：

```
        {
          "name": "",
          "image": "",
          "command": [
            ""
          ],
          "args": [
            ""
          ],
          "env": [
            {
              "name": "",
              "value": ""
            }
          ],
          "imagePullPolicy": "",
          "ports": [
            {
              "containerPort": 0,
              "name": "",
              "protocol": ""
            }
          ],
          "resources": {
            "cpu": ""
            "memory": ""
          }
        }
```

每个容器都有一个镜像，一个命令，如果指定了，会替换 Docker 镜像命令。它还有参数和环境变量。然后，当然还有镜像拉取策略、端口和资源限制。我们在前几章中已经涵盖了这些。

# 在 pod 中部署长时间运行的微服务

长时间运行的微服务应该在 pod 中运行，并且是无状态的。让我们看看如何为 Hue 的一个微服务创建 pod。稍后，我们将提高抽象级别并使用部署。

# 创建 pod

让我们从一个常规的 pod 配置文件开始，为创建 Hue 学习者内部服务。这个服务不需要暴露为公共服务，它将监听一个队列以获取通知，并将其见解存储在一些持久存储中。

我们需要一个简单的容器来运行 pod。这可能是有史以来最简单的 Docker 文件，它将模拟 Hue 学习者：

```
FROM busybox
CMD ash -c "echo 'Started...'; while true ; do sleep 10 ; done"  
```

它使用`busybox`基础镜像，打印到标准输出`Started...`，然后进入一个无限循环，这显然是长时间运行的。

我构建了两个标记为`g1g1/hue-learn:v3.0`和`g1g1/hue-learn:v4.0`的 Docker 镜像，并将它们推送到 Docker Hub 注册表（`g1g1`是我的用户名）。

```
docker build . -t g1g1/hue-learn:v3.0
docker build . -t g1g1/hue-learn:v4.0
docker push g1g1/hue-learn:v3.0
docker push g1g1/hue-learn:v4.0  
```

现在，这些镜像可以被拉入 Hue 的 pod 中的容器。

我们将在这里使用 YAML，因为它更简洁和易读。这里是样板和`元数据`标签：

```
apiVersion: v1
kind: Pod
metadata:
  name: hue-learner
  labels:
    app: hue 
    runtime-environment: production
    tier: internal-service 
  annotations:
    version: "3.0"
```

我使用注释而不是标签的原因是，标签用于标识部署中的一组 pod。不允许修改标签。

接下来是重要的`容器`规范，为每个容器定义了强制的`名称`和`镜像`：

```
spec:
  containers:
  - name: hue-learner
    image: g1g1/hue-learn:v3.0  
```

资源部分告诉 Kubernetes 容器的资源需求，这允许更高效和紧凑的调度和分配。在这里，容器请求`200`毫 CPU 单位（0.2 核心）和`256` MiB：

```
resources:
  requests:
    cpu: 200m
    memory: 256Mi 
```

环境部分允许集群管理员提供将可用于容器的环境变量。这里告诉它通过`dns`发现队列和存储。在测试环境中，可能会使用不同的发现方法：

```
env:
- name: DISCOVER_QUEUE
  value: dns
- name: DISCOVER_STORE
  value: dns 
```

# 用标签装饰 pod

明智地为 pod 贴上标签对于灵活的操作至关重要。它让您可以实时演变您的集群，将微服务组织成可以统一操作的组，并以自发的方式深入观察不同的子集。

例如，我们的 Hue 学习者 pod 具有以下标签：

+   **运行环境**：生产

+   **层级**：内部服务

版本注释可用于支持同时运行多个版本。如果需要同时运行版本 2 和版本 3，无论是为了提供向后兼容性还是在从`v2`迁移到`v3`期间暂时运行，那么具有版本注释或标签允许独立扩展不同版本的 pod 并独立公开服务。`runtime-environment`标签允许对属于特定环境的所有 pod 执行全局操作。`tier`标签可用于查询属于特定层的所有 pod。这只是例子；在这里，您的想象力是限制。

# 使用部署部署长时间运行的进程

在大型系统中，pod 不应该只是创建并放任不管。如果由于任何原因 pod 意外死亡，您希望另一个 pod 替换它以保持整体容量。您可以自己创建复制控制器或副本集，但这也会留下错误的可能性以及部分故障的可能性。在启动 pod 时指定要创建多少副本更有意义。

让我们使用 Kubernetes 部署资源部署我们的 Hue 学习者微服务的三个实例。请注意，部署对象在 Kubernetes 1.9 时变得稳定：

```
apiVersion: apps/v1 (use apps/v1beta2 before 1.9)
 kind: Deployment
 metadata:
 name: hue-learn
 labels:
 app: hue
 spec:
 replicas: 3
 selector:
 matchLabels:
 app: hue
 template:
 metadata:
 labels:
 app: hue
 spec:
 <same spec as in the pod template>
```

`pod spec`与我们之前使用的 pod 配置文件中的`spec`部分相同。

让我们创建部署并检查其状态：

```
> kubectl create -f .\deployment.yaml
deployment "hue-learn" created
> kubectl get deployment hue-learn
NAME        DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
hue-learn   3          3           3             3        4m

> kubectl get pods | grep hue-learn
NAME                        READY     STATUS    RESTARTS   AGE
hue-learn-237202748-d770r   1/1       Running   0          2m
hue-learn-237202748-fwv2t   1/1       Running   0          2m
hue-learn-237202748-tpr4s   1/1       Running   0          2m  
```

您可以使用`kubectl describe`命令获取有关部署的更多信息。

# 更新部署

Hue 平台是一个庞大且不断发展的系统。您需要不断升级。部署可以更新以无痛的方式推出更新。您可以更改 pod 模板以触发由 Kubernetes 完全管理的滚动更新。

目前，所有的 pod 都在运行版本 3.0：

```
> kubectl get pods -o json | jq .items[0].spec.containers[0].image
"3.0"  
```

让我们更新部署以升级到版本 4.0。修改部署文件中的镜像版本。不要修改标签；这会导致错误。通常，您会修改镜像和一些相关的元数据在注释中。然后我们可以使用`apply`命令来升级版本：

```
> kubectl apply -f hue-learn-deployment.yaml
deployment "hue-learn" updated
> kubectl get pods -o json | jq .items[0].spec.containers[0].image
"4.0"  
```

# 分离内部和外部服务

内部服务是只有其他服务或集群中的作业（或登录并运行临时工具的管理员）直接访问的服务。在某些情况下，内部服务根本不被访问，只是执行其功能并将结果存储在其他服务以解耦方式访问的持久存储中。

但是一些服务需要向用户或外部程序公开。让我们看一个虚假的 Hue 服务，它管理用户的提醒列表。它实际上并不做任何事情，但我们将用它来说明如何公开服务。我将一个虚假的`hue-reminders`镜像（与`hue-learn`相同）推送到 Docker Hub：

```
docker push g1g1/hue-reminders:v2.2  
```

# 部署内部服务

这是部署，它与 Hue-learner 部署非常相似，只是我删除了`annotations`、`env`和`resources`部分，只保留了一个标签以节省空间，并在容器中添加了一个`ports`部分。这是至关重要的，因为服务必须通过一个端口公开，其他服务才能访问它：

```
apiVersion: apps/v1a1
kind: Deployment
metadata:
 name: hue-reminders
spec:
 replicas: 2 
 template:
 metadata:
 name: hue-reminders
 labels:
 app: hue-reminders
 spec: 
 containers:
 - name: hue-reminders
 image: g1g1/hue-reminders:v2.2 
 ports:
 - containerPort: 80 
```

当我们运行部署时，两个 Hue `reminders` pod 被添加到集群中：

```
> kubectl create -f hue-reminders-deployment.yaml
> kubectl get pods
NAME                           READY   STATUS    RESTARTS   AGE
hue-learn-56886758d8-h7vm7      1/1    Running       0      49m
hue-learn-56886758d8-lqptj      1/1    Running       0      49m
hue-learn-56886758d8-zwkqt      1/1    Running       0      49m
hue-reminders-75c88cdfcf-5xqtp  1/1    Running       0      50s
hue-reminders-75c88cdfcf-r6jsx  1/1    Running       0      50s 
```

好的，pod 正在运行。理论上，其他服务可以查找或配置其内部 IP 地址，并直接访问它们，因为它们都在同一个网络空间中。但这并不具有可扩展性。每当一个 reminders pod 死掉并被新的 pod 替换，或者当我们只是扩展 pod 的数量时，所有访问这些 pod 的服务都必须知道这一点。服务通过提供所有 pod 的单一访问点来解决这个问题。服务如下：

```
apiVersion: v1
kind: Service
metadata:
 name: hue-reminders
 labels:
 app: hue-reminders 
spec:
 ports:
 - port: 80
 protocol: TCP
 selector:
 app: hue-reminders
```

该服务具有一个选择器，选择所有具有与其匹配的标签的 pod。它还公开一个端口，其他服务将使用该端口来访问它（它不必与容器的端口相同）。

# 创建 hue-reminders 服务

让我们创建服务并稍微探索一下：

```
> kubectl create -f hue-reminders-service.yaml
service "hue-reminders" created
> kubectl describe svc hue-reminders
Name:              hue-reminders
Namespace:         default
Labels:            app=hue-reminders
Annotations:       <none>
Selector:          app=hue-reminders
Type:              ClusterIP
IP:                10.108.163.209
Port:              <unset>  80/TCP
TargetPort:        80/TCP
Endpoints:         172.17.0.4:80,172.17.0.6:80
Session Affinity:  None
Events:            <none>  
```

服务正在运行。其他 pod 可以通过环境变量或 DNS 找到它。所有服务的环境变量都是在 pod 创建时设置的。这意味着如果在创建服务时已经有一个 pod 在运行，您将不得不将其终止，并让 Kubernetes 使用环境变量重新创建它（您通过部署创建 pod，对吧？）：

```
> kubectl exec hue-learn-56886758d8-fjzdd -- printenv | grep HUE_REMINDERS_SERVICE

HUE_REMINDERS_SERVICE_PORT=80
HUE_REMINDERS_SERVICE_HOST=10.108.163.209  
```

但是使用 DNS 要简单得多。您的服务 DNS 名称是：

```
<service name>.<namespace>.svc.cluster.local
> kubectl exec hue-learn-56886758d8-fjzdd -- nslookup hue-reminders
Server:    10.96.0.10
Address 1: 10.96.0.10 kube-dns.kube-system.svc.cluster.local 
Name:      hue-reminders
Address 1: 10.108.163.209 hue-reminders.default.svc.cluster.local  
```

# 将服务暴露给外部

该服务在集群内可访问。如果您想将其暴露给外部世界，Kubernetes 提供了两种方法：

+   为直接访问配置`NodePort`

+   如果在云环境中运行，请配置云负载均衡器

在为外部访问配置服务之前，您应该确保其安全。Kubernetes 文档中有一个涵盖所有细节的很好的示例：

[`github.com/kubernetes/examples/blob/master/staging/https-nginx/README.md`](https://github.com/kubernetes/examples/blob/master/staging/https-nginx/README.md)。

我们已经在第五章中介绍了原则，“配置 Kubernetes 安全性、限制和账户”。

以下是通过`NodePort`向外界暴露 Hue-reminders 服务的`spec`部分：

```
spec:
  type: NodePort
  ports:
  - port: 8080
    targetPort: 80
    protocol: TCP
    name: http
 - port: 443
   protocol: TCP
   name: https
 selector:
   app: hue-reminders
```

# Ingress

`Ingress`是 Kubernetes 的一个配置对象，它可以让您将服务暴露给外部世界，并处理许多细节。它可以执行以下操作：

+   为您的服务提供外部可见的 URL

+   负载均衡流量

+   终止 SSL

+   提供基于名称的虚拟主机

要使用`Ingress`，您必须在集群中运行一个`Ingress`控制器。请注意，Ingress 仍处于测试阶段，并且有许多限制。如果您在 GKE 上运行集群，那么可能没问题。否则，请谨慎操作。`Ingress`控制器目前的一个限制是它不适用于扩展。因此，它还不是 Hue 平台的一个好选择。我们将在第十章“高级 Kubernetes 网络”中更详细地介绍`Ingress`控制器。

以下是`Ingress`资源的外观：

```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
 name: test
spec:
 rules:
 - host: foo.bar.com
 http:
 paths:
 - path: /foo
 backend:
 serviceName: fooSvc
 servicePort: 80
 - host: bar.baz.com
 http:
 paths:
 - path: /bar
 backend:
 serviceName: barSvc
 servicePort: 80
```

Nginx `Ingress`控制器将解释此`Ingress`请求，并为 Nginx Web 服务器创建相应的配置文件：

```
http { 
  server { 
    listen 80; 
    server_name foo.bar.com; 

    location /foo { 
      proxy_pass http://fooSvc; 
    } 
  } 
  server { 
    listen 80; 
    server_name bar.baz.com; 

    location /bar { 
      proxy_pass http://barSvc; 
    } 
  } 
} 
```

可以创建其他控制器。

# 使用命名空间限制访问

Hue 项目进展顺利，我们有几百个微服务和大约 100 名开发人员和 DevOps 工程师在其中工作。相关微服务组出现，并且您会注意到许多这些组是相当自治的。它们完全不知道其他组。此外，还有一些敏感领域，如健康和财务，您将希望更有效地控制对其的访问。输入命名空间。

让我们创建一个新的服务，Hue-finance，并将其放在一个名为`restricted`的新命名空间中。

这是新的`restricted`命名空间的 YAML 文件：

```
kind: Namespace 
 apiVersion: v1
 metadata:
     name: restricted
     labels:
       name: restricted

> kubectl create -f restricted-namespace.yaml
namespace "restricted" created  
```

创建命名空间后，我们需要为命名空间配置上下文。这将允许限制访问仅限于此命名空间：

```
> kubectl config set-context restricted --namespace=restricted --cluster=minikube --user=minikube
Context "restricted" set.

> kubectl config use-context restricted
Switched to context "restricted". 
```

让我们检查我们的`cluster`配置：

```
> kubectl config view
apiVersion: v1
clusters:
- cluster:
 certificate-authority: /Users/gigi.sayfan/.minikube/ca.crt
 server: https://192.168.99.100:8443
 name: minikube
contexts:
- context:
 cluster: minikube
 user: minikube
 name: minikube
- context:
 cluster: minikube
 namespace: restricted
 user: minikube
 name: restricted
current-context: restricted
kind: Config
preferences: {}
users:
- name: minikube
 user:
 client-certificate: /Users/gigi.sayfan/.minikube/client.crt
 client-key: /Users/gigi.sayfan/.minikube/client.key
```

如您所见，当前上下文是`restricted`。

现在，在这个空的命名空间中，我们可以创建我们的`hue-finance`服务，它将独立存在：

```
> kubectl create -f hue-finance-deployment.yaml
deployment "hue-finance" created

> kubectl get pods
NAME                           READY     STATUS    RESTARTS   AGE
hue-finance-7d4b84cc8d-gcjnz   1/1       Running   0          6s
hue-finance-7d4b84cc8d-tqvr9   1/1       Running   0          6s
hue-finance-7d4b84cc8d-zthdr   1/1       Running   0          6s  
```

不需要切换上下文。您还可以使用`--namespace=<namespace>`和`--all-namespaces`命令行开关。

# 启动作业

Hue 部署了许多长时间运行的微服务进程，但也有许多运行、完成某个目标并退出的任务。Kubernetes 通过作业资源支持此功能。Kubernetes 作业管理一个或多个 pod，并确保它们运行直到成功。如果作业管理的 pod 中的一个失败或被删除，那么作业将运行一个新的 pod 直到成功。

这是一个运行 Python 进程计算 5 的阶乘的作业（提示：它是 120）：

```
apiVersion: batch/v1
kind: Job
metadata:
  name: factorial5
spec:
  template:
    metadata:
      name: factorial5
    spec:
      containers:
      - name: factorial5
        image: python:3.6
        command: ["python", 
                  "-c", 
                  "import math; print(math.factorial(5))"]
      restartPolicy: Never      
```

请注意，`restartPolicy`必须是`Never`或`OnFailure`。默认的`Always`值是无效的，因为作业在成功完成后不应重新启动。

让我们启动作业并检查其状态：

```
> kubectl create -f .\job.yaml
job "factorial5" created

> kubectl get jobs
NAME         DESIRED   SUCCESSFUL   AGE
factorial5   1         1            25s  
```

默认情况下不显示已完成任务的 pod。您必须使用`--show-all`选项：

```
> kubectl get pods --show-all
NAME                           READY     STATUS      RESTARTS   AGE
factorial5-ntp22               0/1       Completed   0          2m
hue-finance-7d4b84cc8d-gcjnz   1/1       Running     0          9m
hue-finance-7d4b84cc8d-tqvr9   1/1       Running     0          8m
hue-finance-7d4b84cc8d-zthdr   1/1       Running     0          9m  
```

`factorial5` pod 的状态为`Completed`。让我们查看它的输出：

```
> kubectl logs factorial5-ntp22
120  
```

# 并行运行作业

您还可以使用并行运行作业。规范中有两个字段，称为`completions`和`parallelism`。`completions`默认设置为`1`。如果您需要多个成功完成，则增加此值。`parallelism`确定要启动多少个 pod。作业不会启动比成功完成所需的更多的 pod，即使并行数更大。

让我们运行另一个只睡眠`20`秒直到完成三次成功的作业。我们将使用`parallelism`因子为`6`，但只会启动三个 pod：

```
apiVersion: batch/v1
kind: Job
metadata:
 name: sleep20
spec:
 completions: 3
 parallelism: 6 
 template:
 metadata:
 name: sleep20
 spec:
 containers:
 - name: sleep20
 image: python:3.6
 command: ["python", 
 "-c", 
 "import time; print('started...'); 
 time.sleep(20); print('done.')"]
 restartPolicy: Never 

> Kubectl get pods 
NAME              READY  STATUS   RESTARTS  AGE
sleep20-1t8sd      1/1   Running    0       10s
sleep20-sdjb4      1/1   Running    0       10s
sleep20-wv4jc      1/1   Running    0       10s
```

# 清理已完成的作业

当作业完成时，它会保留下来 - 它的 pod 也是如此。这是有意设计的，这样您就可以查看日志或连接到 pod 并进行探索。但通常，当作业成功完成后，它就不再需要了。清理已完成的作业及其 pod 是您的责任。最简单的方法是简单地删除`job`对象，这将同时删除所有的 pod：

```
> kubectl delete jobs/factroial5
job "factorial5" deleted
> kubectl delete jobs/sleep20
job "sleep20" deleted  
```

# 安排 cron 作业

Kubernetes cron 作业是在指定时间内运行一次或多次的作业。它们的行为类似于常规的 Unix cron 作业，指定在`/etc/crontab`文件中。

在 Kubernetes 1.4 中，它们被称为`ScheduledJob`。但是，在 Kubernetes 1.5 中，名称更改为`CronJob`。从 Kubernetes 1.8 开始，默认情况下在 API 服务器中启用了`CronJob`资源，不再需要传递`--runtime-config`标志，但它仍处于`beta`阶段。以下是启动一个每分钟提醒您伸展的 cron 作业的配置。在计划中，您可以用`?`替换`*`：

```
apiVersion: batch/v1beta1
kind: CronJob
metadata:
 name: stretch
spec:
 schedule: "*/1 * * * *"
 jobTemplate:
 spec:
 template:
 metadata:
 labels:
 name: stretch 
 spec:
 containers:
 - name: stretch
 image: python
 args:
 - python
 - -c
 - from datetime import datetime; print('[{}] Stretch'.format(datetime.now()))
 restartPolicy: OnFailure
```

在 pod 规范中，在作业模板下，我添加了一个名为`name`的标签。原因是 Kubernetes 会为 cron 作业及其 pod 分配带有随机前缀的名称。该标签允许您轻松发现特定 cron 作业的所有 pod。请参阅以下命令行：

```
> kubectl get pods
NAME                       READY     STATUS              RESTARTS   AGE
stretch-1482165720-qm5bj   0/1       ImagePullBackOff    0          1m
stretch-1482165780-bkqjd   0/1       ContainerCreating   0          6s  
```

请注意，每次调用 cron 作业都会启动一个新的`job`对象和一个新的 pod：

```
> kubectl get jobs
NAME                 DESIRED   SUCCESSFUL   AGE
stretch-1482165300   1         1            11m
stretch-1482165360   1         1            10m
stretch-1482165420   1         1            9m
stretch-1482165480   1         1            8m 
```

当 cron 作业调用完成时，它的 pod 进入`Completed`状态，并且不会在没有`-show-all`或`-a`标志的情况下可见：

```
> Kubectl get pods --show-all
NAME                       READY     STATUS      RESTARTS   AGE
stretch-1482165300-g5ps6   0/1       Completed   0          15m
stretch-1482165360-cln08   0/1       Completed   0          14m
stretch-1482165420-n8nzd   0/1       Completed   0          13m
stretch-1482165480-0jq31   0/1       Completed   0          12m  
```

通常情况下，您可以使用`logs`命令来检查已完成的 cron 作业的 pod 的输出：

```
> kubectl logs stretch-1482165300-g5ps6
[2016-12-19 16:35:15.325283] Stretch 
```

当您删除一个 cron 作业时，它将停止安排新的作业，并删除所有现有的作业对象以及它创建的所有 pod。

您可以使用指定的标签（在本例中名称等于`STRETCH`）来定位由 cron 作业启动的所有作业对象。您还可以暂停 cron 作业，以便它不会创建更多的作业，而无需删除已完成的作业和 pod。您还可以通过设置在 spec 历史限制中管理以前的作业：`spec.successfulJobsHistoryLimit`和`.spec.failedJobsHistoryLimit`。

# 混合非集群组件

Kubernetes 集群中的大多数实时系统组件将与集群外的组件进行通信。这些可能是完全外部的第三方服务，可以通过某些 API 访问，但也可能是在同一本地网络中运行的内部服务，由于各种原因，这些服务不是 Kubernetes 集群的一部分。

这里有两个类别：网络内部和网络外部。为什么这种区别很重要？

# 集群外网络组件

这些组件无法直接访问集群。它们只能通过 API、外部可见的 URL 和公开的服务来访问。这些组件被视为任何外部用户一样。通常，集群组件将只使用外部服务，这不会造成安全问题。例如，在我以前的工作中，我们有一个将异常报告给第三方服务的 Kubernetes 集群（[`sentry.io/welcome/`](https://sentry.io/welcome/)）。这是从 Kubernetes 集群到第三方服务的单向通信。

# 网络内部组件

这些是在网络内部运行但不受 Kubernetes 管理的组件。有许多原因可以运行这些组件。它们可能是尚未 Kubernetized 的传统应用程序，或者是一些不容易在 Kubernetes 内部运行的分布式数据存储。将这些组件运行在网络内部的原因是为了性能，并且与外部世界隔离，以便这些组件和 pod 之间的流量更加安全。作为相同网络的一部分确保低延迟，并且减少了身份验证的需求既方便又可以避免身份验证开销。

# 使用 Kubernetes 管理 Hue 平台

在这一部分，我们将看一下 Kubernetes 如何帮助操作像 Hue 这样的大型平台。Kubernetes 本身提供了许多功能来编排 Pod 和管理配额和限制，检测和从某些类型的通用故障（硬件故障、进程崩溃和无法访问的服务）中恢复。但是，在 Hue 这样一个复杂的系统中，Pod 和服务可能正在运行，但处于无效状态或等待其他依赖项以执行其职责。这很棘手，因为如果一个服务或 Pod 还没有准备好，但已经收到请求，那么你需要以某种方式管理它：失败（将责任放在调用者身上），重试（*多少次？* *多长时间？* *多频繁？*），并排队等待以后（*谁来管理这个队列？*）。

如果整个系统能够意识到不同组件的就绪状态，或者只有当组件真正就绪时才可见，通常会更好。Kubernetes 并不了解 Hue，但它提供了几种机制，如活跃性探针、就绪性探针和 Init Containers，来支持你的集群的应用程序特定管理。

# 使用活跃性探针来确保你的容器是活着的

Kubectl 监视着你的容器。如果容器进程崩溃，Kubelet 会根据重启策略来处理它。但这并不总是足够的。你的进程可能不会崩溃，而是陷入无限循环或死锁。重启策略可能不够微妙。通过活跃性探针，你可以决定何时认为容器是活着的。这里有一个 Hue 音乐服务的 Pod 模板。它有一个`livenessProbe`部分，使用了`httpGet`探针。HTTP 探针需要一个方案（HTTP 或 HTTPS，默认为 HTTP），一个主机（默认为`PodIp`），一个`path`和一个`port`。如果 HTTP 状态码在`200`到`399`之间，探针被认为是成功的。你的容器可能需要一些时间来初始化，所以你可以指定一个`initialDelayInSeconds`。在这段时间内，Kubelet 不会进行活跃性检查：

```
apiVersion: v1
kind: Pod
metadata:
 labels:
 app: hue-music
 name: hue-music
spec:
 containers:
 image: the_g1g1/hue-music
 livenessProbe:
 httpGet:
 path: /pulse
 port: 8888
 httpHeaders:
 - name: X-Custom-Header
 value: ItsAlive
 initialDelaySeconds: 30
 timeoutSeconds: 1
 name: hue-music
```

如果任何容器的活跃性探针失败，那么 Pod 的重启策略就会生效。确保你的重启策略不是*Never*，因为那会使探针变得无用。

有两种其他类型的探针：

+   `TcpSocket`：只需检查端口是否打开

+   `Exec`：运行一个返回`0`表示成功的命令

# 使用就绪性探针来管理依赖关系

就绪探针用于不同的目的。您的容器可能已经启动运行，但可能依赖于此刻不可用的其他服务。例如，Hue-music 可能依赖于访问包含您听歌历史记录的数据服务。如果没有访问权限，它将无法执行其职责。在这种情况下，其他服务或外部客户端不应该向 Hue 音乐服务发送请求，但没有必要重新启动它。就绪探针解决了这种情况。当一个容器的就绪探针失败时，该容器的 pod 将从其注册的任何服务端点中移除。这确保请求不会涌入无法处理它们的服务。请注意，您还可以使用就绪探针暂时移除过载的 pod，直到它们排空一些内部队列。

这是一个示例就绪探针。我在这里使用 exec 探针来执行一个 `custom` 命令。如果命令退出时的退出代码为非零，容器将被关闭：

```
readinessProbe:
 exec:
 command: 
 - /usr/local/bin/checker
 - --full-check
 - --data-service=hue-multimedia-service
 initialDelaySeconds: 60
 timeoutSeconds: 5
```

在同一个容器上同时拥有就绪探针和存活探针是可以的，因为它们有不同的用途。

# 使用初始化容器进行有序的 pod 启动

存活和就绪探针非常好用。它们认识到，在启动时，可能会有一个容器尚未准备好的时间段，但不应被视为失败。为了适应这一点，有一个 `initialDelayInSeconds` 设置，容器在这段时间内不会被视为失败。但是，如果这个初始延迟可能非常长呢？也许，在大多数情况下，一个容器在几秒钟后就准备好处理请求了，但是因为初始延迟设置为五分钟以防万一，当容器处于空闲状态时，我们会浪费很多时间。如果容器是高流量服务的一部分，那么在每次升级后，许多实例都可能在五分钟后处于空闲状态，几乎使服务不可用。

初始化容器解决了这个问题。一个 pod 可能有一组初始化容器，在其他容器启动之前完成运行。初始化容器可以处理所有非确定性的初始化，并让应用容器通过它们的就绪探针尽量减少延迟。

初始化容器在 Kubernetes 1.6 中退出了 beta 版。您可以在 pod 规范中指定它们，作为 `initContainers` 字段，这与 `containers` 字段非常相似。以下是一个示例：

```
apiVersion: v1
kind: Pod
metadata:
 name: hue-fitness
spec:
 containers: 
 name: hue-fitness
 Image: hue-fitness:v4.4
 InitContainers:
 name: install
 Image: busybox
 command: /support/safe_init
 volumeMounts:
 - name: workdir
 mountPath: /workdir   
```

# 与 DaemonSet pods 共享

`DaemonSet` pods 是自动部署的 pod，每个节点一个（或指定节点的子集）。它们通常用于监视节点并确保它们正常运行。这是一个非常重要的功能，我们在第三章“监控、日志记录和故障排除”中讨论了节点问题检测器。但它们可以用于更多的功能。默认的 Kubernetes 调度程序的特性是根据资源可用性和请求来调度 pod。如果有很多不需要大量资源的 pod，许多 pod 将被调度到同一个节点上。让我们考虑一个执行小任务的 pod，然后，每秒钟，将其所有活动的摘要发送到远程服务。想象一下，平均每秒钟会有 50 个这样的 pod 被调度到同一个节点上。这意味着，每秒钟，50 个 pod 会进行 50 次几乎没有数据的网络请求。我们能不能将它减少 50 倍，只进行一次网络请求？使用`DaemonSet` pod，其他 50 个 pod 可以与其通信，而不是直接与远程服务通信。`DaemonSet` pod 将收集来自 50 个 pod 的所有数据，并且每秒钟将其汇总报告给远程服务。当然，这需要远程服务 API 支持汇总报告。好处是 pod 本身不需要修改；它们只需配置为与`DaemonSet` pod 在本地主机上通信，而不是与远程服务通信。`DaemonSet` pod 充当聚合代理。

这个配置文件的有趣之处在于，`hostNetwork`、`hostPID`和`hostIPC`选项都设置为`true`。这使得 pod 能够有效地与代理通信，利用它们在同一物理主机上运行的事实。

```
apiVersion: apps/v1
kind: DaemonSet
metadata:
 name: hue-collect-proxy
 labels:
 tier: stats
 app: hue-collect-proxy
spec:
 template:
 metadata:
 labels:
 hue-collect-proxy
 spec:
 hostPID: true
 hostIPC: true
 hostNetwork: true
 containers:
 image: the_g1g1/hue-collect-proxy
 name: hue-collect-proxy
```

# 使用 Kubernetes 发展 Hue 平台

在本节中，我们将讨论扩展 Hue 平台和服务其他市场和社区的其他方法。问题始终是，“我们可以使用哪些 Kubernetes 功能和能力来解决新的挑战或要求？”

# 在企业中利用 Hue

企业通常无法在云中运行，要么是因为安全和合规性原因，要么是因为性能原因，因为系统必须处理数据和传统系统，这些系统不适合迁移到云上。无论哪种情况，企业的 Hue 必须支持本地集群和/或裸金属集群。

虽然 Kubernetes 最常部署在云上，甚至有一个特殊的云提供商接口，但它并不依赖于云，可以在任何地方部署。它需要更多的专业知识，但已经在自己的数据中心上运行系统的企业组织拥有这方面的专业知识。

CoreOS 提供了大量关于在裸机集群上部署 Kubernetes 集群的材料。

# 用 Hue 推动科学的进步

Hue 在整合来自多个来源的信息方面非常出色，这对科学界将是一个福音。想象一下 Hue 如何帮助来自不同领域的科学家进行多学科合作。

科学社区的网络可能需要在多个地理分布的集群上部署。这就是集群联邦。Kubernetes 考虑到了这种用例，并不断发展其支持。我们将在后面的章节中详细讨论这个问题。

# 用 Hue 教育未来的孩子

Hue 可以用于教育，并为在线教育系统提供许多服务。但隐私问题可能会阻止将 Hue 作为单一的集中系统用于儿童。一个可能的选择是建立一个单一的集群，为不同学校设立命名空间。另一个部署选项是每个学校或县都有自己的 Hue Kubernetes 集群。在第二种情况下，Hue 教育必须非常易于操作，以满足没有太多技术专长的学校。Kubernetes 可以通过提供自愈和自动扩展功能来帮助 Hue，使其尽可能接近零管理。

# 总结

在本章中，我们设计和规划了 Hue 平台的开发、部署和管理——一个想象中的全知全能的服务，建立在微服务架构上。当然，我们使用 Kubernetes 作为底层编排平台，并深入探讨了许多它的概念和资源。特别是，我们专注于为长期运行的服务部署 pod，而不是为启动短期或定期作业部署作业，探讨了内部服务与外部服务，还使用命名空间来分割 Kubernetes 集群。然后，我们研究了像活跃性和就绪性探针、初始化容器和守护进程集这样的大型系统的管理。

现在，您应该能够设计由微服务组成的 Web 规模系统，并了解如何在 Kubernetes 集群中部署和管理它们。

在下一章中，我们将深入研究存储这个非常重要的领域。数据为王，但通常是系统中最不灵活的元素。Kubernetes 提供了一个存储模型，并提供了许多与各种存储解决方案集成的选项。
