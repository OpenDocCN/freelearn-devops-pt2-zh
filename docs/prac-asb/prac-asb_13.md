# 第十章：容器和云管理

Ansible 是一个非常灵活的自动化工具，可以轻松用于自动化基础架构的任何方面。在过去几年中，基于容器的工作负载和云工作负载变得越来越受欢迎，因此我们将看看如何使用 Ansible 自动化与这些工作负载相关的任务。在本章中，我们将首先使用 Ansible 设计和构建容器。然后，我们将看看如何运行这些容器，最后，我们将看看如何使用 Ansible 管理各种云平台。

具体来说，本章将涵盖以下主题：

+   使用 playbooks 设计和构建容器

+   管理多个容器平台

+   使用 Ansible 自动化 Docker

+   探索面向容器的模块

+   针对亚马逊网络服务的自动化

+   通过自动化来补充谷歌云平台

+   与 Azure 的无缝自动化集成

+   通过 Rackspace Cloud 扩展您的环境

+   使用 Ansible 编排 OpenStack

让我们开始吧！

# 技术要求

本章假设您已经按照第一章 *开始使用 Ansible*中详细说明的方式设置了 Ansible 的控制主机，并且正在使用最新版本——本章的示例是在 Ansible 2.9 版本下测试的。尽管本章将给出特定的主机名示例，但您可以自由地用您自己的主机名和/或 IP 地址替换它们。如何做到这一点的细节将在适当的地方提供。本章还假设您可以访问 Docker 主机，尽管在大多数操作系统上都可以安装 Docker，但本章提供的所有命令都是针对 GNU/Linux 的，并且仅在该平台上进行了测试。

本章中的所有示例都可以在本书的 GitHub 存储库中找到：[`github.com/PacktPublishing/Practical-Ansible-2/tree/master/Chapter%2010`](https://github.com/PacktPublishing/Practical-Ansible-2/tree/master/Chapter%2010)。

# 使用 playbooks 设计和构建容器

使用 Dockerfile 构建容器可能是最常见的方法，但这并不意味着这是最好的方法。

首先，即使您在自动化路径上处于非常良好的位置，并且为您的基础架构编写了许多 Ansible 角色，您也无法在 Dockerfile 中利用它们，因此您最终会复制您的工作来创建容器。除了需要花费时间和需要学习一种新语言之外，公司很少能够在一夜之间放弃他们的基础架构并转向容器。这意味着您需要保持两个相同的自动化部分处于活动状态并保持最新，从而使自己处于犯错误和环境之间不一致行为的位置。

如果这还不够成问题，当您开始考虑云环境时，情况很快就会恶化。所有云环境都有自己的控制平面和本地自动化语言，因此在很短的时间内，您会发现自己一遍又一遍地重写相同操作的自动化，从而浪费时间并破坏环境的一致性。

Ansible 提供了`ansible-container`，以便您可以使用与创建机器相同的组件来创建容器。您应该做的第一件事是确保您已经安装了`ansible-container`。有几种安装它的方法，但最简单的方法是使用`pip`。为此，您可以运行以下命令：

```
$ sudo pip install ansible-container[docker,k8s]
```

`ansible-container`工具在撰写本文时带有三个支持的引擎：

+   `docker`：如果您想要在 Docker 引擎（即在您的本地机器上）中使用它，就需要它。

+   `k8s`：如果您想要在 Kubernetes 集群中使用它，无论是本地（即 MiniKube）还是远程（即生产集群）都需要它。

+   `openshift`：如果你想要在 OpenShift 集群中使用它，无论是本地（即 MiniShift）还是远程（即生产集群）。

按照以下步骤使用 playbooks 构建容器：

1.  发出`ansible-container init`命令将给我们以下输出：

```
$ ansible-container init
Ansible Container initialized.
```

运行这个命令还将创建以下文件：

+   +   `ansible.cfg`：一个空文件，用于（最终）用于覆盖 Ansible 系统配置

+   `ansible-requirements.txt`：一个空文件，用于（最终）列出构建过程中容器的 Python 要求

+   `container.yml`：一个包含构建的 Ansible 代码的文件

+   `meta.yml`：一个包含 Ansible Galaxy 元数据的文件

+   `requirements.yml`：一个空文件，用于（最终）列出构建所需的 Ansible 角色

1.  让我们尝试使用这个工具构建我们自己的容器-用以下内容替换`container.yml`的内容：

```
version: "2"
settings:
  conductor:
    base: centos:7
  project_name: http-server
services:
  web:
    from: "centos:7"
    roles:
      - geerlingguy.apache
    ports:
      - "80:80"
    command:
      - "/usr/bin/dumb-init"
      - "/usr/sbin/apache2ctl"
      - "-D"
      - "FOREGROUND"
    dev_overrides:
      environment:
        - "DEBUG=1"
```

现在我们可以运行`ansible-container build`来启动构建。

在构建过程结束时，我们将得到一个应用了`geerlingguy.apache`角色的容器。`ansible-container`工具执行多阶段构建能力，启动一个用于构建真实容器的 Ansible 容器。

如果我们指定了多个要应用的角色，输出将是一个具有更多层的图像，因为 Ansible 将为每个指定的角色创建一个层。这样，容器可以很容易地使用现有的 Ansible 角色构建，而不是 Dockerfiles。

现在你已经学会了如何使用 playbooks 设计和构建容器，接下来你将学习如何管理多个容器平台。

# 管理多个容器平台

在今天的世界中，仅仅能够运行一个镜像并不被认为是一个可以投入生产的设置。

要能够称呼一个部署为“投入生产使用”，你需要能够证明你的应用程序提供的服务将在合理的情况下运行，即使是在单个应用程序崩溃或硬件故障的情况下。通常，你的客户会有更多的可靠性约束。

幸运的是，你的软件并不是唯一具有这些要求的数据，因此为此目的开发了编排解决方案。

今天，最成功的一个是 Kubernetes，因为它有各种不同的发行版/版本，所以我们将主要关注它。

Kubernetes 的理念是，你告诉 Kubernetes 控制平面你想要 X 个实例的 Y 应用程序，Kubernetes 将计算运行在 Kubernetes 节点上的 Y 应用程序实例数量，以确保实例数量为 X。如果实例太少，Kubernetes 将负责启动更多实例，而如果实例太多，多余的实例将被停止。

由于 Kubernetes 不断检查所请求的实例数量是否在运行中，所以在应用程序失败或节点失败的情况下，Kubernetes 将重新启动丢失的实例。

由于安装和管理 Kubernetes 的复杂性，多家公司已经开始销售简化其运营的 Kubernetes 发行版，并且他们愿意提供支持。

目前使用最广泛的发行版是 OpenShift：红帽 Kubernetes 发行版。

为了简化开发人员和运维团队的生活，Ansible 提供了`ansible-container`，正如我们在前一节中所看到的，这是一个用于创建容器以及支持容器整个生命周期的工具。

# 使用 ansible-container 部署到 Kubernetes

让我们学习如何运行刚刚用`ansible-container`构建的镜像。

首先，我们需要镜像本身，你应该已经有了，因为这是上一节的输出！

我们假设您可以访问用于测试的 Kubernetes 或 OpenShift 集群。设置这些超出了本书的范围，因此您可能希望查看 Minikube 或 Minishift 等分发版，这两者都旨在快速且易于设置，以便您可以快速开始学习这些技术。我们还需要正确配置`kubectl`客户端或`oc`客户端，根据我们部署的是 Kubernetes 还是 OpenShift。让我们开始吧：

1.  要将应用程序部署到集群，您需要更改`container.yml`文件，以便添加一些附加信息。更具体地说，我们需要添加一个名为`settings`和一个名为`k8s_namespace`的部分来声明我们的部署设置。此部分将如下所示：

```
k8s_namespace:
  name: http-server
  description: An HTTP server
  display_name: HTTP server
```

1.  现在我们已经添加了关于 Kubernetes 部署的必要信息，我们可以继续部署：

```
$ ansible-container --engine kubernetes deploy
```

一旦 Ansible 完成执行，您将能够在 Kubernetes 集群上找到`http-server`部署。

幕后发生的是，Ansible 有一组模块（其名称通常以`k8s`开头），用于驱动 Kubernetes 集群，并使用它们自动部署应用程序。

根据我们在上一节中构建的图像和本节开头添加的其他信息，Ansible 能够填充部署模板，然后使用`k8s`模块部署它。

现在您已经学会了如何在 Kubernetes 集群上部署容器，接下来您将学习如何使用 Ansible 与 Kubernetes 集群进行交互。

# 使用 Ansible 管理 Kubernetes 对象

现在您已经使用`ansible-container`部署了第一个应用程序，与该应用程序进行交互将非常有用。获取有关 Kubernetes 对象状态的信息或部署应用程序，更一般地与 Kubernetes API 进行交互，而无需使用`ansible-containers`。

# 安装 Ansible Kubernetes 依赖项

首先，您需要安装 Python `openshift`包（您可以通过 pip 或操作系统的打包系统安装它）。

我们现在准备好我们的第一个 Kubernetes playbook 了！

# 使用 Ansible 列出 Kubernetes 命名空间

Kubernetes 集群在内部有多个命名空间，通常可以使用`kubectl get namespaces`找到集群中的命名空间。您可以通过创建一个名为`k8s-ns-show.yaml`的文件来使用 Ansible 执行相同的操作，内容如下：

```
---
- hosts: localhost
  tasks:
    - name: Get information from K8s
      k8s_info:
        api_version: v1
        kind: Namespace
      register: ns
    - name: Print info
      debug:
        var: ns
```

我们现在可以执行此操作，如下所示：

```
$ ansible-playbook k8s-ns-show.yaml
```

现在您将在输出中看到有关命名空间的信息。

请注意代码的第七行（`kind: Namespace`），我们在其中指定了我们感兴趣的资源类型。您可以指定其他 Kubernetes 对象类型以查看它们（例如，您可以尝试使用部署、服务和 Pod 进行此操作）。

# 使用 Ansible 创建 Kubernetes 命名空间

到目前为止，我们已经学会了如何显示现有的命名空间，但通常，Ansible 被用于以声明方式实现期望的状态。因此，让我们创建一个名为`k8s-ns.yaml`的新的 playbook，内容如下：

```
---
- hosts: localhost
  tasks:
    - name: Ensure the myns namespace exists
      k8s:
        api_version: v1
        kind: Namespace
        name: myns
        state: present
```

在运行之前，我们可以执行`kubectl get ns`，以确保`myns`不存在。在我的情况下，输出如下：

```
$ kubectl get ns
NAME STATUS AGE
default Active 69m
kube-node-lease Active 69m
kube-public Active 69m
kube-system Active 69m
```

我们现在可以使用以下命令运行 playbook：

```
$ ansible-playbook k8s-ns.yaml
```

输出应该类似于以下内容：

```
PLAY [localhost] *******************************************************************

TASK [Gathering Facts] *************************************************************
ok: [localhost]

TASK [Ensure the myns namespace exists] ********************************************
changed: [localhost]

PLAY RECAP *************************************************************************
localhost : ok=2 changed=1 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
```

正如您所看到的，Ansible 报告说它改变了命名空间的状态。如果我再次执行`kubectl get ns`，很明显 Ansible 创建了我们期望的命名空间：

```
$ kubectl get ns
NAME STATUS AGE
default Active 74m
kube-node-lease Active 74m
kube-public Active 74m
kube-system Active 74m
myns Active 22s 
```

现在，让我们创建一个服务。

# 使用 Ansible 创建 Kubernetes 服务

到目前为止，我们已经看到了如何从 Ansible 创建命名空间，现在让我们在刚刚创建的命名空间中放置一个服务。让我们创建一个名为`k8s-svc.yaml`的新 playbook，内容如下：

```
---
- hosts: localhost
  tasks:
    - name: Ensure the Service mysvc is present
      k8s:
        state: present
        definition:
          apiVersion: v1
          kind: Service
          metadata:
            name: mysvc
            namespace: myns
          spec:
            selector:
              app: myapp
              service: mysvc
            ports:
              - protocol: TCP
                targetPort: 800
                name: port-80-tcp
                port: 80
```

在运行之前，我们可以执行`kubectl get svc`来确保命名空间中没有服务。在运行之前，请确保您在正确的命名空间中！在我的情况下，输出如下：

```
$ kubectl get svc
No resources found in myns namespace.
```

我们现在可以使用以下命令运行它：

```
$ ansible-playbook k8s-svc.yaml
```

输出应该类似于以下内容：

```
PLAY [localhost] *******************************************************************

TASK [Gathering Facts] *************************************************************
ok: [localhost]

TASK [Ensure the myns namespace exists] ********************************************
changed: [localhost]

PLAY RECAP *************************************************************************
localhost : ok=2 changed=1 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0 
```

正如您所看到的，Ansible 报告说它改变了服务状态。如果我再次执行`kubectl get svc`，很明显 Ansible 创建了我们期望的服务：

```
$ kubectl get svc
NAME TYPE CLUSTER-IP EXTERNAL-IP PORT(S) AGE
mysvc ClusterIP 10.0.0.84 <none> 80/TCP 10s
```

正如您所看到的，我们遵循了在命名空间情况下使用的相同过程，但是我们指定了不同的 Kubernetes 对象类型，并指定了所需的服务类型的各种参数。您可以对所有其他 Kubernetes 对象类型执行相同的操作。

现在您已经学会了如何处理 Kubernetes 集群，您将学习如何使用 Ansible 自动化 Docker。

# 使用 Ansible 自动化 Docker

Docker 现在是一个非常常见和普遍的工具。在生产中，它通常由编排器管理（或者至少应该在大多数情况下），但在开发中，环境通常直接使用。

使用 Ansible，您可以轻松管理您的 Docker 实例。

由于我们将要管理一个 Docker 实例，我们需要确保我们手头有一个，并且我们机器上的`docker`命令已经正确配置。我们需要这样做以确保在终端上运行`docker images`足够。假设您得到类似以下的结果：

```
REPOSITORY TAG IMAGE ID CREATED SIZE
```

这意味着一切都正常工作。如果您已经克隆了镜像，可能会提供更多行作为输出。

另一方面，假设它返回了类似于这样的东西：

```
Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?
```

这意味着我们没有运行 Docker 守护程序，或者我们的 Docker 控制台配置不正确。

此外，确保您有`docker` Python 模块是很重要的，因为 Ansible 将尝试使用它与 Docker 守护程序进行通信。让我们来看一下：

1.  首先，我们需要创建一个名为`start-docker-container.yaml`的 playbook，其中包含以下代码：

```
---
- hosts: localhost
  tasks:
    - name: Start a container with a command
      docker_container:
        name: test-container
        image: alpine
        command:
          - echo
          - "Hello, World!"
```

1.  现在我们有了 Ansible playbook，我们只需要执行它：

```
$ ansible-playbook start-docker-container.yaml
```

正如您可能期望的那样，它将给您一个类似于以下的输出：

```
PLAY [localhost] *********************************************************************

TASK [Gathering Facts] ***************************************************************
ok: [localhost]

TASK [Start a container with a command] **********************************************
changed: [localhost]

PLAY RECAP ***************************************************************************
localhost : ok=2 changed=1 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0  
```

1.  现在我们可以检查我们的命令是否正确执行，如下所示：

```
$ docker container list -a
```

这将显示运行的容器：

```
CONTAINER ID IMAGE  COMMAND              CREATED       STATUS                        PORTS NAMES
c706ec55fc0d alpine "echo Hello, World!" 3 minutes ago Exited (0) About a minute ago       test-container
```

这证明了一个容器被执行。

要检查`echo`命令是否被执行，我们可以运行以下代码：

```
$ docker logs c706ec55fc0d
```

这将返回以下输出：

```
Hello, World!
```

在本节中，我们执行了`docker_container`模块。这不是 Ansible 用来控制 Docker 守护程序的唯一模块，但它可能是最广泛使用的模块之一，因为它用于控制在 Docker 上运行的容器。

其他模块包括以下内容：

+   `docker_config`：用于更改 Docker 守护程序的配置

+   `docker_container_info`：用于从容器中收集信息（检查）

+   `docker_network`：用于管理 Docker 网络配置

还有许多以`docker_`开头的模块，但实际上用于管理 Docker Swarm 集群，而不是 Docker 实例。一些示例如下：

+   `docker_node`：用于管理 Docker Swarm 集群中的节点

+   `docker_node_info`：用于检索 Docker Swarm 集群中特定节点的信息

+   `docker_swarm_info`：用于检索有关 Docker Swarm 集群的信息

正如我们将在下一节中看到的，还有许多模块可以用来管理以各种方式编排的容器。

现在您已经学会了如何使用 Ansible 自动化 Docker，您将探索面向容器的模块。

# 探索面向容器的模块

通常，当组织发展壮大时，他们开始在组织的不同部分使用多种技术。通常发生的另一件事是，部门发现某个供应商对他们很有效后，他们更倾向于尝试该供应商提供的新技术。这两个因素的混合以及时间（通常情况下，技术周期较少）最终会在同一组织内为同一个问题创建多种解决方案。

如果您的组织处于这种容器的情况下，Ansible 可以帮助您，因为它能够与大多数，如果不是所有的容器平台进行互操作。

很多时候，使用 Ansible 的最大问题是找到你需要使用的模块的名称，以实现你想要实现的目标。在本节中，我们将尝试帮助您解决这个问题，主要是在容器化领域，但这可能也有助于您寻找不同类型的模块。

所有 Ansible 模块研究的起点应该是模块索引（[`docs.ansible.com/ansible/latest/modules/modules_by_category.html`](https://docs.ansible.com/ansible/latest/modules/modules_by_category.html)）。通常情况下，您可以找到一个明显与您寻找的内容匹配的类别，但情况并非总是如此。

容器是这种例外情况之一（至少在撰写本文时是如此），因此没有“容器”类别。解决方案是转到所有模块。从这里，您可以使用浏览器的内置功能进行搜索（通常可以通过使用*Ctrl*+*F*来实现），以找到可能与包名称或简短描述匹配的字符串。

Ansible 中的每个模块都属于一个类别，但很多时候，模块适用于多个类别，因此并不总是容易找到它们。

例如，许多与容器服务相关的 Ansible 模块属于云模块类别（ECS、Docker、LXC、LXD 和 Podman），而其他模块属于集群模块类别（Kubernetes、OpenShift 等）。

为了进一步帮助您，让我们来看一下一些主要的容器平台和 Ansible 提供的主要模块。

2014 年，亚马逊网络服务推出了弹性容器服务（ECS），这是一种在其基础设施中部署和编排 Docker 容器的方式。在接下来的一年，亚马逊网络服务还推出了弹性容器注册表（ECR），这是一个托管的 Docker 注册表服务。该服务并没有像 AWS 希望的那样普及，因此在 2018 年，AWS 推出了弹性 Kubernetes 服务（EKS），以允许希望在 AWS 上运行 Kubernetes 的人拥有一个托管服务。如果您正在使用或计划使用 EKS，这只是一个标准的托管 Kubernetes 集群，因此您可以使用我们即将介绍的 Kubernetes 特定模块。如果您决定使用 ECS，有几个模块可以帮助您。最重要的是`ecs_cluster`，它允许您创建或终止 ECS 集群；`ecs_ecr`，它允许您管理 ECR；`ecs_service`，它允许您在 ECS 中创建、终止、启动或停止服务；以及`ecs_task`，它允许您在 ECS 中运行、启动或停止任务。除此之外，还有`ecs_service_facts`，它允许 Ansible 列出或描述 ECS 中的服务。

2018 年，微软 Azure 宣布了 Azure 容器服务（ACS），然后宣布了 Azure Kubernetes 服务（AKS）。这些服务由 Kubernetes 解决方案管理，因此可以使用 Kubernetes 模块来管理。除此之外，Ansible 还提供了两个特定的模块：`azure_rm_acs`模块允许我们创建、更新和删除 Azure 容器服务实例，而`azure_rm_aks`模块允许我们创建、更新和删除 Azure Kubernetes 服务实例。

Google Cloud 于 2015 年推出了**Google Kubernetes Engine**（**GKE**）。GKE 是 Google Cloud Platform 的托管 Kubernetes 版本，因此与 Ansible Kubernetes 模块兼容。除此之外，还有各种 GKE 特定的模块，其中一些如下：

+   `gcp_container_cluster`：允许您创建 GCP Cluster

+   `gcp_container_cluster_facts`：允许您收集有关 GCP Cluster 的信息

+   `gcp_container_node_pool`：允许您创建 GCP NodePool

+   `gcp_container_node_pool_facts`：允许您收集有关 GCP NodePool 的信息

Red Hat 于 2011 年启动了 OpenShift，当时它是基于自己的容器运行时的。在 2015 年发布的第 3 版中，它完全基于 Kubernetes 重新构建，因此所有 Ansible Kubernetes 模块都可以使用。除此之外，还有`oc`模块，目前仍然存在但处于弃用状态，更倾向于使用 Kubernetes 模块。

2015 年，Google 发布了 Kubernetes，并迅速形成了一个庞大的社区。Ansible 允许您使用一些模块来管理您的 Kubernetes 集群：

+   `k8s`：允许您管理任何类型的 Kubernetes 对象

+   `k8s_auth`：允许您对需要显式登录的 Kubernetes 集群进行身份验证

+   `k8s_facts`：允许您检查 Kubernetes 对象

+   `k8s_scale`：允许您为部署、副本集、复制控制器或作业设置新的大小

+   `k8s_service`：允许您在 Kubernetes 上管理服务

LXC 和 LXD 也是可以在 Linux 中运行容器的系统。由于以下模块的支持，这些系统也受到 Ansible 的支持：

+   `lxc_container`：允许您管理 LXC 容器

+   `lxd_container`：允许您管理 LXD 容器

+   `lxd_profile`：允许您管理 LXD 配置文件

现在您已经学会了如何探索面向容器的模块，接下来将学习如何针对亚马逊网络服务进行自动化。

# 针对亚马逊网络服务进行自动化

在许多组织中，广泛使用云提供商，而在其他组织中，它们只是被引入。但是，无论如何，您可能都必须处理一些云提供商来完成工作。AWS 是最大的，也是最古老的，可能是您必须使用的东西。

# 安装

要能够使用 Ansible 自动化您的 Amazon Web Service 资源，您需要安装`boto`库。要执行此操作，请运行以下命令：

```
$ pip install boto
```

现在您已经安装了所有必要的软件，可以设置身份验证了。

# 身份验证

`boto`库在`~/.aws/credentials`文件中查找必要的凭据。确保凭据文件正确配置有两种不同的方法。

可以使用 AWS CLI 工具。或者，可以通过创建具有以下结构的文件来使用您选择的文本编辑器：

```
[default] aws_access_key_id = [YOUR_KEY_HERE] aws_secret_access_key = [YOUR_SECRET_ACCESS_KEY_HERE]
```

现在您已经创建了具有必要凭据的文件，`boto`将能够针对您的 AWS 环境进行操作。由于 Ansible 对 AWS 系统的每一次通信都使用`boto`，这意味着即使您不必更改任何特定于 Ansible 的配置，Ansible 也将被适当配置。

# 创建您的第一台机器

现在 Ansible 能够连接到您的 AWS 环境，您可以按照以下步骤进行实际的 playbook：

1.  创建具有以下内容的`aws.yaml` Playbook：

```
---
- hosts: localhost
  tasks:
    - name: Ensure key pair is present
      ec2_key:
        name: fale
        key_material: "{{ lookup('file', '~/.ssh/fale.pub') }}"
    - name: Gather information of the EC2 VPC net in eu-west-1
      ec2_vpc_net_facts:
        region: eu-west-1
      register: aws_simple_net
    - name: Gather information of the EC2 VPC subnet in eu-west-1
      ec2_vpc_subnet_facts:
        region: eu-west-1
        filters:
          vpc-id: '{{ aws_simple_net.vpcs.0.id }}'
      register: aws_simple_subnet
    - name: Ensure wssg Security Group is present
      ec2_group:
        name: wssg
        description: Web Security Group
        region: eu-west-1
        vpc_id: '{{ aws_simple_net.vpcs.0.id }}'
        rules:
          - proto: tcp
            from_port: 22
            to_port: 22
            cidr_ip: 0.0.0.0/0
          - proto: tcp
            from_port: 80
            to_port: 80
            cidr_ip: 0.0.0.0/0
          - proto: tcp
            from_port: 443
            to_port: 443
            cidr_ip: 0.0.0.0/0
        rules_egress:
          - proto: all
            cidr_ip: 0.0.0.0/0
      register: aws_simple_wssg
    - name: Setup instance
      ec2:
        assign_public_ip: true
        image: ami-3548444c
        region: eu-west-1
        exact_count: 1
        key_name: fale
        count_tag:
          Name: ws01.ansible2cookbook.com
        instance_tags:
          Name: ws01.ansible2cookbook.coms
        instance_type: t2.micro
        group_id: '{{ aws_simple_wssg.group_id }}'
        vpc_subnet_id: '{{ aws_simple_subnet.subnets.0.id }}'
        volumes:
          - device_name: /dev/sda1
            volume_type: gp2
            volume_size: 10
            delete_on_termination: True
```

1.  使用以下命令运行它：

```
$ ansible-playbook aws.yaml
```

此命令将返回类似以下内容：

```
PLAY [localhost] **********************************************************************************

TASK [Gathering Facts] ****************************************************************************
ok: [localhost]

TASK [Ensure key pair is present] *****************************************************************
ok: [localhost]

TASK [Gather information of the EC2 VPC net in eu-west-1] *****************************************
ok: [localhost]

TASK [Gather information of the EC2 VPC subnet in eu-west-1] **************************************
ok: [localhost]

TASK [Ensure wssg Security Group is present] ******************************************************
ok: [localhost]

TASK [Setup instance] *****************************************************************************
changed: [localhost]

PLAY RECAP ****************************************************************************************
localhost : ok=6 changed=1 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0 
```

如果您检查 AWS 控制台，您将看到现在有一台机器正在运行！

要在 AWS 中启动虚拟机，我们需要准备一些东西，如下所示：

+   一个 SSH 密钥对

+   一个网络

+   一个子网络

+   一个安全组

默认情况下，您的帐户中已经有一个网络和一个子网络，但您需要检索它们的 ID。

这就是为什么我们首先将 SSH 密钥对的公共部分上传到 AWS，然后查询有关网络和子网络的信息，然后确保我们想要使用的安全组存在，最后触发机器构建。

现在您已经学会了如何针对亚马逊网络服务进行自动化，您将学习如何通过自动化来补充谷歌云平台。

# 通过自动化来补充谷歌云平台

另一个全球云服务提供商是谷歌，其谷歌云平台。谷歌对云的方法与其他提供商的方法相对不同，因为谷歌不试图在虚拟环境中模拟数据中心。这是因为谷歌希望重新思考云提供的概念，以简化它。

# 安装

在您开始使用 Ansible 与谷歌云平台之前，您需要确保已安装适当的组件。具体来说，您需要 Python 的`requests`和`google-auth`模块。要安装这些模块，请运行以下命令：

```
$ pip install requests google-auth
```

现在您已经准备好所有依赖项，可以开始认证过程。

# 认证

在谷歌云平台中获得工作凭据的两种不同方法：

+   服务帐户

+   机器帐户

第一种方法在大多数情况下是建议的，因为第二种方法仅适用于在谷歌云平台环境中直接运行 Ansible 的情况。

创建服务帐户后，您应该设置以下环境变量：

+   `GCP_AUTH_KIND`

+   `GCP_SERVICE_ACCOUNT_EMAIL`

+   `GCP_SERVICE_ACCOUNT_FILE`

+   `GCP_SCOPES`

现在，Ansible 可以使用适当的服务帐户。

第二种方法是最简单的，因为如果您在谷歌云实例中运行 Ansible，它将能够自动检测到机器帐户。

# 创建您的第一台机器

现在 Ansible 能够连接到您的 GCP 环境，您可以继续进行实际的 Playbook：

1.  创建`gce.yaml` Playbook，并包含以下内容：

```
---
- hosts: localhost
  tasks:
    - name: create a instance
      gcp_compute_instance:
        name: TestMachine
        machine_type: n1-standard-1
        disks:
        - auto_delete: 'true'
          boot: 'true'
          initialize_params:
            source_image: family/centos-7
            disk_size_gb: 10
        zone: eu-west1-c
        auth_kind: serviceaccount
        service_account_file: "~/sa.json"
        state: present
```

使用以下命令执行它：

```
$ ansible-playbook gce.yaml
```

这将创建以下输出：

```
PLAY [localhost] **********************************************************************************

TASK [Gathering Facts] ****************************************************************************
ok: [localhost]

TASK [create a instance] **************************************************************************
changed: [localhost]

PLAY RECAP ****************************************************************************************
localhost : ok=2 changed=1 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0 
```

与 AWS 示例一样，在云中运行机器对 Ansible 来说非常容易。

在 GCE 的情况下，您不需要预先设置网络，因为 GCE 默认设置将提供一个功能齐全的机器。

与 AWS 一样，您可以使用的模块列表非常庞大。您可以在[`docs.ansible.com/ansible/latest/modules/list_of_cloud_modules.html#google`](https://docs.ansible.com/ansible/latest/modules/list_of_cloud_modules.html#google)找到完整的列表。

现在您已经学会了如何通过自动化来补充谷歌云平台，您将学习如何无缝地执行与 Azure 的自动化集成。

# 与 Azure 的无缝自动化集成

Ansible 可以管理的另一个全球云是 Microsoft Azure。

与 AWS 类似，Azure 集成需要在 Playbooks 中执行相当多的步骤。

您需要首先设置认证，以便 Ansible 被允许控制您的 Azure 帐户。

# 安装

要让 Ansible 管理 Azure 云，您需要安装 Python 的 Azure SDK。通过执行以下命令来完成：

```
$ pip install 'ansible[azure]'
```

现在您已经准备好所有依赖项，可以开始认证过程。

# 认证

有不同的方法可以确保 Ansible 能够为您管理 Azure，这取决于您的 Azure 帐户设置方式，但它们都可以在`~/.azure/credentials`文件中配置。

如果您希望 Ansible 使用 Azure 帐户的主要凭据，您需要创建一个类似以下内容的文件：

```
[default] subscription_id = [YOUR_SUBSCIRPTION_ID_HERE] client_id = [YOUR_CLIENT_ID_HERE] secret = [YOUR_SECRET_HERE] tenant = [YOUR_TENANT_HERE]
```

如果您希望使用用户名和密码与 Active Directories，可以这样做：

```
[default] ad_user = [YOUR_AD_USER_HERE] password = [YOUR_AD_PASSWORD_HERE]
```

最后，您可以选择使用 ADFS 进行 Active Directory 登录。在这种情况下，您需要设置一些额外的参数。您最终会得到类似这样的东西：

```
[default] ad_user = [YOUR_AD_USER_HERE] password = [YOUR_AD_PASSWORD_HERE] client_id = [YOUR_CLIENT_ID_HERE] tenant = [YOUR_TENANT_HERE] adfs_authority_url = [YOUR_ADFS_AUTHORITY_URL_HERE]
```

相同的参数可以作为参数传递，也可以作为环境变量传递，如果更合理的话。

# 创建你的第一台机器

现在，Ansible 已经能够连接到你的 Azure 环境，你可以继续进行实际的 Playbook 了。

1.  创建`azure.yaml` Playbook，内容如下：

```
---
- hosts: localhost
  tasks:
    - name: Ensure the Storage Account is present
      azure_rm_storageaccount:
        resource_group: Testing
        name: mysa
        account_type: Standard_LRS
    - name: Ensure the Virtual Network is present
      azure_rm_virtualnetwork:
        resource_group: Testing
        name: myvn
        address_prefixes: "10.10.0.0/16"
    - name: Ensure the Subnet is present
      azure_rm_subnet:
        resource_group: Testing
        name: mysn
        address_prefix: "10.10.0.0/24"
        virtual_network: myvn
    - name: Ensure that the Public IP is set
      azure_rm_publicipaddress:
        resource_group: Testing
        allocation_method: Static
        name: myip
    - name: Ensure a Security Group allowing SSH is present
      azure_rm_securitygroup:
        resource_group: Testing
        name: mysg
        rules:
          - name: SSH
            protocol: Tcp
            destination_port_range: 22
            access: Allow
            priority: 101
            direction: Inbound
    - name: Ensure the NIC is present
      azure_rm_networkinterface:
        resource_group: Testing
        name: testnic001
        virtual_network: myvn
        subnet: mysn
        public_ip_name: myip
        security_group: mysg
    - name: Ensure the Virtual Machine is present
      azure_rm_virtualmachine:
        resource_group: Testing
        name: myvm01
        vm_size: Standard_D1
        storage_account: mysa
        storage_container: myvm01
        storage_blob: myvm01.vhd
        admin_username: admin
        admin_password: Password!
        network_interfaces: testnic001
        image:
          offer: CentOS
          publisher: OpenLogic
          sku: '8.0'
          version: latest
```

1.  我们可以使用以下命令运行它：

```
$ ansible-playbook azure.yaml
```

这将返回类似以下的结果：

```
PLAY [localhost] **********************************************************************************

TASK [Gathering Facts] ****************************************************************************
ok: [localhost]

TASK [Ensure the Storage Account is present] ******************************************************
changed: [localhost] TASK [Ensure the Virtual Network is present] ******************************************************
changed: [localhost]

TASK [Ensure the Subnet is present] ***************************************************************
changed: [localhost]

TASK [Ensure that the Public IP is set] ***********************************************************
changed: [localhost]

TASK [Ensure a Security Group allowing SSH is present] ********************************************
changed: [localhost]

TASK [Ensure the NIC is present] ******************************************************************
changed: [localhost]

TASK [Ensure the Virtual Machine is present] ******************************************************
changed: [localhost]

PLAY RECAP ****************************************************************************************
localhost : ok=8 changed=7 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0 
```

你现在可以在 Azure 云中运行你的机器了！

正如你所看到的，在 Azure 中，你需要在发出创建机器命令之前准备好所有资源。这就是你首先创建存储帐户、虚拟网络、子网、公共 IP、安全组和 NIC，然后只有在那时，才创建机器本身的原因。

除了市场上的三大主要参与者外，还有许多其他云选项。一个非常有趣的选择是 RackSpace，因为它的历史：Rackspace Cloud。

# 通过 Rackspace Cloud 扩展你的环境

Rackspace 是公共云业务中最早的公司之一。此外，Rackspace 在 2010 年与 NASA 联合创办了 OpenStack。在过去的 10 年中，Rackspace 一直是云基础设施、OpenStack 以及更广泛的托管领域的非常有影响力的提供商。

# 安装

为了能够从 Ansible 管理 Rackspace，你需要安装`pyrax`。

安装它的最简单方法是运行以下命令：

```
$ pip install pyrax
```

如果可用，你也可以通过系统包管理器安装它。

# 认证

由于`pyrax`没有凭据文件的默认位置，你需要创建一个文件，然后通过指示`pyrax`在文件位置设置一个环境变量来做到这一点。

让我们从在`~/.rackspace_credentials`中创建一个文件开始，文件内容如下：

```
[rackspace_cloud] username = [YOUR_USERNAME_HERE] api_key = [YOUR_API_KEY_HERE]
```

现在，我们可以通过将`RAX_CREDS_FILE`变量设置为正确的位置来继续进行：

```
**$ export RAX_CREDS_FILE=~/.rackspace_credentials** 
```

让我们继续使用 Rackspace Cloud 创建一台机器。

# 创建你的第一台机器

在 Rackspace Cloud 中创建一台机器非常简单，因为它是一个单步操作：

1.  创建`rax.yaml` Playbook，内容如下：

```
---
- hosts: localhost
  tasks:
    - name: Ensure the my_machine exists
      rax:
        name: my_machine
        flavor: 4
        image: centos-8
        count: 1
        group: my_group
        wait: True
```

1.  现在，你可以使用以下命令来执行它：

```
$ ansible-playbook rax.yaml
```

1.  这应该会产生类似以下的结果：

```
PLAY [localhost] **********************************************************************************

TASK [Gathering Facts] ****************************************************************************
ok: [localhost]

TASK [Ensure the my_machine exists] ***************************************************************
changed: [localhost]

PLAY RECAP ****************************************************************************************
localhost : ok=2 changed=1 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0 
```

正如你所看到的，在 Rackspace Cloud 中创建机器非常简单直接，而默认的 Ansible 模块已经集成了一些有趣的概念，比如组和计数。这些选项允许你以与单个实例相同的方式创建和管理实例组。

# 使用 Ansible 来编排 OpenStack

与我们刚讨论的各种公共云服务相反，OpenStack 允许你创建自己的（私有）云。

私有云的缺点是它们向管理员和用户暴露了更多的复杂性，但这也是它们可以被定制以完美适应组织的原因。

# 安装

能够使用 Ansible 控制 OpenStack 集群的第一步是确保安装了`openstacksdk`。

要安装`openstacksdk`，你需要执行以下命令：

```
$ pip install openstacksdk
```

现在你已经安装了`openstacksdk`，你可以开始认证过程了。

# 认证

由于 Ansible 将使用`openstacksdk`作为其后端，你需要确保`openstacksdk`能够连接到 OpenStack 集群。

为了做到这一点，你可以更改`~/.config/openstack/clouds.yaml`文件，确保为你想要使用它的云配置。

一个正确的 OpenStack 凭据集的示例可能如下所示：

```
clouds:
 test_cloud: region_name: MyRegion auth: auth_url: http://[YOUR_AUTH_URL_HERE]:5000/v2.0/     username: [YOUR_USERNAME_HERE]
 password: [YOUR_PASSWORD_HERE] project_name: myProject
```

如果愿意，也可以设置不同的配置文件位置，将`OS_CLIENT_CONFIG_FILE`变量作为环境变量导出。

现在你已经设置了所需的安全性，以便 Ansible 可以管理你的集群，你可以创建你的第一个 Playbook 了。

# 创建你的第一台机器

由于 OpenStack 非常灵活，它的许多组件可以有许多不同的实现，这意味着它们在行为上可能略有不同。为了能够适应各种情况，管理 OpenStack 的 Ansible 模块往往具有较低的抽象级别，与许多公共云的模块相比。

因此，要创建一台机器，您需要确保公共 SSH 密钥为 OpenStack 所知，并确保 OS 镜像也存在。在这样做之后，您可以设置网络、子网络和路由器，以确保我们要创建的机器可以通过网络进行通信。然后，您可以创建安全组及其规则，以便该机器可以接收连接（在本例中为 ping 和 SSH 流量）。最后，您可以创建一个机器实例。

要完成我们刚刚描述的所有步骤，您需要创建一个名为`openstack.yaml`的文件，其中包含以下内容：

```
---
- hosts: localhost
  tasks:
    - name: Ensure the SSH key is present on OpenStack
      os_keypair:
        state: present
        name: ansible_key
        public_key_file: "{{ '~' | expanduser }}/.ssh/id_rsa.pub"
    - name: Ensure we have a CentOS image
      get_url:
        url: http://cloud.centos.org/centos/8/x86_64/images/CentOS-8-GenericCloud-8.1.1911-20200113.3.x86_64.qcow2
        dest: /tmp/CentOS-8-GenericCloud-8.1.1911-20200113.3.x86_64.qcow2
    - name: Ensure the CentOS image is in OpenStack
      os_image:
        name: centos
        container_format: bare
        disk_format: qcow2
        state: present
        filename: /tmp/CentOS-8-GenericCloud-8.1.1911-20200113.3.x86_64.qcow2
    - name: Ensure the Network is present
      os_network:
        state: present
        name: mynet
        external: False
        shared: False
      register: net_out
    - name: Ensure the Subnetwork is present
      os_subnet:
        state: present
        network_name: "{{ net_out.id }}"
        name: mysubnet
        ip_version: 4
        cidr: 192.168.0.0/24
        gateway_ip: 192.168.0.1
        enable_dhcp: yes
        dns_nameservers:
          - 8.8.8.8
    - name: Ensure the Router is present
      os_router:
        state: present
        name: myrouter
        network: nova
        external_fixed_ips:
          - subnet: nova
        interfaces:
          - mysubnet
    - name: Ensure the Security Group is present
      os_security_group:
        state: present
        name: mysg
    - name: Ensure the Security Group allows ICMP traffic
      os_security_group_rule:
        security_group: mysg
        protocol: icmp
        remote_ip_prefix: 0.0.0.0/0
    - name: Ensure the Security Group allows SSH traffic
      os_security_group_rule:
        security_group: mysg
        protocol: tcp
        port_range_min: 22
        port_range_max: 22
        remote_ip_prefix: 0.0.0.0/0
    - name: Ensure the Instance exists
      os_server:
        state: present
        name: myInstance
        image: centos
        flavor: m1.small
        security_groups: mysg
        key_name: ansible_key
        nics:
          - net-id: "{{ net_out.id }}"
```

现在，您可以运行它，如下：

```
$ ansible-playbook openstack.yaml
```

输出应该如下：

```
PLAY [localhost] **********************************************************************************

TASK [Gathering Facts] ****************************************************************************
ok: [localhost]

TASK [Ensure the SSH key is present on OpenStack] *************************************************
changed: [localhost]

TASK [Ensure we have a CentOS image] **************************************************************
changed: [localhost]

TASK [Ensure the CentOS image is in OpenStack] ****************************************************
changed: [localhost]

TASK [Ensure the Network is present] **************************************************************
changed: [localhost]

TASK [Ensure the Subnetwork is present] ***********************************************************
changed: [localhost]

TASK [Ensure the Router is present] ***************************************************************
changed: [localhost]

TASK [Ensure the Security Group is present] *******************************************************
changed: [localhost]

TASK [Ensure the Security Group allows ICMP traffic] **********************************************
changed: [localhost]

TASK [Ensure the Security Group allows SSH traffic] ***********************************************
changed: [localhost]

TASK [Ensure the Instance exists] *****************************************************************
changed: [localhost]

PLAY RECAP ****************************************************************************************
localhost : ok=11 changed=10 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0 
```

正如您所看到的，这个过程比我们涵盖的公共云要长。但是，您确实可以上传您想要运行的镜像，这是许多云不允许的（或者允许的过程非常复杂）。

# 总结

在本章中，您学习了如何使用 playbooks 自动化任务，从设计和构建容器到在 Kubernetes 上管理部署，以及管理 Kubernetes 对象和使用 Ansible 自动化 Docker。您还探索了可以帮助您自动化云环境的模块，如 AWS、Google Cloud Platform、Azure、Rackspace 和 OpenShift。您还了解了各种云提供商使用的不同方法，包括它们的默认值以及您总是需要添加的参数。

现在您已经了解了 Ansible 如何与云进行交互，您可以立即开始自动化云工作流程。还记得要查看*进一步阅读*部分的文档，以查看 Ansible 支持的所有云模块及其选项。

在下一章中，您将学习如何排除故障和创建测试策略。

# 问题

1.  以下哪个不是 GKE Ansible 模块？

A) `gcp_container_cluster`

B) `gcp_container_node_pool`

C) `gcp_container_node_pool_facts`

D) `gcp_container_node_pool_count`

E) `gcp_container_cluster_facts`

1.  真或假：为了管理 Kubernetes 中的容器，您需要在设置部分中添加`k8s_namespace`。

A) True

B) False

1.  真或假：在使用 Azure 时，在创建实例之前，您不需要创建**网络接口控制器**（**NIC**）。

A) True

B) False

1.  真或假：`Ansible-Container`是与 Kubernetes 和 Doc 交互的唯一方式。

A) True

B) False

1.  真或假：在使用 AWS 时，在创建 EC2 实例之前，需要创建一个安全组。

A) True

B) False

# 进一步阅读

+   更多 AWS 模块：[`docs.ansible.com/ansible/latest/modules/list_of_cloud_modules.html#amazon`](https://docs.ansible.com/ansible/latest/modules/list_of_cloud_modules.html#amazon)

+   更多 Azure 模块：[`docs.ansible.com/ansible/latest/modules/list_of_cloud_modules.html#azure`](https://docs.ansible.com/ansible/latest/modules/list_of_cloud_modules.html#azure)

+   更多 Docker 模块：[`docs.ansible.com/ansible/latest/modules/list_of_cloud_modules.html#docker`](https://docs.ansible.com/ansible/latest/modules/list_of_cloud_modules.html#docker)

+   更多 GCP 模块：[`docs.ansible.com/ansible/latest/modules/list_of_cloud_modules.html#google`](https://docs.ansible.com/ansible/latest/modules/list_of_cloud_modules.html#google)

+   更多 OpenStack 模块：[`docs.ansible.com/ansible/latest/modules/list_of_cloud_modules.html#openstack`](https://docs.ansible.com/ansible/latest/modules/list_of_cloud_modules.html#openstack)

+   更多的 Rackspace 模块：[`docs.ansible.com/ansible/latest/modules/list_of_cloud_modules.html#rackspace`](https://docs.ansible.com/ansible/latest/modules/list_of_cloud_modules.html#rackspace)
