# 第五章：托管 Jenkins

|   | *"生产力就是能够做你以前从未能做的事情"* |   |
| --- | --- | --- |
|   | --*弗朗茨·卡夫卡* |

我们已经理解了持续交付和持续部署的概念。我们还看到了如何将`war`文件从 Jenkins 部署到 Tomcat 服务器。现在，我们将探讨如何利用托管的 Jenkins。不同的服务提供商将 Jenkins 作为服务提供。我们将了解 OpenShift 和 CloudBees 如何向用户提供 Jenkins。

本章详细描述了如何使用由流行的 PaaS 提供商（如 Red Hat OpenShift 和 CloudBees）提供的托管 Jenkins。本章还涵盖了根据客户需求使用 Jenkins 的各种细节。本章将探讨如何在 Jenkins 中使用与云相关的插件以有效使用 Jenkins。本章将涵盖以下主题：

+   在 OpenShift PaaS 中探索 Jenkins

+   在云中探索 Jenkins – CloudBees

+   CloudBees 企业插件概览

+   CloudBees 的 Jenkins 案例研究

# 在 OpenShift PaaS 中探索 Jenkins

OpenShift Online 是 Red Hat 提供的公共 PaaS——应用程序开发和托管平台。它自动化了应用程序的配置、取消配置、管理和扩展过程。这支持命令行客户端工具和 Web 管理控制台，以便轻松启动和管理应用程序。Jenkins 应用由 OpenShift Online 提供。OpenShift Online 有一个免费计划。

1.  要注册 OpenShift Online，请访问[`www.openshift.com/app/account/new`](https://www.openshift.com/app/account/new)。![在 OpenShift PaaS 中探索 Jenkins](img/3471_05_01.jpg)

1.  一旦注册，你将在[`openshift.redhat.com/app/console/applications`](https://openshift.redhat.com/app/console/applications)获得欢迎屏幕。

1.  点击**立即创建你的第一个应用程序**。![在 OpenShift PaaS 中探索 Jenkins](img/3471_05_02.jpg)

1.  选择应用程序类型，在我们的例子中，选择**Jenkins 服务器**。![在 OpenShift PaaS 中探索 Jenkins](img/3471_05_03.jpg)

1.  为你的 Jenkins 服务器提供**公共 URL**，如下面的截图所示：![在 OpenShift PaaS 中探索 Jenkins](img/3471_05_04.jpg)

1.  点击**创建应用程序**。![在 OpenShift PaaS 中探索 Jenkins](img/3471_05_05a.jpg)

1.  点击**在浏览器中访问应用**。![在 OpenShift PaaS 中探索 Jenkins](img/3471_05_05.jpg)

1.  在 Web 浏览器中访问 Jenkins。然后，使用 OpenShift 仪表板提供的凭据登录。![在 OpenShift PaaS 中探索 Jenkins](img/3471_05_06.jpg)

1.  以下是 Jenkins 仪表板的截图：![在 OpenShift PaaS 中探索 Jenkins](img/3471_05_07.jpg)

# 在云中探索 Jenkins – CloudBees

DEV@cloud 是在 CloudBees 管理的安全、多租户环境中托管的 Jenkins 服务。它运行特定版本的 Jenkins，以及与该版本良好支持的插件的选定版本。所有更新和补丁都由 CloudBees 管理，并且可用的自定义有限。

1.  访问[`www.cloudbees.com/products/dev`](https://www.cloudbees.com/products/dev)并订阅。![探索云中的 Jenkins – CloudBees](img/3471_05_08.jpg)

1.  完成订阅流程后，我们将获得 CloudBees 的仪表板，如下面的截图所示。点击**构建**。![探索云中的 Jenkins – CloudBees](img/3471_05_09.jpg)

1.  我们将看到 Jenkins 仪表板，如下面的截图所示：![探索云中的 Jenkins – CloudBees](img/3471_05_10.jpg)

1.  点击**管理 Jenkins**以配置和安装插件。![探索云中的 Jenkins – CloudBees](img/3471_05_11.jpg)

    ### 注意

    在配置构建任务之前，我们需要将应用程序的源代码存储在 CloudBees 提供的库服务中。点击**生态系统**，然后点击**库**。

    ![探索云中的 Jenkins – CloudBees](img/3471_05_12.jpg)

1.  点击子版本库或**添加库**，获取库的 URL。![探索云中的 Jenkins – CloudBees](img/3471_05_13.jpg)

1.  点击应用程序文件夹，将其导入 CloudBees 提供的子版本库。使用 TortoiseSVN 或其他 SVN 客户端导入代码。![探索云中的 Jenkins – CloudBees](img/3471_05_14.jpg)

1.  提供从 CloudBees 复制的库 URL，然后点击**确定**。![探索云中的 Jenkins – CloudBees](img/3471_05_15.jpg)

1.  提供认证信息（用户名和密码与我们的 CloudBees 账户相同）。

    点击**确定**。

    ![探索云中的 Jenkins – CloudBees](img/3471_05_16.jpg)

    导入过程将根据源文件的大小耗费一些时间。

    ![探索云中的 Jenkins – CloudBees](img/3471_05_17.jpg)

1.  在浏览器中验证库 URL，我们将找到最近导入的项目。![探索云中的 Jenkins – CloudBees](img/3471_05_18.jpg)

1.  成功导入操作后，验证 Jenkins 仪表板。![探索云中的 Jenkins – CloudBees](img/3471_05_19.jpg)

1.  在 Jenkins 仪表板上点击**新建项目**。选择**自由风格项目**，并为新构建任务命名。点击**确定**。![探索云中的 Jenkins – CloudBees](img/3471_05_20.jpg)

1.  配置页面将允许我们为构建任务配置各种特定设置。![探索云中的 Jenkins – CloudBees](img/3471_05_21.jpg)

1.  在构建任务中配置**Subversion**库。![探索云中的 Jenkins – CloudBees](img/3471_05_22.jpg)

1.  点击**应用**，然后点击**保存**。![探索云中的 Jenkins – CloudBees](img/3471_05_23.jpg)

1.  点击**立即构建**。![在云中探索 Jenkins – CloudBees](img/3471_05_24.jpg)

    验证控制台输出。

    ![在云中探索 Jenkins – CloudBees](img/3471_05_25.jpg)

    接着，它将编译源文件，并根据`build.xml`文件创建一个`war`文件，因为这是一个基于 Ant 的项目。

    ![在云中探索 Jenkins – CloudBees](img/3471_05_26.jpg)

1.  在 Jenkins 仪表板上验证成功构建。![在云中探索 Jenkins – CloudBees](img/3471_05_27.jpg)

# CloudBees 企业插件概览

以下是一些重要的 CloudBees 企业插件：

## **工作流插件**

管理软件交付管道是一项复杂的任务，开发和运维团队需要管理可能需要数天才能完成的复杂作业。工作流插件支持复杂的管道。该插件使用 Groovy DSL 进行工作流，并提供从主从故障中暂停和重新启动作业的设施。

欲了解更多信息，请访问[`www.cloudbees.com/products/cloudbees-jenkins-platform/team-edition/features/workflow-plugin`](https://www.cloudbees.com/products/cloudbees-jenkins-platform/team-edition/features/workflow-plugin)。

## **检查点插件**

让我们考虑一个场景，一个长时间运行的构建作业几乎在其结束阶段失败。这可能会影响交付计划。检查点插件提供了在检查点重新启动工作流的设施。因此，它消除了由于主从故障导致的延迟。此外，它可以帮助 Jenkins 和基础设施故障的恢复。

欲了解更多信息，请访问[`www.cloudbees.com/products/jenkins-enterprise/plugins/checkpoints-plugin`](https://www.cloudbees.com/products/jenkins-enterprise/plugins/checkpoints-plugin)。

## **基于角色的访问控制插件**

认证和授权在安全方面扮演着重要角色。授权策略可以帮助有效控制对 Jenkins 作业的访问。在项目级别和可见性方面设置权限也很重要。CloudBees 提供的**基于角色的访问控制**（**RBAC**）插件具有以下功能：

+   定义各种安全角色

+   为组分配规则

+   在全球或对象级别分配角色

+   将特定对象的组管理委托给用户

欲了解更多关于基于角色的访问控制插件的信息，请访问[`www.cloudbees.com/products/jenkins-enterprise/plugins/role-based-access-control-plugin`](https://www.cloudbees.com/products/jenkins-enterprise/plugins/role-based-access-control-plugin)。

## **高可用性插件**

Jenkins 主节点的停机时间，无论是由软件还是硬件引起的，都会影响整个产品团队。快速恢复 Jenkins 主节点至关重要，而这通常需要数小时。高可用性插件通过保留多个主节点作为备份来消除因主节点故障导致的停机时间。当检测到主节点故障时，备份主节点会自动启动。此插件使得故障检测和恢复成为一个自动过程，而非手动操作。

欲了解更多信息，请访问[`www.cloudbees.com/products/jenkins-enterprise/plugins/high-availability-plugin`](https://www.cloudbees.com/products/jenkins-enterprise/plugins/high-availability-plugin)。

## VMware ESXi/vSphere Auto-Scaling 插件

考虑这样一个场景：您需要在现有的基于 VMware 的虚拟化基础设施中运行多个 Jenkins 从节点，以利用未充分利用的容量。VMware vCenter Auto-Scaling 插件允许您在基于 VMware 的虚拟化基础设施中创建可用的从节点机器。可以配置具有相同配置的多个虚拟机池。

以下操作可在虚拟机上执行：

+   开机

+   关机/挂起

+   恢复到最后一个快照

欲了解更多信息，请访问[`www.cloudbees.com/products/jenkins-enterprise/plugins/vmware-esxivsphere-auto-scaling-plugin`](https://www.cloudbees.com/products/jenkins-enterprise/plugins/vmware-esxivsphere-auto-scaling-plugin)。

要查找 CloudBees 提供的所有插件的详细信息，请访问[`www.cloudbees.com/products/jenkins-enterprise/plugins`](https://www.cloudbees.com/products/jenkins-enterprise/plugins)。

# CloudBees 提供的 Jenkins 案例研究

我们将介绍一些 CloudBees 的案例研究，展示 Jenkins 的有效使用。

## Apache jclouds

Apache jclouds 是一个开源的多云工具包，提供在多个云上管理工作负载的功能。它基于 Java 平台创建，为用户提供了使用特定云平台功能创建和管理应用程序的完整控制权。它支持跨各种云平台的无缝迁移。Apache jclouds 支持 30 个云提供商和云软件栈，如 Joyent、Docker、SoftLayer、Amazon EC2、OpenStack、Rackspace、GoGrid、Azure 和 Google。Apache jclouds 拥有众多知名用户，如 CloudBees、Jenkins、Cloudify、cloudsoft、Twitter、Cloudswitch、enStratus 等。

### 挑战

jclouds 社区使用 Jenkins CI 进行持续集成。随着时间的推移，管理和维护 Jenkins 变得越来越困难，成本也越来越高。管理 Jenkins 是一项耗时且繁琐的任务。大多数时候，开发者忙于管理 Jenkins，而非编写代码以提升 jclouds 的效率。

### 解决方案

jclouds 团队探索了市场上可用的 PaaS 产品，并考虑了 CloudBees，这将帮助他们消除基础设施管理和维护。jclouds 团队认识到，将 Jenkins CI 工作转移到 DEV@cloud 很容易，并立即从开发者那里获得生产力提升。每周从 Jenkins 维护活动中节省了近 4 小时。

### 优势

+   通过消除服务器重启、服务器规模调整、软件更新和补丁等自动从 CloudBees 服务内部执行的活动，实现 100%专注于软件开发

+   开发者生产力提高了 33%

+   CloudBees 对 Jenkins CI 问题的技术支持

欲了解更多关于此案例研究的信息，请访问[`www.cloudbees.com/casestudy/jclouds`](https://www.cloudbees.com/casestudy/jclouds)。

## 全球银行

全球银行是全球顶级金融机构之一，提供企业与投资银行服务、私人银行服务、信用卡服务和投资管理。它拥有庞大的国际业务网络。

### 挑战

全球银行现有的流程正遭受着分散的构建流程、未经批准的软件版本和缺乏技术支持的困扰。缺乏中央控制或管理，以及流程的标准化。构建资产并非始终可访问。需要一个具有审计能力的应用程序构建服务的自动化安全流程。Jenkins 提供了标准化，以及其他集中管理的优势，如稳健性和有用的插件的可用性。在使用开源 Jenkins 后，该金融机构面临了开源 Jenkins 中不存在的其他挑战。需要更多功能来实现审批、安全、备份和审计。

### 解决方案

为了克服现有挑战，全球银行评估并选择了 CloudBees Jenkins Enterprise，考虑到其额外的高可用性、备份、安全性和作业组织插件，以及为开源 Jenkins 和开源 Jenkins 插件获取技术支持的能力。全球银行利用 CloudBees 的技术支持来设置 CloudBees Jenkins Enterprise。

### 优势

+   RBAC 插件提供安全性和额外的企业级功能。Folders 插件提供版本控制，并确保只有经过批准的软件版本被共享。

+   通过消除对每个应用程序的本地构建实例进行监控的需求，每个应用程序节省了半天开发时间。

+   技术支持能力的可用性。

欲了解更多信息，请访问[`www.cloudbees.com/casestudy/global-bank`](https://www.cloudbees.com/casestudy/global-bank)。

## 服务流程

Service-Flow 提供在线集成服务，以连接组织和各种利益相关者使用的不同 IT 服务管理工具。它提供自动创建票据、票据信息交换和票据路由的功能。它有适用于许多 ITSM 工具的适配器，如 ServiceNow 和 BMC，以及 Microsoft Service Manager Fujitsu、Atos、Efecte 和 Tieto。

### 挑战

Service-Flow 希望构建自己的服务，而不使用任何通用集成工具来实现敏捷性。Service-Flow 有多个要求，例如注重敏捷性，这需要一个支持快速开发和频繁增量更新的平台，支持 Jenkins，对数据进行控制，可靠性和可用性。

### 解决方案

Service-Flow 使用 CloudBees 平台构建和部署其 ITSM 集成服务。DEV@cloud 已被用于建立版本控制仓库，编写第一个 Java 类，设置一些基本的 Jenkins 作业，运行单元测试，执行集成测试以及其他质量检查。Service-Flow 服务在云中，通过使用 CloudBees 平台添加新功能，客户群迅速增长。

### 好处

+   开发时间减少了 50%，三个月内生产发布

+   每周多次部署更新，无需服务停机

+   生产中实现了 99.999% 的可用性

欲了解更多信息，请访问 [`www.cloudbees.com/casestudy/service-flow`](https://www.cloudbees.com/casestudy/service-flow)。

更多案例研究，请访问 [`www.cloudbees.com/customers`](https://www.cloudbees.com/customers)。

# 自测问题

Q1. 关于 CloudBees 提供的 Workflow 插件，哪项陈述是正确的？

1.  从主从故障中暂停和重启作业

1.  管理软件交付管道

1.  它使用 Groovy DSL 进行工作流程

1.  上述所有内容

Q2. CloudBees 提供的 RBAC 插件有哪些功能？

1.  定义各种安全角色

1.  将规则分配给组

1.  将角色全局分配或对象级别分配

1.  上述所有内容

Q3. CloudBees 提供的 VMware ESXi/vSphere Auto-Scaling 插件可以执行哪些操作？

1.  开机

1.  关机/挂起

1.  恢复到最后一个快照

1.  上述所有内容

# 总结

关于章节结束的有趣之处在于：每个结束的章节都引领你走向新的开始。我们了解了如何在 PaaS、RedHat OpenShift 和 CloudBees 等云服务模型上配置、管理和使用 Jenkins。我们还介绍了一些来自 CloudBees 的有趣的企业插件，这些插件增加了很大的灵活性和价值。在最后一节中，我们提供了关于 Jenkins 如何为许多组织带来益处的各种案例研究的详细信息，以及他们如何利用 Jenkins 的功能来获得竞争优势。
