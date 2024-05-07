# 第十章：管理容器映像

容器编排平台需要一个坚实的基础来运行我们的容器。基础设施的一个重要组成部分是存储容器映像的位置，这将允许我们在创建 pod 时可靠地获取它们。

从开发人员的角度来看，应该非常容易和快速地推送新的映像，同时开发我们希望部署到 Kubernetes 的软件。我们还希望有机制帮助我们进行版本控制、编目和描述如何使用我们的图像，以便促进部署并减少交付错误版本或配置的风险。

容器映像通常包含知识产权、专有源代码、基础设施配置秘密，甚至商业机密。因此，我们需要适当的身份验证和授权机制来保护它们免受未经授权的访问。

在本章中，我们将学习如何利用 AWS 弹性容器注册表（ECR）服务来存储我们的容器映像，以满足所有这些需求。

在本章中，我们将涵盖以下主题：

+   将 Docker 映像推送到 ECR

+   给图像打标签

+   给图像打标签

# 将 Docker 映像推送到 ECR

目前，存储和传递 Docker 映像的最常见方式是通过 Docker 注册表，这是 Docker 的一个开源应用程序，用于托管 Docker 仓库。这个应用程序可以部署在本地，也可以作为多个提供商的服务使用，比如 Docker Hub、Quay.io 和 AWS ECR。

该应用程序是一个简单的无状态服务，其中大部分维护工作涉及确保存储可用、安全和安全。正如任何经验丰富的系统管理员所知道的那样，这绝非易事，特别是如果有一个大型数据存储。因此，特别是如果您刚开始，强烈建议使用托管解决方案，让其他人负责保持图像的安全和可靠可用。

ECR 是 AWS 对托管 Docker 注册表的方法，每个帐户有一个注册表，使用 AWS IAM 对用户进行身份验证和授权以推送和拉取图像。默认情况下，仓库和图像的限制都设置为 1,000。正如我们将看到的，设置流程感觉非常类似于其他 AWS 服务，同时也对 Docker 注册表用户来说是熟悉的。

# 创建一个仓库

要创建一个仓库，只需执行以下`aws ecr`命令即可：

```
$ aws ecr create-repository --repository-name randserver 
```

这将创建一个存储我们`randserver`应用程序的存储库。其输出应该如下所示：

```
 {
 "repository": {
 "repositoryArn": "arn:aws:ecr:eu-central-1:123456789012:repository/randserver",
 "registryId": "123456789012",
 "repositoryName": "randserver",
 "repositoryUri": "123456789012.dkr.ecr.eu-central-1.amazonaws.com/randserver",
 "createdAt": 1543162198.0
 }
 } 
```

对您的存储库的一个不错的补充是一个生命周期策略，清理旧版本的图像，这样您最终不会被阻止推送更新版本。可以通过使用相同的`aws ecr`命令来实现：

```
$ aws ecr put-lifecycle-policy --registry-id 123456789012 --repository-name randserver --lifecycle-policy-text '{"rules":[{"rulePriority":10,"description":"Expire old images","selection":{"tagStatus":"any","countType":"imageCountMoreThan","countNumber":800},"action":{"type":"expire"}}]}'  
```

一旦在同一个存储库中有超过 800 个图像，这个特定的策略将开始清理。您还可以根据图像、年龄或两者进行清理，以及仅考虑清理中的一些标签。

有关更多信息和示例，请参阅[`docs.aws.amazon.com/AmazonECR/latest/userguide/lifecycle_policy_examples.html`](https://docs.aws.amazon.com/AmazonECR/latest/userguide/lifecycle_policy_examples.html)。

# 从您的工作站推送和拉取图像

为了使用您新创建的 ECR 存储库，首先我们需要对本地 Docker 守护程序进行身份验证，以针对 ECR 注册表。再次，`aws ecr`将帮助您实现这一点：

```
aws ecr get-login --registry-ids 123456789012 --no-include-email  
```

这将输出一个`docker login`命令，该命令将为您的 Docker 配置添加一个新的用户密码对。您可以复制粘贴该命令，或者您可以按照以下方式运行它；结果将是一样的：

```
$(aws ecr get-login --registry-ids 123456789012 --no-include-email)  
```

现在，推送和拉取图像就像使用任何其他 Docker 注册表一样，使用创建存储库时得到的存储库 URI 输出：

```
$ docker push 123456789012.dkr.ecr.eu-central-1.amazonaws.com/randserver:0.0.1    
$ docker pull 123456789012.dkr.ecr.eu-central-1.amazonaws.com/randserver:0.0.1  
```

# 设置推送图像的权限

IAM 用户的权限应该允许您的用户执行严格需要的操作，以避免可能会产生更大影响的任何可能的错误。对于 ECR 管理也是如此，为此，有三个 AWS IAM 托管策略大大简化了实现它的过程：

+   `AmazonEC2ContainerRegistryFullAccess`：这允许用户对您的 ECR 存储库执行任何操作，包括删除它们，因此应该留给系统管理员和所有者。

+   `AmazonEC2ContainerRegistryPowerUser`：这允许用户在任何存储库上推送和拉取图像，对于正在积极构建和部署您的软件的开发人员非常方便。

+   `AmazonEC2ContainerRegistryReadOnly`：这允许用户在任何存储库上拉取图像，对于开发人员不是从他们的工作站推送软件，而是只是拉取内部依赖项来处理他们的项目的情况非常有用。

所有这些策略都可以附加到 IAM 用户，方法是通过将 ARN 末尾的策略名称替换为适当的策略（如前所述），并将`--user-name`指向您正在管理的用户：

```
$ aws iam attach-user-policy --policy-arn arn:aws:iam::aws:policy/AmazonEC2ContainerRegistryReadOnly  --user-name johndoe
```

所有这些由 AWS 管理的策略都具有一个重要特征-它们都为您的注册表上的所有存储库添加权限。您可能会发现有几种情况远非理想-也许您的组织有几个团队不需要访问彼此的存储库；也许您希望有一个有权删除一些存储库但不是全部的用户；或者您只需要对**持续集成**（**CI**）设置中的单个存储库进行访问。

如果您的需求符合上述描述的任何情况，您应该创建自己的策略，并具有所需的细粒度权限。

首先，我们将为我们的`randserver`应用程序的开发人员创建一个 IAM 组：

```
$ aws iam create-group --group-name randserver-developers
    {
          "Group": {
          "Path": "/",
          "GroupName": "randserver-developers",
          "GroupId": "AGPAJRDMVLGOJF3ARET5K",
          "Arn": "arn:aws:iam::123456789012:group/randserver-developers",
          "CreateDate": "2018-10-25T11:45:42Z"
          }
    } 
```

然后我们将`johndoe`用户添加到组中：

```
$ aws iam add-user-to-group --group-name randserver-developers --user-name johndoe  
```

现在我们需要创建我们的策略，以便我们可以将其附加到组上。将此 JSON 文档复制到文件中：

```
{ 
   "Version": "2012-10-17", 
   "Statement": [{ 
         "Effect": "Allow", 
         "Action": [ 
               "ecr:GetAuthorizationToken", 
               "ecr:BatchCheckLayerAvailability", 
               "ecr:GetDownloadUrlForLayer", 
               "ecr:GetRepositoryPolicy", 
               "ecr:DescribeRepositories", 
               "ecr:ListImages", 
               "ecr:DescribeImages", 
               "ecr:BatchGetImage", 
               "ecr:InitiateLayerUpload", 
               "ecr:UploadLayerPart", 
               "ecr:CompleteLayerUpload", 
               "ecr:PutImage"
          ], 
         "Resource": "arn:aws:ecr:eu-central-1:123456789012:repository/randserver" 
   }] 
} 
```

要创建策略，请执行以下操作，传递适当的 JSON 文档文件路径：

```
$ aws iam create-policy --policy-name EcrPushPullRandserverDevelopers --policy-document file://./policy.json

    {
          "Policy": {
          "PolicyName": "EcrPushPullRandserverDevelopers",
          "PolicyId": "ANPAITNBFTFWZMI4WFOY6",
          "Arn": "arn:aws:iam::123456789012:policy/EcrPushPullRandserverDevelopers",
          "Path": "/",
          "DefaultVersionId": "v1",
          "AttachmentCount": 0,
          "PermissionsBoundaryUsageCount": 0,
          "IsAttachable": true,
          "CreateDate": "2018-10-25T12:00:15Z",
          "UpdateDate": "2018-10-25T12:00:15Z"
          }
    }  
```

最后一步是将策略附加到组，以便`johndoe`和这个应用程序的所有未来开发人员可以像我们之前一样从他们的工作站使用存储库：

```
$ aws iam attach-group-policy --group-name randserver-developers --policy-arn arn:aws:iam::123456789012:policy/EcrPushPullRandserverDevelopers 
```

# 在 Kubernetes 中使用存储在 ECR 上的图像

您可能还记得，在第七章中，*生产就绪的集群*，我们将 IAM 策略`AmazonEC2ContainerRegistryReadOnly`附加到了我们集群节点使用的实例配置文件。这允许我们的节点在托管集群的 AWS 帐户中的任何存储库中获取任何图像。

为了以这种方式使用 ECR 存储库，您应该将清单上的 pod 模板的`image`字段设置为指向它，就像以下示例中一样：

```
image: 123456789012.dkr.ecr.eu-central-1.amazonaws.com/randserver:0.0.1.
```

# 给图像打标签

每当将 Docker 图像推送到注册表时，我们需要使用标签标识图像。标签可以是任何字母数字字符串：`latest stable v1.7.3`甚至`c31b1656da70a0b0b683b060187b889c4fd1d958`都是您可能用来标识推送到 ECR 的图像的完全有效的示例标签。

根据软件的开发和版本控制方式，您在此标签中放入的内容可能会有所不同。根据不同类型的应用程序和开发流程，可能会采用三种主要策略来生成图像。

# 版本控制系统（VCS）引用

当您从源代码受版本控制系统管理的软件构建图像时，例如 Git，此时标记图像的最简单方式是利用来自您 VCS 的提交 ID（在使用 Git 时通常称为 SHA）。这为您提供了一个非常简单的方式来确切地检查您的代码当前正在运行的版本。

这种策略通常适用于以增量方式交付小改动的应用程序。您的图像的新版本可能会每天推送多次，并自动部署到测试和类生产环境中。这些应用程序的良好示例是 Web 应用程序和其他以服务方式交付的软件。

通过将提交 ID 通过自动化测试和发布流水线，您可以轻松生成软件确切修订版本的部署清单。

# 语义版本

然而，如果您正在构建用于许多用户使用的容器图像，无论是您组织内的多个用户，还是当您公开发布图像供第三方使用时，这种策略会变得更加繁琐和难以处理。对于这类应用程序，使用具有一定含义的语义版本号可能会有所帮助，帮助依赖于您图像的人决定是否安全地迁移到新版本。

这些图像的常见方案称为**语义化版本**（**SemVer**）。这是由三个由点分隔的单独数字组成的版本号。这些数字被称为**MAJOR**、**MINOR**和**PATCH**版本。语义版本号以`MAJOR.MINOR.PATCH`的形式列出这些数字。当一个数字递增时，右边的不那么重要的数字将被重置为`0`。

这些版本号为下游用户提供了有关新版本可能如何影响兼容性的有用信息：

+   每当实施了一个保持向后兼容的错误修复或安全修复时，`PATCH`版本会递增

+   每当添加一个保持向后兼容的新功能时，`MINOR`版本号会递增

+   任何破坏向后兼容性的更改都应该增加 `MAJOR` 版本号

这很有用，因为镜像的用户知道 `MINOR` 或 `PATCH` 级别的更改不太可能导致任何问题，因此升级到新版本时只需要进行基本测试。但是，如果升级到新的 `MAJOR` 版本，他们应该检查和测试更改的影响，这可能需要更改配置或集成代码。

您可以在 [`semver.org/`](https://semver.org/) 了解更多关于 SemVer 的信息。

# 上游版本号

通常，当我们构建重新打包现有软件的容器镜像时，希望使用打包软件本身的原始版本号。有时，为了对打包软件使用的配置进行版本控制，添加后缀可能会有所帮助。

在较大的组织中，常常会将软件工具与组织特定的默认配置文件打包在一起。您可能会发现将配置文件与软件工具一起进行版本控制很有用。

如果我要为我的组织打包 MySQL 数据库供使用，镜像标签可能看起来像 `8.0.12-c15`，其中 `8.0.12` 是上游 MySQL 版本，`c15` 是我为包含在我的容器镜像中的 MySQL 配置文件创建的版本号。

# 给镜像贴标签

如果您的软件开发和发布流程稍微复杂，您可能会迅速发现自己希望在镜像标签中添加更多关于镜像的语义信息，而不仅仅是简单的版本号。这可能很快变得难以管理，因为每当您想添加一些额外信息时，都需要修改构建和部署工具。

感谢地，Docker 镜像携带标签，可用于存储与镜像相关的任何元数据。

在构建时，可以使用 Dockerfile 中的 `LABEL` 指令为镜像添加标签。`LABEL` 指令接受多个键值对，格式如下：

```
LABEL <key>=<value> <key>=<value> ... 
```

使用此指令，我们可以在镜像上存储任何我们认为有用的任意元数据。由于元数据存储在镜像内部，不像标签那样可以更改。通过使用适当的镜像标签，我们可以发现来自我们版本控制系统的确切修订版，即使镜像已被赋予不透明的标签，如 `latest` 或 `stable`。

如果要在构建时动态设置这些标签，还可以在 Dockerfile 中使用 `ARG` 指令。

让我们看一个使用构建参数设置标签的例子。这是一个示例 Dockerfile：

```
FROM scratch 
ARG SHA  
ARG BEAR=Paddington 
LABEL git-commit=$GIT_COMMIT \ 
      favorite-bear=$BEAR \ 
      marmalade="5 jars" 
```

当我们构建容器时，我们可以使用`--build-arg`标志传递标签的值。当我们想要传递动态值，比如 Git 提交引用时，这是很有用的：

```
docker build --build-arg SHA=`git rev-parse --short HEAD` -t bear . 
```

与 Kubernetes 允许您附加到集群中对象的标签一样，您可以自由地使用任何方案为您的图像添加标签，并保存对您的组织有意义的任何元数据。

开放容器倡议（OCI），一个促进容器运行时和其图像格式标准的组织，已经提出了一套标准标签，可以用来提供有用的元数据，然后可以被其他理解它们的工具使用。如果您决定向您的容器图像添加标签，选择使用这套标签的部分或全部可能是一个很好的起点。

这些标签都以`org.opencontainers.image`为前缀，以便它们不会与您可能已经使用的任何临时标签发生冲突：

+   `* org.opencontainers.image.title`：这应该是图像的可读标题。例如，`Redis`。

+   `org.opencontainers.image.description`：这应该是图像的可读描述。例如，`Redis 是一个开源的键值存储`。

+   `org.opencontainers.image.created`：这应该包含图像创建的日期和时间。它应该按照 RFC 3339 格式。例如，`2018-11-25T22:14:00Z`。

+   `org.opencontainers.image.authors`：这应该包含有关负责此图像的人或组织的联系信息。通常，这可能是电子邮件地址或其他相关联系信息。例如，`Edward Robinson <ed@errm.co.uk>`。

+   `org.opencontainers.image.url`：这应该是一个可以找到有关图像更多信息的 URL。例如，[`github.com/errm/kubegratulations`](https://github.com/errm/kubegratulations)。

+   `org.opencontainers.image.documentation`：这应该是一个可以找到有关图像文档的 URL。例如，[`github.com/errm/kubegratulations/wiki`](https://github.com/errm/kubegratulations/wiki)。

+   `org.opencontainers.image.source`：这应该是一个 URL，可以在其中找到用于构建镜像的源代码。您可以使用它来链接到版本控制存储库上的项目页面，例如 GitHub、GitLab 或 Bitbucket。例如，[`github.com/errm/kubegratulations`](https://github.com/errm/kubegratulations)。

+   `org.opencontainers.image.version`：这可以是打包在此镜像中的软件的语义版本，也可以是您的 VCS 中使用的标签或标记。例如，`1.4.7`。

+   `org.opencontainers.image.revision`：这应该是对您的 VCS 中的修订的引用，例如 Git 提交 SHA。例如，`e2f3bbdf80acd3c96a68ace41a4ac699c203a6a4`。

+   `org.opencontainers.image.vendor`：这应该是分发镜像的组织或个人的名称。例如，**Apache Software Foundation**（**ASF**）。

+   `org.opencontainers.image.licenses`：如果您的镜像包含受特定许可证覆盖的软件，您可以在这里列出它们。您应该使用 SPDX 标识符来引用许可证。您可以在[`spdx.org/licenses/`](https://spdx.org/licenses/)找到完整的列表。例如，`Apache-2.0`。

# 总结

在这一章中，我们学习了如何轻松地在 AWS 上配置 Docker 注册表，以可复制和防错的方式存储我们应用程序的镜像，使用 ECR。

我们发现了如何从我们自己的工作站推送镜像，如何使用 IAM 权限限制对我们镜像的访问，以及如何允许 Kubernetes 直接从 ECR 拉取容器镜像。

您现在应该了解了几种标记镜像的策略，并知道如何向镜像添加附加标签，以存储有关其内容的元数据，并且您已经了解了 Open Container Initiative 镜像规范推荐的标准标签。
