# 下一步使用 Ansible

在本章中，我们将讨论如何将 Ansible 集成到您的日常工作流程中，从持续集成工具到监控工具和故障排除。我们将讨论以下主题：

+   如何将 Ansible 与 Slack 等服务集成

+   您如何可以使用 Ansible 来解决问题

+   一些真实世界的例子

让我们直接深入研究如何将我们的 playbooks 连接到第三方服务。

# 与第三方服务集成

尽管您可能是运行 playbooks 的人，但您可能会保留 playbook 运行的日志，或者让您团队的其他成员，甚至其他部门了解 playbook 运行的结果。Ansible 附带了几个核心模块，允许您与第三方服务一起工作，以提供实时通知。

# Slack

Slack 已经迅速成为各个 IT 服务部门团队协作服务的首选选择。它不仅通过其应用程序目录支持第三方应用程序，而且还具有强大的 API，您可以使用该 API 将您的工具带入 Slack 提供的聊天室。

我们将在本节中查看示例，完整的 playbook 可以在 GitHub 存储库的`Chapter17/slack`文件夹中找到。我已经从第九章中的 playbook 中获取了 playbook，*构建云网络*，在那里我们在 AWS 中创建了一个 VPC，并且我已经改编它以使用`slack` Ansible 模块。

# 生成令牌

在我们的 Playbook 中使用 Slack 模块之前，我们需要一个访问令牌来请求一个登录到您的 Slack 工作区；如果您还没有工作区，您可以免费注册一个工作区[`slack.com/`](https://slack.com/)。

一旦您登录到您的工作区，无论是使用 Web 客户端还是桌面应用程序，都可以从管理应用选项中选择管理应用选项，如下截图所示：

![](img/e1f3b335-5823-4712-bdea-36fa6db8418b.png)

这将打开您的浏览器，并将您带到您工作区的应用程序目录；从这里，搜索`传入 WebHooks`，然后点击添加配置。

配置的第一部分是选择您希望传入 Webhook 发布消息的频道。我选择了通用频道——一旦选择，您将被带到一个页面，该页面会给您一个 Webhook URL；确保您记下这个 URL，因为我们很快就会需要它。在页面底部，您可以选择自定义您的 Webhook。

在页面底部的集成设置中，我输入了以下信息：

+   发布到频道：我将其留在#general

+   Webhook URL：这是为您预填充的；您还可以选择在此重新生成 URL

+   描述性标签：我在这里输入了`Ansible`

+   自定义名称：我也在这里输入了`Ansible`

+   自定义图标：我将其保留为原样

填写完前面的细节后，我点击了保存设置按钮；这让我得到了一个传入的 Webhook：

![](img/76ce0142-b8e1-4210-bfe2-85b7d536d0d8.png)

如前所述，我还记录了 Webhook URL；对我来说，它是：

`https://hooks.slack.com/services/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`

现在我们已经准备好了一切，我们需要在 Slack 端进行配置，以便开始使用 Ansible 向我们的用户发送消息。

# Ansible playbook

我只会在这里介绍单个角色任务的更新，这是创建 VPC 的角色。我做的第一个更改是在`group_vars/common.yml`文件中添加了几行：

```
---

environment_name: "VPC-Slack"
ec2_region: "eu-west-1"

slack:
  token: "TBCRVDMGA/BBCPTPNH1/tyudQIccviG7gh4JnfeoPScc"
  username: "Ansible"
  icon: "https://upload.wikimedia.org/wikipedia/commons/thumb/0/05/Robot_icon.svg/200px-Robot_icon.svg.png"
```

正如您所看到的，我添加了以下三个嵌套变量：

+   `令牌`：这是从 Webhook URL 中获取的；如您所见，我输入了`https://hooks.slack.com/services/`后的所有内容

+   `用户名`：我们可以通过它来覆盖将发布更新的用户名，我只是将其保留为 Ansible

+   `图标`：这是我们发布的一部分将显示的头像

如果您还记得之前的 VPC 角色，您会记得它包含一个使用`ec2_vpc_net`模块创建 VPC 的单个任务。现在，我们想引入 Slack 通知，并能够向用户提供反馈。因此，首先，让我们发送通知，说我们正在检查 VPC 是否存在：

```
- name: Send notification message via Slack all options
  slack:
    token: "{{ slack.token }}"
    msg: "Checking for VPC called '{{ environment_name }}'"
    username: "{{ slack.username }}"
    icon_url: "{{ slack.icon }}"
    link_names: 0
    parse: 'full'
```

从前面的任务中可以看到，我们正在发送一条消息，在我们的情况下，它将读取`Checking for VPC called 'VPC-Slack'`，以及`token`、`username`和`icon`。角色中的下一个任务是原始角色中的任务：

```
- name: ensure that the VPC is present
  ec2_vpc_net:
    region: "{{ ec2_region }}"
    name: "{{ environment_name }}"
    state: present
    cidr_block: "{{ vpc_cidr_block }}"
    resource_tags: { "Name" : "{{ environment_name }}", "Environment" : "{{ environment_name }}" }
  register: vpc_info
```

现在，可能发生了两种情况：一个名为`VPC-Slack`的 VPC 已经创建，或者 Ansible 已经收集了关于名为`VPC-Slack`的现有 VPC 的信息。当我们向用户发送消息时，它应该根据 Ansible 的操作而改变。以下任务发送一条消息，通知我们的用户已经创建了一个新的 VPC：

```
- name: Send notification message via Slack all options
  slack:
    token: "{{ slack.token }}"
    msg: "VPC called '{{ environment_name }}' created with an ID of '{{ vpc_info.vpc.id }}'"
    username: "{{ slack.username }}"
    icon_url: "{{ slack.icon }}"
    link_names: 0
    parse: 'full'
  when: vpc_info.changed
```

请注意，只有在我注册的`vpc_info`变量标记为更改时，我才运行此任务。此外，我将 VPC 的 ID 作为消息的一部分传递。如果`vpc_info`没有注册任何更改，那么前面的任务将被跳过；而后面的任务将会运行：

```
- name: Send notification message via Slack all options
  slack:
    token: "{{ slack.token }}"
    msg: "Found a VPC called '{{ environment_name }}' which has an ID of '{{ vpc_info.vpc.id }}'"
    username: "{{ slack.username }}"
    icon_url: "{{ slack.icon }}"
    link_names: 0
    parse: 'full'
  when: vpc_info.changed == false and vpc_info.failed == false
```

请注意我如何改变措辞，以及它仅在没有更改时才被调用。我浏览了其他角色，添加了使用与前面代码相同逻辑的任务，向 Slack 发送通知；如前所述，您可以在存储库的`Chapter17/slack`文件夹中找到所有添加。

# 运行 playbook

运行 playbook 时，请使用以下命令：

```
$ export AWS_ACCESS_KEY=AKIAI5KECPOTNTTVM3EDA
$ export AWS_SECRET_KEY=Y4B7FFiSWl0Am3VIFc07lgnc/TAtK5+RpxzIGTr
$ ansible-playbook -i production site.yml
```

我从 Slack 收到了以下通知：

![](img/0367dde3-92bd-456f-a67c-c06edc828a1e.png)

正如您所看到的，许多消息都是关于在 VPC 中创建的服务。立即重新运行 playbook 后，返回以下结果：

![](img/b0174ed9-8e7a-4925-928f-f7c13b3d1277.png)

这一次，消息是关于查找现有服务并返回 ID。Slack 只是一个服务。现在让我们简要地看一下您可以从 Ansible playbook 与之交互的其他一些服务。

# 其他服务

Slack 不是 Ansible 可以与之交互的唯一服务；以下是您可能希望在 playbook 中使用的其他一些服务。

# Campfire

Campfire 是内置在 Basecamp 中的聊天服务；您可以使用此模块直接从 Ansible 向项目利益相关者发送更新，例如：

```
- name: Send a message to Campfire
  campfire:
    subscription: "my_subscription"
    token: "my_subscription"
    room: "Demo"
    notify: "loggins"
    msg: "The task has completed and all is well"
```

# Cisco Webex Teams（Cisco Spark）

Cisco Webex Teams，或者正式称为 Cisco Spark，是 Cisco 提供的协作服务，为您的团队提供虚拟会议空间、消息和视频通话。此外，它还具有丰富的 API，可以配置 Ansible 与之交互：

```
- name: Send a message to Cisco Spark
  cisco_spark:
    recipient_type: "roomId"
    recipient_id: "{{ spark.room_id }}"
    message_type: "markdown"
    personal_token: "{{ spark.token }}"
    message: "The task has **completed** and all is well"
```

# CA Flowdock

CA Flowdock 是一个消息服务，从头开始设计，以与面向开发人员的服务集成，如 GitHub、Bitbucket、Jira、Jenkins 和 Ansible：

```
- name: Send a message to a Flowdock inbox
  flowdock:
    type: "inbox"
    token: "{{ flowdock.token }}"
    from_address: "{{ flowdock.email }}"
    source: "{{ flowdock.source }}"
    msg: "The task has completed and all is well"
    subject: "Task Success"
```

# Hipchat

Hipchat 是由 Atlassian 提供的群组消息服务；它与 Atlassian 产品系列的其他产品紧密集成：

```
- name: Send a message to a Hipchat room
  hipchat:
    api: "https://api.hipchat.com/v2/"
    token: "{{ hipchat.token }}"
    room: "{{ hipchat.room }}"
    msg: "The task has completed and all is well"
```

# Mail

这项服务不需要任何介绍；可以配置 Ansible 使用各种配置发送电子邮件。以下示例显示了通过外部 SMTP 服务器发送电子邮件：

```
- name: Send an email using external mail servers
  mail:
    host: "{{ mail.smtp_host }}"
    port: "{{ mail.smtp_port }}"
    username: "{{ mail.smtp_username }}"
    password: "{{ mail.smtp_password }}"
    to: "Russ McKendrick <russ@mckendrick.io>"
    subject: "Task Success"
    body: "The task has completed and all is well"
  delegate_to: localhost
```

# Mattermost

Mattermost 是专有服务的开源替代品，类似于我们在列表中其他地方介绍的服务（例如 Slack、Cisco Webex Teams 和 Hipchat）：

```
- name: Send a message to a Mattermost channel
  mattermost:
    url: "{{ mattermost.url }}"
    api_key: "{{ mattermost.api_key }}"
    text: "The task has completed and all is well"
    channel: "{{ mattermost.channel }}"
    username: "{{ mattermost.username }}"
    icon_url: "{{ mattermost.icon_url }}"
```

# Say

大多数现代计算机都内置了一定程度的语音合成；使用此模块，您可以让 Ansible 口头通知您 playbook 运行的状态：

```
- name: Say a message on your Ansible host
  say:
    msg: "The task has completed and all is well"
    voice: "Daniel"
  delegate_to: localhost
```

# ServiceNow

ServiceNow 是 ServiceNow, Inc.提供的企业级 IT 服务管理软件即服务产品。使用`snow_record`模块，您的 playbook 可以在 ServiceNow 安装中打开事件：

```
- name: Create an incident in ServiceNow
  snow_record:
    username: "{{ snow.username }}"
    password: "{{ snow.password }}"
    instance: "{{ snow.instance }}"
    state: "present"
    data:
      short_description: "The task has completed and all is well"
      severity: "3"
      priority: "3"
  register: snow_incident
```

# Syslog

如果您从主机发送日志文件，则可能希望将 playbook 运行的结果发送到主机 syslog，以便将其发送到您的中央日志服务：

```
- name: Send a message to the hosts syslog
  syslogger:
    msg: "The task has completed and all is well"
    priority: "info"
    facility: "daemon"
    log_pid: "true"
```

# Twilio

使用您的 Twilio 帐户直接从您的 Ansible playbook 发送短信消息，如下所示：

```
- name: Send an SMS message using Twilio
  twilio:
    msg: "The task has completed and all is well"
    account_sid: "{{ twilio.account }}"
    auth_token: "{{ twilio.auth }}"
    from_number: "{{ twilio.from_mumber }}"
    to_number: "+44 7911 123456"
  delegate_to: localhost
```

# 第三方服务摘要

我希望您从这本书中得到的一个要点是自动化很棒——它不仅可以节省时间，而且使用我们在上一章中介绍的工具，如 Ansible Tower 和 Ansible AWX，可以让非系统管理员或开发人员从友好的 Web 界面执行他们的 playbook。

我们在本节中涵盖的模块不仅允许您记录结果，还可以在播放过程中自动进行一些清理工作，并让它自己通知您的用户，从而使您的自动化水平提升到一个新的高度。

例如，假设您需要将新配置部署到服务器。您的服务台为您提出更改，以便您在 ServiceNow 安装中执行工作。您的 playbook 可以这样编写，在执行更改之前，它使用`fetch`模块将配置文件复制到您的 Ansible Controller。然后 playbook 可以使用`snow_record`模块将现有配置文件的副本附加到更改请求，继续进行更改，然后自动更新更改请求的结果。

您可以在本章中提到的服务的以下 URL 中找到详细信息：

+   **Slack**：[`slack.com/`](https://slack.com/)

+   **Campfire**：[`basecamp.com/`](https://basecamp.com/)

+   思科 Webex 团队（思科 Spark）：[`www.webex.com/products/teams/`](https://www.webex.com/products/teams/)

+   **CA Flowdock**：[`www.flowdock.com/`](https://www.flowdock.com/)

+   **Mattermost**：[`mattermost.com/`](https://mattermost.com/)

+   **ServiceNow**：[`www.servicenow.com/`](https://www.servicenow.com/)

+   **Twilio**：[`twilio.com/`](https://twilio.com/)

# Ansible playbook 调试器

Ansible 内置了调试器。让我们看看如何通过创建一个带有错误的简单 playbook 将其构建到您的 playbook 中。正如我们刚才提到的，我们将编写一个使用`say`模块的 playbook。playbook 本身如下所示：

```
---

- hosts: localhost
  gather_facts: false
  debugger: "on_failed"

  vars:
    message: "The task has completed and all is well"
    voice: "Daniel"

  tasks:
    - name: Say a message on your Ansible host
      say:
        msg: "{{ massage }}"
        voice: "{{ voice }}"
```

有两件事需要指出：第一是错误。正如您所看到的，我们正在定义一个名为`message`的变量，但是当我们使用它时，我输错了，输入了`massage`。幸运的是，因为我正在开发 playbook，每当任务失败时，我都指示 Ansible 进入交互式调试器。

# 调试任务

让我们运行 playbook，看看会发生什么：

```
$ ansible-playbook playbook.yml
```

第一个问题是我们没有传递主机清单文件，因此将会收到警告，只有本地主机可用；这没关系，因为我们只想在我们的 Ansible Controller 上运行`say`模块：

```
[WARNING]: Unable to parse /etc/ansible/hosts as an inventory source
[WARNING]: No inventory was parsed, only implicit localhost is available
[WARNING]: provided hosts list is empty, only localhost is available. Note that the implicit
localhost does not match 'all'
```

接下来，Ansible 运行 play 本身；这应该会导致致命错误：

```
PLAY [localhost] ***********************************************************************************

TASK [Say a message on your Ansible host] **********************************************************
fatal: [localhost]: FAILED! => {"msg": "The task includes an option with an undefined variable. The error was: 'massage' is undefined\n\nThe error appears to have been in '/Users/russ/Documents/Code/learn-ansible-fundamentals-of-ansible-2x/chapter17/say/playbook.yml': line 12, column 7, but may\nbe elsewhere in the file depending on the exact syntax problem.\n\nThe offending line appears to be:\n\n tasks:\n - name: Say a message on your Ansible host\n ^ here\n"}
```

通常，playbook 运行将停止，并且您将返回到您的 shell；但是，因为我们已经指示 Ansible 进入交互式调试器，现在我们看到以下提示：

```
[localhost] TASK: Say a message on your Ansible host (debug)>
```

从这里开始，我们可以更仔细地研究问题；例如，我们可以通过输入以下命令来查看错误：

```
p result._result
```

一旦您按下*Enter*键，失败任务的结果将返回：

```
[localhost] TASK: Say a message on your Ansible host (debug)> p result._result
{'failed': True,
 'msg': u"The task includes an option with an undefined variable. The error was: 'massage' is undefined\n\nThe error appears to have been in '/Users/russ/Documents/Code/learn-ansible-fundamentals-of-ansible-2x/chapter17/say/playbook.yml': line 12, column 7, but may\nbe elsewhere in the file depending on the exact syntax problem.\n\nThe offending line appears to be:\n\n tasks:\n - name: Say a message on your Ansible host\n ^ here\n"}
[localhost] TASK: Say a message on your Ansible host (debug)>
```

通过输入以下内容，让我们更仔细地查看任务中使用的变量：

```
p task.args
```

这将返回我们在任务中使用的两个参数：

```
[localhost] TASK: Say a message on your Ansible host (debug)> p task.args
{u'msg': u'{{ massage }}', u'voice': u'{{ voice }}'}
[localhost] TASK: Say a message on your Ansible host (debug)>
```

现在，让我们通过以下方式查看任务可用的变量：

```
p task_vars
```

您可能已经注意到，我们指示 Ansible 在 playbook 运行中执行 setup 模块；这是为了将可用变量列表保持在最小范围内：

```
[localhost] TASK: Say a message on your Ansible host (debug)> p task_vars
{'ansible_check_mode': False,
 'ansible_connection': 'local',
 'ansible_current_hosts': [u'localhost'],
 'ansible_diff_mode': False,
 'ansible_facts': {},
 'ansible_failed_hosts': [],
 'ansible_forks': 5,
 'ansible_inventory_sources': [u'/etc/ansible/hosts'],
 'ansible_play_batch': [],
 'ansible_play_hosts': [u'localhost'],
 'ansible_play_hosts_all': [u'localhost'],
 'ansible_playbook_python': '/usr/bin/python',
 'ansible_python_interpreter': '/usr/bin/python',
 'ansible_run_tags': [u'all'],
 'ansible_skip_tags': [],
 'ansible_version': {'full': '2.5.5',
 'major': 2,
 'minor': 5,
 'revision': 5,
 'string': '2.5.5'},
 'environment': [],
 'group_names': [],
 'groups': {'all': [], 'ungrouped': []},
 'hostvars': {},
 'inventory_hostname': u'localhost',
 'inventory_hostname_short': u'localhost',
 u'message': u'The task has completed and all is well',
 'omit': '__omit_place_holder__0529a2749315462e1ae1a0d261987dedea3bfdad',
 'play_hosts': [],
 'playbook_dir': u'/Users/russ/Documents/Code/learn-ansible-fundamentals-of-ansible-2x/chapter17/say',
 u'voice': u'Daniel'}
[localhost] TASK: Say a message on your Ansible host (debug)>
```

正如您从前面的输出中所看到的，关于我们的 playbook 正在执行的环境有很多信息。在变量列表中，您会注意到其中两个变量以`u`为前缀：它们是`voice`和`message`。我们可以通过运行以下命令来了解更多信息：

```
p task_vars['message']
p task_vars['voice']
```

这将显示变量的内容：

```
[localhost] TASK: Say a message on your Ansible host (debug)> p task_vars['message']
u'The task has completed and all is well'
[localhost] TASK: Say a message on your Ansible host (debug)> p task_vars['voice']
u'Daniel'
[localhost] TASK: Say a message on your Ansible host (debug)>
```

我们知道我们正在将一个拼写错误的变量传递给`msg`参数，因此我们可以即时进行一些更改并继续 playbook 运行。为此，我们将运行以下命令：

```
task.args['msg'] = '{{ message }}'
```

这将更新参数以使用正确的变量意思，这样我们可以通过运行以下命令重新运行任务：

```
redo
```

这将立即使用正确的参数重新运行任务，并且幸运的话，您应该会听到*任务已完成，一切正常*：

```
[localhost] TASK: Say a message on your Ansible host (debug)> task.args['msg'] = '{{ message }}'
[localhost] TASK: Say a message on your Ansible host (debug)> redo
changed: [localhost]

```

```
PLAY RECAP ************************************************************************************************************************************
localhost : ok=1 changed=1 unreachable=0 failed=0
```

如您从前面的输出中所看到的，因为我们只有一个任务，playbook 已经完成。如果我们有更多任务，那么它将从离开的地方继续。现在您可以使用正确的拼写更新您的 playbook，并继续您的一天。

此外，如果我们愿意，我们可以输入`continue`或`quit`来分别继续或停止。

# Ansible 调试器摘要

当您正在创建大型 playbook 时，启用 Ansible 调试器是一个非常有用的选项——例如，想象一下，您有一个大约需要运行 20 分钟的 playbook，但是在结束时出现了一个错误，比如在您第一次运行 playbook 后的 15 分钟。

让 Ansible 进入交互式调试器 shell 不仅意味着您可以准确地看到定义和未定义的内容，而且还意味着您不必盲目地对 playbook 进行更改，然后等待另外 15 分钟以查看这些更改是否解决了导致致命错误的问题。

# 真实世界的例子

在我们结束本章，也结束本书之前，我想给出一些我如何使用 Ansible 和与 Ansible 交互的例子：第一个是使用聊天与 Ansible 交互。

# 聊天示例

几个月前，我需要设置一个演示来展示自动化工作，但是，我需要能够在我的笔记本电脑或手机上展示演示，这意味着我不能假设我可以访问命令行。

我想出的演示最终使用了 Slack 和其他一些我们在这本书中没有涵盖的工具，即 Hubot 和 Jenkins；在我详细介绍之前，让我们快速看一下演示运行的输出：

！[](assets/d7fb0ae5-d86c-4d81-a9b2-b64a66d6dcad.png)

如前面的输出所示，我在 Slack 频道中提出了以下问题：

*@bot 给我一个 Linux 服务器*

然后触发了一个 Ansible playbook 运行，该运行在 AWS 中启动了一个实例，并在 playbook 确认服务器在网络上可用后返回实例的信息。我还配置它通过询问以下内容来删除所有正在运行的实例：

*@bot 终止所有服务器*

如您所见，这运行了另一个 playbook，并且这次，在实例被删除后返回了一个动画 GIF：

！[](assets/2171890b-ee00-4138-adee-d95714316cab.png)

那么，我用什么做到这一点呢？正如前面提到的，首先，我使用了 Hubot。Hubot 是 GitHub 开发的一个开源可扩展的聊天机器人。它是在我的 Slack 频道中使用`hubot-slack`插件进行配置的，并且它会监听任何给定的命令。

我使用`hubot-alias`插件来定义一个别名，将*@bot 给我一个 Linux 服务器*翻译成*build awslaunch OS=linux*；这使用了`hubot-yardmaster`插件来触发我的 Jenkins 安装中的构建。

Jenkins 是一个开源的自动化服务器，主要用于持续集成和持续交付，它也有一个插件架构。使用 Jenkins Ansible 插件和 Jenkins Git 插件，我能够将用于启动 AWS 实例的 playbook 和角色拉到我的 Jenkins 服务器上，然后让 Jenkins 为我运行 playbook——playbook 本身与我们在第九章和第十章中讨论的 playbook 并没有太大不同，分别是*构建云网络*和*高可用云部署*。

playbook 中内置了一些逻辑，限制了可以启动的实例数量，随机化了要启动的实例的名称，并从几个选项中显示了一个随机的 GIF 图像——所有这些信息，以及实例和 AMI 的详细信息，都通过 Ansible Slack 模块传递给用户，给人一种 playbook 实际上做了更多事情的印象。

在前面的两个例子中，机器人用户是 Hubot，而 Jenkins 实际上是 playbook 运行的反馈。

# 自动化部署

另一个例子——我最近与几位开发人员合作，他们需要一种自动将代码部署到开发和分级服务器的方法。使用 Docker、GitHub、Jenkins 和 Ansible AWX 的组合，我能够为开发人员提供一个工作流程，每当他们将代码推送到 GitHub 存储库的开发或分级分支时都会触发。

为了实现这一点，我在他们自己的 Jenkins 服务器上部署了代码，使用 Ansible 在容器中部署了 Jenkins，并在同一台服务器上使用 Docker 部署了 AWX。然后，使用**Jenkins GitHub**插件，我将 Jenkins 项目连接到 GitHub 以创建触发构建所需的 Webhooks。然后使用**Jenkins Ansible Tower**插件，我让 Jenkins 触发 AWX 中的 playbook 运行。

我这样做是因为目前，AWX 与 GitHub Webhooks 的连接并不那么容易，而**Jenkins**和**Jenkins GitHub**插件具有很高的兼容性——我想随着 AWX 的开发速度，这个小问题很快就会得到解决。

AWX 允许您根据角色授予 playbooks 的访问权限，我给了开发经理和运维工程师运行生产 playbook 的权限，开发人员只有只读权限，以便他们可以查看 playbook 运行的结果。

这意味着部署到生产环境也能够自动化，只要有正确权限的人手动触发 playbook 运行。

AWX 允许我们控制谁可以触发部署，这与我们现有的部署策略相吻合，该策略规定开发人员不应该有权限部署他们编写的代码到生产系统。

# 总结

现在我们不仅结束了这一章，也结束了我们的书。我一直在努力想出一个总结 Ansible 的方法，我在 Ansible 的创建者 Michael DeHaan 的一条推特中找到了答案([`twitter.com/laserllama/status/976135074117808129`](https://twitter.com/laserllama/status/976135074117808129))，他在回复一位技术招聘人员时说：

"使用 Ansible 几个月的人和使用 Ansible 三年的人一样好。这是一个故意简单的工具。"

这完美地总结了我的 Ansible 经验，希望也适用于你。一旦掌握了基础知识，就可以很容易地快速进展，开始构建更加复杂的 playbooks，这些 playbooks 不仅可以帮助部署基本的代码和应用程序，还可以帮助部署复杂的云和甚至物理架构。

不仅能够重用自己的角色，而且可以通过 Ansible Galaxy 访问大量社区贡献的角色，意味着你有许多示例或快速起点用于下一个项目。因此，你可以更快地投入工作，比起其他工具可能更快。此外，如果 Ansible 无法做某事，那么很可能有一个可以集成的工具来提供缺失的功能。

回到我们在第一章讨论过的内容，《Ansible 简介》，能够以可重复和可共享的方式以代码定义基础架构和部署，鼓励他人贡献到你的 playbooks 中，这应该是引入 Ansible 到日常工作流程中的最终目标。我希望通过这本书，你已经开始考虑 Ansible 可以帮助你节省时间的日常任务。

# 更多阅读材料

本章提到的工具的更多信息可以在以下网址找到：

+   **Hubot**: [`hubot.github.com`](https://hubot.github.com)

+   **Hubot Slack**: [`github.com/slackapi/hubot-slack`](https://github.com/slackapi/hubot-slack)

+   **Hubot Alias**: [`github.com/dtaniwaki/hubot-alias`](https://github.com/dtaniwaki/hubot-alias)

+   **Hubot Yardmaster**: [`github.com/hacklanta/hubot-yardmaster`](https://github.com/hacklanta/hubot-yardmaster)

+   **Jenkins Git**: [`plugins.jenkins.io/git`](https://plugins.jenkins.io/git)

+   **Jenkins Ansible**: [`plugins.jenkins.io/ansible`](https://plugins.jenkins.io/ansible)

+   **Jenkins GitHub**: [`plugins.jenkins.io/github`](https://plugins.jenkins.io/github)

+   **Jenkins Ansible Tower**: [`plugins.jenkins.io/ansible-tower`](https://plugins.jenkins.io/ansible-tower)
