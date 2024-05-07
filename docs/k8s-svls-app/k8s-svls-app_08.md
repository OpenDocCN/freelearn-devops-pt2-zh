# 第八章：使用 Fission 启动应用程序

接下来我们将看一下 Fission。Fission 是一个快速增长的，基于 Kubernetes 的无服务器框架，而且在我们之前章节中看到的技术中，可能是最多才多艺的。在本章中，我们将涵盖：

+   谁构建了 Fission？

+   安装先决条件

+   在本地安装、配置和运行 Fission

+   命令概述

+   在云中安装、配置和运行 Fission

+   部署一些示例 Fission 应用程序

到本章结束时，我们将在两个不同的目标环境中安装 Fission，并且还将启动多个应用程序。

# Fission 概述

Fission 是由 Platform9 开发的开源无服务器应用程序。它旨在在 Kubernetes 之上运行，并利用一些核心的 Kubernetes 功能。Platform9 是一家托管服务提供商，其核心业务是部署、管理和支持专门从事 OpenStack 和 Kubernetes 的开源云。

OpenStack 是一组开源组件，构成了一个完全功能的基础设施即服务产品。它提供计算、网络、块存储、对象存储、编排，甚至容器服务等功能。

该项目的目标是为多个不同的硬件供应商提供支持，从普通的 x86 硬件到专门的存储解决方案，使最终用户能够构建自己的 AWS 和 Microsoft Azure 风格的产品。

随着 AWS Lambda 和 Azure Functions 等服务成熟到现在几乎在大多数企业中都很普遍，Platform9 看到了提供自己的函数即服务的机会。

作为一家专门从事复杂开源解决方案的公司，他们为他们向社区贡献自己的工作是有意义的，因此他们以 Apache 许可证发布了 Fission。

Apache 软件基金会的 Apache 2.0 许可证允许开发人员免费发布他们的软件，允许最终用户以任何目的使用该软件，并在不必担心版税的情况下修改/重新分发它。为了确保许可证不被违反，最终用户必须保留原始的版权声明和免责声明。

这可能看起来像一个奇怪的决定。然而，就像我们在上一章中介绍的 OpenWhisk 一样，Platform9 为他们的客户以及任何想要开始部署**函数即服务**（**FaaS**）的人提供了一个坚实的基础来构建他们的应用程序。他们不仅给了人们在任何地方部署他们的工作负载的自由，还能够为安装和 Fission 平台提供支持服务。

# 安装先决条件

在我们本地或公共云中安装 Fission 之前，我们需要一些支持工具。第一个工具我们已经安装了，那就是 Kubernetes 命令行接口`kubectl`。我们还没有安装运行 Fission 所需的第二个工具：Helm ([`helm.sh/`](http://helm.sh))。

# 安装 Helm

Helm 是 Kubernetes 的一个包管理器，是 Cloud Native Computing Foundation 的一部分，Bitnami、Google、Microsoft 和 Helm 社区都为其开发做出了贡献。

要在 macOS High Sierra 上安装 Helm，我们可以使用 Homebrew；只需运行：

```
$ brew install kubernetes-helm
```

如果您正在运行 Ubuntu Linux，则可以使用安装脚本下载并安装 Helm：

```
$ curl https://raw.githubusercontent.com/kubernetes/helm/master/scripts/get | bash
```

最后，Windows 10 专业版用户可以从 canary 存储库下载 Helm 的实验版本。该版本的直接下载链接为[`kubernetes-helm.storage.googleapis.com/helm-canary-windows-amd64.zip`](https://kubernetes-helm.storage.googleapis.com/helm-canary-windows-amd64.zip)。由于这是一个实验版本，我建议直接运行它，而不要将其放在系统文件夹中。

安装 Helm 的下一步需要您拥有一个运行中的 Kubernetes 集群，因为这是它的启动位置。我将在本章后面包括安装 Helm 的服务器组件 Tiller 的说明。

# 安装 Fission CLI

我们需要安装的最后一个命令行工具是 Fission 本身的工具。您可以通过在 macOS High Sierra 上运行以下命令来安装它：

```
$ curl -Lo fission https://github.com/fission/fission/releases/download/0.3.0/fission-cli-osx && chmod +x fission && sudo mv fission /usr/local/bin/
```

对于 Ubuntu 17.04，您可以运行：

```
$ curl -Lo fission https://github.com/fission/fission/releases/download/0.3.0/fission-cli-linux && chmod +x fission && sudo mv fission /usr/local/bin/
```

最后，Windows 可执行文件可以从[`github.com/fission/fission/releases/download/0.3.0/fission-cli-windows.exe`](https://github.com/fission/fission/releases/download/0.3.0/fission-cli-windows.exe)下载。我建议与 Helm 的可执行文件一起使用，而不是将其安装在`System32`文件夹中。

运行以下命令应该显示当前安装的版本：

```
$ helm version
$ fission --version
```

![](img/b4a654f6-8c00-4220-b565-f18d42b40e98.png)

如前所述，我们还没有安装 Tiller，因此我们可以安全地忽略关于无法连接到它的错误。

# 在本地运行 Fission

现在我们已经安装了先决条件，我们可以开始创建我们的第一个函数。为此，我们将使用 Minikube。要启动单节点集群，我们只需要运行以下命令：

```
$ minikube start
$ kubectl get nodes
```

这应该启动您的 Minikube 集群，并确认您的本地版本已重新配置以与其通信：

![](img/14b2aeba-7121-49c2-a883-32651ffb3353.png)

一旦我们的集群运行并且可访问，我们需要通过安装 Tiller 来完成 Helm 安装。要做到这一点，我们需要运行以下命令：

```
$ helm init
```

您应该会看到类似以下消息：

![](img/6407f211-02d3-41ab-8863-ba045b51a6e6.png)

# 使用 Helm 启动 Fission

Helm 现在已配置好，我们可以使用它来部署 Fission 的远程组件。可以通过运行以下命令来完成：

```
$ helm install --namespace fission --set serviceType=NodePort https://github.com/fission/fission/releases/download/0.4.0/fission-all-0.4.0.tgz
```

一两分钟后，您应该会收到 Fission 已启动的确认信息。

# 通过输出进行工作

Helm 的输出非常详细。它将为您提供它创建的所有内容的概述，以及开发人员包含的任何附加说明。

输出的这部分包含了部署的基本细节：

```
NAME: lopsided-fox
LAST DEPLOYED: Sat Dec 9 10:52:19 2017
NAMESPACE: fission
STATUS: DEPLOYED
```

接下来，我们会得到有关在 Kubernetes 中部署了什么的信息，从服务账户开始。这些提供运行 pod 的身份服务。这些允许 Fission 的各个组件与 Kubernetes 进行接口交互：

```
==> v1/ServiceAccount
NAME            SECRETS AGE
fission-builder 1       1m
fission-fetcher 1       1m
fission-svc     1       1m
```

然后是绑定。这些为集群提供基于角色的身份验证（RBAC）：

```
==> v1beta1/ClusterRoleBinding
NAME                 AGE
fission-builder-crd  1m
fission-crd          1m
fission-fetcher-crd  1m
```

接下来是服务本身：

```
==> v1/Service
NAME           TYPE        CLUSTER-IP  EXTERNAL-IP PORT(S)        AGE
poolmgr        ClusterIP   10.0.0.134  <none>      80/TCP         1m
buildermgr     ClusterIP   10.0.0.212  <none>      80/TCP         1m
influxdb       ClusterIP   10.0.0.24   <none>      8086/TCP       1m
nats-streaming NodePort    10.0.0.161  <none>      4222:31316/TCP 1m
storagesvc     ClusterIP   10.0.0.157  <none>      80/TCP         1m
controller     NodePort    10.0.0.55   <none>      80:31313/TCP   1m
router         NodePort    10.0.0.106  <none>      80:31314/TCP   1m
```

现在我们有了部署详情。您可能会注意到，如下所示，一些 pod 仍在启动，这就是为什么它们显示为零可用的原因：

```
==> v1beta1/Deployment
NAME.           DESIRED CURRENT UP-TO-DATE AVAILABLE AGE
timer           1       1       1          1         1m
poolmgr         1       1       1          1         1m
influxdb        1       1       1          1         1m
nats-streaming  1       1       1          1         1m
controller      1       1       1          1         1m
mqtrigger       1       1       1          1         1m
router          1       1       1          0         1m
storagesvc      1       1       1          0         1m
kubewatcher     1       1       1          1         1m
buildermgr      1       1       1          0         1m
```

接下来，我们有了部署和服务的 pod：

```
==> v1/Pod(related)
NAME                            READY STATUS            RESTARTS AGE
logger-zp65r                    1/1   Running           0        1m
timer-57f75c486f-9ktbk          1/1   Running           2        1m
poolmgr-69fcff7d7-hbq46         1/1   Running           1        1m
influxdb-c5c6cfd86-wkwrs        1/1   Running           0        1m
nats-streaming-85b9898784-h6j2v 1/1   Running           0        1m
controller-5f964bc987-mmfrx     1/1   Running           0        1m
mqtrigger-c85dd79f7-vj5p7       1/1   Running           0        1m
router-7cfff6794b-gn5pw         0/1   ContainerCreating 0        1m
storagesvc-58d5c8f6-bnqc7       0/1   ContainerCreating 0        1m
kubewatcher-6d784b9987-5wwhv    1/1   Running           0        1m
buildermgr-7ff69c8bb-pvtbx      0/1   ContainerCreating 0        1m
```

然后我们有了命名空间：

```
==> v1/Namespace
NAME.            STATUS AGE 
fission-builder  Active 1m
fission-function Active 1m
```

现在我们有了秘密。这些只是用于正在使用的数据库：

```
==> v1/Secret
NAME     TYPE   DATA AGE 
influxdb Opaque 2    1m
```

我们接近尾声了：持久存储索赔。您可以看到，由于我们在本地启动，它只是使用 VM 上的一个文件夹，而不是创建外部存储：

```
==> v1/PersistentVolumeClaim
NAME.               STATUS VOLUME                                   CAPACITY ACCESS MODES STORAGECLASS AGE
fission-storage-pvc Bound  pvc-082cf8d5-dccf-11e7-bfe6-080027e101f5 8Gi      RWO            standard     1m
```

现在我们有了角色绑定：

```
==> v1beta1/RoleBinding
NAME                   AGE
fission-function-admin 1m
fission-admin          1m
```

最后，我们有了守护进程集：

```
==> v1beta1/DaemonSet
NAME   DESIRED CURRENT READY UP-TO-DATE AVAILABLE NODE SELECTOR AGE
logger 1       1       1     1          1         <none>        1m
```

现在我们已经看到了我们的 Fission 安装的所有 Kubernetes 元素的概述，我们得到了如何与安装进行交互的说明。

# 启动我们的第一个函数

笔记分为三个部分；第一部分提供了如何安装 Fission 命令行客户端的说明。由于我们已经在本章的前一部分中涵盖了这一点，我们可以忽略这一步。

接下来，在第二部分中，我们得到了关于需要设置的环境变量的说明，以便我们的本地 Fission 客户端可以与我们的 Fission 安装进行交互。要设置这些变量，请运行以下命令：

```
$ export FISSION_URL=http://$(minikube ip):31313
$ export FISSION_ROUTER=$(minikube ip):31314 
```

`export`命令仅适用于 macOS High Sierra 和 Ubuntu 17.04。Windows 10 专业版用户将不得不运行以下命令：

```
$ for /f "delims=" %%a in ('minikube ip') do @set minikube_ip=%%a
$ set FISSION_URL=http://%minikube_ip%:31313
$ set FISSION_ROUTER=%minikube_ip%:31314
```

从这些命令中可以看出，我们的 Fission 安装知道它正在运行在 Minikube 安装上，并为我们提供了动态生成 Minikube 安装的 IP 地址的命令。

第三部分包含了一步一步的说明，说明如何运行一个 hello world 函数；让我们现在来运行这些步骤。

首先，我们需要创建一个环境。为此，我们使用以下命令：

```
$ fission env create --name nodejs --image fission/node-env
```

这个命令创建了一个名为`nodejs`的环境，然后指示 Fission 使用来自 Docker Hub 的 Docker 镜像`fission/node-env`——您可以在[`hub.docker.com/r/fission/node-env/`](https://hub.docker.com/r/fission/node-env/)找到这个镜像。

现在我们已经创建了环境，我们需要一个要部署的函数。运行以下命令（仅适用于 macOS 和 Linux）来下载 hello world 示例：

```
$ curl https://raw.githubusercontent.com/fission/fission/master/examples/nodejs/hello.js > /tmp/hello.js
```

这将下载以下代码：

```
module.exports = async function(context) {
    return {
        status: 200,
        body: "Hello, world!\n"
    };
}
```

如您所见，这与我们在早期章节中运行的示例并没有太大不同。现在我们已经下载了一个函数，我们可以使用以下命令部署它：

```
$ fission function create --name hello --env nodejs --code /tmp/hello.js
```

我们快要完成了；最后一步是创建一个到我们函数的路由。要做到这一点，使用以下命令：

```
$ fission route create --method GET --url /hello --function hello
```

现在我们应该能够通过发出 HTTP 请求来调用我们的函数。您可以使用以下命令中的任一个来触发我们的函数：

```
$ curl http://$FISSION_ROUTER/hello
$ http http://$FISSION_ROUTER/hello 
```

对于 Windows 10 专业版，请使用以下命令在 IE 中打开示例：

```
$ explorer http://%FISSION_ROUTER%/hello 
```

HTTPie 将为您提供标头，以及以下输出：

![](img/79722f78-5815-45eb-801c-40cd7b7abb1e.png)

# 一个留言板

现在我们已经有了一个基本的应用程序在运行，让我们来创建一些更复杂的东西。Fission 附带了一个演示应用程序，充当留言板。您可以在伴随本书的 GitHub 存储库中的`/Chapter08/guestbook/`文件夹中找到我们将要部署的文件。

启动应用程序的第一步是启动 Redis 部署；这将用于存储写入留言板的评论。要创建部署，请在`/Chapter08/guestbook/`文件夹中运行以下命令：

```
$ kubectl create -f redis.yaml
```

您可以从以下截图中看到，这创建了一个`namespace`、`deployment`和`service`。

![](img/c999a786-2b0b-42b6-a941-a1bfaf9f659e.png)

现在我们需要创建一个环境来启动我们的函数。由于应用程序是用 Python 编写的，让我们运行以下命令：

```
$ fission env create --name python --image fission/python-env
```

前面命令的输出显示在以下截图中：

![](img/37069961-48b9-4fef-8ac8-6be14d231f3c.png)

现在我们已经创建了两个函数，一个用于显示评论，一个用于写评论。要添加这些，请运行以下命令：

```
$ fission function create --name guestbook-get --env python --code get.py --url /guestbook --method GET
$ fission function create --name guestbook-add --env python --code add.py --url /guestbook --method POST
```

前面命令的输出可以在以下截图中看到：

![](img/79183e9b-e337-4a2e-ab96-8967a67d41b9.png)

您会注意到，用于添加函数的命令与我们在上一节中用于启动 hello world 示例的命令有些不同。在之前的示例中，我们既添加了函数，又创建了路由。您可能还注意到，虽然我们创建了两个函数，但它们都绑定到了相同的路由`/guestbook`。现在不讨论这个问题，让我们启动应用程序并与之交互。

要打开留言板，请运行以下命令：

```
$ open http://$FISSION_ROUTER/guestbook 
```

对于 Windows 10 专业版，请使用：

```
$ explorer http://%FISSION_ROUTER%/guestbook
```

这将在浏览器中打开一个空的留言板页面，如下截图所示：

![](img/b699b897-d8f4-4ef6-b4ce-88019c11b36b.png)

现在让我们通过输入一些文本（比如`Testing Fission`）来添加评论，然后单击添加。刷新后，您应该看到您的评论已添加：

![](img/9d18d10b-b9a8-4b97-858f-1d4dc284f2fd.png)

如果收到内部服务器错误，请不要担心，只需刷新页面并重新提交。查看页面的 HTML 源代码，您可能会注意到表单操作配置为将`POST`提交到`/guestbook`：

```
<form action="/guestbook" method="POST">
  <input type="text" name="text">
  <button type="submit">Add</button>
</form>
```

如果您查看我们用于创建两个函数的命令，您会注意到两者都附有一个方法。`guestbook-add`，运行`add.py`，使用了`POST`方法，如下面的代码所示：

```
#
# Handles POST /guestbook -- adds item to guestbook 
#

from flask import request, redirect
import redis

# Connect to redis.
redisConnection = redis.StrictRedis(host='redis.guestbook', port=6379, db=0)

def main():
    # Read the item from POST params, add it to redis, and redirect
    # back to the list
    item = request.form['text']
    redisConnection.rpush('guestbook', item)
    return redirect('/guestbook', code=303)
```

该函数读取表单提交的数据，将评论推送到 Redis 数据库，然后将我们带回`/guestbook`。`303`代码是在`POST`后重定向使用的状态代码。

每当您的浏览器请求页面时，它都会发送一个`GET`请求。在我们的情况下，所有对`/guestbook`的`GET`请求都被路由到`guestbook-get`函数，这是`get.py`代码：

```
#
# Handles GET /guestbook -- returns a list of items in the guestbook
# with a form to add more.
#

from flask import current_app, escape
import redis

# Connect to redis. This is run only when this file is loaded; as
# long as the pod is alive, the connection is reused.
redisConnection = redis.StrictRedis(host='redis.guestbook', port=6379, db=0)

def main():
    messages = redisConnection.lrange('guestbook', 0, -1)

    items = [("<li>%s</li>" % escape(m.decode('utf-8'))) for m in messages]
    ul = "<ul>%s</ul>" % "\n".join(items)
    return """
      <html><body style="font-family:sans-serif;font-size:2rem;padding:40px">
          <h1>Guestbook</h1> 
          <form action="/guestbook" method="POST">
            <input type="text" name="text">
            <button type="submit">Add</button>
          </form>
          <hr/>
          %s
      </body></html>
      """ % ul
```

从上面的代码中可以看出，这会连接到 Redis 数据库，读取每个条目，将结果格式化为无序的 HTML 列表，然后将列表插入到水平线下方（`<hr/>`）。

# Fission 命令

在我们将 Fission 安装移到公共云之前，我们应该更多地了解一下命令客户端。有几个顶级命令可用于管理我们的函数和路由。

# fission function 命令

这基本上是您在使用 Fission 时将花费大部分时间的地方。函数命令是您创建、管理和删除函数的方式。您可以使用`fission function <command>`或`fission fn <command>`。

# create 命令

我们已经使用过这个命令，所以不需要详细介绍。`fission function create` 命令有几个选项；最常见的是：

+   `--name`：这表示我们想要给我们的函数取什么名字。

+   `--env`：这表示我们想要在哪个环境中部署我们的函数。更多关于环境的内容请参见下一节。

+   `--code`：我们希望部署的代码的路径或 URL。

+   `--url`：我们希望我们的函数在哪个 URL 上可用。

+   `--method`：我们在前面 URL 上访问我们的函数的方式；这里的选项有`GET`、`POST`、`PUT`、`DELETE`、`HEAD`—如果您不使用`--method`但使用`--url`，它将始终默认为`GET`。

正如我们在留言板示例中已经看到的，`fission function create` 命令看起来会像下面这样：

```
$ fission function create \
 --name guestbook-get \
 --env python \
 --code get.py \
 --url /guestbook \
 --method GET
```

# 获取选项

这个选项相当简单；运行`fission function get`将显示您选择的函数的源代码。它接受一个输入：`--name`。这是您希望显示源代码的函数的名称。

运行以下命令将显示 hello world 函数的源代码：

```
$ fission function get --name hello 
```

![](img/136c1f31-a283-4f3b-a893-9e95fedadf5d.png)

# 列出和获取元数据命令

以下两个命令有点类似：

```
$ fission function list
```

这个命令将列出当前安装的函数。列表中包括函数的名称、唯一 ID 以及函数部署的环境：

![](img/de079f74-35db-4cea-a0c8-ce19c6b2f9dd.png)

如果我们已经知道函数的名称，并且想要提醒自己它正在运行的环境，或者需要它的 UID，那么我们可以使用 `fission function getmeta` 命令，并传递函数的名称：

```
$ fission function getmeta --name hello
```

![](img/30ec7ff7-1556-445e-ad25-d077acc4ffde.png)

# 日志命令

虽然目前没有任何视图，但您可以使用 `fission function logs` 命令查看函数的日志。您可以传递一些不同的选项：

+   `--name`：这是您希望查看日志的函数的名称，这总是必需的

+   `--follow`：保持流打开，日志实时显示

+   `--detail`：添加更多详细输出

使用上述选项，命令将看起来像下面这样：

```
$ fission function logs --detail --follow --name hello
```

然而，正如前面提到的，目前没有太多可看的。

# 更新命令

`fission function update` 命令部署函数的更新版本。它使用与 `fission function create` 命令相同的选项。例如，如果我们想要更新我们的 hello world 函数以使用不同的源，我们将运行以下命令：

```
$ fission function update \
 --name hello \
 --env nodejs \
 --code hello-update.js \
```

# 删除命令

我们要看的最后一个命令是 `fission function delete`。这个命令相当不言自明。它删除函数，只接受一个参数，那就是 `--name`。

在使用 `fission function delete` 时请小心；它不会以任何方式提示您，当您按下 *Enter* 时，您的函数将被删除。

要删除 hello world 函数，例如，我们只需运行以下命令：

```
$ fission function delete --name hello
```

![](img/fa8b8ea6-3c38-4779-b118-282ede814f46.png)如您所见，并且正如前面提到的，没有 *您确定吗？* 的提示，因此在使用命令时请小心。

# fission environment 命令

下一个顶级命令是 environment。正如我们已经看到的，环境是我们的函数运行的地方，它们还定义了我们的函数在哪种语言中执行。在撰写本文时，Fission 支持 Node.js、Go、Python、PHP、Ruby、Perl 和 .NET C#。

# 创建命令

`fission environment create` 命令是我们已经使用过的一个命令。例如，当我们创建 guestbook 应用程序时，我们需要一个 Python 环境来运行我们的应用程序，所以我们运行了以下命令：

```
$ fission environment create \
 --name python \
 --image fission/python-env
```

图像的完整列表、要使用的 URL 和用于创建图像的 Dockerfile 如下：

| **语言** | **图像名称** | **源 URL** |
| --- | --- | --- |
| Python 2.7 | `fission/python-env` | [`github.com/fission/fission/tree/master/environments/python`](https://github.com/fission/fission/tree/master/environments/python) |
| Python 3.5 | `fission/python3-env` | [`github.com/fission/fission/tree/master/environments/python`](https://github.com/fission/fission/tree/master/environments/python) |
| Node.js | `fission/nodejs-env` | [`github.com/fission/fission/tree/master/environments/nodejs`](https://github.com/fission/fission/tree/master/environments/nodejs) |
| .NET C# | `fission/dotnet-env` | [`github.com/fission/fission/tree/master/environments/dotnet`](https://github.com/fission/fission/tree/master/environments/dotnet) |
| .NET 2.0 C# | `fission/dotnet20-env` | [`github.com/fission/fission/tree/master/environments/dotnet20`](https://github.com/fission/fission/tree/master/environments/dotnet20) |
| Go | `fission/go-runtime` | [`github.com/fission/fission/tree/master/environments/go`](https://github.com/fission/fission/tree/master/environments/go) |
| PHP | `fission/php7-env` | [`github.com/fission/fission/tree/master/environments/php7`](https://github.com/fission/fission/tree/master/environments/php7) |
| Ruby | `fission/ruby-env` | [`github.com/fission/fission/tree/master/environments/ruby`](https://github.com/fission/fission/tree/master/environments/ruby) |
| Perl | `fission/perl-env` | [`github.com/fission/fission/tree/master/environments/perl`](https://github.com/fission/fission/tree/master/environments/perl) |

# 列出和获取命令

与函数命令一样，环境也有`list`和`get`命令，它们的工作方式也相同。

```
$ fission environment list
```

运行上一个命令将列出所有配置的环境。

```
$ fission environment get --name nodejs
```

运行上一个命令将获取命名环境的详细信息。

![](img/0e86e3f1-0777-4fe1-b51e-2417d7382cf4.png)

# 删除命令

`delete`命令再次按预期工作（请记住它会在没有警告的情况下删除）：

```
$ fission environment delete --name nodejs
```

此外，如果您的环境中有函数，它也将在没有警告的情况下被删除。但是，您的函数将保留，直到您手动删除它们。任何尝试调用没有环境的函数都将导致内部服务器错误。

# 在云中运行 Fission

现在我们知道了在本地运行 Fission 时启动和交互所涉及的内容，让我们看看在云中启动 Kubernetes，然后配置 Fission 在那里运行。

在本节的其余部分，我将仅提供 macOS High Sierra 和 Ubuntu 17.04 主机的说明，因为它们与我们将要运行的命令具有更高的兼容性。

# 启动 Kubernetes 集群

我将使用以下命令在 Google Cloud 中启动我的 Kubernetes：

```
$ gcloud container clusters create kube-cluster
```

前述命令的输出可以在以下截图中看到：

![](img/a9396fbc-ab7c-42f1-91b1-0622bee37be9.png)

一旦启动，最多需要大约 5 分钟，您可以使用以下方法检查您的集群是否按预期运行：

```
$ kubectl get nodes
```

前述命令的输出可以在以下截图中看到：

![](img/1493cd89-dfb1-41cb-8dce-23131d8674ab.png)

现在我们的三节点集群已经运行起来了，并且我们的本地 Kubernetes 客户端正在与之交互，我们可以再次运行以下命令来部署 Helm 的 Kubernetes 端：

```
$ helm init
```

这将返回以下消息：

![](img/e235d7be-a745-4516-8149-9345b8f5317d.png)

现在我们已经准备好了 Helm，我们可以继续启动 Fission。

# 安装 Fission

与之前一样，我们将使用 Helm 来安装 Fission。在本地安装 Fission 和在 Google Cloud、Microsoft Azure 或 AWS 等公共云上安装 Fission 之间唯一的区别是，我们不会使用`--set serviceType=NodePort`选项，而是直接运行以下命令：

```
$ helm install --namespace fission https://github.com/fission/fission/releases/download/0.4.0/fission-all-0.4.0.tgz
```

您可能会注意到这次运行速度要快得多，并且返回的信息与我们在本地单节点集群上启动 Fission 时非常相似。

您可能会注意到，这次您的安装有一个不同的名称：

```
NAME: orange-shark
LAST DEPLOYED: Sun Dec 10 13:46:02 2017
NAMESPACE: fission
STATUS: DEPLOYED
```

此名称用于在整个过程中引用安装，如您从 Google Cloud Web 控制台的工作负载页面中看到的屏幕截图所示：

![](img/eea42255-91b3-4c26-9e15-bd86210b99e4.png)

在控制台中，点击“发现和负载均衡”将显示分配给您的安装的所有外部 IP 地址。由于我们传递了`NodePort`选项，因此已创建了外部负载均衡器：

![](img/de0020bb-b6fb-4bed-b6f2-6c714b115af1.png)

在控制台中查看的最后一件事是存储页面。如您所见，外部块存储已创建并附加到您的安装中。这与我们在本地启动时不同，因为存储实际上是我们单台机器的存储：

![](img/e81c1074-edd4-49ba-9341-dcee15b2f08d.png)

回到命令行，你会注意到，Helm 再次给了我们关于如何完成本地 Fission 客户端配置的指令。然而，由于我们没有使用 Minikube，这次的指令略有不同。

这次设置`FISSION_URL`和`FISSION_ROUTER`变量的命令使用`kubectl`来查询我们的安装，以找出负载均衡器的外部 IP 地址：

```
 $ export FISSION_URL=http://$(kubectl --namespace fission get svc controller -o=jsonpath='{..ip}')
 $ export FISSION_ROUTER=$(kubectl --namespace fission get svc router -o=jsonpath='{..ip}')
```

你可以通过运行以下命令来检查 URL：

```
$ echo $FISSION_URL
$ echo $FISSION_ROUTER
```

这应该会给你类似以下的输出：

![](img/27b2dd66-4b9b-4a2a-a8ee-20857a4d0816.png)

现在我们已经安装了 Fission，并且我们的本地命令行客户端已配置为与我们的基于云的安装进行交互，我们可以通过运行以下命令快速重新运行 hello world 示例：

```
$ fission env create --name nodejs --image fission/node-env
$ curl https://raw.githubusercontent.com/fission/fission/master/examples/nodejs/hello.js > /tmp/hello.js
$ fission function create --name hello --env nodejs --code /tmp/hello.js --url /hello --method GET
```

这应该会给你类似以下的输出：

![](img/e039c924-6268-4b50-a00d-98c7707193a4.png)

一旦启动，你可以使用以下命令之一来调用该函数：

```
$ curl http://$FISSION_ROUTER/hello
$ http http://$FISSION_ROUTER/hello
```

这应该会给你类似以下的输出：

![](img/9395a699-b5b0-467e-926f-ced6004eb30c.png)

正如你已经看到的，就像我们所看到的所有技术一样，一旦安装，与公共云中的 Fission 交互和使用与在本地运行时并无不同。你真的不需要太在意外部访问等等，因为 Fission 和 Kubernetes 都已经为你解决了这个问题。

# guestbook

在我们继续更高级的示例之前，让我们快速再次启动我们的 guestbook 应用程序。要做到这一点，切换到存储库中的`/Chapter08/guestbook/`文件夹，然后运行以下命令：

```
$ kubectl create -f redis.yaml
$ fission env create --name python --image fission/python-env
$ fission function create --name guestbook-get --env python --code get.py --url /guestbook --method GET
$ fission function create --name guestbook-add --env python --code add.py --url /guestbook --method POST
$ open http://$FISSION_ROUTER/guestbook
```

这应该会给你类似以下的输出：

![](img/87759a92-2586-497c-bf0a-32689b6c97e8.png)

这将启动应用程序，并且还会在浏览器中打开，你可以在其中添加评论：

![](img/0abac7c0-97e9-4abe-beb4-bd6d120aa78b.png)

# 更多示例

在我们结束本章之前，让我们看一些在 Fission 中运行的示例代码，首先是一个天气检查器。

# 天气

在存储库的`/Chapter08/weather/`文件夹中，你会找到`weather.js`。这是一个简单的 Node.js 函数，用于查询 Yahoo 天气 API 以返回给定位置的当前天气：

```
'use strict';

const rp = require('request-promise-native');

module.exports = async function (context) {
    const stringBody = JSON.stringify(context.request.body);
    const body = JSON.parse(stringBody);
    const location = body.location;

    if (!location) {
        return {
            status: 400,
            body: {
                text: 'You must provide a location.'
            }
        };
    }

    try {
        const response = await rp(`https://query.yahooapis.com/v1/public/yql?q=select item.condition from weather.forecast where woeid in (select woeid from geo.places(1) where text="${location}") and u="c"&format=json`);
        const condition = JSON.parse(response).query.results.channel.item.condition;
        const text = condition.text;
        const temperature = condition.temp;
        return {
            status: 200,
            body: {
                text: `It is ${temperature} celsius degrees in ${location} and ${text}`
            },
            headers: {
                'Content-Type': 'application/json'
            }
        };
    } catch (e) {
        console.error(e);
        return {
            status: 500,
            body: e
        };
    }
}
```

正如你从前面的代码中看到的，该函数接受 JSON 编码的数据，其中必须包含一个有效的位置。因此，我们需要使用`POST`路由部署该函数，并且如果没有传递位置数据，它会报错，所以我们还应该部署一个`GET`路由。要做到这一点，只需在`/Chapter08/weather/`文件夹中运行以下命令：

```
$ fission env create --name nodejs --image fission/node-env
$ fission function create --name weather --env nodejs --code weather.js --url /weather --method POST
$ fission route create --method GET --url /weather --function weather
```

如果你已经在终端输出中看到了我们最初为 hello world 示例创建并运行的环境，那么第一个命令可能会导致错误：

![](img/81dd0f50-1acc-4c96-a450-df575172498e.png)

现在我们已经部署了我们的函数，可以通过运行以下两个命令之一来快速测试它：

```
$ http http://$FISSION_ROUTER/weather
$ curl http://$FISSION_ROUTER/weather
```

因为我们没有提供位置，所以你应该会看到以下消息：

![](img/a0fdce7d-d631-4a8c-a887-dcb6246727bc.png)

这正是代码的预期行为。正如你所看到的，它返回了一个`400`错误和我们预期的消息。通过运行以下命令之一提供位置（我使用了`英格兰诺丁汉`）应该会告诉你天气情况：

```
$ http POST http://$FISSION_ROUTER/weather location="Nottingham, England"
$ curl -H "Content-Type: application/json" -X POST -d '{"location":"Nottingham, England"}' http://$FISSION_ROUTER/weather 
```

你可以从下面的终端输出中看到，它已经确认了我所在的地方目前天气并不是很好：

![](img/2a20414b-ca48-4dfc-9068-f4c4c69d2a39.png)

# Slack

在这个例子中，我们将在当前 Kubernetes 安装的默认命名空间中每次创建或删除服务时发布一条消息。这些消息将通过名为 Slack 的 Webhook 发布到一个名为 Slack 的群组消息服务中。

Slack 是一个在线协作工具，允许团队在其中与聊天机器人和其他人进行交互。它提供免费和付费的服务，以及一个详尽的 API 供你的应用程序连接到你的聊天室。

我将假设你已经可以访问 Slack 工作空间，并且有权限向其添加应用程序。如果没有，那么你可以在[`slack.com/`](https://slack.com)配置一个新的工作空间。

一旦你进入了你的工作空间，点击屏幕左上角的工作空间名称，然后从下拉列表中选择“管理应用程序”。这将带你进入 Slack 的**应用程序目录**。在这里，在页面顶部的搜索框中输入`Incoming WebHooks`，选择结果，然后点击“添加配置”按钮。

按照屏幕上的说明创建你选择的频道的 Webhook。我选择在随机频道发布我的更新，我还自定义了图标。在这个页面上，你还会找到一个 Webhook URL。我的（现在已被删除）是 `https://hooks.slack.com/services/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`。

记下这一点，因为我们需要用它来更新代码。正如你在下面的代码中所看到的，你也可以在 `/Chapter08/slack/` 仓库中找到，第三行需要用你的 Webhook 详细信息进行更新：

```
'use strict';

let https = require('https');

const slackWebhookPath = "/put/your/url/here"; // Something like "/services/XXX/YYY/zZz123"

function upcaseFirst(s) {
    return s.charAt(0).toUpperCase() + s.slice(1).toLowerCase();
}

async function sendSlackMessage(msg) {
    let postData = `{"text": "${msg}"}`;
    let options = {
        hostname: "hooks.slack.com",
        path: slackWebhookPath,
        method: "POST",
        headers: {
            "Content-Type": "application/json"
        }
    };

    return new Promise(function(resolve, reject) {
        let req = https.request(options, function(res) {
            console.log(`slack request status = ${res.statusCode}`);
            return resolve();
        });
        req.write(postData);
        req.end();
    });
}

module.exports = async function(context) {
    console.log(context.request.headers);

    let obj = context.request.body;
    let version = obj.metadata.resourceVersion;
    let eventType = context.request.get('X-Kubernetes-Event-Type');
    let objType = context.request.get('X-Kubernetes-Object-Type');

    let msg = `${upcaseFirst(eventType)} ${objType} ${obj.metadata.name}`;
    console.log(msg, version);

    if (eventType == 'DELETED' || eventType == 'ADDED') {
        console.log("sending event to slack")
        await sendSlackMessage(msg);
    }

    return {
        status: 200,
        body: ""
    }
}
```

为了做到这一点，粘贴`https://hooks.slack.com`后面的所有内容，包括斜杠(`/`)。对我来说，这是`/services/T8F3CR4GG/B8FNRR3PC/wmLSDgS0fl5SGOcAgNjwr6pC`。

该行应该类似于以下内容：

```
const slackWebhookPath = "/services/T8F3CR4GG/B8FNRR3PC/wmLSDgS0fl5SGOcAgNjwr6pC"; // Something like "/services/XXX/YYY/zZz123"
```

确保文件名为 `kubeEventsSlack.js`，一旦你的 Webhook 详细信息在代码中，我们可以使用以下命令创建和启动函数：

```
$ fission function create --name kubeslack --env nodejs --code kubeEventsSlack.js
```

函数创建后，我们需要创建一些东西来触发它。以前，我们一直在使用 HTTP 调用来调用函数。不过这一次，我们希望在我们的 Kubernetes 集群中发生某些事情时触发函数。为此，我们需要创建一个观察。

为了做到这一点，运行以下命令：

```
$ fission watch create --function kubeslack --type service --ns default
```

`fission watch` 命令是我们尚未讨论过的内容，所以让我们花点时间了解一下更多关于它的信息。

作为我们 Fission 部署的一部分，有一个名为 `kubewatcher` 的服务。默认情况下，Fission 使用这个服务来通过观察 Kubernetes API 来帮助管理自身，但也向最终用户公开。用于创建之前观察的命令创建了一个观察者，它每次在默认命名空间中对服务进行更改时调用我们的函数(`--function kubeslack`)。我们还可以设置一个观察，以查找对 pods、deployments 等的更改，通过更改类型：

![](img/90b42045-f5c6-46a0-b78f-2d663350d73c.png)

现在我们需要在默认命名空间中启动一个服务。为此，切换到 `/Chapter03/` 文件夹并运行以下命令：

```
$ kubectl apply -f cli-hello-world.yml
```

然后，通过运行以下命令删除服务：

```
$ kubectl delete service cli-hello-world 
```

![](img/8eef86e2-c090-45d1-8966-a81edfa74be6.png)

如果你检查 Slack，你应该会看到两条消息，确认一个名为 `cli-hello-world` 的服务已被添加和删除：

![](img/c9bcd145-2011-4126-9bbe-079d7e684278.png)

你应该几乎实时地看到这种情况发生，你可能还会看到有关在默认命名空间内启动其他服务的消息。

# 鲸鱼

接下来，也是最后一个例子，我们要看的是一个二进制环境。这个环境与我们一直在看的环境不同，因为它不包含编程语言。相反，我们将部署一个安装和配置名为`cowsay`的 Unix 工具的 bash 脚本。代码如下，并且位于`/Chapter08/whale/`文件夹中：

```
#!/bin/sh

if ! hash cowsay 2> /dev/null; then
    apk update > /dev/null
    apk add curl perl > /dev/null
    curl https://raw.githubusercontent.com/docker/whalesay/master/cowsay > /bin/cowsay 2> /dev/null
    chmod +x /bin/cowsay
    mkdir -p /usr/local/share/cows/
    curl https://raw.githubusercontent.com/docker/whalesay/master/docker.cow > /usr/local/share/cows/default.cow 2> /dev/null
fi

cowsay
```

如你所见，bash 脚本有两个部分。第一部分运行`cowsay`命令，如果出错，它将使用`apk`来安装`curl`和`perl`。安装完成后，它会下载代码副本，并配置默认行为。然后在安装后运行`cowsay`命令。

也许你会想，APK 是什么，`cowsay`又是什么？由于部署到 Fission 环境中的容器运行的是 Alpine Linux，我们需要使用**Alpine 软件包管理器**（**APK**）来安装我们代码运行所需的必要软件包。

Alpine Linux 是一个 Linux 发行版，在过去的两年里开始在更传统的 Ubuntu/CentOS 安装中获得了很多关注，这是因为它的体积。Alpine Linux 的基本安装只需 8MB 的空间。然而，尽管它很小，但它仍然和其他 Linux 发行版一样功能强大。它小巧的体积加上强大的功能，使其成为构建容器的完美操作系统。

`cowsay`是一个 Unix 命令，它会在一个来自牛的对话气泡中重复你给它的任何输入，因此得名`cowsay`。我们将安装 Docker 自己的版本`cowsay`，它使用的是鲸鱼而不是牛。要部署二进制函数，我们首先需要创建环境：

```
$ fission env create --name binary --image fission/binary-env
```

现在我们可以部署函数并创建`POST`和`GET`路由，以便我们可以访问它：

```
$ fission function create --name whalesay --env binary --deploy whalesay.sh --url /whale --method POST
$ fission route create --method GET --url /whale --function whalesay
```

上述命令的输出如下截图所示：

![](img/d956143c-5c1e-4c68-a677-67e5aba546a7.png)

现在我们已经部署了我们的函数，我们可以使用以下之一来访问它：

```
$ http http://$FISSION_ROUTER/whale
$ curl http://$FISSION_ROUTER/whale
```

这将返回一个 ASCII 鲸鱼，如下终端输出所示：

![](img/28566bb9-7ffc-483d-8579-f670243cb692.png)

你可能会注意到对话框中没有任何内容；那是因为我们需要`POST`一些东西。与之前的示例不同，我们启动的函数将简单地重复我们发布的任何内容。因此，如果我们`POST`一个 JSON 对象，它将返回 JSON 对象。因此，我们将只发布纯文本：

```
$ echo 'Hello from Whalesay !!!' | http POST http://$FISSION_ROUTER/whale
$ curl -X POST -H "Content-Type: text/plain" --data 'Hello from Whalesay !!!' http://$FISSION_ROUTER/whale
```

正如你可以从以下终端输出中看到的那样，这将返回我们发布的消息：

![](img/9e37db66-c2f4-43b5-9be1-6296c995ce70.png)

现在，你可能会认为这似乎是一个相当愚蠢的例子。然而，我们在这里所做的是获取 HTTP 请求的内容，并将其发布到 Linux 二进制文件中，然后使用我们发布的内容执行它。然后，我们通过 HTTP 请求返回运行命令的输出。

此时，您可能希望终止/关闭您已经启动以测试 Fission 的任何 Kubernetes 集群。

# 摘要

在本章中，我们已经研究了 Fission。我们使用 Helm 进行了安装，并在本地和 Google Cloud 上部署了它。我们还启动了几个测试应用程序，一些基本的，一些调用第三方服务来发布和返回信息。在安装和配置示例应用程序的过程中，我希望您开始看到 Fission 的用处以及它和其他无服务器技术如何集成到您自己的应用程序中。

当我开始写这一章时，我希望包括一些关于 Fission 工作流和 Fission UI 的部分。然而，在写作时，这两个附加组件都无法正常工作。现在，不要误会，Fission 是一种强大且易于使用的技术；然而，它非常新，并且仍在开发中，就像 Kubernetes 一样——这意味着在代码基础变得更加稳定之前，新版本中会有破坏性的更新。

例如，我们安装的 Fission 版本 0.4.0 是因为在写作时，最新版本的 Kubernetes 1.8 删除了`ThirdPartyResources`功能，并用`CustomResourceDefinitions`替换，这意味着旧版本的 Fission 将无法在当前版本的 Kubernetes 上运行。

我们将在剩下的章节中研究 Kubernetes 的发布周期以及这可能对您产生的影响。
