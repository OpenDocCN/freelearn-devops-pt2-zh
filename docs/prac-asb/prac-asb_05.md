# playbook 和角色

到目前为止，在这本书中，我们主要使用临时的 Ansible 命令来简化操作，并帮助您理解基本原理。然而，Ansible 的生命线无疑是 playbook，它是任务的逻辑组织（类似于临时命令），以创建有用的结果的结构。这可能是在新建的虚拟机上部署 Web 服务器，也可能是应用安全策略。甚至可能处理虚拟机的整个构建过程！可能性是无限的。正如我们已经介绍过的，Ansible playbook 的设计是简单易写、易读——它们旨在自我记录，因此将成为您 IT 流程中宝贵的一部分。

在本章中，我们将更深入地探讨 playbook，从创建的基础知识到更高级的概念，如循环和块中运行任务、执行条件逻辑，以及 playbook 组织和代码重用中可能最重要的概念之一——Ansible 角色。我们将稍后更详细地介绍角色，但请知道，这是您在创建可管理的 playbook 代码时希望尽可能使用的内容。

具体来说，在本章中，我们将涵盖以下主题：

+   理解 playbook 框架

+   理解角色——playbook 的组织者

+   在代码中使用条件

+   使用循环重复任务

+   使用块分组任务

+   通过策略配置 play 执行

+   使用`ansible-pull`

# 技术要求

本章假设您已经按照第一章中详细介绍的方式在控制主机上安装了 Ansible，并且正在使用最新版本——本章中的示例是使用 Ansible 2.9 进行测试的。本章还假设您至少有一个额外的主机进行测试，并且最好是基于 Linux 的。尽管本章中将给出主机名的具体示例，但您可以自由地用自己的主机名和/或 IP 地址替换它们，如何做到这一点的详细信息将在适当的地方提供。

本章的代码包在此处可用：[`github.com/PacktPublishing/Ansible-2-Cookbook/tree/master/Chapter%204`](https://github.com/PacktPublishing/Ansible-2-Cookbook/tree/master/Chapter%204)。

# 理解 playbook 框架

playbook 允许您简单轻松地管理多台机器上的多个配置和复杂部署。这是使用 Ansible 交付复杂应用程序的关键优势之一。通过 playbook，您可以将任务组织成逻辑结构，因为任务通常按照编写的顺序执行，这使您能够对自动化过程有很好的控制。话虽如此，也可以异步执行任务，因此我们将强调任务不按顺序执行的情况。我们的目标是，一旦您完成本章，您将了解编写自己的 Ansible playbook 的最佳实践。

尽管 YAML 格式易于阅读和编写，但在间距方面非常严谨。例如，您不能使用制表符来设置缩进，即使在屏幕上，制表符和四个空格看起来可能相同——在 YAML 中，它们并不相同。如果您是第一次编写 playbook，我们建议您采用支持 YAML 的编辑器，例如 Vim、Visual Studio Code 或 Eclipse，这些编辑器将帮助您确保缩进正确。为了测试本章中开发的 playbook，我们将重复使用第三章中创建的清单的变体，*定义您的清单*（除非另有说明）：

```
[frontends]
frt01.example.com https_port=8443
frt02.example.com http_proxy=proxy.example.com

[frontends:vars]
ntp_server=ntp.frt.example.com
proxy=proxy.frt.example.com

[apps]
app01.example.com
app02.example.com

[webapp:children]
frontends
apps

[webapp:vars]
proxy_server=proxy.webapp.example.com
health_check_retry=3
health_check_interal=60
```

让我们立即开始编写一个 playbook。在第二章的*理解 Ansible 基础*中的*分解 Ansible 组件*一节中，我们涵盖了 playbook 的一些基本方面，因此我们不会在这里详细重复，而是在此基础上展示 playbook 开发的内容：

1.  创建一个简单的 playbook，在我们的清单文件中定义的`frontends`主机组中运行。我们可以在 playbook 中使用`remote_user`指令设置访问主机的用户，如下所示（您也可以在命令行上使用`--user`开关，但由于本章是关于 playbook 开发的，我们暂时忽略它）：

```
---
- hosts: frontends
  remote_user: danieloh

  tasks:
  - name: simple connection test
    ping:
    remote_user: danieloh
```

1.  在第一个任务下面添加另一个任务来运行`shell`模块（这将依次在远程主机上运行`ls`命令）。我们还将在这个任务中添加`ignore_errors`指令，以确保如果`ls`命令失败（例如，如果我们尝试列出的目录不存在），我们的 playbook 不会失败。小心缩进，并确保它与文件的第一部分匹配：

```
  - name: run a simple command
    shell: /bin/ls -al /nonexistent
    ignore_errors: True
```

让我们看看当我们运行时，我们新创建的 playbook 的行为如何：

```
$ ansible-playbook -i hosts myplaybook.yaml

PLAY [frontends] ***************************************************************

TASK [Gathering Facts] *********************************************************
ok: [frt02.example.com]
ok: [frt01.example.com]

TASK [simple connection test] **************************************************
ok: [frt01.example.com]
ok: [frt02.example.com]

TASK [run a simple command] ****************************************************
fatal: [frt02.example.com]: FAILED! => {"changed": true, "cmd": "/bin/ls -al /nonexistent", "delta": "0:00:00.015687", "end": "2020-04-10 16:37:56.895520", "msg": "non-zero return code", "rc": 2, "start": "2020-04-10 16:37:56.879833", "stderr": "/bin/ls: cannot access /nonexistent: No such file or directory", "stderr_lines": ["/bin/ls: cannot access /nonexistent: No such file or directory"], "stdout": "", "stdout_lines": []}
...ignoring
fatal: [frt01.example.com]: FAILED! => {"changed": true, "cmd": "/bin/ls -al /nonexistent", "delta": "0:00:00.012160", "end": "2020-04-10 16:37:56.930058", "msg": "non-zero return code", "rc": 2, "start": "2020-04-10 16:37:56.917898", "stderr": "/bin/ls: cannot access /nonexistent: No such file or directory", "stderr_lines": ["/bin/ls: cannot access /nonexistent: No such file or directory"], "stdout": "", "stdout_lines": []}
...ignoring

PLAY RECAP *********************************************************************
frt01.example.com : ok=3 changed=1 unreachable=0 failed=0 skipped=0 rescued=0 ignored=1
frt02.example.com : ok=3 changed=1 unreachable=0 failed=0 skipped=0 rescued=0 ignored=1 
```

从 playbook 运行的输出中，您可以看到我们的两个任务是按照指定的顺序执行的。我们可以看到`ls`命令失败，因为我们尝试列出一个不存在的目录，但是 playbook 没有注册任何`failed`任务，因为我们为这个任务设置了`ignore_errors`为`true`（仅针对这个任务）。

大多数 Ansible 模块（除了运行用户定义命令的模块，如`shell`、`command`和`raw`）都被编码为幂等的，也就是说，如果您运行相同的任务两次，结果将是相同的，并且任务不会进行相同的更改两次 - 如果检测到被请求执行的操作已经完成，那么它不会再次执行。当然，对于前述模块来说这是不可能的，因为它们可以用于执行几乎任何可以想象的任务 - 因此，模块如何知道它被执行了两次呢？

每个模块都会返回一组结果，其中包括任务状态。您可以在前面 playbook 运行输出的底部看到这些总结，它们的含义如下：

+   `ok`：任务成功运行，没有进行任何更改。

+   `changed`：任务成功运行，并进行了更改。

+   `failed`：任务运行失败。

+   `unreachable`：无法访问主机以运行任务。

+   `skipped`：此任务被跳过。

+   `ignored`：此任务被忽略（例如，在`ignore_errors`的情况下）。

+   `rescued`：稍后我们将在查看块和救援任务时看到一个例子。

这些状态可能非常有用，例如，如果我们有一个任务从模板部署新的 Apache 配置文件，我们知道必须重新启动 Apache 服务才能应用更改。但是，我们只想在文件实际更改时才这样做 - 如果没有进行任何更改，我们不希望不必要地重新启动 Apache，因为这会打断可能正在使用服务的人。因此，我们可以使用`notify`操作，告诉 Ansible 在任务结果为`changed`时（仅在此时）调用一个`handler`。简而言之，处理程序是一种特殊类型的任务，作为`notify`的结果而运行。但是，与按顺序执行的 Ansible playbook 任务不同，处理程序都被分组在一起，并在 play 的最后运行。此外，它们可以被通知多次，但无论如何只会运行一次，再次防止不必要的服务重启。考虑以下 playbook：

```
---
- name: Handler demo 1
  hosts: frt01.example.com
  gather_facts: no
  become: yes

  tasks:
    - name: Update Apache configuration
      template:
        src: template.j2
        dest: /etc/httpd/httpd.conf
      notify: Restart Apache

  handlers:
    - name: Restart Apache
      service:
        name: httpd
        state: restarted
```

为了保持输出简洁，我已经关闭了这个 playbook 的事实收集（我们不会在任何任务中使用它们）。出于简洁起见，我再次只在一个主机上运行，但您可以根据需要扩展演示代码。如果我们第一次运行这个任务，我们将看到以下结果：

```
$ ansible-playbook -i hosts handlers1.yml

PLAY [Handler demo 1] **********************************************************

TASK [Update Apache configuration] *********************************************
changed: [frt01.example.com]

RUNNING HANDLER [Restart Apache] ***********************************************
changed: [frt01.example.com]

PLAY RECAP *********************************************************************
frt01.example.com : ok=2 changed=2 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
```

请注意，当配置文件更新时，处理程序被运行。然而，如果我们再次运行这个 playbook，而没有对模板或配置文件进行任何更改，我们将看到类似以下的结果：

```
$ ansible-playbook -i hosts handlers1.yml

PLAY [Handler demo 1] **********************************************************

TASK [Update Apache configuration] *********************************************
ok: [frt01.example.com]

PLAY RECAP *********************************************************************
frt01.example.com : ok=1 changed=0 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
```

这一次，由于配置任务的结果是 OK，处理程序没有被调用。所有处理程序的名称应该是全局唯一的，这样通知操作才能调用正确的处理程序。您还可以通过设置一个公共名称来调用多个处理程序，使用`listen`指令——这样，您可以调用`name`或`listen`字符串中的任何一个处理程序，就像下面的示例中演示的那样：

```
---
- name: Handler demo 1
  hosts: frt01.example.com
  gather_facts: no
  become: yes

  handlers:
    - name: restart chronyd
      service:
        name: chronyd
        state: restarted
      listen: "restart all services"
    - name: restart apache
      service:
        name: httpd
        state: restarted
      listen: "restart all services"

  tasks:
    - name: restart all services
      command: echo "this task will restart all services"
      notify: "restart all services"
```

我们的 playbook 中只有一个任务，但当我们运行它时，两个处理程序都会被调用。另外，请记住我们之前说过的，`command`是一组特殊情况下的模块之一，因为它们无法检测到是否发生了更改——因此，它们总是返回`changed`值，因此，在这个演示 playbook 中，处理程序将始终被通知：

```
$ ansible-playbook -i hosts handlers2.yml

PLAY [Handler demo 1] **********************************************************

TASK [restart all services] ****************************************************
changed: [frt01.example.com]

RUNNING HANDLER [restart chronyd] **********************************************
changed: [frt01.example.com]

RUNNING HANDLER [restart apache] ***********************************************
changed: [frt01.example.com]

PLAY RECAP *********************************************************************
frt01.example.com : ok=3 changed=3 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
```

这些是您需要了解的一些基础知识，以开始编写自己的 playbooks。有了这些知识，让我们在下一节中比较临时命令和 playbooks。

# 比较 playbooks 和临时任务

临时命令允许您快速创建和执行一次性命令，而不保留任何已完成的记录（除了可能是您的 shell 历史）。这些命令具有重要的作用，并且在快速进行小改动和学习 Ansible 及其模块方面非常有价值。

相比之下，playbooks 是逻辑上组织的一系列任务（每个任务都可以是一个临时命令），按顺序组合在一起执行一个更大的动作。条件逻辑、错误处理等的添加意味着，很多时候，playbooks 的好处超过了临时命令的用处。此外，只要保持它们有组织，你将拥有你运行的所有以前的 playbooks 的副本，因此你将能够回顾（如果你需要的话）看看你运行了什么以及何时运行的。

让我们来开发一个实际的例子——假设你想在 CentOS 上安装 Apache 2.4。即使默认配置足够（这不太可能，但现在我们将保持例子简单），也涉及到一些步骤。如果你要手动执行基本安装，你需要安装软件包，打开防火墙，并确保服务正在运行（并且在启动时运行）。

要在 shell 中执行这些命令，您可能会这样做：

```
$ sudo yum install httpd
$ sudo firewall-cmd --add-service=http --permanent 
$ sudo firewall-cmd --add-service=https --permanent
$ sudo firewall-cmd --reload
$ sudo systemctl enable httpd.service
$ sudo systemctl restart httpd.service
```

现在，对于这些命令中的每一个，都有一个等效的临时 Ansible 命令可以运行。出于篇幅考虑，我们不会在这里逐个讨论它们；然而，假设你想要重新启动 Apache 服务——在这种情况下，你可以运行类似以下的临时命令（同样，为了简洁起见，我们只在一个主机上执行）：

```
$ ansible -i hosts frt01* -m service -a "name=httpd state=restarted"
```

当成功运行时，您将看到包含从以这种方式运行服务模块返回的所有变量数据的页面式 shell 输出。下面是一个片段供您检查您的结果——关键是命令导致`changed`状态，这意味着它成功运行，并且服务确实被重新启动了：

```
frt01.example.com | CHANGED => {
 "ansible_facts": {
 "discovered_interpreter_python": "/usr/bin/python"
 },
 "changed": true,
 "name": "httpd",
 "state": "started",
```

你可以创建并执行一系列临时命令来复制前面给出的六个 shell 命令，并分别运行它们。通过一些巧妙的方法，你应该可以将这个减少到六个命令（例如，Ansible 的`service`模块可以在一个临时命令中同时启用服务和重新启动它）。然而，你最终仍然会至少需要三到四个临时命令，如果你想在以后的另一台服务器上再次运行这些命令，你将需要参考你的笔记来弄清楚你是如何做的。

因此，playbook 是一种更有价值的方法来处理这个问题——它不仅会一次性执行所有步骤，而且还会为你记录下来以供以后参考。有多种方法可以做到这一点，但请将以下内容作为一个例子：

```
---
- name: Install Apache
  hosts: frt01.example.com
  gather_facts: no
  become: yes

  tasks:
    - name: Install Apache package
      yum:
        name: httpd
        state: latest
    - name: Open firewall for Apache
      firewalld:
        service: "{{ item }}"
        permanent: yes
        state: enabled
        immediate: yes
      loop:
        - "http"
        - "https"
    - name: Restart and enable the service
      service:
        name: httpd
        state: restarted
        enabled: yes
```

现在，当你运行这个时，你应该看到我们所有的安装要求都已经通过一个相当简单和易于阅读的 playbook 完成了。这里有一个新的概念，循环，我们还没有涉及，但不要担心，我们将在本章后面涉及到：

```
$ ansible-playbook -i hosts installapache.yml

PLAY [Install Apache] **********************************************************

TASK [Install Apache package] **************************************************
changed: [frt01.example.com]

TASK [Open firewall for Apache] ************************************************
changed: [frt01.example.com] => (item=http)
changed: [frt01.example.com] => (item=https)

TASK [Restart and enable the service] ******************************************
changed: [frt01.example.com]

PLAY RECAP *********************************************************************
frt01.example.com : ok=2 changed=3 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
```

正如你所看到的，这样做要比实际操作和记录在一个格式中更好，其他人可以很容易地理解。尽管我们将在书的后面讨论循环，但从前面的内容很容易看出它们是如何工作的。有了这个设置，让我们在下一节中更详细地看一下我们已经多次使用的一些术语，以确保你清楚它们的含义：**plays**和**tasks**。

# 定义 plays 和 tasks

到目前为止，当我们使用 playbook 时，我们一直在每个 playbook 中创建一个单一的 play（从逻辑上讲，这是你可以做的最少的）。然而，你可以在一个 playbook 中有多个 play，并且在 Ansible 术语中，“play”简单地是与主机（或主机组）相关联的一组任务（和角色、处理程序和其他 Ansible 方面）。任务是 play 的最小可能元素，负责使用一组参数运行单个模块以实现特定的目标。当然，在理论上，这听起来相当复杂，但在实际示例的支持下，它变得非常容易理解。

如果我们参考我们的示例清单，这描述了一个简单的两层架构（我们暂时忽略了数据库层）。现在，假设我们想编写一个单一的 playbook 来配置前端服务器和应用服务器。我们可以使用两个单独的 playbook 来配置前端和应用服务器，但这会使你的代码变得零散并且难以组织。然而，前端服务器和应用服务器（从它们的本质上）本质上是不同的，因此不太可能使用相同的任务集进行配置。

解决这个问题的方法是创建一个包含两个 play 的单一 playbook。每个 play 的开始可以通过最低缩进的行来识别（即在其前面没有空格）。让我们开始构建我们的 playbook：

1.  将第一个 play 添加到 playbook 中，并定义一些简单的任务来设置前端的 Apache 服务器，如下所示：

```
---
- name: Play 1 - configure the frontend servers
  hosts: frontends
  become: yes

  tasks:
  - name: Install the Apache package
    yum:
      name: httpd
      state: latest
  - name: Start the Apache server
    service:
      name: httpd
      state: started
```

1.  在同一个文件中，立即在下面添加第二个 play 来配置应用程序层服务器：

```
- name: Play 2 - configure the application servers
  hosts: apps
  become: true

  tasks:
  - name: Install Tomcat
    yum:
      name: tomcat
      state: latest
  - name: Start the Tomcat server
    service:
      name: tomcat
      state: started
```

现在，你有两个 plays：一个用于在`frontends`组中安装 web 服务器，另一个用于在`apps`组中安装应用服务器，全部合并成一个简单的 playbook。

当我们运行这个 playbook 时，我们将看到两个 play 按顺序执行，按照 playbook 中的顺序。请注意`PLAY`关键字的存在，它表示每个 play 的开始：

```
$ ansible-playbook -i hosts playandtask.yml

PLAY [Play 1 - configure the frontend servers] *********************************

TASK [Gathering Facts] *********************************************************
changed: [frt02.example.com]
changed: [frt01.example.com]

TASK [Install the Apache package] *********************************************
changed: [frt01.example.com]
changed: [frt02.example.com]

TASK [Start the Apache server] *************************************************
changed: [frt01.example.com]
changed: [frt02.example.com]

PLAY [Play 2 - configure the application servers] *******************************

TASK [Gathering Facts] *********************************************************
changed: [app01.example.com]
changed: [app02.example.com]

TASK [Install Tomcat] **********************************************************
changed: [app02.example.com]
changed: [app01.example.com]

TASK [Start the Tomcat server] *************************************************
changed: [app02.example.com]
changed: [app01.example.com]

PLAY RECAP *********************************************************************
app01.example.com : ok=3 changed=2 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
app02.example.com : ok=3 changed=2 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
frt01.example.com : ok=3 changed=2 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
frt02.example.com : ok=3 changed=2 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
```

我们有一个 playbook，但是有两个不同的 play 在提供的清单中操作不同的主机集。这非常强大，特别是与角色结合使用时（这将在本书的后面部分介绍）。当然，您的 playbook 中可以只有一个 play——您不必有多个 play，但是能够开发多 play playbook 非常重要，因为随着环境变得更加复杂，您几乎肯定会发现它们非常有用。

Playbooks 是 Ansible 自动化的生命线——它们将其扩展到不仅仅是单个任务/命令（它们本身就非常强大），而是一系列以逻辑方式组织的任务。然而，随着您扩展 playbook 库，您如何保持工作的组织？如何有效地重用相同的代码块？在前面的示例中，我们安装了 Apache，这可能是您的许多服务器的要求。但是，您应该尝试从一个 playbook 管理它们所有吗？或者您应该一遍又一遍地复制和粘贴相同的代码块？有一个更好的方法，在 Ansible 术语中，我们需要开始看角色，我们将在下一节中进行介绍。

# 理解角色——playbook 组织者

角色旨在使您能够高效有效地重用 Ansible 代码。它们始终遵循已知的结构，并且通常会包含变量、错误处理、处理程序等的合理默认值。以前一章中的 Apache 安装示例为例，我们知道这是我们可能想一遍又一遍地做的事情，也许每次都使用不同的配置文件，也许每台服务器（或每个清单组）都需要进行一些其他调整。在 Ansible 中，支持以这种方式重用代码的最有效方法是将其创建为一个角色。

创建角色的过程实际上非常简单——Ansible（默认情况下）将在您运行 playbook 的同一目录中寻找`roles/`目录，在这里，您将为每个角色创建一个子目录。角色名称源自子目录名称——无需创建复杂的元数据或其他任何东西——就是这么简单。在每个子目录中，都有一个固定的目录结构，告诉 Ansible 每个角色的任务、默认变量、处理程序等是什么。

`roles/`目录并不是 Ansible 寻找角色的唯一目录——这是它首先查找的目录，但然后它会在`/etc/ansible/roles`中查找任何额外的角色。这可以通过 Ansible 配置文件进一步定制，如第二章中所讨论的那样，*理解 Ansible 的基本原理*。

让我们更详细地探讨一下。考虑以下目录结构：

```
site.yml
frontends.yml
dbservers.yml
roles/
   installapache/
     tasks/
     handlers/
     templates/
     vars/
     defaults/
   installtomcat/
     tasks/
     meta/
```

前面的目录结构显示了在我们假设的 playbook 目录中定义的两个角色，名为`installapache`和`installtomcat`。在这些目录中，您会注意到一系列子目录。这些子目录不需要存在（稍后会详细说明它们的含义，但例如，如果您的角色没有处理程序，则无需创建`handlers/`）。但是，如果您确实需要这样的目录，您应该用名为`main.yml`的 YAML 文件填充它。每个`main.yml`文件都应该有特定的内容，具体取决于包含它们的目录。

角色中可以存在的子目录如下：

+   `tasks`：这是在角色中找到的最常见的目录，它包含角色应执行的所有 Ansible 任务。

+   `handlers`：角色中使用的所有处理程序都应该放在这个目录中。

+   `defaults`：角色的所有默认变量都放在这里。

+   `vars`：这些是其他角色变量——它们会覆盖`defaults/`目录中声明的变量，因为它们在优先顺序中更高。

+   `files`：角色需要的文件应该放在这里 - 例如，需要部署到目标主机的任何配置文件。

+   `templates`：与`files/`目录不同，这个目录应该包含角色使用的所有模板。

+   `meta`：角色所需的任何元数据都放在这里。例如，角色通常按照从父 playbook 调用它们的顺序执行 - 但是，有时角色会有需要先运行的依赖角色，如果是这种情况，它们可以在这个目录中声明。

对于我们在本章的这一部分中将开发的示例，我们将需要一个清单，所以让我们重用我们在上一节中使用的清单（以下是为了方便包含的）：

```
[frontends]
frt01.example.com https_port=8443
frt02.example.com http_proxy=proxy.example.com

[frontends:vars]
ntp_server=ntp.frt.example.com
proxy=proxy.frt.example.com

[apps]
app01.example.com
app02.example.com

[webapp:children]
frontends
apps

[webapp:vars]
proxy_server=proxy.webapp.example.com
health_check_retry=3
health_check_interal=60
```

让我们开始一些实际的练习，帮助你学习如何创建和使用角色。我们将首先创建一个名为`installapache`的角色，该角色将处理我们在上一节中看到的 Apache 安装过程。但是，在这里，我们将扩展它以涵盖在 CentOS 和 Ubuntu 上安装 Apache。这是一个很好的实践，特别是如果您希望将您的角色提交回社区，因为它们越通用（以及能够在更广泛的系统上运行），对人们就越有用。按照以下过程创建您的第一个角色：

1.  从您选择的 playbook 目录中创建`installapache`角色的目录结构 - 这就是这么简单：

```
$ mkdir -p roles/installapache/tasks
```

1.  现在，让我们在我们刚刚创建的`tasks`目录中创建一个必需的`main.yml`。这实际上不会执行 Apache 安装 - 而是在事实收集阶段检测到目标主机的操作系统后，将调用两个外部任务文件中的一个。我们可以使用这个特殊的变量`ansible_distribution`在`when`条件中确定要导入哪个任务文件：

```
---
- name: import a tasks based on OS platform 
 import_tasks: centos.yml 
  when: ansible_distribution == 'CentOS' 
- import_tasks: ubuntu.yml 
  when: ansible_distribution == 'Ubuntu'
```

1.  在`roles/installapache/tasks`中创建一个名为`centos.yml`的文件，以通过`yum`软件包管理器安装 Apache Web 服务器的最新版本。这应该包含以下内容：

```
---
- name: Install Apache using yum
 yum:
    name: "httpd"
    state: latest
- name: Start the Apache server
  service:
    name: httpd
    state: started 
```

1.  在`roles/installapache/tasks`中创建一个名为`ubuntu.yml`的文件，以通过`apt`软件包管理器在 Ubuntu 上安装 Apache Web 服务器的最新版本。注意在 CentOS 和 Ubuntu 主机之间内容的不同：

```
---
- name: Install Apache using apt
 apt:
    name: "apache2"
    state: latest
- name: Start the Apache server
  service:
    name: apache2
    state: started
```

目前，我们的角色代码非常简单 - 但是，您可以看到前面的任务文件就像一个 Ansible playbook，只是它们缺少了 play 定义。由于它们不属于一个 play，所以它们的缩进级别也比 playbook 中的低，但是除了这个差异，代码应该对您来说非常熟悉。事实上，这就是角色的美妙之处之一：只要您注意正确的缩进级别，您几乎可以在 playbook 或角色中使用相同的代码。

现在，角色不能自行运行 - 我们必须创建一个 playbook 来调用它们，所以让我们编写一个简单的 playbook 来调用我们新创建的角色。这与我们之前看到的一样有一个 play 定义，但是不是在 play 中有一个`tasks:`部分，而是有一个`roles:`部分，在那里声明了角色。惯例规定这个文件被称为`site.yml`，但您可以自由地称它为任何您喜欢的名字：

```
---
- name: Install Apache using a role
  hosts: frontends
  become: true

  roles:
    - installapache
```

为了清晰起见，您的最终目录结构应该如下所示：

```
.
├── roles
│   └── installapache
│   └── tasks
│   ├── centos.yml
│   ├── main.yml
│   └── ubuntu.yml
└── site.yml
```

完成后，您现在可以以正常方式使用`ansible-playbook`运行您的`site.yml` playbook - 您应该会看到类似于这样的输出：

```
$ ansible-playbook -i hosts site.yml

PLAY [Install Apache using a role] *********************************************

TASK [Gathering Facts] *********************************************************
ok: [frt01.example.com]
ok: [frt02.example.com]

TASK [installapache : Install Apache using yum] ********************************
changed: [frt02.example.com]
changed: [frt01.example.com]

TASK [installapache : Start the Apache server] *********************************
changed: [frt01.example.com]
changed: [frt02.example.com]

TASK [installapache : Install Apache using apt] ********************************
skipping: [frt01.example.com]
skipping: [frt02.example.com]

TASK [installapache : Start the Apache server] *********************************
skipping: [frt01.example.com]
skipping: [frt02.example.com]

PLAY RECAP *********************************************************************
frt01.example.com : ok=3 changed=2 unreachable=0 failed=0 skipped=2 rescued=0 ignored=0
frt02.example.com : ok=3 changed=2 unreachable=0 failed=0 skipped=2 rescued=0 ignored=0
```

就是这样 - 您已经在最简单的级别上创建了您的第一个角色。当然（正如我们之前讨论的那样），角色不仅仅是我们在这里添加的简单任务，还有更多内容，当我们在本章中进行工作时，我们将看到扩展的示例。然而，前面的示例旨在向您展示如何快速轻松地开始使用角色。

在我们看一些与角色相关的其他方面之前，让我们看一些调用角色的其他方法。当您编写 playbook 时，Ansible 允许您静态导入或动态包含角色。这两种导入或包含角色的语法略有不同，值得注意的是，两者都在 playbook 的任务部分而不是角色部分。以下是一个假设的示例，展示了一个非常简单的 playbook 中的两种选项。包括`common`和`approle`角色的角色目录结构将以与前面示例类似的方式创建：

```
--- 
- name: Play to import and include a role
 hosts: frontends

 tasks:
  - import_role:
      name: common
  - include_role:
      name: approle
```

这些功能在 2.3 之前的 Ansible 版本中是不可用的，并且它们在 2.4 版本中的使用方式略有改变，以保持与其他一些 Ansible 功能的一致性。我们不会在这里担心这些细节，因为现在 Ansible 的版本是 2.9，所以除非您绝对必须运行早期版本的 Ansible，否则可以假定这两个语句的工作方式如我们将在接下来的内容中概述的那样。

基本上，`import_role`语句在解析所有 playbook 代码时执行您指定的角色的静态导入。因此，使用`import_role`语句将角色引入您的 playbook 时，Ansible 在开始解析时将其视为 play 或角色中的任何其他代码一样。使用`import_role`基本上与在`site.yml`中的`roles:`语句之后声明您的角色一样，就像我们在前面的示例中所做的那样。

`include_role`在某种程度上与`import_role`有根本的不同，因为您指定的角色在解析 playbook 时不会被评估，而是在 playbook 运行期间动态处理，在遇到`include_role`时进行处理。

在选择前面提到的`include`或`import`语句之间最基本的原因可能是循环——如果您需要在循环内运行一个角色，您不能使用`import_role`，因此必须使用`include_role`。然而，两者都有好处和局限性，您需要根据您的情况选择最合适的方法——官方的 Ansible 文档（[`docs.ansible.com/ansible/latest/user_guide/playbooks_reuse.html#dynamic-vs-static`](https://docs.ansible.com/ansible/latest/user_guide/playbooks_reuse.html#dynamic-vs-static)）将帮助您做出正确的决定。

正如我们在本节中所看到的，角色非常简单易用，但却提供了一种非常强大的方式来组织和重用您的 Ansible 代码。在下一节中，我们将通过查看如何将角色特定的变量和依赖项添加到您的代码中来扩展我们简单的基于任务的示例。

# 设置基于角色的变量和依赖关系

变量是使 Ansible playbook 和角色可重用的核心，因为它们允许相同的代码以略有不同的值或配置数据重新利用。Ansible 角色目录结构允许在两个位置声明特定于角色的变量。虽然乍一看，这两个位置之间的区别可能并不明显，但它具有根本重要性。

基于角色的变量可以放在两个位置之一：

+   `defaults/main.yml`

+   `vars/main.yml`

这两个位置之间的区别在于它们在 Ansible 变量优先顺序中的位置。放在`defaults/`目录中的变量在优先级方面较低，因此很容易被覆盖。这个位置是你想要轻松覆盖的变量的位置，但你不想让变量未定义。例如，如果你要安装 Apache Tomcat，你可能会构建一个安装特定版本的角色。然而，如果有人忘记设置版本，你不希望角色因此退出错误，而是更愿意设置一个合理的默认值，比如`7.0.76`，然后可以用清单变量或命令行（使用`-e`或`--extra-vars`开关）轻松覆盖它。这样，即使没有人明确设置这个变量，你也知道角色可以正常工作，但如果需要，它可以很容易地更改为更新的 Tomcat 版本。

然而，放在`vars/`目录中的变量在 Ansible 的变量优先顺序中更靠前。这不会被清单变量覆盖，因此应该用于更重要的保持静态的变量数据。当然，这并不是说它们不能被覆盖——`-e`或`--extra-vars`开关是 Ansible 中优先级最高的，因此会覆盖你定义的任何其他内容。

大多数情况下，你可能只会使用基于`defaults/`的变量，但无疑会有时候，拥有更高优先级变量的选项对你的自动化变得更有价值，因此知道这个选项对你是可用的是至关重要的。

除了之前描述的基于角色的变量之外，还可以使用`meta/`目录为角色添加元数据。与之前一样，只需在这个目录中添加一个名为`main.yml`的文件即可。为了解释如何使用`meta/`目录，让我们构建并运行一个实际的例子，展示它如何被使用。在开始之前，重要的是要注意，默认情况下，Ansible 解析器只允许你运行一个角色一次。这在某种程度上类似于我们之前讨论的处理程序，可以被多次调用，但最终只在 play 结束时运行一次。角色也是一样的，它们可以被多次引用，但实际上只会运行一次。有两个例外情况——第一个是如果角色被多次调用，但使用了不同的变量或参数，另一个是如果被调用的角色在其`meta/`目录中将`allow_duplicates`设置为`true`。在构建示例时，我们将看到这两种情况的例子：

1.  在我们实际的例子的顶层，我们将有一个与本章节中一直在使用的清单相同的副本。我们还将创建一个名为`site.yml`的简单 playbook，其中包含以下代码：

```
---
- name: Role variables and meta playbook
  hosts: frt01.example.com

  roles:
    - platform
```

请注意，我们只是从这个 playbook 中调用了一个名为`platform`的角色，playbook 本身没有调用其他内容。

1.  让我们继续创建`platform`角色——与我们之前的角色不同，这个角色不包含任何任务，甚至不包含任何变量数据；相反，它只包含一个`meta`目录。

```
$ mkdir -p roles/platform/meta
```

在这个目录中，创建一个名为`main.yml`的文件，内容如下：

```
---
dependencies:
- role: linuxtype
  vars:
    type: centos
- role: linuxtype
  vars:
    type: ubuntu
```

这段代码将告诉 Ansible 平台角色依赖于`linuxtype`角色。请注意，我们指定了依赖两次，但每次指定时，我们都传递了一个名为`type`的变量，并赋予不同的值。这样，Ansible 解析器允许我们调用角色两次，因为每次作为依赖项引用时都传递了不同的变量值。

1.  现在让我们继续创建`linuxtype`角色，这将不包含任何任务，但会有更多的依赖声明：

```
$ mkdir -p roles/linuxtype/meta/
```

再次在`meta`目录中创建一个`main.yml`文件，但这次包含以下内容：

```
---
dependencies:
- role: version
- role: network
```

再次创建更多的依赖关系——这次，当调用`linuxtype`角色时，它反过来声明对称为`version`和`network`的角色的依赖。

1.  首先创建`version`角色——它将包含`meta`和`tasks`目录：

```
$ mkdir -p roles/version/meta
$ mkdir -p roles/version/tasks
```

在`meta`目录中，我们将创建一个包含以下内容的`main.yml`文件：

```
---
allow_duplicates: true
```

这个声明在这个例子中很重要——正如前面讨论的，通常情况下，Ansible 只允许一个角色被执行一次，即使它被多次调用。将`allow_duplicates`设置为`true`告诉 Ansible 允许角色被执行多次。这是必需的，因为在`platform`角色中，我们通过依赖两次调用了`linuxtype`角色，这意味着我们将两次调用`version`角色。

我们还将在任务目录中创建一个简单的`main.yml`文件，打印传递给角色的`type`变量的值：

```
---
- name: Print type variable
  debug:
    var: type
```

1.  现在我们将使用`network`角色重复这个过程——为了保持我们的示例代码简单，我们将使用与`version`角色相同的内容定义它：

```
$ mkdir -p roles/network/meta
$ mkdir -p roles/network/tasks
```

在`meta`目录中，我们将再次创建一个`main.yml`文件，其中包含以下内容：

```
---
allow_duplicates: true
```

再次在`tasks`目录中创建一个简单的`main.yml`文件，打印传递给角色的`type`变量的值：

```
---
- name: Print type variable
  debug:
    var: type
```

在这个过程结束时，你的目录结构应该是这样的：

```
.
├── hosts
├── roles
│   ├── linuxtype
│   │   └── meta
│   │       └── main.yml
│   ├── network
│   │   ├── meta
│   │   │   └── main.yml
│   │   └── tasks
│   │       └── main.yml
│   ├── platform
│   │   └── meta
│   │       └── main.yml
│   └── version
│       ├── meta
│       │   └── main.yml
│       └── tasks
│           └── main.yml
└── site.yml

11 directories, 8 files
```

让我们看看运行这个剧本会发生什么。现在，你可能会认为剧本会像这样运行：根据我们在前面的代码中创建的依赖结构，我们的初始剧本静态导入`platform`角色。`platform`角色然后声明依赖于`linuxtype`角色，并且每次使用名为`type`的变量声明两次不同的值。`linuxtype`角色然后声明依赖于`network`和`version`角色，这些角色可以运行多次并打印`type`的值。因此，你可能会认为我们会看到`network`和`version`角色被调用两次，第一次打印`centos`，第二次打印`ubuntu`（因为这是我们最初在`platform`角色中指定依赖关系的方式）。然而，当我们运行它时，实际上看到的是这样的：

```
$ ansible-playbook -i hosts site.yml

PLAY [Role variables and meta playbook] ****************************************

TASK [Gathering Facts] *********************************************************
ok: [frt01.example.com]

TASK [version : Print type variable] *******************************************
ok: [frt01.example.com] => {
 "type": "ubuntu"
}

TASK [network : Print type variable] *******************************************
ok: [frt01.example.com] => {
 "type": "ubuntu"
}

TASK [version : Print type variable] *******************************************
ok: [frt01.example.com] => {
 "type": "ubuntu"
}

TASK [network : Print type variable] *******************************************
ok: [frt01.example.com] => {
 "type": "ubuntu"
}

PLAY RECAP *********************************************************************
frt01.example.com : ok=5 changed=0 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
```

发生了什么？尽管我们看到`network`和`version`角色被调用了两次（如预期的那样），但`type`变量的值始终是`ubuntu`。这突出了关于 Ansible 解析器工作方式的重要一点，以及静态导入（我们在这里所做的）和动态包含（我们在前一节中讨论过）之间的区别。

使用静态导入时，角色变量的作用域就好像它们是在播放级别而不是角色级别定义的一样。角色本身在解析时都会被解析并合并到我们在`site.yml`剧本中创建的播放中，因此，Ansible 解析器会创建（在内存中）一个包含来自我们目录结构的所有合并变量和角色内容的大型剧本。这样做并没有错，但意味着`type`变量每次声明时都会被覆盖，因此我们声明的最后一个值（在这种情况下是`ubuntu`）是用于播放运行的值。

那么，我们如何使这个剧本按照我们最初的意图运行——加载我们的依赖角色，但使用我们为`type`变量定义的两个不同值？

这个问题的答案是，如果我们要继续使用静态导入的角色，那么在声明依赖关系时就不应该使用角色变量。相反，我们应该将`type`作为角色参数传递。这是一个小但至关重要的区别——即使在运行 Ansible 解析器时，角色参数仍然保持在角色级别上，因此我们可以在不覆盖变量的情况下声明我们的依赖两次。要做到这一点，将`roles/platform/meta/main.yml`文件的内容更改为以下内容：

```
---
dependencies:
- role: linuxtype
  type: centos
- role: linuxtype
  type: ubuntu
```

您注意到微妙的变化了吗？`vars:`关键字消失了，`type`的声明现在处于较低的缩进级别，这意味着它是一个角色参数。现在，当我们运行 playbook 时，我们得到了我们所希望的结果：

```
$ ansible-playbook -i hosts site.yml

PLAY [Role variables and meta playbook] ****************************************

TASK [Gathering Facts] *********************************************************
ok: [frt01.example.com]

TASK [version : Print type variable] *******************************************
ok: [frt01.example.com] => {
 "type": "centos"
}

TASK [network : Print type variable] *******************************************
ok: [frt01.example.com] => {
 "type": "centos"
}

TASK [version : Print type variable] *******************************************
ok: [frt01.example.com] => {
 "type": "ubuntu"
}

TASK [network : Print type variable] *******************************************
ok: [frt01.example.com] => {
 "type": "ubuntu"
}

PLAY RECAP *********************************************************************
frt01.example.com : ok=5 changed=0 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
```

这是一个相当高级的 Ansible 角色依赖示例，但是提供给您是为了演示了解变量优先级（即变量的作用域）和解析器工作的重要性。如果您编写简单的、按顺序解析的任务，那么您可能永远不需要了解这一点，但我建议您广泛使用调试语句，并测试您的 playbook 设计，以确保在 playbook 开发过程中不会遇到这种问题。

在对角色的许多方面进行了详细的研究之后，让我们在下一节中看一下一个用于公开可用的 Ansible 角色的集中存储——Ansible Galaxy。

# Ansible Galaxy

没有关于 Ansible 角色的部分会完整无缺地提到 Ansible Galaxy。Ansible Galaxy 是 Ansible 托管的一个由社区驱动的 Ansible 角色集合，托管在[`galaxy.ansible.com/`](https://galaxy.ansible.com/)。它包含了许多社区贡献的 Ansible 角色，如果您能构想出一个自动化任务，很有可能已经有人编写了一个角色来完全满足您的需求。它非常值得探索，并且可以让您的自动化项目迅速起步，因为您可以开始使用一组现成的角色。

除了网站之外，`ansible-galaxy`客户端也包含在 Ansible 中，这为您提供了一种快速便捷的方式，让您下载并部署角色到您的 playbook 结构中。假设您想要在目标主机上更新**每日消息**（**MOTD**）—这肯定是有人已经想出来的事情。在 Ansible Galaxy 网站上快速搜索返回（在撰写本文时）106 个设置 MOTD 的角色。如果我们想使用其中一个，我们可以使用以下命令将其下载到我们的角色目录中：

```
$ ansible-galaxy role install -p roles/ arillso.motd
```

这就是您需要做的一切——一旦下载完成，您可以像在本章讨论的手动创建的角色一样，在 playbook 中导入或包含角色。请注意，如果您不指定`-p roles/`，`ansible-galaxy`会将角色安装到`~/.ansible/roles`，这是您的用户帐户的中央角色目录。当然，这可能是您想要的，但如果您希望将角色直接下载到 playbook 目录结构中，您可以添加此参数。

另一个巧妙的技巧是使用`ansible-galaxy`为您创建一个空的角色目录结构，以便您在其中创建自己的角色——这样可以节省我们在本章中一直在进行的所有手动目录和文件创建，就像在这个例子中一样：

```
$ ansible-galaxy role init --init-path roles/ testrole
- Role testrole was created successfully
$ tree roles/testrole/
roles/testrole/
├── defaults
│   └── main.yml
├── files
├── handlers
│   └── main.yml
├── meta
│   └── main.yml
├── README.md
├── tasks
│   └── main.yml
├── templates
├── tests
│   ├── inventory
│   └── test.yml
└── vars
 └── main.yml 
```

这应该为您提供足够的信息，让您开始进入 Ansible 角色的旅程。我无法再次强调开发代码作为角色是多么重要——最初可能看起来不重要，但随着您的自动化用例的扩展，以及重用代码的需求增长，您会为自己的决定感到高兴。在下一节中，让我们扩展一下对 Ansible playbook 的讨论，讨论条件逻辑在您的 Ansible 代码中的使用方式。

# 在您的代码中使用条件

到目前为止，在我们的大多数示例中，我们创建了一组简单的任务集，这些任务总是运行。然而，当你生成你想要应用于更广泛主机数组的任务（无论是在角色还是 playbooks 中），迟早你会想要执行某种条件动作。这可能是只对先前任务的结果执行任务。或者可能是只对从 Ansible 系统中收集的特定事实执行任务。在本节中，我们将提供一些实际的条件逻辑示例，以演示如何在你的 Ansible 任务中应用这个特性。

和以往一样，我们需要一个清单来开始，并且我们将重用本章中一直使用的清单：

```
[frontends]
frt01.example.com https_port=8443
frt02.example.com http_proxy=proxy.example.com

[frontends:vars]
ntp_server=ntp.frt.example.com
proxy=proxy.frt.example.com

[apps]
app01.example.com
app02.example.com

[webapp:children]
frontends
apps

[webapp:vars]
proxy_server=proxy.webapp.example.com
health_check_retry=3
health_check_interal=60
```

假设你只想在某些操作系统上执行 Ansible 任务。我们已经讨论了 Ansible 事实，这为在 playbooks 中探索条件逻辑提供了一个完美的平台。考虑一下：所有你的 CentOS 系统都发布了一个紧急补丁，你想立即应用它。当然，你可以逐个创建一个专门的清单（或主机组）来适用于 CentOS 主机，但这是你不一定需要做的额外工作。

相反，让我们定义一个将执行我们的更新的任务，但在一个简单的示例 playbook 中添加一个包含 Jinja 2 表达式的`when`子句：

```
---
- name: Play to patch only CentOS systems
  hosts: all
  become: true

  tasks:
  - name: Patch CentOS systems
    yum:
      name: httpd
      state: latest
    when: ansible_facts['distribution'] == "CentOS"
```

现在，当我们运行这个任务时，如果你的测试系统是基于 CentOS 的（我的也是），你应该会看到类似以下的输出：

```
$ ansible-playbook -i hosts condition.yml

PLAY [Play to patch only CentOS systems] ***************************************

TASK [Gathering Facts] *********************************************************
ok: [frt02.example.com]
ok: [app01.example.com]
ok: [frt01.example.com]
ok: [app02.example.com]

TASK [Patch CentOS systems] ****************************************************
ok: [app01.example.com]
changed: [frt01.example.com]
ok: [app02.example.com]
ok: [frt02.example.com]

PLAY RECAP *********************************************************************
app01.example.com : ok=2 changed=0 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
app02.example.com : ok=2 changed=0 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
frt01.example.com : ok=2 changed=1 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
frt02.example.com : ok=2 changed=0 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
```

前面的输出显示，我们所有的系统都是基于 CentOS 的，但只有`frt01.example.com`需要应用补丁。现在我们可以使我们的逻辑更加精确——也许只有我们的旧系统运行在 CentOS 6 上需要应用补丁。在这种情况下，我们可以扩展 playbook 中的逻辑，检查发行版和主要版本，如下所示：

```
---
- name: Play to patch only CentOS systems
  hosts: all
  become: true

  tasks:
  - name: Patch CentOS systems
    yum:
      name: httpd
      state: latest
    when: (ansible_facts['distribution'] == "CentOS" and ansible_facts['distribution_major_version'] == "6")
```

现在，如果我们运行我们修改后的 playbook，根据你的清单中有哪些系统，你可能会看到类似以下的输出。在这种情况下，我的`app01.example.com`服务器基于 CentOS 6，因此已应用了补丁。所有其他系统都被跳过，因为它们不符合我的逻辑表达式：

```
$ ansible-playbook -i hosts condition2.yml

PLAY [Play to patch only CentOS systems] ***************************************

TASK [Gathering Facts] *********************************************************
ok: [frt01.example.com]
ok: [app02.example.com]
ok: [app01.example.com]
ok: [frt02.example.com]

TASK [Patch CentOS systems] ****************************************************
changed: [app01.example.com]
skipping: [frt01.example.com]
skipping: [frt02.example.com]
skipping: [app02.example.com]

PLAY RECAP *********************************************************************
app01.example.com : ok=2 changed=1 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
app02.example.com : ok=1 changed=0 unreachable=0 failed=0 skipped=1 rescued=0 ignored=0
frt01.example.com : ok=1 changed=0 unreachable=0 failed=0 skipped=1 rescued=0 ignored=0
frt02.example.com : ok=1 changed=0 unreachable=0 failed=0 skipped=1 rescued=0 ignored=0
```

当你运行任何 Ansible 模块（无论是`shell`、`command`、`yum`、`copy`还是其他模块），模块都会返回详细的运行结果数据。你可以使用`register`关键字将其捕获到一个标准的 Ansible 变量中，然后在 playbook 中稍后进一步处理它。

考虑以下 playbook 代码。它包含两个任务，第一个任务是获取当前目录的列表，并将`shell`模块的输出捕获到一个名为`shellresult`的变量中。然后打印一个简单的`debug`消息，但只有在`shell`命令的输出中包含`hosts`字符串时才会打印：

```
---
- name: Play to patch only CentOS systems
  hosts: localhost
  become: true

  tasks:
    - name: Gather directory listing from local system
      shell: "ls -l"
      register: shellresult

    - name: Alert if we find a hosts file
      debug:
        msg: "Found hosts file!"
      when: '"hosts" in shellresult.stdout'
```

现在，当我们在当前目录中运行这个命令时，如果你是从本书附带的 GitHub 仓库中工作，那么目录中将包含一个名为`hosts`的文件，那么你应该会看到类似以下的输出：

```
$ ansible-playbook condition3.yml
[WARNING]: provided hosts list is empty, only localhost is available. Note that
the implicit localhost does not match 'all'

PLAY [Play to patch only CentOS systems] ***************************************

TASK [Gathering Facts] *********************************************************
ok: [localhost]

TASK [Gather directory listing from local system] ******************************
changed: [localhost]

TASK [Alert if we find a hosts file] *******************************************
ok: [localhost] => {
 "msg": "Found hosts file!"
}

PLAY RECAP *********************************************************************
localhost : ok=3 changed=1 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
```

然而，如果文件不存在，那么你会发现`debug`消息被跳过了。

```
$ ansible-playbook condition3.yml
[WARNING]: provided hosts list is empty, only localhost is available. Note that
the implicit localhost does not match 'all'

PLAY [Play to patch only CentOS systems] ***************************************

TASK [Gathering Facts] *********************************************************
ok: [localhost]

TASK [Gather directory listing from local system] ******************************
changed: [localhost]

TASK [Alert if we find a hosts file] *******************************************
skipping: [localhost]

PLAY RECAP *********************************************************************
localhost : ok=2 changed=1 unreachable=0 failed=0 skipped=1 rescued=0 ignored=0
```

你也可以为生产中的 IT 运维任务创建复杂的条件；但是，请记住，在 Ansible 中，默认情况下变量不会被转换为任何特定的类型，因此即使变量（或事实）的内容看起来像一个数字，Ansible 默认也会将其视为字符串。如果你需要执行整数比较，你必须首先将变量转换为整数类型。例如，这是一个 playbook 的片段，它只在 Fedora 25 及更新版本上运行一个任务：

```
tasks:
  - name: Only perform this task on Fedora 25 and later
 shell: echo "only on Fedora 25 and later"
    when: ansible_facts['distribution'] == "Fedora" and ansible_facts['distribution_major_version']|int >= 25 
```

你可以应用许多不同类型的条件到你的 Ansible 任务中，这一节只是触及了表面；然而，它应该为你扩展你在 Ansible 任务中应用条件的知识提供了一个坚实的基础。你不仅可以将条件逻辑应用到 Ansible 任务中，还可以在一组数据上运行它们，并且我们将在下一节中探讨这一点。

# 使用循环重复任务

通常，我们希望执行一个单一的任务，但使用该单一任务来迭代一组数据。例如，你可能不想创建一个用户帐户，而是创建 10 个。或者你可能想要将 15 个软件包安装到系统中。可能性是无穷无尽的，但要点仍然是一样的——你不想编写 10 个单独的 Ansible 任务来创建 10 个用户帐户。幸运的是，Ansible 支持对数据集进行循环，以确保你可以使用紧密定义的代码执行大规模操作。在本节中，我们将探讨如何在你的 Ansible playbook 中实际使用循环。

和以往一样，我们必须从清单开始工作，并且我们将使用我们在本章中一直使用的熟悉清单：

```
[frontends]
frt01.example.com https_port=8443
frt02.example.com http_proxy=proxy.example.com

[frontends:vars]
ntp_server=ntp.frt.example.com
proxy=proxy.frt.example.com

[apps]
app01.example.com
app02.example.com

[webapp:children]
frontends
apps

[webapp:vars]
proxy_server=proxy.webapp.example.com
health_check_retry=3
health_check_interal=60
```

让我们从一个非常简单的 playbook 开始，向你展示如何在单个任务中循环一组数据。虽然这是一个相当牵强的例子，但它旨在简单地向你展示循环在 Ansible 中的工作原理。我们将定义一个单个任务，在清单中的单个主机上运行`command`模块，并使用`command`模块在远程系统上依次`echo`数字 1 到 6（可以很容易地扩展到添加用户帐户或创建一系列文件）。

考虑以下代码：

```
---
- name: Simple loop demo play
  hosts: frt01.example.com

  tasks:
    - name: Echo a value from the loop
      command: echo "{{ item }}"
      loop:
        - 1
        - 2
        - 3
        - 4
        - 5
        - 6
```

`loop:`语句定义了循环的开始，循环中的项目被定义为一个 YAML 列表。此外，请注意更高的缩进级别，这告诉解析器它们是循环的一部分。在处理循环数据时，我们使用一个名为`item`的特殊变量，其中包含要回显的循环迭代的当前值。因此，如果我们运行这个 playbook，我们应该看到类似以下的输出：

```
$ ansible-playbook -i hosts loop1.yml

PLAY [Simple loop demo play] ***************************************************

TASK [Gathering Facts] *********************************************************
ok: [frt01.example.com]

TASK [Echo a value from the loop] **********************************************
changed: [frt01.example.com] => (item=1)
changed: [frt01.example.com] => (item=2)
changed: [frt01.example.com] => (item=3)
changed: [frt01.example.com] => (item=4)
changed: [frt01.example.com] => (item=5)
changed: [frt01.example.com] => (item=6)

PLAY RECAP *********************************************************************
frt01.example.com : ok=2 changed=1 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
```

你可以将我们在前一节中讨论的条件逻辑与循环结合起来，使循环仅对其数据的子集进行操作。例如，考虑以下 playbook 的迭代：

```

---
- name: Simple loop demo play
  hosts: frt01.example.com

  tasks:
    - name: Echo a value from the loop
      command: echo "{{ item }}"
      loop:
        - 1
        - 2
        - 3
        - 4
        - 5
        - 6
      when: item|int > 3
```

现在，当我们运行这个时，我们会看到任务被跳过，直到我们达到循环内容中的整数值 4 及以上：

```
$ ansible-playbook -i hosts loop2.yml

PLAY [Simple loop demo play] ***************************************************

TASK [Gathering Facts] *********************************************************
ok: [frt01.example.com]

TASK [Echo a value from the loop] **********************************************
skipping: [frt01.example.com] => (item=1)
skipping: [frt01.example.com] => (item=2)
skipping: [frt01.example.com] => (item=3)
changed: [frt01.example.com] => (item=4)
changed: [frt01.example.com] => (item=5)
changed: [frt01.example.com] => (item=6)

PLAY RECAP *********************************************************************
frt01.example.com : ok=2 changed=1 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
```

当然，你可以将这个与基于 Ansible 事实和其他变量的条件逻辑相结合，就像我们之前讨论过的那样。就像我们以前使用`register`关键字捕获模块执行的结果一样，我们也可以使用循环来做到这一点。唯一的区别是，结果现在将存储在一个字典中，每次循环迭代都会有一个字典条目，而不仅仅是一组结果。

因此，让我们看看如果我们进一步增强 playbook 会发生什么：

```
---
- name: Simple loop demo play
  hosts: frt01.example.com

  tasks:
    - name: Echo a value from the loop
      command: echo "{{ item }}"
      loop:
        - 1
        - 2
        - 3
        - 4
        - 5
        - 6
      when: item|int > 3
      register: loopresult

    - name: Print the results from the loop
      debug:
        var: loopresult
```

现在，当我们运行 playbook 时，你将看到包含`loopresult`内容的字典的输出页面。由于空间限制，以下输出被截断，但演示了运行此 playbook 时你应该期望的结果类型：

```
$ ansible-playbook -i hosts loop3.yml

PLAY [Simple loop demo play] ***************************************************

TASK [Gathering Facts] *********************************************************
ok: [frt01.example.com]

TASK [Echo a value from the loop] **********************************************
skipping: [frt01.example.com] => (item=1)
skipping: [frt01.example.com] => (item=2)
skipping: [frt01.example.com] => (item=3)
changed: [frt01.example.com] => (item=4)
changed: [frt01.example.com] => (item=5)
changed: [frt01.example.com] => (item=6)

TASK [Print the results from the loop] *****************************************
ok: [frt01.example.com] => {
 "loopresult": {
 "changed": true,
 "msg": "All items completed",
 "results": [
 {
 "ansible_loop_var": "item",
 "changed": false,
 "item": 1,
 "skip_reason": "Conditional result was False",
 "skipped": true
 },
 {
 "ansible_loop_var": "item",
 "changed": false,
 "item": 2,
 "skip_reason": "Conditional result was False",
 "skipped": true
 },
```

正如你所看到的，输出的结果部分是一个字典，我们可以清楚地看到列表中的前两个项目被`skipped`，因为我们`when`子句的结果(`Conditional`)是`false`。

因此，到目前为止，我们可以看到循环很容易定义和使用，但你可能会问，*你能创建嵌套循环吗？*这个问题的答案是*可以*，但有一个问题——特殊变量`item`会发生冲突，因为内部循环和外部循环都会使用相同的变量名。这意味着你嵌套循环运行的结果将是意想不到的。

幸运的是，有一个名为`loop_control`的`loop`参数，允许您更改包含当前`loop`迭代数据的特殊变量的名称，从`item`更改为您选择的内容。让我们创建一个嵌套循环来看看它是如何工作的。

首先，我们将以通常的方式创建一个 playbook，其中包含一个要在循环中运行的单个任务。为了生成我们的嵌套循环，我们将使用`include_tasks`目录来动态包含另一个 YAML 文件中的单个任务，该文件还将包含一个循环。由于我们打算在嵌套循环中使用此 playbook，因此我们将使用`loop_var`指令将特殊循环内容变量的名称从`item`更改为`second_item`：

```
---
- name: Play to demonstrate nested loops
  hosts: localhost

  tasks:
    - name: Outer loop
      include_tasks: loopsubtask.yml
      loop:
        - a
        - b
        - c
      loop_control:
        loop_var: second_item
```

然后，我们将创建一个名为`loopsubtask.yml`的第二个文件，其中包含内部循环，并包含在前面的 playbook 中。由于我们已经在外部循环中更改了循环项变量名称，因此在这里不需要再次更改它。请注意，此文件的结构非常类似于角色中的任务文件-它不是一个完整的 playbook，而只是一个任务列表：

```
---
- name: Inner loop
  debug:
    msg: "second item={{ second_item }} first item={{ item }}"
  loop:
    - 100
    - 200
    - 300
```

现在，您应该能够运行 playbook，并且您将看到 Ansible 首先迭代外部循环，然后处理由外部循环定义的数据的内部循环。由于循环变量名称不冲突，一切都按我们的预期工作：

```
$ ansible-playbook loopmain.yml
[WARNING]: provided hosts list is empty, only localhost is available. Note that
the implicit localhost does not match 'all'

PLAY [Play to demonstrate nested loops] ****************************************

TASK [Gathering Facts] *********************************************************
ok: [localhost]

TASK [Outer loop] **************************************************************
included: /root/Practical-Ansible-2/Chapter 4/loopsubtask.yml for localhost
included: /root/Practical-Ansible-2/Chapter 4/loopsubtask.yml for localhost
included: /root/Practical-Ansible-2/Chapter 4/loopsubtask.yml for localhost

TASK [Inner loop] **************************************************************
ok: [localhost] => (item=100) => {
 "msg": "second item=a first item=100"
}
ok: [localhost] => (item=200) => {
 "msg": "second item=a first item=200"
}
ok: [localhost] => (item=300) => {
 "msg": "second item=a first item=300"
}

TASK [Inner loop] **************************************************************
ok: [localhost] => (item=100) => {
 "msg": "second item=b first item=100"
}
ok: [localhost] => (item=200) => {
 "msg": "second item=b first item=200"
}
ok: [localhost] => (item=300) => {
 "msg": "second item=b first item=300"
}

TASK [Inner loop] **************************************************************
ok: [localhost] => (item=100) => {
 "msg": "second item=c first item=100"
}
ok: [localhost] => (item=200) => {
 "msg": "second item=c first item=200"
}
ok: [localhost] => (item=300) => {
 "msg": "second item=c first item=300"
}

PLAY RECAP *********************************************************************
localhost : ok=7 changed=0 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
```

循环很容易使用，但非常强大，因为它们允许您轻松地使用一个任务来迭代大量数据。在下一节中，我们将看一下 Ansible 语言的另一个构造，用于控制 playbook 流程-块。

# 使用块分组任务

在 Ansible 中，块允许您逻辑地将一组任务组合在一起，主要用于两个目的之一。一个可能是对整组任务应用条件逻辑；在这个例子中，您可以将相同的 when 子句应用于每个任务，但这很麻烦和低效-最好将所有任务放在一个块中，并将条件逻辑应用于块本身。这样，逻辑只需要声明一次。块在处理错误和特别是从错误条件中恢复时也非常有价值。在本章中，我们将通过简单的实际示例来探讨这两个问题，以帮助您快速掌握 Ansible 中的块。

一如既往，让我们确保我们有一个清单可以使用：

```
[frontends]
frt01.example.com https_port=8443
frt02.example.com http_proxy=proxy.example.com

[frontends:vars]
ntp_server=ntp.frt.example.com
proxy=proxy.frt.example.com

[apps]
app01.example.com
app02.example.com

[webapp:children]
frontends
apps

[webapp:vars]
proxy_server=proxy.webapp.example.com
health_check_retry=3
health_check_interal=60
```

现在，让我们直接看一个如何使用块来对一组任务应用条件逻辑的示例。在高层次上，假设我们想在所有 Fedora Linux 主机上执行以下操作：

+   为 Apache web 服务器安装软件包。

+   安装一个模板化的配置。

+   启动适当的服务。

我们可以通过三个单独的任务来实现这一点，所有这些任务都与一个`when`子句相关联，但是块为我们提供了更好的方法。以下示例 playbook 显示了包含在块中的三个任务（请注意需要额外的缩进级别来表示它们在块中的存在）：

```
---
- name: Conditional block play
  hosts: all
  become: true

  tasks:
  - name: Install and configure Apache
    block:
      - name: Install the Apache package
        dnf:
          name: httpd
          state: installed
      - name: Install the templated configuration to a dummy location
        template:
          src: templates/src.j2
          dest: /tmp/my.conf
      - name: Start the httpd service
        service:
          name: httpd
          state: started
          enabled: True
    when: ansible_facts['distribution'] == 'Fedora'
```

当您运行此 playbook 时，您应该发现与您可能在清单中拥有的任何 Fedora 主机上只运行与 Apache 相关的任务；您应该看到三个任务中的所有任务都被运行或跳过-取决于清单的组成和内容，它可能看起来像这样：

```
$ ansible-playbook -i hosts blocks.yml

PLAY [Conditional block play] **************************************************

TASK [Gathering Facts] *********************************************************
ok: [app02.example.com]
ok: [frt01.example.com]
ok: [app01.example.com]
ok: [frt02.example.com]

TASK [Install the Apache package] **********************************************
changed: [frt01.example.com]
changed: [frt02.example.com]
skipping: [app01.example.com]
skipping: [app02.example.com]

TASK [Install the templated configuration to a dummy location] *****************
changed: [frt01.example.com]
changed: [frt02.example.com]
skipping: [app01.example.com]
skipping: [app02.example.com]

TASK [Start the httpd service] *************************************************
changed: [frt01.example.com]
changed: [frt02.example.com]
skipping: [app01.example.com]
skipping: [app02.example.com]

PLAY RECAP *********************************************************************
app01.example.com : ok=1 changed=0 unreachable=0 failed=0 skipped=3 rescued=0 ignored=0
app02.example.com : ok=1 changed=0 unreachable=0 failed=0 skipped=3 rescued=0 ignored=0
frt01.example.com : ok=4 changed=3 unreachable=0 failed=0 skipped=3 rescued=0 ignored=0
frt02.example.com : ok=4 changed=3 unreachable=0 failed=0 skipped=3 rescued=0 ignored=0
```

这很容易构建，但在控制大量任务流程方面非常强大。

这一次，让我们构建一个不同的示例，以演示如何利用块来帮助 Ansible 优雅地处理错误条件。到目前为止，您应该已经经历过，如果您的 playbook 遇到任何错误，它们可能会在失败点停止执行。在某些情况下，这远非理想，您可能希望在此事件中执行某种恢复操作，而不仅仅是停止 playbook。

让我们创建一个新的 playbook，这次内容如下：

```
---
- name: Play to demonstrate block error handling
  hosts: frontends

  tasks:
    - name: block to handle errors
      block:
        - name: Perform a successful task
          debug:
            msg: 'Normally executing....'
        - name: Deliberately create an error
          command: /bin/whatever
        - name: This task should not run if the previous one results in an error
          debug:
            msg: 'Never print this message if the above command fails!!!!'
      rescue:
        - name: Catch the error (and perform recovery actions)
          debug:
            msg: 'Caught the error'
        - name: Deliberately create another error
          command: /bin/whatever
        - name: This task should not run if the previous one results in an error
          debug:
            msg: 'Do not print this message if the above command fails!!!!'
      always:
        - name: This task always runs!
          debug:
            msg: "Tasks in this part of the play will be ALWAYS executed!!!!"
```

请注意，在前面的 play 中，我们现在有了额外的`block`部分——除了`block`本身中的任务外，我们还有两个标记为`rescue`和`always`的新部分。执行流程如下：

1.  `block`部分中的所有任务都按照其列出的顺序正常执行。

1.  如果`block`中的任务导致错误，则不会运行`block`中的其他任务：

+   `rescue`部分中的任务按其列出的顺序开始运行。

+   如果`block`任务没有导致错误，则`rescue`部分中的任务不会运行。

1.  如果在`rescue`部分运行的任务导致错误，则不会执行进一步的`rescue`任务，执行将移至`always`部分。

1.  `always`部分中的任务始终运行，无论`block`或`rescue`部分是否出现错误。即使没有遇到错误，它们也会运行。

考虑到这种执行流程，当您执行此 playbook 时，您应该会看到类似以下的输出，注意我们故意创建了两个错误条件来演示流程：

```
$ ansible-playbook -i hosts blocks-error.yml

PLAY [Play to demonstrate block error handling] ********************************

TASK [Gathering Facts] *********************************************************
ok: [frt02.example.com]
ok: [frt01.example.com]

TASK [Perform a successful task] ***********************************************
ok: [frt01.example.com] => {
    "msg": "Normally executing...."
}
ok: [frt02.example.com] => {
    "msg": "Normally executing...."
}

TASK [Deliberately create an error] ********************************************
fatal: [frt01.example.com]: FAILED! => {"changed": false, "cmd": "/bin/whatever", "msg": "[Errno 2] No such file or directory", "rc": 2}
fatal: [frt02.example.com]: FAILED! => {"changed": false, "cmd": "/bin/whatever", "msg": "[Errno 2] No such file or directory", "rc": 2}

TASK [Catch the error (and perform recovery actions)] **************************
ok: [frt01.example.com] => {
    "msg": "Caught the error"
}
ok: [frt02.example.com] => {
    "msg": "Caught the error"
}

TASK [Deliberately create another error] ***************************************
fatal: [frt01.example.com]: FAILED! => {"changed": false, "cmd": "/bin/whatever", "msg": "[Errno 2] No such file or directory", "rc": 2}
fatal: [frt02.example.com]: FAILED! => {"changed": false, "cmd": "/bin/whatever", "msg": "[Errno 2] No such file or directory", "rc": 2}

TASK [This task always runs!] **************************************************
ok: [frt01.example.com] => {
    "msg": "Tasks in this part of the play will be ALWAYS executed!!!!"
}
ok: [frt02.example.com] => {
    "msg": "Tasks in this part of the play will be ALWAYS executed!!!!"
}

PLAY RECAP *********************************************************************
frt01.example.com : ok=4 changed=0 unreachable=0 failed=1 skipped=0 rescued=1 ignored=0
frt02.example.com : ok=4 changed=0 unreachable=0 failed=1 skipped=0 rescued=1 ignored=0
```

Ansible 有两个特殊变量，其中包含您可能在`rescue`块中找到有用的信息以执行恢复操作：

+   `ansible_failed_task`：这是一个包含来自`block`失败的任务的详细信息的字典，导致我们进入`rescue`部分。您可以通过使用`debug`显示其内容来探索这一点，但例如，失败任务的名称可以从`ansible_failed_task.name`中获取。

+   `ansible_failed_result`：这是失败任务的结果，并且与如果您在失败的任务中添加了`register`关键字的行为相同。这样可以避免在每个任务中添加`register`以防它失败。

随着您的 playbooks 变得更加复杂，错误处理变得越来越重要（或者条件逻辑变得更加重要），`block`将成为编写良好、健壮的 playbooks 的重要组成部分。让我们在下一节中继续探讨执行策略，以进一步控制您的 playbook 运行。

# 通过策略配置 play 执行

随着您的 playbooks 变得越来越复杂，调试任何可能出现的问题变得越来越重要。例如，您是否可以在执行过程中检查给定变量（或变量）的内容，而无需在整个 playbook 中插入`debug`语句？同样，我们迄今为止已经看到，Ansible 将确保特定任务在应用于所有清单主机之前完成，然后再移动到下一个任务——是否有办法改变这一点？

当您开始使用 Ansible 时，默认情况下看到的执行策略（尽管我们尚未提到它的名称）被称为`linear`。这正是它所描述的——在开始下一个任务之前，每个任务都会在所有适用的主机上依次执行。然而，还有另一种不太常用的策略称为`free`，它允许所有任务在每个主机上尽快完成，而不必等待其他主机。

然而，当您开始使用 Ansible 时，最有用的策略将是`debug`策略，这使得 Ansible 可以在 playbook 中发生错误时直接将您置于集成的调试环境中。让我们通过创建一个有意义的错误的 playbook 来演示这一点。请注意 play 定义中的`strategy: debug`和`debugger: on_failed`语句：

```
---
- name: Play to demonstrate the debug strategy
  hosts: frt01.example.com
  strategy: debug
  debugger: on_failed
  gather_facts: no
  vars:
    username: daniel

  tasks:
    - name: Generate an error by referencing an undefined variable
      ping: data={{ mobile }}
```

现在，如果您执行此 playbook，您应该会看到它开始运行，但是当遇到它包含的故意错误时，它会将您带入集成调试器。输出的开头应该类似于以下内容：

```
$ ansible-playbook -i hosts debug.yml

PLAY [Play to demonstrate the debug strategy] **********************************

TASK [Generate an error by referencing an undefined variable] ******************
fatal: [frt01.example.com]: FAILED! => {"msg": "The task includes an option with an undefined variable. The error was: 'mobile' is undefined\n\nThe error appears to be in '/root/Practical-Ansible-2/Chapter 4/debug.yml': line 11, column 7, but may\nbe elsewhere in the file depending on the exact syntax problem.\n\nThe offending line appears to be:\n\n tasks:\n - name: Generate an error by referencing an undefined variable\n ^ here\n"}
[frt01.example.com] TASK: Generate an error by referencing an undefined variable (debug)>

[frt02.prod.com] TASK: make an error with refering incorrect variable (debug)> p task_vars
{'ansible_check_mode': False,
 'ansible_current_hosts': [u'frt02.prod.com'],
 'ansible_diff_mode': False,
 'ansible_facts': {},
 'ansible_failed_hosts': [],
 'ansible_forks': 5,
...
[frt02.prod.com] TASK: make an error with refering incorrect variable (debug)> quit
User interrupted execution
$ 
```

请注意，剧本开始执行，但在第一个任务上失败，并显示错误，因为变量未定义。但是，它不是退出到 shell，而是进入交互式调试器。本书不涵盖调试器的详尽指南，但如果您有兴趣学习，可以在此处找到更多详细信息：[`docs.ansible.com/ansible/latest/user_guide/playbooks_debugger.html`](https://docs.ansible.com/ansible/latest/user_guide/playbooks_debugger.html)。

然而，为了带您进行一个非常简单的实际调试示例，输入`p task`命令——这将导致 Ansible 调试器打印失败任务的名称；如果您正在进行一个大型剧本，这将非常有用：

```
[frt01.example.com] TASK: Generate an error by referencing an undefined variable (debug)> p task
TASK: Generate an error by referencing an undefined variable
```

现在我们知道了剧本失败的原因，所以让我们通过发出`p task.args`命令来深入了解一下，这将显示传递给任务模块的参数：

```
[frt01.example.com] TASK: Generate an error by referencing an undefined variable (debug)> p task.args
{u'data': u'{{ mobile }}'}
```

因此，我们可以看到我们的模块传递了一个名为`data`的参数，参数值是一个变量（由大括号对表示）称为`mobile`。因此，可能有必要查看任务可用的变量，看看这个变量是否存在，如果存在的话，值是否合理（使用`p task_vars`命令来执行此操作）：

```
[frt01.example.com] TASK: Generate an error by referencing an undefined variable (debug)> p task_vars
{'ansible_check_mode': False,
 'ansible_current_hosts': [u'frt01.example.com'],
 'ansible_dependent_role_names': [],
 'ansible_diff_mode': False,
 'ansible_facts': {},
 'ansible_failed_hosts': [],
 'ansible_forks': 5,
```

上述输出被截断了，您会发现与任务相关的许多变量——这是因为任何收集的事实和内部 Ansible 变量都可用于任务。但是，如果您浏览列表，您将能够确认没有名为`mobile`的变量。

因此，这应该足够的信息来修复您的剧本。输入`q`退出调试器：

```
[frt01.example.com] TASK: Generate an error by referencing an undefined variable (debug)> q
User interrupted execution
$
```

Ansible 调试器是一个非常强大的工具，您应该学会有效地使用它，特别是当您的剧本复杂性增加时。这结束了我们对剧本设计各个方面的实际考察——在下一节中，我们将看看您可以将 Git 源代码管理集成到您的剧本中的方法。

# 使用 ansible-pull

`ansible-pull`命令是 Ansible 的一个特殊功能，允许您一次性从 Git 存储库（例如 GitHub）中拉取一个剧本，然后执行它，因此节省了克隆（或更新工作副本）存储库，然后执行剧本等常规步骤。`ansible-pull`的好处在于它允许您集中存储和版本控制您的剧本，然后使用单个命令执行它们，从而使它们能够使用`cron`调度程序执行，而无需甚至在给定的主机上安装 Ansible 剧本。

然而，需要注意的一点是，虽然`ansible`和`ansible-playbook`命令都可以在整个清单上运行剧本，并针对一个或多个远程主机运行剧本，但`ansible-pull`命令只打算在本地主机上运行从您的源代码控制系统获取的剧本。因此，如果您想在整个基础架构中使用`ansible-pull`，您必须将其安装到每个需要它的主机上。

尽管如此，让我们看看这可能是如何工作的。我们将简单地手动运行命令来探索它的应用，但实际上，您几乎肯定会将其安装到您的`crontab`中，以便定期运行，捕捉您对剧本所做的任何更改版本控制系统中。

由于`ansible-pull`只打算在本地系统上运行剧本，因此清单文件有些多余——相反，我们将使用一个很少使用的清单规范，您可以在命令行上简单地指定清单主机目录为逗号分隔的列表。如果您只有一个主机，只需指定其名称，然后加上逗号。

让我们使用 GitHub 上的一个简单的剧本，根据变量内容设置每日消息。为此，我们将运行以下命令（我们将在一分钟内分解）：

```
$ ansible-pull -d /var/ansible-set-motd -i ${HOSTNAME}, -U https://github.com/jamesfreeman959/ansible-set-motd.git site.yml -e "ag_motd_content='MOTD generated by ansible-pull'" >> /tmp/ansible-pull.log 2>&1
```

这个命令分解如下：

+   `-d /var/ansible-set-motd`：这将设置包含来自 GitHub 的代码检出的工作目录。

+   `-i ${HOSTNAME},`：这仅在当前主机上运行，由适当的 shell 变量指定其主机名。

+   `-U https://github.com/jamesfreeman959/ansible-set-motd.git`：我们使用此 URL 来获取 playbooks。

+   `site.yml`：这是要运行的 playbook 的名称。

+   `-e "ag_motd_content='MOTD generated by ansible-pull'"`：这将设置适当的 Ansible 变量以生成 MOTD 内容。

+   `>> /tmp/ansible-pull.log 2>&1`：这将重定向命令的输出到日志文件，以便以后分析 - 特别是在`cron job`中运行命令时，输出将永远不会打印到用户的终端上，这是非常有用的。

当您运行此命令时，您应该会看到类似以下的输出（请注意，为了更容易看到输出，已删除了日志重定向）：

```
$ ansible-pull -d /var/ansible-set-motd -i ${HOSTNAME}, -U https://github.com/jamesfreeman959/ansible-set-motd.git site.yml -e "ag_motd_content='MOTD generated by ansible-pull'"
Starting Ansible Pull at 2020-04-14 17:26:21
/usr/bin/ansible-pull -d /var/ansible-set-motd -i cookbook, -U https://github.com/jamesfreeman959/ansible-set-motd.git site.yml -e ag_motd_content='MOTD generated by ansible-pull'
cookbook |[WARNING]: SUCCESS = Your git > {
    "aversion isfter": "7d too old t3a191ecb2do fully suebe7f84f4fpport the a5817b0f1bdepth argu49c4cd54",ment.
Fall
    "ansing back tible_factso full che": {
     ckouts.
   "discovered_interpreter_python": "/usr/bin/python"
    },
    "before": "7d3a191ecb2debe7f84f4fa5817b0f1b49c4cd54",
    "changed": false,
    "remote_url_changed": false
}

PLAY [Update the MOTD on hosts] ************************************************

TASK [Gathering Facts] *********************************************************
ok: [cookbook]

TASK [ansible.motd : Add 99-footer file] ***************************************
skipping: [cookbook]

TASK [ansible.motd : Delete 99-footer file] ************************************
ok: [cookbook]

TASK [ansible.motd : Delete /etc/motd file] ************************************
skipping: [cookbook]

TASK [ansible.motd : Check motd tail supported] ********************************
fatal: [cookbook]: FAILED! => {"changed": true, "cmd": "test -f /etc/update-motd.d/99-footer", "delta": "0:00:00.004444", "end": "2020-04-14 17:26:25.489793", "msg": "non-zero return code", "rc": 1, "start": "2020-04-14 17:26:25.485349", "stderr": "", "stderr_lines": [], "stdout": "", "stdout_lines": []}
...ignoring

TASK [ansible.motd : Add motd tail] ********************************************
skipping: [cookbook]

TASK [ansible.motd : Add motd] *************************************************
changed: [cookbook]

PLAY RECAP *********************************************************************
cookbook : ok=4 changed=2 unreachable=0 failed=0 skipped=3 rescued=0 ignored=1
```

这个命令可以成为您整体 Ansible 解决方案的一个非常强大的部分，特别是因为这意味着您不必过于担心集中运行所有 playbooks，或者确保每次运行它们时它们都是最新的。在大型基础架构中，将其安排在`cron`中的能力尤其强大，理想情况下，自动化应该能够自行处理事务。

这结束了我们对 playbooks 的实际观察，以及如何编写自己的代码 - 通过对 Ansible 模块进行一些研究，现在您应该足够轻松地编写自己的强大 playbooks 了。

# 总结

Playbooks 是 Ansible 自动化的生命线，提供了一个强大的框架，用于定义任务的逻辑集合并清晰而强大地处理错误条件。将角色添加到这个混合中对于组织代码和支持代码重用都是有价值的，尤其是在您的自动化需求增长时。Ansible playbooks 为您的技术需求提供了一个真正完整的自动化解决方案。

在本章中，您学习了 playbook 框架以及如何开始编写自己的 playbooks。然后，您学习了如何将代码组织成角色，并设计代码以有效地支持重用。然后，我们探讨了一些更高级的 playbook 编写主题，如使用条件逻辑、块和循环。最后，我们看了一下 playbook 执行策略，特别是为了能够有效地调试您的 playbooks，最后，我们看了一下如何直接从 GitHub 在本地机器上运行 Ansible playbooks。

在下一章中，我们将学习如何使用和创建我们自己的模块，为您提供扩展 Ansible 功能的技能，以适应自己定制的环境，并为社区做出贡献。

# 问题

1.  如何通过临时命令在`frontends`主机组中重新启动 Apache Web 服务器？

A) `ansible frontends -i hosts -a "name=httpd state=restarted"`

B) `ansible frontends -i hosts -b service -a "name=httpd state=restarted"`

C) `ansible frontends -i hosts -b -m service -a "name=httpd state=restarted"`

D) `ansible frontends -i hosts -b -m server -a "name=httpd state=restarted"`

E) `ansible frontends -i hosts -m restart -a "name=httpd"`

1.  Do 块允许您逻辑地组合一组任务，或执行错误处理吗？

A) 正确

B) 错误

1.  默认策略是通过 playbook 中的相关模块进行设置。

A) 正确

B) 错误

# 进一步阅读

`ansible-galaxy`和文档可以在这里找到：[`galaxy.ansible.com/docs/`](https://galaxy.ansible.com/docs/)。
