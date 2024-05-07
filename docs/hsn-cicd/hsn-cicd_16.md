# 第十六章：评估

# 第一章：自动化测试的持续集成/持续交付

1.  手动流程是任何重复且可以自动化的流程。

1.  自动化是一种过程，通过脚本或某种操作使操作自动完成。

1.  打开部门之间的沟通是为了找到手动流程。

1.  持续集成/持续交付

1.  自动化脚本很有用，因为它们有助于自动化手动任务。

1.  公司内部网可以帮助其他部门共享信息并链接断开的信息。

1.  其他部门应该共享数据，因为这有助于在部门之间桥接信息，并增加使用数据的可能性。

# 第二章：持续集成的基础知识

1.  软件构建可以仅包括编译软件组件。构建可以包括编译和运行自动化测试，但一般来说，您在构建中添加的进程越多，反馈循环就会变得越慢。

1.  分阶段构建是将构建分解为较小构建的构建。例如，在第一个构建中，您可以进行编译步骤并运行所有单元测试。可以使用第二个构建来运行更长时间的测试，例如端到端测试。

1.  Make 是一种广泛使用的脚本工具，可用于许多不同类型的编程语言。Maven 是 Java 社区使用的脚本工具。

1.  最好遵循命名约定，因为它有助于更好地组织代码库，并帮助开发人员快速了解源文件的情况。遵循特定的文件夹结构可以帮助您快速设置新项目。

1.  CI 有很多有价值的东西，但特别是 CI 系统有助于将环境配置和设置解耦到一个隔离的环境中，开发人员可以在其中运行代码库中的所有测试，并执行重要任务，如报告和调用其他第三方服务。

# 第三章：持续交付的基础知识

1.  我们所说的交付软件是指实际的软件产品已交付给预期的用户，而不仅仅是软件产品已经获得 QA 部门的批准。换句话说，预期的用户实际上正在使用软件。

1.  手动部署软件是一种反模式，以及手动软件配置。

1.  在交付软件时自动化的一些好处包括团队赋权（团队感到有权做决定）、减少错误（通过消除由手动流程引起的错误）以及减轻压力。

1.  配置管理是指检索、存储、识别和修改与每个给定项目相关的所有软件工件以及软件工件之间的任何关系的过程。

1.  撰写描述性和有意义的提交消息有助于开发人员快速跟踪正在处理的问题，并帮助开发人员了解您实际完成的工作。

1.  部署管道可以被视为将开发人员编写的软件交付给用户手中的过程。

1.  部署应该在每个环境中都是相同的，这样您就可以可靠地知道它在每个环境中都是相同的测试，并避免可能的配置不匹配。

# 第四章：持续集成/持续交付的商业价值

1.  开发人员在没有所有要求的情况下很难开发新功能。没有所有必要的要求，开发人员完成分配的工作的能力会受到很大阻碍。

1.  痛苦驱动开发是关于改进导致您痛苦的流程。主要观点是您感受到的痛苦将帮助您指出改进的方向。

1.  如果开发人员被大量警报轰炸，他们最终会忽略消息。最好的方法是警报有意义，而不仅仅是噪音。

1.  通过将团队成员轮换到不同的团队，您可以帮助塑造他们的视角，增加对开发实践的更广泛理解，并增加他们的产品知识。

1.  这是有益的，因为并非所有的开发实践都有价值，可能是因为还没有想到更好的开发实践，有时询问为什么要做某事将有助于为组织带来必要的变化。

1.  指标和报告是说服利益相关者接受 CI/CD 价值的好方法。记住，有时一张图片胜过千言万语。

1.  领导层可能不理解自动化的含义，也不理解自动化对组织的影响。您可能需要通过午餐和学习或公司演示来对他们进行教育。

# 第五章：Jenkins 的安装和基础知识

1.  Chocolatey

1.  Java

1.  `curl -X POST -u <user>:<password> http://<jenkins.server>/restart`

1.  `sudo ufw allow 8080`

1.  Homebrew

1.  在 Jenkins 仪表板中，单击“管理 Jenkins”，然后单击“管理插件”。

1.  在 Jenkins 仪表板中，单击“管理 Jenkins”，然后单击“配置系统”，然后您需要向下滚动到全局属性和必要的环境变量。

# 第六章：编写自由脚本

1.  问号符号在您需要了解有关构建配置选项的详细信息时非常方便。

1.  这是一个 Crontab 语法，您可以用它来轮询您的版本控制系统。

1.  是的，您可以使用多种语言，例如您可能有一个 go 脚本和一个 Node.js 脚本需要在您的环境中运行。

1.  这是您正在操作的 Unix 环境，许多 Unix 命令都可以使用，如`sed`和`awk`。

1.  全局属性将在您添加的所有构建作业中可用，而项目级环境变量只能在您添加它们的特定项目中可用。这是由 EnvInject 插件启用的。

1.  查看执行的命令和/或命令的输出是有用的，这样您可以更轻松地调试在 CI 环境中运行的每个命令时出现的问题。

1.  后构建操作对于报告和收集指标非常有用，也适用于您认为重要的任何额外操作，超出主要构建脚本操作之外。

# 第七章：开发插件

1.  我们使用了 Maven 构建工具。

1.  我们在 Windows 中使用了 Chocolatey 软件包管理器。

1.  我们在 macOS 中使用了 Homebrew 软件包管理器。

1.  Maven 构建工具的`settings.xml`文件。

1.  您可以转到 URL {{domain}}/pluginManager/advanced 来管理插件。

1.  `mvn install`命令用于构建和安装 Jenkins 插件。

1.  Maven 创建一个名为`pluginname.hpi`的文件，其中插件名称可以是您为实际插件指定的任何名称。

# 第八章：使用 Jenkins 构建流水线

1.  是的，实际上您可以，当您使用 Jenkins 的 Docker 安装时，这是建议安装的插件之一。

1.  通过使用 Pipeline 编辑器，您可以获得有用的调试和可视化管道中的阶段。

1.  蓝色海洋视图仍在积极开发中，因此任何类型的管理任务都需要在经典视图中完成。

1.  是的，如果您单击管道的节点，然后单击弹出窗口，您将获得该特定构建阶段的详细视图。

1.  还没有。

1.  stages 关键字包含一个或多个 stage 指令的序列，stages 部分是 Pipeline 描述的“工作”大部分所在的地方。

1.  是的，它确实需要被包装。

# 第九章：Travis CI 的安装和基础知识

1.  Jenkins 允许完全定制，因为它必须由 Jenkins 管理员安装和设置，而 Travis CI 则更容易设置，因为它使用应用程序 YML 脚本，并且仅在 GitHub 中使用，GitHub 是一个基于 Web 的版本控制托管服务，使用 Git。

1.  不。

1.  您需要转到 Travis 中的个人资料，然后单击同步按钮，然后切换您新同步的存储库。

1.  标量是普通值，意味着它们可以是数字、字符串、布尔值。

1.  YAML 中的列表只是元素的集合。

1.  锚点用作在 YML 文件中重用项目的方法。

1.  是的，在**before_install**块中添加第二个编程语言。

1.  通过在 YML 脚本的 services 块中添加 docker 来启用 docker。

# 第十章：Travis CI CLI 命令和自动化

1.  Travis CLI 用户文档（[`github.com/travis-ci/travis.rb#windows`](https://github.com/travis-ci/travis.rb#windows)）建议您使用 RubyInstaller（[`rubyinstaller.org/`](https://rubyinstaller.org/)）在 Windows 操作系统上安装最新版本的 Ruby。

1.  您应该使用`travis version`命令。

1.  您使用`travis help`命令。例如，要打印有关 token 命令的信息，运行以下命令：`travis help token`。

1.  您需要运行`travis login`命令，然后输入您的 GitHub 用户名和密码。

1.  您需要传递以下 HTTP 标头：`Travis-API-Version: 3`。

1.  `travis report`命令打印出系统配置信息。

1.  `travis lint`命令将检查您的 Travis yml 脚本的语法和有效性。

1.  `travis init`命令可帮助您在项目中设置 Travis，例如，在项目中设置 go，请运行以下命令：`travis init go`。

# 第十一章：Travis CI UI 日志记录和调试

1.  是的，每当您在 GitHub 中合并拉取请求时，Travis CI 将自动启动另一个构建。

1.  不，但您将看到 before_install 和 install 生命周期事件的标签，以及其他一些生命周期事件。

1.  您需要使用 Docker 来拉取一个镜像，您可以在这里找到完整的 Docker 镜像列表（[`docs.travis-ci.com/user/common-build-problems/#Troubleshooting-Locally-in-a-Docker-Image`](https://docs.travis-ci.com/user/common-build-problems/#Troubleshooting-Locally-in-a-Docker-Image)）。

1.  是的，但您需要发送电子邮件至`support@travis-ci.com`，然后请求启用特定存储库的调试模式。此外，您还需要使用相应的作业 ID 调用 Travis API 来触发调试模式中的构建。

1.  您需要通过对/builds 端点进行 GET 请求来对 Travis API 进行 API 调用。以下是使用 curl REST 客户端的示例请求：

```
curl -s -X GET \
 -H "Content-Type: application/json" \
 -H "Accept: application/json" \
 -H "Travis-API-Version: 3" \
 -H "Authorization: token $(travis token)" \
 -d '{ "quiet": true }' \
 https://api.travis-ci.org/builds
```

1.  `travis_run_before_install`是您将使用的便利 bash 函数。

1.  您将使用 travis setup SERVICE cli 命令，以下是在 Travis CI 中设置 Heroku 的示例命令：`travis setup heroku`。

# 第十二章：CircleCI 的安装和基础知识

1.  Jenkins 允许进行全面定制，因为必须由 Jenkins 管理员安装和设置，而 Circle CI 要容易得多，但不允许进行 Jenkins 可以实现的定制。话虽如此，您只需声明要使用的环境（默认为 Linux），并在 yml 脚本中声明要使用的构建语言，例如 Java。

1.  是的，Circle CI 适用于 Bitbucket 和 GitHub。

1.  您只需在 Circle CI 应用程序中点击“添加项目”按钮，然后点击要设置的存储库的“设置项目”按钮。

1.  您点击左侧导航窗格中的 JOBS 链接，然后点击您正在使用的存储库，然后查看最近完成的作业。

1.  我们使用了[Maven 构建工具](https://maven.apache.org/)。

1.  不，您应该使用 Circle CI Syntax 的版本 2，因为版本 1 已被弃用。

1.  您将构建语言放在^(image)字段中的 docker 字段内。以下是一个示例：

```
jobs:
   build:
     docker:
         # specify the version you desire here
         - image: circleci/node:7.10

```

# 第十三章：Circle CI CLI 命令和自动化

1.  您需要安装 Docker 才能使用 Circle CI CLI。

1.  我们从 GitHub Releases 获取了夜间构建（[`github.com/CircleCI-Public/circleci-cli/releases`](https://github.com/CircleCI-Public/circleci-cli/releases)）。

1.  目前 CLI 中有 6 个命令，但将来可能会添加更多命令。

1.  `help`命令很有用，因为它解释了如何使用每个命令以及命令的作用。

1.  workflows 字段是您可以在 Circle CI 中运行并行作业的方式。

1.  我们使用了`circleci config validate`命令。

1.  API 端点是`https://circleci.com/api/v1.1/`。

# 第十四章：Circle CI UI 日志和调试

1.  API 端点是`POST https://circleci.com/api/v1.1/project/:vcs-type/:username/:project/follow?circle-token=:token`。

1.  是的，cat 实用程序可用于创建新文件，您可以执行以下操作：

```
cat > somefile
# input
input
Press Control D
```

1.  您可以使用竖线操作符（|）创建多行命令，例如：

```
- run: name: Run Tests and Run Code Coverage with NYC command: | echo "Generate Code Coverage" npm test echo "Show the coverage" npm run coverage
# or simply
- run: |
 echo "Generate Code Coverage" npm test echo "Show the coverage" npm run coverage
```

1.  是的，如果在脚本中运行`set -x`选项并设置密码，它们可能会泄漏到标准输出，因此请将密码或密钥存储在 CircleCI 应用程序中的项目或上下文设置中。

1.  使用`circleci config validate`命令验证您的配置 yml 脚本。

1.  是的，如果您转到项目设置并查看环境变量，并使用导入变量按钮，它们可以。

1.  我们使用了 save_cache 和 restore_cache 声明。

# 第十五章：最佳实践

1.  这很重要，因为 CI/CD 流水线中的第一个阶段旨在快速运行。

1.  CI/CD 流水线中的提交阶段通常是流水线中的第一个阶段，在这个阶段中，您构建您的构件并运行单元测试。

1.  负载测试。

1.  Vault

1.  您应该小心，因为您可能会意外地暴露密码。

1.  开发人员和运营之间的协作。

1.  goreleaser ([`goreleaser.com/`](https://goreleaser.com/))
