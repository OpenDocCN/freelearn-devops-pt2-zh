# 故障排除和测试策略

与任何其他类型的代码类似，Ansible 代码可能包含问题和错误。Ansible 尝试通过在执行任务之前检查任务语法来尽可能地使其安全。然而，这种检查只能防止一小部分可能的错误类型，如不正确的任务参数，但它不会保护您免受其他错误的影响。

重要的是要记住，由于其性质，在 Ansible 代码中，我们描述的是期望的状态，而不是陈述一系列步骤来实现期望的状态。这种差异意味着系统不太容易出现逻辑错误。

然而，Playbook 中的错误可能意味着所有机器上的潜在错误配置。这应该被非常认真对待。当系统的关键部分发生更改时，如 SSH 守护程序或`sudo`配置时，情况就更加严重，因为风险是您将自己锁在系统外。

有许多不同的方法可以防止或减轻 Ansible playbooks 中的错误。在本章中，我们将涵盖以下主题：

+   深入研究 playbook 执行问题

+   使用主机信息来诊断故障

+   使用 playbook 进行测试

+   使用检查模式

+   解决主机连接问题

+   通过 CLI 传递工作变量

+   限制主机的执行

+   刷新代码缓存

+   检查语法错误

# 技术要求

本章假定您已经按照《第一章》*开始使用 Ansible*中详细介绍的方式设置了控制主机，并且正在使用最新版本——本章的示例是使用 Ansible 2.9 进行测试的。尽管本章将给出特定的主机名示例，但您可以自由地用您自己的主机名和/或 IP 地址替换它们。如何做到这一点的详细信息将在适当的地方提供。

本章中的示例可以在本书的 GitHub 存储库中找到：[`github.com/PacktPublishing/Practical-Ansible-2/tree/master/Chapter%2011`](https://github.com/PacktPublishing/Practical-Ansible-2/tree/master/Chapter%2011)。

# 深入研究 playbook 执行问题

有时，Ansible 执行会中断。许多事情都可能导致这种情况。

在执行 Ansible playbooks 时，我发现的问题中最常见的原因是网络。由于发出命令的机器和执行命令的机器通常通过网络连接，网络中的问题会立即显示为 Ansible 执行问题。

有时，特别是对于某些模块，如`shell`或`command`，返回代码是非零的，即使执行是成功的。在这种情况下，您可以通过在模块中使用以下行来忽略错误：

```
ignore_errors: yes
```

例如，如果运行`/bin/false`命令，它将始终返回`1`。为了在 playbook 中执行这个命令，以避免它在那里阻塞，您可以编写类似以下的内容：

```
- name: Run a command that will return 1
 command: /bin/false ignore_errors: yes
```

正如我们所看到的，`/bin/false`将始终返回`1`作为返回代码，但我们仍然设法继续执行。请注意，这是一个特殊情况，通常，最好的方法是修复您的应用程序，以便遵循 UNIX 标准，并在应用程序适当运行时返回`0`，而不是在您的 Playbooks 中放置一个变通方法。

接下来，我们将更多地讨论我们可以使用的方法来诊断 Ansible 执行问题。

# 使用主机信息来诊断故障

一些执行失败是由目标机器的状态导致的。这种情况最常见的问题是 Ansible 期望文件或变量存在，但实际上并不存在。

有时，打印机器信息就足以找到问题。

为此，我们需要创建一个简单的 playbook，名为`print_facts.yaml`，其中包含以下内容：

```
--- - hosts: target_host
 tasks: - name: Display all variables/facts known for a host debug: var: hostvars[inventory_hostname]
```

这种技术将为您提供有关 Ansible 执行期间目标机器状态的大量信息。

# 使用 playbook 进行测试

在 IT 领域最复杂的事情之一不是创建软件和系统，而是在它们出现问题时进行调试。Ansible 也不例外。无论您在创建 Ansible playbook 方面有多么出色，迟早都会发现自己在调试一个行为不如您所想象的 playbook。

执行基本测试的最简单方法是在执行期间打印变量的值。让我们学习如何使用 Ansible 来做到这一点，如下所示：

1.  首先，我们需要一个名为`debug.yaml`的 playbook，其中包含以下内容：

```
---
- hosts: localhost
  tasks:
    - shell: /usr/bin/uptime
      register: result
    - debug:
        var: result
```

1.  使用以下命令运行它：

```
$ ansible-playbook debug.yaml
```

您将收到类似以下内容的输出：

```
PLAY [localhost] **********************************************************************************

TASK [Gathering Facts] ****************************************************************************
ok: [localhost]

TASK [shell] **************************************************************************************
changed: [localhost]

TASK [debug] **************************************************************************************
ok: [localhost] => {
    "result": {
        "changed": true,
        "cmd": "/usr/bin/uptime",
        "delta": "0:00:00.003461",
        "end": "2019-06-16 11:30:51.087322",
        "failed": false,
        "rc": 0,
        "start": "2019-06-16 11:30:51.083861",
        "stderr": "",
        "stderr_lines": [],
        "stdout": " 11:30:51 up 40 min, 1 user, load average: 1.11, 0.73, 0.53",
        "stdout_lines": [
            " 11:30:51 up 40 min, 1 user, load average: 1.11, 0.73, 0.53"
        ]
    }
}

PLAY RECAP ****************************************************************************************
localhost : ok=3 changed=1 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0 
```

在第一个任务中，我们使用`command`模块执行`uptime`命令，并将其输出保存在`result`变量中。然后，在第二个任务中，我们使用`debug`模块打印`result`变量的内容。

`debug`模块是允许您在 Ansible 执行期间打印变量的值（使用`var`选项）或固定字符串（使用`msg`选项）的模块。

`debug`模块还提供了`verbosity`选项。假设您以以下方式更改了 playbook：

```
---
- hosts: localhost
  tasks:
    - shell: /usr/bin/uptime
      register: result
    - debug:
       var: result
       verbosity: 2
```

现在，如果您尝试以与之前相同的方式执行它，您将注意到调试步骤不会被执行，并且输出中将出现以下行：

```
TASK [debug] **************************************************************************************
skipping: [localhost]
```

这是因为我们将所需的`verbosity`最小设置为`2`，默认情况下，Ansible 以`0`的`verbosity`运行。

要查看使用新 playbook 的 debug 模块的结果，我们需要运行一个稍微不同的命令：

```
$ ansible-playbook debug2.yaml -vv
```

通过在命令行中放置两个`-v`选项，我们将以`2`的`verbosity`运行 Ansible。这不仅会影响这个特定的模块，还会影响所有在不同调试级别下设置为以不同方式运行的模块（或 Ansible 本身）。

现在您已经学会了如何使用 playbook 进行测试，让我们学习如何使用检查模式。

# 使用检查模式

尽管您可能对自己编写的代码很有信心，但在真正的生产环境中运行之前进行测试仍然是值得的。在这种情况下，能够运行您的代码，但同时又有一个安全网是一个好主意。这就是检查模式的作用。请按照以下步骤进行操作：

1.  首先，我们需要创建一个简单的 playbook 来测试这个功能。让我们创建一个名为`check-mode.yaml`的 playbook，其中包含以下内容：

```
---
- hosts: localhost
  tasks:
    - name: Touch a file
      file:
        path: /tmp/myfile
        state: touch
```

1.  现在，我们可以通过在调用中指定`--check`选项来以检查模式运行 playbook：

```
$ ansible-playbook check-mode.yaml --check
```

这将输出一切，就好像它真的执行了操作，如下所示：

```
PLAY [localhost] **********************************************************************************

TASK [Gathering Facts] ****************************************************************************
ok: [localhost]

TASK [Touch a file] *******************************************************************************
ok: [localhost]

PLAY RECAP ****************************************************************************************
localhost : ok=2 changed=0 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0 
```

但是，如果您在`/tmp`中查找，您将找不到`myfile`。

Ansible 检查模式通常被称为干运行。其思想是运行不会改变机器的状态，只会突出当前状态与 playbook 中声明的状态之间的差异。

并非所有模块都支持检查模式，但所有主要模块都支持，而且每个版本都会添加更多的模块。特别要注意的是，`command`和`shell`模块不支持它，因为模块无法判断哪些命令会导致更改，哪些不会。因此，这些模块在检查模式之外运行时总是返回更改，因为它们假设已经进行了更改。

与检查模式类似的功能是`--diff`标志。这个标志允许我们跟踪在 Ansible 执行期间发生了什么变化。因此，假设我们使用以下命令运行相同的 playbook：

```
$ ansible-playbook check-mode.yaml --diff
```

这将返回类似以下的内容：

```
PLAY [localhost] **********************************************************************************

TASK [Gathering Facts] ****************************************************************************
ok: [localhost]

TASK [Touch a file] *******************************************************************************
--- before
+++ after
@@ -1,6 +1,6 @@
 {
- "atime": 1560693571.3594637,
- "mtime": 1560693571.3594637,
+ "atime": 1560693571.3620908,
+ "mtime": 1560693571.3620908,
 "path": "/tmp/myfile",
- "state": "absent"
+ "state": "touch"
 }

changed: [localhost]

PLAY RECAP ****************************************************************************************
localhost : ok=2 changed=1 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0 
```

正如您所看到的，输出显示`changed`，这意味着有些东西发生了改变（更具体地说，文件被创建了），在输出中，我们可以看到类似 diff 的输出，告诉我们状态从`absent`移动到`touch`，这意味着文件被创建了。`mtime`和`atime`也发生了变化，但这可能是由于文件的创建和检查方式。

现在您已经学会了如何使用检查模式，让我们学习如何解决主机连接问题。

# 解决主机连接问题

Ansible 通常用于管理远程主机或系统。为此，Ansible 需要能够连接到远程主机，只有在那之后才能发出命令。有时，问题在于 Ansible 无法连接到远程主机。这种问题的典型例子是当您尝试管理尚未启动的机器时。能够快速识别这类问题并及时解决将帮助您节省大量时间。

按照以下步骤开始：

1.  让我们创建一个名为`remote.yaml`的 playbook，其中包含以下内容：

```
---
- hosts: all
  tasks:
    - name: Touch a file
      file:
        path: /tmp/myfile
        state: touch
```

1.  我们可以尝试针对不存在的 FQDN 运行`remote.yaml` playbook，如下所示：

```
$ ansible-playbook -i host.example.com, remote.yaml
```

在这种情况下，输出将清楚地告诉我们，SSH 服务没有及时回复：

```
PLAY [all] ****************************************************************************************

TASK [Gathering Facts] ****************************************************************************
fatal: [host.example.com]: UNREACHABLE! => {"changed": false, "msg": "Failed to connect to the host via ssh: ssh: Could not resolve hostname host.example.com: Name or service not known", "unreachable": true}

PLAY RECAP ****************************************************************************************
host.example.com : ok=0 changed=0 unreachable=1 failed=0 skipped=0 rescued=0 ignored=0 
```

我们也有可能收到不同的错误：

```
PLAY [all] ****************************************************************************************

TASK [Gathering Facts] ****************************************************************************
fatal: [host.example.com]: UNREACHABLE! => {"changed": false, "msg": "Failed to connect to the host via ssh: fale@host.example.com: Permission denied (publickey,gssapi-keyex,gssapi-with-mic).", "unreachable": true}

PLAY RECAP ****************************************************************************************
host.example.com : ok=0 changed=0 unreachable=1 failed=0 skipped=0 rescued=0 ignored=0 
```

在这种情况下，主机确实有回复，但我们没有足够的访问权限来 SSH 进入它。

SSH 连接通常因以下两个原因而失败：

+   SSH 客户端无法与 SSH 服务器建立连接

+   SSH 服务器拒绝 SSH 客户端提供的凭据

由于 OpenSSH 非常高的稳定性和向后兼容性，当第一个问题发生时，很可能是 IP 地址或端口错误，因此 TCP 连接不可行。很少情况下，这种错误发生在 SSH 特定问题中。通常，仔细检查 IP 和主机名（如果是 DNS，请检查是否解析为正确的 IP）可以解决问题。要进一步调查这个问题，您可以尝试从同一台机器执行 SSH 连接，以检查是否存在问题。例如，我会这样做：

```
$ ssh host.example.com -vvv
```

我已经从错误本身中获取了主机名，以确保我正在模拟 Ansible 正在做的事情。我这样做是为了确保我能看到 SSH 能够给我排除问题的所有可能日志消息。

第二个问题可能会更复杂一些，因为它可能出现多种原因。其中之一是您试图连接到错误的主机，而您没有该机器的凭据。另一个常见情况是用户名错误。要调试它，您可以使用错误中显示的`user@host`地址（在我的情况下是`fale@host.example.com`），并使用您之前使用的相同命令：

```
$ ssh fale@host.example.com -vvv
```

这应该引发与 Ansible 报告给您的相同错误，但更详细。

现在您已经学会了如何解决主机连接问题，让我们学习如何通过 CLI 传递工作变量。

# 通过 CLI 传递工作变量

在调试过程中，有一件事可以帮助，对于代码的可重用性也有帮助，那就是通过命令行向 playbooks 传递变量。每当您的应用程序（无论是 Ansible playbook 还是任何类型的应用程序）从第三方（在这种情况下是人）接收输入时，它都应该确保该值是合理的。一个例子是检查变量是否已设置，因此不是空字符串。这是一个安全的黄金法则，但在用户受信任时也应该应用，因为用户可能会误输入变量名。应用程序应该识别这一点，并通过保护自身来保护整个系统。按照以下步骤：

1.  我们想要的第一件事是一个简单的 playbook，打印变量的内容。让我们创建一个名为`printvar.yaml`的 playbook，其中包含以下内容：

```
---
- hosts: localhost
  tasks:
    - debug:
       var: variable
```

1.  现在我们有了一个 Ansible playbook，可以让我们查看变量是否设置为我们期望的值，让我们在执行语句中声明`variable`并运行它：

```
$ ansible-playbook printvar.yaml --extra-vars='{"variable": "Hello, World!"}'
```

通过运行这个，我们将收到类似以下的输出：

```
PLAY [localhost] **********************************************************************************

TASK [Gathering Facts] ****************************************************************************
ok: [localhost]

TASK [debug] **************************************************************************************
ok: [localhost] => {
 "variable": "Hello, World!"
}

PLAY RECAP ****************************************************************************************
localhost : ok=2 changed=0 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0 
```

Ansible 允许以不同的模式和不同的优先级设置变量。更具体地说，您可以使用以下方式设置它们：

+   命令行值（最低优先级）

+   角色默认值

+   清单文件或脚本组`vars`

+   清单`group_vars/all`

+   Playbook `group_vars/all`

+   清单`group_vars/*`

+   Playbook `group_vars/*`

+   清单文件或脚本主机变量

+   清单`host_vars/*`

+   Playbook `host_vars/*`

+   主机 facts/缓存`set_facts`

+   Play `vars`

+   Play `vars_prompt`

+   Play `vars_files`

+   角色`vars`（在`role/vars/main.yml`中定义）

+   块`vars`（仅适用于块中的任务）

+   任务`vars`（仅适用于该任务）

+   `include_vars`

+   `set_facts`/registered vars

+   角色（和`include_role`）参数

+   `include`参数

+   额外变量（最高优先级）

如您所见，最后一个选项（也是所有选项中的最高优先级）是在执行命令中使用`--extra-vars`。

现在您已经学会了如何通过 CLI 传递工作变量，让我们学习如何限制主机的执行。

# 限制主机的执行

在测试 playbook 时，对一组受限制的机器进行测试可能是有意义的；例如，只有一个。让我们开始吧：

1.  为了使用 Ansible 的目标主机限制，我们需要一个 playbook。创建一个名为`helloworld.yaml`的 playbook，其中包含以下内容：

```
---
- hosts: all
  tasks:
    - debug:
        msg: "Hello, World!"
```

1.  我们还需要创建一个至少包含两个主机的清单。在我的情况下，我创建了一个名为`inventory`的文件，其中包含以下内容：

```
[hosts]
host1.example.com
host2.example.com
host3.example.com
```

让我们用以下命令以通常的方式运行 playbook：

```
$ ansible-playbook -i inventory helloworld.yaml
```

通过这样做，我们将收到以下输出：

```
PLAY [all] ****************************************************************************************

TASK [Gathering Facts] ****************************************************************************
ok: [host1.example.com]
ok: [host3.example.com]
ok: [host2.example.com]

TASK [debug] **************************************************************************************
ok: [host1.example.com] => {
 "msg": "Hello, World!"
}
ok: [host2.example.com] => {
 "msg": "Hello, World!"
}
ok: [host3.example.com] => {
 "msg": "Hello, World!"
}

PLAY RECAP ****************************************************************************************
host1.example.com : ok=2 changed=0 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0 
host2.example.com : ok=2 changed=0 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0 
host3.example.com : ok=2 changed=0 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0 
```

这意味着 playbook 在清单中的所有机器上执行。如果我们只想针对`host3.example.com`运行它，我们需要在命令行中指定如下：

```
$ ansible-playbook -i inventory helloworld.yaml --limit=host3.example.com
```

为了证明这个工作是按预期进行的，我们可以运行它。通过这样做，我们将收到以下输出：

```
PLAY [all] ****************************************************************************************

TASK [Gathering Facts] ****************************************************************************
ok: [host3.example.com]

TASK [debug] **************************************************************************************
ok: [host3.example.com] => {
 "msg": "Hello, World!"
}

PLAY RECAP ****************************************************************************************
host3.example.com : ok=2 changed=0 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0 
```

在 Ansible 执行我们在命令行中提到的 playbook 之前，它会分析清单以检测哪些目标在范围内，哪些不在。通过使用`--limit`关键字，我们可以强制 Ansible 忽略所有超出限制参数指定范围的主机。

可以指定多个主机作为列表或使用模式，因此以下两个命令都将针对`host2.example.com`和`host3.example.com`执行 playbook：

```
$ ansible-playbook -i inventory helloworld.yaml --limit=host2.example.com,host3.example.com

$ ansible-playbook -i inventory helloworld.yaml --limit=host[2-3].example.com
```

限制不会覆盖清单，而是对其添加限制。所以，假设我们限制到不在清单中的主机，如下所示：

```
$ ansible-playbook -i inventory helloworld.yaml --limit=host4.example.com
```

在这里，我们将收到以下错误，并且什么也不会发生：

```
 [WARNING]: Could not match supplied host pattern, ignoring: host4.example.com

ERROR! Specified hosts and/or --limit does not match any hosts
```

现在您已经学会了如何限制主机的执行，让我们学习如何刷新代码缓存。

# 刷新代码缓存

在 IT 中，缓存被用来加快操作的速度，Ansible 也不例外。

通常，缓存是好的，因此它们被广泛地使用。然而，如果它们缓存了不应该缓存的值，或者即使值已经改变但它们没有被刷新，它们可能会造成一些问题。

在 Ansible 中刷新缓存非常简单，只需运行`ansible-playbook`，我们已经在运行了，加上`--flush-cache`选项，如下所示：

```
ansible-playbook -i inventory helloworld.yaml --flush-cache
```

Ansible 使用 Redis 保存主机变量和执行变量。有时，这些变量可能会被遗留下来，影响后续的执行。当 Ansible 发现一个变量应该在它刚刚启动的步骤中设置时，Ansible 可能会假设该步骤已经完成，因此会将旧变量作为刚刚创建的变量。通过使用`--flush-cache`选项，我们可以避免这种情况，因为它会确保 Ansible 在执行过程中刷新 Redis 缓存。

现在您已经学会了如何刷新代码缓存，让我们学习如何检查错误的语法。

# 检查错误的语法

确定一个文件是否具有正确的语法对于机器来说是相当容易的，但对于人类来说可能更复杂。这并不意味着机器能够为您修复代码，但它们可以快速地识别问题是否存在。为了使用 Ansible 内置的语法检查器，我们需要一个具有语法错误的 playbook。让我们开始吧：

1.  让我们创建一个名为`syntaxcheck.yaml`的文件，其中包含以下内容：

```
---
- hosts: all
  tasks:
    - debug:
      msg: "Hello, World!"
```

1.  现在，我们可以使用`--syntax-check`命令：

```
$ ansible-playbook syntaxcheck.yaml --syntax-check
```

通过这样做，我们将收到以下输出：

```
ERROR! 'msg' is not a valid attribute for a Task

The error appears to be in '/home/fale/ansible/Ansible2Cookbook/Ch11/syntaxcheck.yaml': line 4, column 7, but may
be elsewhere in the file depending on the exact syntax problem.

The offending line appears to be:

 tasks:
 - debug:
 ^ here

This error can be suppressed as a warning using the "invalid_task_attribute_failed" configuration
```

1.  现在，我们可以继续修复第 4 行的缩进问题：

```
---
- hosts: all
  tasks:
    - debug:
        msg: "Hello, World!"
```

当我们重新检查语法时，我们会看到它现在不再返回错误：

```
$  ansible-playbook syntaxcheck-fixed.yaml --syntax-check

playbook: syntaxcheck.yaml
```

当语法检查没有发现任何错误时，输出将类似于先前的输出，其中列出了分析的文件而没有列出任何错误。

由于 Ansible 知道所有支持的模块中的所有支持选项，它可以快速读取您的代码并验证您提供的 YAML 是否包含所有必需的字段，并且不包含任何不受支持的字段。

# 总结

在本章中，您了解了 Ansible 提供的各种选项，以便您可以查找 Ansible 代码中的问题。更具体地说，您学会了如何使用主机事实来诊断故障，如何在 playbook 中包含测试，如何使用检查模式，如何解决主机连接问题，如何从 CLI 传递变量，如何将执行限制为主机的子集，如何刷新代码缓存以及如何检查错误语法。

在下一章中，您将学习如何开始使用 Ansible Tower。

# 问题

1.  True 或 False：调试模块允许您在 Ansible 执行期间打印变量的值或固定字符串。

A) True

B) False

1.  哪个关键字允许 Ansible 强制限制主机的执行？

A) `--limit`

B) `--max`

C) `--restrict`

D) `--force`

E) `--except`

# 进一步阅读

关于 Ansible 错误处理的官方文档可以在[`docs.ansible.com/ansible/latest/user_guide/playbooks_error_handling.html`](https://docs.ansible.com/ansible/latest/user_guide/playbooks_error_handling.html)找到。
