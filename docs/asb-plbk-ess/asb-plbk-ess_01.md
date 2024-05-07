# 设置学习环境

要最有效地使用本书并检查、运行和编写本书提供的练习中的代码，建立学习环境至关重要。虽然 Ansible 可以与任何类型的节点、虚拟机、云服务器或已安装操作系统和运行 SSH 服务的裸机主机一起使用，但首选模式是使用虚拟机。

在本次会话中，我们将涵盖以下主题：

+   理解学习环境

+   理解先决条件

+   安装和配置虚拟盒和 vagrant

+   创建虚拟机

+   安装 Ansible

+   使用示例代码

# 理解学习环境

我们假设大多数学习者都希望在本地设置环境，因此建议使用开源和免费的软件 VirtualBox 和 Vagrant，它们支持大多数桌面操作系统，包括 Windows、OSX 和 Linux。

理想的设置包括五台虚拟机，其目的解释如下。您还可以合并一些服务，例如，负载均衡器和 Web 服务器可以是同一主机：

+   **控制器**：这是唯一需要安装 Ansible 并充当控制器的主机。这用于从控制器启动`ansible-playbook`命令。

+   **数据库（Ubuntu）**：此主机配置有 Ansible 以运行 MySQL 数据库服务，并运行 Linux 的 Ubuntu 发行版。

+   **数据库（CentOS）**：此主机配置有 Ansible 以运行 MySQL 数据库服务，但它运行的是 Linux 的 CentOS 发行版。这是为了在为 Ansible 编写 MySQL 角色时测试多平台支持而添加的。

+   **Web 服务器**：此主机配置有 Ansible 以运行 Apache Web 服务器应用程序。

+   **负载均衡器**：此主机配置有 haproxy 应用程序，这是一个开源的 HTTP 代理服务。此主机充当负载均衡器，接受 HTTP 请求并将负载分布到可用的 Web 服务器上。

## 先决条件

有关先决条件、软件和硬件要求以及设置说明的最新说明，请参阅以下 GitHub 存储库：

[`github.com/schoolofdevops/ansible-playbook-essentials`](https://github.com/schoolofdevops/ansible-playbook-essentials)。

### 系统先决条件

适度配置的台式机或笔记本系统应该足以设置学习环境。以下是在软件和硬件上下文中推荐的先决条件：

| **处理器** | 2 个核心 |
| --- | --- |
| **内存** | 2.5 GB 可用内存 |
| **磁盘空间** | 20 GB 的可用空间 |
| **操作系统** | Windows，OS X（Mac），Linux |

## 基本软件

为了设置学习环境，我们建议使用以下软件：

+   **VirtualBox**：Oracle 的 virtualbox 是一种桌面虚拟化软件，可免费使用。它适用于各种操作系统，包括 Windows、OS X、Linux、FreeBSD、Solaris 等。它提供了一个 hypervisor 层，并允许在现有基础 OS 的顶部创建和运行虚拟机。本书提供的代码已经在 virtualbox 的 4.3x 版本上进行了测试。但是，任何与 vagrant 版本兼容的 virtualbox 版本都可以使用。

+   **Vagrant**：这是一个工具，允许用户轻松在大多数虚拟化程序和云平台上创建和共享虚拟环境，包括但不限于 virtualbox。它可以自动化任务，如导入镜像、指定资源（分配给 VM 的内存和 CPU 等）以及设置网络接口、主机名、用户凭据等。由于它提供了一个 Vagrant 文件形式的文本配置，虚拟机可以通过编程方式进行配置，使其易于与其他工具（如 **Jenkins**）一起使用，以自动化构建和测试流水线。

+   **Git for Windows**：尽管我们不打算使用 Git，它是一种版本控制软件，但我们使用此软件在 Windows 系统上安装 SSH 实用程序。Vagrant 需要在路径中可用的 SSH 二进制文件。Windows 未打包 SSH 实用程序，而 Git for Windows 是在 Windows 上安装它的最简单方法。还存在其他选择，如 **Cygwin**。

以下表格列出了用于开发提供的代码的软件版本 OS，附有下载链接：

| 软件 | 版本 | 下载链接 |
| --- | --- | --- |
| VirtualBox | 4.3.30 | [虚拟箱](https://www.virtualbox.org/wiki/Downloads) |
| Vagrant | 1.7.3 | [`www.vagrantup.com/downloads.html`](https://www.vagrantup.com/downloads.html) |
| Git for Windows | 1.9.5 | [`git-scm.com/download/win`](https://git-scm.com/download/win) |

建议学习者在继续之前下载、安装并参考相应的文档页面以熟悉这些工具。

## 创建虚拟机

安装基础软件后，您可以使用 Vagrant 来启动所需的虚拟机。Vagrant 使用一个名为 `Vagrantfile` 的规范文件，示例如下：

```
# -*- mode: ruby -*-
# vi: set ft=ruby :
# Sample Vagranfile to setup Learning Environment
# for Ansible Playbook Essentials

VAGRANTFILE_API_VERSION = "2"
Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.vm.box = "ansible-ubuntu-1204-i386"
  config.vm.box_url = "https://cloud-images.ubuntu.com/vagrant/precise/current/precise-server-cloudimg-i386-vagrant-disk1.box"
  config.vm.define "control" do |control|
    control.vm.network :private_network, ip: "192.168.61.10"
  end
  config.vm.define "db" do |db|
    db.vm.network :private_network, ip: "192.168.61.11"
  end
  config.vm.define "dbel" do |db|
    db.vm.network :private_network, ip: "192.168.61.14"
    db.vm.box = "opscode_centos-6.5-i386"
    db.vm.box = "http://opscode-vm-bento.s3.amazonaws.com/vagrant/virtualbox/opscode_centos-6.5_chef-provisionerless.box"
  end
  config.vm.define "www" do |www|
    www.vm.network :private_network, ip: "192.168.61.12"
  end
  config.vm.define "lb" do |lb|
    lb.vm.network :private_network, ip: "192.168.61.13"
  end
end
```

先前的 Vagrant 文件包含了设置五个虚拟机的规范，如本章开头所述，它们是 `control`、`db`、`dbel`、`www` 和 `lb`。

建议学习者使用以下说明创建和启动所需的虚拟机以设置学习环境：

1.  在系统的任何位置为学习环境设置创建一个目录结构，例如 `learn/ansible`。

1.  将先前提供的 `Vagrantfile` 文件复制到 `learn/ansible` 目录。现在目录树应如下所示：

    ```
                         learn
                            \_ ansible
                                  \_ Vagrantfile
    ```

    ### 注意

    `Vagrantfile` 文件包含了前一部分描述的虚拟机的规格。

1.  打开终端并进入 `learn/ansible` 目录。

1.  启动控制节点并登录到它，步骤如下：

    ```
    $ vagrant up control 
    $ vagrant ssh control

    ```

1.  从单独的终端窗口，在 `learn/ansible` 目录下，逐个启动剩余的虚拟机，步骤如下：

    ```
    $ vagrant up db
    $ vagrant up www
    $ vagrant up lb
    optionally (for centos based mysql configurations)
    $ vagrant up dbel 
    Optionally, to login to to the virtual machines as
    $ vagrant ssh db
    $ vagrant ssh www
    $ vagrant ssh lb
    optionally (for centos based mysql configurations)
    $ vagrant ssh dbel 

    ```

## 安装 Ansible 到控制节点

一旦虚拟机创建并启动，需要在控制节点上安装 Ansible。由于 Ansible 是无代理的，使用 SSH 传输来管理节点，因此在节点上除了确保 SSH 服务正在运行外，不需要进行额外的设置。要在控制节点上安装 Ansible，请参考以下步骤。这些说明特定适用于 Linux 的 Ubuntu 发行版，因为这是我们在控制节点上使用的操作系统。有关通用的安装说明，请参考以下页面：

[`docs.ansible.com/intro_installation.html`](http://docs.ansible.com/intro_installation.html).

步骤如下：

1.  使用以下命令登录到控制节点：

    ```
    # from inside learn/ansible directory 
    $ vagrant ssh control 

    ```

1.  使用以下命令更新仓库缓存：

    ```
    $ sudo apt-get update

    ```

1.  安装先决软件和仓库：

    ```
    # On Ubuntu 14.04 and above 
    $ sudo apt-get install -y software-properties-common
    $ sudo apt-get install -y python-software-properties
    $ sudo apt-add-repository ppa:ansible/ansible

    ```

1.  添加新仓库后，请更新仓库缓存，步骤如下：

    ```
    $ sudo apt-get update 

    ```

1.  使用以下命令安装 Ansible：

    ```
    $ sudo apt-get install -y ansible 

    ```

1.  使用以下命令验证 Ansible：

    ```
    $ ansible --version
    [sample output]
    vagrant@vagrant:~$ ansible --version
    ansible 1.9.2
     configured module search path = None

    ```

## 使用示例代码

本书提供的示例代码按章节号进行划分。以章节号命名的目录包含代码在相应章节结束时的快照。建议学习者独立创建自己的代码，并将示例代码用作参考。此外，如果读者跳过一个或多个章节，他们可以将前一章节的示例代码用作基础。

例如，在使用 Chapter 6 *迭代控制结构 - 循环* 时，您可以将 Chapter 5 *控制执行流程 - 条件* 的示例代码用作基础。

### 提示

**下载示例代码**

您可以从您在 [`www.packtpub.com`](http://www.packtpub.com) 购买的所有 Packt 图书的帐户中下载示例代码文件。如果您在其他地方购买了本书，您可以访问 [`www.packtpub.com/support`](http://www.packtpub.com/support) 并注册以直接通过电子邮件接收文件。
