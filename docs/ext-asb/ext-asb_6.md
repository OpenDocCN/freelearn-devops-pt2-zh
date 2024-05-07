# 第六章：将所有内容整合在一起-集成

当您到达本章时，您将根据自己的需求成功创建自己的自定义模块和插件。现在，您可能会想知道接下来是什么？

Ansible 是一个伟大的社区产品。它为每个人提供了许多模块和插件。现在您已经熟悉了 Python API，已经编写了一个 Ansible 模块，可能还有一个插件，现在是时候回馈社区了。由于您有一些无法满足原生 Ansible 的需求，很可能其他人也需要进一步的帮助。让我们看看可以以各种方式回馈社区。

本章将介绍如何配置 Ansible 以集成您的模块到现有的 Ansible 库中。本章还将介绍如何分发您的模块并帮助改进 Ansible。

# 配置 Ansible

要充分利用 Ansible，有必要正确配置 Ansible。虽然默认设置对大多数用户来说已经足够，但高级用户可能希望微调一些东西并进行一些更改。

全局持久设置在位于`/etc/ansible/ansible.cfg`的 Ansible 配置文件中定义。但是，您也可以将自定义配置文件放在 Ansible play 的根目录或用户的主目录中。还可以通过设置环境变量来更改设置。

有这么多配置 Ansible 的方法，一个重要的问题就是，Ansible 如何优先考虑配置文件？在执行 playbook 过程中，它如何选择要使用的配置？

在 Ansible 版本 1.9 中，配置按以下顺序处理：

+   `ANSIBLE_CONFIG`：环境变量

+   `ansible.cfg`：从中调用 Ansible 的当前工作目录

+   `.ansible.cfg`：配置文件存储在用户的主目录中

+   `/etc/ansible/ansible.cfg`：如果找不到其他配置文件，则为默认配置文件

Ansible 将按照上述顺序处理配置。在执行过程中将使用找到的第一个配置。为了保持一切清晰，Ansible 不会合并配置文件。所有文件都是分开保存的。

## 环境配置

通过设置环境变量，您可以覆盖从配置文件加载的任何现有配置。在当前版本的 Ansible 中，环境配置具有最高优先级。要找到 Ansible 支持的环境变量的完整列表，您需要查看源代码。以下列表包含一些您应该了解的环境变量，以便使您的模块和插件正常工作：

| 环境变量 | 默认值 |
| --- | --- |
| `ANSIBLE_ACTION_PLUGINS` | `~/.ansible/plugins/action:/usr/share/ansible/plugins/action` |
| `ANSIBLE_CACHE_PLUGINS` | `~/.ansible/plugins/cache:/usr/share/ansible/plugins/cache` |
| `ANSIBLE_CALLBACK_PLUGINS` | `~/.ansible/plugins/callback:/usr/share/ansible/plugins/callback` |
| `ANSIBLE_CONNECTION_PLUGINS` | `~/.ansible/plugins/connection:/usr/share/ansible/plugins/connection` |
| `ANSIBLE_LOOKUP_PLUGINS` | `~/.ansible/plugins/lookup:/usr/share/ansible/plugins/lookup` |
| `ANSIBLE_INVENTORY_PLUGINS` | `~/.ansible/plugins/inventory:/usr/share/ansible/plugins/inventory` |
| `ANSIBLE_VARS_PLUGINS` | `~/.ansible/plugins/vars:/usr/share/ansible/plugins/vars` |
| `ANSIBLE_FILTER_PLUGINS` | `~/.ansible/plugins/filter:/usr/share/ansible/plugins/filter` |
| `ANSIBLE_KEEP_REMOTE_FILES` | `False` |
| `ANSIBLE_PRIVATE_KEY_FILE` | `None` |

### 注意

**有趣的事实**

如果在管理节点上安装了 cowsay，Ansible playbook 运行将使用 cowsay 并使输出更有趣。如果您不希望启用 cowsay，只需在配置文件中设置`nocows=0`。

# 贡献给 Ansible

在开始为 Ansible 做贡献之前，重要的是要知道在哪里以及如何做出贡献以及要做出什么样的贡献。为了减少重复劳动，您需要与社区保持联系。可能会出现这样的情况，您想要处理的功能已经被其他人处理，或者您认为可以修复的错误已经被其他人接手并正在处理。此外，可能会出现您需要社区帮助来完成某些任务的情况；也许您在某个地方卡住了，有一些未解答的问题。这就是社区发挥作用的地方。Ansible 有自己的 IRC 频道和邮件列表用于这些目的。

你可以加入`#ansible`频道，地址是[irc.freenode.net](http://irc.freenode.net)，在那里你可以与社区成员交谈，讨论功能，并获得帮助。这是人们互相在线聊天的地方。对于没有 IRC 客户端的用户，他们可以通过[`webchat.freenode.net/`](https://webchat.freenode.net/)连接 Web UI。然而，由于 Ansible 是一个全球社区，不是所有成员都会全天候可用，你的问题可能得不到答复。如果是这样，你可以向邮件列表发送邮件，这样问题更有可能引起核心开发人员和高级用户的注意。

你可能想加入以下邮件列表：

+   Ansible 项目列表：[`groups.google.com/forum/#!forum/ansible-project`](https://groups.google.com/forum/#!forum/ansible-project)（一个用于分享 Ansible 技巧和提问的一般用户讨论邮件列表）

+   Ansible 开发列表：[`groups.google.com/forum/#!forum/ansible-devel`](https://groups.google.com/forum/#!forum/ansible-devel)（讨论正在进行的功能，建议功能请求，获取扩展 Ansible 的帮助）

+   Ansible 公告列表：[`groups.google.com/forum/#!forum/ansible-announce`](https://groups.google.com/forum/#!forum/ansible-announce)（一个只读列表，分享有关 Ansible 新版本发布的信息）

Ansible 是一个托管在 GitHub 上的开源项目。任何拥有 GitHub 账户的人都可以为 Ansible 项目做出贡献。该项目通过 GitHub 拉取请求接受贡献。

# Galaxy-分享角色

为你想要自动化的任务编写一个 playbook 可以帮助你每次部署时节省时间和精力。如果你能与社区分享角色，这也可以为其他人节省时间。

Ansible 提供了一个很好的平台来分享你的操作。Galaxy 是一个平台，你可以在其中分享预打包的工作单元作为“角色”，这些角色可以集成或放入 playbooks 中使用。一些角色可以直接放入，而其他一些可能需要进行一些调整。此外，Galaxy 为每个共享的角色提供了可靠性评分。你可以从许多可用的角色中进行选择，对它们进行评分和评论。

角色托管在 GitHub 上。Galaxy 允许与 GitHub 集成，您可以使用现有的 GitHub 帐户登录到 Galaxy 并分享角色。要分享您的角色，请创建一个 GitHub 存储库，克隆它，并在克隆的存储库中初始化一个 Galaxy 角色。可以使用以下代码来完成：

```
$ ansible-galaxy init <role-name> --force
```

这将创建一个组织代码所需的目录结构。然后，您可以使用此目录结构创建一个 Ansible 角色。一旦您的角色准备就绪，请在 playbook 中对其进行测试，并验证其是否按预期工作。然后，您可以将其推送到 GitHub 存储库中。

将代码上传到 Galaxy，您需要使用您的 GitHub 帐户登录到 Galaxy 平台（[`galaxy.ansible.com`](https://galaxy.ansible.com)）。通过使用菜单中的**添加角色**选项并提供所需的凭据，Galaxy 将从您的 GitHub 存储库导入角色，并使其在 Galaxy 平台上对整个社区可用。

您可能还希望为存储库应用标签，Galaxy 默认将其视为版本号。这允许用户在不同版本之间进行选择。如果没有指定标签，用户将始终只能下载 GitHub 存储库上最新的可用代码。

# Galaxy-最佳实践

在编写任何您可能希望通过 Galaxy 分享的角色时，应遵循一些最佳实践，以确保最终用户一切顺利运行：

+   始终记录您所做的任何内容，并将其放在`README.md`文件中。这是最终用户在使用角色时所参考的文件。

+   明确包含和列出所有依赖项。不要假设任何内容。

+   使用角色名称作为变量的前缀。

+   在分享之前测试角色。您进行的测试越多，出现故障的可能性就越小。

这些最佳实践也适用于您在一般情况下对 Ansible 的任何贡献。无论您是开发模块或插件，还是编写计划与社区共享的角色，这些实践都可以确保一切顺利进行。虽然这不是强制性的，但强烈建议遵循这些最佳实践，以便为他人提供贡献变得更加容易，并在以后需要时理解和扩展。

# 分享模块和插件

到了这个阶段，您将开发自己的 Ansible 模块或插件。现在，您希望与朋友和陌生人分享，并帮助他们简化他们的任务。您可能还希望合作开发一个模块或插件，并需要公众的帮助。

GitHub 是一个很好的开发者协作平台之一。您可以在 GitHub 上创建一个存储库并将代码推送到其中。您可以将模块代码与 Ansible playbook 一起使用，演示您刚刚开发的模块或插件的使用方法。

GitHub 允许人们为一个项目做出贡献。通常将代码放在 GitHub 上是一个好主意，因为它提供了许多优势。除了鼓励协作性，它还提供版本控制，您可以在需要时回滚更改，并跟踪过去对代码库所做的任何更改。在协作过程中，您可以通过查看建议的更改来选择要解决的拉取请求以及要忽略的拉取请求，从而允许您对存储库进行控制。

## 将模块加入 Ansible

Ansible 模块托管在 Ansible 的两个单独的子存储库中，即：

+   `ansible-modules-core`

+   `ansible-modules-extras`

模块存储库`ansible-modules-core`包含了与 Ansible 一起提供的最受欢迎的模块。这些是最常用的核心模块，对于解决系统的基本功能至关重要。该存储库包含了几乎每个 Ansible 正常运行所需的基本功能。该存储库不直接接受模块提交。但是，如果您遇到任何错误，可以报告并修复错误。

模块存储库`ansible-modules-extras`是`ansible-modules`的一个子集，其中包含优先级较低的模块（即不能被视为核心模块的模块）。新模块将提交到此存储库。根据模块的受欢迎程度和完整性，模块可以晋升为核心模块。

作为托管在 GitHub 上的开源项目，Ansible 通过 GitHub 拉取请求接受贡献。要将您的模块加入 Ansible，您需要了解 GitHub 拉取请求的工作原理：

+   将 Ansible 项目从[`github.com/ansible/ansible-modules-extras`](https://github.com/ansible/ansible-modules-extras)或[`github.com/ansible/ansible-modules-core`](https://github.com/ansible/ansible-modules-core)分叉到您的 GitHub 帐户。

+   在[`github.com/ansible/ansible-modules-extras/issues`](https://github.com/ansible/ansible-modules-extras/issues)或[`github.com/ansible/ansible-modules-core/issues`](https://github.com/ansible/ansible-modules-core/issues)上为您要解决的功能提交问题。如果您要修复错误，应该已经存在一个针对该错误的问题。如果没有，请创建一个并分配给自己。

+   将您的模块或修复错误的补丁推送到您刚创建的错误编号。

+   提出一个拉取请求到源存储库（即 Ansible）。

完成后，审阅人员将验证代码，审查它，并检查它是否解决了问题。审查后，您可能会收到一些评论或更改请求，您需要进行修复。在您的代码合并之前可能会有多次迭代。

如果您的模块或插件合并到 Ansible 存储库中，它将在下一个版本中对所有 Ansible 用户可用。

# 将插件引入 Ansible

如前一章所述，Ansible 插件根据其功能被分类为不同的组，如操作、回调、查找等。与模块不同，Ansible 插件是 Ansible 存储库本身的一部分。没有像 extras 和 core 这样的不同存储库。您可以直接在 Ansible 存储库中提出问题，在邮件列表上讨论，并在获得批准后提交拉取请求。

以下链接列出了 Ansible 存储库中的现有插件：

[`github.com/ansible/ansible/tree/devel/lib/ansible/plugins`](https://github.com/ansible/ansible/tree/devel/lib/ansible/plugins)

## 需要记住的要点

提交新模块时，有几件事情需要牢记：

+   始终讨论您提出的功能。这将帮助您节省时间和精力，以防该功能已经在进行中。

+   并非您提出的所有功能都会被接受。始终会根据用例和模块/插件的带来的价值进行评估。

+   维护您编写的模块/插件是一个好的实践。

+   积极地解决并修复针对您的模块报告的任何错误。这将使您的模块更加可靠。

+   尽量使您的模块尽可能通用（即，它应该接受用户参数并相应地进行调整，为用户提供更大的灵活性）。尽管如此，它应该专注于创建时的特定任务。这会增加被接受的机会。

# 最佳实践

到目前为止，您应该已经熟悉并习惯了使用 Ansible Python API。您甚至可能拥有自己的 Ansible 模块或插件，希望与社区分享。在与社区分享您的模块、插件和角色时，您应该遵循以下几项最佳实践：

+   在提交拉取请求之前始终测试您的模块。

+   尽量使您的模块尽可能通用。

+   始终记录您所创建的任何内容，无论是模块、插件还是在 Galaxy 上共享的 Ansible 角色。

+   明确列出任何依赖关系。

+   在邮件列表和 IRC 频道上继续讨论。积极参与可以提高您的知名度。

# 总结

本章涵盖了配置您的 Ansible 环境以及如何将您的模块和插件放入 Ansible 存储库的主题。它还涉及了如何通过 Git 分发您的模块。本章还向您介绍了 Galaxy 平台，这是 Ansible 提供的一个服务，用于分享您的角色。本章还提供了最佳实践的指导，并介绍了在提交模块时应该牢记的各种事项。

下一章将带您通过一系列场景，演示 Ansible 可以派上用场的情况。本章还将整合前几章所涵盖的内容，将其结合起来，并呈现一个场景，让您了解如何充分利用 Ansible。
