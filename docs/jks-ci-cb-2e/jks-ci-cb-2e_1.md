# 第一章：维护 Jenkins

在本章中，我们将涵盖以下步骤:

+   使用测试 Jenkins 实例

+   备份和恢复

+   从命令行修改 Jenkins 配置

+   安装 Nginx

+   配置 Nginx 作为反向代理

+   报告总体存储使用情况

+   通过日志解析故意失败的构建

+   通过日志解析添加警告存储使用违规的作业

+   通过 Firefox 与 Jenkins 保持联系

+   通过 JavaMelody 进行监视

+   跟踪脚本粘合剂

+   编写 Jenkins 命令行界面脚本

+   使用 Groovy 全局修改作业

+   发出归档需求信号

# 介绍

Jenkins 功能丰富，通过插件可以大大扩展。Jenkins 与许多外部系统进行通信，其作业与许多不同的技术合作。在一个运行 24 x 7 的丰富环境中维护 Jenkins 是一个挑战。你必须注意细节。添加新作业很容易，而且你不太可能很快删除旧项目。负载增加，密码过期，存储填满。此外，Jenkins 及其插件的改进速度很快。每周都会发布一个新的 Jenkins 小版本，主要是改进，偶尔会有 bug。在复杂环境中保持系统稳定，你需要监视、清理存储、备份、控制你的 Jenkins 脚本，并始终保持清洁和抛光。本章包含最常见任务的步骤。正确的维护可以降低失败的风险，例如:

+   **新插件引发异常**: 有很多优秀的插件正在快速版本更改中编写。在这种情况下，你很容易意外添加带有新缺陷的插件新版本。在升级期间曾出现过插件突然不起作用的情况。为了防止插件异常的风险，在发布到关键系统之前考虑使用一个测试 Jenkins 实例。

+   **存储溢出的问题**: 如果你保留了包括 war 文件、大量的 JAR 文件或其他类型的二进制文件和源代码在内的构建历史记录，那么你的存储空间会以惊人的速度被消耗掉。存储成本已经大幅降低，但存储使用量意味着更长的备份时间和更多从从节点到主节点的通信。为了最小化磁盘溢出的风险，你需要考虑你的备份和恢复策略，以及作业高级选项中表达的相关构建保留策略。

+   **脚本混乱**: 由于作业由各个开发团队编写，所包含脚本的位置和风格各异。这使得你很难跟踪。考虑使用明确定义的脚本位置和通过插件管理的脚本仓库。

+   **资源耗尽**: 随着内存消耗或强烈作业数量增加，Jenkins 会变慢。正确的监控和快速反应会减少影响。

+   **由于有机增长而导致工作之间普遍缺乏一致性**：Jenkins 安装和使用都很简单。无缝开启插件的能力令人上瘾。Jenkins 在组织内的采用速度可能令人叹为观止。没有一致的政策，你的团队将引入大量插件，并且也会有很多执行相同工作方式的方式。规范提高了工作的一致性和可读性，从而减少了维护工作。

    ### 注意

    本章中的示例旨在解决提到的风险。它们只代表一套方法。如果您有意见或改进意见，请随时通过 `<bergsmooth@gmail.com>` 联系我，或者最好是向 Jenkins 社区维基添加教程。

Jenkins 社区正在为您努力工作。Jenkins 每周都有小版本发布，并且许多插件偶尔都会有增量改进，因为变化的速度，会引入错误。如果您发现问题，请报告。

### 提示

**加入社区**

要添加社区错误报告或修改维基页面，您需要在 [`wiki.jenkins-ci.org/display/JENKINS/Issue+Tracking`](https://wiki.jenkins-ci.org/display/JENKINS/Issue+Tracking) 创建一个帐户。

# 使用测试 Jenkins 实例

**持续集成**（**CI**）服务器在创建确定性发布周期方面至关重要。如果 CI 存在长期不稳定性，那么在项目计划中达到里程碑的速度将会减慢。增量升级令人上瘾并且大多数情况下很简单，但应该以 Jenkins 的关键角色——软件项目的生命周期为依据来看待。

在将插件发布到您的 Jenkins 生产服务器之前，值得积极部署到一个测试 Jenkins 实例，然后坐下来让系统运行作业。这样可以给你足够的时间来对发现的任何轻微缺陷做出反应。

设置测试实例的方法有很多种。其中一种是使用 Ubuntu 的虚拟图像，并与 *主机* 服务器（虚拟机运行的服务器）共享工作区。这种方法有很多优点：

+   **保存状态**：您可以随时保存运行中虚拟图像的状态，并在以后返回到该运行状态。这对于有高风险失败的短期实验非常有用。

+   **共享图像的能力**：您可以在任何可以运行播放器的地方运行虚拟图像。这可能包括您的家庭桌面或一个高级服务器。

+   **使用多种不同操作系统**：这对运行具有多种浏览器类型的集成测试或功能测试的节点机器非常有用。

+   **交换工作区**：通过将工作区放在虚拟服务器主机外部，您可以测试不同版本级别的操作系统与一个工作区。您还可以测试 Jenkins 的一个版本与具有不同插件组合的不同主机工作区。

    ### 提示

    **长期支持版本**

    社区通过使用长期支持版本的 Jenkins 来管理核心稳定性，这个版本相对于最新版本来说更加成熟，功能较少。然而，它被认为是升级最稳定的平台（[`mirrors.jenkins-ci.org/war-stable/latest/jenkins.war`](http://mirrors.jenkins-ci.org/war-stable/latest/jenkins.war)）。

测试实例通常比接受和生产系统的规格低。通过让测试实例处于饥饿状态，你可以及早暴露出某些类型的问题，比如内存泄漏。随着你将配置移到生产环境，你希望扩大容量，这可能涉及从虚拟机移动到硬件。

这个教程详细介绍了如何使用 VirtualBox（[`www.virtualbox.org/`](http://www.virtualbox.org/)），这是一个开源的虚拟图像播放器，带有一个 Ubuntu 镜像（[`www.ubuntu.com/`](http://www.ubuntu.com/)）。虚拟图像将挂载主机服务器上的一个目录。然后你将 Jenkins 指向挂载的目录。当客户端操作系统重新启动时，Jenkins 将自动运行并对共享目录进行操作。

### 注意

在整本书中，将使用 Ubuntu 作为示例操作系统引用各个案例。

## 准备就绪

你需要下载并安装 VirtualBox。你可以在[`www.virtualbox.org/manual/UserManual.html`](https://www.virtualbox.org/manual/UserManual.html)找到下载最新版本 VirtualBox 的详细说明。在撰写本书时，从 VirtualBox 镜像 SourceForge 网站下载的最新版本是 Ubuntu 11.04。解压缩 Ubuntu 11.04 虚拟图像文件从[`sourceforge.net/projects/virtualboximage/files/Ubuntu%20Linux/11.04/ubuntu_11.04-x86.7z/download`](http://sourceforge.net/projects/virtualboximage/files/Ubuntu%20Linux/11.04/ubuntu_11.04-x86.7z/download)。

如果遇到问题，手册是一个很好的起点；特别是，请参考*第十二章*，*故障排除*，网址为[`www.virtualbox.org/manual/ch12.html`](http://www.virtualbox.org/manual/ch12.html)。

请注意，在阅读时可能会有更新的图像可用。随时尝试最新版本；很可能这个教程仍然适用。

你会在[`virtualboxes.org/images/ubuntu-server/`](http://virtualboxes.org/images/ubuntu-server/)找到一系列最新的 Ubuntu 虚拟图像。

### 提示

**安全注意事项**

如果你考虑使用他人的操作系统镜像，这是一个不良的安全实践，那么你应该按照[`wiki.ubuntu.com/Testing/VirtualBox`](https://wiki.ubuntu.com/Testing/VirtualBox)中提到的方法从引导光盘创建一个 Ubuntu 镜像。

## 如何操作...

1.  运行 VirtualBox 并点击左上角的**新建**图标。现在你会看到一个用于安装虚拟图像的向导。

1.  将**名称**设置为`Jenkins_Ubuntu_11.04`。操作系统类型将自动更新。点击**下一步**按钮。

1.  将**内存**设置为**2048 MB**，然后点击**下一步**。

    请注意，主机机器需要比其分配给客户镜像的总内存多 1GB RAM。在本例中，你的主机机器需要 3GB RAM。欲了解更多详情，请访问[`www.oracle.com/us/technologies/virtualization/oraclevm/oracle-vm-virtualbox-ds-1655169.pdf`](http://www.oracle.com/us/technologies/virtualization/oraclevm/oracle-vm-virtualbox-ds-1655169.pdf)。

1.  选择**使用现有硬盘**。单击文件夹图标浏览并选择未打包的 VDI 镜像：![如何做...](img/0082OS_01_01.jpg)

1.  按下**创建**按钮。

1.  点击**启动**图标启动虚拟镜像：![如何做...](img/0082OS_01_02.jpg)

1.  使用用户名和密码`Ubuntu reverse`登录客户操作系统。

1.  从终端更改 Ubuntu 用户的密码如下：

    ```
    sudo passwd

    ```

1.  按照[`pkg.jenkins-ci.org/debian/`](http://pkg.jenkins-ci.org/debian/)中的说明安装 Jenkins 存储库。

1.  根据安全补丁更新操作系统（这可能需要一些时间取决于带宽）：

    ```
    sudo apt-get update
    sudo apt-get upgrade

    ```

1.  安装内核的`dkms`模块：

    ```
    sudo apt-get install dkms

    ```

    注意，`dkms`模块支持安装其他内核模块，例如 VirtualBox 所需的模块。欲了解更多详情，请访问[`help.ubuntu.com/community/DKMS`](https://help.ubuntu.com/community/DKMS)。

1.  安装 Jenkins：

    ```
    sudo apt-get install jenkins

    ```

1.  安装 VirtualBox 的内核模块：

    ```
    sudo /etc/init.d/vboxadd setup

    ```

1.  使用 VirtualBox 窗口中的**设备**菜单选项安装客户附件：![如何做...](img/0082OS_01_03.jpg)

1.  将`jenkins`用户添加到`vboxsf`组，如下所示：

    ```
    sudo gedit /etc/group
    vboxsf:x:1001:Jenkins

    ```

1.  修改`/etc/default/jenkins`中的`JENKINS_HOME`变量，以指向挂载的共享目录：

    ```
    sudo gedit /etc/default/jenkins
    JENKINS_HOME=/media/sf_workspacej

    ```

1.  在主机操作系统上创建名为`workspacej`的目录。

1.  在 VirtualBox 中，右键单击 Ubuntu 镜像并选择**设置**。

1.  将**文件夹路径**字段更新为指向您之前创建的目录。在下面的截屏中，你可以看到该文件夹是在我的`home`目录下创建的：![如何做...](img/0082OS_01_04.jpg)

1.  重新启动 VirtualBox，然后启动 Ubuntu 客户操作系统。

1.  在客户操作系统上运行 Firefox 并浏览`http://localhost:8080`。你将看到一个本地运行的 Jenkins 实例，准备用于实验。

## 它是如何工作的...

首先，你安装了一个 Ubuntu 的虚拟镜像，更改了密码，使其他人更难登录，并更新了客户操作系统的安全补丁。

Jenkins 存储库已添加到客户操作系统中已知存储库的列表。这涉及在本地安装存储库密钥。该密钥用于验证自动下载的软件包属于您同意信任的存储库。一旦信任启用，您可以通过标准软件包管理安装最新版本的 Jenkins，并随后积极更新。

您需要安装一些额外的代码，称为客户端附加组件，以便 VirtualBox 可以从主机共享文件夹。客户端附加组件依赖于**动态内核模块支持**（**DKMS**）。DKMS 允许将代码的部分动态添加到内核中。当您运行`/etc/init.d/vboxadd setup`命令时，VirtualBox 通过 DKMS 添加了客户端附加组件模块。

### 注意

**警告**：如果您忘记添加 DKMS 模块，则共享文件夹将在没有显示任何错误的情况下失败。

默认的 Jenkins 实例现在需要进行一些重新配置：

+   `jenkins`用户需要属于`vboxsf`组，以便具有使用共享文件夹的权限

+   `/etc/init.d/jenkins 启动`脚本指向`/etc/default/jenkins`，从而获取特定属性的值，如`JENKINS_HOME`

接下来，您可以通过 VirtualBox GUI 向宿主操作系统添加共享文件夹，最后重新启动 VirtualBox 和宿主操作系统，以确保系统处于完全配置和正确初始化的状态。

配置 VirtualBox 网络有许多选项。您可以在[`www.virtualbox.org/manual/ch06.html`](http://www.virtualbox.org/manual/ch06.html)找到一个很好的介绍。

## 参见

+   *通过 JavaMelody 进行监控*的方法

+   在[`virtualboximages.com/`](http://virtualboximages.com/)和[`virtualboxes.org/images/`](http://virtualboxes.org/images/)有两个优秀的虚拟镜像来源

# 备份和恢复

对于 Jenkins 的顺利运行来说，一个核心任务是定期备份其主目录（在 Ubuntu 中为`/var/lib/jenkins`），不一定是所有的工件，但至少是其配置以及插件需要生成报告的测试历史记录。

备份没有意义，除非您可以还原。关于此主题有很多故事。我最喜欢的（我不会提及涉及的著名公司）是在 70 年代初期的某个地方，一家公司购买了一台非常昂贵的软件和磁带备份设备，以备份通过他们的主机收集的所有营销结果。然而，并非所有事情都是自动化的。每晚都需要将一盘磁带移入特定插槽。一个工人被分配了这项任务。一年来，工人专业地完成了这项任务。有一天发生了故障，需要备份。备份无法还原。原因是工人每晚还需要按下录制按钮，但这不是分配给他的任务的一部分。没有定期测试还原过程。失败的是过程，而不是薪水微薄的人。因此，吸取历史教训，本配方描述了备份和还原。

目前，备份有多个插件可用。我选择了 thinBackup 插件（[`wiki.jenkins-ci.org/display/JENKINS/thinBackup`](https://wiki.jenkins-ci.org/display/JENKINS/thinBackup)），因为它允许调度。

### 提示

**插件的快速演进和配方的有效性**

插件更新频繁，可能每周都需要更新。然而，核心配置改变的可能性不大，但增加额外选项的可能性很大，这会增加你在 GUI 中输入的变量。因此，本书中显示的截图可能与最新版本略有不同，但配方应该保持不变。

## 准备工作

为 Jenkins 创建一个具有读写权限的目录并安装 thinBackup 插件。

### 小贴士

**把墨菲当朋友**

你应该假设本书中的所有配方情况最糟糕：外星人攻击、咖啡泼在主板上、猫吃电缆、电缆吃猫等等。确保你正在使用一个测试 Jenkins 实例。

## 操作步骤...

1.  在**管理 Jenkins**页面点击**ThinBackup**链接：![操作步骤...](img/0082OS_01_32.jpg)

1.  点击**工具集**图标旁边的**设置**链接。

1.  按照以下截图中显示的细节添加，其中`/data/jenkins/backups`是你之前创建的目录的占位符。注意关于使用`H`语法的警告；这将在稍后解释。![操作步骤...](img/0082OS_01_05.jpg)

1.  点击**保存**。

1.  然后，点击**立即备份**图标。

1.  从命令行访问你的备份目录。现在你应该看到一个名为`FULL-{timestamp}`的额外子目录，其中`{timestamp}`是创建完整备份所需的秒数。

1.  点击**还原**图标。

1.  将显示一个名为**从备份还原**的下拉菜单，其中显示了备份的日期。选择刚刚创建的备份，然后点击**还原**按钮：![操作步骤...](img/0082OS_01_06.jpg)

1.  为了保证一致性，重新启动 Jenkins 服务器。

## 工作原理...

备份调度程序使用 cron 表示法（[`en.wikipedia.org/wiki/Cron`](http://en.wikipedia.org/wiki/Cron)）。`1 0 * * 7`表示每周的第七天在凌晨 00:01。`1 1 * * *`意味着差异备份每天只发生一次，在凌晨 1:01。每隔七天，前一次的差异备份将被删除。

还记得配置时的警告吗？将时间符号替换为`H`允许 Jenkins 选择何时运行 thinBackup 插件。`H H * * *`将在一天中的随机时间触发作业，从而分散负载。

等待 Jenkins/Hudson 空闲以执行备份是一种安全方法，并帮助 Jenkins 分散负载。建议启用此选项；否则，由于构建锁定文件，备份可能会损坏。

在指定的分钟数后强制 Jenkins 进入安静模式，确保在备份时没有作业在运行。此选项在等待 Jenkins 在特定时间内保持安静后强制进入安静模式。这可以避免备份等待 Jenkins 自然达到安静时刻时出现问题。

差异备份仅包含自上次完整备份以来已修改的文件。插件查看最后修改日期以确定需要备份的文件。如果另一个进程更改了最后修改日期但实际上没有更改文件内容，该过程有时可能会出错。

**61** 是使用备份创建的目录数。由于我们通过 **清理差异备份** 选项清理差异，因此在清理最旧的备份之前，我们将达到大约 54 个完整备份，大约一年的存档。

我们选择了备份构建结果，因为我们假设我们是在作业内进行清理。完整存档中不会有太多额外的内容添加。但是，如果配置错误，你应该监视存档的存储使用情况。

清理差异备份可以避免手动进行清理工作。将旧备份移到 ZIP 文件中可以节省空间，但可能会暂时减慢 Jenkins 服务器的速度。

### 注意

为了安全起见，定期将存档复制到系统之外。

名为**备份构建存档**、**备份 'userContent' 文件夹**和**备份下一个构建编号文件**的备份选项会增加备份的内容和系统状态。

恢复是返回到恢复菜单并选择日期的问题。额外选项包括恢复构建编号文件和插件（从外部服务器下载以减小备份大小）。

### 注意

我再次强调，你应该定期进行恢复操作，以避免尴尬。

全面备份是最安全的，因为它们会恢复到一个已知的状态。因此，在完整备份之间不要生成太多差异备份。

## 这还没完呢…

还有几点给你思考。

### 检查权限错误

如果有权限问题，插件将悄无声息地失败。要发现这些问题，你需要检查 Jenkins 的日志文件，`/var/log/jenkins/jenkins.log（适用于 *NIX 发行版）`，查看日志级别为 `SEVERE` 的日志：

```
SEVERE: Cannot perform a backup. Please be sure jenkins/hudson has write privileges in the configured backup path {0}.

```

### 测试排除模式

下面的 Perl 脚本将允许你测试排除模式。只需将 `$content` 值替换为你的 Jenkins 工作区位置，将 `$exclude_pattern` 替换为你要测试的模式。以下脚本将打印排除的文件列表：

```
#!/usr/bin/perl
use File::Find;
my $content = "/var/lib/jenkins";
my $exclude_pattern = '^.*\.(war)|(class)|(jar)$';
find( \&excluded_file_summary, $content );
subexcluded_file_summary {
  if ((-f $File::Find::name)&&( $File::Find::name =~/$exclude_pattern/)){
print "$File::Find::name\n";
  }
}
```

### 小贴士

**下载示例代码**

你可以从你在[`www.packtpub.com`](http://www.packtpub.com)的帐户中下载所有你购买的 Packt Publishing 书籍的示例代码文件。如果你在其他地方购买了这本书，你可以访问[`www.packtpub.com/support`](http://www.packtpub.com/support)注册，以便直接通过电子邮件接收文件。

你可以在[`perldoc.perl.org/File/Find.html`](http://perldoc.perl.org/File/Find.html)找到标准 Perl 模块 `File::Find` 的文档。

对于 `$content` 提及的每个文件和目录，`find(\&excluded_file_summary,$content);` 行调用 `excluded_file_summary` 函数。

排除模式`'^.*\.(war)|(class)|(jar)$`忽略所有的 WAR、class 和 JAR 文件。

### 提示

**EPIC Perl**

如果你是一名偶尔编写 Perl 脚本的 Java 开发人员，请考虑在 Eclipse 中使用 EPIC 插件（[`www.epic-ide.org/`](http://www.epic-ide.org/)）。

## 另请参阅

+   *报告整体存储使用*示例

+   *添加一个通过日志解析警告存储使用违规的作业*示例

# 从命令行修改 Jenkins 配置

你可能会想知道 Jenkins 工作空间顶层的 XML 文件。这些是配置文件。`config.xml` 文件是处理默认服务器值的主要文件，但也有特定的文件用于通过 GUI 设置任何插件的值。

工作空间下还有一个`jobs`子目录。每个单独的作业配置都包含在与作业同名的子目录中。然后，作业特定的配置存储在子目录中的`config.xml`中。对于`users`目录也是类似的情况：每个用户一个子目录，其中个人信息存储在`config.xml`中。

在所有基础设施中的 Jenkins 服务器具有相同的插件和版本级别的受控情况下，您可以在一个测试机器上进行测试，然后将配置文件推送到所有其他机器上。然后，你可以使用**命令行界面**（**CLI**）或`/etc/init.d`下的脚本重启 Jenkins 服务器，如下所示：

```
sudo /etc/init.d/jenkins restart

```

此示例使你熟悉主要的 XML 配置结构，然后根据 XML 的详细信息提供有关插件 API 的提示。

## 准备工作

你需要一个启用了安全性并且能够通过登录并通过命令行或通过文本编辑器进行编辑的能力来编辑文件的 Jenkins 服务器。

## 如何操作...

1.  在 Jenkins 的顶层目录中，寻找`config.xml`文件。编辑带有`numExecutors`的行，将数字`2`改为`3`：

    ```
    <numExecutors>3</numExecutors>
    ```

1.  重新启动服务器。你会看到执行器的数量已从默认的两个增加到三个：![如何操作...](img/0082OS_01_07.jpg)

1.  插件通过 XML 文件持久保存其配置。为了证明这一点，请查找`thinBackup.xml`文件。除非你已安装 thinBackup 插件，否则你找不到它。

1.  再次查看*备份和恢复*的示例。现在你会找到以下 XML 文件：

    ```
    <?xml version='1.0' encoding='UTF-8'?>
    <org.jvnet.hudson.plugins.thinbackup.ThinBackupPluginImpl plugin="thinBackup@1.7.4">
    <fullBackupSchedule>1 0 * *  7</fullBackupSchedule>
    <diffBackupSchedule>1 1 * * *</diffBackupSchedule>
    <backupPath>/data/jenkins/backups</backupPath>
    <nrMaxStoredFull>61</nrMaxStoredFull>
    <excludedFilesRegex></excludedFilesRegex>
    <waitForIdle>false</waitForIdle>
    <forceQuietModeTimeout>120</forceQuietModeTimeout>
    <cleanupDiff>true</cleanupDiff>
    <moveOldBackupsToZipFile>true</moveOldBackupsToZipFile>
    <backupBuildResults>true</backupBuildResults>
    <backupBuildArchive>true</backupBuildArchive>
    <backupUserContents>true</backupUserContents>
    <backupNextBuildNumber>true</backupNextBuildNumber>
    <backupBuildsToKeepOnly>true</backupBuildsToKeepOnly>
    </org.jvnet.hudson.plugins.thinbackup.ThinBackupPluginImpl>
    ```

## 工作原理...

Jenkins 使用 XStream ([`xstream.codehaus.org/`](http://xstream.codehaus.org/)) 将其配置持久化为可读的 XML 格式。工作空间中的 XML 文件是插件、任务和各种其他持久化信息的配置文件。`config.xml` 文件是主配置文件。安全设置和全局配置在这里设置，并反映通过 GUI 进行的更改。插件使用相同的结构，XML 值对应于底层插件类中的成员值。GUI 本身是通过 Jelly 框架 ([`commons.apache.org/jelly/`](http://commons.apache.org/jelly/)) 从 XML 创建的。

通过重新启动服务器，您可以确保在初始化阶段捕获到任何配置更改。

### 注意

还可以从**管理 Jenkins**页面的存储功能中使用**重新加载配置**，在不重新启动的情况下加载更新的配置。

## 还有更多...

这里有几件事情供你考虑。

### 关闭安全性

当您测试新的安全功能时，很容易将自己锁在 Jenkins 外面。您将无法再次登录。要解决此问题，通过编辑 `config.xml` 将 `useSecurity` 修改为 `false`，然后重新启动 Jenkins；现在安全功能已关闭。

### 查找自定义插件扩展的 JavaDoc

下面的代码行是名为 `thinBackup.xml` 的薄插件配置文件的第一行，提到了信息持久化的类。类名是一个很好的 Google 搜索词。插件可以扩展 Jenkins 的功能，可能会为管理 Groovy 脚本公开有用的方法：

```
<org.jvnet.hudson.plugins.thinbackup.ThinBackupPluginImpl>
```

### 添加垃圾的效果

只要它们被识别为有效的 XML 片段，Jenkins 就能很好地识别无效配置。例如，将以下代码添加到 `config.xml` 中：

```
<garbage>yeuchblllllllaaaaaa</garbage>
```

当您重新加载配置时，您将在**管理 Jenkins**屏幕的顶部看到这个：

![添加垃圾的效果](img/0082OS_01_08.jpg)

按下**管理**按钮将返回到详细的调试信息页面，其中包括调和数据的机会：

![添加垃圾的效果](img/0082OS_01_09.jpg)

从中可以看出，Jenkins 在阅读不理解的损坏配置时是开发人员友好的。

## 另见

+   *使用测试 Jenkins 实例* 配方

# 安装 Nginx

此配方描述了安装基本 Nginx 安装所需的步骤。

Nginx（发音为 *engine-x*）是一个免费的、开源的、高性能的 HTTP 服务器和反向代理，以及 IMAP/POP3 代理服务器。Igor Sysoev 在 2002 年开始开发 Nginx，在 2004 年发布了第一个公开版本。Nginx 以其高性能、稳定性、丰富的功能集、简单的配置和低资源消耗而闻名。

### 注意

您可以在 [`wiki.nginx.org/Main`](http://wiki.nginx.org/Main) 找到 Nginx 社区的 wiki 站点。

在你的 Jenkins 服务器前面放置一个 Nginx 服务器有很多优点：

+   **简单配置**: 语法简单直观。配置新服务器的基本细节只需要几行易于阅读的文本。

+   **速度和资源消耗**: Nginx 以比竞争对手更快的速度运行，并且资源消耗更少。

+   **URL 重写**: 强大的配置选项允许你直接管理 Nginx 后面的多个服务器的 URL 命名空间。

+   **抵消 SSL**: Nginx 可以负责安全连接，减少组织中所需的证书数量，并降低 Jenkins 服务器的 CPU 负载。

+   **缓存**: Nginx 可以缓存 Jenkins 的大部分内容，减少 Jenkins 服务器必须返回的请求数量。

+   **监控**: 当 Nginx 部署在多个 Jenkins 服务器前时，其集中日志文件可以作为一个明确的监控点。

## 准备工作

阅读官方安装说明：[`wiki.nginx.org/Install`](http://wiki.nginx.org/Install)。

## 如何操作...

1.  从终端输入：

    ```
    sudo apt-get install nginx

    ```

1.  浏览至本地主机位置。现在你将看到 Nginx 的欢迎页面：![如何操作...](img/0082OS_01_10.jpg)

1.  从终端输入 `sudo /etc/init.d/nginx`，你将获得以下输出：

    ```
    Usage: nginx {start|stop|restart|reload|force-reload|status|configtest|rotate|upgrade}

    ```

    请注意，你不仅可以停止和启动服务器，还可以检查状态并运行配置测试。

1.  通过输入 `sudo /etc/init.d/nginx status` 命令检查服务器状态：

    ```
    * nginx is running

    ```

1.  在 gedit 中编辑欢迎页面：

    ```
    sudo gedit /usr/share/nginx/html/index.html.

    ```

1.  在 `<body>` 标签后，添加 `<h1>Welcome to nginx working with Jenkins</h1>`。

1.  保存文件。

1.  浏览至本地主机位置。你将看到一个修改过的欢迎页面：![如何操作...](img/0082OS_01_11.jpg)

1.  查看 `/etc/nginx/nginx.conf` 配置文件，特别是以下几行：

    ```
    include /etc/nginx/conf.d/*.conf;
    include /etc/nginx/sites-enabled/*;
    access_log /var/log/nginx/access.log;
    error_log /var/log/nginx/error.log;
    ```

1.  编辑并保存 `/etc/nginx/sites-available/default`。对于两个 `listen` 部分，将数字 `80` 改为 `8000`：

    ```
    listen 8000 default_server;
    listen [::]:8000 default_server ipv6only=on;
    ```

    如果端口 `8000` 已被另一个服务器使用，则可以随意更改为其他端口号。

1.  通过终端运行以下命令测试配置：

    ```
    sudo /etc/init.d/nginx configtest
    * Testing nginx configuration   [ OK ]

    ```

1.  从终端重新启动服务器：

    ```
    sudo /etc/init.d/nginx restart
    * Restarting nginx nginx

    ```

1.  浏览至本地主机位置。你将看到无法连接的警告：![如何操作...](img/0082OS_01_12.jpg)

1.  浏览至 `localhost:8000`，你将看到欢迎页面。

## 工作原理...

你使用 `apt` 命令以默认设置安装了 Nginx。 `/etc/init.d/nginx` 命令用于控制服务器。你编辑了欢迎页面，位于 `/usr/share/nginx/html/index.html`，并重新启动了 Nginx。

主配置文件是 `/etc/nginx/nginx.conf`。 `include /etc/nginx/conf.d/*.conf;` 行从 `/etc/nginx/conf.d` 目录中具有 `conf` 扩展名的任何文件收集配置设置。它还通过 `include /etc/nginx/sites-enabled/*;` 命令收集 `/etc/nginx/sites-enabled` 目录中的任何配置文件。

您通过在名为`/etc/nginx/sites-available/default`的默认配置文件中使用`listen`指令更改了 Nginx 服务器监听的端口号。为了避免尴尬，我们在部署更改之前测试了配置。您可以通过终端使用`/etc/init.d/nginx configtest`命令来执行此操作。

### 提示

**支持信息**

*Nginx HTTP 服务器*由*Packt Publishing*出版的书籍详细介绍了 Nginx 的许多方面。您可以在[`www.packtpub.com/nginx-http-server-for-web-applications/book`](https://www.packtpub.com/nginx-http-server-for-web-applications/book)找到此书。

关于配置的示例章节可在线获取，网址为[`www.packtpub.com/sites/default/files/0868-chapter-3-basic-nginx-configuration_1.pdf`](http://www.packtpub.com/sites/default/files/0868-chapter-3-basic-nginx-configuration_1.pdf)。

## 更多内容……

这里还有一些您需要考虑的要点。

### 命名日志文件

Nginx 允许您在多个端口上运行多个虚拟主机。为了帮助您维护服务器，建议您将日志文件分开。为此，您需要更改`/etc/nginx/nginx.conf`中的以下行：

```
access_log /var/log/nginx/access.log;
error_log /var/log/nginx/error.log;
```

为他人提供方便。考虑使用一致的命名约定，例如包括主机名和端口号：

```
access_log /var/log/nginx/HOST_PORT_access.log;
error_log /var/log/nginx/HOST_PORT_error.log;
```

### 备份配置

我再次强调一下这一点。备份配置更改对于您的基础设施的平稳运行至关重要。就个人而言，我将所有配置更改备份到版本控制系统中。我可以查看提交日志，准确地了解何时犯了错误或使用了巧妙的调整。但是，版本控制并不总是可行的，因为可能包含诸如密码之类的敏感信息。至少要在本地自动备份配置。

## 另请参阅

+   *将 Nginx 配置为反向代理*配方

# 配置 Nginx 为反向代理

本文介绍如何将 Nginx 配置为 Jenkins 的反向代理。您将修改日志文件和端口位置，调整缓冲区大小和传递的请求标头。我还会介绍在重新启动 Nginx 之前测试配置的最佳实践。这种最佳实践帮助我避免了许多尴尬的时刻。

## 准备工作

您需要遵循*安装 Nginx*配方，并在`localhost:8080`上运行 Jenkins 实例。

## 如何做……

1.  创建`/etc/nginx/proxy.conf`文件，并添加以下代码：

    ```
    proxy_redirect          off;
    proxy_set_header        Host            $host;
    proxy_set_header        X-Real-IP       $remote_addr;
    proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;
    client_max_body_size    10m;
    client_body_buffer_size 128k;
    proxy_connect_timeout   90;
    proxy_send_timeout      90;
    proxy_read_timeout      90;
    proxy_buffers           32 4k;
    ```

1.  创建`/etc/nginx/sites-enabled/jenkins_8080_proxypass`文件，并添加以下代码：

    ```
    server {
    listen   80;
    server_name  localhost;
    access_log  /var/log/nginx/jenkins _8080_proxypass_access.log;
    error_log  /var/log/nginx/jenkins_8080_proxypass_access.log;

    location / {
    proxy_pass      http://127.0.0.1:7070/;
    include         /etc/nginx/proxy.conf;
            }
    }
    ```

1.  从终端运行`sudo /etc/init.d/nginx configtest`。您将看到以下输出：

    ```
    * Testing nginx configuration   [ OK ]

    ```

1.  在终端中，通过运行以下命令重新启动服务器：

    ```
    sudo /etc/init.d/nginx restart

    ```

1.  浏览至本地主机位置。连接将超时，如下面的截图所示：![如何做……](img/0082OS_01_13.jpg)

1.  查看访问日志 `/var/log/nginx/jenkins _8080_proxypass_access.log`。您将看到类似于以下行的行（请注意，`499` 是状态码）：

    ```
    127.0.0.1 - - [25/Jun/2014:17:50:50 +0200] "GET / HTTP/1.1" 499 0 "-" "Mozilla/5.0 (X11; Ubuntu; Linux i686; rv:30.0) Gecko/20100101 Firefox/30.0"

    ```

1.  编辑 `/etc/nginx/sites-enabled/jenkins_8080_proxypass`，将 `7070` 更改为 `8080`：

    ```
    location / {
    proxy_pass      http://127.0.0.1:8080/;
    include         /etc/nginx/proxy.conf;
            }
    ```

1.  测试配置更改：

    ```
    sudo /etc/init.d/nginx configtest
    * Testing nginx configuration   [ OK ]

    ```

1.  从终端运行以下命令重新启动 Nginx 服务器：

    ```
    sudo /etc/init.d/nginx restart

    ```

1.  浏览到本地主机位置。您将看到 Jenkins 的主页面。

## 工作原理...

Nginx 配置语法的致敬之处在于您只需配置几行即可配置 Nginx。

默认情况下，Nginx 对 `/etc/nginx/sites-enabled/` 目录中的文件中的任何配置都会起作用。在本次操作中，您向该目录添加了一个文件；然后，它被添加到 Nginx 的配置设置中，并在下次重启时生效。

配置文件包含一个带有端口和服务器名称 `localhost` 的 `server` 块。您可以在配置中定义多个服务器，它们监听不同的端口并具有不同的服务器名称。但是，在我们的情况下，我们只需要一个服务器：

```
server {
listen   80;
server_name  localhost;
```

您还定义了日志文件的位置，如下所示：

```
access_log  /var/log/nginx/Jenkins_8080_proxypass_access.log;
error_log  /var/log/nginx/jenkins_8080_proxypass_access.log;
```

Nginx 将请求头中指定的 URI 与服务器块内定义的位置指令的参数进行比较。在本例中，您只有一个指向顶级 `/` 的位置命令：

```
location / {
```

可以配置多个位置。但是，在我们的示例中，只有一个位置将所有请求传递给位于 `127.0.0.1:8080` 的 Jenkins 服务器：

```
  proxy_pass      http://127.0.0.1:8080/;
```

如上所述，当 `proxy_pass` 指向不存在的位置时，将返回 `499` HTTP 状态码。这是 Nginx 特定的标记问题的方式。

### 注意

注意，`proxy_pass` 可以同时使用 HTTP 和 HTTPS 协议。

我们加载了第二个配置文件，该文件处理了代理的详细设置。这是有用的，因为您可以在许多服务器配置中重复相同的设置，使详细信息保持集中。这种方法有助于可读性和维护。

```
include         /etc/nginx/proxy.conf;
```

### 注意

Nginx 配置允许您使用嵌入变量，如 `$remote_addr` 客户端的远程地址。Nginx 参考手册详细介绍了嵌入变量。您可以在 [`nginx.com/wp-content/uploads/2014/03/nginx-modules-reference-r3.pdf`](http://nginx.com/wp-content/uploads/2014/03/nginx-modules-reference-r3.pdf) 找到该手册。

在 `proxy.conf` 中，您设置了头信息。您将 `X-REAL-IP` 和 `X-Forwarded-For` 设置为请求者的远程地址。对于后端服务器和负载均衡器的顺利运行，您需要这两个头信息：

```
proxy_set_header        X-Real-IP       $remote_addr;
proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;
```

### 注意

欲了解更多关于 `X-Forwarded-For` 的信息，请访问 [`en.wikipedia.org/wiki/X-Forwarded-For`](http://en.wikipedia.org/wiki/X-Forwarded-For)。

您还定义了性能相关的其他细节，包括客户端主体的最大大小（10 兆字节）、超时值（90 秒）和内部缓冲区大小（324 千字节）：

```
client_max_body_size    10m;
client_body_buffer_size 128k;
proxy_connect_timeout   90;
proxy_send_timeout      90;
proxy_read_timeout      90;
proxy_buffers           32 4k;
```

### 注意

有关 Nginx 作为反向代理服务器的更多信息，请访问 [`nginx.com/resources/admin-guide/reverse-proxy/`](http://nginx.com/resources/admin-guide/reverse-proxy/)。

## 还有更多……

这里还有一些你需要考虑的要点。

### 测试复杂配置

现代计算机价格便宜而且功能强大。它们能够支持多个测试 Jenkins 和 Nginx 服务器。测试复杂配置的方法有很多。其中一种是在虚拟网络上运行多个虚拟机。另一种是使用不同的环回地址和/或不同的端口（`127.0.0.1:8080`、`127.0.0.2:8080`等）。这两种方法的优点是可以将网络流量保持在以太网卡上，并保留在计算机本地。

正如前言中所述，您可以通过类似以下命令从命令行运行 Jenkins：

```
java –jar jenkins.war –httpsport=8443 –httpPort=-1

```

Jenkins 将开始在端口`8443`上通过 HTTPS 运行。 `-httpPort=-1` 关闭了 HTTP 端口。

要选择一个单独的主目录，您首先需要设置 `JENKINS_HOME` 环境变量。

您将使用以下命令在`127.0.0.2`的端口`80`上运行 Jenkins：

```
sudo –jar jenkins.war  --httpPort=80 --httpListenAddress=127.0.0.2

```

### 卸载 SSL

Nginx 的优点之一是你可以让它处理 SSL 请求，然后将它们作为 HTTP 请求传递给多个 Jenkins 服务器。你可以在 [`wiki.jenkins-ci.org/display/JENKINS/Jenkins+behind+an+nginx+reverse+proxy`](https://wiki.jenkins-ci.org/display/JENKINS/Jenkins+behind+an+nginx+reverse+proxy) 找到这个基本配置。

首先，您需要将端口`80`上的请求重定向到 HTTPS URL。在以下示例中，使用了`301`状态码：

```
server {
listen 80;
return 301 https://$host$request_uri;
}
```

这表示链接已永久移动。这允许重定向被缓存。然后，您将需要在端口`443`上设置服务器，这是 HTTPS 的标准端口，并加载服务器和其关联密钥的证书：

```
server {
listen 443;
server_name localhost;

ssl on;
ssl_certificate /etc/nginx/ssl/server.crt;
ssl_certificate_key /etc/nginx/ssl/server.key;
```

最后，您需要在配置为端口`443`的服务器中使用 `location` 和 `proxy_pass` 将 HTTP 传递给运行 HTTP 的 Jenkins 服务器：

```
location / {
proxy_pass              http://127.0.0.1:8080;
```

### 提示

尽管它很简单，但已知的配置陷阱是众所周知的，其中一些在 [`wiki.nginx.org/Pitfalls`](http://wiki.nginx.org/Pitfalls) 中提到。

## 另请参阅

+   *安装 Nginx* 配方

# 报告整体存储使用情况

组织有各自的方式来处理不断增长的磁盘使用情况。政策从没有政策，依赖于临时人类互动，到拥有最先进的软件和中央报告设施。大多数组织处于这两个极端之间，大部分是临时干预，对于更关键的系统则自动报告一些情况。凭借极小的努力，你可以让 Jenkins 通过 GUI 报告磁盘使用情况，并定期运行触发有用事件的 Groovy 脚本。

该配方突出了磁盘使用插件，并使用该配方来讨论在 Jenkins 工作区中存储归档的成本。

磁盘使用插件与一个早期警告系统结合使用时效果最好，该系统在达到软限制或硬限制时会通知您。通过日志解析来警告存储使用违规的作业添加配方详细说明了解决方案。这个配方表明配置 Jenkins 需要很少的努力。每一步甚至可能看起来都很琐碎。Jenkins 的强大之处在于你可以从一系列简单的步骤和脚本中构建复杂的响应。

## 准备工作

您需要安装磁盘使用插件。

## 如何做...

1.  点击**管理 Jenkins**页面下的**磁盘使用情况**链接：![如何做...](img/0082OS_01_14.jpg)

1.  Jenkins 显示每个项目名称、构建和工作空间磁盘使用情况摘要的页面。点击表格顶部以按文件使用情况对工作空间进行排序：![如何做...](img/0082OS_01_15.jpg)

## 工作原理...

在 Jenkins 中添加插件非常简单。问题是您将如何处理这些信息。

在构建中你很容易忘记一个复选框；也许一个高级选项被错误地启用了。高级选项有时可能会有问题，因为它们不直接显示在 GUI 中，所以您需要先点击**高级**按钮，然后再查看它们。在星期五下午，这可能是一步太远了。

高级选项包括工件保留选项，您需要正确配置以避免磁盘使用过多。在上一个示例中，**Sakai Trunk**的工作空间为**2 GB**。这个大小与作业有自己的本地 Maven 存储库有关，如**使用私有 Maven 存储库**高级选项所定义。你很容易忽略这个选项。在这种情况下，没有什么可以做的，因为 trunk 拉取可能会导致其他项目不稳定的快照 jar。以下截图显示的高级选项包括工件：

![工作原理...](img/0082OS_01_16.jpg)

查看项目的高级选项后，查看项目的磁盘使用情况，可以帮助您找到不必要的私有存储库。

## 还有更多...

如果您保留了大量的工件，这表明您使用 Jenkins 的目的失败了。Jenkins 是推动产品通过其生命周期的引擎。例如，当一个作业每天构建快照时，你应该将快照推送到开发人员认为最有用的地方。那不是 Jenkins，而是一个 Maven 存储库或者像 Artifactory ([`www.jfrog.com/products.php`](http://www.jfrog.com/products.php))、Apache Archiva ([`archiva.apache.org/`](http://archiva.apache.org/)) 或 Nexus ([`nexus.sonatype.org/`](http://nexus.sonatype.org/)) 这样的存储库管理器。这些存储库管理器与将内容转储到磁盘相比具有显著优势，例如：

+   **通过充当缓存加快构建速度**：开发团队往往会处理相似或相同的代码。如果您构建并使用仓库管理器作为镜像，那么仓库管理器将缓存依赖项；当作业 Y 请求相同的构件时，下载将在本地进行。

+   **充当本地共享快照的机制**：也许您的一些快照仅用于本地使用。仓库管理器具有限制访问的功能。

+   **用于便捷的构件管理的图形用户界面**：所有三个仓库管理器都有直观的 GUI，使您的管理任务尽可能简单。

在考虑到这些因素的情况下，如果您在 Jenkins 中看到构件的积累，而它们比部署到仓库更不可访问和有益，请将其视为需要升级基础设施的信号。

欲了解更多信息，请访问[`maven.apache.org/repository-management.html`](http://maven.apache.org/repository-management.html)。

### 注意

**保留策略**

Jenkins 可能会消耗大量磁盘空间。在作业配置中，您可以决定是保留构件还是在一段时间后自动删除它们。删除构件的问题是您也将删除任何自动测试的结果。幸运的是，有一个简单的技巧可以避免这种情况。在配置作业时，点击**丢弃旧构建**，选中**高级**复选框，并定义**保留构件的最大构建数**。然后在指定的构建数后删除构件，但日志和结果会被保留。这有一个重要的后果：即使您已经删除了其他占用磁盘更多的构件，您现在也允许报告插件继续显示测试历史。

## 另请参阅

+   **备份和恢复**配方

# 通过日志解析有意失败的构建

让我们想象一下，您被要求清理没有在其构建过程中运行任何单元测试的代码。代码很多。为了迫使质量的提高，如果您错过了一些残留的缺陷，那么您希望 Jenkins 构建失败。

你需要的是一个灵活的日志解析器，它可以在构建输出中发现的问题失败或警告。救命稻草是，本配方描述了如何配置一个日志解析插件，该插件可以在控制台输出中发现不需要的模式，并在发现模式时失败作业。例如，当没有单元测试时，Maven 会发出警告。

## 准备工作

您需要按照[`wiki.jenkins-ci.org/display/JENKINS/Log+Parser+Plugin`](https://wiki.jenkins-ci.org/display/JENKINS/Log+Parser+Plugin)中提到的安装日志解析器插件。

## 如何做...

1.  在 Jenkins 工作空间下创建由 Jenkins 拥有的`log_rules`目录。

1.  将`no_tests.rule`文件添加到`log_rules`目录中，内容为一行：

    ```
    error /no tests/
    ```

1.  创建一个带有在编译过程中产生弃用警告的源代码的作业。在以下示例中，您正在使用来自 Sakai 项目的 CLOG 工具：

    +   **作业名称**：`Sakai_CLOG_Test`

    +   **Maven 2/3 项目**

    +   **源代码管理**：`Git`

    +   **仓库 URL**：`https://source.sakaiproject.org/contrib/clog/trunk`

    +   **构建**

    +   **Maven 版本**：`3.2.1`（或者您当前版本的标签）

    +   **目标和选项**：`clean install`

1.  运行构建。它不应该失败。

1.  如下截图所示，访问 Jenkins 的**管理配置**页面，并在**控制台输出解析**部分添加描述和解析规则文件的位置：![操作方法...](img/0082OS_01_17.jpg)

1.  在 `Sakai_CLOG_Test` 作业的**后构建操作**部分中选中**控制台输出（构建日志）解析**框。

1.  选中**在错误时标记构建失败**复选框：![操作方法...](img/0082OS_01_18.jpg)

    为**选择解析规则**选择**停止无测试**。

    构建作业，现在它应该失败了。

1.  单击左侧菜单中的**解析的控制台输出**链接。现在您将能够看到解析的错误，如下截图所示：![操作方法...](img/0082OS_01_19.jpg)

## 工作原理...

全局配置页面允许您添加每个带有一组解析规则的文件。规则使用插件主页（[`wiki.jenkins-ci.org/display/JENKINS/Log+Parser+Plugin`](https://wiki.jenkins-ci.org/display/JENKINS/Log+Parser+Plugin)）中提到的正则表达式。

您使用的规则文件由一行组成：`error /no tests/`。

如果在控制台输出中找到**无测试**模式（区分大小写的测试），则插件会将其视为错误，构建将失败。可以添加更多的测试行。找到的第一个规则胜出。其他级别包括`warn`和`ok`。

源代码是从不存在单元测试的 Sakai ([`www.sakaiproject.org`](http://www.sakaiproject.org)) 区域中拉取的。

规则文件具有独特的`.rules`扩展名，以防您想在备份期间编写排除规则。

安装插件后，您可以在先前创建的规则文件之间为每个作业选择。

### 注意

这个插件赋予您周期性扫描明显的语法错误并适应新环境的能力。您应该考虑系统地遍历一系列失败的可疑构建的规则文件，直到完全清理为止。

## 还有更多...

另外两个常见的日志模式示例可能会出现问题，但通常不会导致构建失败：

+   **MD5 校验和**：如果 Maven 仓库有一个工件，但没有其关联的 MD5 校验和文件，那么构建将下载该工件，即使它已经有一个副本。幸运的是，该过程将在控制台输出中留下一条`warn`消息。

+   **启动自定义集成服务失败**：当您真正希望构建失败时，这些失败可能会以`warn`或`info`级别记录。

## 另请参阅

+   *添加一个作业以通过日志解析警告存储使用违规情况*的步骤

# 添加一个作业以通过日志解析警告存储使用违规情况

磁盘使用插件不太可能满足您的所有磁盘维护需求。此方案将向您展示如何通过添加自定义 Perl 脚本来加强磁盘监视，以警告磁盘使用违规行为。

脚本将生成两个警报：当磁盘使用量超出可接受水平时生成硬错误，当磁盘接近该限制时生成软警告。然后，日志解析器插件将相应地做出反应。

### 注意

对于 Jenkins 任务而言，使用 Perl 是典型的，因为 Jenkins 可以很好地适应大多数环境。您可以预期在获取工作时使用 Perl、Bash、Ant、Maven 和全系列的脚本和绑定代码。

## 准备工作

如果尚未这样做，请在 Jenkins 工作区下创建一个由 Jenkins 拥有的名为`log_rules`的目录。还要确保 Perl 脚本语言已安装在您的计算机上，并且 Jenkins 可以访问。Perl 默认安装在 Linux 发行版上。ActiveState 为 Mac 和 Windows 提供了一个体面的 Perl 发行版（[`www.activestate.com/downloads`](http://www.activestate.com/downloads)）。

## 如何操作...

1.  在`log_rules`目录下添加名为`disk.rule`的文件，并包含以下两行：

    ```
    error /HARD_LIMIT/
    warn /SOFT_LIMIT/
    ```

1.  访问 Jenkins 的**管理配置**页面，并将描述`DISC_USAGE`添加到**控制台输出**部分。指向解析规则文件的位置。

1.  将以下名为`disk_limits.pl`的 Perl 脚本添加到选择的位置，确保 Jenkins 用户可以读取该文件：

    ```
    use File::Find;
    my $content = "/var/lib/jenkins";
    if ($#ARGV != 1 ) {
      print "[MISCONFIG ERROR] usage: hard soft (in Bytes)\n";
      exit(-1);
    }
    my $total_bytes=0;
    my $hard_limit=$ARGV[0];
    my $soft_limit=$ARGV[1];

    find( \&size_summary, $content );

    if ($total_bytes>= $hard_limit){
    print "[HARD_LIMIT ERROR] $total_bytes>= $hard_limit (Bytes)\n";
    }elseif ($total_bytes>= $soft_limit){
    print "[SOFT_LIMIT WARN] $total_bytes>= $soft_limit (Bytes)\n";
    }else{
    print "[SUCCESS] total bytes = $total_bytes\n";
    }

    subsize_summary {
      if (-f $File::Find::name){
        $total_bytes+= -s $File::Find::name;
      }
    }
    ```

1.  修改`$content`变量以指向 Jenkins 工作区。

1.  创建一个自由风格软件项目任务。

1.  在**构建**部分下，添加**构建步骤 / 执行 Shell**。对于命令，请添加`perl disk_limits.pl 9000000 2000000`。

1.  随意更改硬限制和软限制（`9000000` `2000000`）。

1.  在**后构建操作**中检查**控制台输出（构建日志）解析**。

1.  勾选**在警告时标记构建不稳定**复选框。

1.  勾选**在错误时标记构建失败**复选框。

1.  将解析规则文件选择为**DISC_USAGE**：![如何操作...](img/0082OS_01_20.jpg)

1.  多次运行构建。

1.  在左侧的**构建历史**下，选择趋势链接。您现在可以查看趋势报告，并查看成功和失败的时间线，如以下屏幕截图所示：![如何操作...](img/0082OS_01_21.jpg)

## 工作原理...

Perl 脚本期望接收两个命令行输入：硬限制和软限制。硬限制是`$content`目录下磁盘利用率不应超过的字节值。软限制是一个较小的字节值，触发警告而不是错误。警告给管理员提供了在达到硬限制之前清理的时间。

Perl 脚本遍历 Jenkins 工作区并计算所有文件的大小。对于工作区下的每个文件或目录，脚本调用`size_summary`方法。

如果硬限制小于内容大小，则脚本会生成日志输出`[HARD_LIMIT ERROR]`。解析规则将捕获此错误并导致构建失败。如果达到软限制，则脚本将生成输出`[SOFT_LIMIT WARN]`。插件将根据`warn /SOFT_LIMIT/`规则检测到这一点，然后发出作业`warn`信号。

## 还有更多...

欢迎来到 Jenkins 的奇妙世界。您现在可以利用所有安装的功能。作业可以按计划执行，并在失败时发送电子邮件。您还可以发布推文、添加 Google 日历条目，并触发额外的事件，例如磁盘清理构建等等。您的想象力和 21 世纪的技术基本上是有限的。

## 另请参阅

+   *备份和恢复*配方

# 通过 Firefox 与 Jenkins 保持联系

如果您是 Jenkins 管理员，则您的角色是密切关注您的基础架构内构建活动的起伏变化。由于非编码原因，构建有时会偶尔冻结或中断。如果构建失败并且这与基础架构问题有关，则您需要迅速收到警告。Jenkins 可以通过多种方式做到这一点。第四章 *通过 Jenkins 进行通信*，专门介绍了针对不同受众的不同方法。从电子邮件、Twitter 和对话服务器，您可以选择广泛的提示、踢、喊和 ping。我甚至可以想象一个 Google 夏季代码项目，其中一个远程控制的小车移动到睡着的管理员身边，然后吹口哨。

这个配方是你被联系的更愉快的方式之一。你将使用 Firefox 附加组件拉取 Jenkins 的 RSS 源。这样你就可以在日常工作中查看构建过程了。

## 准备就绪

您需要在计算机上安装 Jenkins，并在至少一个 Jenkins 实例上拥有一个运行作业历史的帐户。您还需要添加 Status-4-Evar 插件，您可以从[`addons.mozilla.org/zh-CN/firefox/addon/status-4-evar/`](https://addons.mozilla.org/zh-CN/firefox/addon/status-4-evar/)获取。

### 注

以下网址将解释自上一版书籍以来 Firefox 状态栏发生了什么变化[`support.mozilla.org/zh-CN/kb/what-happened-status-bar`](https://support.mozilla.org/zh-CN/kb/what-happened-status-bar)。

### 提示

**为开发者做广告**

如果你喜欢这个附加组件，并希望将来获得更多功能，那么在附加组件作者的网站上捐款几美元是理性的自利行为。

## 如何做...

1.  选择浏览器右上角的打开菜单图标：![如何操作...](img/0082OS_01_22.jpg)

1.  点击附加组件按钮：![如何操作...](img/0082OS_01_23.jpg)

1.  在**搜索批量**（右上角），**搜索所有附加组件**标题搜索 Jenkins。

1.  点击**安装**按钮安装**Jenkins 构建监视器**。

1.  重新启动 Firefox。

1.  现在，在 Firefox 的右下角，您会看到一个小的 Jenkins 图标：![如何操作...](img/0082OS_01_24.jpg)

1.  右键单击图标。

1.  选择**首选项**，然后会出现**订阅**屏幕。

1.  为您的 Jenkins 实例添加一个可识别但简短的名称。例如，`插件测试服务器`。

1.  为 **Feed URL** 添加 URL，结构如下 `http://host:port/rssAll` 例如，`http://localhost:8080/rssAll`：![如何操作...](img/0082OS_01_25.jpg)

1.  检查**启用执行器监控**。

1.  单击**确定**按钮。插件工具栏中会出现一个区域，显示着**插件测试服务器**的订阅 URL 名称和一个健康图标。如果您将鼠标悬停在名称上，将显示更详细的状态信息：![如何操作...](img/0082OS_01_26.jpg)

## 工作原理...

Jenkins 提供了 RSS 订阅以使其状态信息可供各种工具访问。Firefox 插件会轮询配置的订阅，并以易于理解的格式显示信息。

要为特定的关键作业进行配置，您需要使用以下结构：`http://host:port/job/job name/rssAll`

要仅查看构建失败，请将 `rssAll` 替换为 `rssFailed`。要仅查看最后一次构建，请将 `rssAll` 替换为 `rssLatest`。

## 还有更多...

这里还有一些需要考虑的事项。

### RSS 凭据

如果在您的 Jenkins 实例上启用了安全性，则大多数 RSS 订阅将受到密码保护。要添加密码，您需要修改订阅 URL 为以下结构：

`http://username:password@host:port/path`

### 提示

**警告**

使用此插件的负面方面是在编辑期间显示任何订阅 URL 密码的纯文本。

### Firefox 的替代方案

Firefox 可在多个操作系统上运行。这使您可以在这些操作系统上使用一个插件进行通知。然而，缺点是您必须保持 Firefox 浏览器在后台运行。另一种选择是特定于操作系统的通知软件，它会在系统托盘中弹出。这种软件的示例包括用于 Mac OSX 的 CCMenu ([`ccmenu.org`](http://ccmenu.org)) 或用于 Windows 的 CCTray ([`en.sourceforge.jp/projects/sfnet_ccnet/releases/`](http://en.sourceforge.jp/projects/sfnet_ccnet/releases/))。

## 另请参阅

+   第四章中的 *使用 Google 日历进行移动演示* 配方，*通过 Jenkins 进行通信*

# 通过 JavaMelody 进行监控

JavaMelody（[`code.google.com/p/javamelody/`](http://code.google.com/p/javamelody/)）是一个提供全面监控的开源项目。Jenkins 插件监控 Jenkins 的主实例和节点。该插件提供了大量重要信息。你可以查看主要数量（如 CPU 或内存）1 天、1 周甚至数月的演变图表。演变图表非常适合确定资源消耗大的定期作业。JavaMelody 允许您实时监控资源的渐进性退化。它通过将统计数据导出为 PDF 格式来简化报告的编写。JavaMelody 已经拥有超过 25 个人年的努力，功能丰富。

本文介绍了如何轻松安装监控插件（[`wiki.jenkins-ci.org/display/Jenkins/Monitoring`](https://wiki.jenkins-ci.org/display/Jenkins/Monitoring)），然后讨论了故障排除策略及其与生成的指标的关系。

### 提示

**社区合作**

如果你觉得这个插件有用，请考虑回馈给插件或核心 JavaMelody 项目。

## 准备工作

你需要安装监控插件。

## 如何操作...

1.  在**管理 Jenkins**页面点击**监控 Jenkins 主节点**链接。现在你会看到详细的监控信息，如下截图所示：![如何操作...](img/0082OS_01_27.jpg)

1.  在`http://host:port/monitoring?resource=help/help.html`上阅读在线帮助，其中 host 和 port 指向你的服务器。

1.  通过访问`http://host:port/monitoring/nodes`直接查看节点进程的监控。

## 工作原理...

JavaMelody 的优点是以 Jenkins 用户身份运行，并且可以访问所有相关的指标。它的主要缺点是作为服务器的一部分运行，一旦发生故障就会停止监视。因此，由于这个缺点，你应该将 JavaMelody 视为监控解决方案的一部分，而不是整个解决方案。

## 还有更多...

监控是全面测试和故障排除的基础。本节探讨了这些问题与插件中提供的测量之间的关系。

### 使用 JavaMelody 进行故障排除 - 内存

你的 Jenkins 服务器有时可能会出现内存问题，原因是构建过于贪婪、插件泄露或基础架构中的某些隐藏复杂性。JavaMelody 具有广泛的内存测量范围，包括堆转储和内存直方图。

Java 虚拟机将内存分成各种区域，为了清理，它会删除没有对其他对象的引用的对象。当垃圾回收忙碌时，它可能会占用大量 CPU 资源，而且内存越满，垃圾回收就越繁忙。对于外部监控代理来说，这看起来像是一个 CPU 峰值，通常很难追踪到。仅仅因为垃圾收集器管理内存，就认为 Java 中不存在内存泄漏的潜力也是一种错误。许多常见做法，如自定义缓存或调用本地库，都可能导致内存被持有太长时间。

慢慢渗漏的内存泄漏将显示为与内存相关的演化图上的缓缓上升。如果你怀疑有内存泄漏，那么你可以通过**执行垃圾收集器**链接来强制插件进行完整的垃圾收集。如果不是内存泄漏，那么这个缓缓上升将会突然下降。

内存问题也可能表现为大的 CPU 峰值，因为垃圾收集器拼命尝试清理，但几乎无法清理足够的空间。垃圾收集器还可以在全面寻找不再被引用的对象时暂停应用程序（"停止世界"垃圾收集），从而导致网页浏览器请求的响应时间增加。这可以通过**统计 http - 1 天**中的**平均**和**最大**时间来观察到。

### 使用 JavaMelody 进行故障排除 - 痛苦的工作

你应该考虑以下几点：

+   **卸载工作**：为了稳定的基础设施，尽可能地从主实例中卸载尽可能多的工作。如果你有定期任务，请将最重的任务在时间上保持分离。时间分离不仅能均匀负载，而且可以通过观察 JavaMelody 的演化图来更容易地找到有问题的构建。还要考虑空间分离；如果某个节点或一组标记的节点显示出问题，那么开始切换作业的机器位置，并通过 `http://host:port/monitoring/nodes` 查看它们的单独性能特征。

+   **硬件成本低廉**：与支付人工小时相比，购买额外的 8 GB 大约相当于一个人小时的努力。

    ### 注意

    一个常见的错误是向服务器添加内存，同时忘记更新初始化脚本以允许 Jenkins 使用更多内存。

+   **审查构建脚本**：Javadoc 生成和自定义 Ant 脚本可以分叉 JVM 并在其自己的配置中保留定义的内存。编程错误也可能是问题的原因。不要忘记审查 JavaMelody 关于**统计系统错误日志**和**统计 http 系统错误**的报告。

+   **不要忘记外部因素**：因素包括备份、定期任务、更新 locate 数据库和网络维护。这些将显示为演化图中的周期性模式。

+   **人多势众**：结合磁盘使用插件等使用 JavaMelody，以全面了解重要统计信息。每个插件都很容易配置，但它们对您的有用性将比增加额外插件的维护成本增长更快。

## 另请参阅

+   第七章, *插件探索*中的*使用 Groovy 钩子脚本和在启动时触发事件*食谱

# 跟踪脚本粘合剂

如果维护脚本在基础架构中分散，备份和特别是还原将产生负面影响。最好将您的脚本放在一个地方，然后通过节点远程运行它们。考虑将您的脚本放在主 Jenkins 主目录下并备份到 Git 存储库。如果您能在线共享较不敏感的脚本，对社区来说将更好。您的组织将获得好处；然后脚本将得到一些重要的同行审查和改进。有关社区存储库详细信息，请查看`http://localhost:8080/scriptler.git/`中的支持信息。

在本食谱中，我们将探讨 Scriptler 插件的使用，以在本地管理您的脚本并从在线目录下载有用的脚本。

## 准备工作

您需要安装 Scriptler 插件（[`wiki.jenkins-ci.org/display/JENKINS/Scriptler+Plugin`](https://wiki.jenkins-ci.org/display/JENKINS/Scriptler+Plugin)）。

## 如何操作...

1.  在**Manage Jenkins**页面下点击**Scriptler**链接。您会注意到粗体文本。当前您没有任何可用的脚本；您可以从远程目录导入脚本或创建自己的脚本。

1.  在左侧点击**远程脚本目录**链接。

1.  点击**ScriptierWeb**选项卡。

1.  点击**getThreadDump**的软盘图标。如果脚本不可用，则选择另一个您喜欢的脚本。

1.  点击**Submit**按钮。

1.  您现在已经返回到**Scriptler**主页面。您会看到三个图标。选择最右边的图标来执行脚本：![如何操作...](img/0082OS_01_28.jpg)

1.  您现在位于**运行脚本**页面。选择一个节点，然后点击**运行**按钮。

    ### 注意

    如果脚本出现`startup failed`消息，则请在`entry.key`和`for`之间添加一行，然后脚本将正常运行。

1.  要编写新的 Groovy 脚本或上传您本地系统上的脚本，请在左侧点击**添加新脚本**链接。

## 工作原理...

此插件允许您轻松管理您的 Groovy 脚本，并强制将所有 Jenkins 管理员的代码放在一个标准位置，这样您就可以更轻松地计划备份，并间接共享知识。

该插件创建了一个名为 `scriptler` 的目录，位于 Jenkins 工作空间下，并将你创建的文件的元信息持久化到 `scriptler.xml` 文件中。第二个文件名为 `scriptlerweb-catalog.xml`，提到了可以下载的在线文件列表。

所有本地脚本都包含在子目录 scripts 中。

## 还有更多...

如果足够多的人使用这个插件，那么在线脚本列表将大大增加生成一个重要的可重用代码库的过程。因此，如果你有有趣的 Groovy 脚本，那就上传吧。你需要在第一次登录时创建一个新账户，以便登录到 [`scriptlerweb.appspot.com/login.gtpl`](http://scriptlerweb.appspot.com/login.gtpl)。

上传你的脚本允许人们对其投票，并向你发送反馈。免费的同行评审只会提高你的脚本技能，并在更广泛的社区中增加你的认可度。

## 另请参阅

+   *Scripting the Jenkins CLI* 配方

+   *使用 Groovy 对作业进行全局修改* 配方

# Scripting the Jenkins CLI

Jenkins CLI ([`wiki.jenkins-ci.org/display/JENKINS/Jenkins+CLI`](https://wiki.jenkins-ci.org/display/JENKINS/Jenkins+CLI)) 允许你在远程服务器上执行多项维护任务。任务包括将 Jenkins 实例上线和下线、触发构建以及运行 Groovy 脚本。这使得对最常见的琐事进行脚本化变得容易。

在这个配方中，你将登录到 Jenkins 实例，运行一个查找大于某个大小的文件的 Groovy 脚本，然后退出登录。该脚本代表了一个典型的维护任务。在审查输出后，你可以运行第二个脚本来删除你想要删除的文件列表。

### 注意

在撰写本章时，交互式 Groovy shell 无法从 CLI 中工作。这在错误报告中有所提及：[`issues.jenkins-ci.org/browse/JENKINS-5930`](https://issues.jenkins-ci.org/browse/JENKINS-5930)。

## 准备工作

从`http://host/jnlpJars/jenkins-cli.jar`下载 CLI JAR 文件。

将以下脚本添加到 Jenkins 控制下的一个目录，并将其命名为 `large_files.groovy`：

```
root = jenkins.model.Jenkins.instance.getRootDir()
count = 0
size =0
maxsize=1024*1024*32
root.eachFileRecurse() { file ->
count++
size+=file.size();
if (file.size() >maxsize) {
println "Thinking about deleting: ${file.getPath()}"
            // do things to large files here
        }
}
println "Space used ${size/(1024*1024)} MB Number of files ${count}"
```

## 如何做...

1.  从终端运行以下命令，将`http://host`替换为你服务器的真实地址，例如，`http://localhost:8080`。

    ```
    java -jar jenkins-cli.jar -s 
    http://host  login --username username

    ```

1.  输入你的密码。

1.  查看在线帮助：

    ```
    java -jar jenkins-cli.jar -s 
    http://host   help

    ```

1.  运行 Groovy 脚本。输出现在将提到所有超大文件：

    ```
    java -jar jenkins-cli.jar -s http://host groovy large_files.groovy

    ```

1.  通过运行以下命令注销：

    ```
    java -jar jenkins-cli.jar -s http://host logout.

    ```

## 工作原理...

CLI 允许你从命令行工作并执行标准任务。将 CLI 包装在诸如 Bash 等的 shell 脚本中，可以让你同时脚本化维护任务和大量 Jenkins 实例。本配方执行了大量维护工作。在这种情况下，它检查 x 个文件以查找超大的构件，节省你可以用在更有趣任务上的时间。

在执行任何命令之前，你需要通过 `login` 命令进行身份验证。

评审`root = jenkins.model.Jenkins.instance.getRootDir()`脚本使用 Jenkins 框架获取指向 Jenkins 工作空间的`java.io.File`。

通过`maxsize=1024*1024*32`设置最大文件大小为 32 MB。

该脚本使用标准的`root.eachFileRecurse(){ file ->` Groovy 方法访问 Jenkins 工作空间下的每个文件。

### 注意

你可以在[Jenkins 的当前 JavaDoc](http://javadoc.jenkins-ci.org/)中找到最新的文档。

## 还有更多...

此示例中使用的身份验证可以改进。你可以在`http://localhost:8080/user/{username}/configure`（其中`username`是你的用户名）下添加你的 SSH 公钥，方法是将其剪切并粘贴到**SSH 公钥**部分。你可以在[`wiki.jenkins-ci.org/display/JENKINS/Jenkins+CLI`](https://wiki.jenkins-ci.org/display/JENKINS/Jenkins+CLI)找到详细说明。

在撰写本书时，关键方法存在一些问题。有关更多信息，请访问[`issues.jenkins-ci.org/browse/JENKINS-10647`](https://issues.jenkins-ci.org/browse/JENKINS-10647)。尽管安全性较低，但请随时使用在本示例中已被证明稳定运行的方法。

### 注意

CLI 易于扩展，因此随着时间的推移，CLI 的命令列表会增加。因此，偶尔检查帮助选项非常重要。

## 另请参阅

+   *使用 Groovy 全局修改作业* 示例

+   *编写全局构建报告的脚本* 示例

# 使用 Groovy 全局修改作业

Jenkins 不仅是一个持续集成服务器，而且还是一个具有从脚本控制台中可用的暴露内部结构的丰富框架。你可以通过编程方式迭代作业、插件、节点配置和各种丰富的对象。随着作业数量的增加，你会注意到脚本变得更有价值。例如，想象一下，如果你需要在 100 个作业中增加自定义内存设置。一个 Groovy 脚本可以在几秒钟内完成。

这个示例是一个典型的例子：你将运行一个脚本，该脚本通过所有作业进行迭代。然后，脚本通过作业名称找到一个特定的作业，然后使用一个随机数更新该作业的描述。

## 准备就绪

使用管理员帐户登录 Jenkins。

## 操作步骤...

1.  创建一个名为`MyTest`的空作业。

1.  在**管理 Jenkins**页面中，点击**脚本控制台**链接。

1.  点击**添加新脚本**。

1.  将以下脚本剪切并粘贴到**脚本**文本区域输入：

    ```
    import java.util.Random
    Random random = new Random()

    hudson.model.Hudson.instance.items.each{ job ->
    println ("Class: ${job.class}")
    println ("Name: ${job.name}")
    println ("Root Dir: ${job.rootDir}")
    println ("URL: ${job.url}")
    println ("Absolute URL: ${job.absoluteUrl}")

    if ("MyTest".equals(job.name)){
      println ("Description: ${job.description}")
    job.setDescription("This is a test id: ${random.nextInt(99999999)}")
    }
    }
    ```

1.  点击**运行**按钮。结果应与以下屏幕截图类似：![操作步骤...](img/0082OS_01_29.jpg)

1.  再次运行脚本；您会注意到描述中的随机数现在已经改变了。

1.  复制并运行以下脚本：

    ```
    for (slave in hudson.model.Hudson.instance.slaves) {
    println "Slave class: ${slave.class}"
    println "Slave name: ${slave.name}"
    println "Slave URL: ${slave.rootPath}"
    println "Slave URL: ${slave.labelString}\n"
    }
    ```

    如果您的 Jenkins 主服务器上没有`slave`实例，则不会返回任何结果。否则，输出将类似于以下屏幕截图：

    ![操作步骤...](img/0082OS_01_30.jpg)

## 它是如何工作的...

Jenkins 有一个丰富的框架，可以在脚本控制台中使用。第一个脚本遍历其父级为 `AbstractItem` 的作业 ([`javadoc.jenkins-ci.org/hudson/model/AbstractItem.html`](http://javadoc.jenkins-ci.org/hudson/model/AbstractItem.html))。第二个脚本遍历 `slave` 对象的实例 ([`javadoc.jenkins-ci.org/hudson/slaves/SlaveComputer.html`](http://javadoc.jenkins-ci.org/hudson/slaves/SlaveComputer.html))。

## 还有更多...

针对硬核的 Java 开发者：如果您不知道如何执行编程任务，那么示例代码的极佳来源是 Jenkins 插件的 Subversion 目录 ([`svn.jenkins-ci.org/trunk/hudson/plugins/`](https://svn.jenkins-ci.org/trunk/hudson/plugins/)) 和更新更及时的 Github 位置 ([`github.com/jenkinsci`](https://github.com/jenkinsci))。

### 注意

如果您有兴趣捐赠您自己的插件，请查阅[`wiki.jenkins-ci.org/display/JENKINS/Hosting+Plugins`](https://wiki.jenkins-ci.org/display/JENKINS/Hosting+Plugins)上的信息。

## 另请参阅

+   *Scripting the Jenkins CLI* 配方

+   *Scripting global build reports* 配方

# 表示需要归档

每个开发团队都是独特的。团队有自己的业务方式。在许多组织中，周期性需要完成一次性任务。例如，每年结束时，对整个文件系统进行全面备份。

本配方详细介绍了一个脚本，该脚本检查任何作业的最后一次成功运行；如果年份与当前年份不同，则在作业描述的开头设置警告。因此，它向您提示现在是执行某些操作的时候，例如归档然后删除。当然，您也可以通过编程方式执行归档。但是，对于高价值的操作，值得强制干预，让 Groovy 脚本引起您的注意。

## 准备工作

使用管理账户登录 Jenkins。

## 如何执行...

1.  在 **管理 Jenkins** 页面中，点击 **Script Console** 链接，然后运行以下脚本：

    ```
    Import hudson.model.Run;
    Import java.text.DateFormat;

    def warning='<font color=\'red\'>[ARCHIVE]</font> '
    def now=new Date()

    for (job in hudson.model.Hudson.instance.items) {
    println "\nName: ${job.name}"
        Run lastSuccessfulBuild = job.getLastSuccessfulBuild()
    if (lastSuccessfulBuild != null) {
    def time = lastSuccessfulBuild.getTimestamp().getTime()
    if (now.year.equals(time.year)){
    println("Project has same year as build");
    }else {
    if (job.description.startsWith(warning)){
    println("Description has already been changed");
    }else{
    job.setDescription("${warning}${job.description}")
            }
         }
       }
    }
    ```

    任何项目，如果其上次成功构建的年份不同于当前年份，则在其描述的开头添加了以红色显示的 **[ARCHIVE]** 一词，如下图所示：

    ![如何执行...](img/0082OS_01_31.jpg)

## 它是如何工作的...

检查代码清单：

+   定义了一个警告字符串，并将当前日期存储在 now 中。通过 `for` 语句在 Jenkins 中以编程方式迭代每个作业。

+   Jenkins 有一个用于存储构建运行信息的类。运行时信息通过 `job.getLastSuccessfulBuild()` 检索，并存储在 `lastSuccessfulBuild` 实例中。如果没有成功的构建，则 `lastSuccessfulBuild` 被设置为 `null`；否则，它具有运行时信息。

+   检索上次成功构建的时间，然后通过 `lastSuccessfulBuild.getTimestamp().getTime()` 将其存储在 `time` 实例中。

当前年份与上次成功构建的年份进行比较，如果它们不同并且警告字符串尚未添加到作业描述的开头，则更新描述。

### 提示

**Javadoc**

你会在 [`javadoc.jenkins-ci.org/hudson/model/Job.html`](http://javadoc.jenkins-ci.org/hudson/model/Job.html) 找到作业 API，并且在 [`javadoc.jenkins-ci.org/hudson/model/Run.html`](http://javadoc.jenkins-ci.org/hudson/model/Run.html) 找到 `Run` 信息。

## 还有更多...

在编写自己的代码之前，你应该审查已经存在的代码。Jenkins 拥有一个庞大、免费可用且开放授权的示例代码库，插件数量达到了 1,000 个并且在不断扩展。尽管在这种情况下使用了标准 API，但值得仔细审查插件代码库。在这个例子中，你会发现部分代码是从 `lastsuccessversioncolumn` 插件中重用的。([`tinyurl.com/pack-jenkins-1`](http://tinyurl.com/pack-jenkins-1))。

### 提示

如果你在审查插件代码库时发现任何缺陷，请通过补丁和错误报告为社区做出贡献。

## 另请参阅

+   *Scripting the Jenkins CLI* 配方

+   *使用 Groovy 全局修改作业* 配方
