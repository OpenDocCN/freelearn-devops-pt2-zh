# 处理 Kubernetes 软件包管理器

在本章中，我们将深入了解 Helm，即 Kubernetes 软件包管理器。每个成功和重要的平台都必须有一个良好的打包系统。Helm 由 Deis 开发（于 2017 年 4 月被微软收购），后来直接贡献给了 Kubernetes 项目。我们将从理解 Helm 的动机、架构和组件开始。然后，我们将亲身体验并了解如何在 Kubernetes 中使用 Helm 及其图表。这包括查找、安装、自定义、删除和管理图表。最后但同样重要的是，我们将介绍如何创建自己的图表，并处理版本控制、依赖关系和模板化。

将涵盖以下主题：

+   理解 Helm

+   使用 Helm

+   创建自己的图表

# 理解 Helm

Kubernetes 提供了许多在运行时组织和编排容器的方式，但缺乏将一组图像进行更高级别组织的方式。这就是 Helm 的用武之地。在本节中，我们将讨论 Helm 的动机、其架构和组件，并讨论从 Helm Classic 过渡到 Helm 时发生了什么变化。

# Helm 的动机

Helm 支持几个重要的用例：

+   管理复杂性

+   轻松升级

+   简单共享

+   安全回滚

图表可以描述甚至最复杂的应用程序，提供可重复的应用程序安装，并作为单一的权威点。原地升级和自定义钩子允许轻松更新。共享图表很简单，可以在公共或私有服务器上进行版本控制和托管。当您需要回滚最近的升级时，Helm 提供了一个命令来回滚基础设施的一致变化集。

# Helm 架构

Helm 旨在执行以下操作：

+   从头开始创建新图表

+   将图表打包成图表存档（`tgz`）文件

+   与存储图表的图表存储库进行交互

+   将图表安装和卸载到现有的 Kubernetes 集群中

+   管理使用 Helm 安装的图表的发布周期

Helm 使用客户端-服务器架构来实现这些目标。

# Helm 组件

Helm 有一个在 Kubernetes 集群上运行的服务器组件和一个在本地机器上运行的客户端组件。

# Tiller 服务器

服务器负责管理发布。它与 Helm 客户端以及 Kubernetes API 服务器进行交互。其主要功能如下：

+   监听来自 Helm 客户端的传入请求

+   结合图表和配置以构建发布

+   将图表安装到 Kubernetes 中

+   跟踪后续发布

+   通过与 Kubernetes 交互来升级和卸载图表

# Helm 客户端

您在您的机器上安装 Helm 客户端。它负责以下工作：

+   本地图表开发

+   管理存储库

+   与 Tiller 服务器交互

+   发送图表以安装

+   询问有关发布的信息

+   请求升级或卸载现有版本

# 使用 Helm

Helm 是一个丰富的软件包管理系统，可以让您执行管理集群上安装的应用程序所需的所有必要步骤。让我们卷起袖子，开始吧。

# 安装 Helm

安装 Helm 涉及安装客户端和服务器。Helm 是用 Go 实现的，同一个二进制可用作客户端或服务器。

# 安装 Helm 客户端

您必须正确配置 Kubectl 以与您的 Kubernetes 集群通信，因为 Helm 客户端使用 Kubectl 配置与 Helm 服务器（Tiller）通信。

Helm 为所有平台提供二进制发布，网址为[`github.com/kubernetes/helm/releases/latest`](https://github.com/kubernetes/helm/releases/latest)。

对于 Windows，您也可以使用`chocolatey`软件包管理器，但它可能比官方版本慢一点，`https://chocolatey.org/packages/kubernetes-helm/<version>`。

对于 macOS 和 Linux，您可以从脚本安装客户端：

```
$ curl https://raw.githubusercontent.com/kubernetes/helm/master/scripts/get > get_helm.sh
$ chmod 700 get_helm.sh
$ ./get_helm.sh  
```

在 macOS X 上，您还可以使用 Homebrew：

```
brew install kubernetes-helm  
```

# 安装 Tiller 服务器

Tiller 通常在集群内运行。对于开发来说，有时在本地运行 Tiller 会更容易一些。

# 在集群中安装 Tiller

安装 Tiller 的最简单方法是从安装了 Helm 客户端的机器上进行。运行以下命令：

```
helm init
```

这将在远程 Kubernetes 集群上初始化客户端和 Tiller 服务器。安装完成后，您将在集群的`kube-system`命名空间中有一个正在运行的 Tiller pod：

```
> kubectl get po --namespace=kube-system -l name=tiller
NAME                            READY  STATUS   RESTARTS   AGE
tiller-deploy-3210613906-2j5sh  1/1    Running  0          1m  
```

您还可以运行`helm version`来查看客户端和服务器的版本：

```
> helm version
Client: &version.Version{SemVer:"v2.2.3", GitCommit:"1402a4d6ec9fb349e17b912e32fe259ca21181e3", GitTreeState:"clean"}
Server: &version.Version{SemVer:"v2.2.3", GitCommit:"1402a4d6ec9fb349e17b912e32fe259ca21181e3", GitTreeState:"clean"} 
```

# 在本地安装 Tiller

如果要在本地运行 Tiller，首先需要构建它。这在 Linux 和 macOS 上都受支持：

```
> cd $GOPATH
> mkdir -p src/k8s.io
> cd src/k8s.io
> git clone https://github.com/kubernetes/helm.git
> cd helm
> make bootstrap build 
```

引导目标将尝试安装依赖项，重建`vendor/`树，并验证配置。

构建目标将编译 Helm 并将其放置在`bin/helm`中。Tiller 也被编译并放置在`bin/tiller`中。

现在您可以运行 `bin/tiller`。Tiller 将通过您的 Kubectl 配置连接到 Kubernetes 集群。

需要告诉 Helm 客户端连接到本地的 Tiller 服务器。您可以通过设置环境变量来实现：

```
> export HELM_HOST=localhost:44134 
```

否则，您可以将其作为命令行参数传递：`--host localhost:44134`。

# 使用替代存储后端

Helm 2.7.0 添加了将发布信息存储为 **secrets** 的选项。早期版本总是将发布信息存储在 ConfigMaps 中。secrets 后端增加了图表的安全性。它是一种通用 Kubernetes 加密的补充。要使用 Secrets 后端，您需要使用以下命令行运行 Helm：

```
> helm init --override 'spec.template.spec.containers[0].command'='{/tiller,--storage=secret}' 
```

# 查找图表

为了使用 Helm 安装有用的应用程序和软件，您需要先找到它们的图表。这就是 `helm search` 命令发挥作用的地方。默认情况下，Helm 搜索官方的 Kubernetes `chart 仓库`，名为 `stable`：

```
>  helm search
NAME                            VERSION     DESCRIPTION
stable/acs-engine-autoscaler     2.1.1      Scales worker nodes within agent pools
stable/aerospike                0.1.5       A Helm chart for Aerospike in Kubernetes
stable/artifactory             6.2.4        Universal Repository Manager supporting all maj...
stable/aws-cluster-autoscaler  0.3.2        Scales worker nodes within autoscaling groups.
stable/buildkite               0.2.0        Agent for Buildkite
stable/centrifugo              2.0.0        Centrifugo is a real-time messaging server.
stable/chaoskube               0.6.1        Chaoskube periodically kills random pods in you...
stable/chronograf              0.4.0        Open-source web application written in Go and R..
stable/cluster-autoscaler      0.3.1        Scales worker nodes within autoscaling groups. 
```

官方仓库拥有丰富的图表库，代表了所有现代开源数据库、监控系统、特定于 Kubernetes 的辅助工具，以及一系列其他提供，比如 Minecraft 服务器。您可以搜索特定的图表，例如，让我们搜索包含 `kube` 在其名称或描述中的图表：

```
> helm search kube
NAME                            VERSION    DESCRIPTION
stable/chaoskube                0.6.1     Chaoskube periodically kills random pods in you...
stable/kube-lego               0.3.0      Automatically requests certificates from Let's ...
stable/kube-ops-view           0.4.1      Kubernetes Operational View - read-only system ...
stable/kube-state-metrics      0.5.1      Install kube-state-metrics to generate and expo...
stable/kube2iam                0.6.1      Provide IAM credentials to pods based on annota...
stable/kubed                   0.1.0      Kubed by AppsCode - Kubernetes daemon
stable/kubernetes-dashboard    0.4.3      General-purpose web UI for Kubernetes clusters
stable/sumokube                0.1.1      Sumologic Log Collector
stable/aerospike               0.1.5      A Helm chart for Aerospike in Kubernetes
stable/coredns                 0.8.0      CoreDNS is a DNS server that chains plugins and...
stable/etcd-operator           0.6.2      CoreOS etcd-operator Helm chart for Kubernetes
stable/external-dns            0.4.4      Configure external DNS servers (AWS Route53...
stable/keel                    0.2.0      Open source, tool for automating Kubernetes dep...
stable/msoms                   0.1.1      A chart for deploying omsagent as a daemonset... 
stable/nginx-lego              0.3.0      Chart for nginx-ingress-controller and kube-lego
stable/openvpn                 2.0.2      A Helm chart to install an openvpn server insid...
stable/risk-advisor            2.0.0      Risk Advisor add-on module for Kubernetes
stable/searchlight             0.1.0      Searchlight by AppsCode - Alerts for Kubernetes
stable/spartakus               1.1.3      Collect information about Kubernetes clusters t...
stable/stash                   0.2.0      Stash by AppsCode - Backup your Kubernetes Volumes
stable/traefik                 1.15.2     A Traefik based Kubernetes ingress controller w...
stable/voyager                 2.0.0       Voyager by AppsCode - Secure Ingress Controller...
stable/weave-cloud             0.1.2      Weave Cloud is a add-on to Kubernetes which pro...
stable/zetcd                   0.1.4      CoreOS zetcd Helm chart for Kubernetes
stable/buildkite               0.2.0      Agent for Buildkite 
```

让我们尝试另一个搜索：

```
> helm search mysql
NAME                      VERSION    DESCRIPTION
stable/mysql              0.3.4      Fast, reliable, scalable, and easy to use open-...
stable/percona            0.3.0      free, fully compatible, enhanced, open source d...
stable/gcloud-sqlproxy    0.2.2      Google Cloud SQL Proxy
stable/mariadb            2.1.3       Fast, reliable, scalable, and easy to use open-... 
```

发生了什么？为什么 `mariadb` 出现在结果中？原因是 `mariadb`（它是 MySQL 的一个分支）在其描述中提到了 MySQL，即使在截断的输出中看不到。要获取完整的描述，请使用 `helm inspect` 命令：

```
> helm inspect stable/mariadb
appVersion: 10.1.30
description: Fast, reliable, scalable, and easy to use open-source relational database
 system. MariaDB Server is intended for mission-critical, heavy-load production systems
 as well as for embedding into mass-deployed software.
engine: gotpl
home: https://mariadb.org
icon: https://bitnami.com/assets/stacks/mariadb/img/mariadb-stack-220x234.png
keywords:
- mariadb
- mysql
- database
- sql
- prometheus
maintainers:
- email: containers@bitnami.com
 name: bitnami-bot
name: mariadb
sources:
- https://github.com/bitnami/bitnami-docker-mariadb
- https://github.com/prometheus/mysqld_exporter
version: 2.1.3
```

# 安装软件包

好了。您找到了梦想的软件包。现在，您可能想要在 Kubernetes 集群上安装它。当您安装一个软件包时，Helm 会创建一个发布，您可以使用它来跟踪安装进度。让我们使用 `helm install` 命令安装 `MariaDB`。让我们详细查看输出。输出的第一部分列出了发布的名称 - 在这种情况下是 `cranky-whippet`（您可以使用 `--name` 标志选择自己的名称）、命名空间和部署状态：

```
> helm install stable/mariadb
NAME:   cranky-whippet
LAST DEPLOYED: Sat Mar 17 10:21:21 2018
NAMESPACE: default
STATUS: DEPLOYED  
```

输出的第二部分列出了此图表创建的所有资源。请注意，资源名称都是根据发布名称派生的：

```
RESOURCES:
==> v1/Service
NAME                    TYPE       CLUSTER-IP      EXTERNAL-IP  PORT(S)   AGE
cranky-whippet-mariadb  ClusterIP  10.106.206.108  <none>       3306/TCP  1s

==> v1beta1/Deployment
NAME                    DESIRED  CURRENT  UP-TO-DATE  AVAILABLE  AGE
cranky-whippet-mariadb  1        1        1           0          1s

==> v1/Pod(related)
NAME                                     READY  STATUS    RESTARTS  AGE
cranky-whippet-mariadb-6c85fb4796-mttf7  0/1    Init:0/1  0         1s

==> v1/Secret
NAME                    TYPE    DATA  AGE
cranky-whippet-mariadb  Opaque  2     1s

==> v1/ConfigMap
NAME                          DATA  AGE
cranky-whippet-mariadb        1     1s
cranky-whippet-mariadb-tests  1     1s

==> v1/PersistentVolumeClaim
NAME                    STATUS  VOLUME                                    CAPACITY  ACCESS MODES  STORAGECLASS  AGE
cranky-whippet-mariadb  Bound   pvc-9cb7e176-2a07-11e8-9bd6-080027c94384  8Gi       RWO           standard      1s  
```

最后一部分是提供如何在 Kubernetes 集群中使用 `MariaDB` 的易于理解的说明：

```
NOTES:
MariaDB can be accessed via port 3306 on the following DNS name from within your cluster:
cranky-whippet-mariadb.default.svc.cluster.local

To get the root password run:

MARIADB_ROOT_PASSWORD=$(kubectl get secret --namespace default cranky-whippet-mariadb -o jsonpath="{.data.mariadb-root-password}" | base64 --decode)

To connect to your database:

1\. Run a pod that you can use as a client:

kubectl run cranky-whippet-mariadb-client --rm --tty -i --env MARIADB_ROOT_PASSWORD=$MARIADB_ROOT_PASSWORD --image bitnami/mariadb --command -- bash

2\. Connect using the mysql cli, then provide your password:
mysql -h cranky-whippet-mariadb -p$MARIADB_ROOT_PASSWORD

```

# 检查安装状态

Helm 不会等待安装完成，因为这可能需要一些时间。`helm status`命令以与初始`helm install`命令的输出相同的格式显示发布的最新信息。在`install`命令的输出中，您可以看到`PersistentVolumeClaim`的`PENDING`状态。现在让我们来检查一下：

```
> helm status cranky-whippet | grep Persist -A 3
==> v1/PersistentVolumeClaim
NAME STATUS VOLUME CAPACITY ACCESS MODES STORAGECLASS  AGE
cranky-whippet-mariadbBoundpvc-9cb7e176-2a07-11e8-9bd6-080027c943848Gi             RWO                         standard                   5m  
```

万岁！它现在已绑定，并且附加了一个容量为 8GB 的卷。

让我们尝试连接并验证`mariadb`是否确实可访问。让我们稍微修改一下注释中建议的命令以进行连接。我们可以直接在容器上运行`mysql`命令，而不是运行`bash`然后再运行`mysql`：

```
> kubectl run cranky-whippet-mariadb-client --rm --tty -i --image bitnami/mariadb --command -- mysql -h cranky-whippet-mariadb  
```

如果您看不到命令提示符，请尝试按*Enter*键。

```
MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
+--------------------+
3 rows in set (0.00 sec) 
```

# 自定义图表

作为用户，您经常希望自定义或配置您安装的图表。Helm 完全支持通过`config`文件进行自定义。要了解可能的自定义，您可以再次使用`helm inspect`命令，但这次要专注于值。以下是部分输出：

```
> helm inspect values stable/mariadb
## Bitnami MariaDB image version
## ref: https://hub.docker.com/r/bitnami/mariadb/tags/
##
## Default: none
image: bitnami/mariadb:10.1.30-r1

## Specify an imagePullPolicy (Required)
## It's recommended to change this to 'Always' if the image tag is 'latest'
## ref: http://kubernetes.io/docs/user-guide/images/#updating-images
imagePullPolicy: IfNotPresent

## Use password authentication
usePassword: true

## Specify password for root user
## Defaults to a random 10-character alphanumeric string if not set and usePassword is true
## ref: https://github.com/bitnami/bitnami-docker-mariadb/blob/master/README.md#setting-the-root-password-on-first-run
##
# mariadbRootPassword:

## Create a database user
## Password defaults to a random 10-character alphanumeric string if not set and usePassword is true
## ref: https://github.com/bitnami/bitnami-docker-mariadb/blob/master/README.md#creating-a-database-user-on-first-run
##
# mariadbUser:
# mariadbPassword:

## Create a database
## ref: https://github.com/bitnami/bitnami-docker-mariadb/blob/master/README.md#creating-a-database-on-first-run
##
# mariadbDatabase:  
```

例如，如果您想在安装`mariadb`时设置根密码并创建数据库，您可以创建以下 YAML 文件并将其保存为`mariadb-config.yaml`：

```
mariadbRootPassword: supersecret
mariadbDatabase: awesome_stuff 
```

然后，运行`helm`并传递`yaml`文件：

```
> helm install -f config.yaml stable/mariadb  
```

您还可以使用`--set`在命令行上设置单个值。如果`--f`和`--set`都尝试设置相同的值，则`--set`优先。例如，在这种情况下，根密码将是`evenbettersecret`：

```
helm install -f config.yaml --set mariadbRootPassword=evenbettersecret stable/mariadb 
```

您可以使用逗号分隔的列表指定多个值：`--set a=1,b=2`。

# 其他安装选项

`helm install`命令可以从多个来源安装：

+   一个`chart repository`（正如我们所见）

+   本地图表存档（`helm install foo-0.1.1.tgz`）

+   一个解压的`chart`目录（`helm install path/to/foo`）

+   完整的 URL（`helm install https://example.com/charts/foo-1.2.3.tgz`）

# 升级和回滚发布

您可能希望将安装的软件包升级到最新版本。Helm 提供了`upgrade`命令，它可以智能地操作，并且只更新已更改的内容。例如，让我们检查我们`mariadb`安装的当前值：

```
> helm get values cranky-whippet
mariadbDatabase: awesome_stuff
mariadbRootPassword: evenbettersecret 
```

现在，让我们运行、升级并更改数据库的名称：

```
> helm upgrade cranky-whippet --set mariadbDatabase=awesome_sauce stable/mariadb
$ helm get values cranky-whippet
mariadbDatabase: awesome_sauce 
```

请注意，我们已经丢失了`root`密码。当您升级时，所有现有值都将被替换。好的，让我们回滚。`helm history`命令显示了我们可以回滚到的所有可用修订版本：

```
> helm history cranky-whippet
REVISION         STATUS           CHART            DESCRIPTION
1               SUPERSEDED      mariadb-2.1.3      Install complete
2               SUPERSEDED      mariadb-2.1.3      Upgrade complete
3               SUPERSEDED      mariadb-2.1.3      Upgrade complete
4               DEPLOYED        mariadb-2.1.3      Upgrade complete  
```

让我们回滚到修订版本`3`：

```
> helm rollback cranky-whippet 3
Rollback was a success! Happy Helming!

> helm history cranky-whippet
REVISION        STATUS             CHART            DESCRIPTION
1                SUPERSEDED      mariadb-2.1.3     Install complete
2                SUPERSEDED      mariadb-2.1.3     Upgrade complete
3                SUPERSEDED      mariadb-2.1.3     Upgrade complete
4                SUPERSEDED      mariadb-2.1.3     Upgrade complete 
5                DEPLOYED        mariadb-2.1.3     Rollback to 3   
```

让我们验证一下我们的更改是否已回滚：

```
> helm get values cranky-whippet
mariadbDatabase: awesome_stuff
mariadbRootPassword: evenbettersecret  
```

# 删除发布

当然，您也可以使用`helm delete`命令删除一个发布。

首先，让我们检查发布的列表。我们只有`cranky-whippet`：

```
> helm list
NAME            REVISION     STATUS      CHART           NAMESPACE
cranky-whippet    5         DEPLOYED   mariadb-2.1.3      default  
```

现在，让我们删除它：

```
> helm delete cranky-whippet 
release "cranky-whippet" deleted 
```

所以，没有更多的发布了：

```
> helm list 
```

但是，Helm 也会跟踪已删除的发布。您可以使用`--all`标志查看它们：

```
> helm list --all
NAME                 REVISION  STATUS    CHART          NAMESPACE
cranky-whippet        5        DELETED   mariadb-2.1.3  default 
```

要完全删除一个发布，添加`--purge`标志：

```
> helm delete --purge cranky-whippet   
```

# 使用存储库

Helm 将图表存储在简单的 HTTP 服务器存储库中。任何标准的 HTTP 服务器都可以托管 Helm 存储库。在云中，Helm 团队验证了 AWS S3 和 Google Cloud 存储都可以在 Web 启用模式下作为 Helm 存储库。Helm 还附带了一个用于开发人员测试的本地包服务器。它在客户端机器上运行，因此不适合共享。在一个小团队中，您可以在本地网络上的共享机器上运行 Helm 包服务器，所有团队成员都可以访问。

要使用本地包服务器，请键入`helm serve`。请在单独的终端窗口中执行此操作，因为它会阻塞。Helm 将默认从`~/.helm/repository/local`开始提供图表服务。您可以将您的图表放在那里，并使用`helm index`生成索引文件。

生成的`index.yaml`文件列出了所有的图表。

请注意，Helm 不提供将图表上传到远程存储库的工具，因为这将需要远程服务器了解 Helm，知道在哪里放置图表，以及如何更新`index.yaml`文件。

在客户端方面，`helm repo`命令允许您`list`，`add`，`remove`，`index`和`update`：

```
> helm repo  
```

该命令由多个子命令组成，用于与`chart`存储库交互。

它可以用来`add`，`remove`，`list`和`index`图表存储库：

+   **示例用法**：

```
$ helm repo add [NAME] [REPO_URL]
```

+   **用法**：

```
helm repo [command]
```

+   **可用命令**：

```
  add         add a chart repository
  index       generate an index file for a given a directory 
  list        list chart repositories
  remove      remove a chart repository
  update     update information on available charts 
```

# 使用 Helm 管理图表

Helm 提供了几个命令来管理图表。它可以为您创建一个新的图表：

```
> helm create cool-chart
Creating cool-chart  
```

Helm 将在`cool-chart`下创建以下文件和目录：

```
-rw-r--r--  1 gigi.sayfan  gigi.sayfan   333B Mar 17 13:36 .helmignore
-rw-r--r--  1 gigi.sayfan  gigi.sayfan   88B Mar 17 13:36 Chart.yaml
drwxr-xr-x  2 gigi.sayfan  gigi.sayfan   68B Mar 17 13:36 charts
drwxr-xr-x  7 gigi.sayfan  gigi.sayfan   238B Mar 17 13:36 templates
-rw-r--r--  1 gigi.sayfan  gigi.sayfan   1.1K Mar 17 13:36 values.yaml 
```

编辑图表后，您可以将其打包成一个 tar`gzipped`存档：

```
> helm package cool-chart  
```

Helm 将创建一个名为`cool-chart-0.1.0.tgz`的存档，并将两者存储在`local`目录和`local repository`中。

您还可以使用 helm 来帮助您找到图表格式或信息的问题：

```
> helm lint cool-chart
==> Linting cool-chart
[INFO] Chart.yaml: icon is recommended

1 chart(s) linted, no failures  
```

# 利用入门包

`helm create`命令带有一个可选的`--starter`标志，让您指定一个入门图表。

启动器是位于`$HELM_HOME/starters`中的常规图表。作为图表开发者，您可以编写专门用作启动器的图表。这样的图表应该考虑以下几点：

+   `Chart.yaml`将被生成器覆盖

+   用户希望修改这样一个图表的内容，因此文档应该说明用户如何做到这一点

目前，没有办法将图表安装到`$HELM_HOME/starters`，用户必须手动复制。如果您开发启动包图表，请确保在您的图表文档中提到这一点。

# 创建您自己的图表

图表是描述一组相关的 Kubernetes 资源的文件集合。一个单独的图表可以用来部署一些简单的东西，比如一个`memcached` pod，或者一些复杂的东西，比如一个完整的 Web 应用堆栈，包括 HTTP 服务器、数据库和缓存。

图表是以特定目录树布局的文件创建的。然后，它们可以被打包成版本化的存档进行部署。关键文件是`Chart.yaml`。

# Chart.yaml 文件

`Chart.yaml`文件是 Helm 图表的主文件。它需要一个名称和版本字段：

+   `name`: 图表的名称（与目录名称相同）

+   `version`: SemVer 2 版本

它还可以包含各种可选字段：

+   `kubeVersion`: 兼容的 Kubernetes 版本的 SemVer 范围

+   `description`: 该项目的单句描述

+   `keywords`: 关于这个项目的关键字列表

+   `home`: 该项目主页的 URL

+   `sources`: 该项目源代码的 URL 列表

+   `maintainers`:

+   `name`: 维护者的名称（每个维护者都需要）

+   `email`: 维护者的电子邮件（可选）

+   `url`: 维护者的 URL（可选）

+   `engine`: 模板引擎的名称（默认为`gotpl`）

+   `icon`: 用作图标的 SVG 或 PNG 图像的 URL

+   `appVersion`: 包含的应用程序版本

+   `deprecated`: 这个图表是否已被弃用？（布尔值）

+   `tillerVersion`: 该图表所需的 Tiller 版本

# 图表版本控制

`Chart.yaml`中的版本字段由 CLI 和 Tiller 服务器使用。`helm package`命令将使用在`Chart.yaml`中找到的版本来构建包名。图表包名中的版本号必须与`Chart.yaml`中的版本号匹配。

# appVersion 字段

`appVersion`字段与版本字段无关。Helm 不使用它，它作为用户的元数据或文档，用于了解他们正在部署的内容。Helm 不强制正确性。

# 弃用图表

有时，您可能希望弃用一个图表。您可以通过将`Chart.yaml`中的弃用字段设置为`true`来标记图表为弃用状态。弃用最新版本的图表就足够了。稍后您可以重用图表名称并发布一个未弃用的新版本。`kubernetes/charts`项目使用的工作流程是：

+   更新图表的`Chart.yaml`以标记图表为弃用状态并提升版本

+   发布图表的新版本

+   从`源代码库`中删除图表

# 图表元数据文件

图表可能包含各种元数据文件，例如`README.md`、`LICENSE`和`NOTES.txt`，用于描述图表的安装、配置、使用和许可。`README.md`文件应格式化为 markdown。它应提供以下信息：

+   图表提供的应用程序或服务的描述

+   运行图表的任何先决条件或要求

+   `values.yaml`中选项的描述和默认值

+   安装或配置图表的任何其他信息

`templates/NOTES.txt`文件将在安装后或查看发布状态时显示。您应该保持`NOTES`简洁，并指向`README.md`以获取详细说明。通常会放置使用说明和下一步操作，例如有关连接到数据库或访问 Web UI 的信息。

# 管理图表依赖关系

在 Helm 中，一个图表可以依赖任意数量的其他图表。通过在`requirements.yaml`文件中列出它们或在安装期间将依赖图表复制到 charts/子目录中来明确表示这些依赖关系。

依赖可以是图表存档（`foo-1.2.3.tgz`）或未解压的图表目录。但是，其名称不能以`_`或`.`开头。图表加载程序会忽略这样的文件。

# 使用`requirements.yaml`管理依赖关系

不要手动将图表放在`charts/`子目录中，最好使用`requirements.yaml`文件在图表内声明依赖关系。

`requirements.yaml`文件是一个简单的文件，用于列出图表的依赖关系：

```
dependencies:
 - name: foo
   version: 1.2.3
   repository: http://example.com/charts
 - name: bar
   version: 4.5.6
   repository: http://another.example.com/charts
```

`name`字段是您想要的图表名称。

`version`字段是您想要的图表版本。

`repository`字段是指向`图表存储库`的完整 URL。请注意，您还必须使用`helm repo`将该`存储库`添加到本地。

一旦您有了一个依赖文件，您可以运行`helm dep up`，它将使用您的依赖文件将所有指定的图表下载到 charts 子目录中：

```
$ helm dep up foo-chart
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "local" chart repository
...Successfully got an update from the "stable" chart repository
...Successfully got an update from the "example" chart repository
...Successfully got an update from the "another" chart repository
Update Complete. Happy Helming!
Saving 2 charts
Downloading Foo from repo http://example.com/charts
Downloading Bar from repo http://another.example.com/charts
```

Helm 存储依赖图表在`charts/`目录中作为图表存档进行检索。对于前面的示例，这些文件将存在于`charts`目录中：

```
charts/
  foo-1.2.3.tgz
  bar-4.5.6.tgz 
```

使用`requirements.yaml`管理图表及其依赖项是最佳实践，既可以明确记录依赖关系，也可以在团队之间共享，并支持自动化流程。

# 在 requirements.yaml 中使用特殊字段

`requirements.yaml`文件中的每个条目还可以包含可选的`fields`标签和条件。

这些字段可用于动态控制图表的加载（默认情况下，所有图表都会加载）。当存在标签或条件时，Helm 将评估它们并确定是否应加载目标图表：

+   `condition`：`condition`字段包含一个或多个 YAML 路径（用逗号分隔）。如果此路径存在于顶级父级的值中并解析为布尔值，则图表将根据该布尔值启用或禁用。仅评估列表中找到的第一个有效路径，如果没有路径存在，则条件不起作用。

+   `tags`：`tags`字段是一个 YAML 标签列表，用于与该图表关联。在顶级父级的值中，可以通过指定标签和布尔值来启用或禁用具有标签的所有图表。

+   以下是一个很好地利用条件和标签来启用和禁用依赖项安装的`requirements.yaml`和`values.yaml`示例。`requirements.yaml`文件根据`global enabled`字段的值和特定的`sub-charts enabled`字段定义了安装其依赖项的两个条件：

```
    # parentchart/requirements.yaml
    dependencies:
          - name: subchart1
            repository: http://localhost:10191
            version: 0.1.0
            condition: subchart1.enabled, global.subchart1.enabled
            tags:
              - front-end
              - subchart1
          - name: subchart2
            repository: http://localhost:10191
            version: 0.1.0
            condition: subchart2.enabled,global.subchart2.enabled
            tags:
              - back-end
              - subchart2 
```

`values.yaml`文件为一些条件变量分配了值。`subchart2`标签没有获得值，因此被认为是启用的：

```
# parentchart/values.yaml
 subchart1:
   enabled: true
  tags:
   front-end: false
   back-end: true
```

在安装图表时，也可以从命令行设置标签和条件值，并且它们将优先于`values.yaml`文件：

```
helm install --set subchart2.enabled=false
```

标签和条件的解析如下：

+   条件（在值中设置）始终会覆盖标签。存在的第一个条件路径获胜，该图表的后续条件将被忽略。

+   如果图表的任何标签为 true，则启用该图表。

+   标签和条件值必须在顶级父值中设置。

+   值中的标签：键必须是顶级键。不支持全局和嵌套标签。

# 使用模板和值

任何重要的应用程序都需要配置和适应特定的用例。Helm 图表是使用 Go 模板语言填充占位符的模板。Helm 支持来自`Sprig`库和其他一些专门函数的附加功能。模板文件存储在图表的`templates/`子目录中。Helm 将使用模板引擎渲染此目录中的所有文件，并应用提供的值文件。

# 编写模板文件

模板文件只是遵循 Go 模板语言规则的文本文件。它们可以生成 Kubernetes 配置文件。以下是 artifactory 图表中的服务模板文件：

```
kind: Service
apiVersion: v1
kind: Service
metadata:
  name: {{ template "artifactory.fullname" . }}
  labels:
    app: {{ template "artifactory.name" . }}
    chart: {{ .Chart.Name }}-{{ .Chart.Version }}
    component: "{{ .Values.artifactory.name }}"
    heritage: {{ .Release.Service }}
    release: {{ .Release.Name }}
{{- if .Values.artifactory.service.annotations }}
  annotations:
{{ toYaml .Values.artifactory.service.annotations | indent 4 }}
{{- end }}
spec:
  type: {{ .Values.artifactory.service.type }}
  ports:
  - port: {{ .Values.artifactory.externalPort }}
    targetPort: {{ .Values.artifactory.internalPort }}
    protocol: TCP
    name: {{ .Release.Name }}
  selector:
    app: {{ template "artifactory.name" . }}
    component: "{{ .Values.artifactory.name }}"
    release: {{ .Release.Name }}
```

# 使用管道和函数

Helm 允许在模板文件中使用内置的 Go 模板函数、sprig 函数和管道的丰富和复杂的语法。以下是一个利用这些功能的示例模板。它使用 repeat、quote 和 upper 函数来处理 food 和 drink 键，并使用管道将多个函数链接在一起：

```
apiVersion: v1 
kind: ConfigMap 
metadata: 
  name: {{ .Release.Name }}-configmap 
data: 
  greeting: "Hello World" 
  drink: {{ .Values.favorite.drink | repeat 3 | quote }} 
  food: {{ .Values.favorite.food | upper | quote }}  
```

查看值文件是否具有以下部分：

```
favorite: 
  drink: coffee 
  food: pizza 
```

如果是，则生成的图表将如下所示：

```
apiVersion: v1 
kind: ConfigMap 
metadata: 
  name: cool-app-configmap 
data: 
  greeting: "Hello World" 
  drink: "coffeecoffeecoffee" 
  food: "PIZZA" 
```

# 嵌入预定义值

Helm 提供了一些预定义的值，您可以在模板中使用。在先前的 artifactory 图表模板中，`Release.Name`，`Release.Service`，`Chart.Name`和`Chart.Version`是 Helm 预定义值的示例。其他预定义值如下：

+   `Release.Time`

+   `Release.Namespace`

+   `Release.IsUpgrade`

+   `Release.IsInstall`

+   `Release.Revision`

+   `Chart`

+   `Files`

+   `Capabilities`

图表是`Chart.yaml`的内容。文件和功能预定义值是`类似于映射`的对象，允许通过各种函数进行访问。请注意，模板引擎会忽略`Chart.yaml`中的未知字段，并且无法用于`传递`任意结构化数据到模板中。

# 从文件中提供值

这是`artifactory`默认值文件的一部分。该文件中的值用于填充多个模板。例如，先前的服务模板中使用了`artifactory name`和`internalPort`的值：

```
artifactory:
  name: artifactory
   replicaCount: 1
   image:
   # repository: "docker.bintray.io/jfrog/artifactory-oss"
   repository: "docker.bintray.io/jfrog/artifactory-pro"
   version: 5.9.1
   pullPolicy: IfNotPresent
    service:
     name: artifactory
      type: ClusterIP
      annotations: {}
      externalPort: 8081
      internalPort: 8081
      persistence:
        mountPath: "/var/opt/jfrog/artifactory"
        enabled: true
        accessMode: ReadWriteOnce
        size: 20Gi 
```

您可以提供自己的 YAML 值文件来在安装命令期间覆盖默认值：

```
> helm install --values=custom-values.yaml gitlab-ce 
```

# 范围、依赖和值

值文件可以声明顶层图表的值，以及该图表的`charts/`目录中包含的任何图表的值。例如，`artifactory-ce values.yaml`文件包含其依赖图表`postgresql`的一些默认值：

```
## Configuration values for the postgresql dependency
## ref: https://github.com/kubernetes/charts/blob/master/stable/postgressql/README.md
##
postgresql:
postgresUser: "artifactory"
postgresPassword: "artifactory"
postgresDatabase: "artifactory"
persistence:
 enabled: true
```

顶层图表可以访问其依赖图表的值，但反之则不行。还有一个全局值可供所有图表访问。例如，您可以添加类似以下内容：

```
global:
 app: cool-app  
```

当全局存在时，它将被复制到每个依赖图表的值中，如下所示：

```
global:
  app: cool-app

 postgresql:
   global:
     app: cool-app
     ...  
```

# 总结

在本章中，我们看了一下 Helm，Kubernetes 的包管理器。Helm 使 Kubernetes 能够管理由许多 Kubernetes 资源组成的复杂软件，这些资源之间存在相互依赖。它的作用与操作系统的包管理器相同。它组织软件包，让您搜索图表，安装和升级图表，并与合作者共享图表。您可以开发自己的图表并将它们存储在存储库中。

在这一点上，您应该了解 Helm 在 Kubernetes 生态系统和社区中的重要作用。您应该能够有效地使用它，甚至开发和分享您自己的图表。

在下一章中，我们将展望 Kubernetes 的未来，审查其路线图以及我愿望清单中的一些个人项目。
