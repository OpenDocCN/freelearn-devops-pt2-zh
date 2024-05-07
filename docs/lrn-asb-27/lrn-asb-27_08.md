# 从 Ansible 获取通知

与 bash 脚本相比，Ansible 的一个重大优势之一是其幂等性，确保一切井然有序。这是一个非常好的功能，不仅向您保证服务器配置没有变化，而且新配置也将在短时间内生效。

因为这些原因，许多人每天运行他们的 `master.yaml` 文件一次。当您这样做时（也许您应该！），您希望 Ansible 本身向您发送某种反馈。还有许多其他情况，您可能希望 Ansible 向您或您的团队发送消息。例如，如果您使用 Ansible 部署您的应用程序，您可能希望向开发团队频道发送 IRC 消息（或其他类型的群聊消息），以便他们都了解系统的状态。

有时，你希望 Ansible 通知 Nagios 即将破坏某些东西，这样 Nagios 就不会担心，也不会开始向系统管理员发送电子邮件和消息。在本章中，我们将探讨多种方法，帮助您设置 Ansible Playbooks，既可以与您的监控系统配合工作，又可以最终发送通知。

在本章中，我们将探讨以下主题：

+   电子邮件通知

+   Ansible XMPP/Jabber

+   Slack 和 Rocket 聊天

+   向 IRC 频道发送消息（社区信息和贡献）

+   Amazon 简单通知服务

+   Nagios

# 技术要求

许多示例将需要第三方系统（用于发送消息），您可能正在使用或没有使用。如果您无法访问其中一个系统，则相关示例将无法执行。这并不是一个大问题，因为您仍然可以阅读该部分，并且许多通知模块非常相似。您可能会发现，适用于您环境的模块与另一个模块的功能非常相似。您可以参考 Ansible 通知模块的完整列表：[`docs.ansible.com/ansible/latest/modules/list_of_notification_modules.html`](https://docs.ansible.com/ansible/latest/modules/list_of_notification_modules.html)。

您可以从本书的 GitHub 存储库下载所有文件：[`github.com/PacktPublishing/Learning-Ansible-2.X-Third-Edition/tree/master/Chapter06`](https://github.com/PacktPublishing/Learning-Ansible-2.X-Third-Edition/tree/master/Chapter06)。

# 使用 Ansible 发送电子邮件

经常有用户需要及时通知有关 Ansible Playbook 执行的操作。这可能是因为这个用户知道这一点很重要，或者因为有一个自动化系统必须通知以便（不）启动某个过程。

提醒人们的最简单和最常见的方法是发送电子邮件。Ansible 允许您使用`mail`模块从您的 playbook 发送电子邮件。您可以在任何任务之间使用此模块，并在需要时通知用户。此外，在某些情况下，您无法自动化每一件事，因为要么您缺乏权限，要么需要进行一些手动检查和确认。如果是这种情况，您可以通知负责人员 Ansible 已经完成了其工作，现在是他们执行其职责的时候了。让我们使用名为`uptime_and_email.yaml`的非常简单的 playbook 来使用`mail`模块通知您的用户：

```
---
- hosts: localhost 
  connection: local
  tasks: 
    - name: Read the machine uptime 
      command: uptime -p 
      register: uptime 
    - name: Send the uptime via e-mail 
      mail: 
        host: mail.fale.io 
        username: ansible@fale.io 
        password: PASSWORD 
        to: me@fale.io 
        subject: Ansible-report 
        body: 'Local system uptime is {{ uptime.stdout }}.' 

```

前述 playbook 首先读取当前机器的正常运行时间，发出`uptime`命令，然后通过电子邮件发送到`me@fale.io`电子邮件地址。要发送电子邮件，我们显然需要一些额外信息，如 SMTP 主机、有效的 SMTP 凭据以及电子邮件的内容。这个例子非常简单，将使我们能够保持示例简短，但显然，你可以以非常类似的方式在非常长且复杂的 playbooks 中生成电子邮件。如果我们稍微专注于`mail`任务，我们可以看到我们正在使用以下数据：

+   要用于发送电子邮件的电子邮件服务器（还需登录信息，这是该服务器所必需的）

+   接收者电子邮件地址

+   电子邮件主题

+   电子邮件正文

`mail`模块支持的其他有趣参数如下：

+   `attach`参数：这用于向将生成的电子邮件添加附件。例如，当您希望通过电子邮件发送日志时，这非常有用。

+   `port`参数：这用于指定电子邮件服务器使用的端口。

有关此模块的有趣之处在于，唯一强制性字段是`subject`，而不是正文，许多人可能期望正文也是必需的。RFC 2822 不强制要求主题或正文的存在，因此即使没有它们，电子邮件仍然有效，但对于人来说，管理这种格式的电子邮件将非常困难。因此，Ansible 将始终发送带有主题和正文的电子邮件，如果正文为空，则会在主题和正文中都使用`subject`字符串。

我们现在可以继续执行脚本以验证其功能，使用以下命令：

```
    ansible-playbook -i localhost, uptime_and_email.yaml 
```

由于`uptime`的`-p`参数是特定于 Linux 的，可能无法在其他 POSIX 操作系统（如 macOS）上运行，因此此 playbook 可能在某些机器上无法正常工作。

通过运行上述 playbook，我们将得到类似以下的结果：

```
    PLAY [localhost] *************************************************

    TASK [setup] *****************************************************
    ok: [localhost]

    TASK [Read the machine uptime] ***********************************
    changed: [localhost]

    TASK [Send the uptime via email] ********************************
    changed: [localhost]

    PLAY RECAP *******************************************************
    localhost         : ok=3    changed=2    unreachable=0    failed=0

```

正如预期的那样，Ansible 已经向我发送了一封带有以下内容的电子邮件：

```
Local system uptime is up 38 min
```

该模块可以以许多不同的方式使用。我见过一个实际案例，那就是为了自动化一个非常复杂但顺序流程的一部分，涉及多个人员。每个人在流程的特定点必须开始他们的工作，而链中的下一个人在前一个人完成他们的工作之前不能开始他们的工作。保持该过程运作的关键在于每个人手动向链中的下一个人发送电子邮件，通知他们他们自己的部分已经完成，因此接收者需要开始他们在流程中的工作。在我们开始自动化该程序之前，人们通常手动进行电子邮件通知，但当我们开始自动化该程序的一部分时，没有人注意到该部分已经自动化。

对于这样的复杂、顺序流程，通过电子邮件进行跟踪并不是处理它们的最好方式，因为错误很容易犯，可能导致跟踪丢失。此外，此类复杂顺序流程往往非常缓慢，但它们在组织中被广泛使用，通常您无法进行更改。

有些情况下，流程需要以比电子邮件更实时的方式发送通知，因此 XMPP 是一个好的选择。

# XMPP

电子邮件速度慢，不可靠，并且人们通常不会立即对其做出反应。在某些情况下，您希望向您的用户发送实时消息。许多组织依赖 XMPP/Jabber 作为其内部聊天系统，而美妙的事情是 Ansible 能够直接向 XMPP/Jabber 用户和会议室发送消息。

让我们修改之前的示例，在 `uptime_and_xmpp_user.yaml` 文件中发送可靠性信息给某个用户：

```
---
- hosts: localhost 
  connection: local
  tasks: 
    - name: Read the machine uptime 
      command: 'uptime -p' 
      register: uptime 
    - name: Send the uptime to user 
      jabber: 
        user: ansible@fale.io 
        password: PASSWORD 
        to: me@fale.io 
        msg: 'Local system uptime is {{ uptime.stdout }}.' 
```

如果您想要使用 Ansible 的 `jabber` 任务，需要在执行该任务的系统上安装 `xmpppy` 库。其中一种安装方法是使用您的软件包管理器。例如，在 Fedora 上，您只需执行 `sudo dnf install -y python2-xmpp` 即可进行安装。您也可以使用 `pip install xmpppy` 进行安装。

第一个任务与前一节完全相同，而第二个任务则有一些细微的差别。正如您所看到的，`jabber` 模块非常类似于 `mail` 模块，并且需要类似的参数。在 XMPP 的情况下，我们不需要指定服务器主机和端口，因为 XMPP 会从 DNS 自动收集该信息。在需要使用不同服务器主机或端口的情况下，我们可以分别使用 `host` 和 `port` 参数。

现在，我们可以使用以下命令执行该脚本以验证其功能：

```
    ansible-playbook -i localhost, uptime_and_xmpp_user.yaml
```

我们将得到类似于以下的结果：

```
    PLAY [localhost] *************************************************

    TASK [setup] *****************************************************
    ok: [localhost]

    TASK [Read the machine uptime] ***********************************
    changed: [localhost]

    TASK [Send the uptime to user] ***********************************
    changed: [localhost]

    PLAY RECAP *******************************************************
    localhost         : ok=3    changed=2    unreachable=0    failed=0

```

对于我们想要发消息到会议室而不是单个用户的情况，只需要将接收者在 `to` 参数中更改为与会议室相应的接收者即可。

```
to: sysop@conference.fale.io (mailto:sysop@conference.fale.io)/ansiblebot
```

除了接收者更改和添加 `(mailto:sysop@conference.fale.io)/ansiblebot`（标识要使用的聊天句柄 `ansiblebot`，在本例中）之外，XMPP 对用户和会议室的处理方式是相同的，因此从一种切换到另一种非常容易。

尽管 XMPP 相当流行，但并非每家公司都使用它。另一个 Ansible 可以发送消息的协作平台是 Slack。

# Slack

在过去几年中，出现了许多新的聊天和协作平台。其中最常用的之一是 Slack。Slack 是一个基于云的团队协作工具，这使得与 XMPP 相比，与 Ansible 的集成更加容易。

让我们将以下行放入 `uptime_and_slack.yaml` 文件中：

```
---
- hosts: localhost 
  connection: local
  tasks: 
    - name: Read the machine uptime 
      command: 'uptime -p' 
      register: uptime 
    - name: Send the uptime to slack channel 
      slack: 
        token: TOKEN 
        channel: '#ansible' 
        msg: 'Local system uptime is {{ uptime.stdout }}.' 
```

正如我们讨论的那样，此模块的语法甚至比 XMPP 更简单。事实上，它只需要知道令牌（您可以在 Slack 网站上生成），要发送消息的频道以及消息本身。

自 Ansible 的 1.8 版本以来，需要新版本的 Slack 令牌，例如 `G522SJP14/D563DW213/7Qws484asdWD4w12Md3avf4FeD`。

使用以下命令运行 Playbook：

```
    ansible-playbook -i localhost, uptime_and_slack.yaml  
```

这导致以下输出：

```
    PLAY [localhost] *************************************************

    TASK [setup] *****************************************************
    ok: [localhost]

    TASK [Read the machine uptime] ***********************************
    changed: [localhost]

    TASK [Send the uptime to slack channel] **************************
    changed: [localhost]

    PLAY RECAP *******************************************************
    localhost         : ok=3    changed=2    unreachable=0    failed=0

```

由于 Slack 的目标是使通信更有效，它允许我们调整消息的多个方面。从我的角度来看，最有趣的几点是：

+   `color`：这允许您指定一个颜色条，放在消息开头以标识以下状态：

    +   Good: green bar

    +   Normal: no bar

    +   警告：黄色条

    +   Danger: red bar

+   `icon_url`：这允许您为该消息更改用户图像。

例如，以下代码将以警告颜色和自定义用户图像发送消息：

```
    - name: Send the uptime to slack channel 
      slack: 
        token: TOKEN 
        channel: '#ansible' 
        msg: 'Local system uptime is {{ uptime.stdout }}.' 
        color: warning
        icon_url: https://example.com/avatar.png
```

由于并非每家公司都愿意让 Slack 看到他们的私人对话，因此存在替代方案，例如 Rocket Chat。

# Rocket Chat

许多公司喜欢 Slack 的功能，但不想在使用 Slack 时失去本地服务所提供的隐私。**Rocket Chat** 是一个开源软件解决方案，实现了 Slack 的大多数功能以及其大部分界面。作为开源软件，每家公司都可以将其安装在本地，并以符合其 IT 规则的方式进行管理。

由于 Rocket Chat 的目标是成为 Slack 的即插即用替代方案，从我们的角度来看，几乎不需要做任何更改。事实上，我们可以创建 `uptime_and_rocket.yaml` 文件，其中包含以下内容：

```
---
- hosts: localhost 
  connection: local
  tasks: 
    - name: Read the machine uptime 
      command: 'uptime -p' 
      register: uptime 
    - name: Send the uptime to rocketchat channel 
      rocketchat: 
        token: TOKEN 
        domain: chat.example.com 
        channel: '#ansible' 
        msg: 'Local system uptime is {{ uptime.stdout }}.' 
```

如您所见，仅有第六和第七行发生了变化，其中 `slack` 一词已被替换为 `rocketchat`。此外，我们需要添加一个 `domain` 字段，指定我们的 Rocket Chat 安装位于何处。

使用以下命令运行代码：

```
    ansible-playbook -i localhost, uptime_and_rocketchat.yaml  
```

这导致以下输出：

```
    PLAY [localhost] *************************************************

    TASK [setup] *****************************************************
    ok: [localhost]

    TASK [Read the machine uptime] ***********************************
    changed: [localhost]

    TASK [Send the uptime to rocketchat channel] *********************
    changed: [localhost]

    PLAY RECAP *******************************************************
    localhost         : ok=3    changed=2    unreachable=0    failed=0

```

另一种自托管公司对话的方式是使用 IRC，这是一个非常古老但仍然常用的协议。Ansible 也能够使用它发送消息。

# Internet Relay Chat

**互联网中继聊天**（**IRC**）可能是 1990 年代最著名和广泛使用的聊天协议，至今仍在使用。它的受欢迎程度和持续使用主要是由于它在开源社区中的使用和其简单性。从 Ansible 的角度来看，IRC 是一个非常直接的模块，我们可以像以下示例一样使用它（放在 `uptime_and_irc.yaml` 文件中）：

```
---
- hosts: localhost 
  connection: local
  tasks: 
    - name: Read the machine uptime 
      command: 'uptime -p' 
      register: uptime 
    - name: Send the uptime to IRC channel 
      irc: 
        port: 6669 
        server: irc.example.net 
        channel: '#desired_channel'
        msg: 'Local system uptime is {{ uptime.stdout }}.' 
        color: green 
```

您需要安装 `socket` Python 库才能使用 Ansible IRC 模块。

在 IRC 模块中，需要以下字段：

+   `channel`：指定消息将要传送到的频道。

+   `msg`：这是您要发送的消息。

通常您需要指定的其他配置包括：

+   `server`：选择要连接的`服务器`，如果不是`localhost`。

+   `port`：选择连接的`端口`，如果不是`6667`。

+   `color`：指定消息`颜色`，如果不是`黑色`。

+   `nick`：指定发送消息的`昵称`，如果不是`ansible`。

+   `use_ssl`：使用 SSL 和 TLS 安全性。

+   `style`：如果要以粗体、斜体、下划线或反向样式发送消息，则使用此选项。

使用以下命令运行代码：

```
    ansible-playbook uptime_and_irc.yaml  
```

这将产生以下输出：

```
    PLAY [localhost] *************************************************

    TASK [setup] *****************************************************
    ok: [localhost]

    TASK [Read the machine uptime] ***********************************
    changed: [localhost]

    TASK [Send the uptime to IRC channel] ****************************
    changed: [localhost]

    PLAY RECAP *******************************************************
    localhost         : ok=3    changed=2    unreachable=0    failed=0

```

我们已经看到许多不同的通信系统可能在您的公司或项目中使用，但这些通常用于人与人或机器与人的通信。机器对机器的通信通常使用不同的系统，例如亚马逊 SNS。

# 亚马逊简单通知服务

有时，您希望您的 Playbook 在接收警报的方式上是不可知的。这具有多个优点，主要是灵活性。事实上，在这种模型中，Ansible 将消息交付给一个通知服务，然后通知服务将负责交付消息。**亚马逊简单通知服务**（**SNS**）并不是唯一可用的通知服务，但它可能是最常用的。SNS 有以下组件：

+   **消息**：由 UUID 识别的发布者生成的消息

+   **发布者**：生成消息的程序

+   **主题**：命名的消息组，可以类比于聊天频道或房间

+   **订阅者**：订阅了他们感兴趣主题的所有消息的客户端

因此，在我们的情况下，具体如下：

+   **消息**：Ansible 通知

+   **发布者**：Ansible 本身

+   **主题**：根据系统和/或通知类型（例如存储、网络或计算）对消息进行分组的可能不同主题

+   **订阅者**：您团队中必须得到通知的人员

正如我们所说，SNS 的一个重大优势是，您可以将 Ansible 发送消息的方式（即 SNS API）与用户接收消息的方式分离开来。事实上，您将能够为每个用户和每个主题规则选择不同的传送系统，并最终可以动态更改它们，以确保消息以最佳方式发送到任何情况中。目前，SNS 可以发送消息的五种方式如下：

+   Amazon **Lambda** 函数（用 Python、Java 和 JavaScript 编写的无服务器函数）

+   Amazon **Simple Queue Service**（**SQS**）（一种消息队列系统）

+   电子邮件

+   HTTP(S) 调用

+   短信

让我们看看如何使用 Ansible 发送 SNS 消息。为此，我们可以创建一个名为 `uptime_and_sns.yaml` 的文件，其中包含以下内容：

```
---
- hosts: localhost 
  connection: local
  tasks: 
    - name: Read the machine uptime 
      command: 'uptime -p' 
      register: uptime 
    - name: Send the uptime to SNS 
      sns: 
        msg: 'Local system uptime is {{ uptime.stdout }}.' 
        subject: "System uptime" 
        topic: "uptime"
```

在此示例中，我们使用 `msg` 键来设置将要发送的消息，`topic` 选择最合适的主题，并且 `subject` 将用作电子邮件投递的主题。您可以设置许多其他选项。主要用于使用不同的传送方式发送不同的消息。

例如，通过短信发送短消息（最后，**SMS** 中的第一个 **S** 意味着 **短**），通过电子邮件发送更长且更详细的消息是有意义的。为此，SNS 模块为我们提供了以下针对传送的特定选项

+   `email`

+   `http`

+   `https`

+   `sms`

+   `sqs`

正如我们在前一章中所看到的，AWS 模块需要凭据，我们可以以多种方式设置它们。运行此模块所需的三个 AWS 特定参数是：

+   `aws_access_key`：这是 AWS 访问密钥；如果未指定，则将考虑环境变量 `aws_access_key` 或 `~/.aws/credentials` 的内容。

+   `aws_secret_key`：这是 AWS 秘密密钥；如果未指定，则将考虑环境变量 `aws_secret_key` 或 `~/.aws/credentials` 的内容。

+   `region`：这是要使用的 AWS 区域；如果未指定，则将考虑环境变量 `ec2_region` 或 `~/.aws/config` 的内容。

使用以下命令运行代码：

```
    ansible-playbook uptime_and_sns.yaml  
```

这将导致以下输出：

```
PLAY [localhost] ************************************************* 

TASK [setup] ***************************************************** 
ok: [localhost] 

TASK [Read the machine uptime] *********************************** 
changed: [localhost] 

TASK [Send the uptime to SNS] ************************************ 
changed: [localhost] 

PLAY RECAP ******************************************************* 
localhost         : ok=3    changed=2    unreachable=0    failed=0 
```

有时我们希望通知监控系统，以便它不会由于 Ansible 操作而触发任何警报。这种系统的常见示例是 Nagios。

# Nagios

**Nagios**是用于监控服务和服务器状态的最常用工具之一。Nagios 能够定期审计服务器和服务的状态，并在出现问题时通知用户。如果您的环境中有 Nagios，那么在管理机器时必须非常小心，因为在 Nagios 发现服务器或服务处于不健康状态时，它会开始发送电子邮件和短信并向您的团队打电话。当您对由 Nagios 控制的节点运行 Ansible 脚本时，您必须更加小心，因为您会面临在夜间或其他不适当时间触发电子邮件、短信消息和电话的风险。为了避免这种情况，Ansible 能够事先通知 Nagios，以便在那个时间窗口内 Nagios 不会发送通知，即使一些服务处于下线状态（例如，因为它们正在重新启动）或其他检查失败。

在这个例子中，我们将会停止一个服务，等待五分钟，然后再次启动它，因为这实际上会导致在大多数配置中出现 Nagios 故障。事实上，通常情况下，Nagios 被配置为接受最多两次连续的测试失败（通常每分钟执行一次测试），在提升为关键状态之前将服务置于警告状态。我们将创建名为`long_restart_service.yaml`的文件，这将触发 Nagios 关键状态：

```
---
- hosts: ws01.fale.io 
  tasks: 
    - name: Stop the HTTPd service 
      service: 
        name: httpd 
        state: stopped 
    - name: Wait for 5 minutes 
      pause: 
        minutes: 5 
    - name: Start the HTTPd service 
      service: 
        name: httpd 
        state: stopped 
```

运行以下命令来执行代码：

```
ansible-playbook long_restart_service.yaml
```

这应该会触发 Nagios 警报，并导致以下输出：

```
PLAY [ws01.fale.io] ********************************************** 

TASK [setup] ***************************************************** 
ok: [ws01.fale.io] 

TASK [Stop the HTTpd service] ************************************ 
changed: [ws01.fale.io] 

TASK [Wait for 5 minutes] **************************************** 
changed: [ws01.fale.io] 

TASK [Start the HTTpd service] *********************************** 
changed: [ws01.fale.io] 

PLAY RECAP ******************************************************* 
ws01.fale.io      : ok=4    changed=3    unreachable=0    failed=0 
```

如果没有触发 Nagios 警报，那么可能是因为您的 Nagios 安装没有跟踪该服务，或者五分钟不足以使其提升为关键状态。要检查，请联系管理您 Nagios 安装的人员或团队，因为 Nagios 允许完全配置到一个非常难以预测 Nagios 行为的地步，而不知道其配置是很难的。

现在我们可以创建一个非常相似的 playbook，以确保 Nagios 不会发送任何警报。我们将创建一个名为`long_restart_service_no_alert.yaml`的文件，其内容如下（完整代码可在 GitHub 上找到）：

```
---
- hosts: ws01.fale.io 
  tasks: 
    - name: Mute Nagios 
      nagios: 
        action: disable_alerts 
        service: httpd 
        host: '{{ inventory_hostname }}' 
      delegate_to: nagios.fale.io 
    - name: Stop the HTTPd service 
      service: 
        name: httpd 
        state: stopped 
   ...
```

正如你所见，我们添加了两个任务。第一个任务是告诉 Nagios 不要发送有关给定主机上 HTTPd 服务的警报，第二个任务是告诉 Nagios 再次开始发送有关该服务的警报。即使您没有指定服务，因此静音了该主机上的所有警报，我的建议是仅禁用您将要中断的警报，以便 Nagios 仍然能够在大多数基础架构上正常工作。

如果 playbook 运行在恢复警报之前失败，你的警报将保持*禁用*状态。

该模块的目标是切换 Nagios 警报以及安排停机时间，从 Ansible 2.2 开始，该模块还可以取消安排的停机时间。

使用以下命令来运行代码：

```
    ansible-playbook long_restart_service_no_alert.yaml  
```

这将触发 Nagios 警报，并导致以下输出（完整的代码输出可在 GitHub 上获得）：

```
    PLAY [ws01.fale.io] **********************************************

    TASK [setup] *****************************************************
    ok: [ws01.fale.io]

    TASK [Mute Nagios] ***********************************************
    changed: [nagios.fale.io]

    TASK [Stop the HTTpd service] ************************************
    changed: [ws01.fale.io]

  ...
```

要使用 Nagios 模块，您需要使用 `delegate_to` 参数将操作委托给 Nagios 服务器，如示例所示。

有时，与 Nagios 集成要实现的目标完全相反。事实上，你并不想把它静音，而是想让 Nagios 处理你的测试结果。一个常见的情况是，如果您想利用您的 Nagios 配置通知您的管理员一个任务的输出。为此，我们可以使用 Nagios 的 `nsca` 工具，将其集成到我们的 playbook 中。Ansible 还没有一个管理它的特定模块，但您可以使用命令模块来运行它，利用 `send_nsca` CLI 程序。

# 总结

在本章中，我们已经学习了如何让 Ansible 发送通知到其他系统和人员。您学会了通过电子邮件和消息服务（如 Slack）发送通知的方法。最后，你学会了如何在你运行 Nagios 时防止它发送关于系统健康状况的不必要通知。

在下一章中，我们将学习如何创建一个模块，以便您可以扩展 Ansible 来执行任何类型的任务。
