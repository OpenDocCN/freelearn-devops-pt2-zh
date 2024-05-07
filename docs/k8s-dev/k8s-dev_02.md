# 第二章：将您的代码打包以在 Kubernetes 中运行

在本章中，我们将深入探讨使用 Kubernetes 所需的第一件事：将软件放入容器中。我们将回顾容器是什么，如何存储和共享镜像，以及如何构建容器。然后，本章继续进行两个示例，一个是 Python，另一个是 Node.js，它们将引导您如何将这些语言的简单示例代码构建成容器，并在 Kubernetes 中运行它们。本章的部分内容包括：

+   容器镜像

+   制作自己的容器

+   Python 示例-制作容器镜像

+   Node.js 示例-制作容器镜像

+   给您的容器镜像打标签

# 容器镜像

使用 Kubernetes 的第一步是将您的软件放入容器中。Docker 是创建这些容器的最简单方法，而且这是一个相当简单的过程。让我们花点时间来查看现有的容器镜像，以了解在创建自己的容器时需要做出什么选择：

```
docker pull docker.io/jocatalin/kubernetes-bootcamp:v1
```

首先，您将看到它下载具有奥秘 ID 的文件列表。您会看到它们并行更新，因为它尝试在可用时抓取它们：

```
v1: Pulling from jocatalin/kubernetes-bootcamp
5c90d4a2d1a8: Downloading  3.145MB/51.35MB
ab30c63719b1: Downloading  3.931MB/18.55MB
29d0bc1e8c52: Download complete
d4fe0dc68927: Downloading  2.896MB/13.67MB
dfa9e924f957: Waiting
```

当下载完成时，输出将更新为“提取”，最后为“拉取完成”：

```
v1: Pulling from jocatalin/kubernetes-bootcamp
5c90d4a2d1a8: Pull complete
ab30c63719b1: Pull complete
29d0bc1e8c52: Pull complete
d4fe0dc68927: Pull complete
dfa9e924f957: Pull complete
Digest: sha256:0d6b8ee63bb57c5f5b6156f446b3bc3b3c143d233037f3a2f00e279c8fcc64af
Status: Downloaded newer image for jocatalin/kubernetes-bootcamp:v1
```

您在终端中看到的是 Docker 正在下载构成容器镜像的层，将它们全部汇集在一起，然后验证输出。当您要求 Kubernetes 运行软件时，Kubernetes 正是执行这个相同的过程，下载镜像然后运行它们。

如果您现在运行以下命令：

```
docker images
```

您将看到（也许还有其他）列出的镜像类似于这样：

```
REPOSITORY                                         TAG                 IMAGE ID            CREATED             SIZE
jocatalin/kubernetes-bootcamp                      v1                  8fafd8af70e9        13 months ago       211MB
```

该镜像的大小为`211MB`，当我们指定`jocatalin/kubernetes-bootcamp:v1`时，您会注意到我们同时指定了一个名称`jocatalin/kubernetes-bootcamp`和一个标签`v1`。此外，该镜像具有一个`IMAGE ID`（`8fafd8af70e9`），这是整个镜像的唯一 ID。如果您要为镜像指定一个名称而没有标签，那么默认情况下会假定您想要一个默认标签`latest`。

让我们深入了解刚刚下载的镜像，使用`docker history`命令：

```
docker history jocatalin/kubernetes-bootcamp:v1
```

```
IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
8fafd8af70e9        13 months ago       /bin/sh -c #(nop)  CMD ["/bin/sh" "-c" "no...   0B
<missing>           13 months ago       /bin/sh -c #(nop) COPY file:de8ef36ebbfd53...   742B
<missing>           13 months ago       /bin/sh -c #(nop)  EXPOSE 8080/tcp              0B
<missing>           13 months ago       /bin/sh -c #(nop) CMD ["node"]                  0B
<missing>           13 months ago       /bin/sh -c buildDeps='xz-utils'     && set...   41.5MB
<missing>           13 months ago       /bin/sh -c #(nop) ENV NODE_VERSION=6.3.1        0B
<missing>           15 months ago       /bin/sh -c #(nop) ENV NPM_CONFIG_LOGLEVEL=...   0B
<missing>           15 months ago       /bin/sh -c set -ex   && for key in     955...   80.8kB
<missing>           15 months ago       /bin/sh -c apt-get update && apt-get insta...   44.7MB
<missing>           15 months ago       /bin/sh -c #(nop) CMD ["/bin/bash"]             0B
<missing>           15 months ago       /bin/sh -c #(nop) ADD file:76679eeb94129df...   125MB
```

这明确了我们之前在下载容器时看到的情况：容器镜像由层组成，每一层都建立在它下面的层之上。Docker 镜像的层非常简单——每一层都是执行命令和命令在本地文件系统上所做的任何更改的结果。在之前的`docker history`命令中，您将看到任何更改底层文件系统大小的命令报告的大小。

镜像格式是由 Docker 创建的，现在由**OCI**（**Open Container Initiative**）Image Format 项目正式指定。如果您想进一步了解，可以在[`github.com/opencontainers/image-spec`](https://github.com/opencontainers/image-spec)找到格式和所有相关细节。

容器镜像以及镜像中的每个层通常都可以在互联网上找到。我在本书中使用的所有示例都是公开可用的。可以配置 Kubernetes 集群使用私有镜像仓库，Kubernetes 项目的文档中有关于如何执行该任务的详细说明，网址为[`kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry/`](https://kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry/)。这种设置更加私密，但设置起来更加复杂，因此在本书中，我们将继续使用公开可用的镜像。

容器镜像还包括如何运行镜像、要运行什么、要设置哪些环境变量等信息。我们可以使用`docker inspect`命令查看所有这些细节：

```
docker inspect jocatalin/kubernetes-bootcamp:v1
```

上述命令生成了相当多的内容，详细描述了容器镜像以及其中运行代码所需的元数据：

```
[
    {
        "Id": "sha256:8fafd8af70e9aa7c3ab40222ca4fd58050cf3e49cb14a4e7c0f460cd4f78e9fe",
        "RepoTags": [
            "jocatalin/kubernetes-bootcamp:v1"
        ],
        "RepoDigests": [
            "jocatalin/kubernetes-bootcamp@sha256:0d6b8ee63bb57c5f5b6156f446b3bc3b3c143d233037f3a2f00e279c8fcc64af"
        ],
        "Parent": "",
        "Comment": "",
        "Created": "2016-08-04T16:46:35.471076443Z",
        "Container": "976a20903b4e8b3d1546e610b3cba8751a5123d76b8f0646f255fe2baf345a41",
        "ContainerConfig": {
            "Hostname": "6250540837a8",
            "Domainname": "",
            "User": "",
            "AttachStdin": false,
            "AttachStdout": false,
            "AttachStderr": false,
            "ExposedPorts": {
                "8080/tcp": {}
            },
            "Tty": false,
            "OpenStdin": false,
            "StdinOnce": false,
            "Env": [
                "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin",
                "NPM_CONFIG_LOGLEVEL=info",
                "NODE_VERSION=6.3.1"
            ],
            "Cmd": [
                "/bin/sh",
                "-c",
                "#(nop) ",
                "CMD [\"/bin/sh\" \"-c\" \"node server.js\"]"
            ],
            "ArgsEscaped": true,
            "Image": "sha256:87ef05c0e8dc9f729b9ff7d5fa6ad43450bdbb72d95c257a6746a1f6ad7922aa",
            "Volumes": null,
            "WorkingDir": "",
            "Entrypoint": null,
            "OnBuild": [],
            "Labels": {}
        },
        "DockerVersion": "1.12.0",
        "Author": "",
        "Architecture": "amd64",
        "Os": "linux",
        "Size": 211336459,
        "VirtualSize": 211336459,

```

除了基本配置之外，Docker 容器镜像还可以包含运行时配置，因此通常会有一个重复的部分，在`ContainerConfig`键下定义了大部分你所说的内容：

```
        "Config": {
            "Hostname": "6250540837a8",
            "Domainname": "",
            "User": "",
            "AttachStdin": false,
            "AttachStdout": false,
            "AttachStderr": false,
            "ExposedPorts": {
                "8080/tcp": {}
            },
            "Tty": false,
            "OpenStdin": false,
            "StdinOnce": false,
            "Env": [
                "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin",
                "NPM_CONFIG_LOGLEVEL=info",
                "NODE_VERSION=6.3.1"
            ],
            "Cmd": [
                "/bin/sh",
                "-c",
                "node server.js"
            ],
            "ArgsEscaped": true,
            "Image": "sha256:87ef05c0e8dc9f729b9ff7d5fa6ad43450bdbb72d95c257a6746a1f6ad7922aa",
            "Volumes": null,
            "WorkingDir": "",
            "Entrypoint": null,
            "OnBuild": [],
            "Labels": {}
        },
```

最后一个部分包括了文件系统的叠加的显式列表以及它们如何组合在一起：

```
"GraphDriver": {
            "Data": {
                "LowerDir": "/var/lib/docker/overlay2/b38e59d31a16f7417c5ec785432ba15b3743df647daed0dc800d8e9c0a55e611/diff:/var/lib/docker/overlay2/792ce98aab6337d38a3ec7d567324f829e73b1b5573bb79349497a9c14f52ce2/diff:/var/lib/docker/overlay2/6c131c8dd754628a0ad2c2aa7de80e58fa6b3f8021f34af684b78538284cf06a/diff:/var/lib/docker/overlay2/160efe1bd137edb08fe180f020054933134395fde3518449ab405af9b1fb6cb0/diff",
                "MergedDir": "/var/lib/docker/overlay2/40746dcac4fe98d9982ce4c0a0f6f0634e43c3b67a4bed07bb97068485cd137a/merged",
                "UpperDir": "/var/lib/docker/overlay2/40746dcac4fe98d9982ce4c0a0f6f0634e43c3b67a4bed07bb97068485cd137a/diff",
                "WorkDir": "/var/lib/docker/overlay2/40746dcac4fe98d9982ce4c0a0f6f0634e43c3b67a4bed07bb97068485cd137a/work"
            },
            "Name": "overlay2"
        },
        "RootFS": {
            "Type": "layers",
            "Layers": [
                "sha256:42755cf4ee95900a105b4e33452e787026ecdefffcc1992f961aa286dc3f7f95",
                "sha256:d1c800db26c75f0aa5881a5965bd6b6abf5101dbb626a4be5cb977cc8464de3b",
                "sha256:4b0bab9ff599d9feb433b045b84aa6b72a43792588b4c23a2e8a5492d7940e9a",
                "sha256:aaed480d540dcb28252b03e40b477e27564423ba0fe66acbd04b2affd43f2889",
                "sha256:4664b95364a615be736bd110406414ec6861f801760dae2149d219ea8209a4d6"
            ]
        }
    }
]
```

JSON 转储中包含了很多信息，可能比你现在需要或关心的要多。最重要的是，我想让你知道它在`config`部分下指定了一个`cmd`，分为三个部分。这是如果你`run`容器时默认会被调用的内容，通常被称为`Entrypoint`。如果你把这些部分组合起来，想象自己在容器中运行它们，你将会运行以下内容：

```
/bin/sh -c node server.js
```

`Entrypoint`定义了将要执行的二进制文件，以及任何参数，并且是指定你想要运行什么以及如何运行的关键。Kubernetes 与这个相同的`Entrypoint`一起工作，并且可以用命令和参数覆盖它，以运行你的软件，或者运行你在同一个容器镜像中存储的诊断工具。

# 容器注册表

在前面的例子中，当我们调用命令来拉取容器时，我们引用了[`www.docker.com/`](https://www.docker.com/)，这是 Docker 的容器注册表。在使用 Kubernetes 或阅读有关 Kubernetes 的文档时，你经常会看到另外两个常见的注册表：[gcr.io](https://cloud.google.com/container-registry/)，谷歌的容器注册表，以及[quay.io](https://quay.io/)，CoreOS 的容器注册表。其他公司在互联网上提供托管的容器注册表，你也可以自己运行。目前，Docker 和 Quay 都为公共镜像提供免费托管，因此你会经常在文档和示例中看到它们。这三个注册表还提供私有镜像仓库的选项，通常需要相对较小的订阅费用。

公开可用的镜像（以及在这些镜像上进行层叠）的一个好处是，它使得非常容易组合你的镜像，共享底层层。这也意味着这些层可以被检查，并且可以搜索常见的层以查找安全漏洞。有几个旨在帮助提供这些信息的开源项目，还有几家公司成立了帮助协调信息和扫描的公司。如果你为你的镜像订阅了一个镜像仓库，它们通常会在其产品中包括这种漏洞扫描。

作为开发人员，当您在代码中使用库时，您对其操作负有责任。您已经负责熟悉这些库的工作方式（或者不熟悉），并在它们不按预期工作时处理任何问题。通过指定整个容器的灵活性和控制权，您同样负责以相同的方式包含在容器中的所有内容。

很容易忘记软件构建所依赖的层，并且您可能没有时间跟踪所有可能出现的安全漏洞和问题，这些问题可能已经出现在您正在构建的软件中。来自 Clair 等项目的安全扫描（[`github.com/coreos/clair`](https://github.com/coreos/clair)）可以为您提供有关潜在漏洞的出色信息。我建议您考虑利用可以为您提供这些详细信息的服务。

# 创建您的第一个容器

使用 Docker 软件和`docker build`命令很容易创建一个容器。这个命令使用一个详细说明如何创建容器的清单，称为 Dockerfile。

让我们从最简单的容器开始。创建一个名为 Dockerfile 的文件，并将以下内容添加到其中：

```
FROM alpine
CMD ["/bin/sh", "-c", "echo 'hello world'"]
```

然后，调用`build`：

```
docker build .
```

如果您看到这样的响应：

```
"docker build" requires exactly 1 argument.
See 'docker build --help'.
Usage: docker build [OPTIONS] PATH | URL | -
Build an image from a Dockerfile
```

然后，您要么在命令中缺少`.`，要么在与创建 Dockerfile 的目录不同的目录中运行了该命令。`.`告诉`docker`在哪里找到 Dockerfile（`.`表示在当前目录中）。

您应该看到类似以下的输出：

```
Sending build context to Docker daemon  2.048kB
Step 1/2 : FROM alpine
latest: Pulling from library/alpine
88286f41530e: Pull complete
Digest: sha256:f006ecbb824d87947d0b51ab8488634bf69fe4094959d935c0c103f4820a417d
Status: Downloaded newer image for alpine:latest
 ---> 76da55c8019d
Step 2/2 : CMD /bin/sh -c echo 'hello world'
 ---> Running in 89c04e8c5d87
 ---> f5d273aa2dcb
Removing intermediate container 89c04e8c5d87
Successfully built f5d273aa2dcb
```

这个图像只有一个 ID，`f5d273aa2dcb`，没有名称，但这对我们来说已经足够了解它是如何工作的。如果您在本地运行此示例，您将获得一个唯一标识容器图像的不同 ID。您可以使用`docker run f5d273aa2dcb`命令在容器图像中运行代码。这应该会导致您看到以下输出：

```
hello world
```

花点时间在刚刚创建的图像上运行`docker history f5d273aa2dcb`和`docker inspect f5d273aa2dcb`。

完成后，我们可以使用以下命令删除刚刚创建的 Docker 图像：

```
docker rmi **f5d273aa2dcb** 
```

如果您在删除图像时遇到错误，这可能是因为您有一个引用本地图像的已停止容器，您可以通过添加`-f`来强制删除。例如，强制删除本地图像的命令将是：

```
docker rmi -f f5d237aa2dcb
```

# Dockerfile 命令

Docker 有关于如何编写 Dockerfile 的文档，网址是[`docs.docker.com/engine/reference/builder/`](https://docs.docker.com/engine/reference/builder/)，以及他们推荐的一套最佳实践，网址是[`docs.docker.com/engine/userguide/eng-image/dockerfile_best-practices/`](https://docs.docker.com/engine/userguide/eng-image/dockerfile_best-practices/)。我们将介绍一些常见且重要的命令，以便你能够构建自己的容器镜像。

以下是一些重要的 Dockerfile 构建命令，你应该知道：

1.  `FROM` ([`docs.docker.com/engine/reference/builder/#from`](https://docs.docker.com/engine/reference/builder/#from)): `FROM`描述了你用作构建容器基础的图像，并且通常是 Dockerfile 中的第一个命令。Docker 最佳实践鼓励使用 Debian 作为基础 Linux 发行版。正如你之前从我的示例中看到的，我更喜欢使用 Alpine Linux，因为它非常紧凑。你也可以使用 Ubuntu、Fedora 和 CentOS，它们都是更大的图像，并在其基本图像中包含了更多的软件。如果你已经熟悉某个 Linux 发行版及其使用的工具，那么我建议你利用这些知识来制作你的第一个容器。你还经常可以找到专门构建的容器来支持你正在使用的语言，比如 Node 或 Python。在撰写本文时（2017 年秋），我下载了各种这些图像来展示它们的相对大小：

```
REPOSITORY     TAG         IMAGE ID        CREATED          SIZE
alpine         latest      76da55c8019d    2 days ago       3.97MB
debian         latest      72ef1cf971d1    2 days ago       100MB
fedora         latest      ee17cf9e0828    2 days ago       231MB
centos         latest      196e0ce0c9fb    27 hours ago     197MB
ubuntu         latest      8b72bba4485f    2 days ago       120MB
ubuntu         16.10       7d3f705d307c    8 weeks ago      107MB
python         latest      26acbad26a2c    2 days ago       690MB
node           latest      de1099630c13    24 hours ago     673MB
java           latest      d23bdf5b1b1b    8 months ago     643MB
```

正如你所看到的，这些图像的大小差异很大。

你可以在[`hub.docker.com/explore/`](https://hub.docker.com/explore/)上探索这些（以及各种其他基本图像）。

1.  `RUN` ([`docs.docker.com/engine/reference/builder/#run`](https://docs.docker.com/engine/reference/builder/#run)): `RUN`描述了您在构建的容器映像中运行的命令，最常用于添加依赖项或其他库。如果您查看其他人创建的 Dockerfile，通常会看到`RUN`命令用于使用诸如`apt-get install ...`或`rpm -ivh ...`的命令安装库。使用的命令取决于基本映像的选择；例如，`apt-get`在 Debian 和 Ubuntu 基本映像上可用，但在 Alpine 或 Fedora 上不可用。如果您输入一个不可用的`RUN`命令（或只是打字错误），那么在运行`docker build`命令时会看到错误。例如，在构建 Dockerfile 时：

```
FROM alpine
RUN apt-get install nodejs
Results in the following output:
Sending build context to Docker daemon  2.048kB
Step 1/2 : FROM alpine
 ---> 76da55c8019d
Step 2/2 : RUN apt-get install nodejs
 ---> Running in 58200129772d
/bin/sh: apt-get: not found
```

`/bin/sh -c apt-get install nodejs` 命令返回了非零代码：`127`。

1.  `ENV` ([`docs.docker.com/engine/reference/builder/#env`](https://docs.docker.com/engine/reference/builder/#env)): `ENV`定义了将在容器映像中持久存在并在调用软件之前设置的环境变量。这些也在创建容器映像时设置，这可能会导致意想不到的影响。例如，如果您需要为特定的`RUN`命令设置环境变量，最好是使用单个`RUN`命令而不是使用`ENV`命令来定义它。例如，在 Debian 基础映像上使用`ENV DEBIAN_FRONTEND`非交互可能会混淆后续的`RUN apt-get install …`命令。在您想要为特定的`RUN`命令启用它的情况下，您可以通过在单个`RUN`命令前临时添加环境变量来实现。例如：

```
RUN DEBIAN_FRONTEND=noninteractive apt-get install -y ...
```

1.  `COPY` ([`docs.docker.com/engine/reference/builder/#copy`](https://docs.docker.com/engine/reference/builder/#copy)): `COPY`（或`ADD`命令）是将您自己的本地文件添加到容器中的方法。这通常是将代码复制到容器映像中运行的最有效方式。您可以复制整个目录或单个文件。除了`RUN`命令之外，这可能是您使用代码创建容器映像的大部分工作。

1.  `WORKDIR` ([`docs.docker.com/engine/reference/builder/#workdir`](https://docs.docker.com/engine/reference/builder/#workdir))：`WORKDIR`创建一个本地目录，然后将该目录作为以后所有命令（`RUN`，`COPY`等）的基础。对于期望从本地或相对目录运行的`RUN`命令，例如 Node.js `npm`等安装工具，这可能非常方便。

1.  `LABEL` ([`docs.docker.com/engine/reference/builder/#label`](https://docs.docker.com/engine/reference/builder/#label))：`LABEL`添加的值可在`docker inspect`中看到，并通常用作容器内部的责任人或内容的参考。`MAINTAINER`命令以前非常常见，但已被`LABEL`命令取代。标签是基于基本镜像构建的，并且是可累加的，因此您添加的任何标签都将与您使用的基本镜像的标签一起包含在内。

1.  `CMD` ([`docs.docker.com/engine/reference/builder/#cmd`](https://docs.docker.com/engine/reference/builder/#cmd))和`ENTRYPOINT` ([`docs.docker.com/engine/reference/builder/#entrypoint`](https://docs.docker.com/engine/reference/builder/#entrypoint))：`CMD`（和`ENTRYPOINT`命令）是您指定当有人运行容器时要运行什么的方式。最常见的格式是 JSON 数组，其中第一个元素是要调用的命令，而第二个及以后的元素是该命令的参数。`CMD`和`ENTRYPOINT`旨在单独使用，这种情况下，您可以使用`CMD`或`ENTRYPOINT`来指定要运行的可执行文件和所有参数，或者一起使用，这种情况下，`ENTRYPOINT`应该只是可执行文件，而`CMD`应该是该可执行文件的参数。

# 示例 - Python/Flask 容器镜像

为了详细了解如何使用 Kubernetes，我创建了两个示例应用程序，您可以下载或复制以便跟随并尝试这些命令。其中一个是使用 Flask 库的非常简单的 Python 应用程序。示例应用程序直接来自 Flask 文档（[`flask.pocoo.org/docs/0.12/`](http://flask.pocoo.org/docs/0.12/)）。

您可以从 GitHub 上下载这些代码，网址为[`github.com/kubernetes-for-developers/kfd-flask/tree/first_container`](https://github.com/kubernetes-for-developers/kfd-flask/tree/first_container)。由于我们将不断改进这些文件，因此此处引用的代码可在`first_container`标签处获得。如果您想使用 Git 获取这些文件，可以运行以下命令：

```
git clone https://github.com/kubernetes-for-developers/kfd-flask
```

然后，进入存储库并检出标签：

```
cd kfd-flask
git checkout tags/first_container
```

让我们从查看 Dockerfile 的内容开始，该文件定义了构建到容器中的内容以及构建过程。

我们创建这个 Dockerfile 的目标是：

+   获取并安装底层操作系统的任何安全补丁

+   安装我们需要用来运行代码的语言或运行时

+   安装我们的代码中未直接包含的任何依赖项

+   将我们的代码复制到容器中

+   定义如何以及何时运行

```
FROM alpine
# load any public updates from Alpine packages
RUN apk update
# upgrade any existing packages that have been updated
RUN apk upgrade
# add/install python3 and related libraries
# https://pkgs.alpinelinux.org/package/edge/main/x86/python3
RUN apk add python3
# make a directory for our application
RUN mkdir -p /opt/exampleapp
# move requirements file into the container
COPY . /opt/exampleapp/
# install the library dependencies for this application
RUN pip3 install -r /opt/exampleapp/requirements.txt
ENTRYPOINT ["python3"]
CMD ["/opt/exampleapp/exampleapp.py"]
```

此容器基于 Alpine Linux。我欣赏容器的小巧尺寸，并且容器中没有多余的软件。您将看到一些可能不熟悉的命令，特别是`apk`命令。这是一个命令行工具，用于帮助安装、更新和删除 Alpine Linux 软件包。这些命令更新软件包存储库，升级镜像中安装的和预先存在的所有软件包，然后从软件包中安装 Python 3。

如果您已经熟悉 Debian 命令（如`apt-get`）或 Fedora/CentOS 命令（如`rpm`），那么我建议您使用这些基本 Linux 容器进行您自己的工作。

接下来的两个命令在容器中创建一个目录`/opt/exampleapp`来存放我们的源代码，并将所有内容复制到指定位置。`COPY`命令将本地目录中的所有内容添加到容器中，这可能比我们需要的要多。您可以在将来创建一个名为`.dockerignore`的文件，该文件将根据模式`ignore`一组文件，以便在`COPY`命令中忽略一些不想包含的常见文件。

接下来，您将看到一个`RUN`命令，该命令安装应用程序的依赖项，这些依赖项来自名为`requirements.txt`的文件，该文件包含在源代码库中。在这种情况下，将依赖项保存在这样的文件中是一个很好的做法，而`pip`命令就是为了支持这样做而创建的。

最后两个命令分别利用了 `ENTRYPOINT` 和 `CMD`。对于这个简单的例子，我可以只使用其中一个。两者都包括在内是为了展示它们如何一起使用，`CMD` 本质上是传递给 `ENTRYPOINT` 中定义的可执行文件的参数。

# 构建容器

我们将使用 `docker build` 命令来创建容器。在终端窗口中，切换到包含 Dockerfile 的目录，并运行以下命令：

```
docker build .
```

您应该看到类似以下的输出：

```
Sending build context to Docker daemon    107kB
Step 1/9 : FROM alpine
 ---> 76da55c8019d
Step 2/9 : RUN apk update
 ---> Running in f72d5991a7cd
fetch http://dl-cdn.alpinelinux.org/alpine/v3.6/main/x86_64/APKINDEX.tar.gz
fetch http://dl-cdn.alpinelinux.org/alpine/v3.6/community/x86_64/APKINDEX.tar.gz
v3.6.2-130-gfde2d8ebb8 [http://dl-cdn.alpinelinux.org/alpine/v3.6/main]
v3.6.2-125-g93038b573e [http://dl-cdn.alpinelinux.org/alpine/v3.6/community]
OK: 8441 distinct packages available
 ---> b44cd5d0ecaa
Removing intermediate container f72d5991a7cd
```

Dockerfile 中的每一步都会反映在 Docker 构建镜像时发生的输出，对于更复杂的 Dockerfile，输出可能会非常庞大。在完成构建过程时，它会报告总体成功或失败，并且还会报告容器的 ID：

```
Step 8/9 : ENTRYPOINT python3
 ---> Running in 14c58ace8b14
 ---> 0ac8be8b042d
Removing intermediate container 14c58ace8b14
Step 9/9 : CMD /opt/exampleapp/exampleapp.py
 ---> Running in e769a65fedbc
 ---> b704504464dc
Removing intermediate container e769a65fedbc
Successfully built 4ef370855f35
```

当我们构建容器时，如果没有其他信息，它会在本地创建一个我们可以使用的镜像（它有一个 ID），但它没有名称或标签。在选择名称时，通常要考虑您托管容器镜像的位置。在这种情况下，我使用的是 CoreOS 的 Quay.io 服务，该服务为开源容器镜像提供免费托管。

要为刚刚创建的镜像打标签，我们可以使用 `docker tag` 命令：

```
docker tag 4ef370855f35 quay.io/kubernetes-for-developers/flask
```

这个标签包含三个相关部分。第一个 [quay.io](http://quay.io) 是容器注册表。第二个 (`kubernetes-for-developers`) 是您容器的命名空间，第三个 (`flask`) 是容器的名称。我们没有为容器指定任何特定的标签，所以 `docker` 命令将使用 latest。

您应该使用标签来表示发布或开发中的其他时间点，以便您可以轻松地返回到这些时间点，并使用 latest 来表示您最近的开发工作，因此让我们也将其标记为一个特定的版本：

```
docker tag 4ef370855f35 quay.io/kubernetes-for-developers/flask:0.1.0
```

当您与他人共享镜像时，明确指出您正在使用的镜像是一个非常好的主意。一般来说，只考虑自己使用代码，每当与其他人共享镜像时，都要使用明确的标签。标签不一定是一个版本，虽然它的格式有限制，但几乎可以是任何字符串。

您可以使用 `docker push` 命令将已经标记的镜像传输到容器仓库。您需要先登录到您的容器仓库：

```
docker login quay.io
```

然后你可以推送这个镜像：

```
docker push quay.io/kubernetes-for-developers/flask
```

推送是指一个仓库，`[quay.io/kubernetes-for-developers/flask]`：

```
0b3b7598137f: Pushed
602c2b5ffa76: Pushed 217607c1e257: Pushed
40ca06be4cf4: Pushed 5fbd4bb748e7: Pushed
0d2acef20dc1: Pushed
5bef08742407: Pushed
latest: digest: sha256:de0c5b85893c91062fcbec7caa899f66ed18d42ba896a47a2f4a348cbf9b591f size: 5826
```

通常，您希望从一开始就使用标签构建您的容器，而不是必须执行额外的命令。为此，您可以使用`-t <your_name>`选项将标签信息添加到`build`命令中。对于本书中的示例，我使用的名称是`kubernetes-for-developers`，因此我一直在使用以下命令构建示例：

```
docker build -t quay.io/kubernetes-for-developers/flask .
```

如果您正在按照此示例操作，请在先前命令中的`quay.io/kubernetes-for-developers/flask .`处使用您自己的值。您应该看到以下输出：

```
Sending build context to Docker daemon    107kB
Step 1/9 : FROM alpine
 ---> 76da55c8019d
Step 2/9 : RUN apk update
 ---> Using cache
 ---> b44cd5d0ecaa
Step 3/9 : RUN apk upgrade
 ---> Using cache
 ---> 0b1caea1a24d
Step 4/9 : RUN apk add python3
 ---> Using cache
 ---> 1e29fcb9621d
Step 5/9 : RUN mkdir -p /opt/exampleapp
 ---> Using cache
 ---> 622a12042892
Step 6/9 : COPY . /opt/exampleapp/
 ---> Using cache
 ---> 7f9115a50a0a
Step 7/9 : RUN pip3 install -r /opt/exampleapp/requirements.txt
 ---> Using cache
 ---> d8ef21ee1209
Step 8/9 : ENTRYPOINT python3
 ---> Using cache
 ---> 0ac8be8b042d
Step 9/9 : CMD /opt/exampleapp/exampleapp.py
 ---> Using cache
 ---> b704504464dc
Successfully built b704504464dc
Successfully tagged quay.io/kubernetes-for-developers/flask:latest
```

花点时间阅读该输出，并注意在几个地方报告了`Using cache`。您可能还注意到，该命令比您第一次构建镜像时更快。

这是因为 Docker 尝试重用任何未更改的层，以便它不必重新创建该工作。由于我们刚刚执行了所有这些命令，它可以使用在创建上一个镜像时制作的缓存中的层。

如果运行`docker images`命令，您现在应该看到它被列出：

```
REPOSITORY                                TAG       IMAGE ID      CREATED          SIZE
quay.io/kubernetes-for-developers/flask   0.1.0     b704504464dc  2 weeks ago         70.1MB
quay.io/kubernetes-for-developers/flask   latest    b704504464dc  2 weeks ago         70.1MB
```

随着您继续使用容器映像来存储和部署您的代码，您可能希望自动化创建映像的过程。作为一般模式，一个良好的构建过程应该是：

+   从源代码控制获取代码

+   `docker build`

+   `docker tag`

+   `docker push`

这是我们在这些示例中使用的过程，您可以使用最适合您的工具自动化这些命令。我建议您设置一些可以快速且一致地在命令行上运行的东西。

# 运行您的容器

现在，让我们运行刚刚制作的容器。我们将使用`kubectl run`命令来指定最简单的部署——只是容器：

```
kubectl run flask --image=quay.io/kubernetes-for-developers/flask:latest --port=5000 --save-config
deployment “flask” created
```

要查看这是在做什么，我们需要向集群询问我们刚刚创建的资源的当前状态。当我们使用`kubectl run`命令时，它将隐式为我们创建一个 Deployment 资源，并且正如您在上一章中学到的，Deployment 中有一个 ReplicaSet，ReplicaSet 中有一个 Pod：

```
kubectl get deployments
NAME      DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
flask     1         1         1            1           20h
kubectl get pods
NAME                     READY     STATUS    RESTARTS   AGE
flask-1599974757-b68pw   1/1       Running   0          20h
```

我们可以通过请求与 Kubernetes 部署资源`flask`相关联的原始数据来获取有关此部署的详细信息：

```
kubectl get deployment flask -o json
```

我们也可以轻松地请求以`YAML`格式的信息，或者查询这些细节的子集，利用 JsonPath 或`kubectl`命令的其他功能。JSON 输出将非常丰富。它将以一个指示来自 Kubernetes 的 apiVersion 的键开始，资源的种类以及关于资源的元数据：

```
{
    "apiVersion": "extensions/v1beta1",
    "kind": "Deployment",
    "metadata": {
        "annotations": {
            "deployment.kubernetes.io/revision": "1"
        },
        "creationTimestamp": "2017-09-16T00:40:44Z",
        "generation": 1,
        "labels": {
            "run": "flask"
        },
        "name": "flask",
        "namespace": "default",
        "resourceVersion": "51293",
        "selfLink": "/apis/extensions/v1beta1/namespaces/default/deployments/flask",
        "uid": "acbb0128-9a77-11e7-884c-0aef48c812e4"
    },
```

在此之下通常是部署本身的规范，其中包含了大部分正在运行的核心内容。

```
    "spec": {
        "replicas": 1,
        "selector": {
            "matchLabels": {
                "run": "flask"
            }
        },
        "strategy": {
            "rollingUpdate": {
                "maxSurge": 1,
                "maxUnavailable": 1
            },
            "type": "RollingUpdate"
        },
        "template": {
            "metadata": {
                "creationTimestamp": null,
                "labels": {
                    "run": "flask"
                }
            },
            "spec": {
                "containers": [
                    {
                        "image": "quay.io/kubernetes-for-developers/flask:latest",
                        "imagePullPolicy": "Always",
                        "name": "flask",
                        "ports": [
                            {
                                "containerPort": 5000,
                                "protocol": "TCP"
                            }
                        ],
                        "resources": {},
                        "terminationMessagePath": "/dev/termination-log",
                        "terminationMessagePolicy": "File"
                    }
                ],
                "dnsPolicy": "ClusterFirst",
                "restartPolicy": "Always",
                "schedulerName": "default-scheduler",
                "securityContext": {},
                "terminationGracePeriodSeconds": 30
            }
        }
    },
```

最后一部分通常是状态，它指示了部署的当前状态，即您请求信息的时间。

```
    "status": {
        "availableReplicas": 1,
        "conditions": [
            {
                "lastTransitionTime": "2017-09-16T00:40:44Z",
                "lastUpdateTime": "2017-09-16T00:40:44Z",
                "message": "Deployment has minimum availability.",
                "reason": "MinimumReplicasAvailable",
                "status": "True",
                "type": "Available"
            }
        ],
        "observedGeneration": 1,
        "readyReplicas": 1,
        "replicas": 1,
        "updatedReplicas": 1
    }
}
```

在 Kubernetes 中运行 Pod 时，请记住它是在一个沙盒中运行的，与世界其他部分隔离开来。Kubernetes 有意这样做，这样您就可以指定 Pod 应该如何连接以及集群外部可以访问什么。我们将在后面的章节中介绍如何设置外部访问。与此同时，您可以利用`kubectl`中的两个命令直接从开发机器获取访问权限：`kubectl port-forward`或`kubectl proxy`。

这两个命令都是通过将代理从您的本地开发机器到 Kubernetes 集群，为您提供对正在运行的代码的私人访问权限。`port-forward`命令将打开一个特定的 TCP（或 UDP）端口，并安排所有流量转发到集群中的 Pod。代理命令使用已经存在的 HTTP 代理来转发 HTTP 流量进出您的 Pod。这两个命令都依赖于知道 Pod 名称来建立连接。

# Pod 名称

由于我们正在使用一个 Web 服务器，使用代理是最合理的，因为它将根据 Pod 的名称将 HTTP 流量转发到一个 URL。在这之前，我们将使用`port-forward`命令，如果您的编写不使用 HTTP 协议，这将更相关。

你需要的关键是创建的 Pod 的名称。当我们之前运行`kubectl get pods`时，你可能注意到名称不仅仅是`flask`，而是在名称中包含了一些额外的字符：`flask-1599974757-b68pw`。当我们调用`kubectl run`时，它创建了一个部署，其中包括一个包裹在 Pod 周围的 Kubernetes ReplicaSet。名称的第一部分（`flask`）来自部署，第二部分（`1599974757`）是分配给创建的 ReplicaSet 的唯一名称，第三部分（`b68pw`）是分配给创建的 Pod 的唯一名称。如果你运行以下命令：

```
kubectl get replicaset 
```

结果将显示副本集：

```
NAME               DESIRED   CURRENT   READY     AGE
flask-1599974757   1         1         1         21h
```

你可以看到 ReplicaSet 名称是 Pod 名称的前两部分。

# 端口转发

现在我们可以使用该名称来要求`kubectl`设置一个代理，将我们指定的本地端口上的所有流量转发到我们确定的 Pod 关联的端口。通过使用以下命令查看使用部署创建的 Pod 的完整名称：

```
kubectl get pods
```

在我的例子中，结果是`flask-1599974757-b68pw`，然后可以与`port-forward`命令一起使用：

```
kubectl port-forward flask-1599974757-b68pw 5000:5000
```

输出应该类似于以下内容：

```
Forwarding from 127.0.0.1:5000 -> 5000
Forwarding from [::1]:5000 -> 5000
```

这将转发在本地机器上创建的任何流量到 Pod`flask-1599974757-b68pw`的 TCP 端口`5000`上的 TCP 端口`5000`。

你会注意到你还没有回到命令提示符，这是因为命令正在积极运行，以保持我们请求的特定隧道活动。如果我们取消或退出`kubectl`命令，通常通过按*Ctrl* + C，那么端口转发将立即结束。`kubectl proxy`的工作方式相同，因此当你使用诸如`kubectl port-forward`或`kubectl proxy`的命令时，你可能希望打开另一个终端窗口单独运行该命令。

当命令仍在运行时，打开浏览器并输入此 URL：`http://localhost:5000`。响应应该返回`Index Page`。当我们调用`kubectl run`命令时，我特意选择端口`5000`来匹配 Flask 的默认端口。

# 代理

你可以使用的另一个命令来访问你的 Pod 是`kubectl proxy`命令。代理不仅提供对你的 Pod 的访问，还提供对所有 Kubernetes API 的访问。要调用代理，运行以下命令：

```
kubectl proxy
```

输出将显示类似于以下内容：

```
Starting to serve on 127.0.0.1:8001
```

与`port-forward`命令一样，在代理终止之前，您在终端窗口中将不会收到提示。在它处于活动状态时，您可以通过这个代理访问 Kubernetes REST API 端点。打开浏览器，输入 URL `http://localhost:8001/`。

您应该看到一个类似以下的 JSON 格式的 URL 列表：

```
{
 "paths": [ "/api", "/api/v1", "/apis", "/apis/", "/apis/admissionregistration.k8s.io", "/apis/admissionregistration.k8s.io/v1alpha1", "/apis/apiextensions.k8s.io", "/apis/apiextensions.k8s.io/v1beta1", "/apis/apiregistration.k8s.io",
```

这些都是直接访问 Kubernetes 基础设施。其中一个 URL 是`/api/v1` - 尽管它没有被明确列出，它使用 Kubernetes API 服务器来根据名称为 Pod 提供代理。当我们调用我们的`run`命令时，我们没有指定一个命名空间，所以它使用了默认的命名空间，称为`default`。查看 Pod 的 URL 模式是：

`http://localhost:8001/api/v1/proxy/namespaces/<NAME_OF_NAMESPACE>/pods/<POD_NAME>/`

在我们的 Pod 的情况下，这将是：

`http://localhost:8001/api/v1/proxy/namespaces/default/pods/flask-1599974757-b68pw/`

如果您在浏览器中打开一个由 Kubernetes 集群分配的 Pod 名称创建的 URL，它应该显示与使用`port-forward`命令看到的相同的输出。

# 代理是如何知道连接到容器上的端口 5000 的？

当您运行容器时，Kubernetes 并不会神奇地知道您的代码正在侦听哪些 TCP 端口。当我们使用`kubectl run`命令创建此部署时，我们在该命令的末尾添加了`--port=5000`选项。Kubernetes 使用这个选项来知道程序应该在端口`5000`上侦听 HTTP 流量。如果您回顾一下`kubectl get deployment -o json`命令的输出，您会看到其中一个部分在`containers`键下包括我们提供的镜像、部署的名称和一个数据结构，指示访问容器的默认端口为`5000`。如果我们没有提供额外的细节，代理将假定我们希望在端口`80`访问容器。由于我们的开发容器上没有在端口`80`上运行任何内容，您会看到类似于以下的错误：

```
Error: 'dial tcp 172.17.0.4:80: getsockopt: connection refused'
Trying to reach: 'http://172.17.0.4/'
```

# 从您的应用程序获取日志

有更多的方法可以与在容器中运行的代码进行交互，我们将在以后的章节中介绍。如果您运行的代码没有侦听 TCP 套接字以提供 HTTP 流量，或者类似的内容，那么通常您希望查看您的代码创建的输出以知道它是否正在运行。

容器专门设置为捕获您指定的可执行文件的 STDOUT 和 STDERR 的任何输出，并将其捕获到日志中，可以使用另一个`kubectl`命令检索这些日志：`kubectl logs`。与`proxy`和`port-forward`命令一样，此命令需要知道您要交互的 Pod 的名称。

运行以下命令：

```
kubectl logs flask-1599974757-b68pw
```

您应该看到类似以下的输出：

```
 * Running on http://0.0.0.0:5000/ (Press CTRL+C to quit)
 * Restarting with stat
 * Debugger is active!
 * Debugger PIN: 996-805-904
```

# 示例 - Node.js/Express 容器映像

此示例遵循与 Python 示例相同的模式，即使用 Express 库构建的简单 Node.js 应用程序，以详细介绍如何使用 Kubernetes。如果您更熟悉 JavaScript 开发，则此示例可能更有意义。示例应用程序直接来自 Express 文档([`expressjs.com/en/starter/generator.html`](http://flask.pocoo.org/docs/0.12/))。

您可以从 GitHub 下载此代码副本[`github.com/kubernetes-for-developers/kfd-nodejs/tree/first_container`](https://github.com/kubernetes-for-developers/kfd-nodejs/tree/first_container)。由于我们将使这些文件发展，此处引用的代码可在`first_container`标签处获得。如果要使用 Git 检索这些文件，可以使用以下命令：

```
git clone https://github.com/kubernetes-for-developers/kfd-nodejs
cd kfd-nodejs
git checkout tags/first_container
```

与 Python 示例一样，我们将从 Dockerfile 开始。提醒一下，这是定义构建到容器中的内容以及构建方式的文件。此 Dockerfile 的目标是：

+   获取并安装基础操作系统的任何关键安全补丁

+   安装我们将需要用来运行我们的代码的语言或运行时

+   安装我们的代码中未直接包含的任何依赖项

+   将我们的代码复制到容器中

+   定义如何以及何时运行

```
FROM alpine
# load any public updates from Alpine packages
RUN apk update
# upgrade any existing packages that have been updated
RUN apk upgrade
# add/install python3 and related libraries
# https://pkgs.alpinelinux.org/package/edge/main/x86/python3
RUN apk add nodejs nodejs-npm
# make a directory for our application
WORKDIR /src
# move requirements file into the container
COPY package.json .
COPY package-lock.json .
# install the library dependencies for this application
RUN npm install --production
# copy in the rest of our local source
COPY . .
# set the debug environment variable
ENV DEBUG=kfd-nodejs:*
CMD ["npm", "start"]
```

与 Python 示例一样，此容器基于 Alpine Linux。您将看到一些可能不熟悉的命令，特别是`apk`命令。作为提醒，此命令用于安装、更新和删除 Alpine Linux 软件包。这些命令更新 Alpine 软件包存储库，升级图像中安装的所有已安装和预先存在的软件包，然后从软件包安装`nodejs`和`npm`。这些步骤基本上使我们得到一个可以运行 Node.js 应用程序的最小容器。

接下来的命令在容器中创建一个目录`/src`来存放我们的源代码，复制`package.json`文件，然后使用`npm`安装运行代码所需的依赖项。`npm install`命令与`--production`选项一起使用，只安装`package.json`中列出的运行代码所需的项目 - 开发依赖项被排除在外。Node.js 通过其`package.json`格式轻松而一致地维护依赖关系，将生产所需的依赖与开发所需的依赖分开是一个良好的做法。

最后两个命令利用了`ENV`和`CMD`。这与 Python 示例不同，我在 Python 示例中使用了`CMD`和`ENTRYPOINT`来突出它们如何一起工作。在这个示例中，我使用`ENV`命令将`DEBUG`环境变量设置为与 Express 文档中示例指令匹配。然后`CMD`包含一个启动我们代码的命令，简单地利用`npm`运行`package.json`中定义的命令，并使用之前的`WORKDIR`命令为该调用设置本地目录。

# 构建容器

我们使用相同的`docker build`命令来创建容器：

```
docker build .
```

你应该看到类似以下的输出：

```
Sending build context to Docker daemon  197.6kB
Step 1/11 : FROM alpine
 ---> 76da55c8019d
Step 2/11 : RUN apk update
 ---> Using cache
 ---> b44cd5d0ecaa
```

就像你在基于 Python 的示例中看到的那样，Dockerfile 中的每个步骤都会反映出输出，显示 Docker 根据你的指令（Dockerfile）构建容器镜像的过程：

```
Step 9/11 : COPY . .
 ---> 6851a9088ce3
Removing intermediate container 9fa9b8b9d463
Step 10/11 : ENV DEBUG kfd-nodejs:*
 ---> Running in 663a2cd5f31f
 ---> 30c3b45c4023
Removing intermediate container 663a2cd5f31f
Step 11/11 : CMD npm start
 ---> Running in 52cf9638d065
 ---> 35d03a9d90e6
Removing intermediate container 52cf9638d065
Successfully built 35d03a9d90e6
```

与 Python 示例一样，这将构建一个只有 ID 的容器。这个示例还利用 Quay 来公开托管图像，因此我们将适当地获取图像，以便上传到 Quay：

```
docker tag 35d03a9d90e6 quay.io/kubernetes-for-developers/nodejs
```

与 Python 示例一样，标签包含三个相关部分 - [quay.io](http://quay.io) 是容器注册表。第二个（`kubernetes-for-developers`）是容器的命名空间，第三个（`nodejs`）是容器的名称。与 Python 示例一样，使用相同的命令上传容器，引用`nodejs`而不是`flask`：

```
docker login quay.io docker push quay.io/kubernetes-for-developers/nodejs
```

```
The push refers to a repository [quay.io/kubernetes-for-developers/nodejs]
0b6165258982: Pushed
8f16769fa1d0: Pushed
3b43ed4da811: Pushed
9e4ead6d58f7: Pushed
d56b3cb786f1: Pushedfad7fd538fb6: Pushing [==================>                                ]  11.51MB/31.77MB
5fbd4bb748e7: Pushing [==================================>                ]  2.411MB/3.532MB
0d2acef20dc1: Pushing [==================================================>]  1.107MB
5bef08742407: Pushing [================>                                  ]  1.287MB/3.966MB
```

完成后，你应该看到类似以下的内容：

```
The push refers to a repository [quay.io/kubernetes-for-developers/nodejs]
0b6165258982: Pushed
8f16769fa1d0: Pushed
3b43ed4da811: Pushed
9e4ead6d58f7: Pushed
d56b3cb786f1: Pushed
fad7fd538fb6: Pushed
5fbd4bb748e7: Pushed
0d2acef20dc1: Pushed
5bef08742407: Pushed
latest: digest: sha256:0e50e86d27a4b29b5b10853d631d8fc91bed9a37b44b111111dcd4fd9f4bc723 size: 6791
```

与 Python 示例一样，你可能希望在同一命令中构建和标记。对于 Node.js 示例，该命令将是：

```
docker build -t quay.io/kubernetes-for-developers/nodejs:0.2.0 .
```

如果在构建图像后立即运行，应该显示类似以下的输出：

```
Sending build context to Docker daemon  197.6kB
Step 1/11 : FROM alpine
 ---> 76da55c8019d
Step 2/11 : RUN apk update
 ---> Using cache
 ---> b44cd5d0ecaa
Step 3/11 : RUN apk upgrade
 ---> Using cache
 ---> 0b1caea1a24d
Step 4/11 : RUN apk add nodejs nodejs-npm
 ---> Using cache
 ---> 193d3570516a
Step 5/11 : WORKDIR /src
 ---> Using cache
 ---> 3a5d78afa1be
Step 6/11 : COPY package.json .
 ---> Using cache
 ---> 29724b2bd1b9
Step 7/11 : COPY package-lock.json .
 ---> Using cache
 ---> ddbcb9af6ffc
Step 8/11 : RUN npm install --production
 ---> Using cache
 ---> 1556a20af49a
Step 9/11 : COPY . .
 ---> Using cache
 ---> 6851a9088ce3
Step 10/11 : ENV DEBUG kfd-nodejs:*
 ---> Using cache
 ---> 30c3b45c4023
Step 11/11 : CMD npm start
 ---> Using cache
 ---> 35d03a9d90e6
Successfully built 35d03a9d90e6
Successfully tagged quay.io/kubernetes-for-developers/nodejs:latest
```

与使用 Docker 缓存的图像层相比，速度会快得多，因为它使用了先前构建的图像层。

如果运行`docker images`命令，你现在应该能看到它被列出来了：

```
REPOSITORY                                TAG       IMAGE ID      CREATED          SIZE
quay.io/kubernetes-for-developers/nodejs  0.2.0    46403c409d1f  4 minutes ago    81.9MB
```

如果你将自己的镜像推送到`quay.io`作为容器仓库，除了这些命令，你可能需要登录网站并将镜像设为公开。默认情况下，`quay.io`会保持镜像私有，即使是公共的，直到你在他们的网站上批准它们的公开。

# 运行你的容器

现在，让我们运行刚刚创建的容器。我们将使用`kubectl run`命令，就像 Python 示例一样，但是用`nodejs`替换 flask 来指定我们刚刚创建和上传的容器：

```
kubectl run nodejs --image=quay.io/kubernetes-for-developers/nodejs:0.2.0 --port=3000
deployment “nodejs” created
```

为了查看它的运行情况，我们需要向集群请求刚刚创建的资源的当前状态：

```
kubectl get deployments
NAME      DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
nodejs    1         1         1            1           1d
kubectl get pods
NAME                     READY     STATUS    RESTARTS   AGE
nodejs-568183341-2bw5v   1/1       Running   0          1d
```

`kubectl run`命令不受语言限制，并且与 Python 示例的方式相同。在这种情况下创建的简单部署被命名为`nodejs`，我们可以请求与之前 Python 示例相同类型的信息：

```
kubectl get deployment nodejs -o json
```

JSON 输出应该会非常详细，并且会有多个部分。输出的顶部会有关于部署的`apiVersion`，`kind`和`metadata`：

```
{
    "apiVersion": "extensions/v1beta1",
    "kind": "Deployment",
    "metadata": {
        "annotations": {
            "deployment.kubernetes.io/revision": "1"
        },
        "creationTimestamp": "2017-09-16T10:06:30Z",
        "generation": 1,
        "labels": {
            "run": "nodejs"
        },
        "name": "nodejs",
        "namespace": "default",
        "resourceVersion": "88886",
        "selfLink": "/apis/extensions/v1beta1/namespaces/default/deployments/nodejs",
        "uid": "b5d94f83-9ac6-11e7-884c-0aef48c812e4"
    },
```

通常，在这之下会有`spec`，其中包含了你刚刚要求运行的核心内容：

```
    "spec": {
        "replicas": 1,
        "selector": {
            "matchLabels": {
                "run": "nodejs"
            }
        },
        "strategy": {
            "rollingUpdate": {
                "maxSurge": 1,
                "maxUnavailable": 1
            },
            "type": "RollingUpdate"
        },
        "template": {
            "metadata": {
                "creationTimestamp": null,
                "labels": {
                    "run": "nodejs"
                }
            },
            "spec": {
                "containers": [
                    {
                        "image": "quay.io/kubernetes-for-developers/nodejs:0.2.0",
                        "imagePullPolicy": "IfNotPresent",

                        "name": "nodejs",
                        "ports": [
                            {
                                "containerPort": 3000,
                                "protocol": "TCP"
                            }
                        ],
                        "resources": {},
                        "terminationMessagePath": "/dev/termination-log",
                        "terminationMessagePolicy": "File"
                    }
                ],
                "dnsPolicy": "ClusterFirst",
                "restartPolicy": "Always",
                "schedulerName": "default-scheduler",
                "securityContext": {},
                "terminationGracePeriodSeconds": 30
            }
        }
    },
```

最后一部分是`status`，它指示了部署的当前状态（截至请求此信息时）：

```
    "status": {
        "availableReplicas": 1,
        "conditions": [
            {
                "lastTransitionTime": "2017-09-16T10:06:30Z",
                "lastUpdateTime": "2017-09-16T10:06:30Z",
                "message": "Deployment has minimum availability.",
                "reason": "MinimumReplicasAvailable",
                "status": "True",
                "type": "Available"
            }
        ],
        "observedGeneration": 1,
        "readyReplicas": 1,
        "replicas": 1,
        "updatedReplicas": 1
    }
}
```

当 Pod 在 Kubernetes 中运行时，它是在一个与世隔绝的沙盒中运行的。Kubernetes 故意这样做，这样你就可以指定哪些系统可以相互通信，以及可以从外部访问什么。对于大多数集群，Kubernetes 的默认设置允许任何 Pod 与任何其他 Pod 通信。就像 Python 示例一样，你可以利用`kubectl`中的两个命令之一来从开发机器直接访问：`kubectl` port-forward 或`kubectl` proxy。

# 端口转发

现在我们可以使用这个名称来要求`kubectl`设置一个代理，将我们指定的本地端口的所有流量转发到我们确定的 Pod 关联的端口。Node.js 示例在不同的端口上运行（端口`3000`而不是端口`5000`），因此命令需要相应地更新：

```
kubectl port-forward nodejs-568183341-2bw5v 3000:3000
```

输出应该类似于以下内容：

```
Forwarding from 127.0.0.1:3000 -> 3000
Forwarding from [::1]:3000 -> 3000
```

这会将在本地机器上创建的任何流量转发到`nodejs-568183341-2bw5v` Pod 上的 TCP 端口`3000`。

就像 Python 示例一样，因为命令正在运行以保持这个特定的隧道活动，所以你还没有得到一个命令提示符。提醒一下，你可以通过按下 *Ctrl* + *C* 来取消或退出 `kubectl` 命令，端口转发将立即结束。

当命令仍在运行时，打开浏览器并输入此 URL：`http://localhost:3000`。响应应该返回说 `Index Page`。当我们调用 `kubectl run` 命令时，我特意选择端口 `3000` 来匹配 Express 的默认端口。

# 代理

由于这是一个基于 HTTP 的应用程序，我们也可以使用 `kubectl proxy` 命令来访问我们代码的响应：

```
kubectl proxy
```

输出将显示类似于以下内容：

```
Starting to serve on 127.0.0.1:8001
```

提醒一下，在代理终止之前，你不会在终端窗口中得到提示符。就像 Python 示例一样，我们可以根据我们在调用 `kubectl run` 命令时使用的 Pod 名称和命名空间来确定代理将用于转发到我们容器的 URL。由于我们没有指定命名空间，它使用的是默认的 `default`。访问 Pod 的 URL 模式与 Python 示例相同：

`http://localhost:8001/api/v1/proxy/namespaces/<NAME_OF_NAMESPACE>/pods/<POD_NAME>/`

在我们的 Pod 的情况下，这将是：

`http://localhost:8001/api/v1/proxy/namespaces/default/pods/nodejs-568183341-2bw5v/`

如果你在浏览器中打开一个由你的 Kubernetes 集群分配的 Pod 名称创建的 URL，它应该显示与使用 `port-forward` 命令看到的相同的输出。

# 获取应用程序的日志

就像 Python 示例一样，Node.js 示例也会将一些输出发送到 `STDOUT`。由于容器专门设置来捕获你指定的可执行文件的 `STDOUT` 和 `STDERR` 的任何输出，并将其捕获到日志中，相同的命令将起作用，以显示来自 Node.js 应用程序的日志输出：

```
kubectl logs nodejs-568183341-2bw5v
```

这应该显示类似于以下的输出：

```
> kfd-nodejs@0.0.0 start /src
> node ./bin/www
Sat, 16 Sep 2017 10:06:41 GMT kfd-nodejs:server Listening on port 3000
GET / 304 305.615 ms - -
GET /favicon.ico 404 54.056 ms - 855
GET /stylesheets/style.css 200 63.234 ms - 111
GET / 200 48.033 ms - 170
GET /stylesheets/style.css 200 1.373 ms - 111
```

# 给你的容器图像打标签

在 Docker 图像上使用 `:latest` 标签非常方便，但很容易导致混淆，不知道到底在运行什么。如果你使用 `:latest`，那么告诉 Kubernetes 在加载容器时始终尝试拉取新的镜像是一个非常好的主意。我们将在第四章中看到如何设置这一点，*声明式基础设施*，当我们谈论声明性地定义我们的应用程序时。

另一种方法是制作显式标签，使用标签进行构建，并使用`docker tag`将映像标记为`latest`以方便使用，但在提交到源代码控制时保持特定的标签。对于本示例，选择的标签是`0.2.0`，使用语义化版本表示要与容器一起使用的值，并与`git tag`匹配。

在制作这个示例时使用的步骤是：

```
git tag 0.2.0
docker build -t quay.io/kubernetes-for-developers/nodejs:0.2.0 .
git push origin master --tags
docker push quay.io/kubernetes-for-developers/nodejs
```

# 摘要

在本章中，我们回顾了容器的组成，如何在互联网上存储和共享容器，以及一些用于创建自己的容器的命令。然后，我们利用这些知识在 Python 和 Node.js 中进行了示例演示，分别创建了简单的基于 Web 的服务，将它们构建成容器映像，并在 Kubernetes 中运行它们。在下一章中，我们将深入探讨如何与打包成容器的代码进行交互，并探索在开发过程中充分利用容器和 Kubernetes 的技巧。
