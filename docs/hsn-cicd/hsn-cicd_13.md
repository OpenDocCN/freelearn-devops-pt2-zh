# CircleCI CLI 命令和自动化

在上一章中，我们介绍了如何在 Bitbucket 和 GitHub 中设置使用 CircleCI，并向您展示了如何导航 Bitbucket UI，并介绍了 CircleCI Web UI 的基础知识。在本章中，我们将介绍如何在 macOS/Linux 上安装 CircleCI CLI，并向您展示如何从 CLI 获取夜间构建。我们将详细介绍每个 CircleCI CLI 命令，并解释 CircleCI 中的工作流程。我们将向您展示如何使用顺序作业设置更复杂的工作流程。最后，我们将介绍 CircleCI API，并向您展示如何在使用 HTTP 请求时使用`jq` JSON 命令实用程序转换 JSON。

在本章中，我们将涵盖以下主题：

+   CircleCI CLI 安装

+   CircleCI CLI 命令

+   在 CircleCI 中使用工作流

+   使用 CircleCI API

# 技术要求

本章将需要一些基本的 Unix 编程技能，并且我们将在前几章中讨论的**持续集成**（**CI**）和**持续交付**（**CD**）概念上进行一些构建。熟悉使用 RESTful API 可能会有所帮助，因为我们将在本章末尾使用 curl 作为 REST 客户端。

# CircleCI CLI 安装

安装 CircleCI CLI 的第一个先决条件是已安装 Docker ([`docs.docker.com/install/`](https://docs.docker.com/install/))。要在您的操作系统上安装 Docker，请访问 Docker 商店[`store.docker.com/search?type=edition&offering=community`](https://store.docker.com/search?type=edition&offering=community)，并单击适合您的操作系统或云服务的**Docker CE**链接。按照其网站上的安装说明进行安装。

通过在 Windows 命令提示符或 macOS/Linux 终端应用程序上运行类似以下命令的命令来确保已安装 Docker 版本：

![](img/2e5d2e3e-f37b-4356-927d-2c001b684e2b.png)

这里我安装了 Docker 版本 18。

# 在 macOS/Linux 上安装 CircleCI CLI

您需要运行以下命令来安装 CircleCI：

```
curl -o /usr/local/bin/circleci https://circle-downloads.s3.amazonaws.com/releases/build_agent_wrapper/circleci && chmod +x /usr/local/bin/circleci
```

您需要在终端应用程序 shell 会话中运行此命令。

# 通过 GitHub 发布安装夜间版本的 CircleCI

您可以在 GitHub 发布页面安装 CircleCI CLI 的夜间版本：[`github.com/CircleCI-Public/circleci-cli/releases`](https://github.com/CircleCI-Public/circleci-cli/releases)。您需要查看类似于以下内容的“Assets”部分：

![](img/67e6728d-ee02-414c-a6b7-a3c8eb22e1f3.png)

我们将选择`circleci-cli_0.1.771_darwin_amd64.tar.gz`资产，因为我们将在 macOS 操作系统上运行本地 CLI。

在终端 shell 会话中运行以下命令：

```
# Go to the Downloads Folder
cd ~/Downloads

# Unpack the compressed asset
tar -xvzf circleci-cli_0.1.771_darwin_amd64.tar.gz

# Go into the uncompressed directory
cd circleci-cli_0.1.771_darwin_amd64

# Move the circleci binary into the folder /usr/local/bin
mv circleci /usr/local/bin/circleci-beta

# Make sure that the binary is executable
chmod +x /usr/local/bin/circleci-beta

# Check that the binary version to make sure that it is working
circleci-beta help
```

我们现在有一个更新版本的 CircleCI CLI，并且可以验证：

![](img/b9cf3e70-6862-46b8-bfc7-26d2374f5d68.png)

我们将这个二进制可执行文件命名为`circleci-beta`。这样我们就可以运行稳定版和夜间版本的 CircleCI CLI。这不是您必须做的事情；我们只是为了举例说明而这样做。

# CircleCI CLI 命令

就功能对等性而言，CircleCI CLI 并不像 Travis CI CLI 那样功能齐全。未来将会有更多的命令可用，但目前您可以在 CircleCI CLI 中使用六个命令，它们是`build`、`config`、`help`、`step`、`tests`和`version`，如果您从官方 CircleCI 文档中的 AWS 发布的 CircleCI CLI 二进制文件中使用([`circle-downloads.s3.amazonaws.com/releases/build_agent_wrapper/circleci`](https://circle-downloads.s3.amazonaws.com/releases/build_agent_wrapper/circleci))。我们将同时使用稳定版本和夜间构建版本，后者比稳定版本多了几个命令。请记住，我们在本章的*通过 GitHub 发布安装 CircleCI 的夜间构建版本*部分中安装了它。稳定版本的命令将是`circleci`，夜间构建将是`circleci-beta`。

在下面的屏幕截图中，我们运行了`help`命令，显示了可用的命令，并简要概述了每个命令的功能：

![](img/b9b84382-b46d-4674-ab85-9d64b56173fb.png)

# 版本命令

`version`命令输出您在本地系统上安装的 CLI 的当前版本：

![](img/de730070-ae1c-42ac-8b74-1d24e0ef3032.png)

您还可以向 CLI 中的每个命令传递标志/选项，并且可以通过运行`--help`标志找到每个命令接受的选项：

![](img/42c7f35c-48e2-48bd-998a-fcd9188d6ea0.png)

我们可以向`version`命令传递的选项只有一个，即`-h`，`--help`，因为这是一个非常简单的命令。

# 帮助命令

`help`命令将显示所有 CLI 命令，就像我们在本节开头演示的那样，但它也可以用来解释每个命令的工作原理，并显示每个命令接受的任何标志/选项：

![](img/fc870348-c0c8-412d-9fd4-a62c926e929e.png)

在这里，我们对`help`命令本身运行了帮助。

# 配置命令

`config`命令验证并更新 CircleCI 配置 YML 脚本：

![](img/14bdd3d0-5df0-4e41-aacf-aaa0b0d555c1.png)

这里`config`命令还接受`validate`命令，用于验证您的配置 YML 脚本文件。

让我们验证`functional-summer`存储库中的配置脚本([`github.com/packtci/functional-summer`](https://github.com/packtci/functional-summer))：

![](img/c4d99005-c7f1-4ce5-94b5-115781f8e0fd.png)

让我们再次查看配置脚本：

```
version: 2 jobs: build: docker: # specify the version you desire here - image: circleci/node:7.10
  working_directory: ~/repo steps: - checkout - restore_cache: keys: - v1-dependencies-{{ checksum "package.json" }} - v1-dependencies- - run: yarn install - save_cache: paths: - node_modules key: v1-dependencies-{{ checksum "package.json" }} # run tests! - run: yarn test
```

这实际上是配置 YML 脚本中的一个非常微妙的错误，我们只需要缩进`build`字段，因为 CircleCI 认为我们的脚本中没有任何作业。为了解决这个问题，我们只需要缩进`build`字段：

```
version: 2
jobs:
    build:
        ...
```

![](img/57677a87-6dc3-49e9-8113-0de69a9c0863.png)

当我们运行`validate`命令时，它报告说配置 YML 脚本是有效的。

# 构建命令

`build`命令帮助您在本地计算机上运行 CircleCI 构建，并且可以采用各种选项，如下面的屏幕截图所示：

![](img/352fae0a-159c-40e6-b69d-be17fc023ba7.png)

让我们运行我们在第十二章中创建的`go-template-example-with-circle-ci` ([`github.com/packtci/go-template-example-with-circle-ci`](https://github.com/packtci/go-template-example-with-circle-ci))GitHub 存储库，然后在本地系统上运行`circleci build`命令。

在运行构建命令之前，确保进入存储库所在的目录，因为它需要读取`.circleci`文件夹中的`config.yml`文件：

![](img/d0cbff8c-3ea5-4b78-9047-030e15dec87b.png)

`build`命令将执行配置 YML 脚本中的步骤，首先会启动一个环境。如果您尚未拉取配置 YML 脚本中指定的语言映像，则`circleci build`命令将为您拉取 Docker 映像。

默认情况下，`circleci build`命令将运行在`jobs`部分的`build`字段中定义的步骤，因此如果您想运行其他作业，就需要传递`--job string`选项。

这是我们在`go-template-example-with-circle-ci` GitHub 项目中的当前`config.yml`脚本：

```
version: 2 jobs: build: docker: - image: circleci/golang:1.9 working_directory: /go/src/github.com/packtci/go-template-example-with-circle-ci steps: - checkout - run: name: "Print go version" command: go version - run: name: "Run Unit Tests" command: go test
```

如果我们想使用另一个作业，可以使用`--job string`选项，假设有另一个作业：

```
...
    build:
        ...
    integration:
       docker:
            - image: cypress/base:8
                environment:
                    TERM: xterm
       steps:
            - checkout
            - run: npm install
            - run:
                name: "Run Integration Tests"
                command: $(npm bin)/cypress run
```

现在让我们验证我们的 config YML 脚本，以确保它仍然有效：

![](img/3cb68165-ad8a-4945-9822-1dce13ec5dd7.png)

现在我们知道我们的 config YML 脚本仍然有效，我们可以使用`--job string`标志运行新作业。

![](img/51212ff6-1664-4cd8-a934-c4ce2bef6c02.png)

在这里，CLI 正在下载 Docker 映像，因为我们尚未将此特定 Docker 映像拉入我们的本地计算机。

# 步骤命令

`step`命令将执行您定义的配置 YML 脚本中的特定步骤。目前，只有一个`halt`的子命令，它将停止当前执行。

这是`step`命令的一个示例运行：

```
circleci step halt
```

# 配置命令

`configure`命令仅在 CircleCI 的夜间构建版本中可用，它可以帮助您配置您的凭据和将要访问的 API 端点：

![](img/8029ac68-e023-49fb-9ae3-e6cb97c53c96.png)

我们将以无标志运行`configure`命令，这将将其设置为交互模式，然后我们将设置我们的 API 令牌和我们希望访问的 API 端点。

# 在 CircleCI 中设置 API 令牌

您需要点击 CircleCI Web 应用程序右上角的用户头像，它看起来像以下截图：

![](img/b52aeda5-fd1e-4896-b976-68edc24f0cf5.png)

一旦您点击“用户设置”链接，您将被重定向到账户 API 页面，它看起来像这样：

![](img/7e9b20cb-f2ed-4f3b-9aa7-477a40725725.png)

接下来，您需要点击“创建新令牌”按钮，这将弹出一个类似于这样的模态框：

![](img/c710dd6e-0d34-4470-8940-8044863841c4.png)

在这里，我们输入了一个名为`PacktCI`的令牌名称。然后我们只需点击“添加 API 令牌”按钮，这将为我们生成一个新的 API 令牌。您需要将 API 令牌复制到安全位置，因为您只能使用一次。

# 在交互模式下设置 API 令牌和 API 端点

我们将在终端会话中运行`circleci-beta configure`命令，并设置我们的凭据和 API 端点：

![](img/c3d3c273-d75e-424f-a810-3586cbf79dc5.png)

在这里，我们设置了 API 令牌，但出于安全目的，该值被隐藏，我们将 API 端点设置为`https://circleci.com/api/v1.1/`。

`configure`命令仅在夜间版本中可用，而不是稳定版本。

# 测试命令

`tests`命令收集并拆分具有测试的文件：

![](img/976efbd2-7629-4cbd-8165-808896377642.png)

让我们使用`glob`子命令在`go-template-example-with-circle-ci`（[`github.com/packtci/go-template-example-with-circle-ci`](https://github.com/packtci/go-template-example-with-circle-ci)）GitHub 存储库中查找所有 Go 测试文件：

![](img/a1396e90-838a-4cf3-ac36-01701e16b79b.png)

# 在 CircleCI 中使用工作流程

CircleCI 中的工作流程是一种运行并行`build`作业的方式，可以用来定义一组作业并指定作业顺序。让我们在`go-template-example-with-circle-ci`（[`github.com/packtci/go-template-example-with-circle-ci`](https://github.com/packtci/go-template-example-with-circle-ci)）配置 YML 脚本中添加一个工作流字段：

```
version: 2
jobs:
    build:
        ...
    integration:
        ....
workflows:
    version: 2
    build_and_integration:
        jobs:
            - build
            - integration
```

在这个工作流程中，我们分别创建了两个并行作业，分别称为`build`和`integration`。它们彼此独立，这将有助于加快构建过程。

# CircleCI Web UI 中的工作流程

如果我们点击左侧导航窗格中的“工作流程”链接，我们可以在 CircleCI Web UI 中看到工作流程。然后您需要点击特定的项目，本例中是`go-template-example-with-circle-ci`，如下截图所示：

![](img/a9b667b7-79ee-4b5b-bf55-01c74a56b22c.png)

如果您点击“RUNNING”工作流程，您将看到以下页面：

![](img/97bbc387-2a2f-4664-bbc4-83b686d00cb5.png)

构建作业运行了 2 秒，但集成测试运行的时间比构建作业长。最好将这两个作业分开，因为工作流程表明它们彼此不依赖。

# 顺序工作流程示例

我们之前展示的工作流程示例包含了两个独立运行的作业，但我们也可以有需要其他作业完成后才能运行的作业。假设我们有一个只有在构建运行时才运行的验收测试套件，然后我们的应用程序只有在验收测试套件通过后才会部署。

在我们的示例中，我们使用`cypress.io`（[`www.cypress.io/`](https://www.cypress.io/)）运行端到端测试，这是一个端到端的 JavaScript 测试库。假设我们的验收测试在 CI 构建中通过，我们就可以将我们的应用程序部署到 Heroku。我们在第十一章中介绍了如何在 Travis CI 中设置 Heroku，*Travis CI UI Logging and Debugging*部分，所以如果您需要更多关于安装和设置 Heroku 以及在 Heroku 中创建可以部署的应用程序的信息，请阅读那部分。我们需要将 Heroku API 密钥和应用程序名称添加为环境变量。

# 向项目添加环境变量

在我们的 CircleCI 项目中，我们首先需要通过点击`go-template-example-with-circle-ci`（[`circleci.com/gh/packtci/go-template-example-with-circle-ci`](https://circleci.com/gh/packtci/go-template-example-with-circle-ci)）项目旁边的齿轮图标进入项目设置。确保您在“作业”或“工作流程”视图中，然后您应该看到一个齿轮图标：

![](img/66fc9596-b687-45d3-ad52-a4c1640dacc5.png)

一旦您点击齿轮图标，您将被重定向到“项目设置”页面，您需要点击“环境变量”链接。然后您的页面将看起来像以下截图：

![](img/adc3a7b8-517b-4dd3-ad0d-e7826c795c22.png)

我们将通过点击“添加变量”按钮向我们的项目添加两个环境变量，这将弹出一个如下的模态框：

![](img/9befa423-c161-4e90-92a7-4c87e656a223.png)

出于安全目的，我已经删除了项目的应用程序名称和 API 令牌的内容，但是一旦您点击“添加变量”按钮，项目中就会有一个环境变量可用。我们现在有两个可以使用的环境变量，即`HEROKU_API_KEY`和`HEROKU_APP_NAME`。这些环境变量将在我们的`.circleci/config.yml`脚本中可用。

# 更新了工作流程部分和配置 YML 脚本

我们的配置 YML 脚本现在有一个部署`jobs`部分，并且我们已经更新了我们的工作流程字段如下：

```
...
deploy:
    docker:
        - image: buildpack-deps:trusty
    steps:
        - checkout
        - run:
            name: Deploy Master to Heroku
            command: |
                git push https://heroku:$HEROKU_API_KEY@git.heroku.com/$HEROKU_APP_NAME.git master

workflows:
    version: 2
        build_integration_and_deploy:
            jobs:
                - build
                - integration:
                    requires:
                        - build
                - deploy:
                    requires:
                        -integration
```

这次更改的工作流程现在看起来不同，因为我们为作业设置了一个顺序管道：

![](img/02303299-1a33-42ee-acbf-d6477921db32.png)

在前面的截图中，构建作业首先运行，然后是集成作业，最后是部署作业。阅读[`circleci.com/docs/2.0/workflows/`](https://circleci.com/docs/2.0/workflows/)上的工作流程文档，了解更多类型的工作流程信息。

# 使用 CircleCI API

CircleCI API 文档可在[`circleci.com/docs/api/v1-reference/`](https://circleci.com/docs/api/v1-reference/)上找到。要开始使用 API，您需要添加一个 API 令牌。我们已经在本章的*使用 CircleCI 设置 API 令牌*部分设置了一个 API 令牌，因此如有必要，请阅读该部分。

# 测试 CircleCI API 连接

我们将使用`curl`命令和我们的 API 令牌来测试我们是否有一个良好的 CircleCI API 连接：

![](img/ec44ad31-573a-413f-bcf9-74d25ceb84c1.png)

在这里，我们没有得到任何响应头或状态码。为了接收这些，您需要在`curl`命令中使用`-i`、`--include`选项。

# 使用 CircleCI API 获取单个 Git 存储库的构建摘要

我们将使用`GET /project/:vcs-type/:username/:project` API 端点来获取构建摘要信息。您可以在[`circleci.com/docs/api/v1-reference/#recent-builds-project`](https://circleci.com/docs/api/v1-reference/#recent-builds-project)上阅读单个项目的最近构建的文档。

在下面的截图中，我们使用`curl`命令进行 REST 调用，并使用`jq`（[`stedolan.github.io/jq/`](https://stedolan.github.io/jq/)）JSON 命令行处理器来美化 JSON 输出，如下面的截图所示：

![](img/c394b673-bb14-4445-9d0b-cca3ef7d8d9c.png)

# 使用 jq 实用程序计算我们的 CircleCI 构建的一些指标

让我们使用`jq`命令行实用程序来计算 CircleCI API 提供的信息的一些指标。我们可能想要找到的一个是项目中所有已通过的构建。我们可以使用`jq`命令通过使用 jq 中的`map`和`select`内置函数来实现这一点（[`stedolan.github.io/jq/manual/#Builtinoperatorsandfunctions`](https://stedolan.github.io/jq/manual/#Builtinoperatorsandfunctions)）。

在下面的截图中，我们获取了最近 30 次构建的构建摘要，然后只显示实际通过的构建：

![](img/9b06e7b2-40d1-4e8c-8d41-cc1e3f0f6729.png)

在这里，我们使用`jq`实用程序进行了两个不同的查询。

+   第一个查询是`jq 'map(select(.failed == false)) | length'`，它对对象数组进行映射，并在`failed`为`false`时过滤掉顶级属性`failed`。

+   第二个查询是`jq '. | length'`，它只是计算数组的长度，即`5`。

我们运行了第二个命令，以确保第一个命令确实过滤了响应有效负载中的一些条目。从中我们可以得知，在最近的 30 次构建中，`go-template-example-with-circle-ci`（[`github.com/packtci/go-template-example-with-circle-ci`](https://github.com/packtci/go-template-example-with-circle-ci)）GitHub 存储库中有一次构建失败。

# 摘要

在本章中，我们介绍了如何在 macOS/Linux 环境中安装 CircleCI CLI，并向您展示了如何安装 CLI 的夜间构建。我们向您展示了如何使用 CircleCI CLI 中的每个命令，并向您展示了 CircleCI CLI 夜间构建中可用的一些命令功能。我们解释了工作流程为什么有用以及如何在 CircleCI 中使用它们。最后，我们向您展示了如何使用 CircleCI API 以及如何使用`jq`命令实用程序收集有用的指标。

# 问题

1.  安装 CircleCI CLI 的主要先决条件是什么？

1.  我们从哪里获取了 CircleCI CLI 的夜间构建？

1.  CLI 中有多少个命令存在？

1.  CLI 中的哪个命令对于了解特定命令的功能以及给定命令的选项很有用？

1.  我们如何在 CircleCI 中运行并行作业？

1.  我们使用哪个命令来验证我们的 CircleCI YML 脚本？

1.  CircleCI RESTful API 的端点是什么？

# 进一步阅读

您可以通过查看[`circleci.com/docs/2.0/`](https://circleci.com/docs/2.0/)上的官方 CircleCI 文档进一步探索 CircleCI 中的概念。
