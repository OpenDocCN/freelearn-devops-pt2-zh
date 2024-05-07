# 复杂环境

到目前为止，我们已经了解了如何开发 Ansible 剧本并对其进行测试。最后一个方面是如何将剧本发布到生产环境。在大多数情况下，发布剧本到生产环境之前，你将需要处理多个环境。这类似于你的开发人员编写的软件。许多公司有多个环境，通常你的剧本将按照以下步骤进行：

+   开发环境

+   测试环境

+   阶段环境

+   生产

一些公司以不同的方式命名这些环境，有些公司还有额外的环境，比如所有软件都必须在进入生产环境之前通过认证的认证环境。

当你编写你的剧本并设置角色时，我们强烈建议你从一开始就牢记环境的概念。与你的软件和运维团队交流，了解你的设置必须满足多少个环境可能是值得的。我们将列举一些方法，并提供你可以在你的环境中遵循的示例。

本章将涵盖以下主题：

+   基于 Git 分支的代码

+   软件分发策略

+   使用修订控制系统部署 Web 应用程序

+   使用 RPM 包部署 Web 应用程序

+   使用 RPM 打包编译软件

# 技术要求

要能够跟随本章的示例，你需要一台能够构建 RPM 包的 UNIX 机器。我的建议是安装 Fedora 或 CentOS（无论是裸金属还是虚拟机）。

你可以从本书的 GitHub 存储库下载所有文件，网址为[`github.com/PacktPublishing/Learning-Ansible-2.X-Third-Edition/tree/master/Chapter09`](https://github.com/PacktPublishing/Learning-Ansible-2.X-Third-Edition/tree/master/Chapter09)。

# 基于 Git 分支的代码

假设你要照顾四个环境，它们如下：

+   开发

+   测试

+   阶段

+   生产

在基于 Git 分支的方法中，你将每个分支拥有一个环境。你总是首先对**开发**进行更改，然后将这些更改提升到**测试**（在 Git 中合并或挑选，以及标记提交），**阶段**和**生产**。在这种方法中，你将拥有一个单一的清单文件，一组变量文件，最后，针对每个分支的角色和剧本的一堆文件夹。

# 具有多个文件夹的单一稳定分支

在这种方法中，您将始终保持开发和主分支。初始代码提交到开发分支，一旦稳定，将其推广到主分支。在主分支上存在的相同角色和 playbooks 将在所有环境中运行。另一方面，您将为每个环境有单独的文件夹。让我们看一个例子。我们将展示如何针对两个环境（暂存和生产）拥有单独的配置和清单。您可以根据自己的情况来扩展到您使用的所有环境。首先，让我们看一下`playbooks/variables.yaml`中的 playbook，它将在这些多个环境中运行，并具有以下内容。完整代码可在 GitHub 上查看：

```
- hosts: web 
  user: vagrant 
  tasks: 
    - name: Print environment name 
      debug: 
        var: env 
    - name: Print db server url 
      debug: 
        var: db_url 
    - name: Print domain url 
      debug: 
        var: domain 
...
```

正如您所见，在这个 playbook 中有两组任务：

+   运行在 DB 服务器上的任务

+   运行在 Web 服务器上的任务

还有一个额外的任务来打印特定环境中所有服务器共有的环境名称。我们也将有两个不同的清单文件。

第一个将被称为`inventory/production`，内容如下：

```
[web] 
ws01.fale.io 
ws02.fale.io 

[db] 
db01.fale.io 

[production:children] 
db 
web 
```

第二个将被称为`inventory/staging`，内容如下：

```
[web] 
ws01.staging.fale.io 
ws02.staging.fale.io 

[db] 
db01.staging.fale.io 

[staging:children] 
db 
web
```

如您所见，在每个环境中`web`部分有两台机器，`db`部分有一台。此外，我们对阶段和生产环境有不同的机器组。附加部分`[ENVIRONMENT:children]`允许您创建一组组。这意味着在`ENVIRONMENT`部分定义的任何变量都将应用于`db`和`web`组，除非在各自的部分中覆盖。接下来看看如何在每个环境中分离变量值将是有趣的。

我们先从位于`inventory/group_vars/all`的所有环境相同的变量开始：

```
db_user: mysqluser 
```

两个环境中唯一相同的变量是`db_user`。

现在我们可以查看位于`inventory/group_vars/production`的特定于生产环境的变量：

```
env: production 
domain: fale.io 
db_url: db.fale.io 
db_pass: this_is_a_safe_password 
```

如果我们现在查看位于`inventory/group_vars/staging`的特定于阶段环境的变量，我们会发现与生产环境中相同的变量，但值不同：

```
env: staging 
domain: staging.fale.io 
db_url: db.staging.fale.io 
db_pass: this_is_an_unsafe_password 
```

我们现在可以验证我们收到了预期的结果。首先，我们将对暂存环境运行：

```
ansible-playbook -i inventory/staging playbooks/variables.yaml
```

我们应该会收到类似以下的输出。完整代码输出可在 GitHub 上查看：

```
PLAY [web] ***********************************************************

TASK [Gathering Facts] ***********************************************
ok: [ws01.staging.fale.io]
ok: [ws02.staging.fale.io]

TASK [Print environment name] ****************************************
ok: [ws01.staging.fale.io] => {
 "env": "staging"
}
ok: [ws02.staging.fale.io] => {
 "env": "staging"
}

TASK [Print db server url] *******************************************
ok: [ws01.staging.fale.io] => {
 "db_url": "db.staging.fale.io"
}
ok: [ws02.staging.fale.io] => {
 "db_url": "db.staging.fale.io"
}

...
```

现在我们可以针对生产环境运行：

```
ansible-playbook -i inventory/production playbooks/variables.yaml 
```

我们将收到以下结果：

```
PLAY [web] ***********************************************************

TASK [Gathering Facts] ***********************************************
ok: [ws02.fale.io]
ok: [ws01.fale.io]

TASK [Print environment name] ****************************************
ok: [ws01.fale.io] => {
 "env": "production"
}
ok: [ws02.fale.io] => {
 "env": "production"
}

TASK [Print db server url] *******************************************
ok: [ws01.fale.io] => {
 "db_url": "db.fale.io"
}
ok: [ws02.fale.io] => {
 "db_url": "db.fale.io"
}

...
```

您可以看到 Ansible 运行捕获了为暂存环境定义的所有相关变量。

如果您使用这种方法获得多个环境的稳定主分支，最好使用混合环境特定目录，`group_vars`和清单组来应对这种情况。

# 软件分发策略

部署应用程序可能是**信息与通信技术（ICT）**领域中最复杂的任务之一。这主要是因为它经常需要更改作为该应用程序某种程度上组成部分的大多数机器的状态。事实上，通常您会发现自己在部署过程中需要同时更改负载均衡器、分发服务器、应用服务器和数据库服务器的状态。新技术，如容器，正试图简化这些操作，但通常很难或不可能将传统应用程序移至容器中。

现在我们将分析各种软件分发策略以及 Ansible 如何帮助每一种。

# 从本地机器复制文件

这可能是最古老的软件分发策略了。其思想是将文件放在本地机器上（通常用于开发代码），一旦更改完成，文件的副本就会被放在服务器上（通常通过 FTP）。这种部署代码的方式经常用于 Web 开发，其中的代码（通常是 PHP）不需要任何编译。

由于多个问题，应避免使用这种分发策略：

+   回滚很困难。

+   无法跟踪各个部署的更改。

+   没有部署历史记录。

+   在部署过程中很容易出错。

尽管这种分发策略可以非常容易地通过 Ansible 自动化，我强烈建议立即转向其他能够让您拥有更安全的分发策略的策略。

# 带有分支的版本控制系统

许多公司正在使用这种技术来分发他们的软件，主要用于非编译软件。这种技术背后的理念是设置服务器使用代码库的本地副本。通过 SVN，这是可能的，但不是很容易正确管理，而 Git 允许这种技术的简化，使其非常受欢迎。

这种技术与我们刚刚看到的技术相比有很大的优势；其中主要优势如下：

+   易于回滚

+   非常容易获取更改历史记录

+   非常容易的部署（特别是如果使用 Git）

另一方面，这种技术仍然存在多个缺点：

+   没有部署历史记录

+   编译软件困难

+   可能存在安全问题

我想更详细地讨论一下您可能会遇到的这种技术的潜在安全问题。非常诱人的做法是直接在用于分发内容的文件夹中下载您的 Git 存储库，因此，如果这是一个 Web 服务器，那么这将是`/var/www/`文件夹。这样做有明显的优势，因为要部署，您只需要执行`git pull`。缺点是 Git 将创建`/var/www/.git`文件夹，其中包含整个 Git 存储库（包括历史记录），如果没有得到妥善保护，将可以被任何人自由下载。

Alexa 排名前 100 万的网站中约有 1%的网站可以公开访问 Git 文件夹，所以如果您想使用这种分发策略，一定要非常小心。

# 带有标签的修订控制系统

使用稍微复杂但具有一些优点的修订控制系统的另一种方法是利用标记系统。此方法要求您每次进行新部署时都要打标签，然后在服务器上检查特定的标签。

这具有上一种方法的所有优点，并加入了部署历史记录。已编译的软件问题和可能的安全问题与上一种方法相同。

# RPM 软件包

部署软件的一种非常常见的方法（主要用于已编译的应用程序，但对于非编译的应用程序也有优势）是使用某种打包系统。一些语言，如 Java，已经包含了系统（Java 的情况下是 WAR），但也有可以用于任何类型的应用程序的打包系统，比如**RPM 软件包管理器**（**RPM**）。RPM 最初由 Erik Troan 和 Marc Ewing 开发，并于 1997 年发布用于 Red Hat Linux。自那时起，它已被许多 Linux 发行版采用，并且是 Linux 世界中分发软件的两种主要方式之一，另一种是 DEB。这些系统的缺点是它们比以前的方法稍微复杂一些，但是这些系统可以提供更高级别的安全性，以及版本控制。此外，这些系统很容易嵌入到 CI/CD 流水线中，因此实际复杂性远低于乍看之下所见。

# 准备环境

要查看我们如何以我们在*软件分发策略*部分讨论的方式部署代码，我们将需要一个环境，显然我们将使用 Ansible 创建它。首先，为了确保我们的角色被正确加载，我们需要`ansible.cfg`文件，内容如下：

```
[defaults] 
roles_path = roles
```

然后，我们需要`playbooks/groups/web.yaml`文件来正确引导我们的 Web 服务器：

```
- hosts: web 
  user: vagrant 
  roles: 
    - common 
    - webserver 
```

正如您可以从前面的文件内容中想象的那样，我们将需要创建`common`和`webserver`角色，它们与我们在第四章中创建的角色非常相似，*处理复杂的部署*。我们将从`roles/common/tasks/main.yaml`文件开始，内容如下。完整的代码可在 GitHub 上找到：

```
- name: Ensure EPEL is enabled 
  yum: 
    name: epel-release 
    state: present 
  become: True 
- name: Ensure libselinux-python is present 
  yum: 
    name: libselinux-python 
    state: present 
  become: True 
- name: Ensure libsemanage-python is present 
  yum: 
    name: libsemanage-python 
    state: present 
  become: True 
...
```

这是`motd`模板在`roles/common/templates/motd`中的模板：

```
                This system is managed by Ansible 
  Any change done on this system could be overwritten by Ansible 

OS: {{ ansible_distribution }} {{ ansible_distribution_version }} 
Hostname: {{ inventory_hostname }} 
eth0 address: {{ ansible_eth0.ipv4.address }} 

            All connections are monitored and recorded 
    Disconnect IMMEDIATELY if you are not an authorized user
```

现在我们可以转移到`webserver`角色——更具体地说是`roles/webserver/tasks/main.yaml`文件。完整的代码文件可以在 GitHub 上找到：

```
--- 
- name: Ensure the HTTPd package is installed 
  yum: 
    name: httpd 
    state: present 
  become: True 
- name: Ensure the HTTPd service is enabled and running 
  service: 
    name: httpd 
    state: started 
    enabled: True 
  become: True 
- name: Ensure HTTP can pass the firewall 
  firewalld: 
    service: http 
    state: enabled 
    permanent: True 
    immediate: True 
  become: True 
...
```

我们还需要在`roles/webserver/handlers/main.yaml`中创建处理程序，内容如下：

```
--- 
- name: Restart HTTPd 
  service: 
    name: httpd 
    state: restarted 
  become: True
```

我们在`roles/webserver/templates/index.html.j2`文件中添加以下内容：

```
<html> 
    <body> 
        <h1>Hello World!</h1> 
        <p>This page was created on {{ ansible_date_time.date }}.</p> 
        <p>This machine can be reached on the following IP addresses</p> 
        <ul> 
{% for address in ansible_all_ipv4_addresses %} 
            <li>{{ address }}</li> 
{% endfor %} 
        </ul> 
    </body> 
</html> 
```

最后，我们需要触及`roles/webserver/files/website.conf`文件，暂时将其留空，但它必须存在。

现在我们可以预配一对 CentOS 机器（我预配了`ws01.fale.io`和`ws02.fale.io`），并确保清单正确。我们可以通过运行它们的组播放本来配置这些机器：

```
ansible-playbook -i inventory/production playbooks/groups/web.yaml 
```

我们将收到以下输出。完整的代码输出可在 GitHub 上找到：

```
PLAY [web] ***********************************************************

TASK [Gathering Facts] ***********************************************
ok: [ws01.fale.io]
ok: [ws02.fale.io]

TASK [common : Ensure EPEL is enabled] *******************************
ok: [ws02.fale.io]
ok: [ws01.fale.io]

TASK [common : Ensure libselinux-python is present] ******************
ok: [ws02.fale.io]
ok: [ws01.fale.io]

TASK [common : Ensure libsemanage-python is present] *****************
ok: [ws01.fale.io]
ok: [ws02.fale.io]

TASK [common : Ensure we have last version of every package] *********
changed: [ws02.fale.io]
changed: [ws01.fale.io]

...
```

现在，我们可以在端口`80`上指向我们的节点，检查是否如预期显示了 HTTPd 页面。既然我们已经有了基本的 Web 服务器运行，我们现在可以专注于部署 Web 应用程序。

# 使用修订控制系统部署 Web 应用

现在我们将从修订控制系统（Git）直接将 Web 应用程序首次部署到我们的服务器上，使用 Ansible。因此，我们将部署一个简单的 PHP 应用程序，只由一个单独的 PHP 页面组成。源代码可在以下存储库找到：[`github.com/Fale/demo-php-app`](https://github.com/Fale/demo-php-app)。

要部署它，我们将需要将以下代码放置在`playbooks/manual/rcs_deploy.yaml`中：

```
- hosts: web 
  user: vagrant 
  tasks:
    - name: Ensure git is installed
      yum:
        name: git
        state: present 
      become: True
    - name: Install or update website 
      git: 
        repo: https://github.com/Fale/demo-php-app.git 
        dest: /var/www/application 
      become: True
```

现在我们可以使用以下命令运行部署器：

```
ansible-playbook -i inventory/production/playbooks/manual/rcs_deploy.yaml 
```

这是预期的结果：

```
PLAY [web] ***********************************************************

TASK [Gathering Facts] ***********************************************
ok: [ws02.fale.io]
ok: [ws01.fale.io]

TASK [Ensure git is installed] ***************************************
changed: [ws01.fale.io]
changed: [ws02.fale.io]

TASK [Install or update website] *************************************
changed: [ws02.fale.io]
changed: [ws01.fale.io]

PLAY RECAP ***********************************************************
ws01.fale.io                  : ok=3 changed=2 unreachable=0 failed=0 
ws02.fale.io                  : ok=3 changed=2 unreachable=0 failed=0
```

目前，我们的应用程序还无法访问，因为我们没有 HTTPd 规则来访问该文件夹。为了实现这一点，我们将需要更改`roles/webserver/files/website.conf`文件，内容如下：

```
<VirtualHost *:80> 
    ServerName app.fale.io 
    DocumentRoot /var/www/application 
    <Directory /var/www/application> 
        Options None 
    </Directory> 
    <DirectoryMatch ".git*"> 
        Require all denied 
    </DirectoryMatch> 
</VirtualHost>
```

正如您所见，我们只向通过`app.fale.io`URL 到达我们服务器的用户显示此应用程序，而不是向所有人。这将确保所有用户都拥有一致的体验。此外，您可以看到我们阻止所有对`.git`文件夹（及其所有内容）的访问。这是出于我们在本章的*软件分发策略*部分提到的安全原因。

现在我们可以重新运行 Web 播放本以确保我们的 HTTPd 配置被传播：

```
ansible-playbook -i inventory/production playbooks/groups/web.yaml 
```

这是我们将要收到的结果。完整的代码输出可在 GitHub 上找到：

```
PLAY [web] ***********************************************************

TASK [Gathering Facts] ***********************************************
ok: [ws01.fale.io]
ok: [ws02.fale.io]

TASK [common : Ensure EPEL is enabled] *******************************
ok: [ws02.fale.io]
ok: [ws01.fale.io]

TASK [common : Ensure libselinux-python is present] ******************
ok: [ws01.fale.io]
ok: [ws02.fale.io]

TASK [common : Ensure libsemanage-python is present] *****************
ok: [ws01.fale.io]
ok: [ws02.fale.io]
 ...
```

您现在可以检查并确保一切正常运行。

我们已经看到了如何从 Git 获取源代码并将其部署到 Web 服务器，以便及时提供给我们的用户。现在，我们将深入研究另一种分发策略：使用 RPM 软件包部署 Web 应用程序。

# 使用 RPM 软件包部署 Web 应用程序

要部署 RPM 软件包，我们首先需要创建它。为此，我们需要的第一件事是一个**SPEC 文件**。

# 创建 SPEC 文件

我们需要做的第一件事是创建一个 SPEC 文件，这是一个指导`rpmbuild`如何实际创建 RPM 软件包的配方。我们将把 SPEC 文件定位在`spec/demo-php-app.spec`中。以下是代码片段内容，完整代码可在 GitHub 上找到：

```
%define debug_package %{nil} 
%global commit0 b49f595e023e07a8345f47a3ad62a6f50f03121e 
%global shortcommit0 %(c=%{commit0}; echo ${c:0:7}) 

Name: demo-php-app 
Version: 0 
Release: 1%{?dist} 
Summary: Demo PHP application 

License: PD 
URL: https://github.com/Fale/demo-php-app 
Source0: %{url}/archive/%{commit0}.tar.gz#/%{name}-%{shortcommit0}.tar.gz 

%description 
This is a demo PHP application in RPM format 
 ...
```

在继续之前，让我们看看各个部分的作用和含义：

```
%define debug_package %{nil} 
%global commit0 b49f595e023e07a8345f47a3ad62a6f50f03121e 
%global shortcommit0 %(c=%{commit0}; echo ${c:0:7}) 
```

前三行是变量声明。

第一步将禁用调试包的生成。默认情况下，`rpmbuild`每次都会创建一个调试包并包含所有调试符号，但在本例中我们没有任何调试符号，因为我们没有进行任何编译。

第二个将提交的哈希放入`commit0`变量。第三个计算`shortcommit0`的值，该值计算为`commit0`字符串的前八个字符：

```
Name:       demo-php-app 
Version:    0 
Release:    1%{?dist} 
Summary:    Demo PHP application 

License:    PD 
URL:        https://github.com/Fale/demo-php-app 
Source0:    %{url}/archive/%{commit0}.tar.gz#/%{name}-%{shortcommit0}.tar.gz 
```

在第一行中，我们声明名称、版本、发布编号和摘要。版本和发布的区别在于版本是上游版本，而发布是上游版本的 SPEC 版本。

许可证是源许可证，而不是 SPEC 许可证。URL 用于跟踪上游网站。`source0`字段用于`rpmbuild`查找源文件的名称（如果存在多个文件，我们可以使用`source1`、`source2`等）。此外，如果源字段是有效的 URI，则可以使用`spectool`来自动下载它们。这是打包到 RPM 软件包中的软件的`description`：

```
%description 
This is a demo PHP application in RPM format
```

`prep`阶段是源文件解压缩和最终应用补丁的阶段。`%autosetup`将解压缩第一个源文件，并应用所有补丁。在这部分，您还可以执行在构建阶段前需要执行的其他操作，其目的是为构建阶段准备环境：

```
%prep 
%autosetup -n %{name}-%{commit0}
```

在这里，我们将列出所有构建阶段的操作。在我们的情况下，我们的源代码不需要编译，因此为空：

```
%build
```

在`install`阶段，我们将文件放入`%{buildroot}`文件夹，模拟目标文件系统：

```
%install 
mkdir -p %{buildroot}/var/www/application 
ls -alh 
cp index.php %{buildroot}/var/www/application 
```

`files`部分需要声明要放入软件包中的文件：

```
%files 
%dir /var/www/application 
/var/www/application/index.php 
```

`changelog`用于跟踪谁何时发布了新版本以及带有哪些更改：

```
%changelog  
* Sun Feb 24 2019 Fabio Alessandro Locati - 0.1
- Initial packaging 
```

现在我们有了 SPEC 文件，我们需要构建它。为此，我们可以使用生产机器，但这样会增加对该机器的攻击面，所以最好避免。构建 RPM 软件的多种方式。主要的四种方式如下：

+   手动

+   使用 Ansible 自动化手动方式

+   Jenkins

+   Koji

让我们简要看一下区别。

# 手动构建 RPM 包

构建 RPM 软件包的最简单方式是以手动方式进行。

最大的优势在于您只需要几个简单的步骤来安装`build`，因此许多刚开始使用 RPM 的人都从这里开始。缺点是这个过程将是手动的，因此人为的错误可能会破坏结果。此外，手动构建非常难以审计，因为唯一可审计的部分是输出而非过程本身。

要构建 RPM 软件包，您需要一个 Fedora 或 EL（Red Hat Enterprise Linux，CentOS，Scientific Linux，Oracle Enterprise Linux）系统。如果您使用 Fedora，您需要执行以下命令来安装所有必需的软件：

```
sudo dnf install -y fedora-packager 
```

如果您正在运行 EL 系统，则需要执行以下命令：

```
sudo yum install -y mock rpm-build spectool 
```

无论哪种情况，您都需要将要使用的用户添加到 `mock` 组中，为此，您需要执行以下操作：

```
sudo usermod -a -G mock [yourusername] 
```

Linux 在登录时加载用户，因此要应用组更改，您需要重新启动会话。

此时，我们可以将 SPEC 文件复制到文件夹中（通常情况下，`$HOME` 是一个不错的选择），然后执行以下操作：

```
mkdir -p ~/rpmbuild/SOURCES 
```

这将创建所需的 `$HOME/rpmbuild/SOURCES` 文件夹。 `-p` 选项将自动创建路径中缺失的所有文件夹。我们使用 `spectool` 下载源文件并将其放置在适当的目录中。 `spectool` 将自动从 SPEC 文件中获取 URL，因此我们不必记住它：

```
spectool -R -g demo-php-app.spec 
```

现在我们需要创建一个 `src.rpm` 文件，为此，我们可以使用 `rpmbuild`：

```
rpmbuild -bs demo-php-app.spec
```

此命令将输出类似于以下内容：

```
Wrote: /home/fale/rpmbuild/SRPMS/demo-php-app-0-1.fc28.src.rpm
```

名称中可能存在一些小差异；例如，您可能具有不同于 Fedora 24 的 `$HOME` 文件夹，如果您使用的是 Fedora 24 以外的其他版本，则可能有 `fc24` 以外的内容。此时，我们可以使用以下代码创建二进制文件：

```
mock -r epel-7-x86_64 /home/fale/rpmbuild/SRPMS/demo-php-app-0-1.fc28.src.rpm 
```

Mock 允许我们在干净的环境中构建 RPM 包，并且还由于 `-r` 选项而允许我们构建不同版本的 Fedora、EL 和 Mageia。该命令将给出非常长的输出，我们在此不涵盖，但在最后几行中有有用的信息。如果一切都构建正确，这是您应该看到的最后几行：

```
Wrote: /builddir/build/RPMS/demo-php-app-0-1.el7.centos.x86_64.rpm
Executing(%clean): /bin/sh -e /var/tmp/rpm-tmp.d4vPhr
+ umask 022
+ cd /builddir/build/BUILD
+ cd demo-php-app-b49f595e023e07a8345f47a3ad62a6f50f03121e
+ /usr/bin/rm -rf /builddir/build/BUILDROOT/demo-php-app-0-1.el7.centos.x86_64
+ exit 0
Finish: rpmbuild demo-php-app-0-1.fc28.src.rpm
Finish: build phase for demo-php-app-0-1.fc28.src.rpm
INFO: Done(/home/fale/rpmbuild/SRPMS/demo-php-app-0-1.fc28.src.rpm) Config(epel-7-x86_64) 0 minutes 58 seconds
INFO: Results and/or logs in: /var/lib/mock/epel-7-x86_64/result
Finish: run 
```

倒数第二行包含您可以找到结果的路径。如果您在该文件夹中查找，应该会找到以下文件：

```
drwxrwsr-x. 2 fale mock 4.0K Feb 24 12:26 .
drwxrwsr-x. 4 root mock 4.0K Feb 24 12:25 ..
-rw-rw-r--. 1 fale mock 4.6K Feb 24 12:26 build.log
-rw-rw-r--. 1 fale mock 3.3K Feb 24 12:26 demo-php-app-0-1.el7.centos.src.rpm
-rw-rw-r--. 1 fale mock 3.1K Feb 24 12:26 demo-php-app-0-1.el7.centos.x86_64.rpm
-rw-rw-r--. 1 fale mock 184K Feb 24 12:26 root.log
-rw-rw-r--. 1 fale mock  792 Feb 24 12:26 state.log
```

在编译过程中出现问题时，这三个日志文件非常有用。 `src.rpm` 文件将是我们使用第一个命令创建的 `src.rpm` 文件的副本，而 `x86_64.rpm` 文件是我们创建的 mock 文件，也是我们需要在机器上安装的文件。

# 使用 Ansible 构建 RPM 包

由于手动执行所有这些步骤可能会很长、乏味且容易出错，因此我们可以使用 Ansible 自动化它们。生成的 playbook 可能不是最清晰的，但可以以可重复的方式执行所有操作。

出于这个原因，我们将从头开始构建一个新的机器。我将这台机器称为 `builder01.fale.io`，我们还将更改 `inventory/production` 文件以匹配此更改：

```
[web] 
ws01.fale.io 
ws02.fale.io 

[db] 
db01.fale.io 

[builders] 
builder01.fale.io 

[production:children] 
db 
web 
builders 
```

在深入研究 `builders` 角色之前，我们需要对 `webserver` 角色进行一些更改，以启用新的存储库。首先是在 `roles/webserver/tasks/main.yaml` 文件末尾添加一个任务，其中包含以下代码：

```
- name: Install our private repository 
  copy: 
    src: privaterepo.repo 
    dest: /etc/yum.repos.d/privaterepo.repo 
  become: True
```

第二个更改实际上是使用以下内容创建 `roles/webserver/files/privaterepo.repo` 文件：

```
[privaterepo] 
name=Private repo that will keep our apps packages 
baseurl=http://repo.fale.io/ 
skip_if_unavailable=True 
gpgcheck=0 
enabled=1 
enabled_metadata=1
```

现在我们可以执行 `webserver` 组 playbook 以使更改生效，命令如下：

```
ansible-playbook -i inventory/production playbooks/groups/web.yaml 
```

应该显示如下输出。完整代码输出可在 GitHub 上找到：

```
PLAY [web] ***********************************************************

TASK [Gathering Facts] ***********************************************
ok: [ws02.fale.io]
ok: [ws01.fale.io]

TASK [common : Ensure EPEL is enabled] *******************************
ok: [ws02.fale.io]
ok: [ws01.fale.io]

TASK [common : Ensure libselinux-python is present] ******************
ok: [ws01.fale.io]
ok: [ws02.fale.io]

TASK [common : Ensure libsemanage-python is present] *****************
ok: [ws01.fale.io]
ok: [ws02.fale.io]

...
```

如预期的那样，唯一的变化是部署了我们新生成的仓库文件。

我们还需要为`builders`创建一个角色，其中包含位于`roles/builder/tasks/main.yaml`的`tasks`文件，内容如下。完整代码可在 GitHub 上找到：

```
- name: Ensure needed packages are present 
  yum: 
    name: '{{ item }}' 
    state: present 
  become: True 
  with_items: 
    - mock 
    - rpm-build 
    - spectool 
    - createrepo 
    - httpd 

- name: Ensure the user ansible is in the mock group 
  user: 
    name: ansible 
    groups: mock 
    append: True 
  become: True 

...
```

同样作为`builder`角色的一部分，我们需要`roles/builder/handlers/main.yaml`处理文件，内容如下：

```
- name: Restart HTTPd 
  service: 
    name: httpd 
    state: restarted 
  become: True 
```

如您从`tasks`文件中可以猜想的，我们还需要`roles/builder/files/repo.conf`文件，内容如下：

```
<VirtualHost *:80> 
    ServerName repo.fale.io 
    DocumentRoot /var/www/repo 
    <Directory /var/www/repo> 
        Options Indexes FollowSymLinks 
    </Directory> 
</VirtualHost>
```

我们还需要一个新的`group` playbook，位于`playbooks/groups/builders.yaml`，内容如下：

```
- hosts: builders 
  user: vagrant 
  roles: 
    - common 
    - builder 
```

现在我们可以创建主机本身，内容如下：

```
ansible-playbook -i inventory/production playbooks/groups/builders.yaml 
```

我们期望得到类似下面的结果：

```
PLAY [builders] ******************************************************

TASK [Gathering Facts] ***********************************************
ok: [builder01.fale.io]

TASK [common : Ensure EPEL is enabled] *******************************
changed: [builder01.fale.io]

TASK [common : Ensure libselinux-python is present] ******************
ok: [builder01.fale.io]

TASK [common : Ensure libsemanage-python is present] *****************
changed: [builder01.fale.io]

TASK [common : Ensure we have last version of every package] *********
changed: [builder01.fale.io]

TASK [common : Ensure NTP is installed] ******************************
changed: [builder01.fale.io]

TASK [common : Ensure the timezone is set to UTC] ********************
changed: [builder01.fale.io]

...
```

现在我们已经准备好基础设施的所有部分，可以创建`playbooks/manual/rpm_deploy.yaml`文件，内容如下。完整代码可在 GitHub 上找到：

```
- hosts: builders 
  user: vagrant 
  tasks: 
    - name: Copy SPEC file to user folder 
      copy: 
        src: ../../spec/demo-php-app.spec 
        dest: /home/vagrant
    - name: Ensure rpmbuild exists 
      file: 
        name: ~/rpmbuild 
        state: directory 
    - name: Ensure rpmbuild/SOURCES exists 
      file: 
        name: ~/rpmbuild/SOURCES 
        state: directory 
   ...
```

正如我们讨论过的，此 playbook 有许多不太干净的命令和 shell。将来可能有可能编写一个具有相同功能但使用模块的 playbook。大多数操作与我们在前一节讨论过的相同。新操作在最后; 实际上，在这种情况下，我们将生成的 RPM 文件复制到特定文件夹，我们调用`createrepo`在该文件夹中生成一个仓库，然后强制所有 Web 服务器更新生成的软件包至最新版本。

为确保应用程序的安全性，重要的是仓库仅在内部可访问，而不是公开的。

现在我们可以用以下命令运行 playbook：

```
ansible-playbook -i inventory/production playbooks/manual/rpm_deploy.yaml 
```

我们期望得到类似以下的结果。完整代码输出在 GitHub 上找到：

```
PLAY [builders] ******************************************************

TASK [setup] *********************************************************
ok: [builder01.fale.io]

TASK [Copy SPEC file to user folder] *********************************
changed: [builder01.fale.io]

TASK [Ensure rpmbuild exists] ****************************************
changed: [builder01.fale.io]

TASK [Ensure rpmbuild/SOURCES exists] ********************************
changed: [builder01.fale.io]

TASK [Download the sources] ******************************************
changed: [builder01.fale.io]

TASK [Ensure no SRPM files are present] ******************************
changed: [builder01.fale.io]

TASK [Build the SRPM file] *******************************************
changed: [builder01.fale.io]
...
```

# 使用 CI/CD 流水线构建 RPM 软件包

虽然本书未涉及此内容，但在更复杂的情况下，您可能希望使用 CI/CD 流水线来创建和管理 RPM 软件包。这两个主要的流水线基于两种不同类型的软件：

+   Koji

+   Jenkins

Koji 软件由 Fedora 社区和 Red Hat 开发。它根据 LGPL 2.1 许可证发布。这是目前由 Fedora、CentOS 以及许多其他公司和社区用来创建所有他们的 RPM 软件包（包括官方测试，也称为**临时构建**）的流水线。 Koji 默认情况下不会由提交触发；需要通过用户（通过 Web 界面或 CLI）进行**手动**调用。 Koji 将自动从 Git 下载 SPEC 文件的最新版本，从侧边缓存（这是可选的，但建议的）或原始位置下载源代码，并触发模拟构建。 Koji 仅支持模拟构建，因为它是唯一支持一致和可重复构建的系统。 Koji 可以永久存储所有输出的构件或根据配置的设置存储一段时间。这是为了确保非常高的审计级别。

Jenkins 是最常用的 CI/CD 管理器之一，也可以用于 RPM 流水线。其主要缺点是需要从头开始配置，这意味着需要更多的时间，但这意味着它具有更高的灵活性。此外，Jenkins 的一个重大优势是许多公司已经有了 Jenkins 实例，这使得设置和维护基础设施更容易，因为您可以重用您已经拥有的安装，因此您总体上不必管理较少的系统。

# 使用 RPM 打包构建编译软件

**RPM 打包**对于非二进制应用程序非常有用，并且对于二进制应用程序几乎是必需的。这也是因为非二进制和二进制情况之间的复杂性差异非常小。事实上，构建和安装将以完全相同的方式工作。唯一会改变的是 SPEC 文件。

让我们看看编写一个简单的用 C 编写的`Hello World!`应用程序所需的 SPEC 文件：

```
%global commit0 7c288b9d80a6ef525c0cca8a744b32e018eaa386 
%global shortcommit0 %(c=%{commit0}; echo ${c:0:7}) 

Name:           hello-world 
Version:        1.0 
Release:        1%{?dist} 
Summary:        Hello World example implemented in C 

License:        GPLv3+ 
URL:            https://github.com/Fale/hello-world 
Source0:        %{url}/archive/%{commit0}.tar.gz#/%{name}-%{shortcommit0}.tar.gz 

BuildRequires:  gcc 
BuildRequires:  make 

%description 
The description for our Hello World Example implemented in C 

%prep 
%autosetup -n %{name}-%{commit0} 

%build 
make %{?_smp_mflags} 

%install 
%make_install 

%files 
%license LICENSE 
%{_bindir}/hello 

%changelog 
* Sun Feb 24 2019 Fabio Alessandro Locati - 1.0-1 
- Initial packaging
```

正如你所看到的，这与我们在 PHP 演示应用程序中看到的非常相似。让我们看看其中的区别。

让我们稍微深入了解 SPEC 文件的各个部分：

```
%global commit0 7c288b9d80a6ef525c0cca8a744b32e018eaa386 
%global shortcommit0 %(c=%{commit0}; echo ${c:0:7}) 
```

正如你所看到的，我们没有禁用调试包的行。每次打包编译应用程序时，都应该让`rpm`创建调试符号包，这样在崩溃的情况下，调试和理解问题会更容易。

SPEC 文件的以下部分显示在这里：

```
Name:           hello-world 
Version:        1.0 
Release:        1%{?dist} 
Summary:        Hello World example implemented in C 

License:        GPLv3+ 
URL:            https://github.com/Fale/hello-world 
Source0:        %{url}/archive/%{commit0}.tar.gz#/%{name}-%{shortcommit0}.tar.gz 
```

正如你所看到的，此部分的更改仅是由于新包具有不同的名称和`URL`，但它们与这是一个可编译应用程序的事实无关联：

```
BuildRequires:  gcc 
BuildRequires:  make
```

在非编译应用程序中，我们不需要任何构建时存在的软件包，而在这种情况下，我们将需要`make`和`gcc`（编译器）应用程序。不同的应用程序可能需要不同的工具和/或库在构建时存在于系统中：

```
%description 
The description for our Hello World Example implemented in C 

%prep 
%autosetup -n %{name}-%{commit0} 

%build 
make %{?_smp_mflags} 
```

`description`是特定于包的，并不受包的编译影响。同样，`%prep`阶段也是如此。

在`%build`阶段，我们现在必须制作`%{?_smp_mflags}`。这是需要告诉`rpmbuild`实际运行`make`来构建我们的应用程序。`_smp_mflags`变量将包含一组参数，以优化编译为多线程：

```
%install 
%make_install 
```

在`%install`阶段，我们将发出`%make_install`命令。此宏将调用`make install`，并带有一组额外的参数，以确保库位于正确的文件夹中，以及二进制文件等等：

```
%files 
%license LICENSE 
%{_bindir}/hello 
```

在这种情况下，我们只需要将`hello`二进制文件放置在`%install`阶段的`buildroot`正确文件夹中，并添加包含许可证的`LICENSE`文件：

```
%changelog 
* Sun Feb 24 2019 Fabio Alessandro Locati - 1.0-1 
- Initial packaging 
```

`%changelog`与我们看到的其他 SPEC 文件非常相似，因为它不受编译的影响。

完成后，您可以将其放在 `spec/hello-world.spec` 中，并通过以下代码段将 `playbooks/manual/rpm_deploy.yaml` 调整为 `playbooks/manual/hello_deploy.yaml` 保存。全部代码可在 GitHub 上找到：

```
- hosts: builders 
  user: vagrant 
  tasks: 
    - name: Copy SPEC file to user folder 
      copy: 
        src: ../../spec/hello-world.spec 
        dest: /home/ansible 
    - name: Ensure rpmbuild exists 
      file: 
        name: ~/rpmbuild 
        state: directory 
    - name: Ensure rpmbuild/SOURCES exists 
      file: 
        name: ~/rpmbuild/SOURCES 
        state: directory 
    ...
```

正如您所见，我们唯一更改的是所有对 `demo-php-app` 的引用都替换为 `hello-world`。使用以下命令运行它：

```
ansible-playbook -i inventory/production playbooks/manual/hello_deploy.yaml
```

我们将得到以下结果。全部代码输出可在 GitHub 上找到：

```
PLAY [builders] ******************************************************

TASK [setup] *********************************************************
ok: [builder01.fale.io]

TASK [Copy SPEC file to user folder] *********************************
changed: [builder01.fale.io]

TASK [Ensure rpmbuild exists] ****************************************
ok: [builder01.fale.io]

TASK [Ensure rpmbuild/SOURCES exists] ********************************
ok: [builder01.fale.io]

TASK [Download the sources] ******************************************
changed: [builder01.fale.io]

TASK [Ensure no SRPM files are present] ******************************
changed: [builder01.fale.io]

TASK [Build the SRPM file] *******************************************
changed: [builder01.fale.io]

TASK [Execute mock] **************************************************
changed: [builder01.fale.io]

...
```

您最终可以创建一个接受要构建的包的名称作为参数的 Playbook，这样您就不需要为每个包创建不同的 Playbook。

# 部署策略

我们已经看到如何在您的环境中分发软件，所以现在，我们将谈论部署策略，即如何升级您的应用程序而不会使您的服务受到影响。

在更新期间可能遇到三种不同的问题：

+   更新推出期间的停机时间。

+   新版本存在问题。

+   新版本似乎工作正常，直到它失败。

第一个问题是每个系统管理员都知道的。在更新期间，您可能会重新启动一些服务，而在服务启动和结束之间的时间内，您的应用程序将无法在该机器上使用。为了解决这个问题，您需要具有智能负载均衡器的多台机器，该负载均衡器将在执行特定节点的升级之前，从可用节点池中删除指定节点，然后在节点升级后尽快将它们添加回去。

第二个问题可以通过多种方式预防。最清洁的方法是在 CI/CD 流水线中进行测试。事实上，这些问题很容易通过简单的测试找到。我们即将看到的方法也可以预防这种情况。

第三个问题迄今为止是最复杂的。许多次，甚至是全球范围内的问题，都是由这些问题引起的。通常，问题在于新版本存在一些性能问题或内存泄漏。由于大多数部署是在服务器负载最轻的时期完成的，一旦负载增加，性能问题或内存泄漏可能会导致服务器崩溃。

要能够正确使用这些方法，您必须能够确保您的软件可以接受回滚。有些情况下这是不可能的（即，在更新中删除了一个数据库表）。我们不会讨论如何避免这种情况，因为这是开发策略的一部分，与 Ansible 无关。

为了解决这些问题，通常使用两种常见的部署模式：**金丝雀部署** 和 **蓝绿部署**。

# 金丝雀部署

金丝雀部署是一种技术，涉及将你的一小部分机器（通常为 5%）更新到新版本，并指示负载均衡器仅将等量的流量发送到它。这有几个优点：

+   在更新期间，你的容量永远不会低于 95%

+   如果新版本完全失败，你会损失 5% 的容量。

+   由于负载均衡器在新旧版本之间分配流量，如果新版本有问题，只有你的用户的 5% 将看到问题。

+   只需要比预期负载多 5% 的容量

金丝雀部署能够避免我们提到的所有三个问题，而且额外开销很小（5%），并且在回滚时成本低廉（5%）。因此，许多大型公司都广泛使用这种技术。通常，为了确保用户在相近地理位置的体验相似，会根据地理位置选择用户是使用旧版本还是新版本。

当测试看起来成功时，可以逐步增加百分比，直到达到 100%。

可以以多种方式在 Ansible 中实现金丝雀部署。我建议的方式是最干净的方式，即使用清单文件，这样你会有如下内容：

```
[web-main] 
ws[00:94].fale.io 

[web-canary] 
ws[95:99].fale.io 

[web:children] 
web-main 
web-canary
```

通过这种方式，你可以在 web 组上设置所有变量（变量将是相同的，无论是什么版本的操作系统，或者至少应该是相同的），但你可以很容易地对金丝雀组、主要组或同时对两个组运行 playbook。另一个选项是创建两个不同的清单文件，一个用于金丝雀组，另一个用于主要组，组的名称相同，以便共享变量。

# 蓝/绿部署

蓝/绿部署与金丝雀部署非常不同，它有一些优点和一些缺点。主要优点如下：

+   更容易实现

+   允许更快的迭代

+   所有用户同时转移

+   回滚不会有性能下降

缺点中，主要的是需要比应用程序所需的机器多一倍。如果应用程序在云上运行（无论是私有、公共还是混合），这个缺点可以很容易地缓解，为部署扩展应用程序资源，然后再缩减它们。

在 Ansible 中实现蓝/绿部署非常简单。最简单的方法是创建两个不同的清单（一个用于蓝色，一个用于绿色），然后简单地管理你的基础设施，就像它们是不同的环境，如生产、暂存、开发等。

# 优化

有时，Ansible 感觉很慢，主要是因为要执行一个非常长的任务列表和/或有大量的机器。有多种原因和方法可以避免这种情况，我们将看一下其中的三种方式。

# 流水线

Ansible 默认较慢的原因之一是，对于每个模块的执行和每个主机，Ansible 将执行以下操作：

+   SSH 握手

+   执行任务

+   关闭 SSH 连接

正如你所看到的，这意味着如果你有 10 个任务要在单个远程服务器上执行，Ansible 将会打开（并关闭）10 次连接。由于 SSH 协议是一种加密协议，这使得 SSH 握手过程变得更长，因为两个部分必须每次都要协商密码。

Ansible 允许我们通过在 playbook 开始时初始化连接并在整个执行过程中保持连接处于活动状态来大幅减少执行时间，这样就不需要在每个任务中重新打开连接。在 Ansible 的发展过程中，这个特性已经多次改名，以及启用方式也有所变化。从 1.5 版本开始，它被称为**pipelining**，启用它的方式是在你的 `ansible.cfg` 文件中添加以下行：

```
pipelining=True 
```

这个功能默认没有启用的原因是许多发行版都带有 `sudo` 中的 `requiretty` 选项。Ansible 中的 pipelining 模式和 `sudo` 中的 `requiretty` 选项会冲突，并且会导致你的 playbook 失败。

如果你想要启用 pipelining 模式，请确保你的目标机器上已禁用了 `sudo requiretty` 模式。

# 使用 `with_items` 进行优化

如果你想要多次执行类似的操作，可以多次使用相同的任务并带有不同的参数，或者使用 `with_items` 选项。除了使你的代码更易于阅读和理解之外，`with_items` 还可以提高你的性能。一个例子是在安装软件包（即 `apt`，`dnf`，`yum`，`package` 模块）时，如果使用 `with_items`，Ansible 将执行一个命令，而不是如果不使用则为每个软件包执行一个命令。你可以想象，这可以帮助提高你的性能。

# 理解任务执行时发生的情况

即使你已经实施了我们刚刚讨论过的加快 playbook 执行速度的方法，你可能仍然会发现一些任务需要很长时间。即使对许多其他模块来说可能是可能的，对一些任务来说这是非常普遍的。通常会给你带来这种问题的模块如下：

+   包管理（即 `apt`，`dnf`，`yum`，`package`）

+   云机器创建（即 `digital_ocean`，`ec2`）

这种慢的原因通常不是特定于 Ansible 的。一个示例情况可能是，如果你使用了一个包管理模块来更新你的机器。这需要在每台机器上下载几十甚至几百兆的软件并安装大量软件。加快这种操作的方法是在你的数据中心中拥有一个本地仓库，并让所有的机器指向它，而不是你的发行版仓库。这将允许你的机器以更高的速度下载，并且不使用通常带宽有限或计量的公共连接。

了解模块在后台执行的操作对优化 playbook 的执行至关重要。

在云机器创建的情况下，Ansible 只需向所选的云提供商执行 API 调用，并等待机器准备就绪。DigitalOcean 的机器可能需要长达一分钟才能创建（其他云可能需要更长时间），因此 Ansible 将等待该时间。一些模块具有异步模式，以避免此等待时间，但您必须确保机器准备就绪后才能使用它；否则，使用创建的机器的模块将失败。

# 总结

在本章中，我们看到了如何使用 Ansible 部署应用程序，以及您可以使用的各种分发和部署策略。我们还看到了如何使用 Ansible 创建 RPM 包以及如何使用不同的方法优化 Ansible 的性能。

在下一章中，我们将学习如何在 Windows 机器上使用 Ansible，以及如何找到其他人编写的角色并如何使用它们，还有一个用于 Ansible 的用户界面。
