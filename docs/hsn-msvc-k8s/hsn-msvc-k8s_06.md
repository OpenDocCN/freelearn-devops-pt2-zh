# 第六章：在 Kubernetes 上保护微服务

在本章中，我们将深入研究如何在 Kubernetes 上保护您的微服务。这是一个广泛的主题，我们将专注于对于在 Kubernetes 集群中构建和部署微服务的开发人员最相关的方面。您必须非常严格地处理安全问题，因为您的对手将积极尝试找到漏洞，渗透您的系统，访问敏感信息，运行僵尸网络，窃取您的数据，破坏您的数据，销毁您的数据，并使您的系统不可用。安全性应该设计到系统中，而不是作为事后的附加物。我们将通过涵盖一般安全原则和最佳实践来解决这个问题，然后深入探讨 Kubernetes 提供的安全机制。

在本章中，我们将涵盖以下主题：

+   应用健全的安全原则

+   用户帐户和服务帐户之间的区分

+   使用 Kubernetes 管理机密

+   使用 RBAC 管理权限

+   通过身份验证、授权和准入控制访问

+   通过使用安全最佳实践来加固 Kubernetes

# 技术要求

在本章中，我们将查看许多 Kubernetes 清单，并使 Delinkcious 更安全。没有必要安装任何新内容。

# 代码

代码分为两个 Git 存储库：

+   您可以在此处找到代码示例：[`github.com/PacktPublishing/Hands-On-Microservices-with-Kubernetes/tree/master/Chapter06`](https://github.com/PacktPublishing/Hands-On-Microservices-with-Kubernetes/tree/master/Chapter06)

+   您可以在此处找到更新的 Delinkcious 应用程序：[`github.com/the-gigi/delinkcious/releases/tag/v0.4`](https://github.com/the-gigi/delinkcious/releases/tag/v0.4)

# 应用健全的安全原则

有许多通用原则。让我们回顾最重要的原则，并了解它们如何帮助防止攻击，使攻击变得更加困难，从而最小化任何攻击造成的损害，并帮助从这些攻击中恢复：

+   **深度防御**：深度防御意味着多层和冗余的安全层。其目的是使攻击者难以破坏您的系统。多因素身份验证是一个很好的例子。您有用户名和密码，但您还必须输入发送到您手机的一次性代码。如果攻击者发现了您的凭据，但无法访问您的手机，他们将无法登录系统并造成破坏。深度防御有多个好处，例如：

+   使您的系统更安全

+   使攻破您的安全成本过高，以至于攻击者甚至不会尝试

+   更好地保护免受非恶意错误

+   **最小权限原则**：最小权限原则类似于间谍世界中著名的“需要知道基础”。您不能泄露您不知道的东西。您无法破坏您无权访问的东西。任何代理都可能被破坏。仅限制到必要的权限将在违规发生时最小化损害，并有助于审计、缓解和分析事件。

+   **最小化攻击面**：这个原则非常明确。您的攻击面越小，保护起来就越容易。请记住以下几点：

+   不要暴露您不需要的 API

+   不要保留您不使用的数据

+   不要提供执行相同任务的不同方式

最安全的代码是根本不写代码。这也是最高效和无 bug 的代码。非常谨慎地考虑要添加的每个新功能的业务价值。在迁移到一些新技术或系统时，请确保不要留下遗留物品。除了防止许多攻击向量外，当发生违规时，较小的攻击面将有助于集中调查并找到根本原因。

+   **最小化爆炸半径**：假设您的系统将被破坏或可能已经被破坏。然而，威胁的级别是不同的。最小化爆炸半径意味着受损组件不能轻易接触其他组件并在系统中传播。这也意味着对这些受损组件可用的资源不超过应该在那里运行的合法工作负载的需求。

+   **不要相信任何人**：以下是您不应该信任的实体的部分列表：

+   您的用户

+   您的合作伙伴

+   供应商

+   您的云服务提供商

+   开源开发人员

+   您的开发人员

+   您的管理员

+   你自己

+   你的安全

当我们说*不要相信*时，我们并不是恶意的。每个人都是可犯错误的，诚实的错误可能会和有针对性的攻击一样有害。*不要相信任何人*原则的伟大之处在于你不必做出判断。最小信任的相同方法将帮助你预防和减轻错误和攻击。

+   **保守一点**：林迪效应表明，对于一些不易腐烂的东西，它们存在的时间越长，你就可以期待它们存在的时间越长。例如，如果一家餐厅存在了 20 年，你可以期待它还会存在很多年，而一个刚刚开业的全新餐厅更有可能在短时间内关闭。这对软件和技术来说非常真实。最新的 JavaScript 框架可能只有短暂的寿命，但像 jQuery 这样的东西会存在很长时间。从安全的角度来看，使用更成熟和经过严峻考验的软件会有其他好处，其安全性经受了严峻考验。从别人的经验中学习往往更好。考虑以下事项：

+   不要升级到最新和最好的（除非明确修复了安全漏洞）。

+   更看重稳定性而不是能力。

+   更看重简单性而不是强大性。

这与*不要相信任何人*原则相辅相成。不要相信新的闪亮东西，也不要相信你当前依赖的新版本。当然，微服务和 Kubernetes 是相对较新的技术，生态系统正在快速发展。在这种情况下，我假设你已经做出了决定，认为这些创新的整体好处和它们当前的状态已经足够成熟，可以进行建设。

+   **保持警惕**：安全不是一劳永逸的事情。你必须积极地继续努力。以下是一些你应该执行的全球性持续活动和流程：

+   定期打补丁。

+   旋转你的秘密。

+   使用短寿命的密钥、令牌和证书。

+   跟进 CVEs。

+   审计一切。

+   测试你系统的安全性。

+   **做好准备**：当不可避免的违规发生时，做好准备并确保你已经或正在做以下事情：

+   建立一个事件管理协议。

+   遵循你的协议。

+   堵住漏洞。

+   恢复系统安全。

+   进行安全事件的事后分析。

+   评估和学习。

+   更新你的流程、工具和安全性以提高你的安全姿态。

+   **不要编写自己的加密算法**：很多人对加密算法感到兴奋和/或失望，当强加密影响性能时。控制你的兴奋和/或失望。让专家来做加密。这比看起来要困难得多，风险也太高了。

既然我们清楚了良好安全性的一般原则，让我们来看看 Kubernetes 在安全方面提供了什么。

# 区分用户账户和服务账户

账户是 Kubernetes 中的一个核心概念。对 Kubernetes API 服务器的每个请求都必须来自一个特定的账户，API 服务器将在进行操作之前对其进行身份验证、授权和准入。有两种类型的账户：

+   用户账户

+   服务账户

让我们来检查两种账户类型，并了解它们之间的区别以及在何时适合使用每种类型。

# 用户账户

用户账户是为人类（集群管理员或开发人员）设计的，他们通常通过 kubectl 或以编程方式从外部操作 Kubernetes。最终用户不应该拥有 Kubernetes 用户账户，只能拥有应用级别的用户账户。这与 Kubernetes 无关。请记住，Kubernetes 会为您管理容器-它不知道容器内部发生了什么，也不知道您的应用实际在做什么。

您的用户凭据存储在`~/.kube/config`文件中。如果您正在使用多个集群，则您的`~/.kube/config`文件中可能有多个集群、用户和上下文。有些人喜欢为每个集群单独创建一个配置文件，并使用`KUBECONFIG`环境变量在它们之间切换。这取决于您。以下是我本地 Minikube 集群的配置文件：

```
apiVersion: v1
clusters:
- cluster:
    certificate-authority: /Users/gigi.sayfan/.minikube/ca.crt
    server: https://192.168.99.123:8443
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

正如您在上面的代码块中所看到的，这是一个遵循典型 Kubernetes 资源约定的 YAML 文件，尽管它不是您可以在集群中创建的对象。请注意，一切都是复数形式：集群、上下文、用户。在这种情况下，只有一个集群和一个用户。但是，您可以创建多个上下文，这些上下文是集群和用户的组合，这样您就可以在同一个集群中拥有不同权限的多个用户，甚至在同一个 Minikube 配置文件中拥有多个集群。`current-context`确定了`kubectl`的每个操作的目标（使用哪个用户凭据访问哪个集群）。用户账户具有集群范围，这意味着我们可以访问任何命名空间中的资源。

# 服务账户

服务帐户是另一回事。每个 pod 都有一个与之关联的服务帐户，并且在该 pod 中运行的所有工作负载都使用该服务帐户作为其身份。服务帐户的范围限定为命名空间。当您创建一个 pod（直接或通过部署）时，可以指定一个服务帐户。如果创建 pod 而没有指定服务帐户，则使用命名空间的默认服务帐户。每个服务帐户都有一个与之关联的秘密，用于与 API 服务器通信。

以下代码块显示了默认命名空间中的默认服务帐户：

```
$ kubectl get sa default -o yaml
apiVersion: v1
kind: ServiceAccount
metadata:
 creationTimestamp: 2019-01-11T15:49:27Z
 name: default
 namespace: default
 resourceVersion: "325"
 selfLink: /api/v1/namespaces/default/serviceaccounts/default
 uid: 79e17169-15b8-11e9-8591-0800275914a6
secrets:
- name: default-token-td5tz
```

服务帐户可以有多个秘密。我们很快将讨论秘密。服务帐户允许在 pod 中运行的代码与 API 服务器通信。

您可以从`/var/run/secrets/kubernetes.io/serviceaccount`获取令牌和 CA 证书，然后通过授权标头传递这些凭据来构造`REST HTTP`请求。例如，以下代码块显示了在默认命名空间中列出 pod 的请求：

```
# TOKEN=$(cat /var/run/secrets/kubernetes.io/serviceaccount/token)
# CA_CERT=$(cat /var/run/secrets/kubernetes.io/serviceaccount/ca.crt)
# URL="https://${KUBERNETES_SERVICE_HOST}:${KUBERNETES_SERVICE_PORT}"

# curl --cacert "$CERT" -H "Authorization: Bearer $TOKEN" "$URL/api/v1/namespaces/default/pods"
{
  "kind": "Status",
  "apiVersion": "v1",
  "metadata": {

  },
  "status": "Failure",
  "message": "pods is forbidden: User \"system:serviceaccount:default:default\" cannot list resource \"pods\" in API group \"\" in the namespace \"default\"",
  "reason": "Forbidden",
  "details": {
    "kind": "pods"
  },
  "code": 403
}
```

结果是 403 禁止。默认服务帐户不允许列出 pod，实际上它不允许做任何事情。在`授权`部分，我们将看到如何授予服务帐户权限。

如果您不喜欢手动构造 curl 请求，也可以通过客户端库以编程方式执行。我创建了一个基于 Python 的 Docker 镜像，其中包括 Kubernetes 的官方 Python 客户端库（[`github.com/kubernetes-client/python`](https://github.com/kubernetes-client/python)）以及一些其他好东西，如 vim、IPython 和 HTTPie。

这是构建镜像的 Dockerfile：

```
FROM python:3

RUN apt-get update -y
RUN apt-get install -y vim
RUN pip install kubernetes \
                httpie     \
                ipython

CMD bash
```

我将其上传到 DockerHub 作为`g1g1/py-kube:0.2`。现在，我们可以在集群中将其作为一个 pod 运行，并进行良好的故障排除或交互式探索会话：

```
$ kubectl run trouble -it --image=g1g1/py-kube:0.2 bash
```

执行上述命令将使您进入一个命令行提示符，您可以在其中使用 Python、IPython、HTTPie 以及可用的 Kubernetes Python 客户端包做任何您想做的事情。以下是我们如何从 Python 中列出默认命名空间中的 pod：

```
# ipython
Python 3.7.2 (default, Feb  6 2019, 12:04:03)
Type 'copyright', 'credits' or 'license' for more information
IPython 7.2.0 -- An enhanced Interactive Python. Type '?' for help.

In [1]: from kubernetes import client, config
In [2]: config.load_incluster_config()
In [3]: api = client.CoreV1Api()
In [4]: api.list_namespaced_pod(namespace='default')

```

结果将类似 - 一个 Python 异常 - 因为默认帐户被禁止列出 pod。请注意，如果您的 pod 不需要访问 API 服务器（非常常见），您可以通过设置`automountServiceAccountToken: false`来明确表示。

这可以在服务账户级别或 pod 规范中完成。这样，即使在以后有人或某物在您的控制之外添加了对服务账户的权限，由于没有挂载令牌，pod 将无法对 API 服务器进行身份验证，并且不会获得意外访问。Delinkcious 服务目前不需要访问 API 服务器，因此通过遵循最小权限原则，我们可以将其添加到部署中的规范中。

以下是如何为 LinkManager 创建服务账户（无法访问 API 服务器）并将其添加到部署中：

```
apiVersion: v1
kind: ServiceAccount
metadata:
  name: link-manager
  automountServiceAccountToken: false
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: link-manager
  labels:
    svc: link
    app: manager
spec:
  replicas: 1
  selector:
    matchLabels:
      svc: link
      app: manager
  serviceAccountName: link-manager
...
```

在使用 RBAC 授予我们的服务账户超级权限之前，让我们回顾一下 Kubernetes 如何管理秘密。Kubernetes 默认情况下将秘密存储在 etcd 中。可以将 etcd 与第三方解决方案集成，但在本节中，我们将专注于原始的 Kubernetes。秘密应该在静止和传输中进行加密，etcd 自版本 3 以来就支持了这一点。

现在我们了解了 Kubernetes 中账户如何工作，让我们看看如何管理秘密。

# 使用 Kubernetes 管理秘密

在使用 RBAC 授予我们的服务账户超级权限之前，让我们回顾一下 Kubernetes 如何管理秘密。Kubernetes 默认情况下将秘密存储在 etcd ([`coreos.com/etcd/`](https://coreos.com/etcd/))中。Kubernetes 可以管理不同类型的秘密。让我们看看各种秘密类型，然后创建我们自己的秘密并将它们传递给容器。最后，我们将一起构建一个安全的 pod。

# 了解 Kubernetes 秘密的三种类型

有三种不同类型的秘密：

+   服务账户 API 令牌（用于与 API 服务器通信的凭据）

+   注册表秘密（用于从私有注册表中拉取图像的凭据）

+   不透明秘密（Kubernetes 一无所知的您的秘密）

服务账户 API 令牌是每个服务账户内置的（除非您指定了`automountServiceAccountToken: false`）。这是`link-manager`的服务账户 API 令牌的秘密：

```
$ kubectl get secret link-manager-token-zgzff | grep link-manager-token
link-manager-token-zgzff   kubernetes.io/service-account-token  3   20h
```

`pull secrets`图像稍微复杂一些。不同的私有注册表行为不同，并且需要不同的秘密。此外，一些私有注册表要求您经常刷新令牌。让我们以 DockerHub 为例。DockerHub 默认情况下允许您拥有单个私有存储库。我将`py-kube`转换为私有存储库，如下截图所示：

！[](assets/bf7fadd6-8d83-4238-9652-66c89f6bd039.png)

我删除了本地 Docker 镜像。要拉取它，我需要创建一个注册表机密：

```
$ kubectl create secret docker-registry private-dockerhub \
 --docker-server=docker.io \
 --docker-username=g1g1 \
 --docker-password=$DOCKER_PASSWORD \
 --docker-email=$DOCKER_EMAIL
secret "private-dockerhub" created
$ kubectl get secret private-dockerhub
NAME                TYPE                             DATA      AGE
private-dockerhub   kubernetes.io/dockerconfigjson   1         16s
```

最后一种类型的机密是`Opaque`，是最有趣的机密类型。您可以将敏感信息存储在 Kubernetes 不会触及的不透明机密中。它只为您提供了一个强大且安全的机密存储库，并提供了一个用于创建、读取和更新这些机密的 API。您可以通过许多方式创建不透明机密，例如以下方式：

+   从文字值

+   从文件或目录

+   从`env`文件（单独行中的键值对）

+   使用`kind`创建一个 YAML 清单

这与 ConfigMaps 非常相似。现在，让我们创建一些机密。

# 创建您自己的机密

创建机密的最简单和最有用的方法之一是通过包含键值对的简单`env`文件：

```
a=1
b=2
```

我们可以使用`-o yaml`标志（YAML 输出格式）来创建一个机密，以查看创建了什么：

```
$ kubectl create secret generic generic-secrets --from-env-file=generic-secrets.txt -o yaml

apiVersion: v1
data:
 a: MQ==
 b: Mg==
kind: Secret
metadata:
 creationTimestamp: 2019-02-16T21:37:38Z
 name: generic-secrets
 namespace: default
 resourceVersion: "1207295"
 selfLink: /api/v1/namespaces/default/secrets/generic-secrets
 uid: 14e1db5c-3233-11e9-8e69-0800275914a6
type: Opaque
```

类型是`Opaque`，返回的值是 base64 编码的。要获取这些值并解码它们，您可以使用以下命令：

```
$ echo -n $(kubectl get secret generic-secrets -o jsonpath="{.data.a}") | base64 -D
1
```

`jsonpath`输出格式允许您深入到对象的特定部分。如果您喜欢，您也可以使用`jq`（[`stedolan.github.io/jq/`](https://stedolan.github.io/jq/)）。

请注意，机密不会被存储或传输；它们只是以 base-64 进行加密或编码，任何人都可以解码。当您使用您的用户帐户创建机密（或获取机密）时，您会得到解密后机密的 base-64 编码表示。但是，它在磁盘上是加密的，并且在传输过程中也是加密的，因为您是通过 HTTPS 与 Kubernetes API 服务器通信的。

现在我们已经了解了如何创建机密，我们将使它们可用于在容器中运行的工作负载。

# 将机密传递给容器

有许多方法可以将机密传递给容器，例如以下方法：

+   您可以将机密嵌入到容器镜像中。

+   您可以将它们传递到环境变量中。

+   您可以将它们挂载为文件。

最安全的方式是将你的秘密作为文件挂载。当你将你的秘密嵌入到镜像中时，任何有权访问镜像的人都可以检索你的秘密。当你将你的秘密作为环境变量传递时，它们可以通过`docker inspect`、`kubectl describe pod`以及如果你不清理环境的话，子进程也可以查看。此外，通常在报告错误时记录整个环境，这需要所有开发人员的纪律来清理和编辑秘密。挂载的文件不会受到这些弱点的影响，但请注意，任何可以`kubectl exec`进入你的容器的人都可以检查任何挂载的文件，包括秘密，如果你不仔细管理权限的话。

让我们从一个 YAML 清单中创建一个秘密。选择这种方法时，你有责任对值进行 base64 编码：

```
$ echo -n top-secret | base64
dG9wLXNlY3JldA==

$ echo -n bottom-secret | base64
Ym90dG9tLXNlY3JldA==

apiVersion: v1
kind: Secret
type: Opaque
metadata:
  name: generic-secrets2
  namespace: default
data:
  c: dG9wLXNlY3JldA==
  d: Ym90dG9tLXNlY3JldA==
```

让我们创建新的秘密，并通过使用`kubectl get secret`来验证它们是否成功创建：

```
$ kubectl create -f generic-secrets2.yaml
secret "generic-secrets2" created

$ echo -n $(kubectl get secret generic-secrets2 -o jsonpath="{.data.c}") | base64 -d
top-secret

$ echo -n $(kubectl get secret generic-secrets2 -o jsonpath="{.data.d}") | base64 -d
bottom-secret
```

现在我们知道如何创建不透明/通用的秘密并将它们传递给容器，让我们把所有的点连接起来，构建一个安全的 pod。

# 构建一个安全的 pod

该 pod 有一个自定义服务，不需要与 API 服务器通信（因此不需要自动挂载服务账户令牌）；相反，该 pod 提供`imagePullSecret`来拉取我们的私有仓库，并且还挂载了一些通用秘密作为文件。

让我们开始学习如何构建一个安全的 pod：

1.  第一步是自定义服务账户。以下是 YAML 清单：

```
apiVersion: v1
kind: ServiceAccount
metadata:
  name: service-account
automountServiceAccountToken: false
```

让我们创建它：

```
$ kubectl create -f service-account.yaml
serviceaccount "service-account" created
```

1.  现在，我们将把它附加到我们的 pod 上，并设置之前创建的`imagePullSecret`。这里有很多事情要做。我附加了一个自定义服务账户，创建了一个引用`generic-secrets2`秘密的秘密卷，然后挂载到`/etc/generic-secrets2`；最后，我将`imagePullSecrets`设置为`private-dockerhub`秘密：

```
apiVersion: v1
kind: Pod
metadata:
  name: trouble
spec:
  serviceAccountName: service-account
  containers:
  - name: trouble
    image: g1g1/py-kube:0.2
    command: ["/bin/bash", "-c", "while true ; do sleep 10 ; done"]
    volumeMounts:
    - name: generic-secrets2
      mountPath: "/etc/generic-secrets2"
      readOnly: true
  imagePullSecrets:
  - name: private-dockerhub
  volumes:
  - name: generic-secrets2
    secret:
      secretName: generic-secrets2
```

1.  接下来，我们可以创建我们的 pod 并开始玩耍：

```
$ kubectl create -f pod-with-secrets.yaml
pod "trouble" created
```

Kubernetes 能够从私有仓库中拉取镜像。我们不希望有 API 服务器令牌（`/var/run/secrets/kubernetes.io/serviceaccount/`不应该存在），我们的秘密应该作为文件挂载在`/etc/generic-secrets2`中。让我们通过使用`kubectl exec -it`启动一个交互式 shell 来验证这一点，并检查服务账户文件是否存在，但通用秘密`c`和`d`存在：

```
$ kubectl exec -it trouble bash

# ls /var/run/secrets/kubernetes.io/serviceaccount/
ls: cannot access '/var/run/secrets/kubernetes.io/serviceaccount/': No such file or directory

# cat /etc/generic-secrets2/c
top-secret

# cat /etc/generic-secrets2/d
bottom-secret
```

太好了，它起作用了！

在这里，我们着重于管理自定义密钥并构建一个无法访问 Kubernetes API 服务器的安全 Pod，但通常您需要仔细管理不同实体对 Kubernetes API 服务器的访问权限。Kubernetes 具有明确定义的**基于角色的访问控制模型**（也称为**RBAC**）。让我们看看它的运作方式。

# 使用 RBAC 管理权限

RBAC 是用于管理对 Kubernetes 资源的访问的机制。从 Kubernetes 1.8 开始，RBAC 被认为是稳定的。使用`--authorization-mode=RBAC`启动 API 服务器以启用它。当请求发送到 API 服务器时，RBAC 的工作原理如下：

1.  首先，它通过调用者的用户凭据或服务账户凭据对请求进行身份验证（如果失败，则返回 401 未经授权）。

1.  接下来，它检查 RBAC 策略，以验证请求者是否被授权对目标资源执行操作（如果失败，则返回 403 禁止）。

1.  最后，它通过一个准入控制器运行，该控制器可能因各种原因拒绝或修改请求。

RBAC 模型由身份（用户和服务账户）、资源（Kubernetes 对象）、动词（标准操作，如`get`、`list`和`create`）、角色和角色绑定组成。Delinkcious 服务不需要访问 API 服务器，因此它们不需要访问权限。但是，持续交付解决方案 Argo CD 绝对需要访问权限，因为它部署我们的服务和所有相关对象。

让我们来看一下角色中的以下片段，并详细了解它。您可以在这里找到源代码：[`github.com/argoproj/argo-cd/blob/master/manifests/install.yaml#L116`](https://github.com/argoproj/argo-cd/blob/master/manifests/install.yaml#L116)：

```
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  labels:
    app.kubernetes.io/component: server
    app.kubernetes.io/name: argo-cd
  name: argocd-server
rules:
- apiGroups:
  - ""
  resources:
  - secrets
  - configmaps
  verbs:
  - create
  - get
  - list
  ...
- apiGroups:
  - argoproj.io
  resources:
  - applications
  - appprojects
  verbs:
  - create
  - get
  - list
  ...
- apiGroups:
  - ""
  resources:
  - events
  verbs:
  - create
  - list
```

角色有规则。每个规则将允许的动词列表分配给每个 API 组和该 API 组内的资源。例如，对于空的 API 组（表示核心 API 组）和`configmaps`和`secrets`资源，Argo CD 服务器可以应用所有这些动词：

```
- apiGroups:
  - ""
  resources:
  - secrets
  - configmaps
  verbs:
  - create
  - get
  - list
  - watch
  - update
  - patch
  - delete
```

`argoproj.io` API 组和`applications`和`appprojects`资源（都是由 Argo CD 定义的 CRD）有另一个动词列表。最后，对于核心组的`events`资源，它只能使用`create`或`list`动词：

```
- apiGroups:
  - ""
  resources:
  - events
  verbs:
  - create
- list
```

RBAC 角色仅适用于创建它的命名空间。这意味着 Argo CD 可以对`configmaps`和`secrets`做任何事情并不太可怕，如果它是在专用命名空间中创建的。您可能还记得，我在名为`argocd`的命名空间中在集群上安装了 Argo CD。

然而，类似于角色，RBAC 还有一个`ClusterRole`，其中列出的权限在整个集群中都是允许的。Argo CD 也有集群角色。例如，`argocd-application-controller`具有以下集群角色：

```
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  labels:
    app.kubernetes.io/component: application-controller
    app.kubernetes.io/name: argo-cd
  name: argocd-application-controller
rules:
- apiGroups:
  - '*'
  resources:
  - '*'
  verbs:
  - '*'
- nonResourceURLs:
  - '*'
  verbs:
- '*'
```

这几乎可以访问集群中的任何内容。这相当于根本没有 RBAC。我不确定为什么 Argo CD 应用程序控制器需要这样的全局访问权限。我猜想这只是为了更容易地访问任何内容，而不是明确列出所有内容，如果是一个很长的列表。然而，从安全的角度来看，这并不是最佳做法。

角色和集群角色只是一系列权限列表。为了使其正常工作，您需要将角色绑定到一组帐户。这就是角色绑定和集群角色绑定发挥作用的地方。角色绑定仅在其命名空间中起作用。您可以将角色绑定到角色和集群角色（在这种情况下，集群角色仅在目标命名空间中处于活动状态）。这是一个例子：

```
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  labels:
    app.kubernetes.io/component: application-controller
    app.kubernetes.io/name: argo-cd
  name: argocd-application-controller
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: argocd-application-controller
subjects:
- kind: ServiceAccount
name: argocd-application-controller
```

集群角色绑定适用于整个集群，只能绑定集群角色（因为角色受限于其命名空间）。

现在我们了解了如何使用 RBAC 控制对 Kubernetes 资源的访问权限，让我们继续控制对我们自己的微服务的访问权限。

# 通过身份验证、授权和准入来控制访问

Kubernetes 具有一个有趣的访问控制模型，超出了标准的访问控制。对于您的微服务，它提供了身份验证、授权和准入的三重保证。您可能熟悉身份验证（谁在调用？）和授权（调用者被允许做什么？）。准入并不常见。它可以用于更动态的情况，即使调用者经过了正确的身份验证和授权，也可能被拒绝请求。

# 微服务认证

服务账户和 RBAC 是管理 Kubernetes 对象的身份和访问的良好解决方案。然而，在微服务架构中，微服务之间会有大量的通信。这种通信发生在集群内部，可能被认为不太容易受到攻击。但是，深度防御原则指导我们也要加密、认证和管理这种通信。这里有几种方法。最健壮的方法需要你自己的**私钥基础设施**（**PKI**）和**证书颁发机构**（**CA**），可以处理证书的发布、吊销和更新，因为服务实例的出现和消失。这相当复杂（如果你使用云提供商，他们可能会为你提供）。一个相对简单的方法是利用 Kubernetes secrets，并在每两个可以相互通信的服务之间创建共享的密钥。然后，当请求到来时，我们可以检查调用服务是否传递了正确的密钥，从而对其进行认证。

让我们为`link-manager`和`graph-manager`创建一个共享密钥（记住它必须是 base64 编码的）：

```
$ echo -n "social-graph-manager: 123" | base64
c29jaWFsLWdyYXBoLW1hbmFnZXI6IDEyMw==
```

然后，我们将为`link-manager`创建一个密钥，如下所示：

```
apiVersion: v1
kind: Secret
type: Opaque
metadata:
  name: mutual-auth
  namespace: default
data:
  mutual-auth.yaml: c29jaWFsLWdyYXBoLW1hbmFnZXI6IDEyMw==
```

永远不要将密钥提交到源代码控制。我在这里只是为了教育目的而这样做。

要使用`kubectl`和`jsonpath`格式查看密钥的值，您需要转义`mutual-auth.yaml`中的点：

```
$ kubectl get secret link-mutual-auth -o "jsonpath={.data['mutual-auth\.yaml']}" | base64 -D
social-graph-manager: 123
```

我们将重复这个过程为`social-graph-manager`：

```
$ echo -n "link-manager: 123" | base64
bGluay1tYW5hZ2VyOiAxMjM=
```

然后，我们将为`social-graph-manager`创建一个密钥，如下所示：

```
apiVersion: v1
kind: Secret
type: Opaque
metadata:
  name: mutual-auth
  namespace: default
data:
  mutual-auth.yaml: bGluay1tYW5hZ2VyOiAxMjM=
```

此时，`link-manager`和`social-graph-manager`有一个共享的密钥，我们可以将其挂载到各自的 pod 上。这是`link-manager`部署中的 pod 规范，它将密钥从一个卷挂载到`/etc/delinkcious`。密钥将显示为`mutual-auth.yaml`文件：

```
spec:
  containers:
  - name: link-manager
    image: g1g1/delinkcious-link:0.3
    imagePullPolicy: Always
    ports:
    - containerPort: 8080
    envFrom:
    - configMapRef:
        name: link-manager-config
    volumeMounts:
    - name: mutual-auth
      mountPath: /etc/delinkcious
      readOnly: true
  volumes:
  - name: mutual-auth
    secret:
      secretName: link-mutual-auth
```

我们可以将相同的约定应用于所有服务。结果是每个 pod 都将有一个名为`/etc/delinkcious/mutual-auth.yaml`的文件，其中包含它需要通信的所有服务的令牌。基于这个约定，我们创建了一个叫做`auth_util`的小包，它读取文件，填充了一些映射，并暴露了一些用于映射和匹配调用方和令牌的函数。`auth_util`包期望文件本身是一个 YAML 文件，格式为`<caller>: <token>`的键值对。

以下是声明和映射：

```
package auth_util

import (
   _ "github.com/lib/pq"
   "gopkg.in/yaml.v2"
   "io/ioutil"
   "os"
)

const callersFilename = "/etc/delinkcious/mutual-auth.yaml"

var callersByName = map[string]string{}
var callersByToken = map[string][]string{}
```

`init()`函数读取文件（除非`env`变量`DELINKCIOUS_MUTUAL_AUTH`设置为`false`），将其解组为`callersByName`映射，然后遍历它并填充反向`callersByToken`映射，其中令牌是键，调用者是值（可能重复）：

```
func init() {
   if os.Getenv("DELINKCIOUS_MUTUAL_AUTH") == "false" {
      return
   }

   data, err := ioutil.ReadFile(callersFilename)
   if err != nil {
      panic(err)
   }
   err = yaml.Unmarshal(data, callersByName)
   if err != nil {
      panic(err)
   }

   for caller, token := range callersByName {
      callersByToken[token] = append(callersByToken[token], caller)
   }
}
```

最后，`GetToken()`和`HasCaller()`函数提供了被服务和客户端用来相互通信的包的外部接口：

```
func GetToken(caller string) string {
   return callersByName[caller]
}

func HasCaller(caller string, token string) bool {
   for _, c := range callersByToken[token] {
      if c == caller {
         return true
      }
   }

   return false
}
```

让我们看看链接服务如何调用社交图服务的`GetFollowers()`方法。`GetFollowers()`方法从环境中提取认证令牌，并将其与标头中提供的令牌进行比较（这仅为链接服务所知），以验证调用者是否真的是链接服务。与往常一样，核心逻辑不会改变。整个身份验证方案都隔离在传输和客户端层。由于社交图服务使用 HTTP 传输，客户端将令牌存储在名为`Delinkcious-Caller-Service`的标头中。它通过`auth_util`包的`GetToken()`函数获取令牌，而不知道秘密来自何处（在我们的情况下，Kubernetes 秘密被挂载为文件）：

```
// encodeHTTPGenericRequest is a transport/http.EncodeRequestFunc that
// JSON-encodes any request to the request body. Primarily useful in a client.
func encodeHTTPGenericRequest(_ context.Context, r *http.Request, request interface{}) error {
   var buf bytes.Buffer
   if err := json.NewEncoder(&buf).Encode(request); err != nil {
      return err
   }
   r.Body = ioutil.NopCloser(&buf)

   if os.Getenv("DELINKCIOUS_MUTUAL_AUTH") != "false" {
      token := auth_util.GetToken(SERVICE_NAME)
      r.Header["Delinkcious-Caller-Token"] = []string{token}
   }

   return nil
}
```

在服务端，社交图服务传输层确保`Delinkcious-Caller-Token`存在，并且包含有效调用者的令牌：

```
func decodeGetFollowersRequest(_ context.Context, r *http.Request) (interface{}, error) {
   if os.Getenv("DELINKCIOUS_MUTUAL_AUTH") != "false" {
      token := r.Header["Delinkcious-Caller-Token"]
      if len(token) == 0 || token[0] == "" {
         return nil, errors.New("Missing caller token")
      }

      if !auth_util.HasCaller("link-manager", token[0]) {
         return nil, errors.New("Unauthorized caller")
      }
   }
   parts := strings.Split(r.URL.Path, "/")
   username := parts[len(parts)-1]
   if username == "" || username == "followers" {
      return nil, errors.New("user name must not be empty")
   }
   request := getByUsernameRequest{Username: username}
   return request, nil
}
```

这种机制的美妙之处在于，我们将解析文件和从 HTTP 请求中提取标头等繁琐的管道工作都保留在传输层，并保持核心逻辑原始。

在第十三章中，*服务网格-使用 Istio 工作*，我们将看到使用服务网格对微服务进行身份验证的另一种解决方案。现在，让我们继续授权微服务。

# 授权微服务

授权微服务可能非常简单，也可能非常复杂。在最简单的情况下，如果调用微服务经过身份验证，则被授权执行任何操作。然而，有时这是不够的，您需要根据其他请求参数进行非常复杂和细粒度的授权。例如，在我曾经工作过的一家公司，我为具有空间和时间维度的传感器网络开发了授权方案。用户可以查询数据，但他们可能仅限于特定的城市、建筑物、楼层或房间。

如果他们从未经授权的位置请求数据，他们的请求将被拒绝。他们还受到时间范围的限制，不能在指定的时间范围之外查询。

对于 Delinkcious，您可以想象用户可能只能查看自己的链接和他们关注的用户的链接（如果获得批准）。

# 承认微服务

身份验证和授权是非常著名和熟悉的访问控制机制（尽管难以强大地实施）。准入是跟随授权的另一步。即使请求经过身份验证和授权，也可能无法立即满足请求。这可能是由于服务器端的速率限制或其他间歇性问题。Kubernetes 实现了额外的功能，例如作为准入的一部分改变请求。对于您自己的微服务，可能并不需要这样做。

到目前为止，我们已经讨论了帐户、秘密和访问控制。然而，为了更接近一个安全和加固的集群，还有很多工作要做。

# 使用安全最佳实践加固您的 Kubernetes 集群

在本节中，我们将介绍各种最佳实践，并看看 Delinkcious 离正确的方式有多近。

# 保护您的镜像

最重要的之一是确保您部署到集群的镜像是安全的。这里有几个很好的指导方针要遵循。

# 始终拉取镜像

在容器规范中，有一个名为`ImagePullPolicy`的可选键。默认值是`IfNotPresent`。这个默认值有一些问题，如下所示：

+   如果您使用*latest*等标签（您不应该这样做），那么您将无法获取更新的镜像。

+   您可能会与同一节点上的其他租户发生冲突。

+   同一节点上的其他租户可以运行您的镜像。

Kubernetes 有一个名为`AlwaysPullImages`的准入控制器，它将每个 pod 的`ImagePullPolicy`设置为`AlwaysPullImages`。这可以防止所有问题，但会拉取镜像，即使它们已经存在并且您有权使用它们。您可以通过将其添加到传递给`kube-apiserver`的启用准入控制器列表中的`--enable-admission-controllers`标志来启用此准入控制器。

# 扫描漏洞

代码或依赖项中的漏洞会使攻击者能够访问你的系统。国家漏洞数据库（[`nvd.nist.gov/`](https://nvd.nist.gov/)）是了解新漏洞和管理漏洞的流程的好地方，比如**安全内容自动化协议**（**SCAP**）。

开源解决方案，如 Claire（[`github.com/coreos/clair`](https://github.com/coreos/clair)）和 Anchore（[`anchore.com/kubernetes/`](https://anchore.com/kubernetes/)）可用，还有商业解决方案。许多镜像注册表也提供扫描服务。

# 更新你的依赖项

保持依赖项的最新状态，特别是如果它们修复了已知的漏洞。在这里，你需要在警惕和保守之间找到合适的平衡。

# 固定基础镜像的版本

基础镜像的版本固定对于确保可重复构建至关重要。如果未指定基础镜像的版本，将会获取最新版本，这可能并非你想要的。

# 使用最小的基础镜像

最小化攻击面的原则敦促你尽可能使用最小的基础镜像；越小越受限制，越好。除了这些安全好处，你还可以享受更快的拉取和推送（尽管层只有在升级基础镜像时才会使其相关）。Alpine 是一个非常受欢迎的基础镜像。Delinkcious 服务采用了极端的方法，使用`SCRATCH`镜像作为基础镜像。

几乎整个服务只是一个 Go 可执行文件，就是这样。它小巧、快速、安全，但当你需要解决问题时，你会为此付出代价，因为没有工具可以帮助你。

如果我们遵循所有这些准则，我们的镜像将是安全的，但我们仍应用最小权限和零信任的基本原则，并在网络层面最小化影响范围。如果容器、Pod 或节点某种方式被 compromise，它们不应该被允许访问网络的其他部分，除非是工作负载运行所需的。这就是命名空间和网络策略发挥作用的地方。

# 划分和征服你的网络

除了身份验证作为深度防御的一部分，你还可以通过使用命名空间和网络策略来确保服务只有在必要时才能相互通信。

命名空间是一个非常直观但强大的概念。然而，它们本身并不能阻止同一集群中的 pod 相互通信。在 Kubernetes 中，集群中的所有 pod 共享相同的平面网络地址空间。这是 Kubernetes 网络模块的一个很大的简化之一。你的 pod 可以在同一节点上，也可以在不同的节点上 - 这并不重要。

每个 pod 都有自己的 IP 地址（即使多个 pod 在同一物理节点或 VM 上运行，只有一个 IP 地址）。这就是网络策略的作用。网络策略基本上是一组规则，指定了 pod 之间的集群内通信（东西流量），以及集群中服务与外部世界之间的通信（南北流量）。如果没有指定网络策略，所有传入流量（入口）默认情况下都允许访问每个 pod 的所有端口。从安全的角度来看，这是不可接受的。

让我们首先阻止所有的入口流量，然后根据需要逐渐开放：

```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all
spec:
  podSelector: {}
  policyTypes:
  - Ingress
```

请注意，网络策略是在 pod 级别工作的。您可以使用有意义的标签正确地对 pod 进行分组，这是您应该这样做的主要原因之一。

在应用此策略之前，最好知道它是否可以从故障排除的 pod 中工作，如下面的代码块所示：

```
# http GET http://$SOCIAL_GRAPH_MANAGER_SERVICE_HOST:9090/following/gigi

HTTP/1.1 200 OK
Content-Length: 37
Content-Type: text/plain; charset=utf-8
Date: Mon, 18 Feb 2019 18:00:52 GMT

{
    "err": "",
    "following": {
        "liat": true
    }
}
```

然而，在应用了`deny-all`策略之后，我们得到了一个超时错误，如下所示：

```
# http GET http://$SOCIAL_GRAPH_MANAGER_SERVICE_HOST:9090/following/gigi

http: error: Request timed out (30s).
```

现在所有的 pod 都被隔离了，让我们允许`social-graph-manager`访问它的数据库。这是一个网络策略，只允许`social-graph-manager`访问端口`5432`上的`social-graph-db`：

```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-social-graph-db
  namespace: default
spec:
  podSelector:
    matchLabels:
      svc: social-graph
      app: db
  ingress:
  - from:
    - podSelector:
        matchLabels:
          svc: social-graph
          app: manger
    ports:
    - protocol: TCP
      port: 5432
```

以下附加策略允许从`link-manager`对`social-graph-manager`的端口`9090`进行入口，如下面的代码所示：

```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-link-to-social-graph
  namespace: default
spec:
  podSelector:
    matchLabels:
      svc: social-graph
      app: manager
  ingress:
  - from:
    - podSelector:
        matchLabels:
          svc: link
          app: manger
    ports:
    - protocol: TCP
      port: 9090
```

除了安全性的好处之外，网络策略还作为系统中信息流动的实时文档。您可以准确地知道哪些服务与其他服务通信，以及外部服务。

我们已经控制了我们的网络。现在，是时候把注意力转向我们的镜像注册表了。毕竟，这是我们获取镜像的地方，我们给予了很多权限。

# 保护您的镜像注册表

强烈建议使用私有图像注册表。如果您拥有专有代码，那么您不得以公共访问方式发布您的容器，因为对您的图像进行逆向工程将授予攻击者访问权限。但是，这也有其他原因。您可以更好地控制（和审计）从注册表中拉取和推送图像。

这里有两个选择：

+   使用由 AWS、Google、Microsoft 或 Quay 等第三方管理的私有注册表。

+   使用您自己的私有注册表。

如果您在云平台上部署系统，并且该平台与其自己的图像注册表有良好的集成，或者如果您在云原生计算的精神中不管理自己的注册表，并且更喜欢让 Quay 等第三方为您完成，那么第一个选项是有意义的。

第二个选项（运行自己的容器注册表）可能是最佳选择，如果您需要对所有图像（包括基本图像和依赖项）进行额外控制。

# 根据需要授予对 Kubernetes 资源的访问权限

最小特权原则指导您仅向实际需要访问 Kubernetes 资源的服务授予权限（例如，Argo CD）。RBAC 在这里是一个很好的选择，因为默认情况下所有内容都被锁定，您可以明确添加权限。但是，要注意不要陷入给予通配符访问所有内容的陷阱，只是为了克服 RBAC 配置的困难。例如，让我们看一个具有以下规则的集群角色：

```
rules:
- apiGroups:
  - '*'
  resources:
  - '*'
  verbs:
  - '*'
- nonResourceURLs:
  - '*'
  verbs:
- '*'
```

这比禁用 RBAC 更糟糕，因为它会给您一种虚假的安全感。另一个更动态的选择是通过 Webhook 和外部服务器进行动态身份验证、授权和准入控制。这些给您提供了最大的灵活性。

# 使用配额来最小化爆炸半径

限制和配额是 Kubernetes 的机制，您可以控制分配给集群、Pod 和容器的各种有限资源，如 CPU 和内存。它们非常有用，有多种原因：

+   性能。

+   容量规划。

+   成本管理。

+   它们帮助 Kubernetes 根据资源利用率安排 Pod。

当您的工作负载在预算内运行时，一切都变得更可预测和更容易推理，尽管您必须做出努力来弄清楚实际需要多少资源，并随着时间的推移进行调整。这并不像听起来那么糟糕，因为通过水平 pod 自动缩放，您可以让 Kubernetes 动态调整服务的 pod 数量，即使每个 pod 都有非常严格的配额。

从安全的角度来看，如果攻击者获得对集群上运行的工作负载的访问权限，它将限制它可以使用的物理资源的数量。如今最常见的攻击之一就是用加密货币挖矿来饱和目标。类似的攻击类型是 fork 炸弹，它通过使一个恶意进程无法控制地复制自己来消耗所有可用资源。网络策略通过限制对网络上其他 pod 的访问来限制受损工作负载的爆炸半径。资源配额最小化了受损 pod 的主机节点上利用资源的爆炸半径。

有几种类型的配额，例如以下内容：

+   计算配额（CPU 和内存）

+   存储配额（磁盘和外部存储）

+   对象（Kubernetes 对象）

+   扩展资源（非 Kubernetes 资源，如 GPU）

资源配额非常微妙。您需要理解几个概念，例如单位和范围，以及请求和限制之间的区别。我将解释基础知识，并通过为 Delinkcious 用户服务添加资源配额来演示它们。为容器分配资源配额，因此您可以将其添加到容器规范中，如下所示：

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: user-manager
  labels:
    svc: user
    app: manager
spec:
  replicas: 1
  selector:
    matchLabels:
      svc: user
      app: manager
  template:
    metadata:
      labels:
        svc: user
        app: manager
    spec:
      containers:
      - name: user-manager
        image: g1g1/delinkcious-user:0.3
        imagePullPolicy: Always
        ports:
        - containerPort: 7070
        resources:
          requests:
            memory: 64Mi
            cpu: 250m
          limits:
            memory: 64Mi
            cpu: 250m
```

资源下有两个部分：

+   **请求**：请求是容器为了启动而请求的资源。如果 Kubernetes 无法满足特定资源的请求，它将不会启动 pod。您的工作负载可以确保在其整个生命周期中分配了这么多 CPU 和内存，并且您可以将其存入银行。

在上面的块中，我指定了`64Mi`内存和`250m` CPU 单位的请求（有关这些单位的解释，请参见下一节）。

+   **限制**：限制是工作负载可能访问的资源的上限。超出其内存限制的容器可能会被杀死，并且整个 pod 可能会从节点中被驱逐。如果被杀死，Kubernetes 将重新启动容器，并在被驱逐时重新调度 pod，就像它对任何类型的故障一样。如果容器超出其 CPU 限制，它将不会被杀死，甚至可能会在一段时间内逃脱，但是由于 CPU 更容易控制，它可能只是得不到它请求的所有 CPU，并且会经常休眠以保持在其限制范围内。

通常，最好的方法是将请求指定为限制，就像我为用户管理器所做的那样。工作负载知道它已经拥有了它将来需要的所有资源，不必担心在同一节点上有其他饥饿的邻居竞争相同资源池的情况下试图接近限制。

虽然资源是针对每个容器指定的，但是当 pod 具有多个容器时，重要的是考虑整个 pod 的总资源请求（所有容器请求的总和）。这是因为 pod 总是作为一个单元进行调度。如果您有一个具有 10 个容器的 pod，每个容器都要求 2 Gib 的内存，那么这意味着您的 pod 需要一个具有 20 Gib 空闲内存的节点。

# 请求和限制的单位

您可以使用以下后缀来请求和限制内存：E、P、T、G、M 和 K。您还可以使用 2 的幂后缀（它们总是稍微大一些），即 Ei、Pi、Ti、Gi、Mi 和 Ki。您还可以只使用整数，包括字节的指数表示法。

以下大致相同：257,988,979, 258e6, 258M 和 246Mi。CPU 单位相对于托管环境如下：

+   1 个 AWS vCPU

+   1 个 GCP Core

+   1 个 Azure vCore

+   1 个 IBM vCPU

+   在具有超线程的裸机英特尔处理器上的 1 个超线程

您可以请求 CPU 的分辨率为 0.001 的分数。更方便的方法是使用 milliCPU 和带有`m`后缀的整数；例如，100 m 是 0.1 CPU。

# 实施安全上下文

有时，Pod 和容器需要提升的特权或访问节点。这对于您的应用工作负载来说将是非常罕见的。但是，在必要时，Kubernetes 具有一个安全上下文的概念，它封装并允许您配置多个 Linux 安全概念和机制。从安全的角度来看，这是至关重要的，因为它打开了一个从容器世界到主机机器的隧道。

以下是一些安全上下文涵盖的一些机制的列表：

+   允许（或禁止）特权升级

+   通过用户 ID 和组 ID 进行访问控制（`runAsUser`，`runAsGroup`）

+   能力与无限制的根访问相对

+   使用 AppArmor 和 seccomp 配置文件

+   SELinux 配置

有许多细节和交互超出了本书的范围。我只分享一个`SecurityContext`的例子：

```
apiVersion: v1
kind: Pod
metadata:
  name: secure-pod
spec:
  containers:
  - name: some-container
    image: g1g1/py-kube:0.2
    command: ["/bin/bash", "-c", "while true ; do sleep 10 ; done"]
    securityContext:
      runAsUser: 2000
      allowPrivilegeEscalation: false
      capabilities:
        add: ["NET_ADMIN", "SYS_TIME"]
      seLinuxOptions:
        level: "s0:c123,c456"
```

安全策略执行不同的操作，比如将容器内的用户 ID 设置为`2000`，并且不允许特权升级（获取 root），如下所示：

```
$ kubectl exec -it secure-pod bash

I have no name!@secure-pod:/$ whoami
whoami: cannot find name for user ID 2000

I have no name!@secure-pod:/$ sudo su
bash: sudo: command not found
```

安全上下文是集中化 Pod 或容器的安全方面的一个很好的方式，但在一个大型集群中，您可能安装第三方软件包（如 helm charts），很难确保每个 Pod 和容器都获得正确的安全上下文。这就是 Pod 安全策略出现的地方。

# 使用安全策略加固您的 Pod

Pod 安全策略允许您设置一个适用于所有新创建的 Pod 的全局策略。它作为访问控制的准入阶段的一部分执行。Pod 安全策略可以为没有安全上下文的 Pod 创建安全上下文，或者拒绝创建和更新具有不符合策略的安全上下文的 Pod。以下是一个安全策略，它将阻止 Pod 获取允许访问主机设备的特权状态：

```
apiVersion: policy/v1beta1
kind: PodSecurityPolicy
metadata:
  name: disallow-privileged-access
spec:
  privileged: false
  allowPrivilegeEscalation: false
  # required fields.
  seLinux:
    rule: RunAsAny
  supplementalGroups:
    rule: RunAsAny
  runAsUser:
    rule: RunAsAny
  fsGroup:
    rule: RunAsAny
  volumes:
  - '*'
```

以下是一些很好的策略要执行（如果您不需要这些能力）：

+   只读根文件系统

+   控制挂载主机卷

+   防止特权访问和升级

最后，让我们确保我们将用于与 Kubernetes 集群一起工作的工具也是安全的。

# 加固您的工具链

Delinkcious 相当完善。它使用的主要工具是 Argo CD。Argo CD 可能会造成很大的损害，它在集群内运行并从 GitHub 拉取。然而，它有很多权限。在我决定将 Argo CD 作为 Delinkcious 的持续交付解决方案之前，我从安全的角度认真审查了它。Argo CD 的开发人员在考虑如何使 Argo CD 安全方面做得很好。他们做出了明智的选择，实施了这些选择，并记录了如何安全地运行 Argo CD。以下是 Argo CD 提供的安全功能：

+   通过 JWT 令牌对管理员用户进行身份验证

+   通过 RBAC 进行授权

+   通过 HTTPS 进行安全通信

+   秘密和凭证管理

+   审计

+   集群 RBAC

让我们简要地看一下它们。

# 通过 JWT 令牌对管理员用户进行身份验证

Argo CD 具有内置的管理员用户。所有其他用户必须使用**单点登录**（**SSO**）。对 Argo CD 服务器的身份验证始终使用**JSON Web Token**（**JWT**）。管理员用户的凭据也会转换为 JWT。

它还支持通过`/api/v1/projects/{project}/roles/{role}/token`端点进行自动化，生成由 Argo CD 本身签发的自动化令牌。这些令牌的范围有限，并且过期得很快。

# 通过 RBAC 进行授权

Argo CD 通过将用户的 JWT 组声明映射到 RBAC 角色来授权请求。这是行业标准认证与 Kubernetes 授权模型 RBAC 的非常好的结合。

# 通过 HTTPS 进行安全通信

Argo CD 的所有通信，以及其自身组件之间的通信，都是通过 HTTPS/TLS 完成的。

# 秘密和凭证管理

Argo CD 需要管理许多敏感信息，例如：

+   Kubernetes 秘密

+   Git 凭证

+   OAuth2 客户端凭证

+   对外部集群的凭证（当未安装在集群中时）

Argo CD 确保将所有这些秘密保留给自己。它永远不会通过在响应中返回它们或记录它们来泄露它们。所有 API 响应和日志都经过清理和编辑。

# 审计

您可以通过查看 git 提交日志来审计大部分活动，这会触发 Argo CD 中的所有内容。但是，Argo CD 还发送各种事件以捕获集群内的活动，以提供额外的可见性。这种组合很强大。

# 集群 RBAC

默认情况下，Argo CD 使用集群范围的管理员角色。这并不是必要的。建议将其写权限限制在需要管理的命名空间中。

# 总结

在本章中，我们认真看待了一个严肃的话题：安全。基于微服务的架构和 Kubernetes 对于支持关键任务目标并经常管理敏感信息的大规模企业分布式系统是最有意义的。除了开发和演进这样复杂系统的挑战之外，我们必须意识到这样的系统对攻击者非常诱人。

我们必须使用严格的流程和最佳实践来保护系统、用户和数据。从这里开始，我们涵盖了安全原则和最佳实践，我们也看到它们如何相互支持，以及 Kubernetes 如何致力于允许它们安全地开发和操作我们的系统。

我们还讨论了作为 Kubernetes 微服务安全基础的支柱：认证/授权/准入的三重 A，集群内外的安全通信，强大的密钥管理（静态和传输加密），以及分层安全策略。

在这一点上，您应该清楚地了解了您可以使用的安全机制，并且有足够的信息来决定如何将它们整合到您的系统中。安全永远不会完成，但利用最佳实践将使您能够在每个时间点上找到安全和系统其他要求之间的正确平衡。

在下一章中，我们将最终向世界开放 Delinkcious！我们将研究公共 API、负载均衡器以及我们需要注意的性能和安全性重要考虑因素。

# 进一步阅读

有许多关于 Kubernetes 安全性的良好资源。我收集了一些非常好的外部资源，这些资源将帮助您在您的旅程中：

+   Kubernetes 安全性：[`kubernetes-security.info/`](https://kubernetes-security.info/)

+   微软 SDL 实践：[`www.microsoft.com/en-us/securityengineering/sdl/practices`](https://www.microsoft.com/en-us/securityengineering/sdl/practices)

以下 Kubernetes 文档页面扩展了本章涵盖的许多主题：

+   网络策略：[`kubernetes.io/docs/concepts/services-networking/network-policies/`](https://kubernetes.io/docs/concepts/services-networking/network-policies/)

+   资源配额：[`kubernetes.io/docs/concepts/policy/resource-quotas/`](https://kubernetes.io/docs/concepts/policy/resource-quotas/)

+   为 Pod 或容器配置安全上下文：[`kubernetes.io/docs/tasks/configure-pod-container/security-context/`](https://kubernetes.io/docs/tasks/configure-pod-container/security-context/)
