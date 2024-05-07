# 第三章：高级 playbooks

到目前为止，我们看到的 playbooks 都很简单，只是按顺序运行了一些模块。Ansible 允许更多地控制 playbook 的执行。使用以下技术，你应该能够执行甚至最复杂的部署：

+   并行运行操作

+   循环

+   条件执行

+   任务委派

+   额外变量

+   使用变量查找文件

+   环境变量

+   外部数据查找

+   存储数据

+   处理数据

+   调试 playbooks

# 并行运行操作

默认情况下，Ansible 最多只会分叉五次，因此一次只会在五台不同的机器上运行一个操作。如果你有大量的机器，或者你已经降低了最大分叉值，那么你可能希望异步启动任务。Ansible 执行此操作的方法是启动任务，然后轮询以等待其完成。这允许 Ansible 在所有所需的机器上启动作业，同时仍然使用最大分叉。

要并行运行操作，请使用`async`和`poll`关键字。`async`关键字触发 Ansible 并行运行任务，并且它的值将是 Ansible 等待命令完成的最长时间。`poll`的值表示 Ansible 轮询检查命令是否已完成的频率。

如果你想要在整个机群上运行`updatedb`，代码可能如下所示：

```
- hosts: all
  tasks:
    - name: Install mlocate
      yum: name=mlocate state=installed

    - name: Run updatedb
      command: /usr/bin/updatedb
      async: 300
      poll: 10
```

当你在超过五台机器上运行前面的例子时，你会注意到`yum`模块的行为与`command`模块不同。`yum`模块将在前五台机器上运行，然后在下一个五台机器上运行，依此类推。然而，`command`模块将在所有机器上运行，并在完成后指示状态。

如果你的命令启动了最终监听端口的守护进程，你可以启动它而不轮询，这样 Ansible 就不会检查它是否完成。然后你可以继续其他操作，并稍后使用`wait_for`模块检查完成情况。要配置 Ansible 不等待任务完成，将`poll`的值设置为`0`。

最后，如果你的任务运行时间非常长，你可以告诉 Ansible 等待任务完成的时间。为此，将`async`的值设置为`0`。

在以下情况下，你将想要使用 Ansible 的轮询：

+   你有一个可能会超时的长时间任务

+   你需要在大量机器上运行一个操作

+   你有一个不需要等待完成的操作

还有一些情况，你不应该使用`async`或`poll`：

+   如果你的任务获取了锁，阻止其他任务运行

+   你的任务只需要很短的时间运行

# 循环

Ansible 允许你多次重复使用一个模块，例如，如果你有几个应该设置相似权限的文件。这可以节省大量重复工作，并允许你迭代 facts 和 variables。

为此，你可以在操作上使用`with_items`关键字，并将值设置为你要迭代的项目列表。这将为模块创建一个名为`item`的变量，该变量将被设置为模块迭代时依次设置的每个项目。一些模块，如`yum`，将对此进行优化，以便它们不会为每个软件包执行单独的事务，而是一次性操作所有软件包。

使用`with_items`，代码如下：

```
tasks:
- name: Secure config files file:
    path: "/etc/{{ item }}"
    mode: 0600
    owner: root
    group: root with_items: - my.cnf - shadow - fstab
```

除了循环固定的项目或变量之外，Ansible 还为我们提供了一个称为**lookup 插件**的工具。这些插件允许你告诉 Ansible 从外部某处获取数据。例如，你可能想要找到所有匹配特定模式的文件，然后上传它们。

在这个例子中，我们上传目录中的所有公钥，然后将它们组合成 root 用户的`authorized_keys`文件，如下例所示：

```
tasks: - name: Make key directory file:
    path: /root/.sshkeys
    ensure: directory
    mode: 0700
    owner: root
    group: root - name: Upload public keys copy:
    src: "{{ item }}"
    dest: /root/.sshkeys
    mode: 0600
    owner: root
    group: root with_fileglob: - keys/*.pub - name: Assemble keys into authorized_keys file assemble:
    src: /root/.sshkeys
    dest: /root/.ssh/authorized_keys
    mode: 0600
    owner: root
    group: root
```

可以在以下情况下使用重复模块：

+   多次重复使用相似设置的模块

+   迭代列表的所有值

+   使用`assemble`模块创建许多文件，以便将其合并为一个大文件以供以后使用

+   与`with_fileglob`查找插件结合使用时，复制文件目录

# 条件执行

某些模块（例如`copy`模块）提供了配置以跳过模块执行的机制。您还可以配置自己的跳过条件，只有在解析为`true`时才执行模块。如果您的服务器使用不同的打包系统或具有不同的文件系统布局，则这可能很方便。它还可以与`set_fact`模块一起使用，以允许您计算许多不同的事情。

要跳过一个模块，您可以使用`when`键；这让您可以提供一个条件。如果您设置的条件解析为 false，则将跳过该模块。您分配给`when`的值是 Python 表达式。您可以在此时使用任何可用的变量或事实。

### 注意

如果要根据条件处理列表中的某些项目，只需使用`when`子句。`when`子句将单独处理列表中的每个项目；正在处理的项目可作为变量使用`{{ item }}`。

以下代码是一个示例，显示如何在 Debian 和 Red Hat 系统上选择`apt`和`yum`之间的选择。

```
---
- name: Install VIM
  hosts: all
  tasks:
    - name: Install VIM via yum
      yum:
        name: vim-enhanced
        state: installed
      when: ansible_os_family == "RedHat"

    - name: Install VIM via apt
      apt:
        name: vim
        state: installed
      when: ansible_os_family == "Debian"

    - name: Unexpected OS family
      debug:
        msg: "OS Family {{ ansible_os_family }} is not supported"
        fail: yes
      when: ansible_os_family != "RedHat" and ansible_os_family != "Debian"
```

还有第三个子句，用于打印消息并在未识别操作系统时失败。

### 注意

此功能可用于在特定点暂停，并等待用户干预以继续。通常，当 Ansible 遇到错误时，它将简单地停止正在进行的操作，而不运行任何处理程序。使用此功能，您可以添加`pause`模块，并在其中设置一个条件，以在意外情况下触发。这样，`pause`模块将在正常情况下被忽略；但是，在意外情况下，它将允许用户干预，并在安全时继续。任务将如下所示：

```
name: pause for unexpected conditions
pause: prompt="Unexpected OS"
when: ansible_os_family != "RedHat"
```

跳过操作有许多用途；以下是其中一些：

+   解决操作系统之间的差异

+   提示用户，然后执行他们请求的操作

+   通过避免不会改变任何内容但可能需要一段时间才能执行的模块来提高性能

+   拒绝更改具有特定文件的系统

+   检查自定义脚本是否已运行

# 任务委派

默认情况下，Ansible 会在配置的机器上同时运行其任务。当您有一堆单独的机器需要配置时，或者每台机器负责向其他远程机器通信其状态时，这非常有用。但是，如果您需要在与 Ansible 操作的主机不同的主机上执行操作，可以使用委派。

Ansible 可以配置为在除正在配置的主机之外的其他主机上运行任务，使用`delegate_to`键。模块仍将为每台机器运行一次，但是不会在目标机器上运行，而是在委派的主机上运行。可用的事实将适用于当前主机。在这里，我们展示了一个将使用`get_url`选项从一堆 Web 服务器下载配置的 playbook。

```
---
- name: Fetch configuration from all webservers
  hosts: webservers
  tasks:
    - name: Get config
      get_url:
        dest: "configs/{{ ansible_hostname }}"
        force: yes
        url: "http://{{ ansible_hostname }}/diagnostic/config"
      delegate_to: localhost
```

如果您正在委派给`localhost`，在定义操作时可以使用快捷方式，该快捷方式会自动使用本地机器。如果您将操作行的键定义为`local_action`，则委派给`localhost`是隐含的。如果我们在先前的示例中使用了这个功能，它会稍微缩短，并且看起来像这样：

```
--- #1
- name: Fetch configuration from all webservers     #2
  hosts: webservers     #3
  tasks:     #4
    - name: Get config     #5
      local_action: get_url dest=configs/{{ ansible_hostname }}.cfg url=http://{{ ansible_hostname }}/diagnostic/config     #6
```

委派不仅限于本地机器。您可以委派给清单中的任何主机。您可能希望委派的其他原因包括：

+   在部署之前从负载均衡器中删除主机

+   更改 DNS 以将流量从即将更改的服务器转移

+   在存储设备上创建 iSCSI 卷

+   使用外部服务器检查网络外部访问是否正常工作

# 额外变量

您可能已经在上一章的模板示例中看到我们使用了一个名为`group_names`的变量。这是 Ansible 本身提供的魔术变量之一。在撰写本文时，有七个这样的变量，这些变量将在接下来的章节中描述。

## hostvars 变量

`hostvars`变量允许您检索当前 play 已处理的所有主机的变量。如果在当前 play 中尚未在受管主机上运行`setup`模块，则只有其变量可用。您可以像访问其他复杂变量一样访问它，例如`${hostvars.hostname.fact}`，因此要获取名为`ns1`的服务器上运行的 Linux 发行版，它将是`${hostvars.ns1.ansible_distribution}`。以下示例将一个名为 zone master 的变量设置为名为`ns1`的服务器。然后调用`template`模块，该模块将使用此设置每个区域的主服务器。

```
---
- name: Setup DNS Servers
  hosts: allnameservers
  tasks:
    - name: Install BIND
      yum:
        name: named
        state: installed

- name: Setup Slaves
  hosts: slavenamesservers
  tasks:
    - name: Get the masters IP
      set_fact:
        dns_master: "{{ hostvars.ns1.ansible_default_ipv4.address }}"

    - name: Configure BIND
      template:
        dest: /etc/named.conf src: templates/named.conf.j2
```

### 注意

使用`hostvars`，您可以进一步将模板与环境分离。如果您嵌套变量调用，那么在 play 的变量部分放置 IP 地址的地方，您可以添加主机名。要查找名为`the_machine`变量中的机器的地址，您可以使用`{{ hostvars.[the_machine].default_ipv4.address }}`。

## groups 变量

`groups`变量包含按清单组分组的所有主机的列表。这使您可以访问您配置的所有主机。这是一个潜力非常强大的工具。它允许您在整个组中进行迭代，并对每个主机应用当前机器的操作。

```
---
- name: Configure the database
  hosts: dbservers
  user: root
  tasks:
    - name: Install mysql
      yum:
        name: "{{ item }}"
        state: installed
      with_items:
      - mysql-server
      - MySQL-python

    - name: Start mysql
      service:
        name: mysqld
        state: started
        enabled: true

    - name: Create a user for all app servers
      with_items: groups.appservers
      mysql_user:
        name: kate
        password: test
        host: "{{ hostvars.[item].ansible_eth0.ipv4.address }}" state: present
```

### 注意

`groups`变量不包含组中的实际主机；它包含表示其在清单中的名称的字符串。这意味着如果需要，您必须使用嵌套变量扩展来访问`hostvars`变量。

您甚至可以使用此变量为包含所有其他机器的`host`密钥的所有机器创建`known_hosts`文件。这将允许您从一台机器 SSH 到另一台机器，而无需确认远程主机的身份。它还将在机器离开服务或更换机器时处理删除机器或更新机器。以下是执行此操作的`known_hosts`文件的模板：

```
{% for host in groups['all'] %}
{{ hostvars[host]['ansible_hostname'] }}
{{ hostvars[host]['ansible_ssh_host_key_rsa_public'] }}
{% endfor %}
```

使用此模板的 playbook 将如下所示：

```
---
hosts: all
tasks:
- name: Setup known hosts
  hosts: all
  tasks:
    - name: Create known_hosts
      template:
        src: templates/known_hosts.j2 dest: /etc/ssh/ssh_known_hosts
        owner: root
        group: root mode: 0644
```

## group_names 变量

`group_names`变量包含当前主机所在的所有组的名称列表。这不仅对于调试很有用，而且对于检测组成员资格的条件也很有用。这在上一章中用于设置域名服务器。

此变量主要用于跳过任务或在模板中作为条件。例如，如果 SSH 守护程序有两种配置，一种安全，一种不太安全，但您只想在安全组中的机器上使用安全配置，您可以这样做：

```
- name: Setup SSH
  hosts: sshservers
  tasks:
    - name: For secure machines
      set_fact:
        sshconfig: files/ssh/sshd_config_secure
      when: "'secure' in group_names"

    - name: For non-secure machines
      set_fact:
        sshconfig: files/ssh/sshd_config_default
      when: "'secure' not in group_names"

    - name: Copy over the config
      copy:
        src: "{{ sshconfig }}"
        dest: /tmp/sshd_config
```

### 注意

在上一个示例中，我们使用`set_fact`模块为每种情况设置事实，然后使用`copy`模块。我们本可以在`set_facts`模块的位置使用`copy`模块，并少使用一个任务。之所以这样做是因为`set_fact`模块在本地运行，而`copy`模块在远程运行。当您首先使用`set_facts`模块并仅调用一次`copy`模块时，副本将并行在所有机器上制作。如果使用两个带条件的`copy`模块，那么每个都将在相关机器上单独执行。由于`copy`是这两个任务中较长的任务，因此它最能从并行运行中受益。

## inventory_hostname 变量

`inventory_hostname`变量存储了在清单中记录的服务器的主机名。如果你选择不在当前主机上运行`setup`模块，或者由于各种原因，`setup`模块检测到的值不正确，你应该使用这个。当你正在进行机器的初始设置并更改主机名时，这很有用。

## inventory_hostname_short 变量

`inventory_hostname_short`变量与前一个变量相同；但是，它只包括第一个点之前的字符。因此对于`host.example.com`，它将返回`host`。

## inventory_dir 变量

`inventory_dir`变量是包含清单文件的目录的路径名。

## inventory_file 变量

`inventory_file`变量与前一个变量相同，只是它还包括文件名。

# 使用变量查找文件

所有模块都可以通过解引用`{{`和`}}`将变量作为其参数的一部分。你可以使用这个基于变量加载特定文件。例如，你可能想要根据使用的架构选择不同的`config`文件用于 NRPE（Nagios 检查守护程序）。以下是它的样子：

```
---
- name: Configure NRPE for the right architecture
  hosts: ansibletest
  user: root
  tasks:
    - name: Copy in the correct NRPE config file
      copy:
        src: "files/nrpe.{{ ansible_architecture }}.conf" dest: "/etc/nagios/nrpe.cfg"
```

在`copy`和`template`模块中，你还可以配置 Ansible 查找一组文件，并且它会使用找到的第一个文件。这让你可以配置一个要查找的文件；如果找不到该文件，将使用第二个文件，依此类推直到列表末尾。如果找不到文件，那么模块将失败。该功能使用`first_available_file`键触发，并在操作中引用`{{ item }}`。以下代码是此功能的示例：

```
---
- name: Install an Apache config file
  hosts: ansibletest
  user: root
  tasks:
   - name: Get the best match for the machine
     copy:
       dest: /etc/apache.conf
       src: "{{ item }}"
     first_available_file:
      - "files/apache/{{ ansible_os_family }}-{{ ansible_architecture }}.cfg"
      - "files/apache/default-{{ ansible_architecture }}.cfg"
      - files/apache/default.cfg
```

### 注意

记住你可以从 Ansible 命令行工具运行 setup 模块。当你在 playbooks 或模板中大量使用变量时，这非常方便。要检查特定 play 可用的事实，只需复制主机模式的值并运行以下命令：

```
**ansible [host pattern] -m setup**

```

在 CentOS x86_64 机器上，这个配置将首先在通过`files/apache/`导航时查找`RedHat-x86_64.cfg`文件。如果该文件不存在，它将在通过`file/apache/`导航时查找`default-x86_64.cfg`文件，最后如果什么都不存在，它将尝试使用`default.cfg`。

# 环境变量

通常，Unix 命令利用某些环境变量。这些的普遍例子是 C makefiles、安装程序和 AWS 命令行工具。幸运的是，Ansible 使这变得非常容易。如果你想要在远程机器上上传文件到 Amazon S3，你可以设置 Amazon 访问密钥如下。你还会看到我们安装 EPEL 以便安装 pip，而 pip 用于安装 AWS 工具。

```
---
- name: Upload a remote file via S3
  hosts: ansibletest
  user: root
  tasks:
    - name: Setup EPEL
      command: >
        rpm -ivh http://download.fedoraproject.org/pub/epel/6/i386/ epel-release-6-8.noarch.rpm
        creates=/etc/yum.repos.d/epel.repo

    - name: Install pip
      yum:
        name: python-pip
        state: installed

    - name: Install the AWS tools
      pip:
        name: awscli
        state: present

    - name: Upload the file
      shell: >
        aws s3 put-object
        --bucket=my-test-bucket
        --key={{ ansible_hostname }}/fstab
        --body=/etc/fstab
        --region=eu-west-1
      environment:
        AWS_ACCESS_KEY_ID: XXXXXXXXXXXXXXXXXXX
        AWS_SECRET_ACCESS_KEY: XXXXXXXXXXXXXXXXXXXXX
```

### 注意

在内部，Ansible 将环境变量设置到 Python 代码中；这意味着任何已经使用环境变量的模块都可以利用这里设置的变量。如果你编写自己的模块，你应该考虑某些参数是否最好作为环境变量而不是参数来使用。

一些 Ansible 模块，如`get_url`、`yum`和`apt`，也会使用环境变量来设置它们的代理服务器。你可能希望设置环境变量的其他情况包括：

+   运行应用程序安装程序

+   在使用`shell`模块时将额外项添加到路径

+   从系统库搜索路径中未包含的位置加载库

+   在运行模块时使用`LD_PRELOAD`黑客

# 外部数据查找

Ansible 在 0.9 版中引入了查找插件。这些插件允许 Ansible 从外部源获取数据。Ansible 提供了几个插件，但你也可以编写自己的插件。这真的打开了大门，让你在配置中更加灵活。

查找插件是用 Python 编写的，并在控制机器上运行。它们以两种不同的方式执行：直接调用和`with_*`键。直接调用在您想像变量一样使用它们时很有用。使用`with_*`键在您想要将它们用作循环时很有用。在前面的部分中，我们介绍了`with_fileglob`，这是一个示例。

在下一个示例中，我们直接使用查找插件从`environment`中获取`http_proxy`的值，并将其发送到配置的机器。这确保我们正在配置的机器将使用相同的代理服务器下载文件。

```
---
- name: Downloads a file using a proxy
  hosts: all
  tasks:
    - name: Download file
      get_url:
        dest: /var/tmp/file.tar.gz url: http://server/file.tar.gz
      environment:
        http_proxy: "{{ lookup('env', 'http_proxy') }}"
```

### 注意

您还可以在变量部分使用查找插件。这不会立即查找结果并将其放入变量中，而是将其存储为宏，并在每次使用时查找。如果您使用的值可能随时间变化，这是很重要的。

在`with_*`形式中使用查找插件将允许您迭代通常无法迭代的内容。您可以使用任何此类插件，但返回列表的插件最有用。在下面的代码中，我们展示了如何动态注册`webapp` farm。

```
---
- name: Registers the app server farm
  hosts: localhost
  connection: local
  vars:
    hostcount: 5
  tasks:
   - name: Register the webapp farm
      local_action: add_host name={{ item }} groupname=webapp
      with_sequence: start=1 end={{ hostcount }} format=webapp%02x
```

如果您使用此示例，您将附加一个任务来创建每个虚拟机，然后创建一个新的 play 来配置它们。

查找插件有用的情况如下：

+   将整个 Apache 配置目录复制到`conf.d`样式目录

+   使用环境变量来调整 playbook 的操作

+   从 DNS TXT 记录获取配置

+   将命令的输出获取到一个变量中

# 存储结果

几乎每个模块都会输出一些内容，即使是`debug`模块也是如此。大多数情况下，唯一使用的变量是名为`changed`的变量。`changed`变量帮助 Ansible 决定是否运行处理程序，以及输出的颜色。但是，如果您希望，可以存储返回的值并在以后的 playbook 中使用它们。在这个例子中，我们查看`/tmp`目录中的模式，并创建一个名为`/tmp/subtmp`的新目录，其模式与此处显示的相同。

```
---
- name: Using register
  hosts: ansibletest
  user: root
  tasks:
    - name: Get /tmp info
      file:
        dest: /tmp
        state: directory
      register: tmp

    - name: Set mode on /var/tmp
      file:
        dest: /tmp/subtmp
        mode: "{{ tmp.mode }}"
        state: directory
```

一些模块，例如前面示例中的`file`模块，可以配置为仅提供信息。通过结合注册功能，您可以创建可以检查环境并计算如何进行的 playbook。

### 注意

结合注册功能和`set_fact`模块允许您对从模块返回的数据进行数据处理。这使您能够计算值并对这些值执行数据处理。这使您的 playbook 比以往更加智能和灵活。

注册允许您根据您已经可用的模块为主机创建自己的事实。这在许多不同的情况下都很有用：

+   获取远程目录中的文件列表并使用 fetch 下载它们

+   在前一个任务更改时运行任务，然后运行处理程序

+   获取远程主机 SSH 密钥的内容并构建`known_hosts`文件

# 处理数据

Ansible 使用 Jinja2 过滤器允许您以基本模板无法实现的方式转换数据。当 playbooks 中可用的数据不是我们想要的格式，或者在可以与模块或模板一起使用之前需要进一步的复杂处理时，我们使用过滤器。过滤器可以用于我们通常使用变量的任何地方，例如在模板中，作为模块的参数以及在条件语句中。通过提供变量名称、管道字符，然后是过滤器名称来使用过滤器。我们可以使用多个过滤器名称，用管道字符分隔，以使用多个管道，然后从左到右应用。下面是一个示例，我们确保所有用户都使用小写用户名创建：

```
---
- name: Create user accounts
  hosts: all
  vars:
    users:
  tasks:
    - name: Create accounts
      user: name={{ item|lower }} state=present
      with_items:
        - Fred
        - John
        - DanielH
```

以下是一些您可能会发现有用的流行过滤器：

| 过滤器 | 描述 |
| --- | --- |
| `min` | 当参数是一个列表时，它只返回最小值。 |
| `max` | 当参数是一个列表时，仅返回最大值。 |
| `random` | 当参数是一个列表时，它会从列表中随机选择一个项目。 |
| `changed` | 当在使用 register 关键字创建的变量上使用时，如果任务更改了任何内容，则返回`true`；否则返回`false`。 |
| `failed` | 当在使用 register 关键字创建的变量上使用时，如果任务失败，则返回`true`；否则返回`false`。 |
| `skipped` | 当在使用 register 关键字创建的变量上使用时，如果任务更改了任何内容，则返回`true`；否则返回`false`。 |
| `default(X)` | 如果变量不存在，则将使用 X 的值。 |
| `unique` | 当参数是一个列表时，返回一个没有重复项的列表。 |
| `b64decode` | 将变量中的 base64 编码字符串转换为其二进制表示。这在与 slurp 模块一起使用时非常有用，因为它将其数据作为 base64 编码的字符串返回。 |
| `replace(X, Y)` | 返回一个将字符串中任何出现的`X`替换为`Y`的副本。 |
| `join(X)` | 当变量是一个列表时，返回一个所有条目由`X`分隔的字符串。 |

# 调试 playbooks

有几种方法可以调试 playbook。Ansible 包括冗长模式和专门用于调试的`debug`模块。您还可以使用`fetch`和`get_url`等模块进行帮助。这些调试技术也可以用于检查模块在您希望学习如何使用它们时的行为。

## 调试模块

使用`debug`模块非常简单。它接受两个可选参数，`msg`和`fail.msg`，用于设置模块将打印的消息和`fail`，如果设置为`yes`，则表示对 Ansible 的失败，这将导致它停止处理该主机的 playbook。我们在前面的跳过模块部分中使用了此模块，以便在操作系统未被识别时退出 playbook。

在下面的示例中，我们将展示如何使用`debug`模块列出机器上所有可用的接口：

```
---
- name: Demonstrate the debug module
  hosts: ansibletest
  user: root
  vars:
    hostcount: 5
  tasks:
    - name: Print interface
      debug:
        msg: "{{ item }}"
      with_items: ansible_interfaces
```

上述代码给出了以下输出：

```
PLAY [Demonstrate the debug module] *********************************

GATHERING FACTS *****************************************************
ok: [ansibletest]

TASK: [Print interface] *********************************************
ok: [ansibletest] => (item=lo) => {"item": "lo", "msg": "lo"}
ok: [ansibletest] => (item=eth0) => {"item": "eth0", "msg": "eth0"}

PLAY RECAP **********************************************************
ansibletest                : ok=2    changed=0    unreachable=0    failed=0
```

正如您所看到的，`debug`模块很容易用于查看 play 期间变量的当前值。

## 冗长模式

调试的另一个选项是冗长选项。当使用冗长模式运行 Ansible 时，它会在运行后打印出每个模块返回的所有值。如果您在上一节中使用了`register`关键字，则这将特别有用。要在冗长模式下运行`ansible-playbook`，只需在命令行中添加`--verbose`即可。

```
**ansible-playbook --verbose playbook.yml**

```

## 检查模式

除了冗长模式，Ansible 还包括检查模式和差异模式。您可以通过在命令行中添加`--check`来使用检查模式，并使用`--diff`来使用差异模式。检查模式指示 Ansible 在实际上不对远程系统进行任何更改的情况下执行 play。这允许您获取 Ansible 计划对配置系统进行的更改的列表。

### 注意

这里需要注意的是，Ansible 的检查模式并不完美。任何不实现检查功能的模块都将被跳过。此外，如果跳过了提供更多变量的模块，或者变量取决于实际更改某些内容的模块（例如文件大小），那么它们将不可用。当使用`command`或`shell`模块时，这是一个明显的限制。

差异模式显示了`template`模块所做的更改。这是因为`template`文件只能处理文本文件。如果您要提供来自 copy 模块的二进制文件的差异，结果几乎无法阅读。差异模式还与检查模式一起工作，以显示由于处于检查模式而未进行的计划更改。

## 暂停模块

另一种技术是使用`pause`模块在检查配置的机器运行时暂停 playbook。这样，您可以在 play 的当前位置看到模块所做的更改，然后在其余的 play 继续执行时观察。

# 总结

在本章中，我们探讨了编写 playbooks 的更高级细节。现在，您应该能够使用委派、循环、条件和事实注册等功能，使您的 plays 更容易维护和编辑。我们还看了如何从其他主机访问信息，为模块配置环境，并从外部来源收集数据。最后，我们介绍了一些调试 plays 的技巧，以解决它们的行为与预期不符的问题。

在下一章中，我们将介绍如何在更大的环境中使用 Ansible。它将包括改进 playbooks 性能的方法，这些 playbooks 可能需要很长时间才能执行。我们还将介绍一些使 plays 易于维护的功能，特别是按目的将它们分成多个部分。
