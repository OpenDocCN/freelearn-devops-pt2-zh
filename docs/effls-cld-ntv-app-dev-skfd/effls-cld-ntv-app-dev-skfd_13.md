# 第十章：探索 Skaffold 替代方案、最佳实践和陷阱

在上一章中，您了解了如何使用 GitHub Actions 和 Skaffold 为 Spring Boot 应用程序创建 CI/CD 流水线。在本章中，我们将首先看看市场上提供类似功能的其他工具。我们将了解开发人员在使用 Skaffold 开发云原生 Kubernetes 应用程序时可以遵循的技巧和诀窍。最后，我们将了解开发人员通常可以避免的 Skaffold 陷阱。

在本章中，我们将涵盖以下主要主题：

+   与其他替代方案比较 Skaffold

+   应用 Skaffold 最佳实践

+   避免 Skaffold 陷阱

+   未来路线图

+   最终思考

通过本章结束时，您将对除 Skaffold 之外的工具有扎实的了解，以改善 Kubernetes 的开发者体验。您还将了解可以用于开发工作流程的 Skaffold 最佳实践以及在开发生命周期中可以避免的一些常见陷阱。

# 与其他替代方案比较 Skaffold

在本节中，我们将比较 Skaffold 与其他解决类似问题的替代工具，即改善 Kubernetes 开发者体验的工具。然而，可能存在 Skaffold 不适用的情况，因此在下一节中，我们将研究除 Skaffold 之外的工具，以解决复杂的使用情况。我们还将比较这些 Kubernetes 开发工具与 Skaffold 提供的功能。让我们开始吧！

## Telepresence

**Telepresence** ([`www.telepresence.io/`](https://www.telepresence.io/)) 是由 Ambassador Labs 开发的工具。它是一个 Cloud Native Computing Foundation 的沙盒项目。其目标是改善 Kubernetes 的开发者体验。Telepresence 允许您在本地运行单个服务，同时将该服务连接到远程 Kubernetes 集群。您可以在这里阅读更多信息：[`www.telepresence.io/about/`](https://www.telepresence.io/about/)。

使用 Telepresence，您可以作为集群的一部分在本地开发和调试服务。您不必不断在 Kubernetes 集群中发布和部署新的构件。与 Skaffold 相比，Telepresence 不需要本地 Kubernetes 集群。它在远程集群中运行一个 pod 作为您的应用程序的占位符，并将传入的流量路由到在本地工作站上运行的容器。当开发人员更改应用程序代码时，它将在远程集群中反映出来，而无需部署新的容器。

另一个优点是，您只需要计算资源，即 CPU 和内存，来在本地运行您的服务，因为您直接与远程集群进行交互。此外，您不必设置和运行本地 Kubernetes 集群，比如 minikube 或 Docker Desktop。在您运行五到六个微服务并且您的应用程序必须与它们交互的情况下，这是有帮助的。相比之下，Skaffold 更像是一个打包成一个解决方案，满足您的本地开发需求和 CI/CD 工作流。但假设您的应用程序需要与许多微服务进行交互。在这种情况下，情况就会变得棘手，因为由于资源限制，难以在本地运行所有实例，并且您可能最终会模拟一些可能无法复制实际生产行为的服务。这就是 Telepresence 可以帮助您的地方，它具有从您的笔记本电脑进行远程开发并且资源使用最小化的能力。

它也有一些缺点，比如在 Windows 操作系统上有一些已知的卷挂载问题，并且需要高速网络连接。

## Tilt.dev

**Tilt** ([`tilt.dev/`](https://tilt.dev/)) 是一个用于改善 Kubernetes 开发体验的开源工具。

在 Skaffold 中，我们使用`skaffold.yaml`配置文件来定义、构建和部署；同样，在 Tilt 中，我们使用 Tiltfile 进行配置。Tiltfiles 是用一种称为**Starlark**的 Python 方言编写的。在这里查看 API 参考：[`docs.tilt.dev/api.html`](https://docs.tilt.dev/api.html)。以下是一个 Tiltfile 示例：

```
docker_build('example-html-image', '.')
k8s_yaml('kubernetes.yaml')
k8s_resource('example-html', port_forwards=8000)
```

接下来，您可以运行`tilt run`命令。与 Skaffold 相比，Tilt 不仅是一个 CLI 工具。Tilt 还提供了一个整洁的 UI，您可以在其中查看每个服务的健康状态、它们的构建和运行时日志。虽然 Skaffold 是一个没有供应商支持的开源项目，但 Tilt 的企业版提供了供应商支持。

Tilt 的缺点是你必须熟悉其 Starlark Python 语法语言，这是编写 Tiltfile 所必需的，而 Skaffold 使用 skaffold.yaml 配置文件作为 YAML 语法文件。但如果你使用 Kubernetes 清单，那么这并不是什么新鲜事，大多数开发人员已经熟悉了。

## OpenShift Do

使用 Kubernetes 或 Platform-as-a-Service（PaaS）提供的 OpenShift 等开发应用程序是很困难的，如果你在开发过程中没有使用正确的工具。在 OpenShift 的情况下，它已经有一个 CLI 工具 oc，但不幸的是，它更多地专注于帮助运维人员，对开发人员不太友好。oc CLI 要求你了解与 OpenShift 相关的概念，比如部署和构建配置等，作为开发人员，你可能对这些并不感兴趣。Red Hat 团队意识到了这个问题，并开发了一个新的 CLI 工具叫做 OpenShift Do（odo），它更加针对开发人员。它还有助于改善开发云原生应用程序部署到 Kubernetes 或 OpenShift 时的开发体验。

让我们来看一下它的一些特点，如下所示：

+   更快地开发并加速内部开发循环。

+   使用 odo watch 命令进行实时反馈。如果你曾经使用过 Skaffold，那么它与 skaffold dev 命令非常相似。odo CLI 使用开发人员关注的概念，如项目、应用程序和组件。

+   它是一个完全基于 CLI 客户端的工具。

OpenShift odo 非常特定于 OpenShift 本身；即使文档说它可以与原始的 Kubernetes 发行版一起使用。缺乏文档和实际示例来使用 odo 与 minikube 或其他工具。

## Oketo

Oketo（https://okteto.com/）是一个 CLI 工具，其方法与 Skaffold 完全不同。Oketo 不是在本地工作站自动化你的内部开发循环，而是将内部开发循环移到集群本身。你可以在一个 YAML 清单文件 oketo.yaml 中定义你的开发环境，然后使用 oketo init 和 oketo up 来启动你的开发环境。

以下是 Oketo 一些突出的特点：

+   本地和远程 Kubernetes 集群之间的文件同步。

+   一个二进制文件可以在不同的操作系统上运行。

+   您可以在容器开发环境中获得远程终端。

+   热重载您的代码。

+   与本地和远程 Kubernetes 集群、Helm 和无服务器函数一起使用。

+   双向端口转发。

+   在开发过程中，不需要构建、推送或部署，因为您直接在集群上开发。

+   无需在工作站上安装 Docker 或 Kubernetes。

+   甚至运行时（JRE、npm、Python）也不需要，因为一切都在 Docker 镜像中。

## Garden

**Garden** ([`garden.io/`](https://garden.io/))遵循与 Oketo 相同的哲学，即部署到远程集群而不是在本地系统上进行设置。Garden 是一个开源工具，用于在远程集群中运行 Kubernetes 应用程序，用于开发、自动化测试、手动测试和审查。

使用 Garden，您可以通过诸如`garden create project`的 CLI 辅助命令开始。您将通过一个 YAML 配置文件管理 Garden 配置。

以下是 Garden YAML 配置文件的关键元素：

+   **模块**：在模块中，您指定如何构建您的容器。

+   **服务**：在服务中，您指定如何在 Kubernetes 上运行您的容器。

+   **测试**：在测试中，您指定单元测试和集成测试。

以下是 Garden 提供的一些功能：

+   当源代码更改时，它会自动将应用程序重新部署到远程集群。

+   它支持多模块和多服务操作（依赖关系树）。

+   它为依赖项提供了一个图形化的仪表板。

+   运行任务的能力（例如，在构建流程中作为数据库迁移的一部分）。

+   它支持 Helm 和 OpenFass 部署。

+   它支持热重载功能，其中源代码直接发送到正在运行的容器。

+   将容器日志流式传输到您的终端的能力。

+   它支持文件监视和远程集群代码的热重载。

Garden 的设置比 Skaffold 更复杂，需要一段时间才能熟悉其概念，因此涉及陡峭的学习曲线。使用 Skaffold，您可以使用熟悉的构建和部署工具，并且很容易开始使用。对于小型应用程序来说，使用 Garden 可能也是过度复杂的。Garden 是商业开源的，因此与完全开源的 Skaffold 相比，它的一些功能是付费的。

## docker-compose

`docker-compose` 是一个主要用于本地容器开发的工具。它允许您在本地运行多个容器，并模拟应用程序在部署到 Kubernetes 时的外观。要开始使用它，需要在工作站上安装 Docker。虽然 `docker-compose` 可能会让一些开发人员错误地认为他们的应用程序在 Kubernetes 环境（如 minikube）中运行，但实际上，它与在 Kubernetes 集群中运行完全不同。这也意味着，因为您的应用程序在 `docker-compose` 上运行正常，当部署到生产环境的 Kubernetes 集群时，它将无法正常工作或表现类似。虽然我们知道容器解决了“在我的机器上运行”的问题，但是使用 `docker-compose`，我们引入了一个新问题，即“在我的 docker-compose 设置上运行”。

也许会诱人将 `docker-compose` 用作云应用程序内部开发循环的替代品，但正如前面所解释的，您的本地和生产环境将不相同。由于这种差异，很难调试任何环境，而使用 Skaffold，您可以在本地和远程构建和部署时使用完全相同的堆栈。如果由于某种原因您被困在 `docker-compose` 设置中，甚至可以将其与 Skaffold 配对使用。Skaffold 内部使用 Kompose ([`kompose.io/`](https://kompose.io/)) 将 `docker-compose.yaml` 转换为 Kubernetes 清单。

您可以使用以下命令将现有的 `docker-compose.yaml` 文件与 Skaffold 一起使用：

```
skaffold init --compose-file docker-compose.yml 
```

在本节中，我们已经了解了除 Skaffold 之外的 Kubernetes 开发工具，帮助开发人员在内部开发循环中更快地开发并获得快速反馈。

在接下来的部分中，我们将学习如何将一些最佳实践应用到我们现有或新的 Skaffold 工作流中。

# 应用 Skaffold 最佳实践

在本节中，我们将了解 Skaffold 的最佳实践，作为开发人员，您可以利用这些最佳实践，加快内部或外部开发循环中的部署速度，或者在使用 Skaffold 时使用一些标志来简化事情。让我们开始：

+   在部署到 Kubernetes 的多个微服务应用程序中工作时，有时很难为每个应用程序创建单个的`skaffold.yaml`配置文件。在这些常见情况下，您可以为每个应用程序创建范围为`skaffold.yaml`，然后独立运行`skaffold dev`或`run`命令。您甚至可以在单个 Skaffold 会话中同时迭代这两个应用程序。假设我们有一个前端应用和一个后端应用，对于它们两个，您的单个`skaffold.yaml`文件应如下所示：

```
apiVersion: skaffold/v2beta18
kind: Config
requires:
- path: ./front-end-app
- path: ./backend-app
```

当您使用 Skaffold 引导项目并且没有 Kubernetes 清单时，您可以使用`skaffold init`命令传递`--generate-manifests`标志来为项目生成基本的 Kubernetes 清单。

+   最好始终与 Skaffold 一起使用`default-repo`功能。如果您使用`default-repo`，则无需手动编辑 YAML 文件，因为 Skaffold 可以使用您指定的容器镜像注册表前缀图像名称。因此，您可以在`skaffold.yaml`配置文件中输入图像名称，而不是编写`gcr.io/myproject/imagename`。另一个优点是，您可以轻松地与其他团队共享您的`skaffold.yaml`文件，因为如果他们使用不同的容器镜像注册表，则无需手动编辑 YAML 文件。因此，基本上，您可以使用`default-repo`功能，而无需在`skaffold.yaml`配置文件中硬编码容器镜像注册表名称。

您可以以以下三种方式利用`default-repo`功能：

a. 通过传递`--default-repo`标志：

```
skaffold dev --default-repo gcr.io/myproject/imagename 
```

b. 通过传递`SKAFFOLD_DEFAULT_REPO`环境变量：

```
SKAFFOLD_DEFAULT_REPO= gcr.io/myproject/imagename skaffold dev
```

c. 通过设置 Skaffold 的全局配置：

```
skaffold config set default-repo gcr.io/myproject
/imagename
```

+   当您遇到 Skaffold 命令的问题时，了解实际问题变得棘手。在某些情况下，您可能需要比 Skaffold 通常显示的流式日志更多的信息。对于这种情况，您可以使用`-v`或`-verbosity`标志使用特定的日志级别。例如，您可以使用`skaffold dev -v info`来查看信息级别的日志。

Skaffold 支持以下日志级别，默认为`warn`：

+   `info`

+   `warn`

+   `error`

+   `fatal`

+   `debug`

+   `trace`

+   为了更快地构建，您可以通过将并发标志设置为`0`来利用并发标志。默认值为`0`，表示没有限制的并行构建，因此所有构建都是并行进行的。只有在本地构建并发性方面，该值才会默认为`1`，这意味着构建将按顺序进行，以避免任何副作用。

+   如果您正在使用 Jib 来构建和推送容器镜像，那么您可以使用自动配置来使用特殊的同步支持。您可以通过使用`sync:`选项来启用它，如下面的`skaffold.yaml`文件中所述：

```
apiVersion: skaffold/v2beta16
kind: Config
build: 
  artifacts: 
    - 
      image: file-sync
      jib: {}
      sync: 
        auto: true
deploy: 
  kubectl: 
    manifests: 
      - k8s-*
```

使用此选项，Jib 可以将您的类文件、资源文件和 Jib 的*额外目录*文件同步到本地或远程运行的容器中。您不必为内部开发循环中的每次更改重新构建、重新部署或重新启动 pod。但是，要使其与您的 Spring Boot 应用程序配合使用，您需要在 Maven 项目的`pom.xml`文件中具有`spring-boot-devtools`依赖项。

+   Skaffold 还支持 Cloud Native Buildpacks 来构建您的容器镜像，并且与 Jib 类似，它还支持`sync`选项，以在对某种类型的文件进行更改时自动重新构建和重新启动应用程序。它支持以下类型的源文件：

+   Go: `*.go`

+   Java: `*.java`, `*.kt`, `*.scala`, `*.groovy`, `*.clj`

+   Node.js: `*.js`, `*.mjs`, `*.coffee`, `*.litcoffee`, `*.json`

在本节中，我们已经学习了一些最佳实践，我们可以应用这些最佳实践来高效开发，并且甚至可以更快地加速 Skaffold 的开发循环。

在下一节中，我们将看一些我们作为开发人员应该注意的常见 Skaffold 陷阱。

# 避免 Skaffold 陷阱

在本书中，我们已经使用了 Skaffold 提供的各种功能。现在让我们讨论一些常见的 Skaffold 陷阱，作为开发人员，我们应该了解并尽量避免：

+   Skaffold 要求您具有一些本地或远程 Kubernetes 设置，因此与我们在上一节中讨论的其他工具相比，Skaffold 并不会减少设置开发环境所需的时间。

+   在大多数情况下，使用 Skaffold 时，您会使用一些本地 Kubernetes，如 minikube 或 Docker Desktop，并且由于它们的限制，您无法复制整个类似生产环境的设置。这留下了集成问题的空间，这些问题在本地系统上可能看不到，但可能会在更高的环境中出现。

+   有时，使用 Skaffold 会在您的机器上浪费更多的硬件资源。例如，如果您需要运行，比如说，10 个微服务，那么这将变得具有挑战性，因为您的笔记本电脑资源有限。

+   Skaffold 内置支持通过（beta）`skaffold debug`命令与调试器集成。使用这个调试选项，Skaffold 会自动配置应用程序运行时进行远程调试。这是一个很棒的功能，但在微服务环境中使用调试器最好是棘手的。在远程集群中使用会更加困难。要谨慎使用。

+   Skaffold 没有 Web UI。虽然在上一节中我们讨论了许多提供 UI 以获得更好体验的工具，但我不会为此而哭泣。这更多是个人偏好，因为有些人倾向于喜欢 UI，而有些人则喜欢 CLI。如果你更喜欢 UI，那么你可能无法与 Skaffold 相处。

+   Skaffold 非常适合在本地开发和测试中使用。尽管它被宣传为一些用例的完整 CI/CD 解决方案，但它可能不是最适合的工具。例如，如果我们想要扩展到生产或预生产用例，最好使用专门的工具，比如**Spinnaker pipelines** ([`spinnaker.io/`](https://spinnaker.io/)) 或 **Argo Rollouts** ([`argoproj.github.io/argo-rollouts/`](https://argoproj.github.io/argo-rollouts/))。这些工具 Spinnaker/Agro Rollouts 相对于 Skaffold 提供了一些优势。让我们来看看它们：

1.  在 Spinnaker/Agro Rollouts 的情况下，两者都支持复杂的部署策略。您可以定义诸如金丝雀部署和蓝绿部署之类的部署策略。

1.  Spinnaker 允许多集群部署。此外，您可以配置基于易用 UI 的部署到多个集群。

1.  Spinnaker 具有出色的可视化效果。它提供了一个丰富的 UI，显示跨集群、区域、命名空间和云提供商的任何部署或 Pod 状态。

在这一部分，我们已经涵盖了 Skaffold 的一些陷阱，在决定是否继续在 Kubernetes 工作负载中使用之前，你应该注意这些。

# 未来路线图

由于 Skaffold 是一个开源工具，社区主要推动了 Skaffold 的路线图，而来自 Google 的工程师团队做出最终决定。Google 开发人员还提出了一些令人兴奋的新功能，这些功能将增强 Skaffold 的用户体验，除了社区提出的变更。

然而，路线图不应被视为无论如何都会实现的承诺清单。这是 Skaffold 工程团队认为值得投入时间的愿望清单。路线图背后的主要动机是从社区中获取他们希望在 Skaffold 中看到的功能的反馈。

您可以通过访问[`github.com/GoogleContainerTools/skaffold/blob/master/ROADMAP.md#2021-roadmap`](https://github.com/GoogleContainerTools/skaffold/blob/master/ROADMAP.md#2021-roadmap) URL 来查看 2021 年的 Skaffold 路线图。

# 最后的思考

近年来，围绕 Kubernetes 开发工具的工具化显著改善。这背后的主要动机是 Kubernetes 在行业中的日益普及。现代开发人员希望有一个能够提高他们在为云开发应用程序时的生产力的工具。Skaffold 极大地提高了开发人员构建和部署 Kubernetes 应用程序的生产力。

许多工具内部使用 Skaffold，例如 Jenkins X 和 Cloud Code，以改善 Kubernetes 的整体开发体验。与 Jenkins X 不同，后者在流水线中使用 Skaffold 构建和推送镜像，Cloud Code 完全围绕 Skaffold 及其支持的工具构建，以为 Kubernetes 应用程序提供无缝的入门体验。

最后，我想总结一下，Skaffold 简化了 Kubernetes 开发，在我看来，它做得很好。它提供了灵活性和可扩展性，可以选择与之一起使用的集成类型。其可扩展和可插拔的架构允许开发人员为构建和部署应用程序的每个步骤选择适当的工具。

# 总结

在本章中，我们首先将 Skaffold 与其他工具（如 Tilt、Telepresence、Garden、Oketo、`docker-compose`和 OpenShift odo）进行了比较。这些工具原则上试图提供与 Skaffold 解决的类似问题的解决方案。然后，我们比较了这些工具相对于 Skaffold 提供的功能。我们还研究了一些可以与 Skaffold 一起使用以获得更好开发体验的最佳实践。最后，我们通过解释与 Skaffold 相关的一些潜在问题来总结，如果您的用例更高级，您应该注意这些问题。

你已经发现了如何通过遵循我们试图解释的一些最佳实践来利用 Skaffold。现在你更有能力决定 Skaffold 是否满足你的使用情况，或者你是否需要考虑本章中涵盖的其他选项。

通过这一点，我们已经到达了旅程的尽头，我希望你会被鼓励尝试并探索大量的 Skaffold！
