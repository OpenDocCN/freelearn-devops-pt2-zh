# 前言

在过去的几年里，敏捷软件开发模式在全球范围内得到了相当大的增长。特别是在电子商务领域，对于一种快速灵活应对频繁修改的软件交付解决方案的需求非常巨大。因此，持续集成和持续交付方法越来越受欢迎。

无论是小项目还是大项目，都会获得诸如早期问题检测、避免糟糕的代码进入生产以及更快的交付等好处，这导致了生产力的增加。

本书，*使用 Jenkins 学习持续集成第二版*，作为一个逐步指南，通过实际示例来设置持续集成、持续交付和持续部署系统。这本书是 20% 的理论和 80% 的实践。它首先解释了持续集成的概念及其在敏捷世界中的重要性，并专门有一整章介绍这个概念。用户随后学习如何配置和设置 Jenkins，然后实现使用 Jenkins 的持续集成和持续交付。还有一个关于持续部署的小章节，主要讨论持续交付和持续部署之间的区别。

# 本书内容

第一章，*持续集成的概念*，介绍了一些最流行和广泛使用的软件开发方法如何催生了持续集成。接着详细解释了实现持续集成所需的各种要求和最佳实践。

第二章，*安装 Jenkins*，是一份关于在各种平台上安装 Jenkins 的分步指南，包括 Docker。

第三章，*新 Jenkins*，提供了新 Jenkins 2.x 的外观和感觉概述，并深入解释了其重要组成部分。它还向读者介绍了 Jenkins 2.x 中新增的功能。

第四章，*配置 Jenkins*，专注于完成一些基本的 Jenkins 管理任务。

第五章，*分布式构建*，探讨了如何使用 Docker 实现构建农场，并讨论了将独立机器添加为 Jenkins 从属节点的方法。

第六章，*安装 SonarQube 和 Artifactory*，涵盖了为持续集成安装和配置 SonarQube 和 Artifactory 的步骤。

第七章，*使用 Jenkins 进行持续集成*，带领你设计持续集成并使用 Jenkins 以及其他一些 DevOps 工具来实现它的步骤。

第八章，*使用 Jenkins 进行持续交付*，概述了持续交付的设计以及使用 Jenkins 与其他一些 DevOps 工具实现它的方法。

第九章，*使用 Jenkins 进行持续部署*，解释了持续交付与持续部署之间的区别。它还提供了使用 Jenkins 实现持续部署的逐步指南。

附录，*支持工具和安装指南*，介绍了使您的 Jenkins 服务器在互联网上可访问所需的步骤以及 Git 的安装指南。

# 本书所需的内容

要能够理解本书中描述的所有内容，您需要一台具有以下配置的计算机：

+   **操作系统**：

    +   Windows 7/8/10

    +   Ubuntu 14 及更高版本

+   **硬件要求**：

    +   至少拥有 4 GB 内存和多核处理器的计算机

+   **其他要求**：

    +   GitHub 账户（公共或私人）

# 本书面向的读者

本书旨在读者具有较少或没有敏捷或持续集成和持续交付方面的经验。对于任何新手想要利用持续集成和持续交付的好处以提高生产力并缩短交付时间的人来说，它都是一个很好的起点。

构建和发布工程师、DevOps 工程师、（软件配置管理）SCM 工程师、开发人员、测试人员和项目经理都可以从本书中受益。

已经在使用 Jenkins 进行持续集成的读者可以学习如何将他们的项目提升到下一个级别，即持续交付。

本书的当前版本是其前任的完全重启。第一版读者可以利用当前版本讨论的一些新内容，例如 Pipeline as Code、Multibranch Pipelines、Jenkins Blue Ocean、使用 Docker 的分布式构建农场等。

# 约定

在本书中，您将找到许多用于区分不同类型信息的文本样式。以下是其中一些样式的示例及其含义的解释。文本中的代码词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 句柄显示如下：“这将在您的系统上下载一个`.hpi`文件。”

代码块设置如下：

```
stage ('Performance Testing'){
    sh '''cd /opt/jmeter/bin/
    ./jmeter.sh -n -t $WORKSPACE/src/pt/Hello_World_Test_Plan.jmx -l
    $WORKSPACE/test_report.jtl''';
    step([$class: 'ArtifactArchiver', artifacts: '**/*.jtl'])
}
```

当我们希望引起您对代码块的特定部分的注意时，相关行或项将加粗显示：

```
stage ('Performance Testing'){
    sh '''cd /opt/jmeter/bin/
    ./jmeter.sh -n -t $WORKSPACE/src/pt/Hello_World_Test_Plan.jmx -l
    $WORKSPACE/test_report.jtl''';
    step([$class: 'ArtifactArchiver', artifacts: '**/*.jtl'])
}
```

在一些命令中使用的额外的“**\**”仅用于指示命令在下一行继续。任何命令行输入或输出都按以下方式编写：

```
 cd /tmp
   wget https://archive.apache.org/dist/tomcat/tomcat-8/ \
   v8.5.16/bin/apache-tomcat-8.5.16.tar.gz
```

**新术语** 和 **重要单词** 以粗体显示。屏幕上看到的单词，例如在菜单或对话框中，会以如下方式出现在文本中：“从 Jenkins 仪表板上，点击 “Manage Jenkins”|“Plugin Manager”|“Available” 选项卡。”

警告或重要提示呈现如下。

提示和技巧呈现如下。

# 读者反馈

我们始终欢迎读者的反馈。请告诉我们您对本书的看法-您喜欢或不喜欢的地方。读者反馈对我们很重要，因为它帮助我们开发让您真正受益的标题。要向我们发送一般反馈，只需发送电子邮件至`feedback@packtpub.com`，并在消息主题中提到书的标题。如果您对某个专题有专业知识，并有兴趣参与撰写或投稿书籍，请查看我们的作者指南，网址为[www.packtpub.com/authors](http://www.packtpub.com/authors)。 

# 客户支持

既然您是 Packt 书籍的自豪所有者，我们有许多措施可帮助您充分利用您的购买。

# 下载示例代码

您可以从您在 [`www.packtpub.com`](http://www.packtpub.com) 的账户下载本书的示例代码文件。如果您在其他地方购买了这本书，您可以访问[`www.packtpub.com/support`](http://www.packtpub.com/support)，注册后文件将直接发送到您的邮箱。按照以下步骤下载代码文件：

1.  使用您的电子邮件地址和密码登录或注册到我们的网站。

1.  将鼠标指针悬停在顶部的“支持”标签上。

1.  点击“代码下载 & 勘误”。

1.  在搜索框中输入书名。

1.  选择您想要下载代码文件的书籍。

1.  从下拉菜单中选择您购买本书的位置。

1.  点击“代码下载”。

下载文件后，请确保使用最新版本的解压软件解压文件夹：

+   Windows 下的 WinRAR / 7-Zip

+   Mac 下的 Zipeg / iZip / UnRarX

+   Linux 下的 7-Zip / PeaZip

本书的代码包也托管在 GitHub 上，链接为[`github.com/PacktPublishing/Learning-Continuous-Integration-with-Jenkins-Second-Edition`](https://github.com/PacktPublishing/Learning-Continuous-Integration-with-Jenkins-Second-Edition)。我们还有其他丰富的图书和视频代码包可供下载，地址为[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)。赶紧去看看吧！

# 下载本书的彩色图片

我们还为您提供了一个 PDF 文件，其中包含本书中使用的截图/图表的彩色图片。彩色图片将帮助您更好地理解输出中的变化。您可以从[`www.packtpub.com/sites/default/files/downloads/LearningContinuousIntegrationwithJenkinsSecondEdition_ColorImages.pdf`](https://www.packtpub.com/sites/default/files/downloads/LearningContinuousIntegrationwithJenkinsSecondEdition_ColorImages.pdf)下载此文件。

# 勘误

尽管我们已经尽一切努力确保内容的准确性，但错误还是会发生。如果你在我们的书中发现了错误——也许是文本或代码中的错误——我们将不胜感激地接受您的报告。通过这样做，你可以帮助其他读者避免困扰，也可以帮助我们改进后续版本的这本书。如果你发现任何勘误，请访问 [`www.packtpub.com/submit-errata`](http://www.packtpub.com/submit-errata)，选择你的书，点击勘误提交表格链接，并输入勘误的详细信息。一旦你的勘误被核实，你的提交将被接受，并将勘误上传到我们的网站，或者添加到该标题的现有勘误列表的勘误部分。要查看以前提交的勘误，请访问 [`www.packtpub.com/books/content/support`](https://www.packtpub.com/books/content/support)，并在搜索字段中输入书名。所需信息将出现在勘误部分下。

# 盗版

互联网上对受版权保护的材料的盗版是所有媒体持续存在的问题。在 Packt，我们非常重视对我们的版权和许可的保护。如果你在互联网上发现我们作品的任何形式的非法副本，请立即向我们提供位置地址或网站名称，以便我们采取措施。请通过 `copyright@packtpub.com` 联系我们，并附上可疑盗版材料的链接。感谢你帮助我们保护我们的作者和我们提供有价值内容的能力。

# 问题

如果你在阅读本书的过程中遇到任何问题，可以通过 `questions@packtpub.com` 联系我们，我们将尽力解决问题。
