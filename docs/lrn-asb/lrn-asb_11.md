# 第十一章：构建 VMware 部署

现在我们知道如何在 AWS 中启动网络和服务，我们现在将讨论在 VMware 环境中部署类似设置，并讨论核心 VMware 模块。

在本章中，我们将：

+   快速介绍 VMware

+   审查 Ansible VMware 模块

+   通过一个示例 playbook 来启动几个虚拟机

# 技术要求

在本章中，我们将讨论 VMware 产品系列的各种组件，以及如何使用 Ansible 与它们进行交互。虽然本章中有一个示例 playbook，但它可能不容易转移到您的安装中。因此，建议您在更新之前不要使用本章中的任何示例。

# VMware 简介

VMware 有近 20 年的历史，从一个隐秘的初创公司到被戴尔拥有并被 EMC 收购，收入达 79.2 亿美元。VMware 产品组合目前有大约 30 种产品；最常见的是其 hypervisors，其中有两种不同的类型。

第一个 hypervisor，VMware ESXi，是一种直接在硬件上运行的类型 1，使用大多数现代 64 位英特尔和 AMD CPU 中找到的指令集。其原始的类型 2 hypervisor 不需要 CPU 中存在虚拟化指令，就像它们需要在类型 1 中一样。它以前被称为 GSX；这个 hypervisor 早于类型 1 hypervisor，这意味着它可以支持更旧的 CPU。

VMware 在大多数企业中非常普遍；它允许管理员快速在许多标准的基于 x86 的硬件配置和类型上部署虚拟机。

# VMware 模块

如前所述，VMware 范围内大约有 30 种产品；这些产品涵盖了从 hypervisors 到虚拟交换机、虚拟存储以及与基于 VMware 的主机和虚拟机进行交互的几个接口。在本节中，我们将介绍随 Ansible 一起提供的核心模块，以管理您的 VMware 资产的所有方面。

我尝试将它们分成逻辑组，并且对于每个组，都会简要解释模块所针对的产品。

# 要求

所有模块都有一个共同点：它们都需要安装一个名为`PyVmomi`的 Python 模块。要安装它，请运行以下`pip`命令：

```
$ sudo pip install PyVmomi
```

该模块包含了 VMware vSphere API Python 绑定，没有它，我们将在本章中要讨论的模块无法与您的 VMware 安装进行交互。

虽然本章中的模块已经在 vSphere 5.5 到 6.5 上进行了测试，但您可能会发现一些旧模块在较新版本的 vSphere 上存在一些问题。

# vCloud Air

vCloud Air 是 VMware 的**基础设施即服务**（**IaaS**）产品，我说*是*因为 vCloud Air 业务部门和负责该服务的团队于 2017 年中被法国托管和云公司 OVH 从 VMware 收购。有三个 Ansible 模块直接支持 vCloud Air，以及**VMware vCloud Hybrid Service**（**vCHS**）和**VMware vCloud Director**（**vCD**）。

# vca_fw 模块

该模块使您能够从 vCloud Air 网关中添加和删除防火墙规则。以下示例向您展示了如何添加一个允许 SSH 流量的规则：

```
- name: example fireware rule
  vca_fw:
   instance_id: "abcdef123456-1234-abcd-1234-abcdef123456"
   vdc_name: "my_vcd"
   service_type: "vca"
   state: "present"
   fw_rules:
     - description: "Allow SSH"
       source_ip: "10.20.30.40"
       source_port: "Any"
       dest_port: "22"
       dest_ip: "192.0.10.20"
       is_enable: "true"
       enable_logging: "false"
       protocol: "Tcp"
       policy: "allow"
```

注意我们传递了一个`service_type`；这可以是`vca`、`vcd`或`vchs`。

# vca_nat 模块

该模块允许您管理**网络地址转换**（**NAT**）规则。在下面的示例中，我们要求所有命中公共 IP 地址`123.123.123.123`上的端口`2222`的流量被转发到 IP 地址为`192.0.10.20`的虚拟机上的端口`22`：

```
- name: example nat rule
  vca_nat:
   instance_id: "abcdef123456-1234-abcd-1234-abcdef123456"
   vdc_name: "my_vcd"
   service_type: "vca"
   state: "present"
   nat_rules:
      - rule_type: "DNAT"
        original_ip: "123.123.123.123"
        original_port: "2222"
        translated_ip: "192.0.10.20"
        translated_port: "22"
```

这意味着要从我们的外部网络访问虚拟机`192.0.10.20`上的 SSH，我们需要运行类似以下命令：

```
$ ssh username@123.123.123.123 -p2222
```

假设我们已经设置了正确的防火墙规则，我们应该通过`192.0.10.20`虚拟机进行路由。

# vca_vapp 模块

该模块用于创建和管理 vApps。vApp 是一个或多个虚拟机的组合，用于提供一个应用程序：

```
- name: example vApp
  vca_vapp:
    vapp_name: "Example"
    vdc_name: "my_vcd"
    state: "present"
    operation: "poweron"
    template_name: "CentOS7 x86_64 1804"
```

上一个示例是使用`vca_vapp`模块的一个非常基本的示例，以确保存在名为`Example`的 vApp 并且处于开启状态。

# VMware vSphere

VMware vSphere 是由 VMware 组件组成的软件套件。这就是 VMware 可能会让人有点困惑的地方，因为 VMware vSphere 由 VMware vCentre 和 VMware ESXi 组成，它们各自也有自己的 Ansible 模块，并且在表面上，它们似乎完成了类似的任务。

# vmware_cluster 模块

该模块允许您管理您的 VMware vSphere 集群。VMware vSphere 集群是一组主机，当它们被集群在一起时，它们共享资源，允许您添加**高可用性**（**HA**），并且还可以启动**分布式资源调度器**（**DRS**），它管理集群中工作负载的放置：

```
- name: Create a cluster
  vmware_cluster:
    hostname: "{{ item.ip }}"
    datacenter_name: "my_datacenter"
    cluster_name: "cluster"
    enable_ha: "yes"
    enable_drs: "yes"
    enable_vsan: "yes"
    username: "{{ item.username }}"
    password: "{{ item.password }}"
  with_items: "{{ vsphere_hosts }}"
```

前面的代码将循环遍历主机、用户名和密码列表以创建一个集群。

# vmware_datacenter 模块

VMware vSphere 数据中心是指支持您的集群的物理资源、主机、存储和网络的集合的名称：

```
- name: Create a datacenter
  vmware_datacenter:
    hostname: "{{ item.ip }}"
    username: "{{ item.username }}"
    password: "{{ item.password }}"
    datacenter_name: "my_datacenter"
    state: present
  with_items: "{{ vsphere_hosts }}"
```

上一个示例将`vsphere_hosts`中列出的主机添加到`my_datacenter` VMware vSphere 数据中心。

# vmware_vm_facts 模块

该模块可用于收集运行在您的 VMware vSphere 集群中的虚拟机或模板的信息：

```
- name: Gather facts on all VMs in the cluster
  vmware_vm_facts:
    hostname: "{{ vsphere_host }}"
    username: "{{ vsphere_username }}"
    password: "{{ vsphere_password }}"
    vm_type: "vm"
  delegate_to: "localhost"
  register: vm_facts
```

上一个示例仅收集了在我们的集群中创建的虚拟机的信息，并将结果注册为`vm_facts`变量。如果我们想要找到有关模板的信息，我们可以将`vm_type`更新为 template，或者我们可以通过将`vm_type`更新为 all 来列出所有虚拟机和模板。

# vmware_vm_shell 模块

该模块可用于连接到使用 VMware 的虚拟机并运行 shell 命令。在任何时候，Ansible 都不需要使用诸如 SSH 之类的基于网络的服务连接到虚拟机，这对于在虚拟机上线之前配置 VM 非常有用：

```
- name: Shell example
  vmware_vm_shell:
    hostname: "{{ vsphere_host }}"
    username: "{{ vsphere_username }}"
    password: "{{ vsphere_password }}"
    datacenter: "my_datacenter"
    folder: "/my_vms"
    vm_id: "example_vm"
    vm_username: "root"
    vm_password: "supersecretpassword"
    vm_shell: "/bin/cat"
    vm_shell_args: " results_file "
    vm_shell_env:
      - "PATH=/bin"
      - "VAR=test"
    vm_shell_cwd: "/tmp"
  delegate_to: "localhost"
  register: shell_results
```

上一个示例连接到名为`example_vm`的 VM，该 VM 存储在`my_datacenter`数据中心根目录下的`my_vms`文件夹中。一旦使用我们提供的用户名和密码连接后，它将运行以下命令：

```
$ /bin/cat results_file
```

在 VM 的`/tmp`文件夹中，运行命令的输出被注册为`shell_results`，以便我们以后可以使用它。

# vmware_vm_vm_drs_rule 模块

使用此模块，您可以配置 VMware DRS 亲和性规则。这允许您控制集群中虚拟机的放置：

```
- name: Create DRS Affinity Rule for VM-VM
  vmware_vm_vm_drs_rule:
    hostname: "{{ vsphere_host }}"
    username: "{{ vsphere_username }}"
    password: "{{ vsphere_password }}"
    cluster_name: "cluster"
    vms: "{{ item }}"
    drs_rule_name: ""
    enabled: "True"
    mandatory: "True"
    affinity_rule: "True"
  with_items:
    - "example_vm"
    - "another_example_vm"
```

在上一个示例中，我们正在创建一个规则，使得 VMs `example_vm`和`another_example_vm`永远不会在同一台物理主机上运行。

# vmware_vm_vss_dvs_migrate 模块

该模块将指定的虚拟机从标准 vSwitch 迁移到分布式 vSwitch，后者可在整个集群中使用。

```
- name: migrate vm to dvs
  vmware_vm_vss_dvs_migrate"
    hostname: "{{ vsphere_host }}"
    username: "{{ vsphere_username }}"
```

```
    password: "{{ vsphere_password }}"
    vm_name: "example_vm"
    dvportgroup_name: "example_portgroup"
  delegate_to: localhost
```

正如你所看到的，我们正在将`example_vm`从标准 vSwitch 移动到名为`example_portgroup`的分布式 vSwitch。

# vsphere_copy 模块

该模块有一个单一的目的——将本地文件复制到远程数据存储：

```
- name: copy file to datastore
  vsphere_copy:
    hostname: "{{ vsphere_host }}"
    username: "{{ vsphere_username }}"
    password: "{{ vsphere_password }}"
    src: "/path/to/local/file"
    datacenter: "my_datacenter"
    datastore: "my_datastore"
    path: "path/to/remove/file"
  transport: local
```

正如你所看到的，我们正在将文件从`/path/to/local/file`复制到`my_datacenter`数据中心中托管的`my_datastore`数据存储中的`path/to/remove/file`。

# vsphere_guest 模块

该模块已被弃用，并将在 Ansible 2.9 中删除；建议您改用`vmware_guest`模块。

# VMware vCentre

VMware vCentre 是 VMware vSphere 套件的重要组件；它使诸如 vMotion、VMware 分布式资源调度器和 VMware 高可用性等功能进行集群化。

# vcenter_folder 模块

此模块使 vCenter 文件夹管理成为可能。例如，以下示例为您的虚拟机创建一个文件夹：

```
- name: Create a vm folder
  vcenter_folder:
    hostname: "{{ item.ip }}"
    username: "{{ item.username }}"
    password: "{{ item.password }}"
    datacenter_name: "my_datacenter"
    folder_name: "virtual_machines"
    folder_type: "vm"
    state: "present"
```

以下是为您的主机创建文件夹的示例：

```
- name: Create a host folder
  vcenter_folder:
    hostname: "{{ item.ip }}"
    username: "{{ item.username }}"
    password: "{{ item.password }}"
    datacenter_name: "my_datacenter"
    folder_name: "hosts"
    folder_type: "host"
    state: "present"
```

# vcenter_license 模块

此模块允许您添加和删除 VMware vCenter 许可证：

```
- name: Add a license
  vcenter_license:
    hostname: "{{ item.ip }}"
    username: "{{ item.username }}"
    password: "{{ item.password }}"
    license: "123abc-456def-abc456-def123"
    state: "present"
  delegate_to: localhost
```

# vmware_guest 模块

此模块允许您在 VMware 集群中启动和管理虚拟机；以下示例显示了如何使用模板启动 VM：

```
- name: Create a VM from a template
  vmware_guest:
    hostname: "{{ vsphere_host }}"
    username: "{{ vsphere_username }}"
    password: "{{ vsphere_password }}"
    validate_certs: "no"
    datacenter: "my-datacenter"
    folder: "/vms"
    name: "yet_another_example_vm"
    state: "poweredon"
    template: "centos7-x86_64-1804"
    disk:
      - size_gb: "40"
        type: "thin"
        datastore: "my_datastore"
    hardware:
      memory_mb: "4048"
      num_cpus: "4"
      max_connections: "3"
      hotadd_cpu: "True"
      hotremove_cpu: "True"
      hotadd_memory: "True"
    networks:
      - name: "VM Network"
        ip: "192.168.1.100"
        netmask: "255.255.255.0"
        gateway: "192.168.1.254"
        dns_servers:
          - "192.168.1.1"
          - "192.168.1.2"
    wait_for_ip_address: "yes"
  delegate_to: "localhost"
  register: deploy
```

正如您所看到的，我们对 VM 及其配置有相当多的控制权。硬件、网络和存储配置有单独的部分；我们将在本章末稍微详细地看一下这个模块。

# vmware_guest_facts 模块

此模块收集有关已创建的 VM 的信息：

```
- name: Gather facts on the yet_another_example_vm vm
  vmware_guest_facts:
    hostname: "{{ vsphere_host }}"
    username: "{{ vsphere_username }}"
    password: "{{ vsphere_password }}"
    validate_certs: "no"
    datacenter: "my-datacenter"
    folder: "/vms"
    name: "yet_another_example_vm"
  delegate_to: localhost
  register: facts
```

前面的示例收集了我们在上一节中定义的机器的大量信息，并将信息注册为变量，以便我们可以在 playbook 运行的其他地方使用它。

# vmware_guest_file_operation 模块

此模块是在 Ansible 2.5 中引入的；它允许您在 VM 上添加和获取文件，而无需 VM 连接到网络。它还允许您在 VM 内创建文件夹。以下示例在 VM 内创建一个目录：

```
- name: create a directory on a vm
  vmware_guest_file_operation:
    hostname: "{{ vsphere_host }}"
    username: "{{ vsphere_username }}"
    password: "{{ vsphere_password }}"
    validate_certs: "no"
    datacenter: "my-datacenter"
    vm_id: "yet_another_example_vm"
    vm_username: "root"
    vm_password: "supersecretpassword"
    directory:
      path: "/tmp/imported/files"
      operation: "create"
      recurse: "yes"
  delegate_to: localhost
```

以下示例将名为`config.zip`的文件从我们的 Ansible 主机复制到先前创建的目录中：

```
- name: copy file to vm
  vmware_guest_file_operation:
    hostname: "{{ vsphere_host }}"
    username: "{{ vsphere_username }}"
    password: "{{ vsphere_password }}"
    validate_certs: "no"
    datacenter: "my-datacenter"
    vm_id: "yet_another_example_vm"
    vm_username: "root"
    vm_password: "supersecretpassword"
    copy:
        src: "files/config.zip"
        dest: "/tmp/imported/files/config.zip"
        overwrite: "False"
  delegate_to: localhost
```

# vmware_guest_find 模块

我们知道 VM 运行的文件夹的名称。如果我们不知道，或者由于任何原因发生了更改，我们可以使用`vmware_guest_find`模块动态发现位置：

```
- name: Find vm folder location
  vmware_guest_find:
    hostname: "{{ vsphere_host }}"
    username: "{{ vsphere_username }}"
    password: "{{ vsphere_password }}"
    validate_certs: "no"
    name: "yet_another_example_vm"
  register: vm_folder
```

文件夹的名称将注册为`vm_folder`。

# vmware_guest_powerstate 模块

这个模块很容易理解；它用于管理 VM 的电源状态。以下示例重新启动了一个 VM：

```
- name: Powercycle a vm
  vmware_guest_powerstate:
    hostname: "{{ vsphere_host }}"
    username: "{{ vsphere_username }}"
    password: "{{ vsphere_password }}"
    validate_certs: "no"
    folder: "/vms"
    name: "yet_another_example_vm"
    state: "reboot-guest"
  delegate_to: localhost
```

您还可以安排对电源状态的更改。以下示例在 2019 年 4 月 1 日上午 9 点关闭 VM：

```
- name: April fools
  vmware_guest_powerstate:
    hostname: "{{ vsphere_host }}"
    username: "{{ vsphere_username }}"
    password: "{{ vsphere_password }}"
    validate_certs: "no"
    folder: "/vms"
    name: "yet_another_example_vm"
    state: "powered-off"
    scheduled_at: "01/04/2019 09:00"
  delegate_to: localhost
```

并不是我会做这样的事情！

# vmware_guest_snapshot 模块

此模块允许您管理 VM 快照；例如，以下创建了一个快照：

```
- name: Create a snapshot
  vmware_guest_snapshot:
    hostname: "{{ vsphere_host }}"
    username: "{{ vsphere_username }}"
    password: "{{ vsphere_password }}"
    validate_certs: "no"
    datacenter: "my-datacenter"
    folder: "/vms"
    name: "yet_another_example_vm"
    snapshot_name: "pre-patching"
    description: "snapshot made before patching"
    state: "present"
  delegate_to: localhost
```

从前面的示例中可以看出，这个快照是因为我们即将对 VM 进行打补丁。如果打补丁顺利进行，那么我们可以运行以下任务：

```
- name: Remove a snapshot
  vmware_guest_snapshot:
    hostname: "{{ vsphere_host }}"
    username: "{{ vsphere_username }}"
    password: "{{ vsphere_password }}"
    validate_certs: "no"
    datacenter: "my-datacenter"
    folder: "/vms"
    name: "yet_another_example_vm"
    snapshot_name: "pre-patching"
    state: "remove"
  delegate_to: localhost
```

如果一切不如预期那样进行，打补丁破坏了我们的 VM，那么不用担心，我们有一个可以恢复的快照：

```
- name: Revert to a snapshot
  vmware_guest_snapshot:
    hostname: "{{ vsphere_host }}"
    username: "{{ vsphere_username }}"
    password: "{{ vsphere_password }}"
    validate_certs: "no"
    datacenter: "my-datacenter"
    folder: "/vms"
    name: "yet_another_example_vm"
    snapshot_name: "pre-patching"
    state: "revert"
  delegate_to: localhost
```

祈祷您永远不必恢复快照（除非计划中）。

# vmware_guest_tools_wait 模块

本节的最后一个模块是另一个很容易理解的模块；它只是等待 VMware tools 可用，然后收集有关该机器的信息：

```
- name: Wait for VMware tools to become available by name
  vmware_guest_tools_wait:
    hostname: "{{ vsphere_host }}"
    username: "{{ vsphere_username }}"
    password: "{{ vsphere_password }}"
    validate_certs: "no"
    folder: "/vms"
    name: "yet_another_example_vm"
  delegate_to: localhost
  register: facts
```

VMware tools 是在 VM 内部运行的应用程序。一旦启动，它允许 VMware 与 VM 进行交互，从而使诸如`vmware_guest_file_operation`和`vmware_vm_shell`等模块能够正常运行。

# VMware ESXi

在大多数 VMware 安装的核心是一些 VMware ESXi 主机。VMware ESXi 是一种类型 1 的 hypervisor，可以使 VM 运行。Ansible 提供了几个模块，允许您配置和与您的 VMware ESXi 主机进行交互。

# vmware_dns_config 模块

此模块允许您管理 ESXi 主机的 DNS 方面；它允许您设置主机名、域和 DNS 解析器：

```
- name: Configure the hostname and dns servers
  local_action
    module: vmware_dns_config:
    hostname: "{{ exsi_host }}"
    username: "{{ exsi_username }}"
    password: "{{ exsi_password }}"
    validate_certs: "no"
    change_hostname_to: "esxi-host-01"
    domainname: "my-domain.com"
    dns_servers:
        - "8.8.8.8"
        - "8.8.4.4"
```

在前面的示例中，我们将主机的 FQDN 设置为`esxi-host-01.my-domain.com`，并配置主机使用 Google 公共 DNS 解析器。

# vmware_host_dns_facts 模块

一个简单的模块，用于收集您的 VMware ESXi 主机的 DNS 配置信息：

```
- name: gather facts on dns config
  vmware_host_dns_facts:
    hostname: "{{ vsphere_host }}"
    username: "{{ vsphere_username }}"
    password: "{{ vsphere_password }}"
    validate_certs: "no"
    cluster_name: "my_cluster"
```

# vmware_host 模块

您可以使用此模块将您的 ESXi 主机附加到 vCenter：

```
- name: add an esxi host to vcenter
  vmware_host:
    hostname: "{{ vsphere_host }}"
    username: "{{ vsphere_username }}"
    password: "{{ vsphere_password }}"
    datacenter_name: "my-datacenter"
    cluster_name: "my_cluster"
    esxi_hostname: "{{ exsi_host }}"
    esxi_username: "{{ exsi_username }}"
    esxi_password: "{{ exsi_password }}"
    state: present
```

您还可以使用该模块重新连接主机到您的 vCenter 集群：

```
- name: reattach an esxi host to vcenter
  vmware_host:
    hostname: "{{ vsphere_host }}"
    username: "{{ vsphere_username }}"
    password: "{{ vsphere_password }}"
    datacenter_name: "my-datacenter"
    cluster_name: "my_cluster"
    esxi_hostname: "{{ exsi_host }}"
    esxi_username: "{{ exsi_username }}"
    esxi_password: "{{ exsi_password }}"
    state: reconnect
```

您还可以从 vCenter 集群中删除主机：

```
- name: remove an esxi host to vcenter
  vmware_host:
    hostname: "{{ vsphere_host }}"
    username: "{{ vsphere_username }}"
    password: "{{ vsphere_password }}"
    datacenter_name: "my-datacenter"
    cluster_name: "my_cluster"
    esxi_hostname: "{{ exsi_host }}"
    esxi_username: "{{ exsi_username }}"
    esxi_password: "{{ exsi_password }}"
    state: absent
```

# vmware_host_facts 模块

正如您可能已经猜到的那样，此模块收集有关您的 vSphere 或 vCenter 集群中的 VMware ESXi 主机的信息：

```
- name: Find out facts on the esxi hosts
  vmware_host_facts:
    hostname: "{{ vsphere_host }}"
    username: "{{ vsphere_username }}"
    password: "{{ vsphere_password }}"
    validate_certs: "no"
  register: host_facts
  delegate_to: localhost
```

# vmware_host_acceptance 模块

使用此模块，您可以管理 VMware ESXi 主机的接受级别。VMware 支持四个接受级别，它们是：

+   VMwareCertified

+   VMwareAccepted

+   PartnerSupported

+   CommunitySupported

这些级别控制着可以安装在 ESXi 主机上的 VIB；VIB 是 ESXi 软件包。这通常决定了您将从 VMware 或 VMware 合作伙伴那里获得的支持水平。以下任务将为指定集群中的所有 ESXi 主机设置接受级别为 CommunitySupported：

```
- name: Set acceptance level for all esxi hosts in the cluster
  vmware_host_acceptance:
    hostname: "{{ vsphere_host }}"
    username: "{{ vsphere_username }}"
    password: "{{ vsphere_password }}"
    validate_certs: "no"
    cluster_name: "my_cluster"
    acceptance_level: "community"
    state: present
  register: cluster_acceptance_level
```

# vmware_host_config_manager 模块

使用此模块，您可以在各个 VMware ESXi 主机上设置配置选项，例如：

```
- name: Set some options on our esxi host
  vmware_host_config_manager:
    hostname: "{{ vsphere_host }}"
    username: "{{ vsphere_username }}"
    password: "{{ vsphere_password }}"
    validate_certs: "no"
    esxi_hostname: "{{ exsi_host }}"
    options:
        "Config.HostAgent.log.level": "verbose"
        "Annotations.WelcomeMessage": "Welcome to my awesome Ansible managed ESXi host"
        "Config.HostAgent.plugins.solo.enableMob": "false"
```

Ansible 将从您的 VMware 主机映射高级配置选项，因此有关可用选项的更多信息，请参阅您的文档。

# vmware_host_datastore 模块

此模块使您能够在 VMware ESXi 主机上挂载和卸载数据存储；在以下示例中，我们正在在清单中的所有 VMware ESXi 主机上挂载三个数据存储：

```
- name: Mount datastores on our cluster
  vmware_host_datastore:
    hostname: "{{ vsphere_host }}"
    username: "{{ vsphere_username }}"
    password: "{{ vsphere_password }}"
    validate_certs: "no"
    datacenter_name: "my-datacenter"
    datastore_name: "{{ item.name }}"
    datastore_type: "{{ item.type }}"
    nfs_server: "{{ item.server }}"
    nfs_path: "{{ item.path }}"
    nfs_ro: "no"
    esxi_hostname: "{{ inventory_hostname }}"
    state: present
  delegate_to: localhost
  with_items:
      - { "name": "ds_vol01", "server": "nas", "path": "/mnt/ds_vol01", 'type': "nfs"} 
      - { "name": "ds_vol02", "server": "nas", "path": "/mnt/ds_vol02", 'type': "nfs"} 
      - { "name": "ds_vol03", "server": "nas", "path": "/mnt/ds_vol03", 'type': "nfs"} 
```

# vmware_host_firewall_manager 模块

此模块允许您配置 VMware ESXi 主机上的防火墙规则：

```
- name: set some firewall rules on the esxi hosts
  vmware_host_firewall_manager:
    hostname: "{{ vsphere_host }}"
    username: "{{ vsphere_username }}"
    password: "{{ vsphere_password }}"
    validate_certs: "no"
    esxi_hostname: "{{ inventory_hostname }}"
    rules:
      - name: "vvold"
        enabled: "True"
      - name: "CIMHttpServer"
        enabled: "False"
```

上一个示例在主机清单中的每个 VMware ESXi 主机上启用了`vvold`并禁用了`CIMHttpServer`。

# vmware_host_firewall_facts 模块

正如您可能已经猜到的那样，此模块与其他事实模块一样，用于收集我们集群中所有主机的防火墙配置的信息：

```
- name: Get facts on all cluster hosts
  vmware_host_firewall_facts:
    hostname: "{{ vsphere_host }}"
    username: "{{ vsphere_username }}"
    password: "{{ vsphere_password }}"
    validate_certs: "no"
    cluster_name: "my_cluster"
```

它也可以仅收集单个主机的信息：

```
- name: Get facts on a single host
  vmware_host_firewall_facts:
    hostname: "{{ vsphere_host }}"
    username: "{{ vsphere_username }}"
    password: "{{ vsphere_password }}"
    validate_certs: "no"
    esxi_hostname: "{{ exsi_host }}"
```

# vmware_host_lockdown 模块

此模块带有一个警告，内容为：此模块具有破坏性，因为管理员权限是使用 API 管理的，请仔细阅读选项并继续。

您可以使用以下代码锁定主机：

```
- name: Lockdown an ESXi host
  vmware_host_lockdown:
    hostname: "{{ vsphere_host }}"
    username: "{{ vsphere_username }}"
    password: "{{ vsphere_password }}"
    validate_certs: "no"
    esxi_hostname: "{{ exsi_host }}"
    state: "present"
```

您可以使用以下方法将主机解除锁定：

```
- name: Remove the lockdown on an ESXi host
  vmware_host_lockdown:
    hostname: "{{ vsphere_host }}"
    username: "{{ vsphere_username }}"
    password: "{{ vsphere_password }}"
    validate_certs: "no"
    esxi_hostname: "{{ exsi_host }}"
    state: "absent"
```

如先前提到的，此模块可能会产生一些意想不到的副作用，因此您可能希望逐个主机执行此操作，而不是使用以下选项，该选项将使指定集群中的所有主机进入锁定状态：

```
- name: Lockdown all the ESXi hosts
  vmware_host_lockdown:
    hostname: "{{ vsphere_host }}"
    username: "{{ vsphere_username }}"
    password: "{{ vsphere_password }}"
    validate_certs: "no"
    cluster_name: "my_cluster"
    state: "present"
```

# vmware_host_ntp 模块

使用此模块，您可以管理每个 VMware ESXi 主机的 NTP 设置。以下示例配置所有主机使用相同的 NTP 服务器：

```
- name: Set NTP servers for all hosts
  vmware_host_ntp:
    hostname: "{{ vsphere_host }}"
    username: "{{ vsphere_username }}"
    password: "{{ vsphere_password }}"
    validate_certs: "no"
    cluster_name: "my_cluster"
    state: present
    ntp_servers:
        - 0.pool.ntp.org
        - 1.pool.ntp.org
        - 2.pool.ntp.org
```

# vmware_host_package_facts 模块

此模块可用于收集有关您集群中所有 VMware ESXi 主机的信息：

```
- name: Find out facts about the packages on all the ESXi hosts
  vmware_host_package_facts:
    hostname: "{{ vsphere_host }}"
    username: "{{ vsphere_username }}"
    password: "{{ vsphere_password }}"
    validate_certs: "no"
    cluster_name: "my_cluster"
  register: cluster_packages
```

与其他事实模块一样，它也可以仅收集单个主机的信息：

```
- name: Find out facts about the packages on a single ESXi host
  vmware_host_package_facts:
    hostname: "{{ vsphere_host }}"
    username: "{{ vsphere_username }}"
    password: "{{ vsphere_password }}"
    validate_certs: "no"
    esxi_hostname: "{{ exsi_host }}"
  register: host_packages
```

# vmware_host_service_manager 模块

此模块可让您管理集群成员或单个主机上的 ESXi 服务器：

```
- name: Start the ntp service on all esxi hosts
  vmware_host_service_manager:
    hostname: "{{ vsphere_host }}"
    username: "{{ vsphere_username }}"
    password: "{{ vsphere_password }}"
    validate_certs: "no"
    cluster_name: "my_cluster"
    service_name: "ntpd"
    service_policy: "automatic"
    state: "present"
```

在此示例中，我们正在启动集群中所有主机的 NTP 服务（`service_name`）；由于我们将`service_policy`定义为`automatic`，因此只有在配置了与防火墙规则相对应的服务时，服务才会启动。如果我们希望服务无论防火墙规则如何都启动，那么我们可以将`service_policy`设置为`on`，或者如果希望停止服务，则应将`service_policy`设置为`off`。

# vmware_host_service_facts 模块

使用此模块，您可以查找集群中每个 VMware ESXi 主机上配置的服务的信息：

```
- name: Find out facts about the services on all the ESXi hosts
  vmware_host_service_facts:
    hostname: "{{ vsphere_host }}"
    username: "{{ vsphere_username }}"
    password: "{{ vsphere_password }}"
    validate_certs: "no"
    cluster_name: "my_cluster"
  register: cluster_services
```

# vmware_datastore_facts 模块

这是一个旧式事实模块，可用于收集数据中心中配置的数据存储的信息：

```
- name: Find out facts about the datastores
  vmware_datastore_facts:
    hostname: "{{ vsphere_host }}"
    username: "{{ vsphere_username }}"
    password: "{{ vsphere_password }}"
    validate_certs: "no"
    datacenter: "my_datacenter"
  delegate_to: localhost
  register: datastore_facts
```

您可能会注意到这个和之前的事实模块之间的语法有一点不同。

# vmware_host_vmnic_facts 模块

从旧式事实模块返回到新模块，此模块可用于收集有关 VMware ESXi 主机上物理网络接口的信息：

```
- name: Find out facts about the vmnics on all the ESXi hosts
  vmware_host_vmnic_facts:
    hostname: "{{ vsphere_host }}"
    username: "{{ vsphere_username }}"
    password: "{{ vsphere_password }}"
    validate_certs: "no"
    datacenter: "my_datacenter"
  register: cluster_vmnics
```

对于单个 ESXi 主机，我们可以使用以下任务：

```
- name: Find out facts about the vmnics on a single ESXi host
  vmware_host_vmnic_facts:
    hostname: "{{ vsphere_host }}"
    username: "{{ vsphere_username }}"
    password: "{{ vsphere_password }}"
    validate_certs: "no"
    esxi_hostname: "{{ exsi_host }}"
  register: host_vmnics
```

# vmware_local_role_manager 模块

使用此模块，您可以在集群上配置角色；这些角色可用于分配特权。在以下示例中，我们正在为`vmware_qa`角色分配一些特权：

```
- name: Add a local role
  vmware_local_role_manager:
    hostname: "{{ vsphere_host }}"
    username: "{{ vsphere_username }}"
    password: "{{ vsphere_password }}"
    validate_certs: "no"
  local_role_name: "vmware_qa"
  local_privilege_ids: [ "Folder.Create", "Folder.Delete"]
  state: "present"
```

# vmware_local_user_manager 模块

使用此模块，您可以通过添加用户并设置其密码来管理本地用户：

```
- name: Add local user to ESXi
  vmware_local_user_manager:
    hostname: "{{ vsphere_host }}"
    username: "{{ vsphere_username }}"
    password: "{{ vsphere_password }}"
    validate_certs: "no"
    local_user_name: "myuser"
    local_user_password: "my-super-secret-password"
    local_user_description: "An example user added by Ansible"
  delegate_to: "localhost"
```

# vmware_cfg_backup 模块

使用此模块，您可以创建 VMware ESXi 主机配置的备份：

```
- name: Create an esxi host configuration backup
  vmware_cfg_backup:
    hostname: "{{ vsphere_host }}"
    username: "{{ vsphere_username }}"
    password: "{{ vsphere_password }}"
    validate_certs: "no"
    state: "saved"
    dest: "/tmp/"
    esxi_hostname: "{{ exsi_host }}"
  delegate_to: "localhost"
  register: cfg_backup
```

请注意，此模块将自动将主机置于维护状态，然后保存配置。在前面的示例中，您可以使用`fetch`模块使用`/tmp`中注册的信息来获取备份的副本。

您还可以使用此模块恢复配置：

```
- name: Restore an esxi host configuration backup
  vmware_cfg_backup:
    hostname: "{{ vsphere_host }}"
    username: "{{ vsphere_username }}"
    password: "{{ vsphere_password }}"
```

```
    validate_certs: "no"
    state: "loaded"
    dest: "/tmp/my-host-backup.tar.gz"
    esxi_hostname: "{{ exsi_host }}"
  delegate_to: "localhost"
```

最后，您还可以通过运行以下代码将主机配置重置为默认设置：

```
- name: Reset a host configuration to the default values
  vmware_cfg_backup:
    hostname: "{{ vsphere_host }}"
    username: "{{ vsphere_username }}"
    password: "{{ vsphere_password }}"
    validate_certs: "no"
    state: "absent"
    esxi_hostname: "{{ exsi_host }}"
  delegate_to: "localhost"
```

# vmware_vmkernel 模块

此模块允许您在主机上添加 VMkernel 接口，也称为虚拟 NIC：

```
- name: Add management port with a static ip
   vmware_vmkernel:
    hostname: "{{ vsphere_host }}"
    username: "{{ vsphere_username }}"
    password: "{{ vsphere_password }}"
    validate_certs: "no"
    esxi_hostname: "{{ exsi_host }}"
  vswitch_name: "my_vSwitch"
  portgroup_name: "my_portgroup"
  vlan_id: "the_vlan_id"
  network:
    type: "static"
    ip_address: "192.168.127.10"
    subnet_mask: "255.255.255.0"
  state: "present"
  enable_mgmt: "True"
```

在前面的示例中，我们添加了一个管理接口；还有以下选项：

+   `enable_ft`：启用容错流量的接口

+   `enable_mgmt`：启用管理流量的接口

+   `enable_vmotion`：启用 VMotion 流量的接口

+   `enable_vsan`：启用 VSAN 流量的接口

# vmware_vmkernel_facts 模块

另一个事实模块，这是一个新式模块；您可能已经猜到任务的样子：

```
- name: Find out facts about the vmkernel on all the ESXi hosts
  vmware_vmkernel_facts:
    hostname: "{{ vsphere_host }}"
    username: "{{ vsphere_username }}"
    password: "{{ vsphere_password }}"
    validate_certs: "no"
    cluster_name: "my_cluster"
  register: cluster_vmks

- name: Find out facts about the vmkernel on a single ESXi host
  vmware_vmkernel_facts:
    hostname: "{{ vsphere_host }}"
    username: "{{ vsphere_username }}"
    password: "{{ vsphere_password }}"
    validate_certs: "no"
    esxi_hostname: "{{ exsi_host }}"
  register: host_vmks
```

# vmware_target_canonical_facts 模块

使用此模块，您可以找出 SCSI 目标的规范名称；您只需要知道目标设备的 ID：

```
- name: Get Canonical name of SCSI device
  vmware_target_canonical_facts"
    hostname: "{{ vsphere_host }}"
    username: "{{ vsphere_username }}"
    password: "{{ vsphere_password }}"
    validate_certs: "no"
    target_id: "6"
   register: canonical_name
```

# vmware_vmotion 模块

您可以使用此模块执行虚拟机从一个 VMware ESXi 主机迁移到另一个主机的 vMotion：

```
- name: Perform vMotion of VM
  vmware_vmotion
    hostname: "{{ vsphere_host }}"
    username: "{{ vsphere_username }}"
    password: "{{ vsphere_password }}"
    validate_certs: "no"
    vm_name: "example_vm"
    destination_host: "esxi-host-02"
  delegate_to: "localhost"
  register: vmotion_results
```

# vmware_vsan_cluster 模块

您可以使用此模块注册 VSAN 集群；此模块的工作方式与本章中的其他模块略有不同，您首先需要在单个主机上生成集群 UUID，然后再使用生成的 UUID 在其余主机上部署 VSAN。

以下任务假定您有一个名为`esxi_hosts`的主机组，其中包含多个主机。第一个任务将 VSAN 分配给组中的第一个主机，然后注册结果：

```
- name: Configure VSAN on first host in the group
  vmware_vsan_cluster:
    hostname: "{{ groups['esxi_hosts'][0] }}"
    username: "{{ vsphere_username }}"
    password: "{{ vsphere_password }}"
  register: vsan_cluster
```

作为`vsan_cluster`注册的结果包含我们将需要在组中其余主机上使用的 VSAN 集群 UUID。以下代码配置了其余主机上的集群，跳过原始主机：

```
- name: Configure VSAN on the remaining hosts in the group
  vmware_vsan_cluster:
    hostname: "{{ item }}"
    username: "{{ vsphere_username }}"
    password: "{{ vsphere_password }}"
    cluster_uuid: "{{ vsan_cluster.cluster_uuid }}"
  with_items: "{{ groups['esxi_hosts'][1:] }}"
```

# vmware_vswitch 模块

使用此模块，您可以向 ESXi 主机添加或删除**VMware 标准交换机**（**vSwitch**）：

```
- name: Add a vSwitch
  vmware_vswitch:
    hostname: "{{ vsphere_host }}"
    username: "{{ vsphere_username }}"
    password: "{{ vsphere_password }}"
    validate_certs: "no"
    switch: "vswitch_name"
    nics:
      - "vmnic1"
      - "vmnic2"
    mtu: "9000"
  delegate_to: "localhost"
```

在此示例中，我们添加了一个连接到多个 vmnic 的 vSwitch。

# vmware_drs_rule_facts 模块

您可以使用此模块收集整个集群或单个数据中心中配置的 DRS 的事实：

```
- name: Find out facts about drs on all the hosts in the cluster
  vmware_drs_rule_facts:
    hostname: "{{ vsphere_host }}"
    username: "{{ vsphere_username }}"
    password: "{{ vsphere_password }}"
    validate_certs: "no"
    cluster_name: "my_cluster"
  delegate_to: "localhost"
  register: cluster_drs

- name: Find out facts about drs in a single data center
  vmware_drs_rule_facts:
    hostname: "{{ vsphere_host }}"
    username: "{{ vsphere_username }}"
    password: "{{ vsphere_password }}"
    validate_certs: "no"
    datacenter: "my_datacenter"
  delegate_to: "localhost"
  register: datacenter_drs
```

# vmware_dvswitch 模块

此模块允许您创建和删除分布式 vSwitches：

```
- name: Create dvswitch
  vmware_dvswitch:
    hostname: "{{ vsphere_host }}"
    username: "{{ vsphere_username }}"
    password: "{{ vsphere_password }}"
    validate_certs: "no"
    datacenter: "my_datacenter"
    switch_name: "my_dvSwitch"
    switch_version: "6.0.0"
    mtu: "9000"
    uplink_quantity: "2"
    discovery_proto: "lldp"
    discovery_operation: "both"
    state: present
  delegate_to: "localhost"
```

# vmware_dvs_host 模块

使用此模块，您可以向分布式虚拟交换机添加或删除主机：

```
- name: Add host to dvs
  vmware_dvs_host:
    hostname: "{{ vsphere_host }}"
    username: "{{ vsphere_username }}"
    password: "{{ vsphere_password }}"
    validate_certs: "no"
    esxi_hostname: "{{ exsi_host }}"
    switch_name: "my_dvSwitch"
    vmnics:
      - "vmnic1"
      - "vmnic2"
    state: "present"
  delegate_to: "localhost"
```

# vmware_dvs_portgroup 模块

使用此模块，您可以管理您的 DVS 端口组：

```
- name: Create a portgroup with vlan 
  vmware_dvs_portgroup:
    hostname: "{{ vsphere_host }}"
    username: "{{ vsphere_username }}"
    password: "{{ vsphere_password }}"
    validate_certs: "no"
    portgroup_name: "my_portgroup_vlan123"
    switch_name: "my_dvSwitch"
    vlan_id: "123"
    num_ports: "120"
    portgroup_type: "earlyBinding"
    state: "present"
  delegate_to: "localhost"
```

# vmware_maintenancemode 模块

使用此模块，您可以将主机置于维护模式。以下示例向您展示了如何在 VSAN 上保持对象可用性的同时将主机置于维护模式：

```
- name: Put host into maintenance mode
  vmware_maintenancemode:
    hostname: "{{ vsphere_host }}"
    username: "{{ vsphere_username }}"
    password: "{{ vsphere_password }}"
    validate_certs: "no"
    esxi_hostname: "{{ exsi_host }}"
    vsan: "ensureObjectAccessibility"
    evacuate: "yes"
    timeout: "3600"
    state: "present"
  delegate_to: "localhost"
```

# vmware_portgroup 模块

此模块允许您在给定集群中的主机上创建 VMware 端口组：

```
- name: Create a portgroup with vlan
  vmware_portgroup:
    hostname: "{{ vsphere_host }}"
    username: "{{ vsphere_username }}"
    password: "{{ vsphere_password }}"
    validate_certs: "no"
    cluster_name: "my_cluster"
    switch_name: "my_switch"
    portgroup_name: "my_portgroup_vlan123"
    vlan_id: "123"
  delegate_to: "localhost"
```

# vmware_resource_pool 模块

使用这个，我们要看的最后一个模块，您可以创建一个资源池。以下是如何执行此操作的示例：

```
- name: Add resource pool
  vmware_resource_pool:
    hostname: "{{ vsphere_host }}"
    username: "{{ vsphere_username }}"
    password: "{{ vsphere_password }}"
    validate_certs: "no"
    datacenter: "my_datacenter"
    cluster: "my_new_cluster"
    resource_pool: "my_resource_pool"
    mem_shares: "normal"
    mem_limit: "-1"
    mem_reservation: "0"
    mem_expandable_reservations: "True"
    cpu_shares: "normal"
    cpu_limit: "-1"
    cpu_reservation: "0"
    cpu_expandable_reservations: "True"
    state: present
  delegate_to: "localhost"
```

# 一个示例 playbook

在完成本章之前，我将分享一个我为在 VMware 集群中部署少量虚拟机编写的示例 playbook。该项目的想法是将七台虚拟机启动到客户的网络中，如下所示：

+   一个 Linux 跳板主机

+   一个 NTP 服务器

+   一个负载均衡器

+   两个 Web 服务器

+   两个数据库服务器

所有 VM 都必须从现有模板构建；不幸的是，这个模板是使用`/etc/sysconfig/network`文件中硬编码的网关 IP 地址`192.168.1.254`构建的。这意味着为了让这些机器正确出现在网络上，我必须在每台虚拟机启动后进行更改。

我首先在我的`group_vars`文件夹中设置了一个名为`vmware.yml`的文件；其中包含了连接到我的 VMware 安装所需的信息，以及 VM 的默认凭据：

```
vcenter:
  host: "cluster.cloud.local"
  username: "svc_ansible@cloud.local"
  password: "mymegasecretpassword"

wait_for_ip_address: "yes"
machine_state: "poweredon"

deploy:
  datacenter: "Cloud DC4"
  folder: "/vm/Ansible"
  resource_pool: "/Resources/"

vm_shell:
  username: "root"
  password: "hushdonttell"
  cwd: "/tmp"
  cmd: "/bin/sed"
  args: "-i 's/GATEWAY=192.168.1.254/GATEWAY={{ item.gateway }}/g' /etc/sysconfig/network"
```

我将使用两个角色中定义的变量。接下来是`group_vars/vms.yml`文件；其中包含了在我的 VMware 环境中启动虚拟机所需的所有信息：

```
vm:
  - name: "NTPSERVER01"
    machine_name: "ntpserver01"
    machine_template: "RHEL6_TEMPLATE"
    guest_id: "rhel6_64Guest"
    host: "compute-host-01.cloud.local"
    cpu: "1"
    ram: "1024"
    networks: 
      - name: "CLOUD-CUST|Customer|MANGMENT"
        ip: "192.168.99.10"
        netmask: "255.255.255.0"
        device_type: "vmxnet3"
    gateway: "192.168.99.254"
    disk:
      - size_gb: "30"
        type: "thin"
        datastore: "cust_sas_esx_nfs_01"
  - name: "JUMPHOST01"
    machine_name: "jumphost01"
    machine_template: "RHEL6_TEMPLATE"
    guest_id: "rhel6_64Guest"
    host: "compute-host-02.cloud.local"
    cpu: "1"
    ram: "1024"
    networks: 
      - name: "CLOUD-CUST|Customer|MANGMENT"
        ip: "192.168.99.20"
        netmask: "255.255.255.0"
        device_type: "vmxnet3"
    gateway: "192.168.99.254"
    disk:
      - size_gb: "30"
        type: "thin"
        datastore: "cust_sas_esx_nfs_01"
  - name: "LOADBALANCER01"
    machine_name: "loadbalancer01"
    machine_template: "LB_TEMPLATE"
    guest_id: "rhel6_64Guest"
    host: "compute-host-03.cloud.local"
    cpu: "4"
    ram: "4048"
    networks: 
      - name: "CLOUD-CUST|Customer|DMZ"
        ip: "192.168.98.100"
        netmask: "255.255.255.0"
        device_type: "vmxnet3"
    gateway: "192.168.99.254"
    disk:
      - size_gb: "30"
        type: "thin"
        datastore: "cust_sas_esx_nfs_02"    
  - name: "WEBSERVER01"
    machine_name: "webserver01"
    machine_template: "RHEL6_TEMPLATE"
    guest_id: "rhel6_64Guest"
    host: "compute-host-01.cloud.local"
    cpu: "1"
    ram: "1024"
    networks: 
      - name: "CLOUD-CUST|Customer|APP"
        ip: "192.168.100.10"
        netmask: "255.255.255.0"
        device_type: "vmxnet3"
    gateway: "192.168.100.254"
    disk:
      - size_gb: "30"
        type: "thin"
        datastore: "cust_sas_esx_nfs_01"
  - name: "WEBSERVER02"
    machine_name: "webserver02"
    machine_template: "RHEL6_TEMPLATE"
    guest_id: "rhel6_64Guest"
    host: "compute-host-02.cloud.local"
    cpu: "1"
    ram: "1024"
    networks: 
      - name: "CLOUD-CUST|Customer|APP"
        ip: "192.168.100.20"
        netmask: "255.255.255.0"
        device_type: "vmxnet3"
    gateway: "192.168.100.254"
    disk:
      - size_gb: "30"
        type: "thin"
        datastore: "cust_sas_esx_nfs_02"      
  - name: "DBSERVER01"
    machine_name: "dbserver01"
    machine_template: "RHEL6_TEMPLATE"
    guest_id: "rhel6_64Guest"
    host: "compute-host-10.cloud.local"
    cpu: "8"
    ram: "32000"
    networks: 
      - name: "CLOUD-CUST|Customer|DB"
        ip: "192.168.101.10"
        netmask: "255.255.255.0"
        device_type: "vmxnet3"
    gateway: "192.168.101.254"
    disk:
      - size_gb: "30"
        type: "thin"
        datastore: "cust_sas_esx_nfs_01"
      - size_gb: "250"
        type: "thick"
        datastore: "cust_ssd_esx_nfs_01" 
      - size_gb: "250"
        type: "thick"
        datastore: "cust_ssd_esx_nfs_01" 
      - size_gb: "250"
        type: "thick"
        datastore: "cust_ssd_esx_nfs_01" 
  - name: "DBSERVER02"
    machine_name: "dbserver02"
    machine_template: "RHEL6_TEMPLATE"
    guest_id: "rhel6_64Guest"
    host: "compute-host-11.cloud.local"
    cpu: "8"
    ram: "32000"
    networks: 
      - name: "CLOUD-CUST|Customer|DB"
        ip: "192.168.101.11"
        netmask: "255.255.255.0"
        device_type: "vmxnet3"
    gateway: "192.168.101.254"
    disk:
      - size_gb: "30"
        type: "thin"
        datastore: "cust_sas_esx_nfs_02"
      - size_gb: "250"
        type: "thick"
        datastore: "cust_ssd_esx_nfs_02" 
      - size_gb: "250"
        type: "thick"
        datastore: "cust_ssd_esx_nfs_02" 
      - size_gb: "250"
        type: "thick"
        datastore: "cust_ssd_esx_nfs_02"
```

正如你所看到的，我正在为所有七台 VM 定义规格、网络和存储；在可能的情况下，我正在进行存储的薄配置，并确保在一个角色中有多个虚拟机时，我正在使用不同的存储池。

现在我已经拥有了我虚拟机所需的所有细节，我可以创建角色了。首先是`roles/vmware/tasks/main.yml`：

```
- name: Launch the VMs
  vmware_guest:
    hostname: "{{vcenter.host}}"
    username: "{{ vcenter.username }}"
    password: "{{ vcenter.password }}"
    validate_certs: no
    datacenter: "{{ deploy.datacenter }}"
    folder: "{{ deploy.folder }}"
    name: "{{ item.machine_name | upper }}"
    state: "{{ machine_state }}"
    guest_id: "{{ item.guest_id }}"
    esxi_hostname: "{{ item.host }}"
    hardware:
      memory_mb: "{{ item.ram }}"
      num_cpus: "{{ item.cpu }}"
    networks: "{{ item.networks }}"
    disk: "{{ item.disk }}"
    template: "{{ item.machine_template }}"
    wait_for_ip_address: "{{ wait_for_ip_address }}"
    customization:
      hostname: "{{ item.machine_name | lower }}"
  with_items: "{{ vm }}"
```

正如你所看到的，这个任务循环遍历`vm`变量中的项目；一旦虚拟机启动，它将等待我分配的 IP 地址在 VMware 中可用。这确保了在启动下一个虚拟机或继续下一个角色之前，虚拟机已经正确启动。

下一个角色解决了在虚拟机模板中硬编码为`192.168.1.254`的网关的问题；它可以在`roles/fix/tasks/main.yml`中找到。该角色中有两个任务；第一个任务将网关更新为虚拟机所在网络的正确网关：

```
- name: Sort out the wrong IP address in the /etc/sysconfig/network file on the vms
  vmware_vm_shell:
    hostname: "{{vcenter.host}}"
    username: "{{ vcenter.username }}"
    password: "{{ vcenter.password }}"
    validate_certs: no
    vm_id: "{{ item.machine_name | upper }}"
    vm_username: "{{ vm_shell.username }}"
    vm_password: "{{ vm_shell.password }}"
    vm_shell: "{{ vm_shell.cmd }}"
    vm_shell_args: " {{ vm_shell.args }} "
    vm_shell_cwd: "{{ vm_shell.cwd }}"
  with_items: "{{ vm }}"
```

正如你所看到的，这个任务循环遍历定义为`vm`的虚拟机列表，并执行我们在`group_vars/vmware.yml`文件中定义的`sed`命令。一旦这个任务运行完毕，我们需要再运行一个任务。这个任务重新启动所有虚拟机上的网络，以便网关的更改被接受：

```
- name: Restart networking on all VMs
  vmware_vm_shell:
    hostname: "{{vcenter.host}}"
    username: "{{ vcenter.username }}"
    password: "{{ vcenter.password }}"
    validate_certs: no
    vm_id: "{{ item.machine_name | upper }}"
    vm_username: "{{ vm_shell.username }}"
    vm_password: "{{ vm_shell.password }}"
    vm_shell: "/sbin/service"
    vm_shell_args: "network restart"
  with_items: "{{ vm }}"
```

当我运行 playbook 时，大约需要 30 分钟才能运行完，但最终我启动了七台虚拟机，并且可以使用，所以我随后能够运行一系列的 playbooks，对环境进行引导，以便我可以将它们交给客户，让他们部署他们的应用程序。

# 总结

正如你从非常长的模块列表中所看到的，你可以使用 Ansible 来完成大部分作为 VMware 管理员的常见任务。再加上我们在第七章中所看到的*核心网络模块*，用于管理网络设备，以及支持 NetApp 存储设备的模块，你可以构建一些跨物理设备、VMware 元素甚至在虚拟化基础设施中运行的虚拟机的复杂 playbooks。

在下一章中，我们将看到如何使用 Vagrant 在本地构建我们的 Windows 服务器，然后将我们的 playbooks 移到公共云。

# 问题

1.  你需要在你的 Ansible 控制器上安装哪个 Python 模块才能与 vSphere 进行交互？

1.  真或假：`vmware_dns_config`只允许你在你的 ESXi 主机上设置 DNS 解析器。

1.  列举我们已经涵盖的两个可以用来启动虚拟机的模块的名称；有三个，但其中一个已被弃用。

1.  我们已经查看的模块中，你会使用哪一个来确保虚拟机在进行与 VMware 交互的任务之前完全可用？

1.  真或假：使用 Ansible 可以安排更改电源状态。

# 进一步阅读

关于 VMware vSphere 的一个很好的概述，我推荐观看以下视频：[`www.youtube.com/watch?v=3OvrKZYnzjM`](https://www.youtube.com/watch?v=3OvrKZYnzjM)。
