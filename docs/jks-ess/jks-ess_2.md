# 第二章：代码仓库和构建工具的安装与配置

|   | *"生活本就简单，是我们执意将其复杂化"* |   |
| --- | --- | --- |
|   | --*孔子* |

我们在上一章中探讨了部署管道，其中源代码仓库和自动化构建构成了重要部分。SVN、Git、CVS 和 StarTeam 是一些流行的代码仓库，它们管理代码、制品或文档的变更，而 Ant 和 Maven 则是 Java 应用程序中流行的构建自动化工具。

本章详细描述了如何为 Java 应用程序的生命周期管理准备运行时环境，并使用 Jenkins 进行配置。它将涵盖如何将 Eclipse 与 SVN 等代码仓库集成，为持续集成创建基础。以下是本章涵盖的主题列表：

+   概述 Jenkins 中的构建及其要求

+   安装 Java 并配置环境变量

+   CentOS 和 Windows 上的 SVN 安装、配置和操作

+   安装 Ant

+   在 Jenkins 中配置 Ant、Maven 和 JDK

+   将 Eclipse 与代码仓库集成

+   安装和配置 Git

+   在 Jenkins 中创建一个新的使用 Git 的构建作业

# 概述 Jenkins 中的构建及其要求

为了解释持续集成，我们将使用安装在物理机或笔记本电脑上的代码仓库，而 Jenkins 则安装在虚拟机上，正如第一章 *探索 Jenkins* 中所建议的不同方式。下图描绘了运行时环境的设置：

![Jenkins 中构建的概述及其要求](img/3471_02_01.jpg)

我们在第一章 *探索 Jenkins* 中看到，仪表板上的**管理 Jenkins**链接用于配置系统。点击**配置系统**链接以配置 Java、Ant、Maven 和其他第三方产品相关信息。我们可以使用 Virtual Box 或 VMware 工作站创建虚拟机。我们需要安装所有必需的软件以提供持续集成的运行时环境。我们假设系统中已安装 Java。

# 安装 Java 并配置环境变量

如果系统中尚未安装 Java，则可以按以下步骤进行安装：

在 CentOS 仓库中查找 Java 相关的包，并定位到合适的安装包。

```
[root@localhost ~]# yum search java
Loaded plugins: fastestmirror, refresh-packagekit, security
.
.
ant-javamail.x86_64 : Optional javamail tasks for ant
eclipse-mylyn-java.x86_64 : Mylyn Bridge:  Java Development
.
.
java-1.5.0-gcj.x86_64 : JPackage runtime compatibility layer for GCJ
java-1.5.0-gcj-devel.x86_64 : JPackage development compatibility layer for GCJ
java-1.5.0-gcj-javadoc.x86_64 : API documentation for libgcj
java-1.6.0-openjdk.x86_64 : OpenJDK Runtime Environment
java-1.6.0-openjdk-devel.x86_64 : OpenJDK Development Environment
java-1.6.0-openjdk-javadoc.x86_64 : OpenJDK API Documentation
java-1.7.0-openjdk.x86_64 : OpenJDK Runtime Environment
jcommon-serializer.x86_64 : JFree Java General Serialization Framework
.
.
Install the identified package java-1.7.0-openjdk.x86_64
[root@localhost ~]# yum install java-1.7.0-openjdk.x86_64
Loaded plugins: fastestmirror, refresh-packagekit, security
No such command: in. Please use /usr/bin/yum –help

```

现在通过执行`yum install`命令安装本地仓库中可用的 Java 包，如下所示：

```
[root@localhost ~]# yum install java-1.7.0-openjdk.x86_64
Loaded plugins: fastestmirror, refresh-packagekit, security
Loading mirror speeds from cached hostfile
Setting up Install Process
Resolving Dependencies
--> Running transaction check
---> Package java-1.7.0-openjdk.x86_64 1:1.7.0.3-2.1.el6.7 will be installed
--> Finished Dependency Resolution

Dependencies Resolved
.
.
Install       1 Package(s)

Total download size: 25 M
Installed size: 89 M
Is this ok [y/N]: y
Downloading Packages:
java-1.7.0-openjdk-1.7.0.3-2.1.el6.7.x86_64.rpm                                                                                                  |  25 MB     00:00
Running rpm_check_debug
Running Transaction Test
Transaction Test Succeeded
Running Transaction
 Installing : 1:java-1.7.0-openjdk-1.7.0.3-2.1.el6.7.x86_64                                       1/1
 Verifying  : 1:java-1.7.0-openjdk-1.7.0.3-2.1.el6.7.x86_64                                      1/1

Installed:
 java-1.7.0-openjdk.x86_64 1:1.7.0.3-2.1.el6.7
Complete!

```

Java 已成功从本地仓库安装。

## 配置环境变量

以下是配置环境变量的步骤：

1.  设置`JAVA_HOME`和`JRE_HOME`变量

1.  转到`/root`

1.  按下*Ctrl* + *H*以列出隐藏文件

1.  找到`.bash_profile`并编辑它，通过追加 Java 路径，如下面的截图所示：![配置环境变量](img/3471_02_02.jpg)

# 在 CentOS 和 Windows 上安装、配置和操作 SVN

从本地仓库在 CentOS 上安装 SVN。

## 在 CentOS 上安装 SVN

要在 CentOS 机器上安装 SVN，请执行以下`yum install mod_dav_svn subversion`命令：

```
[root@localhost ~]# yum install mod_dav_svn subversion
Loaded plugins: fastestmirror, refresh-packagekit, security
Loading mirror speeds from cached hostfile
Setting up Install Process
Resolving Dependencies
--> Running transaction check
---> Package mod_dav_svn.x86_64 0:1.6.11-7.el6 will be installed
---> Package subversion.x86_64 0:1.6.11-7.el6 will be installed
--> Processing Dependency: perl(URI) >= 1.17 for package: subversion-1.6.11-7.el6.x86_64
--> Running transaction check
---> Package perl-URI.noarch 0:1.40-2.el6 will be installed
--> Finished Dependency Resolution

Dependencies Resolved
.
.
Installed:
 mod_dav_svn.x86_64 0:1.6.11-7.el6                                                   subversion.x86_64 0:1.6.11-7.el6

Dependency Installed:
 perl-URI.noarch 0:1.40-2.el6
Complete!
[root@localhost ~]#

```

### 配置 SVN

使用`htpasswd`命令创建密码文件。最初使用`-cm`参数。这会创建文件并用 MD5 加密密码。如果需要添加用户，确保您只使用`-m`标志，而不是初始创建后的`–c`。

```
[root@localhost conf.d]# htpasswd -cm /etc/svn-auth-conf yourusername
New password:
Re-type new password:
Adding password for user yourusername
[root@localhost conf.d]#

[root@localhost conf.d]# htpasswd -cm /etc/svn-auth-conf mitesh
New password:
Re-type new password:
Adding password for user mitesh
[root@localhost conf.d]#

```

现在在 Apache 中配置 SVN 以整合两者。编辑`/etc/httpd/conf.d/subversion.conf`。位置是 Apache 将在 URL 栏中传递的内容。

```
LoadModule dav_svn_module     modules/mod_dav_svn.so
LoadModule authz_svn_module   modules/mod_authz_svn.so

#
# Example configuration to enable HTTP access for a directory
# containing Subversion repositories, "/var/www/svn".  Each repository
# must be both:
#
#   a) readable and writable by the 'apache' user, and
#
#   b) labelled with the 'httpd_sys_content_t' context if using
#   SELinux
#

#
# To create a new repository "http://localhost/repos/stuff" using
# this configuration, run as root:
#
#   # cd /var/www/svn
#   # svnadmin create stuff
#   # chown -R apache.apache stuff
#   # chcon -R -t httpd_sys_content_t stuff
#

<Location />
 DAV svn
 SVNParentPath /var/www/svn/
#
#   # Limit write permission to list of valid users.
#   <LimitExcept GET PROPFIND OPTIONS REPORT>
#      # Require SSL connection for password protection.
#      # SSLRequireSSL
#
 AuthType Basic
 SVNListParentPath on
 AuthName "Subversion repos"
 AuthUserFile /etc/svn-auth-conf
 Require valid-user
#   </LimitExcept>
</Location>

```

现在所有配置都已完成。让我们在 SVN 上执行操作。

### SVN 操作

在 CentOS 虚拟机上创建实际仓库以执行 SVN 操作。

```
[root@localhost ~] cd /var/www/ -- Or wherever you placed your path above
[root@localhost ~] mkdir svn
[root@localhost ~] cd svn
[root@localhost ~] svnadmin create repos
[root@localhost ~] chown -R apache:apache repos
[root@localhost ~] service httpd restart

```

### 将目录导入 SVN

创建一个示例文件夹结构以测试 SVN 操作。创建`mytestproj`目录，其下包含名为`main`、`configurations`和`resources`的子目录。在每个子目录中创建示例文件。

```
[root@localhost mytestproj]# svn import /tmp/mytestproj/ file:///var/www/svn/repos/mytestproj -m "Initial repository layout for mytestproj"
Adding         /tmp/mytestproj/main
Adding         /tmp/mytestproj/main/mainfile1.cfg
Adding         /tmp/mytestproj/configurations
Adding         /tmp/mytestproj/configurations/testconf1.cfg
Adding         /tmp/mytestproj/resources
Adding         /tmp/mytestproj/resources/testresources1.cfg
Committed revision 1.

```

在 Web 浏览器中验证仓库：`http://localhost/repos`。

### 从 SVN 检出

要从仓库检出源代码，请执行以下操作：

1.  启动`httpd`服务。

    ```
    [root@localhost testmit]# service httpd restart
    Stopping httpd:
    [  OK  ]
    Starting httpd: httpd: Could not reliably determine the server's fully qualified domain name, using localhost.localdomain for ServerName
    [  OK  ]

    ```

1.  检出源代码。

    ```
    [root@localhost testmit]# svn co http://localhost/repos/mytestproj
    Authentication realm: <http://localhost:80> Subversion repos
    Password for 'root':
    Authentication realm: <http://localhost:80> Subversion repos
    Username: mitesh
    Password for 'mitesh':xxxxxxxxx

    -----------------------------------------------------------------------
    ATTENTION!  Your password for authentication realm:

     <http://localhost:80> Subversion repos

    can only be stored to disk unencrypted! You are advised to configure your system so that Subversion can store passwords encrypted, if possible. See the documentation for details.

    ```

1.  您可以通过在`/root/.subversion/servers`中将`store-plaintext-passwords`选项的值设置为`yes`或`no`来避免未来出现此警告。

    ```
    -----------------------------------------------------------------------
    Store password unencrypted (yes/no)? no
    A    mytestproj/main
    A    mytestproj/main/mainfile1.cfg
    A    mytestproj/configurations
    A    mytestproj/configurations/testconf1.cfg
    A    mytestproj/options
    A    mytestproj/options/testopts1.cfg
    Checked out revision 1.

    ```

## 在 Windows 上安装 VisualSVN Server

1.  从[`www.visualsvn.com/server/download/`](https://www.visualsvn.com/server/download/)下载 VisualSVN 服务器。它允许您在 Windows 上安装和管理一个完全功能的 Subversion 服务器。

1.  执行`VisualSVN-Server-x.x.x-x64.msi`并按照向导安装 VisualSVN Server。

1.  打开 VisualSVN Server 管理器。

1.  创建一个新仓库，名为`JenkinsTest`。![Windows 上的 VisualSVN Server](img/3471_02_03.jpg)

1.  选择常规 Subversion 仓库并点击**下一步 >**。![Windows 上的 VisualSVN Server](img/3471_02_04.jpg)

1.  提供**仓库名称**并点击**下一步 >**。![Windows 上的 VisualSVN Server](img/3471_02_05.jpg)

1.  选择**单项目仓库**并点击**>**。![Windows 上的 VisualSVN Server](img/3471_02_06.jpg)

1.  根据您的需求选择仓库访问权限，然后点击**创建**。![Windows 上的 VisualSVN Server](img/3471_02_07.jpg)

1.  查看创建的仓库详情并点击**完成**。![Windows 上的 VisualSVN Server](img/3471_02_08.jpg)

1.  在 VisualSVN Server 管理器中验证新创建的仓库。![Windows 上的 VisualSVN Server](img/3471_02_09.jpg)

1.  在浏览器中验证仓库位置，如下所示：![Windows 上的 VisualSVN Server](img/3471_02_10.jpg)

1.  现在从[`sourceforge.net/projects/tortoisesvn/`](http://sourceforge.net/projects/tortoisesvn/)安装 SVN 客户端，以执行 SVN 操作。

让我们在 Eclipse 中创建一个示例 JEE 项目，以说明 SVN 和 Eclipse 的集成。

1.  打开 Eclipse，转到**文件**菜单并点击**动态 Web 项目**。![Windows 上的 VisualSVN Server](img/3471_02_11.jpg)

1.  将弹出一个对话框以创建一个**新动态 Web 项目**。![Windows 上的 VisualSVN Server](img/3471_02_12.jpg)

1.  为简单项目创建源文件和`build`文件。![Windows 上的 VisualSVN Server](img/3471_02_13.jpg)

1.  转到**应用程序目录**，右键点击，选择**TortoiseSVN**，然后从子菜单中选择**导入**。![Windows 上的 VisualSVN Server](img/3471_02_14.jpg)

1.  输入仓库 URL 并点击**确定**。![Windows 上的 VisualSVN Server](img/3471_02_15.jpg)

1.  它将把应用程序中的所有文件添加到 SVN，如下面的截图所示。![Windows 上的 VisualSVN Server](img/3471_02_16.jpg)

1.  通过在浏览器中访问 SVN 仓库来验证导入，如下所示：![Windows 上的 VisualSVN Server](img/3471_02_17.jpg)

# 将 Eclipse 与代码仓库集成

1.  打开 Eclipse IDE，转到**帮助**菜单并点击**安装新软件**。

1.  通过添加此 URL 添加仓库：[`subclipse.tigris.org/update_1.10.x`](http://subclipse.tigris.org/update_1.10.x)，然后选择所有包并点击**下一步 >**。![将 Eclipse 与代码仓库集成](img/3471_02_18.jpg)

1.  在向导中查看要安装的项和许可证协议。接受条款并点击**完成**。

1.  重启 Eclipse。转到**窗口**菜单，选择**显示视图**，点击**其他**，并找到 SVN 和 SVN 仓库。

1.  在 SVN 仓库区域，右键点击并选择**新建**；从子菜单中选择**仓库位置…**。![将 Eclipse 与代码仓库集成](img/3471_02_19.jpg)

1.  在 Eclipse 中添加一个新的 SVN 仓库，使用此 URL：`https://<Ip 地址/本地主机/主机名>/svn/JenkinsTest/`。

1.  点击**完成**。![将 Eclipse 与代码仓库集成](img/3471_02_20.jpg)

1.  验证 SVN 仓库。![将 Eclipse 与代码仓库集成](img/3471_02_21.jpg)

尝试将安装在 CentOS 上的 SVN 与 Eclipse IDE 集成，作为练习。

# 安装和配置 Ant

1.  从[`ant.apache.org/bindownload.cgi`](https://ant.apache.org/bindownload.cgi)下载 Ant 发行版并解压。

1.  设置`ANT_HOME`和`JAVA_HOME`环境变量。![安装和配置 Ant](img/3471_02_22.jpg)

Jenkins 中有一个选项可以自动安装 Ant 或 Maven。我们将在*Jenkins 中配置 Ant、Maven 和 JDK*部分学习这一点。

# 安装 Maven

从[`maven.apache.org/download.cgi`](https://maven.apache.org/download.cgi)下载 Maven 二进制 ZIP 文件，并将其提取到 Jenkins 安装的本地系统中。

![安装 Maven](img/3471_02_23.jpg)

# 在 Jenkins 中配置 Ant、Maven 和 JDK

1.  使用此 URL 在浏览器中打开 Jenkins 仪表板：`http://<ip_address>:8080/configure`。转到**管理 Jenkins**部分并点击**系统配置**。

1.  根据以下截图所示安装配置 Java：![在 Jenkins 中配置 Ant、Maven 和 JDK](img/3471_02_24.jpg)

1.  在同一页面上自动配置或安装 Ant，并配置 Maven。![在 Jenkins 中配置 Ant、Maven 和 JDK](img/3471_02_25.jpg)

# 安装和配置 Git

Git 是一个免费且开源的分布式版本控制系统。本节我们将尝试安装和配置 Git。

1.  在基于 CentOS 的系统中打开终端，并在终端中执行`yum install git`命令。

1.  成功安装后，使用`git --version`命令验证版本。

1.  使用`git config`命令提供用户信息，以便`commit`消息将附带正确的信息。

1.  提供姓名和电子邮件地址以嵌入提交中。

1.  要创建工作区环境，请在主目录中创建一个名为`git`的目录，然后在该目录内创建一个名为`development`的子目录。

    在终端中使用`mkdir -p ~/git/development ; cd ~/git/development`。

1.  将`AntExample1`目录复制到`development`文件夹中。

1.  使用`git init`命令将现有项目转换为工作区环境。

1.  初始化仓库后，添加文件和文件夹。![安装和配置 Git](img/3471_02_26.jpg)

1.  执行`git commit -m "初始提交" –a`进行提交。![安装和配置 Git](img/3471_02_27.jpg)

1.  验证 Git 仓库![安装和配置 Git](img/3471_02_28.jpg)

1.  在 Git 仓库中验证项目。![安装和配置 Git](img/3471_02_29.jpg)

# 在 Jenkins 中使用 Git 创建新构建作业

1.  在 Jenkins 仪表板上，点击**管理 Jenkins**并选择**管理插件**。点击**可用**标签并在搜索框中输入`github`插件。

1.  勾选复选框并点击按钮，**立即下载并在重启后安装**。

1.  重启 Jenkins。![在 Jenkins 中使用 Git 创建新构建作业](img/3471_02_30.jpg)

1.  创建新的**自由风格项目**。提供**项目名称**并点击**确定**。![在 Jenkins 中使用 Git 创建新构建作业](img/3471_02_31.jpg)

1.  在**源代码管理**部分配置**Git**。![在 Jenkins 中使用 Git 创建新构建作业](img/3471_02_32.jpg)

1.  通过点击**添加构建步骤**添加**调用 Ant**构建步骤。![在 Jenkins 中使用 Git 创建新构建作业](img/3471_02_33.jpg)

1.  执行构建。![在 Jenkins 中使用 Git 创建新构建作业](img/3471_02_34.jpg)

1.  点击**控制台输出**查看构建进度。![在 Jenkins 中使用 Git 创建新构建作业](img/3471_02_35.jpg)

1.  构建成功后，验证构建作业中的**工作区**。![在 Jenkins 中使用 Git 创建新构建作业](img/3471_02_36.jpg)

1.  完成！

# 自测题

Q1. 在哪里设置`JAVA_HOME`和`JRE_HOME`环境变量？

1.  `/root/ .bash_profile`

1.  `/root/ .env_profile`

1.  `/root/ .bash_variables`

1.  `/root/ .env_variables`

Q2. 哪些是有效的 SVN 操作？

1.  `svn import /tmp/mytestproj/`

1.  `svn co http://localhost/repos/mytestproj`

1.  上述两者皆是。

Q3\. 在 Jenkins 中，您在哪里配置 Java 和 Ant？

1.  前往**管理 Jenkins**部分，然后点击**系统配置**。

1.  前往**管理 Jenkins**部分，然后点击**全局配置**。

# 总结

好极了！我们已到达本章的结尾。我们介绍了如何通过设置本地 CentOS 仓库、在 CentOS 和 Windows 上安装如 SVN 的代码仓库以及构建工具 Ant 来准备持续集成的环境。我们还详细说明了如何在 Jenkins 中配置仓库和构建工具。最后，我们探讨了如何将集成开发环境与代码仓库集成，以便进行高效的开发和简便的`commit`操作，从而促进部署流水线流程。
