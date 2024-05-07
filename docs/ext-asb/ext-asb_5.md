# 第五章。深入了解 Ansible 插件

前一章向您介绍了 Python API 以及 Ansible 提供的各种扩展点。当您到达本章时，您应该已经知道 Ansible 如何加载插件。前一章列出了不同类型的 Ansible 插件。

本章深入探讨了 Ansible 插件是什么，以及如何编写自定义的 Ansible 插件。在本章中，我们将详细讨论不同类型的 Ansible 插件，并在代码级别上进行探索。一起，我们将通过 Ansible Python API，使用扩展点来编写自己的 Ansible 插件。

如前一章所述，插件按以下方式分类：

+   查找插件

+   动作插件

+   缓存插件

+   回调插件

+   连接插件

+   变量插件

+   过滤器插件

这些插件中，最常用的是查找插件、回调插件、变量插件、过滤器插件和连接插件。让我们逐个探索这些插件。

# 查找插件

查找插件旨在从不同来源读取数据并将其提供给 Ansible。数据源可以是控制节点上的本地文件系统，也可以是外部数据源。这些数据源也可以是 Ansible 本身不原生支持的文件格式。

如果您决定编写自己的查找插件，您需要将其放入以下一个目录中，以便 Ansible 在执行 Ansible playbook 时捡起它。

+   项目`Root`中名为`lookup_plugins`的目录

+   在`~/.ansible/plugins/lookup_plugins/`或

+   /usr/share/ansible_plugins/lookup_plugins/

默认情况下，Ansible 中已经有许多查找插件可用。让我们讨论一些最常用的查找插件。

## 查找插件文件

这是 Ansible 中最基本的查找插件类型。它通过控制节点上的文件内容进行读取。然后可以将从文件中读取的数据传递给 Ansible playbook 作为变量。在其最基本形式中，文件查找的使用方法在以下 Ansible playbook 中进行了演示：

```
---
- hosts: all
  vars:
    data: "{{ lookup('file', './test-file.txt') }}"
  tasks:
- debug: msg="File contents {{ data }}"
```

上述 playbook 将从 playbook 根目录中的本地文件`test-file.txt`中读取数据到变量`data`中。然后，这个变量被传递给`task: debug`模块，并使用数据变量将其打印在屏幕上。

## 查找插件 - csvfile

`csvfile`查找插件设计用来从控制节点上的 CSV 文件中读取数据。这个查找模块设计为接受几个参数，如下所述：

| 参数 | 默认值 | 描述 |
| --- | --- | --- |
| `file` | `ansible.csv` | 要从中读取数据的文件。 |
| `delimiter` | TAB | CSV 文件中使用的分隔符。通常是'`,`'。 |
| `col` | `1` | 列号（索引）。 |
| `default` | 空字符串 | 如果在 CSV 文件中找不到请求的键，则返回此值 |

让我们以读取以下 CSV 文件中的数据为例。CSV 文件包含不同城市的人口和面积详情：

```
File: city-data.csv
City, Area, Population
Pune, 700, 2.5 Million
Bangalore, 741, 4.3 Million
Mumbai, 603, 12 Million
```

这个文件位于 Ansible play 的控制节点的根目录下。要从这个文件中读取数据，使用了`csvfile`查找插件。以下的 Ansible play 尝试从之前的 CSV 文件中读取孟买的人口。

**Ansible Play**：`test-csv.yaml`

```
---
- hosts: all 
  tasks:
    - debug:
        msg="Population of Mumbai is {{lookup('csvfile', 'Mumbai file=city-data.csv delimiter=, col=2')}}"
```

## 查找插件 - dig

`dig`查找插件可以用来对**FQDN**（**完全合格域名**）运行 DNS 查询。您可以通过使用插件支持的不同标志来自定义查找插件的输出。在其最基本的形式中，它返回给定 FQDN 的 IP。

这个插件依赖于`python-dns`包。这应该安装在控制节点上。

以下的 Ansible play 解释了如何获取任何 FQDN 的 TXT 记录：

```
---
- hosts: all 
  tasks:
    - debug: msg="TXT record {{ lookup('dig', 'yahoo.com./TXT') }}" 
    - debug: msg="IP of yahoo.com {{lookup('dig', 'yahoo.com', wantlist=True)}}"
```

之前的 Ansible play 将在第一步获取 TXT 记录，并在第二步获取与 FQDN `yahoo.com`相关联的任何 IP。

还可以使用 dig 插件执行反向 DNS 查找，方法是使用以下语法：

```
**- debug: msg="Reverse DNS for 8.8.8.8 is {{ lookup('dig', '8.8.8.8/PTR') }}"**

```

## 查找插件 - ini

`ini`查找插件设计用来读取`.ini`文件中的数据。一般来说，`ini`文件是在定义的部分下的键-值对的集合。`ini`查找插件支持以下参数：

| 参数 | 默认值 | 描述 |
| --- | --- | --- |
| `type` | `ini` | 文件类型。目前支持两种格式 - ini 和 property。 |
| `file` | `ansible.ini` | 要读取数据的文件名。 |
| `section` | `global` | 需要从`ini`文件中读取指定键的部分。 |
| `re` | `False` | 如果键是正则表达式，将其设置为`true`。 |
| `default` | 空字符串 | 如果在`ini`文件中找不到请求的键，则返回此值。 |

以以下`ini`文件为例，让我们尝试使用`ini`查找插件读取一些键。文件名为`network.ini`：

```
[default]
bind_host = 0.0.0.0
bind_port = 9696
log_dir = /var/log/network

[plugins]
core_plugin = rdas-net
firewall = yes 
```

以下 Ansible play 将从`ini`文件中读取键：

```
---
- hosts: all
  tasks:
      - debug: msg="core plugin {{ lookup('ini', 'core_plugin file=network.ini section=plugins') }}"
      - debug: msg="core plugin {{ lookup('ini', 'bind_port file=network.ini section=default') }}"
```

`ini`查找插件也可以用于通过不包含部分的文件读取值，例如 Java 属性文件。

# 循环-迭代的查找插件

有时您可能需要一遍又一遍地执行相同的任务。这可能是安装软件包的各种依赖项的情况，或者多个输入经过相同的操作，例如检查和启动各种服务。就像任何其他编程语言提供了迭代数据以执行重复任务的方法一样，Ansible 也提供了一种清晰的方法来执行相同的操作。这个概念被称为循环，并由 Ansible 查找插件提供。

Ansible 中的循环通常被标识为以`with_`开头的循环。Ansible 支持许多循环选项。以下部分讨论了一些最常用的循环选项。

## 标准循环-with_items

这是 Ansible 中最简单和最常用的循环。它用于对项目列表进行迭代并对其执行某些操作。以下 Ansible play 演示了使用`with_items`查找循环的用法：

```
---
- hosts: all 
  tasks:
    - name: Install packages
      yum: name={{ item }} state=present
      with_items:
        - vim
        - wget
        - ipython
```

`with_items`循环支持使用哈希，您可以在 Ansible playbook 中使用项目`<keyname>`来访问变量。以下 playbook 演示了使用`with_items`来迭代给定哈希：

```
---
- hosts: all 
  tasks:
    - name: Create directories with specific permissions
      file: path={{item.dir}} state=directory mode={{item.mode | int}}
      with_items:
        - { dir: '/tmp/ansible', mode: 755 }
        - { dir: '/tmp/rdas', mode: 755 }
```

前面的 playbook 将创建两个具有指定权限集的目录。如果您仔细查看从`item`中访问`mode`键时，存在一个名为`int`的代码块。这是一个`jinja2`过滤器，用于将字符串转换为整数。

## 直到循环-until

这个循环与任何其他编程语言的实现相同。它至少执行一次，并且除非达到特定条件，否则会继续执行。

让我们看一下以下代码，以了解`do-until`循环：

```
- **name**: Clean up old file. Keep only the latest 5 
  **action**: shell /tmp/clean-files.sh 
  **register**: number 
  **until**: number.stdout.find('5') != -1 
  **retries**: 6 
  **delay**: 10
```

`clean-files.sh`脚本对指定目录执行清理操作，并仅保留最新的五个文件。在每次执行时，它会删除最旧的文件，并在清理的目录中返回剩余文件的数量作为`stdout`的输出。脚本看起来像这样：

```
#!/bin/bash 

**DIR_CLEAN**='/tmp/test' 
cd $DIR_CLEAN 
**OFNAME**=`ls -t | tail -1` 
rm -f $OFNAME 
**COUNT**=`ls | wc -w` 
echo $COUNT
```

此操作将重试最多六次，间隔为 10。一旦在数字寄存器变量中找到 5，循环就会结束。

如果未明确指定“重试”和“延迟”，在这种情况下，默认情况下任务将重试三次，间隔五次。

## 创建自己的查找插件

上一章介绍了 Python API，并解释了 Ansible 如何加载各种插件以用于 Ansible play。本章涵盖了一些已经可用的 Ansible 查找插件，并解释了如何使用这些插件。本节将尝试复制`dig`查找的功能，以获取给定 FQDN 的 IP 地址。这将在不使用`dnspython`库的情况下完成，并将使用 Python 的基本 socket 库。以下示例仅演示了如何编写自己的 Ansible 查找插件：

```
import socket

class LookupModule(object):

    def __init__(self, basedir=None, **kwargs):
        self.basedir = basedir

    def run(self, hostname, inject=None, **kwargs):
        hostname = str(hostname)
        try:
            host_detail = socket.gethostbyname(hostname)
        except:
            host_detail = 'Invalid Hostname'
        return host_detail
```

上述代码是一个查找插件；让我们称之为`hostip`。

如您所见，存在一个名为`LookupModule`的类。只有当存在一个名为`LookupModule`的类时，Ansible 才将 Python 文件或模块识别为查找插件。该模块接受一个名为 hostname 的参数，并检查是否存在与之对应的 IP（即，是否可以解析为有效的 IP 地址）。如果是，则返回请求的 FQDN 的 IP 地址。如果不是，则返回“无效的主机名”。

要使用此模块，请将其放置在 Ansible play 根目录下的`lookup_plugins`目录中。以下 playbook 演示了如何使用新创建的`hostip`查找：

```
---

- hosts: all 
  tasks:
    - debug:
        msg="{{lookup('hostip', item, wantlist=True)}}"
      with_items:
        - www.google.co.in
        - saliux.wordpress.com
        - www.twitter.com
```

上述 play 将循环遍历网站列表，并将其作为参数传递给`hostip`查找插件。这将返回与请求的域关联的 IP。您可能已经注意到，还有一个名为`wantlist=True`的参数在调用`hostip`查找插件时也被传递进去。这是为了处理多个输出（即，如果有多个值与请求的域关联，这些值将作为列表返回）。这样可以轻松地对输出值进行迭代。

# 回调插件

回调是 Ansible 中最广泛使用的插件之一。它们允许您在运行时对 Ansible 运行的事件做出响应。回调是一种最常定制的插件类型。

尽管有一些通用的回调插件，但您肯定会自己编写一个来满足您的需求。这是因为每个人对数据想要做什么有不同的看法。Ansible 不仅仅是一个限于配置管理和编排的工具。您可以做更多的事情，例如，在 Ansible play 期间收集数据并稍后处理它们。回调提供了一个广阔的可能性探索空间。这一切取决于您对结果的期望。

这一部分不是通过现有的回调模块，而是更专注于编写一个。

从前面的章节中，以一个场景为例，您创建了自己的`dmidecode`模块，该模块在目标机器上执行并返回硬件规格的 JSON。该模块还支持一个标志，允许您将此结果存储在目标机器上的 JSON 文件中。

看看这种情况，有两个主要问题：

+   您没有 playbook 执行的日志。一切都在`stdout`上。

+   即使在调用`dmidecode`模块时将保存标志设置为 true，结果也会存储在目标机器上，而不是控制节点上。在 playbook 执行后，您将不得不从每个目标主机单独收集这些 JSON 文件。

第一点是您在生产环境中绝对不希望出现的问题。您总是希望有 Ansible play 的日志。这将使您能够在 playbook 执行期间追溯任何失败。Ansible 代码存储库中已经有一些通用的回调插件可供此目的使用。您可以在[`github.com/ansible/ansible/tree/devel/lib/ansible/plugins/callback`](https://github.com/ansible/ansible/tree/devel/lib/ansible/plugins/callback)找到一些现有的回调模块。如果它们满足您的需求，您可以选择其中之一。本节将不讨论现有的回调模块。

第二点是人们选择开发自己的回调插件的一个主要原因。它解决了你实际想要对数据做什么的问题。在这种情况下，该模块收集系统信息，以备后续审计之用。在其他情况下，您可能仍希望处理收集到的信息和 Ansible play 的日志，以确定失败的原因，生成报告，跟踪生产变更等。可能有许多可能性。

本节将通过创建一个自定义回调插件来解决第二点，该插件可以帮助您从目标机器获取 JSON 数据，该数据是使用您在第三章中创建的`dmidecode`模块生成的。*深入了解 Ansible 模块*。

在深入编写回调模块之前，了解回调模块的工作原理非常重要。

回调模块在播本执行期间发生的事件上起作用。Ansible 支持的常用事件包括：

+   `runner_on_failed`

+   `runner_on_ok`

+   `runner_on_skipped`

+   `runner_on_unreachable`

+   `runner_on_no_hosts`

+   `playbook_on_start`

以`runner_`开头的事件名称特定于任务。以`playbook_`开头的事件名称特定于整个播本。显然，事件名称是不言自明的；因此，我们不会详细介绍每个事件的含义。

如前一章所述，回调插件应该有一个名为`CallbackModule`的类，否则 Ansible 将无法识别它作为回调插件。Python API 要求`CallbackModule`类来识别模块作为回调插件。这是为了区分不同的 Python 文件，因为不同的 Python 模块可能驻留在同一目录中，回调插件可能正在使用同一目录中的一个 Python 模块的方法。

在讨论了事件和类的要求之后，现在是时候动手了。让我们继续编写一个非常基本的回调插件，与第三章中创建的`dmidecode`模块，*深入了解 Ansible 模块*进行集成。

如果你还记得，Ansible 播本将 JSON 输出记录在名为`dmi_data`的寄存器中。然后通过调试模块在`stdout`上回显这些数据。因此，在播本执行期间，回调模块需要查找`dmi_data`键。该键将包含输出的 JSON 数据。回调插件将尝试在控制节点上将这些 JSON 数据转储到一个 JSON 文件中，并将其命名为目标机器的 IP 或 FQDN，后跟`.json`扩展名。回调模块名为`logvar`，需要放置在 Ansible 播本根目录下的`callback_plugins`目录中。

```
import json

class CallbackModule(object):

    ''' 
    This logs the debug variable 'var' and writes it in a JSON file
    '''

    def runner_on_ok(self, host, result):
        try:
            if result['var']['dmi_data[\'msg\']']:
                fname = '%s.json' % host
                with open(fname, 'w') as ofile:
                    json.dump(result['var']['dmi_data[\'msg\']'], ofile)
        except:
            pass
```

在将上述模块放置在 Ansible play 根目录中的`callback_plugins`目录中后，执行`dmidecode` playbook 将导致输出文件命名为`<taget>.json`。这些文件包含由`dmidecode`模块返回的目标机器的`dmidecode`信息。

# Var 插件

在编写 Ansible play 时，您肯定会使用一些变量。它可能是特定于主机的`host_vars`或常用的`group_vars`。从这些变量中读取的任何数据并馈送到 Ansible playbook 都是使用 var 插件完成的。

var 插件由类名`VarModule`标识。如果您在代码级别上探索 var 插件，在类内部有三种方法：

+   `run`：此方法应返回特定于主机的变量以及从其成员组计算出的变量

+   `get_host_vars`：返回特定于主机的变量

+   `get_group_vars`：返回特定于组的变量

# 连接插件

连接插件定义了 Ansible 如何连接到远程机器。Ansible 可以通过定义 playbooks 来在各种平台上执行操作。因此，对于不同的平台，您可能需要使用不同的连接插件。

默认情况下，Ansible 附带`paramiko_ssh`，本机 SSH 和本地连接插件。还添加了对 docker 的支持。还有其他不太知名，不太常用的连接插件，如 chroot，jail zone 和 libvirt。

连接插件由其类连接标识。

让我们在代码级别上探索 Paramiko 连接插件。连接类包含四种主要方法。这些方法又调用了一些私有函数来进行一些操作。主要方法是：

+   `exec_command`：此方法在远程目标上运行请求的命令。您可能需要使用`sudo`运行命令的要求，默认情况下需要一个 PTY。Paramiko 通过默认传递`pty=True`来处理这个问题。

+   `put_file`：此方法接受两个参数 - `in_path`和`out_path`。此函数用于从本地控制器节点复制文件到远程目标机器。

+   `fetch_file`：这种方法类似于`put_file`方法，也接受两个参数 - `in_path`和`out_path`。该方法用于从远程机器获取文件到本地控制器节点。

+   `Close`：此函数在操作完成时终止连接。

# 过滤器插件

Ansible 支持 Jinja2 模板化，但为什么不使用 Jinja2 过滤器？您想要它；Ansible 有它！

过滤器插件是 Jinja2 模板过滤器，可用于将模板表达式从一种形式修改或转换为另一种形式。Ansible 已经默认提供了一组 Jinja2 过滤器。例如，`to_yaml`和`to_json`。Ansible 还支持从已格式化的文本中读取数据。例如，如果您已经有一个需要从中读取数据的 YAML 文件或 JSON 文件，您可以使用`from_json`或`from_yaml`过滤器。

您还可以选择使用`int`过滤器将字符串转换为整数，就像在*循环-迭代的查找插件*部分中创建具有定义权限的目录时所示的那样。

让我们讨论如何以及在哪里可以实现过滤器，以便更充分地利用 Ansible。

## 使用带条件的过滤器

在运行脚本时，可能会出现一种情况，根据上一步的结果，您需要执行特定的步骤。这就是条件出现的地方。在正常编程中，您可以使用`if-else`条件语句。在 Ansible 中，您需要检查上一条命令的输出，并在`when`子句中应用过滤器，如下面的代码所示：

```
---
- hosts: all 
  tasks:
    - name: Run the shell script
      shell: /tmp/test.sh
      register: output

    - name: Print status
    - debug: msg="Success"
      when: output|success

    - name: Print status
    - debug: msg="Failed"
      when: output|failed
```

在上述脚本中，shell 脚本`test.sh`的执行结果存储在寄存器变量 output 中。如果状态是成功，任务将打印`Success`；否则，将打印`Failed`。

## 版本比较

此过滤器可用于检查目标主机上安装的请求应用程序的版本。它返回`True`或`False`状态。版本比较过滤器接受以下运算符：

```
<, lt, <=, le, >, gt, >=, ge, ==, =, eq, !=, <>, ne
```

## IP 地址过滤器

IP 地址过滤器可用于检查提供的字符串是否为有效的 IP 地址。甚至可以指定您要检查的协议：IPv4 或 IPv6。

以下过滤器将检查 IP 地址是否为有效的 IPv4 地址：

```
{{ host_ip | ipv4 }}
```

同样，可以通过使用以下方式来检查 IP 地址是否为有效的 IPv6 地址：

```
{{ host_ip | ipv6 }}
```

## 理解代码

Ansible 通过查找名为`FilterModule`的类来识别 Python 模块为过滤器插件。在这个类内部存在一个名为`filters`的方法，它将过滤器映射到它们在`FilterModule`类之外的对应文件中。

如果您选择自己编写过滤器插件，以下是过滤器插件的结构：

```
# Import dependencies

def custom_filter(**kwargs):
    # filter operation code

class FilterModule(object):
    def filter(self):
        return {
            'custom_filter': custom_filter
        }   
```

在前面的示例代码中，在`FilterModule`类内部的过滤器方法中，`custom_filter`键映射到类外部的`custom_filter`函数。

`custom_filter`函数包含实际的过滤器实现代码。Ansible 只是加载`FilterModule`类并浏览定义的过滤器。然后将定义的过滤器提供给最终用户使用。

在 Ansible 代码库中，任何关于过滤器的新建议通常都会添加到过滤器插件内部的`core.py`文件中。

# 摘要

本章继续了第四章*探索 API*的内容，并介绍了 Ansible 插件的 Python API 在各种 Ansible 插件中的实现方式。在本章中，我们详细讨论了各种类型的插件，从实现角度和代码层面进行了讨论。本章还演示了如何通过编写自定义查找和回调插件来编写示例插件。现在，您应该能够为 Ansible 编写自己的自定义插件。

下一章将探讨如何配置 Ansible，并将到目前为止讨论的所有内容整合在一起。本章还将指导您如何共享您的插件和角色，并探索一些最佳实践。
