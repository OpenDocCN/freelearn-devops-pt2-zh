# 第二章：了解 Ansible 模块

Ansible 模块是可以使用 Ansible API 或通过 Ansible Playbook 调用的可重用代码片段。模块是 Ansible 的支柱。这些都是可以用任何语言编写的简单代码片段。

本章将介绍如何编写 Ansible 模块。本章分为四个部分：

+   编写你的第一个 Ansible 模块

+   模块编写助手

+   提供事实

+   测试和调试模块

# 编写你的第一个 Ansible 模块

Ansible 模块可以用任何语言编写，尽管在编写模块时需要遵守一些规定。具体如下：

+   模块必须只输出有效的 JSON。

+   模块应该是一个文件中自包含的，以便被 Ansible 自动传输。

+   尽量包含尽可能少的依赖项。如果存在依赖关系，请在模块文件顶部记录它们，并在导入失败时使模块引发 JSON 错误消息。

## 执行环境

要编写自己的 Ansible 模块，首先需要了解执行环境（即脚本将在何处以及如何执行）。

Ansible 在目标机器上执行脚本或播放。因此，您的脚本或已编译的二进制文件将被复制到目标机器上，然后执行。请注意，Ansible 只是简单地复制模块文件和生成的代码到目标机器上，并且不会尝试解决任何必要的依赖关系。因此，建议在您的 Ansible 模块文件中尽量包含尽可能少的依赖项。模块中的任何依赖关系都需要得到适当的记录，并在执行 Ansible 播放期间或之前进行处理。

## 步骤 1 - 模块放置

一旦您的模块文件准备好，您需要确切地知道应该将它放在哪里，以便在 Ansible 剧本中使用该模块。

您可以将您的模块放在 Ansible 寻找模块的不同位置：

+   在配置文件中由`library`变量指定的路径，位于`/etc/ansible/ansible.cfg`

+   由命令行中的`–module-path`参数指定的路径

+   在 Ansible 剧本的根目录下的`library`目录内

+   如果使用，位于角色的`library`目录内

## 编写基本的 Bash 模块

由于 Ansible 模块可以用任何语言编写，我们将首先尝试用 Bash 编写一个简单的 Ansible 模块。

我们将编写的第一个 Bash 模块将简单地检查目标机器的正常运行时间，并按照任何 Ansible 模块所需的格式返回输出。我们将命名该模块为`chkuptime`，并编写一个 playbook 来在目标机器上执行相同的模块。

该模块将被放置在 Ansible playbook 根目录中的`library`目录中，并将被 Ansible 自动包含。

以下是一个基本的 Bash 模块，用于检查目标机器的正常运行时间：

**Bash 模块代码**：(`library/chkuptime`)

```
#!/bin/bash

# The module checks for system uptime of the target machine.
# It returns a JSON output since an Ansible module should
# output a Valid JSON.

if [ -f "/proc/uptime" ]; then
    uptime=`cat /proc/uptime`
    uptime=${uptime%%.*}
    days=$(( uptime/60/60/24 ))
    hours=$(( uptime/60/60%24 ))
    uptime="$days days, $hours hours"
else
    uptime=""
fi

echo -e "{\"uptime\":\""$uptime"\"}"
```

为了让 Ansible 在执行 Ansible playbook 时包含上述模块代码，我们将其放在 Ansible playbook 根目录中的`library`目录中。

为了针对目标主机组运行这个模块，我们将创建一个名为`hosts`的清单文件，其中包括目标机器的分组列表。为了测试该模块，我们只对一个目标主机运行它。

现在，我们将创建一个 Ansible play 来执行新创建的模块。我们将 play 命名为`basic_uptime.yml`。

`basic_uptime.yml`

```
---
- hosts: remote
  user: rdas

  tasks:
    - name: Check uptime
      action: chkuptime
      register: uptime

    - debug: var=uptime
```

Playbook 的目录结构：

```
.
├── basic_uptime.yml
├── group_vars
├── hosts
├── library
│   └── chkuptime
└── roles
```

清单文件（`hosts`）

```
[remote]
192.168.122.191
```

现在，我们运行这个 play，它应该返回目标机器的正常运行时间：

```
**[rdas@localhost ]$ ansible-playbook -i hosts basic_uptime.yml**

**PLAY [remote] *******************************************************************

**GATHERING FACTS *****************************************************************
**ok: [192.168.122.191]**

**TASK: [Check uptime] ************************************************************
**ok: [192.168.122.191]**

**TASK: [debug var=uptime] ********************************************************
**ok: [192.168.122.191] => {**
 **"var": {**
 **"uptime": {**
 **"invocation": {**
 **"module_args": "",**
 **"module_name": "chkuptime"**
 **},**
 **"uptime": "0 days, 4 hours"**
 **}**
 **}**
**}**

**PLAY RECAP **********************************************************************
**192.168.122.191            : ok=3    changed=0    unreachable=0    failed=0** 

```

## 读取参数

如果你注意到上述模块，它不接受任何参数。让我们称这样的模块为`静态`模块。该模块在功能和行为上非常有限；否则，输出无法被改变。该模块将在目标机器上执行并返回固定的输出。如果用户期望以其他形式输出，这个模块就没有用了。该模块对用户没有灵活性。用户要想得到自己想要的输出，要么就得寻找这个模块的替代品（如果有的话），要么就得自己编写一个。

为了使模块更加灵活，它应该能够响应用户的要求，根据需要修改输出，或者至少提供用户可以与之交互的方式。这是通过允许模块接受参数来实现的。用户在运行时指定这些参数的值。

模块所期望的参数应该是明确定义的。参数也应该有良好的文档记录-既用于代码文档，也用于生成模块文档。参数类型和默认值（如果有）应该明确定义。

由于模块可以用任何语言编写，因此在代码级别上，Ansible 模块接受参数的方式可能不同。然而，无论模块是用什么语言编写的，从 Ansible playbook 传递参数的方式都保持不变。在 Bash 中，参数存储在按顺序编号的变量中。例如，第一个参数为$1，第二个参数为$2，依此类推。然而，参数类型和参数的默认值需要在代码中处理。Ansible 提供了一个 Python API，它提供了更好的处理参数的方式。它允许您明确定义参数的类型，强制要求参数，甚至为参数指定默认值。本章后面将介绍通过 Python API 处理参数。

我们将扩展上一个模块，以接受用户参数以更详细的格式打印系统运行时间。使用`detailed`标志，用户可以请求以完整的方式打印运行时间（即天，小时，分钟，秒），如果省略`detailed`标志，则保留先前的格式（即天，小时）。

以下是`chkuptime`模块的扩展，它根据用户指定的`detailed`标志返回输出：

**Bash 模块**：(`library/chkuptime`)

```
#!/bin/bash

# The module checks for system uptime of the target machine.
# The module takes in 'detailed' bool argument from the user
# It returns a JSON output since an Ansible module should
# output a Valid JSON.

source $1

if [ -f "/proc/uptime" ]; then
    uptime=`cat /proc/uptime`
    uptime=${uptime%%.*}
    days=$(( uptime/60/60/24 ))
    hours=$(( uptime/60/60%24 ))
    if [ $detailed ]; then
        minutes=$(( uptime/60%60 ))
        seconds=$(( uptime%60 ))
        uptime="$days days, $hours hours, $minutes minutes, $seconds seconds"
    else
        uptime="$days days, $hours hours"
    fi
else
    uptime=""
fi

echo -e "{\"uptime\":\""$uptime"\"}"
```

在 Ansible play 中唯一需要更改的是在调用模块时传递一个 Bool 类型的`detailed`参数。

Ansible Play `(uptime_arg.yml)`

```
---
- hosts: remote
  user: rdas

  tasks:
    - name: Check uptime
      action: chkuptime detailed=true
      register: uptime

- debug: var=uptime
```

执行 play 后，我们得到以下输出：

```
**[rdas@localhost bash-arg-example]$ ansible-playbook -i hosts uptime_arg.yml**
**PLAY [remote] *******************************************************************

**GATHERING FACTS *****************************************************************
**ok: [192.168.122.191]**

**TASK: [Check uptime] ************************************************************
**ok: [192.168.122.191]**

**TASK: [debug var=uptime] ********************************************************
**ok: [192.168.122.191] => {**
 **"var": {**
 **"uptime": {**
 **"invocation": {**
 **"module_args": "detailed=true",**
 **"module_name": "chkuptime"**
 **},**
 **"uptime": "1 days, 2 hours, 2 minutes, 53 seconds"**
 **}**
 **}**
**}**

**PLAY RECAP **********************************************************************
**192.168.122.191            : ok=3    changed=0    unreachable=0    failed=0** 

```

如果将输出与上一个 Ansible play 的输出进行比较，现在的运行时间包括了分钟和秒，而在上一个示例中是缺少的。通过设置`detailed`标志为 false，也可以使用新模块来实现先前的输出。

## 处理错误

您已经学会了如何创建自定义模块和读取用户输入。由于模块旨在在目标机器上执行某些功能，有时可能会失败。失败的原因可能是从目标机器上的权限问题到无效的用户输入，或其他任何原因。无论原因是什么，您的模块都应该能够处理错误和失败，并返回带有适当信息的错误消息，以便用户了解根本原因。所有失败都应该通过在返回数据中包含`failed`来明确报告。

例如，让我们创建一个简单的模块，接受用户输入的进程名称，并返回指定服务是否在目标机器上运行。如果服务正在运行，它将简单地返回一个包含请求进程的进程 ID 的消息。如果没有，它将通过将`failed`设置为`true`来明确失败模块执行。

以下是一个包含`failed`在返回数据中并明确失败模块执行的示例模块：

模块`library/chkprocess`

```
#!/bin/bash

# This module checks if the pid of the specified
# process exists. If not, it returns a failure msg

source $1

pid=`pidof $process`
if [[ -n $pid ]]; then
    printf '{
        "msg" : "%s is running with pid %s",
        "changed" : 1
    }' "$process" "$pid"
else
    printf '{
        "msg" : "%s process not running",
        "failed" : "True"
    }' "$process"
fi
```

**Ansible play** `chkprocess.yml`

```
---
- hosts: remote
  user: rdas

  tasks:
    - name: Check if process running
      action: chkprocess process=httpd
      register: process

    - debug: msg="{{ process.msg }}"
```

正如您所看到的，我们将检查指定的`httpd`进程是否在目标主机上运行。如果没有，这应该会导致 Ansible 运行失败。

现在让我们执行针对目标机器的 Ansible play：

```
**[rdas@localhost process-bash]$ ansible-playbook -i hosts chkprocess.yml**
**PLAY [remote] *******************************************************************

**GATHERING FACTS *****************************************************************
**ok: [192.168.122.191]**

**TASK: [Check if process running] ************************************************
**failed: [192.168.122.191] => {"failed": "True"}**
**msg: httpd process not running**

**FATAL: all hosts have already failed -- aborting**

**PLAY RECAP **********************************************************************
 **to retry, use: --limit @/home/rdas/chkprocess.retry**

**192.168.122.191            : ok=1    changed=0    unreachable=0    failed=1** 

```

正如您可能注意到的，由于`httpd`进程未在目标主机上运行，Ansible 按照请求失败了运行。此外，还显示了一个有意义的消息，以通知用户失败的根本原因。

# 在 Python 中创建 Ansible 模块

在这一点上，您已经熟悉了编写 Ansible 模块的基本概念。我们还讨论了一些用 Bash 编写的示例 Ansible 模块。

虽然可以用任何语言编写 Ansible 模块，但 Ansible 为用 Python 编写的模块提供了更友好的环境。

在不同语言中编写模块时，如上所述，处理参数、处理失败、检查输入等任务都是在模块代码中处理的。在 Python 中，Ansible 提供了一些辅助工具和语法糖来执行常见任务。例如，您不需要像在之前的示例中所示的那样解析参数。

Ansible 提供的常见例程能够处理返回状态、错误、失败和检查输入。这种语法糖来自 AnsibleModule 样板。使用 AnsibleModule 样板，您可以以更高效的方式处理参数和返回状态。这将帮助您更多地集中精力在模块上，而不必对输入进行明确的检查。

让我们更好地理解 AnsibleModule 样板。

## AnsibleModule 样板

为了从 AnsibleModule 样板中受益，您只需要导入`ansible.module_utils.basic`。

将导入放在文件末尾，并确保您的实际模块主体包含在传统的`main`函数中。

AnsibleModule 样板还为模块参数提供了一种规范语言。它允许您指定参数是可选的还是必需的。它还处理了一些数据类型，如枚举。

在下面的代码中，模块接受一个强制参数`username`，由设置`required=True`指定：

```
    module = AnsibleModule(
        argument_spec = dict(
            username = dict(required=True)
        )
    )   
    username = module.params.get('username')
    module.exit_json(changed=True, msg=str(status))
```

对象`module`使用一个常见的函数`exit_json`，它返回`true`并向 Ansible 返回一个成功消息。`module`对象提供了一组常见的函数，例如：

+   `run_command`：此函数运行外部命令并获取返回代码，`stdout`，`stderr`

+   `exit_json`：此函数向 Ansible 返回一个成功消息

+   `fail_json`：此函数返回一个失败和错误消息给 Ansible

参数可以通过`module.params`实例变量访问。每个参数都将有一个键值对。

`AnsibleModule`助手，在解析参数时，将执行一系列验证和请求类型转换。参数规范字典描述了模块的每个可能的参数。参数可以是可选的或必需的。可选参数可以有默认值。此外，可以使用`choice`关键字限制特定参数的可能输入。

# 模块文档

如果你正在编写一个模块，正确地记录它是非常重要的。文档是对模块功能更好理解所必需的。建议始终记录一个模块。

你所需要做的就是在你的模块文件中包含一个`DOCUMENTATION`全局变量，如下面的代码所示。该变量的内容应该是有效的 YAML。

```
DOCUMENTATION = """
---
module: chkuser
version_added: 0.1
short_description: Check if user exists on the target machine
options:
    username:
        decription:
            - Accept username from the user
        required: True
"""
```

可以使用`ansible-doc`命令阅读此文档。不幸的是，目前这仅适用于基于 Python 的模块。

除了详细说明每个选项的文档外，还可以提供一些可能涵盖模块的一些基本用例的示例。这可以通过添加另一个名为`EXAMPLES`的全局变量来完成。

```
EXAMPLES = """
#Usage Example
    - name: Check if user exists
      action: chkuser username=rdas
"""
```

让我们在一个检查目标机器上的用户是否存在的 Ansible 模块中实现 AnsibleModule 样板和上述文档。

以下是一个使用 AnsibleModule 样板构建的示例 Ansible 模块`chkuser`。该模块还包含模块文档以及使用示例：

**模块名称**：`chkuser`

```
#!/bin/python

DOCUMENTATION = """
---
module: chkuser
version_added: 0.1
short_description: Check if user exists on the target machine
options:
    username:
        decription:
            - Accept username from the user
        required: True
"""

EXAMPLES = """
#Usage Example
    - name: Check if user exists
      action: chkuser username=rdas
"""

def is_user_exists(username):
    try:
        import pwd
        return(username in [entry.pw_name for entry in pwd.getpwall()])
    except:
        module.fail_json(msg='Module pwd does not exists')

def main():
    module = AnsibleModule(
        argument_spec = dict(
            username = dict(required=True)
        )
    )   
    username = module.params.get('username')
    exists = is_user_exists(username)
    if exists:
        status = '%s user exists' % username
    else:
        status = '%s user does not exist' % username
    module.exit_json(changed=True, msg=str(status))

from ansible.module_utils.basic import *
main()
```

要使用这个模块，我们创建一个 Ansible play，将用户名作为参数传递给`chkuser`模块，如下面的代码所示：

**Ansible Play**：`chkuser.yml`

```
---
- hosts: remote
  user: rdas

  tasks:
    - name: Check if user exists
      action: chkuser username=rdas
      register: user

debug: msg="{{ user.msg }}
```

对目标机器执行 play 会返回一条消息，说明查询的用户是否存在于目标机器上。

# 测试和调试模块

编写模块很容易，但仅仅开发一个模块是不够的。您需要测试模块是否在所有情况下都能按预期执行所有操作。

第一次尝试很难成功。在工作时尝试一些东西是一种常见的技巧。这是为什么动态编程和具有短编辑和执行周期的编程环境变得非常流行的主要原因之一。

下一节，*快速本地执行*，涉及到在尽可能与您的 Ansible 环境隔离的情况下运行您的模块的问题。这在早期开发和调试过程中非常有帮助。

# 快速本地执行

在开发模块时，您可能希望缩短编辑/运行周期，并跳过实际通过 Ansible 执行模块的开销。正如我们在之前的 Bash 示例中看到的那样，执行环境非常简单，直接运行脚本直到正确是非常直接的。

然而，使用 AnsibleModule 样板的 Python 模块会变得更加棘手。

Ansible 在 Python 脚本的幕后进行了一些黑魔法，以便不需要在目标机器上安装 Ansible 组件。您可以通过采用两个简单的技巧来探索这种技术：

```
#!/bin/python

import sys

def main():
    module = AnsibleModule(
        argument_spec = dict()
    )
    f = open('/tmp/magicmirror', 'w')
    print >>f, file(sys.argv[0]).read()
    module.exit_json(changed=True)

from ansible.module_utils.basic import *
main()
```

在本地执行此模块将生成文件`/tmp/magicmirror`，其中包含已经通过对 Ansible 运行时进行了增强的代码。这使您能够从共享功能中受益，并避免在目标机器上引入依赖项。

另一种方法是在控制主机上设置环境变量`ANSIBLE_KEEP_REMOTE_FILES=1`，以防止 Ansible 清理远程机器，不删除生成的 Ansible 脚本，然后可以用于调试您的模块。

# 最佳实践

通过遵循一些最佳实践，模块始终可以在开发过程中得到改进。这有助于在需要时保持模块的卫生和易于理解和扩展。应该遵循的一些最佳实践包括：

+   模块必须是自包含的

+   将依赖项减少到最低限度

+   在`msg`键中写入错误原因

+   尽量只返回有用的输出

开发模块最重要的部分是不要忘记它应该返回有效的 JSON。

# 总结

在本章中，您学习了编写模块的基础知识。您了解了模块的放置位置以及在开发自定义模块时需要牢记的事项。您学习了如何在 Bash 中编写模块，并了解了 AnsibleModule 样板，因此最终开发了在 Bash 和 Python 中的示例 Ansible 模块。本章还涵盖了应该遵循的最佳实践。

在下一章中，您将了解错误处理，并通过一个真实场景，您可以创建一个 Ansible 模块并利用 Ansible 的强大功能。下一章还将涵盖一些复杂的数据结构与 Ansible。
