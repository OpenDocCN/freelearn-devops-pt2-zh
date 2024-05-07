# Ansible Playbooks 和 Roles 介绍

根据维基百科，Ansible 是一个自动化软件配置、配置管理和应用部署的开源自动化引擎。但你已经知道这一点了。这本书是关于将 IT 自动化软件的理念应用于信息安全自动化领域。

本书将带领你踏上 *安全自动化* 之旅，展示 Ansible 在现实世界中的应用。

在这本书中，我们将以一种结构化、模块化的方式，使用一种简单易读的格式 YAML 自动化执行与安全相关的任务。最重要的是，你将学会创建的内容是可重复的。这意味着一旦完成，你可以专注于微调、扩展范围等。这个工具确保我们可以构建和销毁从简单应用堆栈到简单但广泛的多应用框架一切。

如果你已经玩过 Ansible，我们假设你已经这样做了，那么你肯定遇到了以下一些术语：

+   Playbook

+   Ansible 模块

+   YAML

+   Roles

+   模板（Jinja2）

别担心，我们将在本章节中解释所有上述术语。一旦你对这些主题感到舒适，我们将继续覆盖调度器工具，然后转向构建安全自动化 playbooks。

# 需记住的 Ansible 术语

像所有新的主题或话题一样，熟悉该主题或话题的术语是一个不错的主意。我们将介绍一些我们将在整本书中使用的 Ansible 术语，如果在任何时候你无法跟上，你可能需要回到这一章节，重新理解特定术语。

# Playbooks

在经典意义上，playbook 是关于足球比赛中的进攻和防守战术的。球员们会记录这些战术（行动计划），通常以图表形式呈现在一本书中。

在 Ansible 中，playbook 是一个 IT 过程的一系列有序步骤或指令。把它想象成一本可以被人类和计算机同时阅读和理解的精心撰写的说明书。

在随后的章节中，我们将专注于安全方面的自动化，引导我们构建简单和复杂的 playbooks。

一个 Ansible playbook 命令看起来像这样：

```
ansible-playbook -i inventory playbook.yml
```

暂时忽略`-i` 标志，并注意 playbook 文件的扩展名。

正如 [`docs.ansible.com/ansible/playbooks_intro.html`](http://docs.ansible.com/ansible/playbooks_intro.html) 中所述：

"Playbooks 以 YAML 格式表达（参见 YAML 语法 ([`docs.ansible.com/ansible/YAMLSyntax.html`](http://docs.ansible.com/ansible/YAMLSyntax.html)）, 且具有最少的语法，其旨在有意尝试不成为编程语言或脚本，而是一个配置或过程的模型。"

# Ansible 模块

Ansible 随附了一些模块（称为 **模块库**），可以直接在远程主机上执行或通过 playbook 执行。Playbook 中的任务调用模块来完成工作。

Ansible 有许多模块，其中大多数是由社区贡献和维护的。核心模块由 Ansible 核心工程团队维护，并将随着 Ansible 一起发布。

用户还可以编写自己的模块。这些模块可以控制系统资源，如服务、软件包或文件（实际上是任何东西），或者处理执行系统命令。

下面是 Ansible 提供的模块列表：[`docs.ansible.com/ansible/latest/modules_by_category.html#module-index`](http://docs.ansible.com/ansible/latest/modules_by_category.html#module-index)。

如果您使用 Dash ([`kapeli.com/dash`](https://kapeli.com/dash)) 或 Zeal ([`zealdocs.org/`](https://zealdocs.org/))，您可以下载离线版本以便参考。

模块也可以通过命令行执行。我们将使用模块来编写 playbook 中的所有任务。所有模块在技术上都返回 JSON 格式的数据。

模块应该是幂等的，如果检测到当前状态与期望的最终状态匹配，则应该避免进行任何更改。当使用 Ansible playbook 时，这些模块可以触发 *change events*，以通知 *handlers* 运行额外的任务。

每个模块的文档可以通过命令行工具 `ansible-doc` 访问：

```
$ ansible-doc apt

```

我们可以列出主机上所有可用的模块：

```
$ ansible-doc -l
```

通过执行 `httpd` 模块，在所有节点分组为 `webservers` 下启动 Apache Web 服务器。注意使用了 `-m` 标志：

```
$ ansible webservers -m service -a "name=httpd state=started"
```

这个片段展示了完全相同的命令，但是在一个 YAML 语法的 playbook 中：

```
- name: restart webserver
  service:
    name: httpd
    state: started
```

每个模块包含多个参数和选项，通过查看它们的文档和示例来了解模块的特性。

# 用于编写 Ansible playbook 的 YAML 语法

Ansible playbook 使用 **YAML** 编写，它代表 **YAML Ain't Markup Language**。

根据官方文档 ([`yaml.org/spec/current.html`](http://yaml.org/spec/current.html))：

"YAML Ain't Markup Language"（缩写为 YAML）是一种设计成对人类友好并且与现代编程语言一起使用的数据序列化语言，用于日常任务。

Ansible 使用 YAML 是因为它比其他常见的数据格式（如 XML 或 JSON）更易于人类阅读和编写。所有的 YAML 文件（无论它们是否与 Ansible 相关）都可以选择以 `---` 开始并以 `...` 结束。这是 YAML 格式的一部分，表示文档的开始和结束。

YAML 文件应该以 `.yaml` 或 `.yml` 结尾。YAML 区分大小写。

您还可以使用诸如 [www.yamllint.com](http://www.yamllint.com) 这样的 Linters，或者您的文本编辑器插件来进行 YAML 语法的代码检查，这有助于您排除任何语法错误等等。

这里是一个简单 playbook 的示例，展示了来自 Ansible 文档的 YAML 语法（[`docs.ansible.com/ansible/playbooks_intro.html#playbook-language-example`](http://docs.ansible.com/ansible/playbooks_intro.html#playbook-language-example)）：

```
- hosts: webservers
  vars:
    http_port: 80
    max_clients: 200
  remote_user: root

  tasks:
  - name: Ensure apache is at the latest version
    yum:
      name: httpd
      state: latest
  - name: Write the apache config file
    template:
      src: /srv/httpd.j2
      dest: /etc/httpd.conf

    notify:
    - restart apache

  - name: Ensure apache is running (and enable it at boot)
    service:
      name: httpd
      state: started
      enabled: yes

  handlers:
    - name: Restart apache
      service:
        name: httpd
        state: restarted
```

# Ansible 角色

虽然 playbook 提供了一种以预定义顺序执行 *plays* 的好方法，但 Ansible 上有一个很棒的功能将整个理念提升到完全不同的层次。角色是一种方便的方式来捆绑任务，支持文件和模板等支持资产，并带有一组自动搜索路径。

通过使用大多数程序员熟悉的 *包含* 文件和文件夹的概念，并指定被包含的内容，playbook 变得无限可读且易于理解。角色基本上由任务、处理程序和配置组成，但通过向 playbook 结构添加附加层，我们可以轻松获得大局观以及细节。

这样可以实现可重用的代码，并且在负责编写 playbook 的团队中分工明确。例如，数据库专家编写了一个角色（几乎像是一个部分 playbook）来设置数据库，安全专家则编写了一个加固此类数据库的角色。

虽然可以在一个非常大的文件中编写 playbook，但最终你会想要重用文件并开始组织事务。

大而复杂的 playbook 难以维护，很难重用大 playbook 的部分。将 playbook 拆分为角色允许非常高效的代码重用，并使 playbook 更容易理解。

在构建大 playbook 时使用角色的好处包括：

+   协作编写 playbooks

+   重用现有角色

+   角色可以独立更新、改进

+   处理变量、模板和文件更容易

**LAMP** 通常代表 **Linux，Apache，MySQL，PHP**。这是一种流行的软件组合，用于构建 Web 应用程序。如今，在 PHP 世界中另一个常见的组合是 **LEMP**，即 **Linux，NGINX，MySQL，PHP**。

这是一个可能的 LAMP 堆栈 `site.yml` 的示例：

```
- name: LAMP stack setup on ubuntu 16.04
  hosts: all
  gather_facts: False
  remote_user: "{{remote_username}}"
  become: yes

 roles:
   - common
   - web
   - db
   - php
```

注意角色列表。仅通过阅读角色名称，我们就可以了解该角色下可能包含的任务类型。

# 使用 Jinja2 的模板

Ansible 使用 Jinja2 模板来实现动态表达式和访问变量。在 playbook 和任务中使用 Jinja2 变量和表达式使我们能够创建非常灵活的角色。通过向这种方式编写的角色传递变量，我们可以使相同的角色执行不同的任务或配置。使用模板语言，比如 Jinja2，我们能够编写简洁且易于阅读的 playbooks。

通过确保所有模板化发生在 Ansible 控制器上，Jinja2 不需要在目标机器上。只复制所需的数据，这减少了需要传输的数据量。正如我们所知，数据传输量越少，通常执行速度越快，反馈越快。

# Jinja 模板示例

一个良好模板语言的标志是允许控制内容而不显得是完整的编程语言。Jinja2 在这方面表现出色，通过提供条件输出的能力，如使用循环进行迭代等等。

让我们看一些基本示例（显然是 Ansible playbook 相关的），看看是什么样子。

# 条件示例

仅当操作系统家族为 `Debian` 时执行：

```
tasks:
  - name: "shut down Debian flavored systems"
    command: /sbin/shutdown -t now
    when: ansible_os_family == "Debian"
```

# 循环示例

下面的任务使用 Jinja2 模板添加用户。这允许 playbooks 中的动态功能。我们可以使用变量在需要时存储数据，只需更新变量而不是整个 playbook：

```
- name: add several users
  user:
    name: "{{ item.name }}"
    state: present
    groups: "{{ item.groups }}"
  with_items:
    - { name: 'testuser1', groups: 'wheel' }
    - { name: 'testuser2', groups: 'root' }
```

# LAMP 栈 playbook 示例 - 结合所有概念

我们将看看如何使用我们迄今学到的技能编写一个 LAMP 栈 playbook。这是整个 playbook 的高层次结构：

```
inventory               # inventory file
group_vars/             #
   all.yml              # variables
site.yml                # master playbook (contains list of roles)
roles/                  #
    common/             # common role
        tasks/          #
            main.yml    # installing basic tasks
    web/                # apache2 role
        tasks/          #
            main.yml    # install apache
        templates/      #
            web.conf.j2 # apache2 custom configuration
        vars/           # 
            main.yml    # variables for web role 
        handlers/       #
            main.yml    # start apache2
    php/                # php role
        tasks/          # 
            main.yml    # installing php and restart apache2
    db/                 # db role
        tasks/          #
            main.yml    # install mysql and include harden.yml
            harden.yml  # security hardening for mysql
        handlers/       #
            main.yml    # start db and restart apache2
        vars/           #
            main.yml    # variables for db role
```

让我们从创建一个清单文件开始。下面的清单文件是使用静态手动输入创建的。这是一个非常基本的静态清单文件，我们将在其中定义一个主机，并设置用于连接到它的 IP 地址。

根据需要配置以下清单文件：

```
[lamp]
lampstack    ansible_host=192.168.56.10
```

下面的文件是 `group_vars/lamp.yml`，其中包含所有全局变量的配置：

```
remote_username: "hodor"
```

下面的文件是 `site.yml`，这是启动的主要 playbook 文件：

```
- name: LAMP stack setup on Ubuntu 16.04
 hosts: lamp
 gather_facts: False
 remote_user: "{{ remote_username }}"
 become: True

 roles:
   - common
   - web
   - db
   - php
```

下面是 `roles/common/tasks/main.yml` 文件，它将安装 `python2`、`curl` 和 `git`：

```
# In ubuntu 16.04 by default there is no python2
- name: install python 2
  raw: test -e /usr/bin/python || (apt -y update && apt install -y python-minimal)

- name: install curl and git
  apt:
    name: "{{ item }}"
    state: present
    update_cache: yes

  with_items:
    - curl
    - git
```

下面的任务，`roles/web/tasks/main.yml`，执行多个操作，如安装和配置 `apache2`。它还将服务添加到启动过程中：

```
- name: install apache2 server
  apt:
    name: apache2
    state: present

- name: update the apache2 server configuration
  template: 
    src: web.conf.j2
    dest: /etc/apache2/sites-available/000-default.conf
    owner: root
    group: root
    mode: 0644

- name: enable apache2 on startup
  systemd:
    name: apache2
    enabled: yes
  notify:
    - start apache2
```

`notify` 参数将触发 `roles/web/handlers/main.yml` 中找到的处理程序：

```
- name: start apache2
  systemd:
    state: started
    name: apache2

- name: stop apache2
  systemd:
    state: stopped
    name: apache2

- name: restart apache2
  systemd:
    state: restarted
    name: apache2
    daemon_reload: yes
```

模板文件将从 `role/web/templates/web.conf.j2` 中获取，该文件使用 Jinja 模板，还从本地变量中获取值：

```
<VirtualHost *:80><VirtualHost *:80>
    ServerAdmin {{server_admin_email}}
    DocumentRoot {{server_document_root}}

    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

本地变量文件位于 `roles/web/vars/main.yml`：

```
server_admin_email: hodor@localhost.local
server_document_root: /var/www/html
```

类似地，我们也会编写数据库角色。下面的文件 `roles/db/tasks/main.yml` 包括在提示时安装数据库服务器并分配密码。在文件末尾，我们包含了 `harden.yml`，它执行另一组任务：

```
- name: set mysql root password
  debconf:
    name: mysql-server
    question: mysql-server/root_password
    value: "{{ mysql_root_password | quote }}"
    vtype: password

- name: confirm mysql root password
  debconf: 
    name: mysql-server
    question: mysql-server/root_password_again
    value: "{{ mysql_root_password | quote }}"
    vtype: password

- name: install mysqlserver
  apt:
    name: "{{ item }}"
    state: present 
  with_items:
    - mysql-server
    - mysql-client

- include: harden.yml
```

`harden.yml` 执行 MySQL 服务器配置的加固：

```
- name: deletes anonymous mysql user
  mysql_user:
    user: ""
    state: absent
    login_password: "{{ mysql_root_password }}"
    login_user: root

- name: secures the mysql root user
  mysql_user: 
    user: root
    password: "{{ mysql_root_password }}"
    host: "{{ item }}"
    login_password: "{{mysql_root_password}}"
    login_user: root
 with_items:
   - 127.0.0.1
   - localhost
   - ::1
   - "{{ ansible_fqdn }}"

- name: removes the mysql test database
  mysql_db:
    db: test
    state: absent
    login_password: "{{ mysql_root_password }}"
    login_user: root

- name: enable mysql on startup
  systemd:
    name: mysql
    enabled: yes

  notify:
    - start mysql
```

`db` 服务器角色也有类似于 `web` 角色的 `roles/db/handlers/main.yml` 和本地变量：

```
- name: start mysql
  systemd:
    state: started
    name: mysql

- name: stop mysql
  systemd:
    state: stopped
    name: mysql

- name: restart mysql
  systemd:
    state: restarted
    name: mysql
    daemon_reload: yes
```

下面的文件是 `roles/db/vars/main.yml`，其中包含配置服务器时的 `mysql_root_password`。我们将在未来的章节中看到如何使用 `ansible-vault` 来保护这些明文密码：

```
mysql_root_password: R4nd0mP4$$w0rd
```

现在，我们将安装 PHP 并通过重新启动 `roles/php/tasks/main.yml` 服务来配置它与 `apache2` 一起工作：

```
- name: install php7
  apt:
    name: "{{ item }}"
    state: present
  with_items:
    - php7.0-mysql
    - php7.0-curl
    - php7.0-json
    - php7.0-cgi
    - php7.0
    - libapache2-mod-php7

- name: restart apache2
  systemd:
    state: restarted
    name: apache2
    daemon_reload: yes
```

要运行此剧本，我们需要在系统路径中安装 Ansible。请参阅[`docs.ansible.com/ansible/intro_installation.html`](http://docs.ansible.com/ansible/intro_installation.html)获取安装说明。

然后针对 Ubuntu 16.04 服务器执行以下命令来设置 LAMP 堆栈。当提示系统访问用户 `hodor` 时，请提供密码：

```
$ ansible-playbook -i inventory site.yml
```

完成剧本执行后，我们将准备在 Ubuntu 16.04 机器上使用 LAMP 堆栈。您可能已经注意到，每个任务或角色都可以根据我们在剧本中的需要进行配置。角色赋予了将剧本泛化并使用变量和模板轻松定制的能力。

# 摘要

我们已经使用 Ansible 的各种功能对一个相当不错的真实世界堆栈进行了编码。通过思考 LAMP 堆栈概述中的内容，我们可以开始创建角色。一旦我们确定了这一点，就可以将单个任务映射到 Ansible 中的模块。任何需要复制预定义配置但具有动态生成输出的任务都可以使用我们模板中的变量和 Jinja2 提供的结构来完成。

我们将使用同样的方法来进行各种安全相关的设置，这些设置可能需要一些自动化用于编排、操作等。一旦我们掌握了在运行于我们笔记本电脑上的虚拟机上执行此操作的方法，它也可以被重新用于部署到您喜欢的云计算实例上。输出是人类可读的文本，因此可以添加到版本控制中，各种角色也可以被重复使用。

现在，我们已经对本书中将要使用的术语有了相当不错的了解，让我们准备好最后一块拼图。在下一章中，我们将学习和理解如何使用自动化和调度工具，例如 Ansible Tower、Jenkins 和 Rundeck，根据某些事件触发器或时间持续时间来管理和执行基于剧本的操作。
