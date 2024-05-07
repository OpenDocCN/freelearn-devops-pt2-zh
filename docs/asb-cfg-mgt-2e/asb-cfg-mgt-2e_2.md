# 第二章：简单的 Playbooks

Ansible 可以作为一个命令行工具来进行小的更改。然而，它真正的力量在于其脚本能力。在设置机器时，我们几乎总是需要一次做多件事情。Ansible 使用名为**playbook**的概念来实现这一点。使用 playbooks，我们可以一次执行多个操作，并跨多个系统执行。它们提供了一种编排部署、确保一致的配置，或者简单执行常见任务的方式。

Playbooks 以**YAML**形式表示，大部分情况下，Ansible 使用标准的 YAML 解析器。这意味着我们在编写 playbooks 时可以使用 YAML 的所有功能。例如，我们可以在 playbook 中使用与 YAML 相同的注释系统。playbook 的许多行也可以用 YAML 数据类型编写和表示。有关更多信息，请参阅[`www.yaml.org/`](http://www.yaml.org/)。

Playbooks 还开启了许多机会。它们允许我们将状态从一个命令传递到另一个命令。例如，我们可以在一台机器上获取文件的内容，将其注册为变量，然后在另一台机器上使用该值。这使我们能够创建复杂的部署机制，这是仅使用 Ansible 命令无法实现的。此外，由于每个模块都试图是幂等的，我们应该能够多次运行 playbook，只有在需要时才会进行更改。

执行 playbook 的命令是`ansible-playbook`。它接受类似于 Ansible 命令行工具的参数。例如，`-k`（`--ask-pass`）和`-K`（`--ask-sudo`）会让 Ansible 分别提示输入 SSH 和 sudo 密码；`-u`可以用来设置 SSH 连接的用户名。然而，这些选项也可以在 playbook 的目标部分内设置。例如，要使用名为`example-play.yml`的 play，我们可以使用以下命令：

```
**$ ansible-playbook example-play.yml**

```

Ansible playbooks 由一个或多个 play 组成。一个 play 包括三个部分：

+   **目标部分**定义了 play 将在哪些主机上运行以及如何运行。这是我们设置 SSH 用户名和其他 SSH 相关设置的地方。

+   **变量部分**定义了在运行 play 时将可用的变量。

+   **任务部分**按照我们希望 Ansible 运行的顺序列出了所有模块。

我们可以在单个 YAML 文件中包含尽可能多的 play。YAML 文件以`---`开头，并包含许多键值和列表。在 YAML 中，行缩进用于指示变量嵌套给解析器，这也使文件更易于阅读。

一个完整的 Ansible play 示例如下代码片段所示：

```
---
- hosts: webservers
  user: root
  vars:
    apache_version: 2.6
    motd_warning: 'WARNING: Use by ACME Employees ONLY'
    testserver: yes
  tasks:
    - name: setup a MOTD
      copy:
        dest: /etc/motd
        content: "{{ motd_warning }}"
```

在接下来的几个部分中，我们将逐一检查每个部分，并详细解释它们的工作原理。

# 目标部分

目标部分看起来像以下代码片段：

```
- hosts: webservers
  user: root
```

这是一个非常简单的版本，但在大多数情况下可能是我们所需要的。每个 play 都存在于一个列表中。根据 YAML 语法，行必须以破折号开头。play 将要运行的主机必须在`hosts`的值中设置。这个值使用与使用 Ansible 命令行选择主机时相同的语法，我们在上一章中讨论过。Ansible 的主机模式匹配功能也在上一章中讨论过。在下一行中，用户告诉 Ansible playbook 要连接到机器的用户。

在这个部分中，我们可以提供的其他行如下：

| 名称 | 描述 |
| --- | --- |
| `sudo` | 如果要让 Ansible 在连接到 play 中的机器后使用`sudo`成为 root 用户，则将其设置为`yes`。 |
| `user` | 这定义了最初连接到机器的用户名，在配置了`sudo`之前。 |
| `sudo_user` | 这是 Ansible 将尝试使用`sudo`成为的用户。例如，如果我们将`sudo`设置为`yes`，`user`设置为`daniel`，将`sudo_user`设置为`kate`将导致 Ansible 在登录后使用`sudo`从`daniel`到`kate`。如果您在交互式 SSH 会话中执行此操作，我们可以在以`daniel`登录时使用`sudo -u kate`。 |
| `connection` | 这允许我们告诉 Ansible 要使用什么传输来连接到远程主机。我们将主要使用`ssh`或`paramiko`来连接远程主机。但是，当在`localhost`上运行时，我们也可以使用`local`来避免连接开销。大多数情况下，我们将在这里使用`local`、`winrm`或`ssh`。 |
| `gather_facts` | 除非我们告诉它不要这样做，否则 Ansible 将自动在远程主机上运行 setup 模块。如果我们不需要来自 setup 模块的变量，我们可以现在设置这个并节省一些时间。 |

# 变量部分

在这里，我们可以定义适用于所有机器上整个 play 的变量。我们还可以让 Ansible 提示变量，如果它们没有在命令行上提供。这使我们可以轻松地维护 play，并防止我们在 play 的几个部分中更改相同的内容。这也使我们可以将整个 play 的整个配置存储在顶部，这样我们可以轻松阅读和修改它，而不用担心 play 的其余部分。

play 中的这一部分变量可以被机器事实（由模块设置的事实）覆盖，但它们本身会覆盖我们在清单中设置的事实。因此，它们用于定义我们可能在稍后的模块中收集的默认值，但不能用于保留清单变量的默认值，因为它们将覆盖这些默认值。

变量声明发生在`vars`部分，看起来像目标部分中的值，并包含一个 YAML 字典或列表。一个例子看起来像以下代码片段：

```
vars:
  apache_version: 2.6
  motd_warning: 'WARNING: Use by ACME Employees ONLY'
  testserver: yes
```

变量也可以通过给 Ansible 提供要加载的变量文件列表来从外部 YAML 文件中加载。这是通过使用`vars_files`指令以类似的方式完成的。然后简单地提供另一个包含自己的字典的 YAML 文件的名称。这意味着，我们可以将变量存储和分发分开，从而可以与他人共享我们的 playbook。

使用`vars_files`，在我们的 playbook 中，文件看起来像以下代码片段：

```
vars_files:
  conf/country-AU.yml
  conf/datacenter-SYD.yml
  conf/cluster-mysql.yml
```

在前面的例子中，Ansible 会在与 playbook 路径相关的`conf`文件夹中查找`country-AU.yml`、`datacenter-SYD.yml`和`cluster-mysql.yml`。每个 YAML 文件看起来类似于以下代码片段：

```
---
ntp: ntp1.au.example.com
TZ: Australia/Sydney
```

最后，我们可以让 Ansible 与用户交互地询问每个变量。当我们有不希望用于自动化的变量，并且需要人工输入时，这是很有用的。一个有用的例子是提示输入用于解密 HTTPS 服务器的秘密密钥的密码短语。

我们可以使用以下代码片段指示 Ansible 提示变量：

```
vars_prompt:
  - name: https_passphrase
    prompt: Key Passphrase
    private: yes
```

在前面的例子中，`https_passphrase`是输入数据将被存储的地方。用户将被提示输入`Key Passphrase`，因为`private`设置为`yes`，所以当用户输入时，值不会在屏幕上显示。

我们可以使用`{{ variablename }}`来使用变量、事实和清单变量。我们甚至可以使用点表示法引用复杂的变量，比如字典。例如，一个名为`httpd`的变量，其中有一个名为`maxclients`的键，将被访问为`{{ httpd.maxclients }}`。这也适用于来自 setup 模块的事实。例如，我们可以使用`{{ ansible_eth0.ipv4.address }}`来获取名为`eth0`的网络接口的 IPv4 地址。

在变量部分设置的变量在同一 playbook 中的不同 play 之间不会保留。但是，由 setup 模块收集的事实或由`set_fact`设置的事实会保留。这意味着如果我们在同一台机器上运行第二个 play，或者在较早的 play 中运行机器的子集，我们可以在目标部分将`gather_facts`设置为`false`。`setup`模块有时可能需要一段时间才能运行，因此这可以显著加快 play 的速度，特别是在将串行设置为较低值的 play 中。

# 任务部分

任务部分是每个剧本的最后部分。它包含我们希望 Ansible 按照我们希望的顺序执行的操作列表。我们可以用几种风格来表达每个模块的参数。我们建议您尽可能坚持一种风格，并仅在必要时使用其他风格。这样可以使我们的 playbooks 更容易阅读和维护。以下代码片段展示了任务部分的三种风格：

```
tasks:
  - name: install apache
    action: yum name=httpd state=installed

  - name: configure apache
    copy: src=files/httpd.conf dest=/etc/httpd/conf/httpd.conf

  - name: restart apache
    service:
      name: httpd
      state: restarted
```

在这里，我们看到了三种不同的语法风格被用来在 CentOS 机器上安装、配置和启动 Apache web 服务器。第一个任务向我们展示了如何使用原始语法安装 Apache，这需要我们在`action`关键字内部首先调用模块。第二个任务使用了第二种风格，将 Apache 的配置文件复制到指定位置。在这种风格中，使用模块名称代替`action`关键字，其值简单地成为其参数。最后，第三种风格的最后一个任务展示了如何使用服务模块来重新启动 Apache。在这种风格中，我们像往常一样使用模块名称作为关键字，但我们将参数作为 YAML 字典提供。当我们向单个模块提供大量参数时，或者模块需要以复杂形式提供参数时，这种风格会很有用，比如云形成模块。后一种风格正在迅速成为编写 playbooks 的首选方式，因为越来越多的模块需要复杂的参数。在本书中，我们将使用这种风格，以节省示例的空间并防止行换行。

请注意，任务不需要名称。但是，它们可以成为良好的文档，并且在需要时允许我们稍后引用每个任务。当运行 playbook 时，名称也会输出到控制台，以便用户了解发生了什么。如果我们不提供名称，Ansible 将只使用任务或处理程序的动作行。

### 注意

与其他配置管理工具不同，Ansible 不提供完整的依赖系统。这既是一种福音也是一种诅咒；有了完整的依赖系统，我们可能永远不确定对特定机器将应用哪些更改。然而，Ansible 确保我们的更改将按照它们编写的顺序执行。因此，如果一个模块依赖于在其之前执行的另一个模块，只需在 playbook 中将一个放在另一个之前即可。

# 处理程序部分

处理程序部分在语法上与任务部分相同，并支持调用模块的相同格式。只有当调用处理程序的任务在执行过程中记录到有变化发生时，处理程序才会被调用。要触发处理程序，向任务添加一个 notify 关键字，其值设置为任务的名称。

当 Ansible 完成任务列表的运行时，处理程序将在先前触发时运行。它们按照处理程序部分中列出的顺序运行，即使它们在任务部分中被多次调用，它们也只会运行一次。这经常用于在升级和配置后重新启动守护程序。以下 play 演示了我们将如何将**ISC** **DHCP**（动态主机配置协议）服务器升级到最新版本、配置它并设置它在启动时启动。如果这个 playbook 在 ISC DHCP 守护程序已经运行最新版本并且配置文件没有改变的服务器上运行，处理程序将不会被调用，DHCP 也不会被重新启动。例如，考虑以下代码：

```
---
- hosts: dhcp
  tasks:
  - name: update to latest DHCP
    yum
      name: dhcp
      state: latest
    notify: restart dhcp

  - name: copy the DHCP config
    copy:
      src: dhcp/dhcpd.conf
      dest: /etc/dhcp/dhcpd.conf
    notify: restart dhcp

  - name: start DHCP at boot
    service:
      name: dhcpd
      state: started
      enabled: yes

  handlers:
  - name: restart dhcp
    service:
      name: dhcpd
      state: restarted
```

每个处理程序只能是一个单独的模块，但我们可以从单个任务中通知一系列处理程序。这使我们能够从任务列表中的单个步骤触发许多处理程序。例如，如果我们刚刚检出了任何 Django 应用的更新版本，我们可以设置一个处理程序来迁移数据库，部署静态文件并重新启动 Apache。我们可以通过在通知操作上简单使用 YAML 列表来实现这一点。这可能看起来像以下代码片段：

```
---
- hosts: qroud
  tasks:
  - name: checkout Qroud
    git:
      repo:git@github.com:smarthall/Qroud.git
      dest: /opt/apps/Qroud force=no
    notify:
      - migrate db
      - generate static
      - restart httpd

  handlers:
  - name: migrate db
    command: ./manage.py migrate –all
    args:
      chdir: /opt/apps/Qroud

  - name: generate static
    command: ./manage.py collectstatic -c –noinput
    args:
       chdir: /opt/apps/Qroud

  - name: restart httpd
    service:
      name: httpd
      state: restarted
```

我们可以看到`git`模块用于检出一些公共 GitHub 代码，如果导致任何更改，它会触发`migrate db`、`generate static`和`restart httpd`操作。

# playbook 模块

在 playbooks 中使用模块与在命令行中使用模块有一些不同。这主要是因为我们可以从先前的模块和`setup`模块中获得许多事实。某些模块在 Ansible 命令行中无法工作，因为它们需要访问这些变量。其他模块在命令行版本中可以工作，但在 playbook 中使用时可以提供增强功能。

## 模板模块

`template`模块是一个需要 Ansible 提供事实的最常用的模块之一。该模块允许我们设计配置文件的大纲，然后让 Ansible 在正确的位置插入值。为了实现这一点，Ansible 使用 Jinja2 模板语言。实际上，Jinja2 模板可以比这更复杂，包括条件语句、`for`循环和宏等内容。以下是一个用于配置 BIND 的 Jinja2 配置模板的示例：

```
# {{ ansible_managed }}
options {
  listen-on port 53 {
    127.0.0.1;
    {% for ip in ansible_all_ipv4_addresses %}
      {{ ip }};
    {% endfor %}
  };
  listen-on-v6 port 53 { ::1; };
  directory       "/var/named";
  dump-file       "/var/named/data/cache_dump.db";
  statistics-file "/var/named/data/named_stats.txt";
  memstatistics-file "/var/named/data/named_mem_stats.txt";
};

zone "." IN {
  type hint;
  file "named.ca";
};

include "/etc/named.rfc1912.zones";
include "/etc/named.root.key";

{# Variables for zone config #}
{% if 'authorativenames' in group_names %}
  {% set zone_type = 'master' %}
  {% set zone_dir = 'data' %}
{% else %}
  {% set zone_type = 'slave' %}
  {% set zone_dir = 'slaves' %}
{% endif %}

zone "internal.example.com" IN {
  type {{ zone_type }};
  file "{{ zone_dir }}/internal.example.com";
  {% if 'authorativenames' not in group_names %}
    masters { 192.168.2.2; };
  {% endif %}
};
```

按照惯例，Jinja2 模板的文件扩展名为`.j2`；然而，这并不是严格要求的。现在让我们将这个示例分解成其各个部分。示例从以下代码行开始：

```
# {{ ansible_managed }}
```

这一行在文件顶部添加了一个注释，显示文件来自哪个模板、主机、模板的修改时间和所有者。将这些作为注释放在模板中是一个好的做法，它确保人们知道如果他们希望永久更改它们应该编辑什么。

稍后，在第五行，有一个`for`循环：

```
    {% for ip in ansible_all_ipv4_addresses %}
      {{ ip }};
    {% endfor %}
```

`For`循环会遍历列表中的所有元素，每个元素遍历一次。它们可以选择将项目分配给我们选择的变量，以便我们可以在循环内部使用它。这个循环遍历`ansible_all_ipv4_addresses`中的所有值，这是由`setup`模块提供的一个列表，其中包含主机的所有 IPv4 地址。在`for`循环内部，它简单地将它们中的每一个添加到配置中，以确保 BIND 将在该接口上监听。

在第 24 行的模板中也可以添加注释。

```
{# Variables for zone config #}
```

在`{#`和`#}`之间的任何内容都会被 Jinja2 模板处理器忽略。这使我们可以在模板中添加注释，而这些注释不会出现在最终文件中。如果我们正在做一些复杂的事情，在模板中设置变量，或者配置文件不允许注释，这是特别方便的。

接下来的几行是`if`语句的一部分，为模板后面的使用设置了`zone_type`和`zone_dir`变量：

```
{% if 'authorativenames' in group_names %}
  {% set zone_type = 'master' %}
  {% set zone_dir = 'data' %}
{% else %}
  {% set zone_type = 'slave' %}
  {% set zone_dir = 'slaves' %}
{% endif %}
```

在`{% if %}`和`{% else %}`之间的任何内容，如果`if`标签中的语句为`false`，则会被忽略。在这里，我们检查值`authorativenames`是否在适用于此主机的组名称列表中。如果是`true`，则下面的两行将设置两个自定义变量。`zone_type`设置为 master，`zone_dir`设置为 data。如果此主机不在`authorativenames`组中，则`zone_type`和`zone_dir`将分别设置为`slave`和`slaves`。

最后，从第 33 行开始，我们提供了区域的实际配置：

```
zone "internal.example.com" IN {
  type {{ zone_type }};
  file "{{ zone_dir }}/internal.example.com";
  {% if zone_type == 'slave' %}
    masters { 192.168.2.2; };
  {% endif %}
};
```

我们将类型设置为我们之前创建的`zone_type`变量，并将位置设置为`zone_dir`。最后，我们检查区域类型是否为从属，如果是，我们将其主配置为特定的 IP 地址。

要使此模板设置权威名称服务器，我们需要在清单文件中创建一个名为`authorativenames`的组，并在其中添加一些主机。如何做到这一点在第一章中已经讨论过，*开始使用 Ansible*。

我们可以简单地调用`templates`模块和机器的事实将被发送，包括机器所在的组。这就像调用任何其他模块一样简单。`template`模块还接受类似`copy`模块的参数，如 owner、group 和 mode。例如考虑以下代码：

```
---
- name: Setup BIND
  host: allnames
  tasks:
  - name: configure BIND
    template: src=templates/named.conf.j2 dest=/etc/named.conf owner=root group=named mode=0640
```

## set_fact 模块

`set_fact`模块允许我们在 Ansible play 中在机器上构建自己的事实。然后可以在模板中使用这些事实或作为 playbook 中的变量。事实就像来自`setup`模块等模块的参数一样，它们是基于每个主机的。我们应该使用这个来避免将复杂的逻辑放入模板中。例如，如果我们试图配置一个缓冲区以占用内存的一定百分比，我们应该在 playbook 中计算该值。

以下示例显示了如何使用`set_fact`来配置 MySQL 服务器，使其具有大约机器上可用总内存的一半的 InnoDB 缓冲区大小：

```
---
- name: Configure MySQL
  hosts: mysqlservers
  tasks:
  - name: install MySql
    yum:
      name: mysql-server
      state: installed

  - name: Calculate InnoDB buffer pool size
    set_fact:
      innodb_buffer_pool_size_mb="{{ansible_memtotal_mb/2}}"

  - name: Configure MySQL
    template:
      src: templates/my.cnf.j2
      dest: /etc/my.cnf
      owner: root
      group: root
      mode: 0644
    notify: restart mysql

  - name: Start MySQL
    service:
      name: mysqld
      state: started
      enabled: yes

  handlers:
  - name: restart mysql
    service:
      name: mysqld
      state: restarted
```

这里的第一个任务只是使用 yum 安装 MySQL。第二个任务通过获取受管机器的总内存，除以二，去除任何非整数余数，并将其放入名为`innodb_buffer_pool_size_mb`的事实中。然后下一行将一个模板加载到`/etc/my.cnf`中以配置 MySQL。最后，启动 MySQL 并设置为在启动时启动。还包括一个处理程序，以在其配置更改时重新启动 MySQL。

然后模板只需要获取`innodb_buffer_pool_size`的值并将其放入配置中。这意味着我们可以在缓冲池应该是内存的五分之一或八分之一的地方重复使用相同的模板，并简单地更改那些主机的 playbook。在这种情况下，模板将看起来像以下代码片段：

```
# {{ ansible_managed }}
[mysqld]
datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock
# Disabling symbolic-links is recommended to prevent assorted security risks
symbolic-links=0
# Settings user and group are ignored when systemd is used.
# If we need to run mysqld under a different user or group,
# customize our systemd unit file for mysqld according to the
# instructions in http://fedoraproject.org/wiki/Systemd

# Configure the buffer pool
innodb_buffer_pool_size = {{ innodb_buffer_pool_size_mb|default(128) }}M

[mysqld_safe]
log-error=/var/log/mysqld.log
pid-file=/var/run/mysqld/mysqld.pid
```

我们可以看到在前面的模板中，我们只是将 play 中获取的变量放入模板中。如果模板没有看到`innodb_buffer_pool_size_mb`事实，它将简单地使用默认值`128`。

## 暂停模块

`pause`模块停止 playbook 的执行一段时间。我们可以配置它等待一段特定的时间，或者我们可以让它提示用户继续。当从 Ansible 命令行使用时实际上是无用的，但在 playbook 中使用时非常方便。

通常，当我们希望用户确认后继续，或者在特定点需要手动干预时，会使用`pause`模块。例如，如果我们刚刚将新版本的 Web 应用部署到服务器上，并且需要用户手动检查以确保它看起来正常，然后再配置它们接收生产流量之前，我们可以在那里设置一个暂停。它还可以方便地警告用户可能出现的问题，并给他们继续的选项。这将使 Ansible 打印出服务器的名称，并要求用户按*Enter*键继续。如果与目标部分中的 serial 键一起使用，它将针对 Ansible 正在运行的每组主机询问一次。这样，我们可以让用户以自己的节奏运行部署，同时他们可以交互式地监视进度。

不太有用的是，此模块可以简单地等待指定的一段时间。这并不总是有用，因为通常我们不知道特定操作可能需要多长时间，猜测可能会产生灾难性的结果。我们不应该用它来等待网络守护程序启动；相反，我们应该使用`wait_for`模块（在下一节中描述）来执行此任务。以下播放首先演示了在用户交互模式和定时模式中使用`pause`模块：

```
---
- hosts: localhost
  tasks:
  - name: wait on user input
    pause:
      prompt: "Warning! Press ENTER to continue or CTRL-C to quit."

  - name: timed wait
    pause:
      seconds: 30
```

## wait_for 模块

`wait_for`模块用于轮询特定的 TCP 端口，并且直到该端口接受远程连接后才继续执行。轮询是从远程机器进行的。如果我们只提供一个端口，或者将主机参数设置为`localhost`，则轮询将尝试连接到受控机器。我们可以利用`local_action`从控制器机器运行命令，并使用`ansible_hostname`变量作为我们的主机参数，使其尝试从控制器机器连接到受控机器。

此模块特别适用于需要一段时间才能启动的守护程序，或者我们希望在后台运行的事物。Apache Tomcat 附带一个 init 脚本，当我们尝试启动它时，它会立即返回，使 Tomcat 在后台启动。根据 Tomcat 配置加载的应用程序，它可能需要 2 秒到 10 分钟不等的时间才能完全启动并准备好接受连接。我们可以计时应用程序的启动并使用`pause`模块。然而，下一次部署可能需要更长或更短的时间，这将破坏我们的部署机制。使用`wait_for`模块，我们可以让 Ansible 识别 Tomcat 何时准备好接受连接。以下是一个执行此操作的播放：

```
---
- hosts: webapps
  tasks:
  - name: Install Tomcat
    yum:
      name: tomcat7
      state: installed

  - name: Start Tomcat
    service:
      name: tomcat7
      state: started

  - name: Wait for Tomcat to start
    wait_for:
      port: 8080
      state: started
```

在此播放完成后，Tomcat 应该已安装、启动并准备好接受请求。我们可以在此示例中追加更多模块，并依赖于 Tomcat 可用并监听。

## assemble 模块

`assemble`模块将受控机器上的多个文件组合在一起，并将它们保存到受控机器上的另一个文件中。在 playbooks 中，当我们有一个`config`文件不允许包含或在其包含中使用通配符时，这是有用的。对于例如 root 用户的`authorized_keys`文件非常有用。以下播放将发送一堆 SSH 公钥到受控机器，然后让它将它们全部组合在一起并放置在 root 用户的主目录中：

```
---
- hosts: all
  tasks:
  - name: Make a Directory in /opt
    file:
      path: /opt/sshkeys
      state: directory
      owner: root
      group: root
      mode: 0700

  - name: Copy SSH keys over
    copy:
      src: "keys/{{ item }}.pub"
      dest: "/opt/sshkeys/{{ item }}.pub"
      owner: root
      group: root
      mode: 0600
    with_items:
      - dan
      - kate
      - mal

  - name: Make the root users SSH config directory
    file:
      path: /root/.ssh
      state: directory
      owner: root
      group: root
      mode: 0700

  - name: Build the authorized_keys file
    assemble:
      src: /opt/sshkeys
      dest: /root/.ssh/authorized_keys
      owner: root
      group: root
      mode: 0700
```

到目前为止，这一切应该看起来很熟悉。我们可能会注意到任务中的`with_items`键以及`{{ items }}`变量。这些将在第三章中稍后解释，但现在我们需要知道的是，我们提供给`with_items`键的任何项目都将替换为`{{ items }}`变量，类似于`for`循环的工作方式。这简单地让我们一次轻松地将多个文件复制到远程主机。

最后一个任务展示了`assemble`模块的用法。我们将包含要连接到输出中的文件的目录作为`src`参数传递，然后将`dest`作为输出文件传递。它还接受许多创建文件的其他模块相同的参数（`owner`、`group`和`mode`）。它还按照`ls -1`命令列出的顺序组合文件。这意味着我们可以使用与`udev`和`rc.d`相同的方法，并在文件前面添加数字，以确保它们以正确的顺序结束。

## add_host 模块

`add_host`模块是 playbook 中可用的最强大的模块之一。`add_host`让我们可以在 play 中动态添加新的机器。我们可以使用`uri`模块从我们的**配置管理数据库**（**CMDB**）中获取主机，然后将其添加到当前 play 中。该模块还会将我们的主机添加到一个组中，如果该组尚不存在，则动态创建该组。

该模块简单地接受`name`和`groups`参数，这些参数相当不言自明，并设置主机名和组。我们还可以发送额外的参数，这些参数的处理方式与清单文件中的额外值的处理方式相同。这意味着我们可以设置`ansible_ssh_user`、`ansible_ssh_port`等。

如果我们使用云提供商，如 RackSpace 或 Amazon EC2，Ansible 中有可用的模块可以让我们管理计算资源。如果我们找不到它们在清单中，我们可能会决定在 play 开始时创建机器。如果我们这样做，我们可以使用此模块将机器添加到清单中，以便稍后对其进行配置。以下是使用 Google Compute 模块执行此操作的示例：

```
---
- name: Create infrastructure
  hosts: localhost
  connection: local
  tasks:
    - name: Make sure the mailserver exists
      gce:
        image: centos-6
        name: mailserver
        tags: mail
        zone: us-central1-a
      register: mailserver
      when: '"mailserver" not in groups.all'

    - name: Add new machine to inventory
      add_hosts:
        name: mailserver
        ansible_ssh_host: "{{ mailserver.instance_data[0].public_ip }}"
        groups: tag_mail
      when: not mailserver|skipped
```

## group_by 模块

除了在 play 中动态创建主机，我们还可以创建组。`group_by`模块可以根据关于机器的事实创建组，包括我们使用之前解释的`add_fact`模块设置的事实。`group_by`模块接受一个参数`key`，它接受机器将被添加到的组的名称。通过将其与变量的使用结合起来，我们可以使模块根据其操作系统、虚拟化技术或我们可以访问的任何其他事实将服务器添加到组中。然后我们可以在任何后续 play 的目标部分或模板中使用此组。

因此，如果我们想创建一个根据操作系统对主机进行分组的组，我们将调用该模块如下：

```
---
- name: Create operating system group
  hosts: all
  tasks:
    - group_by: key=os_{{ ansible_distribution }}

- name: Run on CentOS hosts only
  hosts: os_CentOS
  tasks:
  - name: Install Apache
    yum: name=httpd state=latest

- name: Run on Ubuntu hosts only
  hosts: os_Ubuntu
  tasks:
  - name: Install Apache
    apt: pkg=apache2 state=latest
```

然后我们可以使用这些组来使用正确的打包程序安装软件包。在实践中，这经常用于避免 Ansible 在执行时输出大量的“跳过”消息。我们可以创建一个组，用于应该发生操作的机器，而不是为每个需要跳过的任务添加`when`子句，然后使用一个单独的 play 来单独配置这些机器。以下是在不使用`when`子句的情况下在 Debian 和 RedHat 机器上安装 ssl 私钥的示例：

```
---
- name: Catergorize hosts
  hosts: all
  tasks:
    - name: Gather hosts by OS
      group_by:
        key: "os_{{ ansible_os_family }}"

- name: Install keys on RedHat
  hosts: os_RedHat
  tasks:
    - name: Install SSL certificate
      copy:
        src: sslcert.pem
        dest: /etc/pki/tls/private/sslcert.pem

- name: Install keys on Debian
  hosts: os_Debian
  tasks:
    - name: Install SSL certificate
      copy:
        src: sslcert.pem
        dest: /etc/ssl/private/sslcert.pem
```

## slurp 模块

`slurp`模块从远程系统抓取文件，使用 base 64 对其进行编码，然后返回结果。我们可以利用 register 关键字将内容放入事实中。在使用`slurp`模块获取文件时，我们应该注意文件大小。该模块将整个文件加载到内存中，因此使用`slurp`处理大文件可能会消耗所有可用的 RAM 并导致系统崩溃。文件还需要从受控机器传输到控制器机器，对于大文件，这可能需要相当长的时间。

将此模块与复制模块结合使用可以在两台机器之间复制文件。这在以下 playbook 中进行了演示：

```
---
- name: Fetch a SSH key from a machine
  hosts: bastion01
  tasks:
    - name: Fetch key
      slurp:
        src: /root/.ssh/id_rsa.pub
      register: sshkey

- name: Copy the SSH key to all hosts
  hosts: all
  tasks:
    - name: Make directory for key
      file:
        state: directory
        path: /root/.ssh
        owner: root
        group: root
        mode: 0700

    - name: Install SSH key
      copy:
        contents: "{{ hostvars.bastion01.sshkey|b64decode }}"
        dest: /root/.ssh/authorized_keys
        owner: root
        group: root
        mode: 0600
```

### 注意

请注意，由于`slurp`模块使用 base 64 对数据进行编码，因此我们必须使用名为`b64decode`的 jinja2 过滤器来在复制模块使用数据之前对数据进行解码。过滤器将在第三章*高级 Playbooks*中进行更详细的介绍。

# Windows playbook modules

Windows 支持是 Ansible 的新功能，因此没有为其提供许多模块。仅适用于 Windows 的模块以`win_`开头命名。还有一些可用的模块，可以在 Windows 和 Unix 系统上使用，例如我们之前介绍的`slurp`模块。

在 Windows 模块中，需要特别注意引用路径字符串。反斜杠是 YAML 中的重要字符，它们用于转义字符，并且在 Windows 路径中，它们表示目录。因此，YAML 可能会将我们路径的某些部分误解为转义序列。为了防止这种情况，我们在字符串上使用单引号。此外，如果我们的路径本身是一个目录，我们应该省略尾随的反斜杠，以便 YAML 不会将字符串的结尾误解为转义序列。如果我们必须以反斜杠结尾，那么将其变为双反斜杠，第二个将被忽略。以下是一些正确和不正确的字符串示例：

```
# Correct
'C:\Users\Daniel\Documents\secrets.txt'
'C:\Program Files\Fancy Software Inc\Directory'
'D:\\' # \\ becomes \
# Incorrect
"C:\Users\Daniel\newcar.jpg" # \n becomes a new line
'C:\Users\Daniel\Documents\' # \' becomes '
```

# 云基础设施模块

基础设施模块不仅允许我们管理机器的设置，还允许我们创建这些机器本身。除此之外，我们还可以自动化围绕它们的大部分基础设施。这可以作为对亚马逊云形成等服务的简单替代。

在创建我们希望在同一 playbook 中的后续 play 中管理的机器时，我们将希望使用`add_hosts`模块将机器添加到内存中的清单中，以便它可以成为进一步 play 的目标。我们可能还希望运行`group_by`模块，将它们排列成我们将其他机器排列的组。还应该使用`wait_for`模块来检查机器是否响应 SSH 连接，然后再尝试管理它。

云基础设施模块可能有点复杂，因此我们将展示如何设置和安装 Amazon 模块。有关如何配置其他模块的详细信息，请参阅其文档，使用`ansible-doc`。

## AWS 模块

AWS 模块的工作方式类似于大多数 AWS 工具的工作方式。这是因为它们使用了流行的 python **boto**库，该库与许多其他工具一起使用，并遵循了亚马逊发布的原始 AWS 工具的约定。

最好以与我们安装 Ansible 相同的方式安装 boto。对于大多数用例，我们将在托管的机器上运行模块，因此我们只需要在那里安装 boto 模块。我们可以以以下方式安装 boto 库：

+   Centos/RHEL/Fedora: `yum install python-boto`

+   Ubuntu: `apt-get install python-boto`

+   Pip: `pip install boto`

然后我们需要设置正确的环境变量。最简单的方法是在本地机器上使用 localhost 连接运行模块。如果我们这样做，那么我们的 shell 中的变量将被传递并自动可用于 Ansible 模块。这里是 boto 库用于连接到 AWS 的变量：

| 变量名 | 描述 |
| --- | --- |
| `AWS_ACCESS_KEY` | 这是有效 IAM 帐户的访问密钥 |
| `AWS_SECRET_KEY` | 这是对应于上面访问密钥的秘密密钥 |
| `AWS_REGION` | 这是默认区域，除非被覆盖 |

我们可以使用以下代码在我们的示例中设置这些环境变量：

```
export AWS_ACCESS_KEY="AKIAIOSFODNN7EXAMPLE"
export AWS_SECRET_KEY="wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY"
export AWS_REGION="us-east-1"
```

这些只是示例凭据，不起作用。一旦我们设置好这些，我们就可以使用 AWS 模块。在下一段代码中，我们将结合本章的几个模块来创建一个机器并将其添加到清单中。以下示例中使用了一些尚未讨论的功能，比如`register`和`delegate_to`，这些将在第三章 *高级 Playbooks*中介绍：

```
---
- name: Setup an EC2 instance
  hosts: localhost
  connection: local
  tasks:
    - name: Create an EC2 machine
      ec2:
        key_name: daniel-keypair
        instance_type: t2.micro
        image: ami-b66ed3de
        wait: yes
        group: webserver
        vpc_subnet_id: subnet-59483
        assign_public_ip: yes
      register: newmachines

    - name: Wait for SSH to start
      wait_for:
        host: "{{ newmachines.instances[0].public_ip }}"
        port: 22
        timeout: 300
      delegate_to: localhost

    - name: Add the machine to the inventory
      add_host:
        hostname: "{{ newmachines.instances[0].public_ip }}"
        groupname: new

- name: Configure the new machines
  hosts: new
  sudo: yes
  tasks:
    - name: Install a MOTD
      template:
        src: motd.j2
        dest: /etc/motd
```

# 总结

在本章中，我们介绍了 playbook 文件中可用的部分。我们还学习了如何使用变量使我们的 playbooks 易于维护，如何在进行更改时触发处理程序，最后，我们看了一下在 playbook 中使用某些模块时更有用的一些模块。您可以使用官方文档进一步探索 Ansible 提供的模块，网址为[`docs.ansible.com/modules_by_category.html`](http://docs.ansible.com/modules_by_category.html)。

在下一章中，我们将深入研究 playbooks 的更复杂功能。这将使我们能够构建更复杂的 playbooks，能够部署和配置整个系统。
