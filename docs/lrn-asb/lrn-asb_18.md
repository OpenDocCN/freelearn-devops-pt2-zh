# 第十八章：评估

# 第二章，安装和运行 Ansible

1.  安装 Ansible 使用 pip 的命令是什么？

`sudo -H pip install ansible`

1.  真或假：在使用 Homebrew 时，您可以选择要安装或回滚到的确切 Ansible 版本。

假

1.  真或假：Windows 子系统在虚拟机中运行。

假

1.  列出三个 Vagrant 支持的虚拟化程序。

VirtualBox，VMware 和 Hyper-V

1.  状态并解释主机清单是什么。

主机清单是一个主机列表，以及用于访问它们的选项，Ansible 将针对它们

1.  真或假：YAML 文件中的缩进对于它们的执行非常重要，而不仅仅是装饰性的。

真

# 第三章，Ansible 命令

1.  在本章中提供有关主机清单的信息的命令中，哪个是默认与 Ansible 一起提供的？

`ansible-inventory`命令

1.  真或假：使用 Ansible Vault 加密字符串的变量文件将与低于 2.4 版本的 Ansible 一起使用。

假

1.  您将运行什么命令来获取如何调用`yum`模块作为任务的示例？

您将使用`ansible-doc`命令

1.  解释为什么您希望针对清单中的主机运行单个模块。

如果您想要使用 Ansible 以受控的方式针对多个主机运行临时命令，您将使用单个模块。

# 第四章，部署 LAMP 堆栈

1.  您会使用哪个 Ansible 模块来下载和解压缩 zip 文件？

该模块称为`unarchive`

1.  真或假：在**`roles/rolename/default/`**文件夹中找到的变量会覆盖同一变量的所有其他引用。

假

1.  解释如何向我们的 playbook 添加第二个用户？

通过向用户变量添加第二行，例如：`{ name: "user2", group: "lamp", state: "present", key: "{{ lookup('file', '~/.ssh/id_rsa.pub') }}" }`

1.  真或假：您只能从一个任务中调用一个处理程序。

假

# 第五章，部署 WordPress

1.  在`setup`模块执行期间收集的哪个事实可以告诉我们的 playbook 目标主机有多少处理器？

事实是`ansible_processor_count`

1.  真或假：在`lineinfile`模块中使用`backref`可以确保如果正则表达式不匹配，则不会应用任何更改。

真

1.  解释为什么我们希望在 playbook 中构建逻辑来检查 WordPress 是否已经安装。

这样我们可以在下次运行 playbook 时跳过下载和安装 WordPress 的任务。

1.  我们使用哪个模块来定义作为 playbook 运行的一部分的变量？

`set_fact`模块

1.  我们传递哪个参数给`shell`模块，以便在我们选择的目录中执行我们想要运行的命令？

参数是`chdir`

1.  真或假：将 MariaDB 绑定到`127.0.0.1`将允许我们从外部访问它。

假

# 第六章，针对多个发行版

1.  真或假：我们需要仔细检查 playbook 中的每个任务，以便它可以在两个操作系统上运行。

真

1.  哪个配置选项允许我们定义 Python 的路径，Ansible 将使用？

选项是`ansible_python_interpreter`

1.  解释为什么我们需要对配置并与 PHP-FPM 服务交互的任务进行更改。

配置文件的路径不同，而且在 Ubuntu 上 PHP-FPM 默认在不同的组下运行

1.  真或假：每个操作系统的软件包名称完全对应。

假

# 第七章，核心网络模块

1.  真或假：您必须在模板中使用`with_items`与`for`循环。

假

1.  哪个字符用于将您的变量分成多行？

您将使用`|`字符

1.  真或假：使用 VyOS 模块时，我们不需要在主机清单文件中传递设备的详细信息。

真

# 第八章，转向云端

1.  我们需要安装哪个 Python 模块来支持`digital_ocean`模块？

该模块称为`dopy`

1.  真或假：您应该始终加密诸如 DigitalOcean 个人访问令牌之类的敏感值。

真

1.  我们使用哪个过滤器来查找我们需要使用的 SSH 密钥的 ID 来启动我们的 Droplet？

过滤器将是`[?name=='Ansible']`

1.  陈述并解释为什么我们在`digital_ocean`任务中使用了`unique_name`选项。

确保我们不会在每个 playbook 运行时启动具有相同名称的多个 droplets。

1.  从另一个 Ansible 主机访问变量的正确语法是什么？

使用`hostvars`，例如使用`{{ hostvars['localhost'].droplet_ip }}`，这已在 Ansible 控制器上注册。

1.  真或假：`add_server`模块用于将我们的 Droplet 添加到主机组。

错误

# 第九章，构建云网络

1.  哪两个环境变量被 AWS 模块用来读取您的访问 ID 和秘密？

它们是`AWS_ACCESS_KEY`和`AWS_SECRET_KEY`

1.  真或假：每次运行 playbook 时，您都会获得一个新的 VPC。

错误

1.  陈述并解释为什么我们不费心注册创建子网的结果。

这样我们可以通过我们稍后在 playbook 运行中分配给它们的角色将子网 ID 列表分组在一起

1.  在定义安全组规则时，使用`cidr_ip`和`group_id`有什么区别？

`cidr_ip`创建一个规则，将提供的端口锁定到特定 IP 地址，而`group_id`将端口锁定到您提供的`group_id`中的所有主机

1.  真或假：在使用具有`group_id`定义的规则时，添加安全组的顺序并不重要。

错误

# 第十章，高可用云部署

1.  使用`gather_facts`选项注册的变量的名称是什么，其中包含我们执行 playbook 的日期和时间？

这是`ansible_date_time`事实

1.  真或假：Ansible 自动找出需要执行的任务，这意味着我们不必自己定义任何逻辑。

错误

1.  解释为什么我们必须使用`local_action`模块。

因为我们不想从我们使用 Ansible 的主机与 AWS API 进行交互； 相反，我们希望所有 AWS API 交互都发生在我们的 Ansible 控制器上

1.  我们在`ansible-playbook`命令之前添加哪个命令来记录我们的命令执行花费了多长时间？

`time`命令

1.  真或假：在使用自动扩展时，您必须手动启动 EC2 实例。

错误

# 第十一章，构建 VMware 部署

1.  您需要在 Ansible 控制器上安装哪个 Python 模块才能与 vSphere 进行交互？

该模块称为 PyVmomi

1.  真或假：`vmware_dns_config`只允许您在 ESXi 主机上设置 DNS 解析器。

错误

1.  列举我们已经介绍的两个可以用于启动虚拟机的模块名称；有三个，但一个已被弃用。

`vca_vapp`和`vmware_guest`模块；已弃用`vsphere_guest`模块

1.  我们已经查看的模块中，您将使用哪个模块来确保在进行与 VMware 通过 VMware 交互的任务之前，虚拟机完全可用？

`vmware_guest_tools_wait`模块

1.  真或假：可以安排使用 Ansible 更改电源状态。

真

# 第十二章，Ansible Windows 模块

1.  以下两个模块中哪一个可以在 Windows 和 Linux 主机上使用：setup 或 file？

`setup`模块

1.  真或假：您可以使用 SSH 访问您的 Windows 目标。

错误

1.  解释 WinRM 使用的接口类型。

WinRM 使用 SOAP 接口而不是交互式 shell

1.  您需要安装哪个 Python 模块才能与 macOS 和 Linux 上的 WinRM 进行交互？

`pywinrm`模块

1.  真或假：您可以在使用`win_chocolatey`模块之前单独安装 Chocolatey 的任务。

错误

# 第十三章，使用 Ansible 和 OpenSCAP 加固您的服务器

1.  将`>`添加到多行变量会产生什么影响？

当 Ansible 将其插入 playbook 运行时，该变量将呈现为单行

1.  真或假：OpenSCAP 获得了 NIST 的认证。

真

1.  为什么我们告诉 Ansible 如果`scan`命令标记为失败就继续？

因为如果得不到 100%的分数，任务将总是失败

1.  解释为什么我们对某些角色使用标签。

这样我们就可以在使用`--tags`标志时运行 playbook 的某些部分

1.  真或假：我们使用`copy`命令将 HTML 报告从远程主机复制到 Ansible 控制器。

假

# 第十四章，部署 WPScan 和 OWASP ZAP

1.  为什么我们使用 Docker 而不是直接在我们的 Vagrant box 上安装 WPScan 和 OWASP ZAP？

简化部署过程；部署两个容器比安装两个工具的支持软件堆栈更容易

1.  真或假：`pip`默认安装在我们的 Vagrant box 上。

假

1.  我们需要安装哪个 Python 模块才能使 Ansible Docker 模块正常工作？

`docker`模块

# 第十五章，介绍 Ansible Tower 和 Ansible AWX

1.  阐述 Ansible Tower 和 Ansible AWX 之间的区别并解释。

Ansible Tower 是由 Red Hat 提供的商业支持的企业级软件。Ansible AWX 是未来版本的 Ansible Tower 的开源上游；它经常更新并按原样提供。
