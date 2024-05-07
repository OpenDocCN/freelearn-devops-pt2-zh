# Ansible 自动化基础设施

我们已经介绍了如何编写 playbooks 以及如何使用一些方便的模块填充它们。现在让我们把所有东西混合在一起，构建真实的日常基础设施管理情况。本章将提供一系列示例，我们将使用 Ansible playbooks，并借助一些 Linux 工具来自动化日常任务和其他非工作时间发生的任务。这些 playbooks 将有多个任务按顺序工作，以使您能够有效地计划工作。

本章将涵盖以下主题：

+   Linux 系统和应用程序自动化

+   Windows 系统和应用程序自动化

+   容器配置管理

+   网络配置自动化

+   虚拟和云基础设施自动化

# Linux 基础设施自动化

我们将首先看一些涉及 Linux 管理的用例。在这一部分，我们将确定通常需要手动完成的任务，并尝试尽可能自动化。在出现错误或配置错误的情况下，仍可能需要管理员。

我们将把以下用例分成子类别，以更好地确定它们在一般情况下的作用。在每种情况下，我们将查看几个 Ansible 任务。这些任务要么遵循一个 playbook 序列，要么在满足某些条件时执行，要么在循环内执行。

# 系统管理自动化

在这个小节中，我们将展示一些涉及系统管理任务的用例，这些任务可以使用 Ansible playbooks 进行自动化。我们将首先描述任务和执行任务的环境，然后编写一个格式良好且命名规范的 playbook，以说明如何使用 Ansible。

# 用例 1 - 系统更新自动化

这个用例旨在更新和清理基于 Linux 的主机，主要分为 Debian 和 Red Hat 两大家族。任务应该能够更新软件列表索引，安装任何可用的更新，删除不必要的软件包，清理软件包管理器缓存，并在需要时重新启动主机。这个 playbook 可以用于可访问 Ansible 管理服务器的物理或虚拟 Linux 主机。

这个 playbook 的代码如下：

```
---
- name: Update and clean up Linux OS 
  hosts: Linux
  become: yes
  gather_facts: yes
  tasks:
    - name: Update Debian Linux packages with Index 
      updated
      apt: 
        upgrade: dist
        update_cache: yes
      when: ansible_os_family == "Debian"

    - name: Update Red Hat Linux packages with Index 
      updated
      yum: 
        name: "*"
        state: latest
        update_cache: yes
      when: ansible_os_family == "RedHat"

    - name: Clean up Debian Linux from cache and unused 
      packages
      apt: 
        autoremove: yes 
        autoclean: yes
      when: ansible_os_family == "Debian"

    - name: Clean up Red Hat Linux from cache and unused 
      packages
      shell: yum clean all; yum autoremove
      when: ansible_os_family == "RedHat"
      ignore_errors: yes

   - name: Check if Debian system requires a reboot
     shell: "[ -f /var/run/reboot-required ]"
     failed_when: False
     register: reboot_required
     changed_when: reboot_required.rc == 0
     notify: reboot
     when: ansible_os_family == "Debian"
     ignore_errors: yes

   - name: Check if Red Hat system requires a reboot
     shell: "[ $(rpm -q kernel|tail -n 1) != 
     kernel-$(uname -r) ]"
     failed_when: False
     register: reboot_required
     changed_when: reboot_required.rc == 0
     notify: reboot
     when: ansible_os_family == "RedHat" 
     ignore_errors: yes

  handlers:
   - name: reboot
     command: shutdown -r 1 "A system reboot triggered 
     after and Ansible automated system update"
     async: 0
     poll: 0
     ignore_errors: true
```

然后可以安排执行这个 playbook，使用`crontab`作业在周末或深夜系统空闲时执行。或者，可以安排在系统全天候活动的维护期间运行。为了适应冗余主机，用户可以在定义任务之前，在 playbook 头部添加一个批处理大小和最大失败百分比参数。以下代码行可用于启用一定程度的保护：

```
---
- name: Update and clean up Linux OS 
  hosts: Linux
  max_fail_percentage: 20
  serial: 5
  become: yes
  become_user: setup
  gather_facts: yes
  tasks: ...
```

这允许您一次处理五个主机。如果总主机数量的 20%失败，playbook 将停止。

# 用例 2 - 创建一个新用户及其所有设置

这个用例允许您自动添加新用户到系统中。基本上，我们将在所有 Linux 主机上创建一个新用户，并设置好密码。我们还将创建一个 SSH 密钥，以便可以远程访问，并添加一些`sudo`配置以便更容易管理。这在以下代码中实现：

```
---
- name: Create a dedicated remote management user 
  hosts: Linux
  become: yes
  gather_facts: yes
  tasks:
    - name: Create a now basic user
      user: 
         name: 'ansuser' 
         password: 
   $6$C2rcmXJPhMAxLLEM$N.XOWkuukX7Rms7QlvclhWIOz6.MoQd/
   jekgWRgDaDH5oU2OexNtRYPTWwQ2lcFRYYevM83wIqrK76sgnVqOX. 
         # A hash for generic password.
         append: yes
         groups: sudo
         shell: /bin/bash
         state: present

    - name: Create the user folder to host the SSH key
      file: 
         path: /home/ansuser/.ssh
         state: directory
         mode: 0700
         owner: ansuser

    - name: Copy server public SSH key to the newly 
      created folder
      copy: 
         src: /home/admin/.ssh/ansible_rsa
         dest: /home/ansuser/.ssh/id_rsa
         mode: 0600
         owner: ansuser

    - name: Configure the sudo group to work without a 
       password
      lineinfile: 
         dest: /etc/sudoers
         regexp: '^%sudo\s'
         line: "%sudo ALL=(ALL) NOPASSWD{{':'}} ALL" 
         validate: 'visudo -cf %s'
         state: present

    - name: Install favourite text editor for Debian 
      family
      apt: 
         name: nano
         state: latest
         update_cache: yes
      when: ansible_os_family == "Debian"

    - name: Install favourite text editor for Red Hat 
      family
      yum: 
         name: nano
         state: latest
      when: ansible_os_family == "RedHat"

    - name: remove old editor configuration file
      file: 
         path: /home/ansuser/.selected_editor
         state: absent
      ignore_errors: yes

    - name: Create a new configuration file with the 
      favorite text editor
      lineinfile: 
         dest: /home/ansuser/.selected_editor
         line: "SELECTED_EDITOR='/usr/bin/nano'" 
         state: present
         create: yes

    - name: Make the user a system user to hide it from 
      login interface
      blockinfile: 
         path: /var/lib/AccountsService/users/ansuser
         state: present
         create: yes
         block: |
             [User]
             SystemAccount=true
```

在正确的清单配置上执行时，这个 playbook 应该能够取代通常需要访问多个主机来配置单个用户的数小时工作。通过一些调整，可以向任何 playbook 添加额外的功能。在这种情况下，我们可以将任何用户配置添加到管道中。

# 用例 3 - 服务（systemd）管理

在这个用例中，我们将使用 Ansible 手册自动设置和配置多个主机上的一些系统服务。以下代码显示了如何确保安装服务，然后如何进行配置检查以确保其配置良好。最后，我们启动服务并启用它在系统启动时启动：

```
---
- name: Setup and configured recommended Linux services
  hosts: Linux
  become: yes
  gather_facts: yes
  tasks:
    - name: Install a list of services on Linux hosts
      package: 
         name: '{{ item }}'
         state: latest
      with_items:
         - ntp
         - tzdate
         - autofs

    - name: Fix time zone on Red Hat 6
      lineinfile: 
         path: /etc/sysconfig/clock
         line: "ZONE='Europe/London'"
         state: present
         create: yes
      when: ansible_os_family == 'RedHat' and 
      ansible_distribution_version.split('.')[0] == '6'

    - name: Setup time zone on all local hosts
      timezone: 
         name: "Europe/London"

    - name: Fix time zone on Red Hat 6
      blockinfile:
         path: /etc/ntp.conf
         block: |
            server time.nist.gov iburst
            server 0.uk.pool.ntp.org iburst
            server 1.uk.pool.ntp.org iburst
         insertafter: "# Specify one or more NTP 
         servers."
         state: present
      when: ansible_os_family == 'RedHat'
- name: Restart NTP service to apply change and enable    
   it on Debian
  systemd:
  name: ntp
  enabled: True
  state: restarted
  when: ansible_os_family == 'Debian'

 - name: Restart NTP service to apply change and enable 
  it on Red Hat
  systemd:
  name: ntpd
  enabled: True
  state: restarted
  when: ansible_os_family == 'RedHat'

 - name: Add NFS and SMB support to automount
  blockinfile: 
  path: /etc/auto.master
  block: |
  /nfs /etc/auto.nfs
  /cifs /etc/auto.cifs 
  state: present

 - name: create the NFS and SMB AutoFS configuration    
   files
  file: 
  name: '{{ item }}'
  state: touch
  with_items:
  - '/etc/auto.nfs'
  - '/etc/auto.cifs'

 - name: Restart AutoFS service to apply a change and 
   enable it
  systemd:
  name: autofs
  enabled: True
  state: restarted 
```

这本手册可以作为配置任务的一部分由另一个手册调用，以在构建后配置主机。还可以添加额外的功能来启用更大的 Ansible 角色的方面。

# 用例 4 - 自动网络驱动器挂载（NFS，SMB）

现在我们将设置一些远程主机作为 NFS 和 SMB 客户端。我们还将配置一些驱动器以使用`AutoFS`自动连接，这是在之前的用例中安装的。以下代码安装依赖项，配置客户端，然后启动服务。这本手册适用于 Debian 和 Red Hat Linux 系列：

```
---
- name: Setup and connect network shared folders
  hosts: Linux
  become: yes
  gather_facts: yes
  tasks:
    - name: Install the dependencies to enable NFS and 
      SMB clients on Linux Debian family
      apt: 
         name: '{{ item }}'
         state: latest
      with_items:
         - nfs-common
         - rpcbind
         - cifs-utils
         - autofs
      when: ansible_os_family == 'Debian'

    - name: Install the dependencies to enable NFS and 
      SMB clients on Linux Red Hat family
      yum: 
         name: '{{ item }}'
         state: latest
      with_items:
         - nfs-common
         - rpcbind
         - cifs-utils
         - nfs-utils
         - nfs-utils-lib
         - autofs
      when: ansible_os_family == 'RedHat'

    - name: Block none authorised NFS servers using 
      rpcbind
      lineinfile: 
         path: /etc/hosts.deny
         line: "rpcbind: ALL"
         state: present
         create: yes

    - name: Allow the target NFS servers using rpcbind
      lineinfile: 
         path: /etc/hosts.allow
         line: "rpcbind: 192.168.10.20"
         state: present
         create: yes

    - name: Configure NFS share on Fstab
      mount: 
         name: nfs shared
         path: /nfs/shared
         src: "192.168.10.20:/media/shared"
         fstype: nfs
         opts: defaults
         state: present

    - name: Create the shared drive directories
      file:
         name: '{{ item }}'
         state: directory
         with_items:
         - '/nfs/shared'
         - '/cifs/winshared'

    - name: Configure NFS share on AutoFS
      lineinfile: 
         path: /etc/auto.nfs
         line: "shared -fstype=nfs,rw, 
         192.168.10.20:/media/shared”
         state: present

    - name: Configure SMB share on AutoFS
      lineinfile: 
         path: /etc/auto.cifs
         line: "winshared 
         -fstype=cifs,rw,noperm,credentials=/etc/crd.txt 
          ://192.168.11.20/winshared”
         state: present

    - name: Restart AutoFS service to apply NFS and SMB 
      changes
      systemd:
         name: autofs
         state: restarted
```

这本手册可以个性化，就像任何手册一样。例如，它可以安排在负责设置共享驱动器服务器的手册之后运行。

# 用例 5 - 重要文档的自动备份

在这个用例中，我们试图构建一个备份解决方案，它不会使用太多的带宽，通过归档需要备份的所有内容。我们基本上将选择一个要压缩并移动到安全主机的文件夹。以下代码确保安装了所有必要的依赖项，准备备份文件夹，压缩它，然后发送它。我们将使用一个称为同步的模块，它基本上是`rsync`的包装器，这个著名的数据同步工具。它经常用于提供快速备份解决方案：

```
---
- name: Setup and connect network shared folders
  hosts: Linux
  become: yes
  gather_facts: yes
  tasks:
    - name: Install the dependencies to for archiving the 
      backup
      package: 
         name: '{{ item }}'
         state: latest
      with_items:
         - zip
         - unzip
         - gunzip
         - gzip
         - bzip2
         - rsync

    - name: Backup the client folder to the vault 
      datastore server
      synchronize:
         mode: push 
         src: /home/client1
         dest: client@vault.lab.edu:/media/vault1/client1
         archive: yes
         copy_links: yes
         delete: no
         compress: yes
         recursive: yes
         checksum: yes
         links: yes
         owner: yes
         perms: yes
         times: yes
         set_remote_user: yes
         private_key: /home/admin/users_SSH_Keys/id_rsa
      delegate_to: "{{ inventory_hostname }}"
```

这本手册可以添加到`crontab`作业中，以安排定期备份到特定文件夹。

# 应用程序和服务的自动化

这个小节与上一个小节并没有太大不同，但它侧重于系统向外部世界提供的应用程序和服务，而不是与主机内部系统管理相关的内容。在这里，我们将介绍一些处理与应用程序或服务相关的任务的用例。

# 用例 1 - 设置具有一些预安装工具的 Linux 桌面环境

Linux 管理不仅限于管理服务器。如今，由于新的科学研究和其他复杂工具的出现，Linux GUI 用户正在增加。这些工具中有些需要使用终端，但也有一些需要 GUI 界面，例如显示 3D 渲染的分子结构。在这个第一个用例中，我们将制作一个手册，确保 Linux 主机具有特定用途所需的所有必要工具。这个脚本将安装一个简单的 Linux 图形界面，Openbox。这个脚本只兼容 Debian 系列的 Linux 系统，但也可以很容易地转换为支持 Red Hat 系列。

以下手册代码包括在 Linux 环境中设置应用程序的多种方式：

```
---
- name: Setup and connect network shared folders
  hosts: Linux
  become: yes
  gather_facts: yes
  tasks:
    - name: Install OpenBox graphical interface
      apt: 
         name: '{{ item }}'
         state: latest
         update_cache: yes
      with_items:
         - openbox
         - nitrogen
         - pnmixer
         - conky
         - obconf
         - xcompmgr
         - tint2

    - name: Install basic tools for desktop Linux usage 
     and application build
      apt: 
         name: '{{ item }}'
         state: latest
         update_cache: yes
      with_items:
         - htop
         - screen
         - libreoffice-base
         - libreoffice-calc
         - libreoffice-impress
         - libreoffice-writer
         - gnome-tweak-tool
         - firefox
         - thunderbird
         - nautilus
         - build-essential
         - automake
         - autoconf
         - unzip
         - python-pip
         - default-jre
         - cmake
         - git
         - wget
         - cpanminus
         - r-base
         - r-base-core
         - python3-dev
         - python3-pip
         - libgsl0-dev

    - name: Install tools using Perl CPAN
      cpanm:
          name: '{{ item }}'
      with_items:
         - Data::Dumper
         - File::Path
         - Cwd

    - name: Install tools using Python PyPip
      shell: pip3 install -U '{{ item }}'
      with_items:
         - numpy 
         - cython
         - scipy
         - biopython
         - pandas

    - name: Install tools on R CRAN using Bioconductor as 
      source 
      shell: Rscript --vanilla -e   
       "source('https://bioconductor.org/biocLite.R'); 
        biocLite(c('ggplots2', 'edgeR','optparse'), 
        ask=FALSE);"

    - name: Download a tool to be compiled on each host
      get_url: 
          url: http://cegg.unige.ch/pub/newick-utils-1.6-
          Linux-x86_64-enabled-extra.tar.gz 
          dest: /usr/local/newick.tar.gz
          mode: 0755

    - name: Unarchive the downloaded tool on each host
      unarchive: 
          src: /usr/local/newick.tar.gz
          dest: /usr/local/
          remote_src: yes
          mode: 0755

    - name: Configure the tool before to the host before 
      building
      command: ./configure chdir="/usr/local/newick-
      utils-1.6"

    - name: Build the tool on the hosts
      make:
          chdir: /usr/local/newick-utils-1.6
          target: install

    - name: Create Symlink to the tool’s binary to be 
      executable from anywhere in the system 
      shell: ln -s -f /usr/local/newick-utils-1.6/src
          /nw_display /usr/local/bin/nw_display

    - name: Installing another tool located into a github 
      repo
      git: 
          repo: https://github.com/chrisquince/DESMAN.git
          dest: /usr/local/DESMAN
          clone: yes

    - name: Setup the application using python compiler
      command: cd /usr/local/DESMAN; python3 setup.py install
```

这本手册可以在部署了几台主机后执行，可以在第一个脚本完成后调用它，也可以设置一个监视脚本来等待特定主机可用以启动这本手册。

# 用例 2 - LAMP 服务器设置和配置

这个用例自动化了通常由系统管理员手动执行的任务。使用以下手册，我们将设置一个 LAMP 服务器，基本上是一个 Web 服务器，Apache2；一个内容管理器 PHP；和一个数据库管理器，MySQL 服务器。我们还将添加一些插件和配置，符合最佳实践标准。以下脚本仅适用于 Debian Linux 系列：

```
---
- name: Install a LAMP on Linux hosts
  hosts: webservers
  become: yes
  gather_facts: yes
  tasks:
    - name: Install Lamp packages
      apt: 
         name: '{{ item }}'
         state: latest
         update_cache: yes
      with_items:
         - apache2
         - mysql-server
         - php
         - libapache2-mod-php
         - python-mysqldb

    - name: Create the Apache2 web folder
      file: 
         dest: "/var/www"
         state: directory
         mode: 0700
         owner: "www-data"
         group: "www-data"   

    - name: Setup Apache2 modules
      command: a2enmod {{ item }} creates=/etc/apache2
      /mods-enabled/{{ item }}.load
      with_items:
         - deflate
         - expires
         - headers
         - macro
         - rewrite
         - ssl

    - name: Setup PHP modules
      apt: 
         name: '{{ item }}'
         state: latest
         update_cache: yes
      with_items:
         - php-ssh2
         - php-apcu
         - php-pear
         - php-curl
         - php-gd
         - php-imagick
         - php-mcrypt
         - php-mysql
         - php-json

    - name: Remove MySQL test database
      mysql_db:  db=test state=absent login_user=root 
      login_password="DBp@55w0rd"

    - name: Restart mysql server
      service: 
         name: mysql
         state: restarted

    - name: Restart Apache2
      service: 
         name: apache2
         state: restarted
```

通过修改一些配置文件并填充 Apache2 网页文件夹，可以个性化此操作手册。

# Windows 基础设施自动化

使用 Ansible 操作手册，自动化 Windows 基础设施和自动化 Linux 基础设施一样容易。在本节中，我们将探讨一些自动化一些 Windows 管理任务的用例。

这些用例在 Windows 10 上进行了测试。可能需要额外的配置才能在 Windows 7 或 8 上运行。

# 系统管理自动化

在本小节中，我们将重点关注与 Windows 系统管理相关的用例。

# 用例 1-系统更新自动化

这个用例解决了 Windows 主机系统和一些应用程序更新的自动化。我们将通过禁用自动更新并仅更新允许的类别，使更新受到操作手册的限制：

```
---
- name: Windows updates management
  hosts: windows
  gather_facts: yes
  tasks:
   - name: Create the registry path for Windows Updates
     win_regedit:
       path: HKLM:\SOFTWARE\Policies\Microsoft\Windows
      \WindowsUpdate\AU
       state: present
     ignore_errors: yes

   - name: Add register key to disable Windows AutoUpdate
     win_regedit:
       path: HKLM:\SOFTWARE\Policies\Microsoft\Windows
      \WindowsUpdate\AU
       name: NoAutoUpdate
       data: 1
       type: dword
     ignore_errors: yes

    - name: Make sure that the Windows update service is 
      running
      win_service:
        name: wuauserv
        start_mode: auto
        state: started
      ignore_errors: yes

    - name: Executing Windows Updates on selected 
      categories
      win_updates:
        category_names:
          - Connectors
          - SecurityUpdates
          - CriticalUpdates
          - UpdateRollups
          - DefinitionUpdates
          - FeaturePacks
          - Application
          - ServicePacks
          - Tools
          - Updates
          - Guidance
        state: installed
        reboot: yes
      become: yes
      become_method: runas
      become_user: SYSTEM
      ignore_errors: yes
      register: update_result

    - name: Restart Windows hosts in case of update 
      failure 
      win_reboot:
      when: update_result.failed
```

可以安排此操作手册在非工作时间或计划维护期间执行。重新启动模块用于处理需要系统重新启动的 Windows 更新，因为它们需要系统重新启动而无法通过更新模块完成。通常，大多数更新将触发`require_reboot`的返回值，该值在安装更新后启动机器的重新启动。

# 用例 2-自动化 Windows 优化

这个模块在某种程度上是对系统的清理和组织。它主要针对桌面 Windows 主机，但是一些任务也可以用于服务器。

此操作手册将首先展示如何远程启动已关闭的 Windows 主机。然后等待直到它正常开机以进行磁盘碎片整理。之后，我们执行一些注册表优化任务，并最后将主机加入域：

```
---
- name: Windows system configuration and optimisation
  hosts: windows
  gather_facts: yes
  vars:
     macaddress: "{{ 
     (ansible_interfaces|first).macaddress|default
     (mac|default('')) }}"
      tasks:
   - name: Send magic Wake-On-Lan packet to turn on    
     individual systems
     win_wakeonlan:
       mac: '{{ macaddress }}'
       broadcast: 192.168.11.255

   - name: Wait for the host to start it WinRM service
     wait_for_connection:
       timeout: 20

   - name: start a defragmentation of the C drive
     win_defrag:
       include_volumes: C
       freespace_consolidation: yes

   - name: Setup some registry optimization
     win_regedit:
       path: '{{ item.path }}'
       name: '{{ item.name }}'
       data: '{{ item.data|default(None) }}'
       type: '{{ item.type|default("dword") }}'
       state: '{{ item.state|default("present") }}'
     with_items:

    # Set primary keyboard layout to English (UK)
    - path: HKU:\.DEFAULT\Keyboard Layout\Preload
      name: '1'
      data: 00000809
      type: string

    # Show files extensions on Explorer
    - path: HKCU:\Software\Microsoft\Windows
      \CurrentVersion\Explorer\Advanced
      name: HideFileExt
      data: 0

    # Make files and folders search faster on the 
      explorer
    - path: HKCU:\Software\Microsoft\Windows
     \CurrentVersion\Explorer\Advanced
      name: Start_SearchFiles
      data: 1

  - name: Add Windows hosts to local domain
    win_domain_membership:
      hostname: '{{ inventory_hostname_short }}'
      dns_domain_name: lab.edu
      domain_ou_path: lab.edu
      domain_admin_user: 'admin'
      domain_admin_password: '@dm1nP@55'
      state: domain
```

# 应用程序和服务自动化

在本小节中，我们将重点关注与 Chocolatey 存储库上可用的 Windows 应用程序相关的用例，以及出于各种原因我们希望传统安装的其他应用程序。

# 用例 1-自动化 Windows 应用程序管理

由于 Windows 一直缺乏软件包管理器，因此在 Windows 机器上管理应用程序可能会有些混乱。Chocolatey 是可以帮助解决此问题的解决方案之一。以下操作手册代码确保安装了 Chocolatey 的所有要求，然后检查由 Chocolatey 安装的所有应用程序的更新。最后，它安装新应用程序的最新版本。

建议在桌面型 Windows 主机上使用此用例，而不是服务器。但是也可以在服务器上使用，因为大多数 Windows 服务器现在也有图形界面。

以下操作手册代码显示了如何执行前述操作：

```
---
- name: Application management on Windows hosts
  hosts: windows
  gather_facts: yes
  tasks:
   - name: Install latest updated PowerShell for 
    optimized Chocolatey commands
     win_chocolatey:
       name: powershell
       state: latest

   - name: Update Chocolatey to its latest version
     win_chocolatey:
       name: chocolatey
       state: latest

   - name: Install a list of applications via Chocolatey
     win_chocolatey:
       name: "{{ item }}"
       state: latest
     with_items:
         - javaruntime
         - flashplayeractivex
         - 7zip
         - firefox
         - googlechrome
         - atom
         - notepadplusplus
         - vlc
         - adblockplus-firefox
         - adblockplus-chrome
         - adobereader
      ignore_errors: yes
```

在 Chocolatey 软件包索引网页（[`chocolatey.org/packages`](https://chocolatey.org/packages)）上提供了更多应用程序的列表。

此操作手册可用于为经常使用一些特定应用程序的特定用户设置通用镜像。

# 用例 2-设置 NSclient Nagios 客户端

我们总是向特定环境引入新设备。设置新主机的一个必要任务是将其链接到监控系统。对于这个用例，我们将展示如何在 Windows 主机上设置 Nagios 代理并从示例配置文件中进行配置：

```
---
- name: Setup Nagios agent on Windows hosts
  hosts: windows
  gather_facts: yes
  tasks:
   - name: Copy the MSI file for the NSClient to the 
     windows host
     win_copy:
       src: ~/win_apps/NSCP-0.5.0.62-x64.msi
       dest: C:\NSCP-0.5.0.62-x64.msi

   - name: Install an NSClient with the appropriate 
     arguments
     win_msi:
       path: C:\NSCP-0.5.0.62-x64.msi
       extra_args: ADDLOCAL=FirewallConfig,LuaScript,DotNetPluginSupport,Documentation,CheckPlugins,NRPEPlugins,NSCPlugins,NSCAPlugin,PythonScript,ExtraClientPlugin,SampleScripts ALLOWED_HOSTS=127.0.0.1,192.168.10.10 CONF_NSCLIENT=1 CONF_NRPE=1 CONF_NSCA=1 CONF_CHECKS=1 CONF_NSCLIENT=1 CONF_SCHEDULER=1 CONF_CAN_CHANGE=1 MONITORING_TOOL=none NSCLIENT_PWD=”N@g10sP@55w0rd”
        wait: true

   - name: Copying NSClient personalised configuration 
     file
     win_copy:
       src: ~/win_apps/conf_files/nsclient.ini
       dest: C:\Program Files\NSClient++\nsclient.ini

   - name: Change execution policy to allow the NSClient script remote Nagios execution
     raw: Start-Process powershell -verb RunAs -ArgumentList 'Set-ExecutionPolicy RemoteSigned -Force'

   - name: Restart the NSclient service to apply the 
     configuration change
     win_service:
       name: nscp
       start_mode: auto
       state: restarted

   - name: Delete the MSI file
     win_file: path=C:\NSCP-0.5.0.62-x64.msi state=absent
```

此操作手册可应用于使用 MSI 文件安装的大量应用程序。

# 网络自动化

就像计算机一样，如果网络设备运行某种远程服务，最好是 SSH，Ansible 可以用来自动化网络设备的管理。在本节中，我们将探讨一些关于思科网络设备的用例。我们将研究一些手动操作时耗时的各种任务。

# 用例 1-网络设备的自动打补丁

我们将按照升级网络设备的推荐方法进行操作。我们需要确保备份运行和启动配置。然后，我们将使用串行选项逐个设备进行打补丁：

```
---
- name: Patch CISCO network devices 
  hosts: ciscoswitches
  remote_user: admin
  strategy: debug
  connection: ssh
  serial: 1
  gather_facts: yes
  tasks:
    - name: Backup the running-config and the startup-
      config to the local machine
      ntc_save_config:
         local_file: "images/{{ inventory_hostname 
         }}.cfg"
         platform: 'cisco_ios_ssh'
         username: admin
         password: "P@55w0rd"
         secret: "5ecretP@55"
         host: "{{ inventory_hostname }}"

    - name: Upload binary file to the CISCO devices
      ntc_file_copy:
         local_file: " images/ios.bin'"
         remote_file: 'cXXXX-adventerprisek9sna.bin'
         platform: 'cisco_ios_ssh'
         username: admin
         password: "P@55w0rd"
         secret: "5ecretP@55"
         host: "{{ inventory_hostname }}"

    - name: Reload CISCO device to apply new patch
      ios_command:
         commands:
           - "reload in 5\ny"
         platform: 'cisco_ios_ssh'
         username: admin
         password: "P@55w0rd"
         secret: "5ecretP@55"
         host: "{{ inventory_hostname }}"
```

您可以创建一个名为 `provider` 的事实变量，其中包含有关要用于运行命令的设备的所有凭据和信息。定义变量可以最小化可以放入 playbook 中的代码量。

# 用例 2 - 在网络设备中添加新配置

在这个用例中，我们将更改 Cisco 设备上的一些通用配置。我们将更改主机名，创建横幅，升级 SSH 到版本 2，更改 Cisco VTP 模式，并配置 DNS 服务器和 NTP 服务器：

```
---
- name: Patch CISCO network devices 
  hosts: ciscoswitches
  become: yes
  become_method: enable
  ansible_connection: network_cli
  ansible_ssh_pass=admin
  ansible_become_pass=”P@55w0rd”
  ansible_network_os=ios
  strategy: debug
  connection: ssh
  serial: 1
  gather_facts: yes
  tasks:
    - name: Update network device hostname to match the 
      one used in the inventory
      ios_config:
         authorize: yes
         lines: ['hostname {{ inventory_hostname }}'] 
         force: yes

    - name: Change the CISCO devices login banner
      ios_config:
         authorize: yes
         lines:
            - banner motd ^This device is controlled via 
             Ansible. Please refrain from doing any 
             manual modification^

    - name: upgrade SSh service to version2
      ios_config:
         authorize: yes
         lines:
            - ip ssh version 2

    - name: Configure VTP to use transparent mode
      ios_config:
         authorize: yes
         lines:
            - vtp mode transparent

    - name: Change DNS servers to point to the Google DNS
      ios_config:
         authorize: yes
         lines:
            - ip name-server 8.8.8.8
            - ip name-server 8.8.4.4

    - name: Configure some realisable NTP servers
      ios_config:
         authorize: yes
         lines:
            - ntp server time.nist.gov
            - ntp server 0.uk.pool.ntp.org
```

建议在停机时间或计划维护窗口期间使用这些 playbook。一个设备的配置可能出错，但对其他设备来说完全正常。Ansible 摘要始终具有详细的执行状态，可以跟踪有问题的设备和任务。

# 云和容器基础设施的自动化

这一部分与资源管理更相关，而不是主机本身。之前的任何用例都可以用于本地或云上的裸机或虚拟主机。

在云或虚拟环境中，远程唤醒模块的用处较小。更容易使用专用模块来管理虚拟主机和实例。

# VMware 自动化

在这一小节中，我们将看一些 VMware 环境中主机管理的用例，包括管理它们周围的基础设施。

# 用例 1 - 从模板创建虚拟机

这个用例展示了如何从预定义模板创建虚拟机。之后，我们确保所有虚拟机都已根据正确的参数添加到清单中：

```
---
- name: Create a virtual machine from a template
  hosts: localhost
  gather_facts: False
  tasks:
    - name: Create a virtual machine
       vmware_guest:
          hostname: 'vcenter.edu.lab'
          username: 'vmadmin@lab.edu'
          password: 'VMp@55w0rd'
          datecenter: 'vcenter.edu.lab'
          validate_certs: no
          esxi_hostname: 'esxi1.lab.edu'
          template: ubuntu1404Temp
          folder: '/DeployedVMs'
          name: '{{ item.hostname }}'
          state: poweredon
          disk:
            - size_gb: 50
               type: thin
               datastore: 'datastore1'
          networks:
            - name: 'LabNetwork'
                ip: '{{ item.ip }}'
                netmask: '255.255.255.0'
                gateway: '192.168.13.1'
                dns_servers:
                  - '8.8.8.8'
                  - '8.8.4.4'
          hardware:
              memory_mb: '1024'
              num_cpus: '2'
          wait_for_ip_address: yes
        delegate_to: localhost
        with_items:
            - { hostname: vm1, ip: 192.168.13.10 }
            - { hostname: vm2, ip: 192.168.13.11 }
            - { hostname: vm3, ip: 192.168.13.12 }

    - name: add newly created VMs to the Ansible 
      inventory
       add_host:
          hostname: "{{ item.hostname }}"
          ansible_host: "{{ item.ip }}"
          ansible_ssh_user: setup
          ansible_ssh_pass: "L1nuxP@55w0rd"
          ansible_connection: ssh
          groupname: Linux
       with_items:
            - { hostname: vm1, ip: 192.168.13.10 }
            - { hostname: vm2, ip: 192.168.13.11 }
            - { hostname: vm3, ip: 192.168.13.12 }
```

此 playbook 中的项目可以通过使用预定义变量进行更改。

# 用例 2 - ESXi 主机和集群管理

我们现在将尝试进行一些更高级的基础设施管理。我们将尝试创建一个 VMware 集群并将一个 ESXi 主机添加到其中：

```
---
- name: Create a VMware cluster and populate it
  hosts: localhost
  gather_facts: False
  tasks:
    - name: Create a VMware virtual cluster
      vmware_cluster:
          hostname: 'vcenter.edu.lab'
          username: 'vmadmin@lab.edu'
          password: 'VMp@55w0rd'
          datecenter: 'vcenter.edu.lab'
          validate_certs: no
          cluster_name: "LabCluster"
          state: present
          enable_ha: yes 
          enable_drs: yes
          enable_vsan: no 

    - name: Add a VMware ESXi host to the newly created 
      Cluster
      vmware_host:
          hostname: 'vcenter.edu.lab'
          username: 'vmadmin@lab.edu'
          password: 'VMp@55w0rd'
          datecenter: 'vcenter.edu.lab'
          validate_certs: no
          cluster_name: " LabCluster "
          esxi_hostname: "esxi1.lab.edu"
          esxi_username: "root"
          esxi_password: "E5X1P@55w0rd"
          state: present
```

这些 playbook 可以替代用于管理 VCenter 的 PowerCLI 命令，也可以替代手动访问 Windows 客户端或 Web 界面来管理主机和集群的过程。

# 总结

在本章中，我们涵盖了许多有趣的用例，任何系统管理员在某个时候都需要运行。许多其他任务也可以执行，就像我们使用自定义 playbook 一样。但并非每个脚本都被认为是良好的自动化；重要的是正确的节点在较短的时间内从状态 A 转换到状态 B，没有错误。在 第六章 *Ansible 配置管理编码* 中，我们将学习一些基于最佳实践的高级脚本优化技术，以便充分利用 Ansible 自动化。

# 参考

Ansible 文档：[`docs.ansible.com/ansible/latest/`](https://docs.ansible.com/ansible/latest/)

Ansible GitHub 项目：[`github.com/ansible`](https://github.com/ansible)

Chocolatey 软件包索引：[`chocolatey.org/packages`](https://chocolatey.org/packages)
