# 与世界交流- API 和负载均衡器

在本章中，我们最终将向外部打开 Delinkcious，让用户可以从集群外部与其进行交互。这很重要，因为 Delinkcious 用户无法访问集群内部运行的内部服务。我们将通过添加基于 Python 的 API 网关服务并将其暴露给世界（包括社交登录）来显著扩展 Delinkcious 的功能。我们将添加一个基于 gRPC 的新闻服务，用户可以使用它来获取关注的其他用户的新闻。最后，我们将添加一个消息队列，让服务以松散耦合的方式进行通信。

在本章中，我们将涵盖以下主题：

+   熟悉 Kubernetes 服务

+   东西向与南北向通信

+   理解入口和负载均衡

+   提供和使用公共 REST API

+   提供和使用内部 gRPC API

+   通过消息队列发送和接收事件

+   为服务网格做准备

# 技术要求

在本章中，我们将向 Delinkcious 添加一个 Python 服务。无需安装任何新内容。我们稍后将为 Python 服务构建一个 Docker 镜像。

# 代码

您可以在这里找到更新的 Delinkcious 应用程序：[`github.com/the-gigi/delinkcious/releases/tag/v0.5`](https://github.com/the-gigi/delinkcious/releases/tag/v0.5)

# 熟悉 Kubernetes 服务

Pod（一个或多个容器捆绑在一起）是 Kubernetes 中的工作单位。部署确保有足够的 Pod 在运行。但是，单个 Pod 是短暂的。Kubernetes 服务是行动所在的地方，以及您如何将您的 Pod 公开为一个连贯的服务，供集群中的其他服务甚至外部世界使用。Kubernetes 服务提供稳定的标识，并且通常将应用程序服务（可以是微服务或传统的大型服务）进行 1：1 映射。让我们看看所有的服务：

```
$ kubectl get svc
NAME                TYPE      CLUSTER-IP    EXTERNAL-IP   PORT(S)   AGE
api-gateway      LoadBalancer   10.103.167.102  <pending> 80:31965/TCP  6m2s
kubernetes         ClusterIP    10.96.0.1         <none>    443/TCP      25m
link-db            ClusterIP    10.107.131.61     <none>    5432/TCP     8m53s 
link-manager       ClusterIP    10.109.32.254     <none>    8080/TCP     8m53s
news-manager       ClusterIP    10.99.206.183     <none>    6060/TCP     7m45s
news-manager-redis ClusterIP     None             <none>    6379/TCP     7m45s
social-graph-db    ClusterIP    10.106.164.24     <none>    5432/TCP     8m38s
social-graph-manager ClusterIP   10.100.107.79    <none>    9090/TCP     8m37s
user-db             ClusterIP    None             <none>    5432/TCP     8m10s
user-manager        ClusterIP    10.108.45.93     <none>    7070/TCP     8m10s
```

您已经看到了 Delinkcious 微服务是如何使用 Kubernetes 服务部署的，以及它们如何通过 Kubernetes 提供的环境变量进行发现和调用。Kubernetes 还提供基于 DNS 的发现。

每个服务都可以通过 DNS 名称在集群内部访问：

```
<service name>.<namespace>.svc.cluster.local
```

我更喜欢使用环境变量，因为这样可以让我在 Kubernetes 之外运行服务进行测试。

以下是如何使用环境变量和 DNS 查找`social-graph-manager`服务的 IP 地址：

```
$ dig +short social-graph-manager.default.svc.cluster.local
10.107.162.99

$ env | grep SOCIAL_GRAPH_MANAGER_SERVICE_HOST
SOCIAL_GRAPH_MANAGER_SERVICE_HOST=10.107.162.99
```

Kubernetes 通过指定标签选择器将服务与其支持的 pod 关联起来。例如，如下所示的代码，`news-service`由具有`svc: link`和`app: manager`标签的 pod 支持：

```
spec:
  replicas: 1
  selector:
    matchLabels:
      svc: link
      app: manager
```

然后，Kubernetes 使用`endpoints`资源管理与标签选择器匹配的所有 pod 的 IP 地址，如下所示：

```
$ kubectl get endpoints
NAME                   ENDPOINTS                                            AGE
api-gateway            172.17.0.15:5000                                     1d
kubernetes             192.168.99.137:8443                                  51d
link-db                172.17.0.19:5432                                     40d
.
.
.
social-graph-db        172.17.0.16:5432                                     50d
social-graph-manager   172.17.0.18:9090                                     43d
```

`endpoints`资源始终保持支持服务的所有 pod 的 IP 地址和端口的最新列表。当添加、删除或重新创建具有另一个 IP 地址和端口的 pod 时，将更新`endpoints`资源。现在，让我们看看 Kubernetes 中有哪些类型的服务。

# Kubernetes 中的服务类型

Kubernetes 服务始终具有类型。了解何时使用每种类型的服务非常重要。让我们来看看各种服务类型及其之间的区别：

+   ClusterIP（默认）：ClusterIP 类型意味着服务只能在集群内部访问。这是默认设置，非常适合微服务之间的通信。为了测试目的，您可以使用`kube-proxy`或`port-forwarding`来暴露这样的服务。这也是查看 Kubernetes 仪表板或内部服务的其他 UI（例如 Delinkcious 中的 Argo CD）的好方法。

如果不指定 ClusterIP 的类型，请将`ClusterIP`设置为`None`。

+   NodePort：NodePort 类型的服务通过所有节点上的专用端口向世界公开。您可以通过`<Node IP>:<NodePort>`访问服务。如果您自己运行 Kubernetes API 服务器，则可以通过`--service-node-port-range`控制范围来选择 NodePort（默认情况下为 30000-32767）。

您还可以在服务定义中明确指定 NodePort。如果您通过指定的节点端口暴露了大量服务，则必须小心管理这些端口，以避免冲突。当请求通过专用 NodePort 进入任何节点时，kubelet 将负责将其转发到具有其中一个支持 pod 的节点（您可以通过 endpoints 找到它）。

+   **LoadBalancer**：当您的 Kubernetes 集群在提供负载均衡器支持的云平台上运行时，这种类型的服务最常见。尽管在本地集群中也有适用于 Kubernetes 的负载均衡器，但外部负载均衡器将负责接受外部请求并将其通过服务路由到后端 Pod。通常存在云提供商特定的复杂性，例如特殊注释或必须创建双重服务来处理内部和外部请求。我们将使用 LoadBalancer 类型来将 Delinkcious 暴露给 minikube 的世界，该世界提供了负载均衡器仿真。

+   **ExternalName**：这些服务只是将请求解析到外部提供的 DNS 名称。如果您的服务需要与集群外部未运行的外部服务通信，但仍希望能够像它们是 Kubernetes 服务一样找到它们，这将非常有用。如果您计划将这些外部服务迁移到集群中，这可能会很有用。

现在我们了解了服务的全部内容，让我们讨论一下集群内部的跨服务通信和将服务暴露到集群外部之间的区别。

# 东西通信与南北通信

东西通信是指服务/Pod/容器在集群内部相互通信。正如您可能还记得的那样，Kubernetes 通过 DNS 和环境变量公开了集群内的所有服务。这解决了集群内部的服务发现问题。您可以通过网络策略或其他机制来进一步施加限制。例如，在第五章中，*使用 Kubernetes 配置微服务*，我们在链接服务和社交图服务之间建立了相互认证。

南北通信是指向世界暴露服务。理论上，您可以仅通过 NodePort 暴露您的服务，但这种方法存在许多问题，包括以下问题：

+   您必须自行处理安全/加密传输

+   无法控制哪些 Pod 实际上会为请求提供服务

+   您必须让 Kubernetes 为您的服务选择随机端口，或者仔细管理端口冲突。

+   每个端口只能暴露一个服务（例如，令人垂涎的端口`80`不能被重用）

批准生产的暴露服务的方法是通过入口控制器和/或负载均衡器使用。

# 理解入口和负载均衡

Kubernetes 中的入口概念是关于控制对您的服务的访问，并可能提供其他功能，例如以下内容：

+   SSL 终止

+   身份验证

+   路由到多个服务

有一个入口资源，定义其他相关信息的路由规则，还有一个入口控制器，它读取集群中定义的所有入口资源（跨所有命名空间）。入口资源接收所有请求并路由到分发它们到后台 pod 的目标服务。入口控制器充当集群范围的软件负载均衡器和路由器。通常，会有一个硬件负载均衡器坐在集群前面，并将所有流量发送到入口控制器。

让我们把所有这些概念放在一起，通过添加一个公共 API 网关来向世界展示 Delinkcious。

# 提供和使用公共 REST API

在这一部分，我们将构建一个全新的 Python 服务（API 网关），以证明 Kubernetes 实际上是与语言无关的。然后，我们将通过 OAuth2 添加用户身份验证，并将 API 网关服务暴露给外部。

# 构建基于 Python 的 API 网关服务

API 网关服务旨在接收来自集群外部的所有请求，并将它们路由到适当的服务。以下是目录结构：

```
$ tree
 .
 ├── Dockerfile
 ├── README.md
 ├── api_gateway_service
 │   ├── __init__.py
 │   ├── api.py
 │   ├── config.py
 │   ├── news_client.py
 │   ├── news_client_test.py
 │   ├── news_pb2.py
 │   ├── news_pb2_grpc.py
 │   └── resources.py
 ├── k8s
 │   ├── api_gateway.yaml
 │   ├── configmap.yaml
 │   └── secrets.yaml
 ├── requirements.txt
 ├── run.py
 └── tests
 └── api_gateway_service_test.py
```

这与 Go 服务有些不同。代码位于`api_gateway_service`目录下，这也是一个 Python 包。Kubernetes 资源位于`k8s`子目录下，还有一个`tests`子目录。在顶级目录中，`run.py`文件是入口点，如`Dockerfile`中定义的那样。`run.py`中的`main()`函数调用了从`api.py`模块导入的`app.run()`方法：

```
import os
from api_gateway_service.api import app

def main():
    port = int(os.environ.get('PORT', 5000))
    login_url = 'http://localhost:{}/login'.format(port)
    print('If you run locally, browse to', login_url)
    host = '0.0.0.0'
    app.run(host=host, port=port)

if __name__ == "__main__":
    main()
```

`api.py`模块负责创建应用程序，连接路由，并实现社交登录。

# 实现社交登录

`api-gateway`服务利用了几个 Python 包来帮助通过 GitHub 实现社交登录。稍后，我们将介绍用户流程，但首先，我们将看一下实现它的代码。`login()`方法正在与 GitHub 联系，并请求对当前用户进行授权，该用户必须已登录 GitHub 并授权给 Delinkcious。

`logout()`方法只是从当前会话中删除访问令牌。`authorized()`方法在 GitHub 成功登录尝试后被调用，并提供一个访问令牌，该令牌将在用户的浏览器中显示。这个访问令牌必须作为标头传递给 API 网关的所有未来请求：

```
@app.route('/login')
def login():
    callback = url_for('authorized', _external=True)
    result = app.github.authorize(callback)
    return result

@app.route('/login/authorized')
def authorized():
    resp = app.github.authorized_response()
    if resp is None:
        # return 'Access denied: reason=%s error=%s' % (
        #     request.args['error'],
        #     request.args['error_description']
        # )
        abort(401, message='Access denied!')
    token = resp['access_token']
    # Must be in a list or tuple because github auth code extracts the first
    user = app.github.get('user', token=(token,))
    user.data['access_token'] = token
    return jsonify(user.data)

@app.route('/logout')
def logout():
    session.pop('github_token', None)
    return 'OK'
```

当用户传递有效的访问令牌时，Delinkcious 可以从 GitHub 检索他们的姓名和电子邮件。如果访问令牌丢失或无效，请求将被拒绝，并显示 401 访问被拒绝错误。这发生在`resources.py`中的`_get_user()`函数中：

```
def _get_user():
    """Get the user object or create it based on the token in the session

    If there is no access token abort with 401 message
    """
    if 'Access-Token' not in request.headers:
        abort(401, message='Access Denied!')

    token = request.headers['Access-Token']
    user_data = github.get('user', token=dict(access_token=token)).data
    if 'email' not in user_data:
        abort(401, message='Access Denied!')

    email = user_data['email']
    name = user_data['name']

    return name, email
```

GitHub 对象是在`api.py`模块的`create_app()`函数中创建和初始化的。首先，它导入了一些第三方库，即`Flask`、`OAuth`和`Api`类：

```
import os

from flask import Flask, url_for, session, jsonify
from flask_oauthlib.client import OAuth
from flask_restful import Api, abort
from . import resources
from .resources import Link
```

然后，它使用 GitHub `Oauth`提供程序初始化`Flask`应用程序：

```
def create_app():
    app = Flask(__name__)
    app.config.from_object('api_gateway_service.config')
    oauth = OAuth(app)
    github = oauth.remote_app(
        'github',
        consumer_key=os.environ['GITHUB_CLIENT_ID'],
        consumer_secret=os.environ['GITHUB_CLIENT_SECRET'],
        request_token_params={'scope': 'user:email'},
        base_url='https://api.github.com/',
        request_token_url=None,
        access_token_method='POST',
        access_token_url='https://github.com/login/oauth/access_token',
        authorize_url='https://github.com/login/oauth/authorize')
    github._tokengetter = lambda: session.get('github_token')
    resources.github = app.github = github
```

最后，它设置路由映射并存储初始化的`app`对象：

```
api = Api(app)
    resource_map = (
        (Link, '/v1.0/links'),
    )

    for resource, route in resource_map:
        api.add_resource(resource, route)

    return app

app = create_app()
```

# 将流量路由到内部微服务

API 网关服务的主要工作是实现我们在第二章中讨论的 API 网关模式，*开始使用微服务*。例如，它是如何将获取链接请求路由到链接微服务的适当方法的。

`Link`类是从`Resource`基类派生的。它从环境中获取主机和端口，并构造基本 URL。

当 GET 请求`links`端点时，将调用`get()`方法。它从`_get_user()`函数中的 GitHub 令牌中提取用户名，并解析请求 URL 的查询部分以获取其他参数。然后，它会向链接管理器服务发出自己的请求：

```
class Link(Resource):
    host = os.environ.get('LINK_MANAGER_SERVICE_HOST', 'localhost')
    port = os.environ.get('LINK_MANAGER_SERVICE_PORT', '8080')
    base_url = 'http://{}:{}/links'.format(host, port)

    def get(self):
        """Get all links

        If user doesn't exist create it (with no goals)
        """
        username, email = _get_user()
        parser = RequestParser()
        parser.add_argument('url_regex', type=str, required=False)
        parser.add_argument('title_regex', type=str, required=False)
        parser.add_argument('description_regex', type=str, required=False)
        parser.add_argument('tag', type=str, required=False)
        parser.add_argument('start_token', type=str, required=False)
        args = parser.parse_args()
        args.update(username=username)
        r = requests.get(self.base_url, params=args)

        if not r.ok:
            abort(r.status_code, message=r.content)

        return r.json()
```

# 利用基础 Docker 镜像来减少构建时间

当我们为 Delinkcious 构建 Go 微服务时，我们使用了 scratch 镜像作为基础，只是复制了 Go 二进制文件。这些镜像非常轻量，不到 10MB。然而，即使使用`python:alpine`，API 网关也几乎有 500MB，这比标准的基于 Debian 的 Python 镜像要轻得多：

```
$ docker images | grep g1g1.*0.3
g1g1/delinkcious-user              0.3    07bcc08b1d73   38 hours ago    6.09MB
g1g1/delinkcious-social-graph      0.3    0be0e9e55689   38 hours ago    6.37MB
g1g1/delinkcious-news              0.3    0ccd600f2190   38 hours ago    8.94MB
g1g1/delinkcious-link              0.3    9fcd7aaf9a98   38 hours ago    6.95MB
g1g1/delinkcious-api-gateway       0.3    d5778d95219d   38 hours ago    493MB
```

此外，API 网关需要构建一些与本地库的绑定。安装 C/C++工具链，然后构建本地库需要很长时间（超过 15 分钟）。Docker 在这里表现出色，具有可重用的层和基础镜像。我们可以将所有繁重的东西放入一个单独的基础镜像中，位于`svc/shared/docker/python_flask_grpc/Dockerfile`：

```
FROM python:alpine
RUN apk add build-base
COPY requirements.txt /tmp
WORKDIR /tmp
RUN pip install -r requirements.txt
```

`requirements.txt`文件包含执行社交登录并需要使用 gRPC 服务的`Flask`应用程序的依赖项（稍后详细介绍）：

```
requests-oauthlib==1.1.0
Flask-OAuthlib==0.9.5
Flask-RESTful==0.3.7
grpcio==1.18.0
grpcio-tools==1.18.0
```

将所有这些放在一起，我们可以构建基础镜像，然后 API 网关 Dockerfile 可以基于它。以下是在`svc/shared/docker/python_flask_grpc/build.sh`中的超级简单构建脚本，用于构建基础镜像并将其推送到 DockerHub：

```
IMAGE=g1g1/delinkcious-python-flask-grpc:0.1
docker build . -t $IMAGE
docker push $IMAGE
```

让我们看一下`svc/api_gateway_service/Dockerfile`中 API 网关服务的 Dockerfile。它基于我们的基础镜像。然后，它复制`api_gate_service`目录，公开`5000`端口，并执行`run.py`脚本：

```
FROM g1g1/delinkcious-python-flask-grpc:0.1
MAINTAINER Gigi Sayfan "the.gigi@gmail.com"
COPY . /api_gateway_service
WORKDIR /api_gateway_service
EXPOSE 5000
ENTRYPOINT python run.py
```

好处是只要重型基础镜像不改变，对实际 API 服务网关代码进行更改将导致闪电般快速的 Docker 镜像构建。我们说的是几秒钟，而不是 15 分钟。在这一点上，我们对 API 网关服务有了一个不错而快速的构建-测试-调试-部署。现在是向集群添加入口的好时机。

# 添加入口

在 Minikube 上，您必须启用入口附加组件：

```
$ minikube addons enable ingress 
 ingress was successfully enabled
```

在其他 Kubernetes 集群上，您可能希望安装自己喜欢的入口控制器（例如 Contour、Traefik 或 Ambassador）。

以下代码是 API 网关服务的入口清单。通过使用这种模式，我们的整个集群将有一个单一的入口，将每个请求引导到我们的 API 网关服务，然后将其路由到适当的内部服务：

```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: api-gateway
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: delinkcio.us
    http:
      paths:
      - path: /*
        backend:
          serviceName: api-gateway
          servicePort: 80
```

单个入口服务简单而有效。在大多数云平台上，您按入口资源付费，因为为每个入口资源创建了一个负载均衡器。您可以轻松扩展 API 网关实例的数量，因为它完全无状态。

Minikube 在网络下面做了很多魔术，模拟负载均衡器，并隧道流量。我不建议使用 Minikube 来测试对集群的入口。相反，我们将使用 LoadBalancer 类型的服务，并通过 Minikube 集群 IP 访问它。

# 验证 API 网关在集群外部是否可用

Delinkcious 使用 GitHub 作为社交登录提供程序。您必须拥有 GitHub 帐户才能跟随。

用户流程如下：

1.  查找 Delinkcious URL（在 Minikube 上，这将经常更改）。

1.  登录并获取访问令牌。

1.  从集群外部访问 Delinkcious API 网关。

让我们深入详细讨论一下。

# 查找 Delinkcious URL

在生产集群中，您将配置一个众所周知的 DNS 名称，并连接一个负载均衡器到该名称。使用 Minikube，我们可以使用以下命令获取 API 网关服务的 URL：

```
$ minikube service api-gateway --url
http://192.168.99.138:31658
```

为了与命令进行交互使用，将其存储在环境变量中是方便的，如下所示：

```
$ export DELINKCIOUS_URL=$(minikube service api-gateway --url)
```

# 获取访问令牌

获取访问令牌的步骤如下：

1.  现在我们有了 API 网关 URL，我们可以浏览到登录端点，即 `http://192.168.99.138:31658/login`。如果您已登录到您的 GitHub 帐户，您将看到以下对话框：

![](img/806ac3a0-d918-4aef-9c87-de521ba70753.png)

1.  接下来，如果这是您第一次登录 Delinkcious，GitHub 将要求您授权 Delinkcious 获取访问您的电子邮件和姓名：

![](img/975e7472-5cc3-4b1c-ad44-32550e819dae.png)

1.  如果您同意，那么您将被重定向到一个页面，该页面将向您显示有关您的 GitHub 个人资料的大量信息，但更重要的是，向您提供一个访问令牌，如下截图所示：

![](img/d2ca79b6-c9ee-4133-b2cc-621775611cb3.png)

让我们也将访问令牌存储在环境变量中：

```
$ export DELINKCIOUS_TOKEN=def7de18d9c05ce139e37140871a9d16fd37ea9d
```

现在我们已经获得了从外部访问 Delinkcious 所需的所有信息，让我们来试一试。

# 从集群外部访问 Delinkcious API 网关

我们将使用 HTTPie 命中 `${DELINKCIOUS_URL}/v1.0/links` 的 API 网关端点。要进行身份验证，我们必须将访问令牌作为标头提供，即 `"Access-Token: ${DELINKCIOUS_TOKEN}"`。

从零开始，让我们验证一下是否没有任何链接：

```
$ http "${DELINKCIOUS_URL}/v1.0/links" "Access-Token: ${DELINKCIOUS_TOKEN}"
HTTP/1.0 200 OK
Content-Length: 27
Content-Type: application/json
Date: Mon, 04 Mar 2019 00:52:18 GMT
Server: Werkzeug/0.14.1 Python/3.7.2

{
    "err": "",
    "links": null
}
```

好了，到目前为止一切都很顺利。让我们通过向 `/v1.0/links` 端点发送 POST 请求来添加一些链接。这是第一个链接：

```
$ http POST "${DELINKCIOUS_URL}/v1.0/links" "Access-Token: ${DELINKCIOUS_TOKEN}" url=http://gg.com title=example
HTTP/1.0 200 OK
Content-Length: 12
Content-Type: application/json
Date: Mon, 04 Mar 2019 00:52:49 GMT
Server: Werkzeug/0.14.1 Python/3.7.2

{
    "err": ""
}
```

这是第二个链接：

```
$ http POST "${DELINKCIOUS_URL}/v1.0/links" "Access-Token: ${DELINKCIOUS_TOKEN}" url=http://gg2.com title=example
HTTP/1.0 200 OK
Content-Length: 12
Content-Type: application/json
Date: Mon, 04 Mar 2019 00:52:49 GMT
Server: Werkzeug/0.14.1 Python/3.7.2

{
    "err": ""
}
```

没有错误。太好了。通过再次获取链接，我们可以看到我们刚刚添加的新链接：

```
$ http "${DELINKCIOUS_URL}/v1.0/links" "Access-Token: ${DELINKCIOUS_TOKEN}"
HTTP/1.0 200 OK
Content-Length: 330
Content-Type: application/json
Date: Mon, 04 Mar 2019 00:52:52 GMT
Server: Werkzeug/0.14.1 Python/3.7.2

{
    "err": "",
    "links": [
        {
            "CreatedAt": "2019-03-04T00:52:35Z",
            "Description": "",
            "Tags": null,
            "Title": "example",
            "UpdatedAt": "2019-03-04T00:52:35Z",
            "Url": "http://gg.com"
        },
        {
            "CreatedAt": "2019-03-04T00:52:48Z",
            "Description": "",
            "Tags": null,
            "Title": "example",
            "UpdatedAt": "2019-03-04T00:52:48Z",
            "Url": "http://gg2.com"
        }
    ]
}
```

我们已成功建立了端到端的流程，包括用户身份验证，因此通过其内部 HTTP REST API 与 Go 微服务通信的 Python API 网关服务，并将信息存储在关系型数据库中。现在，让我们提高赌注并添加另一个服务。

这次，将使用 gRPC 传输的 Go 微服务。

# 提供和使用内部 gRPC API

我们将在本节中实现的服务称为新闻服务。它的工作是跟踪链接事件，如添加链接或更新链接，并向用户返回新事件。

# 定义 NewsManager 接口

此接口公开了一个`GetNews()`方法。用户可以调用它并从他们关注的用户那里收到链接事件列表。以下是 Go 接口和相关结构。它并不复杂：一个带有`username`和`token`字段的请求结构体，以及一个结果结构体。结果结构体包含一个`Event`结构体列表，其中包含以下信息：`EventType`、`Username`、`Url`和`Timestamp`：

```
type NewsManager interface {
        GetNews(request GetNewsRequest) (GetNewsResult, error)
}

type GetNewsRequest struct {
        Username   string
        StartToken string
}

type Event struct {
        EventType EventTypeEnum
        Username  string
        Url       string
        Timestamp time.Time
}

type GetNewsResult struct {
        Events    []*Event
        NextToken string
}
```

# 实现新闻管理器包

核心逻辑服务的实现在`pkg/news_manager`中。让我们看一下`new_manager.go`文件。`NewsManager`结构有一个名为`eventStore`的`InMemoryNewsStore`，它实现了`NewsManager`接口的`GetNews()`方法。它将实际获取新闻的工作委托给存储。

但是，它知道分页并负责将令牌从字符串转换为整数以匹配存储偏好：

```
package news_manager

import (
        "errors"
        "github.com/the-gigi/delinkcious/pkg/link_manager_events"
        om "github.com/the-gigi/delinkcious/pkg/object_model"
        "strconv"
        "time"
)

type NewsManager struct {
        eventStore *InMemoryNewsStore
}

func (m *NewsManager) GetNews(req om.GetNewsRequest) (resp om.GetNewsResult, err error) {
        if req.Username == "" {
                err = errors.New("user name can't be empty")
                return
        }

        startIndex := 0
        if req.StartToken != "" {
                startIndex, err := strconv.Atoi(req.StartToken)
                if err != nil || startIndex < 0 {
                        err = errors.New("invalid start token: " + req.StartToken)
                        return resp, err
                }
        }

        events, nextIndex, err := m.eventStore.GetNews(req.Username, startIndex)
        if err != nil {
                return
        }

        resp.Events = events
        if nextIndex != -1 {
                resp.NextToken = strconv.Itoa(nextIndex)
        }

        return
}
```

存储非常基础，只是在用户名和所有事件之间保持映射，如下所示：

```
package news_manager

import (
        "errors"
        om "github.com/the-gigi/delinkcious/pkg/object_model"
)

const maxPageSize = 10

// User events are a map of username:userEvents
type userEvents map[string][]*om.Event

// InMemoryNewsStore manages a UserEvents data structure
type InMemoryNewsStore struct {
        userEvents userEvents
}

func NewInMemoryNewsStore() *InMemoryNewsStore {
        return &InMemoryNewsStore{userEvents{}}
}
```

存储实现了自己的`GetNews()`方法（与`interface`方法的签名不同）。它只是根据起始索引和最大页面大小返回目标用户请求的切片：

```
func (m *InMemoryNewsStore) GetNews(username string, startIndex int) (events []*om.Event, nextIndex int, err error) {
        userEvents := m.userEvents[username]
        if startIndex > len(userEvents) {
                err = errors.New("Index out of bounds")
                return
        }

        pageSize := len(userEvents) - startIndex
        if pageSize > maxPageSize {
                pageSize = maxPageSize
                nextIndex = startIndex + maxPageSize
        } else {
                nextIndex = -1
        }

        events = userEvents[startIndex : startIndex+pageSize]
        return
}
```

它还有一种添加新事件的方法：

```
func (m *InMemoryNewsStore) AddEvent(username string, event *om.Event) (err error) {
        if username == "" {
                err = errors.New("user name can't be empty")
                return
        }

        if event == nil {
                err = errors.New("event can't be nil")
                return
        }

        if m.userEvents[username] == nil {
                m.userEvents[username] = []*om.Event{}
        }

        m.userEvents[username] = append(m.userEvents[username], event)
        return
}
```

现在我们已经实现了存储和向用户提供新闻的核心逻辑，让我们看看如何将这个功能公开为 gRPC 服务。

# 将 NewsManager 公开为 gRPC 服务

在深入了解新闻服务的 gRPC 实现之前，让我们看看到底是怎么回事。gRPC 是一组用于连接服务和应用程序的传输协议、有效载荷格式、概念框架和代码生成工具。它起源于 Google（因此在 gRPC 中有 g），是一个高性能且成熟的 RPC 框架。它有很多优点，比如以下：

+   跨平台

+   行业广泛采用

+   所有相关编程语言的惯用客户端库

+   极其高效的传输协议

+   Google 协议缓冲区用于强类型合同

+   HTTP/2 支持实现双向流

+   高度可扩展（自定义您自己的身份验证、授权、负载均衡和健康检查）

+   出色的文档

总之，对于内部微服务而言，它在几乎所有方面都优于基于 HTTP 的 REST API。

对于 Delinkcious 来说，这非常合适，因为我们选择的微服务框架 Go-kit 对 gRPC 有很好的支持。

# 定义 gRPC 服务契约

gRPC 要求您使用受协议缓冲区启发的特殊 DSL 为您的服务定义契约。它非常直观，并且让 gRPC 为您生成大量样板代码。我选择将契约和生成的代码放在一个名为**pb**（协议缓冲区的常用简称）的单独顶级目录中，因为生成的代码的不同部分将被服务和消费者使用。在这些情况下，最好将共享代码放在一个单独的位置，而不是随意地将其放入服务或客户端。

这是`pb/new-service/pb/news.proto`文件：

```
syntax = "proto3";
package pb;

import "google/protobuf/timestamp.proto";

service News {
    rpc GetNews(GetNewsRequest) returns (GetNewsResponse) {}
}

message GetNewsRequest {
    string username = 1;
    string startToken = 2;
}

enum EventType {
    LINK_ADDED = 0;
    LINK_UPDATED = 1;
    LINK_DELETED = 2;
}

message Event  {
        EventType eventType = 1;
        string username = 2;
        string url = 3;
        google.protobuf.Timestamp timestamp = 4;
}

message GetNewsResponse {
        repeated Event events = 1;
        string nextToken = 2;
    string err = 3;
}
```

我们不需要逐行讨论每一行的语法和含义。简而言之，请求和响应始终是消息。服务级错误需要嵌入在响应消息中。其他错误，如网络或无效的有效载荷，将被单独报告。一个有趣的细节是，除了原始数据类型和嵌入的消息之外，您还可以使用其他高级类型，例如`google.protobuf.Timestamp`数据类型。这显著提高了抽象级别，并为诸如日期和时间戳之类的事物带来了强类型化的好处，这些事物在使用 JSON 进行 HTTP/REST 工作时，您总是需要自己进行序列化和反序列化。

服务定义很酷，但我们需要一些实际的代码来连接这些点。让我们看看 gRPC 如何帮助完成这个任务。

# 使用 gRPC 生成服务存根和客户端库

gRPC 模型用于使用一个名为`protoc`的工具生成服务存根和客户端库。我们需要为新闻服务本身生成 Go 代码，以及为消费它的 API 网关生成 Python 代码。

您可以通过运行以下命令生成`news.pb.go`：

```
protoc --go_out=plugins=grpc:. news.proto
```

您可以通过运行以下命令生成`news_pb2.py`和`news_pb2_grpc.py`：

```
python -m grpc_tools.protoc -I. --python_out=. --grpc_python_out=. news.proto
```

此时，Go 客户端代码和 Python 客户端代码都可以用于从 Go 代码或 Python 代码调用新闻服务。

# 使用 Go-kit 构建 NewsManager 服务

这是在`news_service.go`中服务本身的实现。它看起来非常类似于 HTTP 服务。让我们分解一下重要的部分。首先，它导入一些库，包括在`pb/news-service-pb`、`pkg/news_manager`和一个名为`google.golang.org/grpc`的一般 gRPC 库中生成的 gRPC 代码。在`Run()`函数的开头，它从环境中获取`service`端口来监听：

```
package service

import (
        "fmt"
        "github.com/the-gigi/delinkcious/pb/news_service/pb"
        nm "github.com/the-gigi/delinkcious/pkg/news_manager"
        "google.golang.org/grpc"
        "log"
        "net"
        "os"
)

func Run() {
        port := os.Getenv("PORT")
        if port == "" {
                port = "6060"
        }
```

现在，我们需要在目标端口上创建一个标准的 TCP 监听器：

```
listener, err := net.Listen("tcp", ":"+port)
        if err != nil {
                log.Fatal(err)
        }
```

此外，我们必须连接到一个 NATS 消息队列服务。我们将在下一节中详细讨论这个问题：

```
natsHostname := os.Getenv("NATS_CLUSTER_SERVICE_HOST")
        natsPort := os.Getenv("NATS_CLUSTER_SERVICE_PORT")
```

这里是主要的初始化代码。它实例化一个新的新闻管理器，创建一个新的 gRPC 服务器，创建一个新闻管理器对象，并将新闻管理器注册到 gRPC 服务器。`pb.RegisterNewsManager()`方法是由 gRPC 从`news.proto`文件生成的：

```
svc, err := nm.NewNewsManager(natsHostname, natsPort)
        if err != nil {
                log.Fatal(err)
        }

        gRPCServer := grpc.NewServer()
        newsServer := newNewsServer(svc)
        pb.RegisterNewsServer(gRPCServer, newsServer)
```

最后，gRPC 服务器开始在 TCP 监听器上监听：

```
fmt.Printf("News service is listening on port %s...\n", port)
        err = gRPCServer.Serve(listener)
        fmt.Println("Serve() failed", err)
}
```

# 实现 gRPC 传输

拼图的最后一部分是在`transport.go`文件中实现 gRPC 传输。在概念上，它类似于 HTTP 传输，但有一些不同的细节。让我们分解一下，以便清楚地了解所有部分是如何组合在一起的。

首先，导入所有相关的包，包括来自 go-kit 的 gRPC 传输。请注意，在`news_service.go`中，没有任何地方提到 go-kit。您肯定可以直接在 Go 中使用一般的 gRPC 库实现 gRPC 服务。然而，在这里，通过其服务和端点的概念，go-kit 将帮助使这变得更容易：

```
package service

import (
        "context"
        "github.com/go-kit/kit/endpoint"
        grpctransport "github.com/go-kit/kit/transport/grpc"
        "github.com/golang/protobuf/ptypes/timestamp"
        "github.com/the-gigi/delinkcious/pb/news_service/pb"
        om "github.com/the-gigi/delinkcious/pkg/object_model"
)
```

`newEvent()`函数是一个辅助函数，它从我们的抽象对象模型中采用`om.Event`到 gRPC 生成的事件对象。最重要的部分是翻译事件类型和时间戳：

```
func newEvent(e *om.Event) (event *pb.Event) {
        event = &pb.Event{
                EventType: (pb.EventType)(e.EventType),
                Username:  e.Username,
                Url:       e.Url,
        }

        seconds := e.Timestamp.Unix()
        nanos := (int32(e.Timestamp.UnixNano() - 1e9*seconds))
        event.Timestamp = &timestamp.Timestamp{Seconds: seconds, Nanos: nanos}
        return
}
```

解码请求和编码响应非常简单 - 没有必要序列化或反序列化任何 JSON 代码：

```
func decodeGetNewsRequest(_ context.Context, r interface{}) (interface{}, error) {
        request := r.(*pb.GetNewsRequest)
        return om.GetNewsRequest{
                Username:   request.Username,
                StartToken: request.StartToken,
        }, nil
}

func encodeGetNewsResponse(_ context.Context, r interface{}) (interface{}, error) {
        return r, nil
}
```

创建端点类似于您在其他服务中看到的 HTTP 传输。它调用实际的服务实现，然后翻译响应并处理错误（如果有的话）：

```
func makeGetNewsEndpoint(svc om.NewsManager) endpoint.Endpoint {
        return func(_ context.Context, request interface{}) (interface{}, error) {
                req := request.(om.GetNewsRequest)
                r, err := svc.GetNews(req)
                res := &pb.GetNewsResponse{
                        Events:    []*pb.Event{},
                        NextToken: r.NextToken,
                }
                if err != nil {
                        res.Err = err.Error()
                }
                for _, e := range r.Events {
                        event := newEvent(e)
                        res.Events = append(res.Events, event)
                }
                return res, nil
        }
}
```

处理程序实现了从生成的代码中的 gRPC 新闻接口：

```
type handler struct {
        getNews grpctransport.Handler
}

func (s *handler) GetNews(ctx context.Context, r *pb.GetNewsRequest) (*pb.GetNewsResponse, error) {
        _, resp, err := s.getNews.ServeGRPC(ctx, r)
        if err != nil {
                return nil, err
        }

        return resp.(*pb.GetNewsResponse), nil
}
```

`newNewsServer()`函数将所有内容联系在一起。它返回一个包装在 Go-kit 处理程序中的 gRPC 处理程序，连接端点、请求解码器和响应编码器：

```
func newNewsServer(svc om.NewsManager) pb.NewsServer {
        return &handler{
                getNews: grpctransport.NewServer(
                        makeGetNewsEndpoint(svc),
                        decodeGetNewsRequest,
                        encodeGetNewsResponse,
                ),
        }
}
```

这可能看起来非常混乱，有着各种层和嵌套函数，但底线是你只需要编写很少的粘合代码（并且可以生成它，这是理想的），最终得到一个非常干净、安全（强类型）和高效的 gRPC 服务。

现在我们有了一个可以提供新闻的 gRPC 新闻服务，让我们看看如何为其提供新闻。

# 通过消息队列发送和接收事件

新闻服务需要为每个用户存储链接事件。链接服务知道不同用户何时添加、更新或删除链接。解决这个问题的一种方法是向新闻服务添加另一个 API，并让链接服务调用此 API，并通知新闻服务每个相关事件。然而，这种方法会在链接服务和新闻服务之间创建紧密耦合。链接服务并不真正关心新闻服务，因为它不需要任何来自新闻服务的东西。相反，让我们选择一种松散耦合的解决方案。链接服务只会向一个通用消息队列服务发送事件。然后，独立地，新闻服务将订阅从该消息队列接收消息。这种方法有几个好处，如下所示：

+   不需要更复杂的服务代码

+   与事件通知的交互模型完美契合

+   很容易在不改变代码的情况下添加额外的监听器到相同的事件

我在这里使用的术语，即*消息*、*事件*和*通知*，是可以互换的。这个想法是，源有一些信息以一种即时即忘的方式与世界分享。

它不需要知道谁对信息感兴趣（可能是没有人或多个监听器），以及是否成功处理。Delinkcious 使用 NATS 消息系统进行服务之间的松散耦合通信。

# NATS 是什么？

NATS（[`nats.io/`](https://nats.io/)）是一个开源消息队列服务。它是一个**Cloud Native Computing Foundation**（**CNCF**）项目，用 Go 实现，被认为是在 Kubernetes 中需要消息队列时的顶级竞争者之一。NATS 支持多种消息传递模型，如下所示：

+   发布-订阅

+   请求-回复

+   排队

NATS 非常灵活，可以用于许多用例。它也可以在高可用的集群中运行。对于 Delinkcious，我们将使用发布-订阅模型。以下图表说明了发布-订阅消息传递模型。发布者发布一条消息，所有订阅者都会收到相同的消息：

![](img/f623f1c6-9276-40f2-b18b-989601014789.png)

让我们在我们的集群中部署 NATS。

# 在集群中部署 NATS

首先，让我们安装 NATS 操作员（[`github.com/nats-io/nats-operator`](https://github.com/nats-io/nats-operator)）。NATS 操作员可以帮助您在 Kubernetes 中管理 NATS 集群。以下是安装它的命令：

```
$ kubectl apply -f https://github.com/nats-io/nats-operator/releases/download/v0.4.5/00-prereqs.yaml
$ kubectl apply -f https://github.com/nats-io/nats-operator/releases/download/v0.4.5/10-deployment.yaml
```

NATS 操作员提供了一个 NatsCluster **自定义资源定义**（**CRD**），我们将使用它在我们的 Kubernetes 集群中部署 NATS。不要被 Kubernetes 集群内的 NATS 集群关系所困扰。这真的很好，因为我们可以像内置的 Kubernetes 资源一样部署 NATS 集群。以下是在`svc/shared/k8s/nats_cluster.yaml`中可用的 YAML 清单：

```
apiVersion: nats.io/v1alpha2
kind: NatsCluster
metadata:
  name: nats-cluster
spec:
  size: 1
  version: "1.3.0"
```

让我们使用`kubectl`部署它，并验证它是否被正确部署：

```
$ kubectl apply -f nats_cluster.yaml
natscluster.nats.io "nats-cluster" configured

$ kubectl get svc -l app=nats
NAME                TYPE      CLUSTER-IP   EXTERNAL-IP   PORT(S)    AGE
nats-cluster       ClusterIP  10.102.48.27  <none>       4222/TCP    5d
nats-cluster-mgmt  ClusterIP   None         <none>        6222/TCP,8222/TCP,7777/TCP   5d
```

看起来不错。监听端口`4222`的`nats-cluster`服务是 NATS 服务器。另一个服务是管理服务。让我们向 NATS 服务器发送一些事件。

# 使用 NATS 发送链接事件

正如你可能记得的，我们在我们的对象模型中定义了一个`LinkManagerEvents`接口：

```
type LinkManagerEvents interface {
        OnLinkAdded(username string, link *Link)
        OnLinkUpdated(username string, link *Link)
        OnLinkDeleted(username string, url string)
}
```

`LinkManager`包在其`NewLinkManager()`方法中接收此事件链接：

```
func NewLinkManager(linkStore LinkStore,
        socialGraphManager om.SocialGraphManager,
        eventSink om.LinkManagerEvents,
        maxLinksPerUser int64) (om.LinkManager, error) {
        if linkStore == nil {
                return nil, errors.New("link store")
        }

        if eventSink != nil && socialGraphManager == nil {
                msg := "social graph manager can't be nil if event sink is not nil"
                return nil, errors.New(msg)
        }

        return &LinkManager{
                linkStore:          linkStore,
                socialGraphManager: socialGraphManager,
                eventSink:          eventSink,
                maxLinksPerUser:    maxLinksPerUser,
        }, nil
}
```

稍后，当链接被添加、更新或删除时，`LinkManager`将调用相应的`OnLinkXXX()`方法。例如，当调用`AddLink()`时，对于每个关注者，都会在接收器上调用`OnLinkAdded()`方法：

```
if m.eventSink != nil {
                followers, err := m.socialGraphManager.GetFollowers(request.Username)
                if err != nil {
                        return err
                }

                for follower := range followers {
                        m.eventSink.OnLinkAdded(follower, link)
                }
        }
```

这很棒，但这些事件将如何传送到 NATS 服务器？这就是链接服务发挥作用的地方。在实例化`LinkManager`对象时，它将传递一个专用的事件发送对象作为实现`LinkManagerEvents`的接收器。每当它接收到诸如`OnLinkAdded()`或`OnLinkUpdated()`之类的事件时，它会将事件发布到`link-events`主题的 NATS 服务器上。它暂时忽略`OnLinkDeleted()`事件。这个对象位于`pkg/link_manager_events package/sender.go`中：

```
package link_manager_events

import (
        "github.com/nats-io/go-nats"
        "log"

        om "github.com/the-gigi/delinkcious/pkg/object_model"
)

type eventSender struct {
        hostname string
        nats     *nats.EncodedConn
}
```

这里是`OnLinkAdded()`、`OnLinkUpdated()`和`OnLinkDeleted()`方法的实现：

```
func (s *eventSender) OnLinkAdded(username string, link *om.Link) {
        err := s.nats.Publish(subject, Event{om.LinkAdded, username, link})
        if err != nil {
                log.Fatal(err)
        }
}

func (s *eventSender) OnLinkUpdated(username string, link *om.Link) {
        err := s.nats.Publish(subject, Event{om.LinkUpdated, username, link})
        if err != nil {
                log.Fatal(err)
        }
}

func (s *eventSender) OnLinkDeleted(username string, url string) {
        // Ignore link delete events
}
```

`NewEventSender()`工厂函数接受 NATS 服务的 URL，将事件发送到 NATS 服务，并返回一个`LinkManagerEvents`接口，可以作为`LinkManager`的接收端：

```
func NewEventSender(url string) (om.LinkManagerEvents, error) {
        ec, err := connect(url)
        if err != nil {
                return nil, err
        }
        return &eventSender{hostname: url, nats: ec}, nil
}
```

现在，链接服务所需做的就是找出 NATS 服务器的 URL。由于 NATS 服务器作为 Kubernetes 服务运行，其主机名和端口可以通过环境变量获得，就像 Delinkcious 微服务一样。以下是链接服务的`Run()`函数中的相关代码：

```
natsHostname := os.Getenv("NATS_CLUSTER_SERVICE_HOST")
        natsPort := os.Getenv("NATS_CLUSTER_SERVICE_PORT")

        var eventSink om.LinkManagerEvents
        if natsHostname != "" {
                natsUrl := natsHostname + ":" + natsPort
                eventSink, err = nats.NewEventSender(natsUrl)
                if err != nil {
                        log.Fatal(err)
                }
        } else {
                eventSink = &EventSink{}
        }

        svc, err := lm.NewLinkManager(store, socialGraphClient, eventSink, maxLinksPerUser)
        if err != nil {
                log.Fatal(err)
        }
```

此时，每当为用户添加或更新新链接时，`LinkManager`将为每个关注者调用`OnLinkAdded()`或`OnLinkUpdated()`方法，这将导致该事件被发送到`link-events`主题的 NATS 服务器上，所有订阅者都将收到并处理它。下一步是新闻服务订阅这些事件。

# 使用 NATS 订阅链接事件

新闻服务使用`pkg/link_manager_events/listener.go`中的`Listen()`函数。它接受 NATS 服务器的 URL 和实现`LinkManagerEvents`接口的事件接收端。它连接到 NATS 服务器，然后订阅`link-events`主题。这与事件发送器发送这些事件的主题相同：

```
package link_manager_events

import (
        om "github.com/the-gigi/delinkcious/pkg/object_model"
)

func Listen(url string, sink om.LinkManagerEvents) (err error) {
        conn, err := connect(url)
        if err != nil {
                return
        }

        conn.Subscribe(subject, func(e *Event) {
                switch e.EventType {
                case om.LinkAdded:
                        {
                                sink.OnLinkAdded(e.Username, e.Link)
                        }
                case om.LinkUpdated:
                        {
                                sink.OnLinkAdded(e.Username, e.Link)
                        }
                default:
                        // Ignore other event types
                }
        })

        return
}
```

现在，让我们看一下定义`link-events`主题的`nats.go`文件，以及`connect()`函数，该函数被事件发送器和`Listen()`函数使用。连接函数使用`go-nats`客户端建立连接，然后用 JSON 编码器包装它，这使它能够自动序列化发送和接收 Go 结构。这很不错：

```
package link_manager_events

import "github.com/nats-io/go-nats"

const subject = "link-events"

func connect(url string) (encodedConn *nats.EncodedConn, err error) {
        conn, err := nats.Connect(url)
        if err != nil {
                return
        }

        encodedConn, err = nats.NewEncodedConn(conn, nats.JSON_ENCODER)
        return
}
```

新闻服务在其`NewNewsManager()`工厂函数中调用`Listen()`函数。首先，它实例化实现`LinkManagerEvents`的新闻管理器对象。然后，如果提供了 NATS 主机名，则组合 NATS 服务器 URL 并调用`Listen()`函数，从而将新闻管理器对象作为接收端传递：

```
func NewNewsManager(natsHostname string, natsPort string) (om.NewsManager, error) {
        nm := &NewsManager{eventStore: NewInMemoryNewsStore()}
        if natsHostname != "" {
                natsUrl := natsHostname + ":" + natsPort
                err := link_manager_events.Listen(natsUrl, nm)
                if err != nil {
                        return nil, err
                }
        }

        return nm, nil
}
```

下一步是对传入的事件进行处理。

# 处理链接事件

新闻管理器通过`NewNewsManager()`函数订阅链接事件，结果是这些事件将作为对`OnLinkAdded()`和`OnlinkUpdated()`的调用到达（删除链接事件被忽略）。新闻管理器创建了一个在抽象对象模型中定义的`Event`对象，用`EventType`、`Username`、`Url`和`Timestamp`填充它，然后调用事件存储的`AddEvent()`函数。这是`OnLinkAdded()`方法：

```
func (m *NewsManager) OnLinkAdded(username string, link *om.Link) {
        event := &om.Event{
                EventType: om.LinkAdded,
                Username:  username,
                Url:       link.Url,
                Timestamp: time.Now().UTC(),
        }
        m.eventStore.AddEvent(username, event)
}
```

这是`OnLinkUpdated()`方法：

```
func (m *NewsManager) OnLinkUpdated(username string, link *om.Link) {
        event := &om.Event{
                EventType: om.LinkUpdated,
                Username:  username,
                Url:       link.Url,
                Timestamp: time.Now().UTC(),
        }
        m.eventStore.AddEvent(username, event)
}
```

让我们看看存储在其`AddEvent()`方法中做了什么。这很简单：订阅用户位于`userEvents`映射中。如果他们还不存在，那么将创建一个空条目并添加新事件。如果目标用户调用`GetNews()`，他们将收到为他们收集的事件：

```
func (m *InMemoryNewsStore) AddEvent(username string, event *om.Event) (err error) {
        if username == "" {
                err = errors.New("user name can't be empty")
                return
        }
        if event == nil {
                err = errors.New("event can't be nil")
                return
        }
        if m.userEvents[username] == nil {
                m.userEvents[username] = []*om.Event{}
        }
        m.userEvents[username] = append(m.userEvents[username], event)
        return
}
```

这就结束了我们对新闻服务及其通过 NATS 服务与链接管理器的交互的覆盖。这是我们在第二章中讨论的**命令查询责任分离**（**CQRS**）模式的应用，*使用微服务入门*。现在 Delinkcious 系统看起来是这样的：

![](img/47cae881-3b57-49f8-84b6-7fd50313a172.png)

现在我们了解了 Delinkcious 中如何处理事件，让我们快速看一下服务网格。

# 理解服务网格

服务网格是在您的集群中运行的另一层管理。我们将在第十三章中详细了解服务网格和特别是 Istio，*服务网格-使用 Istio*。在这一点上，我只想提一下，服务网格经常也承担入口控制器的角色。

使用服务网格进行入口的主要原因之一是，内置的入口资源非常通用，受到多个问题的限制，例如以下问题：

+   没有很好的方法来验证规则

+   入口资源可能会相互冲突

+   使用特定的入口控制器通常很复杂，并且需要自定义注释。

# 总结

在本章中，我们完成了许多任务并连接了所有要点。特别是，我们实现了两种微服务设计模式（API 网关和 CQRS），添加了一个用 Python 实现的全新服务（包括一个分割的 Docker 基础镜像），添加了一个 gRPC 服务，向我们的集群添加了一个开源消息队列系统（NATS）并将其与发布-订阅消息传递集成，最后，我们向世界打开了我们的集群，并通过向 Delinkcious 添加和获取链接来演示端到端的交互。

在这一点上，Delinkcious 可以被视为 Alpha 级软件。它是功能性的，但离生产就绪还差得远。在下一章中，我们将通过处理任何软件系统的最有价值的商品 - 数据，使 Delinkcious 更加健壮。Kubernetes 提供了许多管理数据和有状态服务的设施，我们将充分利用它们。

# 进一步阅读

您可以参考以下来源，了解本章涵盖的更多信息：

+   **Kubernetes 服务**：[`kubernetes.io/docs/concepts/services-networking/service/`](https://kubernetes.io/docs/concepts/services-networking/service/)

+   将您的应用程序公开为服务：[`kubernetes.io/docs/tutorials/kubernetes-basics/expose/expose-intro/`](https://kubernetes.io/docs/tutorials/kubernetes-basics/expose/expose-intro/)

+   构建 Oauth 应用程序：[`developer.github.com/apps/building-oauth-apps/`](https://developer.github.com/apps/building-oauth-apps/)

+   **高性能 gRPC**：[`grpc.io/`](https://grpc.io/)

[`www.devx.com/architect/high-performance-services-with-grpc.html`](http://www.devx.com/architect/high-performance-services-with-grpc.html)

+   **NATS 消息代理**：[`nats.io/`](https://nats.io/)
