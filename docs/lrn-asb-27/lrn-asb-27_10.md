# 创建自定义模块

本章将重点介绍如何编写和测试自定义模块。我们已经讨论了模块的工作原理以及如何在任务中使用它们。为了快速回顾，Ansible 中的一个模块是每次运行 Ansible 任务时传输和执行到你的远程主机上的代码片段（如果你使用了`local_action`，它也可以在本地运行）。

从我的经验来看，每当需要将某个特定功能暴露为一流任务时，就会编写自定义模块。虽然可以通过现有模块执行相同的功能，但为了实现最终目标，可能需要一系列任务（有时还包括命令和 shell 模块）。例如，假设你想通过**预启动执行环境**（**PXE**）配置服务器。没有自定义模块，你可能会使用一些 shell 或命令任务来完成这个任务。然而，通过自定义模块，你只需将所需参数传递给它，业务逻辑将嵌入到自定义模块中，以执行 PXE 启动。这使你能够编写更简单易读的 playbook，并提供了更好的可重用性，因为你只需创建一次模块，就可以在角色和 playbook 中的任何地方使用它。

你传递给模块的参数（只要它们以**键值**格式提供）将与模块一起在一个单独的文件中转发。Ansible 期望你的模块输出至少有两个变量（即，模块运行的结果），无论它是通过还是失败的，以及用户的消息 - 它们都必须以 JSON 格式提供。如果你遵循这个简单的规则，你可以根据自己的需要定制你的模块！

在本章中，我们将涵盖以下主题：

+   Python 模块

+   Bash 模块

+   Ruby 模块

+   测试模块

# 先决条件

当你选择特定的技术或工具时，通常从它提供的内容开始。你慢慢地了解哲学，然后开始构建工具以及它帮助你解决的问题。然而，只有当你深入了解它的工作原理时，你才真正感到舒适和掌控。在某个阶段，为了充分利用工具的全部功能，你将不得不以适合你特定需求的方式定制它。随着时间的推移，那些提供了方便的方式来插入新功能的工具会留下来，而那些没有的则会从市场上消失。Ansible 也是类似的情况。Ansible playbook 中的所有任务都是某种类型的模块，并且它加载了数百个模块。你几乎可以找到任何你可能需要的模块。然而，总会有例外，这就是通过添加自定义模块来扩展 Ansible 功能的力量所在。

Chef 提供了**轻量级资源和提供者**（**LWRPs**）来执行此操作，而 Ansible 允许您使用自定义模块扩展其功能。使用 Ansible，您可以使用您选择的任何语言编写模块（前提是您有该语言的解释器），而在 Chef 中，模块必须是 Ruby 编写的。Ansible 开发人员建议对于任何复杂模块都使用 Python，因为有内置支持来解析参数；几乎所有的***nix**系统默认都安装了 Python，并且 Ansible 本身也是用 Python 编写的。在本章中，我们还将学习如何使用其他语言编写模块。

要使自定义模块可用于 Ansible，您可以执行以下操作之一：

+   在`ANSIBLE_LIBRARY`环境变量中指定自定义模块的路径。

+   使用`--module-path`命令行选项。

+   将模块放在您的 Ansible 顶层目录中的`library`目录中，并在`ansible.cfg`文件的`[default]`部分中添加`library=library`。

您可以从本书的 GitHub 存储库中下载所有文件，网址为[`github.com/PacktPublishing/Learning-Ansible-2.X-Third-Edition/tree/master/Chapter07`](https://github.com/PacktPublishing/Learning-Ansible-2.X-Third-Edition/tree/master/Chapter07)。

现在我们已经了解了这些背景信息，让我们来看一些代码吧！

# 使用 Python 编写模块

Ansible 允许用户使用任何语言编写模块。但是，使用 Python 编写模块具有自己的优势。您可以利用 Ansible 的库来缩短您的代码 - 这是其他语言编写的模块所不具备的优势。借助 Ansible 库的帮助，解析用户参数、处理错误并返回所需值变得更加容易。此外，由于 Ansible 是用 Python 编写的，因此您将在整个 Ansible 代码库中使用相同的语言，使审查更加容易，可维护性更高。我们将看到两个自定义 Python 模块的示例，一个使用 Ansible 库，一个不使用，以便让您了解自定义模块的工作原理。在创建模块之前，请确保按照前一节中提到的方式组织您的目录结构。第一个示例创建了一个名为`check_user`的模块。为此，我们需要在 Ansible 顶层目录中的`library`文件夹中创建`check_user.py`文件。完整的代码可以在 GitHub 上找到：

```
def main(): 
    # Parsing argument file 
    args = {} 
    args_file = sys.argv[1] 
    args_data = file(args_file).read() 
    arguments = shlex.split(args_data) 
    for arg in arguments: 
        if '=' in arg: 
            (key, value) = arg.split('=') 
            args[key] = value 
    user = args['user'] 

    # Check if user exists 
    try: 
        pwd.getpwnam(user) 
        success = True 
        ret_msg = 'User %s exists' % user 
    except KeyError: 
        success = False 
        ret_msg = 'User %s does not exists' % user 
...
```

先前的自定义模块`check_user`，将检查用户是否存在于主机上。该模块期望来自 Ansible 的用户参数。让我们分解前面的模块，看看它的作用。我们首先声明 Python 解释器，并导入解析参数所需的库：

```
#!/usr/bin/env python 

import pwd 
import sys 
import shlex 
import json 
```

使用`sys`库，然后解析由 Ansible 在文件中传递的参数。参数的格式为`param1=value1 param2=value2`，其中`param1`和`param2`是参数，`value1`和`value2`是参数的值。有多种方式可以拆分参数并创建字典，我们选择了一种简单的方式来执行操作。我们首先通过空格字符拆分参数创建参数列表，然后通过`=`字符拆分参数将键和值分开，并将其赋给 Python 字典。例如，如果您有一个字符串，如`user=foo gid=1000`，那么您将首先创建一个列表，`["user=foo", "gid=1000"]`，然后循环遍历该列表以创建字典。此字典将是`{"user": "foo", "gid": 1000}`；此操作使用以下行执行：

```
def main(): 
    # Parsing argument file 
    args = {} 
    args_file = sys.argv[1] 
    args_data = file(args_file).read() 
    arguments = shlex.split(args_data) 
    for arg in arguments: 
        if '=' in arg: 
            (key, value) = arg.split('=') 
            args[key] = value 
    user = args['user'] 
```

我们根据空格字符分隔参数，因为这是核心 Ansible 模块遵循的标准。您可以使用任何分隔符来代替空格，但我们建议您保持统一性。

一旦我们有了用户参数，我们就会检查该用户是否存在于主机上，具体如下：

```
    # Check if user exists 
    try: 
        pwd.getpwnam(user) 
        success = True 
        ret_msg = 'User %s exists' % user 
    except KeyError: 
        success = False 
        ret_msg = 'User %s does not exists' % user 
```

我们使用`pwd`库来检查用户的`passwd`文件。为简单起见，我们使用两个变量：一个用于存储成功或失败的消息，另一个用于存储用户的消息。最后，我们使用`try-catch`块中创建的变量来检查模块是否成功或失败：

```
    # Error handling and JSON return 
    if success: 
        print json.dumps({ 
            'msg': ret_msg 
        }) 
        sys.exit(0) 
    else: 
        print json.dumps({ 
            'failed': True, 
            'msg': ret_msg 
        }) 
        sys.exit(1) 
```

如果模块成功，则会以`0`的退出代码（`exit(0)`）退出执行；否则，它将以非零代码退出。Ansible 将查找失败的变量，如果它设置为`True`，则退出，除非您已明确要求 Ansible 使用`ignore_errors`参数忽略错误。您可以像使用 Ansible 的任何其他核心模块一样使用自定义模块。为了测试自定义模块，我们将需要一个 playbook，所以让我们使用以下代码创建`playbooks/check_user.yaml`文件：

```
---
- hosts: localhost
  connection: local
  vars:
    user_ok: root
    user_ko: this_user_does_not_exists
  tasks:
    - name: 'Check if user {{ user_ok }} exists'
      check_user:
        user: '{{ user_ok }}'
    - name: 'Check if user {{ user_ko }} exists'
      check_user:
        user: '{{ user_ko }}'
```

如您所见，我们像使用任何其他核心模块一样使用了`check_user`模块。Ansible 将通过将模块复制到远程主机并在单独的文件中使用参数执行该模块。让我们看看这个 playbook 如何运行，使用以下代码：

```
ansible-playbook playbooks/check_user.yaml
```

我们应该收到以下输出：

```
PLAY [localhost] ***************************************************

TASK [Gathering Facts] *********************************************
ok: [localhost]

TASK [Check if user root exists] ***********************************
ok: [localhost]

TASK [Check if user this_user_does_not_exists exists] **************
fatal: [localhost]: FAILED! => {"changed": false, "msg": "User this_user_does_not_exists does not exists"}
 to retry, use: --limit @playbooks/check_user.retry

PLAY RECAP *********************************************************
localhost                   : ok=2 changed=0 unreachable=0 failed=1
```

如预期，由于我们有`root`用户，但没有`this_user_does_not_exists`，它通过了第一个检查，但在第二个检查失败了。

Ansible 还提供了一个 Python 库来解析用户参数并处理错误和返回值。现在是时候探索 Ansible Python 库如何使您的代码更短、更快且更不容易出错了。为此，让我们创建一个名为`library/check_user_py2.py`的文件，其中包含以下代码。完整的代码可在 GitHub 上找到：

```
#!/usr/bin/env python 

import pwd 
from ansible.module_utils.basic import AnsibleModule 

def main(): 
    # Parsing argument file 
    module = AnsibleModule( 
        argument_spec = dict( 
            user = dict(required=True) 
        ) 
    ) 
    user = module.params.get('user') 

    # Check if user exists 
    try: 
        pwd.getpwnam(user) 
        success = True 
        ret_msg = 'User %s exists' % user 
    except KeyError: 
        success = False 
        ret_msg = 'User %s does not exists' % user 

...
```

让我们分解前面的模块，看看它是如何工作的，具体如下：

```
#!/usr/bin/env python 

import pwd 
from ansible.module_utils.basic import AnsibleModule 
```

如您所见，我们不再导入`sys`，`shlex`和`json`； 我们不再使用它们，因为所有需要它们的操作现在都由 Ansible 的`module_utils`模块完成：

```
    # Parsing argument file 
    module = AnsibleModule( 
        argument_spec = dict( 
            user = dict(required=True) 
        ) 
    ) 
    user = module.params.get('user') 
```

以前，我们对参数文件进行了大量处理，以获取最终的用户参数。 Ansible 通过提供一个`AnsibleModule`类来简化这个过程，该类自行处理所有处理并为我们提供最终参数。 `required=True`参数意味着该参数是必需的，如果未传递该参数，则执行将失败。 默认值`required`为`False`，这将允许用户跳过该参数。 然后，您可以通过在`module.params`字典上调用`module.params`上的`get`方法来访问参数的值。 远程主机上检查用户的逻辑将保持不变，但错误处理和返回方面将如下更改：

```
    # Error handling and JSON return 
    if success: 
        module.exit_json(msg=ret_msg) 
    else: 
        module.fail_json(msg=ret_msg) 
```

使用`AnsibleModule`对象的一个优点是你有一个非常好的设施来处理返回值到 playbook。 我们将在下一节中更深入地讨论。

我们本可以将检查用户和返回方面的逻辑压缩在一起，但我们将它们分开以提高可读性。

要验证一切是否按预期工作，我们可以在`playbooks/check_user_py2.yaml`中创建一个新的 playbook，并使用以下代码：

```
---
- hosts: localhost
  connection: local
  vars:
    user_ok: root
    user_ko: this_user_does_not_exists
  tasks:
    - name: 'Check if user {{ user_ok }} exists'
      check_user_py2:
        user: '{{ user_ok }}'
    - name: 'Check if user {{ user_ko }} exists'
      check_user_py2:
        user: '{{ user_ko }}'
```

你可以使用以下代码来运行它：

```
ansible-playbook playbooks/check_user_py2.yaml
```

然后，我们应该收到以下输出：

```
PLAY [localhost] ***************************************************

TASK [Gathering Facts] *********************************************
ok: [localhost]

TASK [Check if user root exists] ***********************************
ok: [localhost]

TASK [Check if user this_user_does_not_exists exists] **************
fatal: [localhost]: FAILED! => {"changed": false, "msg": "User this_user_does_not_exists does not exists"}
 to retry, use: --limit @playbooks/check_user_py2.retry

PLAY RECAP *********************************************************
localhost                   : ok=2 changed=0 unreachable=0 failed=1
```

这个输出与我们的预期一致。

# 使用 exit_json 和 fail_json

Ansible 通过`exit_json`和`fail_json`方法提供了更快和更短的处理成功和失败的方法，分别。 您可以直接将消息传递给这些方法，Ansible 将处理剩余的部分。 您还可以将其他变量传递给这些方法，并且 Ansible 将这些变量打印到`stdout`。 例如，除了消息之外，您可能还想打印用户的`uid`和`gid`参数。 您可以通过将这些变量分隔符传递给`exit_json`方法来实现这一点。

让我们看看如何将多个值返回到`stdout`，如下面的代码所示，放置在`library/check_user_id.py`中。 完整的代码在 GitHub 上可用：

```
#!/usr/bin/env python 

import pwd 
from ansible.module_utils.basic import AnsibleModule 

class CheckUser: 
    def __init__(self, user): 
        self.user = user 

    # Check if user exists 
    def check_user(self): 
        uid = '' 
        gid = '' 
        try: 
            user = pwd.getpwnam(self.user) 
            success = True 
            ret_msg = 'User %s exists' % self.user 
            uid = user.pw_uid 
            gid = user.pw_gid 
        except KeyError: 
            success = False 
            ret_msg = 'User %s does not exists' % self.user 
        return success, ret_msg, uid, gid 

...
```

正如你所见，我们返回了用户的`uid`和`gid`参数，以及消息`msg`。 你可以有多个值，Ansible 将以字典格式打印所有这些值。 创建一个包含以下内容的 playbook：`playbooks/check_user_id.yaml`：

```
---
- hosts: localhost
  connection: local
  vars:
    user: root
  tasks:
    - name: 'Retrive {{ user }} data if it exists'
      check_user_id:
        user: '{{ user }}'
      register: user_data
    - name: 'Print user {{ user }} data'
      debug:
        msg: '{{ user_data }}'
```

你可以使用以下代码来运行它：

```
ansible-playbook playbooks/check_user_id.yaml
```

我们应该收到以下输出：

```
PLAY [localhost] ***************************************************

TASK [Gathering Facts] *********************************************
ok: [localhost] 
TASK [Retrive root data if it exists] ******************************
ok: [localhost]

TASK [Print user root data] ****************************************
ok: [localhost] => {
 "msg": {
 "changed": false, 
 "failed": false, 
 "gid": 0, 
 "msg": "User root exists", 
 "uid": 0
 }
}

PLAY RECAP *********************************************************
localhost : ok=3 changed=0 unreachable=0 failed=0
```

在这里，我们完成了两种方法的工作，这反过来又帮助我们找到了在 Ansible 中处理成功和失败的更快方式，同时向用户传递参数。

# 测试 Python 模块

正如你所见，你可以通过创建非常简单的 playbooks 来测试你的模块。 为此，我们需要克隆 Ansible 官方仓库（如果你还没有这样做）：

```
git clone git://github.com/ansible/ansible.git --recursive
```

接下来，像下面这样来源一个环境文件：

```
source ansible/hacking/env-setup
```

现在我们可以使用`test-module`工具通过将文件名作为命令行参数来运行脚本：

```
ansible/hacking/test-module -m library/check_user_id.py -a "user=root"
```

结果将类似于以下输出：

```
* including generated source, if any, saving to: /home/fale/.ansible_module_generated 
* ansiballz module detected; extracted module source to: /home/fale/debug_dir 
*********************************** 
RAW OUTPUT 

{"msg": "User root exists", "invocation": {"module_args": {"user": "root"}}, "gid": 0, "uid": 0, "changed": false} 

*********************************** 
PARSED OUTPUT 
{ 
    "changed": false, 
    "gid": 0, 
    "invocation": { 
        "module_args": { 
            "user": "root" 
        } 
    }, 
    "msg": "User root exists", 
    "uid": 0 
}
```

如果你没有使用`AnsibleModule`，直接执行脚本也很容易。这是因为该模块需要很多 Ansible 特定的变量，所以"模拟" Ansible 运行比实际运行 Ansible 本身更复杂。

# 使用 bash 模块

Ansible 中的 Bash 模块与任何其他 bash 脚本没有任何区别，唯一不同之处在于它们将数据打印在`stdout`上。Bash 模块可以是非常简单的，比如检查远程主机上是否有进程正在运行，也可以是运行一些更复杂的命令。

如前所述，一般推荐使用 Python 编写模块。在我看来，第二选择（仅适用于非常简单的模块）是`bash` 模块，因为它简单易用，用户基础广泛。

让我们创建一个名为`library/kill_java.sh`的文件，并包含以下内容：

```
#!/bin/bash 
source $1 

SERVICE=$service_name 

JAVA_PIDS=$(/usr/java/default/bin/jps | grep ${SERVICE} | awk '{print $1}') 

if [ ${JAVA_PIDS} ]; then 
    for JAVA_PID in ${JAVA_PIDS}; do 
        /usr/bin/kill -9 ${JAVA_PID} 
    done 
    echo "failed=False msg=\"Killed all the orphaned processes for ${SERVICE}\"" 
    exit 0 
else 
    echo "failed=False msg=\"No orphaned processes to kill for ${SERVICE}\"" 
    exit 0 
fi
```

前述的`bash`模块将使用`service_name`参数并强制终止所有属于该服务的 Java 进程。正如你所知，Ansible 将参数文件传递给模块。然后我们使用`$1` source 来来源参数文件。这将实际上设置一个名为`service_name`的环境变量。然后我们使用`$service_name`来访问这个变量，如下所示：

```
source $1 

SERVICE=$service_name 
```

然后我们检查我们是否得到了该服务的 `PIDS`，并遍历这些 `PIDS` 来强制终止所有与 `service_name` 匹配的 Java 进程。一旦它们被终止，我们通过`failed=False`退出模块，并且附上一个退出码为`0`的消息，如下所示：

```
if [ ${JAVA_PIDS} ]; then 
    for JAVA_PID in ${JAVA_PIDS}; do 
        /usr/bin/kill -9 ${JAVA_PID} 
    done 
    echo "failed=False msg=\"Killed all the orphaned processes for ${SERVICE}\"" 
    exit 0 
```

如果我们没有找到该服务的正在运行的进程，我们仍将以退出码`0`退出模块，因为终止 Ansible 运行可能没有意义：

```
else 
    echo "failed=False msg=\"No orphaned processes to kill for ${SERVICE}\"" 
    exit 0 
fi 
```

你也可以通过打印`failed=True`并将退出码设置为`1`来终止 Ansible 运行。

如果语言本身不支持 JSON，则 Ansible 允许返回键值输出。这使得 Ansible 更加友好于开发人员和系统管理员，并允许以您选择的任何语言编写自定义模块。让我们通过将参数文件传递给模块来测试`bash`模块。现在我们可以在`/tmp/arguments`中创建一个参数文件，将`service_name`参数设置为`jenkins`，如下所示：

```
service_name=jenkins
```

现在，你可以像运行任何其他 bash 脚本一样运行模块。让我们看看当我们用以下代码运行时会发生什么：

```
bash library/kill_java.sh /tmp/arguments
```

我们应该收到以下输出：

```
failed=False msg="No orphaned processes to kill for jenkins"
```

如预期，即使本地主机上没有运行 Jenkins 进程，模块也没有失败。

如果你收到 `jps command does not exists` 错误而不是上述输出，那么你的机器可能缺少 Java。如果是这样，你可以按照操作系统的说明安装它：[`www.java.com/en/download/help/download_options.xml`](https://www.java.com/en/download/help/download_options.xml)。

# 使用 Ruby 模块

在 Ruby 中编写模块与在 Python 或 bash 中编写模块一样简单。您只需要注意参数、错误、返回语句，当然还需要了解基本的 Ruby！让我们创建`library/rsync.rb`文件，其中包含以下代码。完整代码可在 GitHub 上找到：

```
#!/usr/bin/env ruby 

require 'rsync' 
require 'json' 

src = '' 
dest = '' 
ret_msg = '' 
SUCCESS = '' 

def print_message(state, msg, key='failed') 
    message = { 
        key => state, 
        "msg" => msg 
    } 
    print message.to_json 
    exit 1 if state == false 
    exit 0 
...
```

在前述模块中，我们首先处理用户参数，然后使用`rsync`库复制文件，最后返回输出。

要使用这个功能，您需要确保系统上存在用于 Ruby 的`rsync`库。为此，您可以执行以下命令：

```
gem install rsync
```

让我们逐步分解上述代码，看看它是如何工作的。

我们首先编写一个名为`print_message`的方法，该方法将以 JSON 格式打印输出。通过这样做，我们可以在多个地方重用相同的代码。请记住，如果要使 Ansible 运行失败，则您的模块的输出应包含`failed=true`；否则，Ansible 将认为模块成功并将继续进行下一个任务。获得的输出如下所示：

```
#!/usr/bin/env ruby 

require 'rsync' 
require 'json' 

src = '' 
dest = '' 
ret_msg = '' 
SUCCESS = '' 

def print_message(state, msg, key='failed') 
    message = { 
        key => state, 
        "msg" => msg 
    } 
    print message.to_json 
    exit 1 if state == false 
    exit 0 
end
```

然后我们处理参数文件，其中包含由空格字符分隔的键值对。这类似于我们之前使用 Python 模块时所做的，我们负责解析参数。我们还执行一些检查，以确保用户没有漏掉任何必需的参数。在这种情况下，我们检查是否已指定了`src`和`dest`参数，并在未提供参数时打印一条消息。进一步的检查可能包括参数的格式和类型。您可以添加这些检查和您认为重要的任何其他检查。例如，如果您的参数之一是`date`参数，则需要验证输入是否确实是正确的日期。考虑以下代码片段，其中显示了讨论过的参数：

```
args_file = ARGV[0] 
data = File.read(args_file) 
arguments = data.split(" ") 
arguments.each do |argument| 
    print_message(false, "Argument should be name-value pairs. Example name=foo") if not argument.include?("=") 
    field, value = argument.split("=") 
    if field == "src" 
        src = value 
    elsif field == "dest" 
        dest = value 
    else print_message(false, "Invalid argument provided. Valid arguments are src and dest.") 
    end 
end 
```

一旦我们拥有了必需的参数，我们将使用`rsync`库复制文件，如下所示：

```
result = Rsync.run("#{src}", "#{dest}") 
if result.success? 
    success = true 
    ret_msg = "Copied file successfully" 
else 
    success = false 
    ret_msg = result.error 
end 
```

最后，我们检查`rsync`任务是通过还是失败，然后调用`print_message`函数将输出打印在`stdout`上，如下所示：

```
if success 
    print_message(false, "#{ret_msg}") 
else 
    print_message(true, "#{ret_msg}") 
end
```

您可以通过简单地将参数文件传递给模块来测试您的 Ruby 模块。为此，我们可以创建`/tmp/arguments`文件，并包含以下内容：

```
src=/etc/resolv.conf dest=/tmp/resolv_backup.conf
```

现在让我们运行模块，如下所示：

```
ruby library/rsync.rb /tmp/arguments
```

我们将收到以下输出：

```
{"failed":false,"msg":"Copied file successfully"} 
```

我们将留下`serverspec`测试由您完成。

# 测试模块

由于对其目的和其为业务带来的好处的理解不足，测试经常被低估。测试模块与测试 Ansible playbook 的任何其他部分一样重要，因为模块中的微小更改可能会破坏整个 playbook。我们将以本章的 *使用 Python 编写模块* 部分中编写的 Python 模块为例，并使用 Python 的 `nose` 测试框架编写一个集成测试。虽然也鼓励进行单元测试，但对于我们的场景，即检查远程用户是否存在，集成测试更有意义。

`nose` 是一个 Python 测试框架；您可以在 [`nose.readthedocs.org/en/latest/`](https://nose.readthedocs.org/en/latest/) 上找到有关该测试框架的更多信息。

为了对模块进行测试，我们将先前的模块转换为一个 Python 类，以便我们可以直接将该类导入到我们的测试中，并仅运行模块的主要逻辑。以下代码显示了重组的 `library/check_user_py3.py` 模块，该模块将检查远程主机上是否存在用户。完整代码可在 GitHub 上找到：

```
#!/usr/bin/env python 

import pwd 
from ansible.module_utils.basic import AnsibleModule 

class User: 
    def __init__(self, user): 
        self.user = user 

    # Check if user exists 
    def check_if_user_exists(self): 
        try: 
            user = pwd.getpwnam(self.user) 
            success = True 
            ret_msg = 'User %s exists' % self.user 
        except KeyError: 
            success = False 
            ret_msg = 'User %s does not exists' % self.user 
        return success, ret_msg 

 ...
```

正如您在上述代码中所见，我们创建了一个名为 `User` 的类。我们实例化了该类并调用了 `check_if_user_exists` 方法来检查用户是否实际存在于远程计算机上。现在是时候编写集成测试了。我们假设您已经在系统上安装了 `nose` 包。如果没有，不用担心！您仍然可以使用以下命令安装该包：

```
pip install nose
```

现在让我们在 `library/test_check_user_py3.py` 中编写集成测试文件，如下所示：

```
from nose.tools import assert_equals, assert_false, assert_true 
import imp 
imp.load_source("check_user","check_user_py3.py") 
from check_user import User 

def test_check_user_positive(): 
    chkusr = User("root") 
    success, ret_msg = chkusr.check_if_user_exists() 
    assert_true(success) 
    assert_equals('User root exists', ret_msg) 

def test_check_user_negative(): 
    chkusr = User("this_user_does_not_exists") 
    success, ret_msg = chkusr.check_if_user_exists() 
    assert_false(success) 
    assert_equals('User this_user_does_not_exists does not exists', ret_msg) 
```

在上述集成测试中，我们导入了 `nose` 包和我们的 `check_user` 模块。我们通过传递我们想要检查的用户来调用 `User` 类。然后，我们通过调用 `check_if_user_exists()` 方法来检查用户是否存在于远程主机上。`nose` 方法 - `assert_true`、`assert_false` 和 `assert_equals` - 可以用于比较预期值与实际值。只有当 `assert` 方法通过时，测试也会通过。您可以通过具有以 `test_` 开头的多个方法来在同一个文件中拥有多个测试；例如，`test_check_user_positive()` 和 `test_check_user_negative()` 方法。`nose` 测试将获取所有以 `test_` 开头的方法并执行它们。

正如您所见，我们实际上为一个函数创建了两个测试。这是测试的一个关键部分。始终尝试您知道会成功的情况，但也不要忘记测试您期望失败的情况。

现在我们可以通过运行以下代码使用 `nose` 来测试它是否正常工作：

```
cd library
nosetests -v test_check_users_py3.py
```

您应该会收到类似以下代码块的输出：

```
test_check_user_py3.test_check_user_positive ... ok test_check_user_py3.test_check_user_negative ... ok --------------------------------------------------- Ran 2 tests in 0.001sOK
```

正如您所见，测试通过了，因为主机上存在 `root` 用户，而 `this_user_does_not_exists` 用户不存在。

我们使用 `nose` 测试的 `-v` 选项来启用 **详细模式**。

对于更复杂的模块，我们建议您编写单元测试和集成测试。您可能会想知道为什么我们没有使用 `serverspec` 来测试模块。

我们仍然建议将 `serverspec` 测试作为 playbook 的功能测试的一部分进行运行；然而，对于单元测试和集成测试，建议使用知名框架。同样，如果您编写了 Ruby 模块，我们建议您使用诸如 `rspec` 等框架为其编写测试。如果您的自定义 Ansible 模块具有多个参数和多个组合，则您将编写更多的测试来测试每个场景。最后，我们建议您将所有这些测试作为您的 CI 系统的一部分运行，无论是 Jenkins、Travis 还是其他任何系统。

# 摘要

到此，我们结束了这一小而重要的章节，重点介绍了如何通过编写自定义模块来扩展 Ansible。您学会了如何使用 Python、bash 和 Ruby 来编写自己的模块。我们还学习了如何为模块编写集成测试，以便将其集成到您的 CI 系统中。希望未来通过使用模块来扩展 Ansible 的功能将会更加容易！

在下一章中，我们将进入供应、部署和编排的世界，看看当我们为环境中的各种实例提供新的实例或者想要将软件更新部署到各种实例时，Ansible 如何解决我们的基础设施问题。我们承诺这段旅程将会很有趣！
