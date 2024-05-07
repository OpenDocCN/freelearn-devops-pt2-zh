# 第四章：探索 API

Ansible 插件是一个高级话题。有各种插件可用于 Ansible。本章将简要介绍不同的 Python API 和查找插件，并探讨它们如何适应通用的 Ansible 架构。

Ansible 在很多方面都是可插拔的。可能存在一些业务逻辑组件不太适合的情况。因此，Ansible 提供了可以用于满足您业务需求的扩展点。Ansible 插件是另一个这样的扩展点，您可以构建自己的插件来扩展 Ansible 以满足您的业务逻辑。

# Python API

在探索插件之前，了解 Ansible Python API 很重要。Ansible Python API 可用于以下用途：

+   控制节点

+   响应各种 Python 事件

+   根据需求编写各种插件

+   还可以将来自各种外部数据存储的清单数据插入其中

Ansible 的 Python API 允许以编程方式运行 Ansible。通过 Python API 以编程方式运行 Ansible 具有以下优势：

+   **更好的错误处理**：由于一切都是 Python，因此很容易在发生错误时进行处理。这样可以通过提供更好的上下文，在发生错误时更好地控制和信心。

+   **扩展 Ansible**：正如您在之前的运行中可能已经注意到的，一个缺点是，默认情况下，Ansible 只是将输出写入`stdout`，并不会将任何内容记录到文件中。为了解决这个问题，您可以编写自己的自定义插件，将输出保存到文件或数据库以供将来参考。

+   **未知变量**：可能存在只有在运行时才能完全了解所需变量的情况，例如，在 Ansible 执行期间在云上启动实例的 IP。使用 Python API 以编程方式运行 Ansible 可以解决这个问题。

现在您已经了解了使用 Python API 进行 Ansible 的优势，让我们来探索 Python API，并看看如何通过 API 与 Ansible 进行交互。

本章将介绍三个最重要的广泛使用的类：

+   **Runner**：用于执行单个模块

+   **Playbook**：帮助执行 Ansible playbook

+   **回调**：在控制节点上获取运行结果

让我们深入了解这些类是什么，并探索各种扩展点。

## Runner

`runner`类是 Ansible 的核心 API 接口。`runner`类用于执行单个模块。如果有一个需要执行的单个模块，例如`setup`模块，我们可以使用`runner`类来执行此模块。

### 提示

可以在同一个 Python 文件中有多个`runner`对象来运行不同的模块。

让我们来探索一个示例代码，其中`runner`类将用于在本地主机上执行`setup`模块。这将打印有关本地主机的许多详细信息，例如时间、操作系统（发行版）、IP、子网掩码以及硬件详细信息，例如架构、空闲内存、已用内存、机器 ID 等等。

```
from ansible import runner

runner = runner.Runner(
    module_name = 'setup',
    transport = 'local'
)

print runner.run()
```

这将在本地主机上执行`setup`模块。这相当于运行以下命令：

```
**ansible all -i "localhost," -c local -m setup**

```

要在远程主机或一组主机上运行上述模块，可以在清单中指定主机，然后将其作为参数传递给`runner`对象，以及应用于登录远程机器的远程用户。您还可以指定主机的模式，特别是需要执行模块的主机。这是通过将模式参数传递给`runner`对象来完成的。

### 提示

您还可以使用`module_args`键传递模块参数。

例如，如果您需要获取设置为`store1.mytestlab.com`、`store2.mytestlab.com`、`store12.mytestlab.com`等的远程主机的内存详细信息，可以简单地以以下方式实现：

```
from ansible import runner

runner = runner.Runner(
    module_name = 'setup',
    pattern = 'store*',
    module_args = 'filter=ansible_memory_mb'
)

print runner.run()
```

上述代码将在所有十二个主机上执行`setup`模块，并打印每个主机可访问的内存状态。可访问的主机将列在“contacted”下，而不可访问的主机将列在“dark”下。

除了上面讨论的参数外，`runner`类通过其接受的参数提供了大量的接口选项。以下是源代码中定义的一些参数及其用途的列表：

| 参数/默认值 | 描述 |
| --- | --- |
| `host_list=C.DEFAULT_HOST_LIST` | 示例：`/etc/ansible/hosts`，传统用法 |
| `module_path=None` | 示例：`/usr/share/ansible` |
| `module_name=C.DEFAULT_MODULE_NAME` | 示例：`copy` |
| `module_args=C.DEFAULT_MODULE_ARGS` | 示例："`src=/tmp/a dest=/tmp/b`" |
| `forks=C.DEFAULT_FORKS` | 并行级别 |
| `timeout=C.DEFAULT_TIMEOUT` | SSH 超时 |
| `pattern=C.DEFAULT_PATTERN` | 哪些主机？示例："all" `acme.example.org` |
| `remote_user=C.DEFAULT_REMOTE_USER` | 例如：“`username`” |
| `remote_pass=C.DEFAULT_REMOTE_PASS` | 例如：“`password123`”或使用密钥时为“`None`” |
| `remote_port=None` | 如果 SSH 在不同的端口上 |
| `private_key_file=C.DEFAULT_PRIVATE_KEY_FILE` | 如果不使用密钥/密码 |
| `transport=C.DEFAULT_TRANSPORT` | “`SSH`”，“`paramiko`”，“`Local`” |
| `conditional=True` | 仅在此事实表达式评估为`true`时运行 |
| `callbacks=None` | 用于输出 |
| `sudo=False` | 是否运行 sudo |
| `inventory=None` | 对清单对象的引用 |
| `environment=None` | 在命令内部使用的环境变量（作为`dict`） |
| `complex_args=None` | 除了`module_args`之外的结构化数据，必须是`dict` |

## Playbook

正如您在之前的章节中学到的那样，Playbook 是以 YAML 格式运行的一系列指令或命令。Ansible 的 Python API 提供了一个丰富的接口，通过`PlayBook`类运行已创建的 playbooks。

您可以创建一个`PlayBook`对象，并将现有的 Ansible playbook 作为参数传递，以及所需的参数。需要注意的一点是，多个 play 不会同时执行，但是 play 中的任务可以根据请求的 forks 数量并行执行。创建对象后，可以通过调用`run`函数轻松执行 Ansible playbook。

您可以创建一个`Playbook`对象，稍后可以使用以下模板执行：

```
pb = PlayBook(
    playbook = '/path/to/playbook.yaml',
    host_list = '/path/to/inventory/file',
    stats = 'object/of/AggregateStats',
    callbacks = 'playbookCallbacks object',
    runner_callbacks = 'callbacks/used/for/Runner()'
)
```

这里需要注意的一件事是，`PlayBook`对象需要至少传递四个必需的参数。这些是：

+   `playbook`：Playbook 文件的路径

+   `stats`：保存每个主机发生的事件的聚合数据

+   `callbacks`：Playbook 的输出回调

+   `runner_callbacks`：`runner` API 的回调

您还可以在`0`-`4`的范围内定义详细程度，这是`callbacks`和`runner_callbacks`对象所需的。如果未定义详细程度，则默认值为`0`。将详细程度定义为`4`相当于在命令行中执行 Ansible playbook 时使用`-vvvv`。

例如，您有名为`hosts`的清单文件和名为`webservers.yaml`的 playbook。要使用 Python API 在清单主机上执行此 playbook，您需要创建一个带有所需参数的`PlayBook`对象。您还需要要求详细输出。可以按以下方式完成：

```
from ansible.playbook import PlayBook
from ansible import callbacks
VERBOSITY = 4
pb = PlayBook(
    playbook = 'webservers.yaml',
    host_list = 'hosts',
    stats = callbacks.AggregateStats(),
    callbacks = callbacks.PlaybookCallbacks(verbose=VERBOSITY),
    runner_callbacks = callbacks.PlaybookRunnerCallbacks(
                        callbacks.AggregateStats(),
                        verbose=VERBOSITY)
)

pb.run()
```

这将在 `hosts` 清单文件中指定的远程主机上执行 `webservers.yaml` playbook。

要在本地执行相同的 playbook，就像之前在 `runner` 对象中所做的那样，您需要在 `PlayBook` 对象中传递参数 `transport=local` 并删除 `host_list` 参数。

除了讨论过的参数，PlayBook 还接受更多。 

以下是 `PlayBook` 对象接受的所有参数列表，以及它们的目的：

| 参数 | 描述 |
| --- | --- |
| `playbook` | playbook 文件的路径 |
| `host_list` | 文件的路径，如 `/etc/ansible/hosts` |
| `module_path` | Ansible 模块的路径，如 `/usr/share/ansible/` |
| `forks` | 期望的并行级别 |
| `timeout` | 连接超时 |
| `remote_user` | 如果在特定 play 中未指定，则以此用户身份运行 |
| `remote_pass` | 使用此远程密码（对所有 play）而不是使用 SSH 密钥 |
| `sudo_pass` | 如果 `sudo=true` 并且需要密码，则为 sudo 密码 |
| `remote_port` | 如果在主机或 play 中未指定，默认的远程端口 |
| `transport` | 如何连接未指定传输方式的主机（本地，paramiko 等） |
| `callbacks` | playbook 的输出回调 |
| `runner_callbacks` | 更多的回调，这次是为 runner API |
| `stats` | 包含关于每个主机发生的事件的聚合数据 |
| `sudo` | 如果没有在每个 play 中指定，请求所有 play 使用 `sudo` 模式 |
| `inventory` | 可以指定而不是 `host_list` 来使用预先存在的清单对象 |
| `check` | 不要更改任何内容；只是尝试检测一些潜在的更改 |
| `any_errors_fatal` | 当其中一个主机失败时立即终止整个执行 |
| `force_handlers` | 即使任务失败，仍然通知并运行处理程序 |

## 回调

Ansible 提供了在主机上运行自定义回调的钩子，因为它调用各种模块。回调允许我们记录已启动或已完成的事件和操作，并从模块执行中聚合结果。Python API 提供了用于此目的的回调，可以在其默认状态下使用，也可以用来开发自己的回调插件。

回调允许执行各种操作。回调也可以被利用作为 Ansible 的扩展点。在 Python API 中包装 Ansible 时，一些最常用的回调操作是：

+   `AggregateStats`：顾名思义，`AggregateStats`保存了在剧本运行期间围绕每个主机活动的汇总统计信息。`AggregateStats`的对象可以作为`PlayBook`对象中`stats`的参数传递。

+   `PlaybookRunnerCallbacks`：`PlaybookRunnerCallbacks`的对象用于`Runner()`，例如，当使用`Runner` API 接口执行单个模块时，将使用`PlaybookRunnerCallbacks`返回任务状态。

+   `PlaybookCallbacks`：`PlaybookCallbacks`的对象由 Python API 的 playbook API 接口在从 Python API 执行 playbook 时使用。这些回调被`/usr/bin/ansible-playbook`使用。

+   `DefaultRunnerCallbacks`：当没有为`Runner`指定回调时，将使用`DefaultRunnerCallbacks`。

+   `CliRunnerCallbacks`：这扩展了`DefaultRunnerCallbacks`并覆盖了事件触发函数，基本上优化用于`/usr/bin/ansible`。

# Ansible 插件

插件是本书尚未涉及的另一个扩展点。此外，即使在互联网上，关于插件的文档也非常有限。

插件是下一章将涵盖的一个高级主题。然而，了解插件背后的 Python API 是很重要的，以便了解插件的工作原理以及如何扩展插件。

## PluginLoader

正如代码文档所述，`PluginLoader`是从配置的插件目录加载插件的基类。它遍历基于播放的目录列表、配置的路径和 Python 路径，以搜索插件。第一个匹配项被使用。

`PluginLoader`的对象接受以下参数：

+   `class_name`：插件类型的特定类名

+   `required_base_class`：插件模块所需的基类

+   `package`：包信息

+   `config`：指定配置的默认路径

+   `subdir`：包中的所有子目录

+   `aliases`：插件类型的替代名称

对于每个 Ansible 插件，都有一个定义的类名需要使用。在`PluginLoader`中，这个类由`required_base_class`标识。Ansible 插件的不同类别以及它们的基本名称列在下表中：

| 插件类型 | 类名 |
| --- | --- |
| 动作插件 | `ActionModule` |
| 缓存插件 | `CacheModule` |
| 回调插件 | `CallbackModule` |
| 连接插件 | `Connection` |
| Shell 插件 | `ShellModule` |
| 查找插件 | `LookupModule` |
| 变量插件 | `VarsModule` |
| 过滤器插件 | `FilterModule` |
| 测试插件 | `TestModule` |
| 策略插件 | `StrategyModule` |

# 总结

本章带您了解了 Ansible 的 Python API，并向您介绍了更高级的使用 Ansible 的方法。这包括执行单个任务而不创建整个 playbook，以及以编程方式执行 playbook。

本章还从技术角度向您介绍了 Ansible Python API 的各种组件，探索了各种扩展点和利用它们的方法。

本章还为下一章奠定了基础，下一章将深入探讨 Ansible 插件。下一章将利用本章所学知识来创建自定义的 Ansible 插件。我们将在接下来的章节中探索不同的 Ansible 插件，并指导您编写自己的 Ansible 插件。
