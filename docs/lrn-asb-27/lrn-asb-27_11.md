# 调试和错误处理

像软件代码一样，测试基础架构代码是一项非常重要的任务。在生产环境中最好没有未经测试的代码浮现，尤其是当您有严格的客户 SLA 需要满足时，即使对于基础架构也是如此。在本章中，我们将看到语法检查、在不将代码应用于机器上进行测试（no-op 模式）以及用于 playbook 的功能测试，playbook 是 Ansible 的核心，触发您想在远程主机上执行的各种任务。建议您将其中一些集成到您为 Ansible 设置的**持续集成**（**CI**）系统中，以更好地测试您的 playbook。我们将看到以下要点：

+   语法检查

+   带有和不带有`--diff`的检查模式

+   功能测试

作为功能测试的一部分，我们将关注以下内容：

+   对系统最终状态的断言

+   使用标签进行测试

+   使用`--syntax-check`选项

+   使用`ANSIBLE_KEEP_REMOTE_FILES`和`ANSIBLE_DEBUG`标志

然后，我们将看看如何处理异常以及如何自愿触发错误。

# 技术要求

对于本章，除了常规要求外，没有特定要求，例如 Ansible、Vagrant 和一个 shell。 

您可以从本书的 GitHub 存储库中下载所有文件，网址为[`github.com/PacktPublishing/Learning-Ansible-2.X-Third-Edition/tree/master/Chapter08`](https://github.com/PacktPublishing/Learning-Ansible-2.X-Third-Edition/tree/master/Chapter08)。

# 语法检查

每次运行 playbook 时，Ansible 首先检查 playbook 文件的语法。如果遇到错误，Ansible 会报告语法错误，并且不会继续，除非您修复了该错误。仅当运行`ansible-playbook`命令时才执行此语法检查。在编写大型 playbook 或包含任务文件时，可能很难修复所有错误；这可能会浪费更多时间。为了应对这种情况，Ansible 提供了一种方式，让您随着 playbook 的编写进度来检查您的 YAML 语法。对于此示例，我们将需要创建名为`playbooks/setup_apache.yaml`的文件，并包含以下内容：

```
---
- hosts: all
  tasks: 
    - name: Install Apache 
      yum: 
        name: httpd 
        state: present 
      become: True
    - name: Enable Apache 
    service: 
        name: httpd 
        state: started 
        enabled: True 
      become: True
```

现在我们有了示例文件，我们需要用`--syntax-check`参数运行它；因此，您需要按照以下方式调用 Ansible：

```
ansible-playbook playbooks/setup_apache.yaml --syntax-check
```

`ansible-playbook`命令检查了`setup_apache.yml` playbook 的 YAML 语法，并显示了 playbook 的语法是正确的。让我们看看 playbook 中无效语法导致的结果错误：

```
ERROR! Syntax Error while loading YAML.
  did not find expected '-' indicator

The error appears to have been in '/home/fale/Learning-Ansible-2.X-Third-Edition/Ch8/playbooks/setup_apache.yaml': line 10, column 5, but may
be elsewhere in the file depending on the exact syntax problem.

The offending line appears to be:

    - name: Enable Apache
    service:
    ^ here
```

错误显示`Enable Apache`任务中存在缩进错误。Ansible 还会提供错误的行号、列号和发现错误的文件名（即使这并不是确切错误位置的保证）。这肯定应该是您为 Ansible 的 CI 运行的基本测试之一。

# 检查模式

检查模式（也称为**dry-run**或**no-op 模式**）将以无操作模式运行你的 playbook - 也就是说，它不会将任何更改应用于远程主机; 相反，它只会显示运行任务时将引入的更改。检查模式是否实际启用取决于每个模块。有一些命令可能会引起你的兴趣。所有这些命令都必须在`/usr/lib/python2.7/site-packages/ansible/modules`或你的 Ansible 模块文件夹所在的位置运行（根据你使用的操作系统以及你安装 Ansible 的方式，可能存在不同的路径）。

要计算安装的可用模块数量，你可以执行此命令：

```
find . -type f | grep '.py$' | grep -v '__init__' | wc -l 
```

使用 Ansible 2.7.2，此命令的结果是`2095`，因为 Ansible 有那么多模块。

如果你想看看有多少支持检查模式的模块，你可以运行以下代码：

```
grep -r 'supports_check_mode=True' | awk -F: '{print $1}' | sort | uniq | wc -l 
```

使用 Ansible 2.7.2，此命令的结果是`1239`。

你可能还会发现以下命令对于列出支持检查模式的所有模块很有用：

```
grep -r 'supports_check_mode=True' | awk -F: '{print $1}' | sort | uniq 
```

这有助于你测试 playbook 的行为方式，并在将其运行在生产服务器之前检查是否可能存在任何故障。你只需简单地在`ansible-playbook`命令中传递`--check`选项来运行检查模式下的 playbook。让我们看看检查模式如何与`setup_apache.yml` playbook 一起运行，运行以下代码：

```
ansible-playbook --check -i ws01, playbooks/setup_apache.yaml
```

结果将如下所示：

```
PLAY [all] ***********************************************************

TASK [Gathering Facts] ***********************************************
ok: [ws01]

TASK [Install Apache] ************************************************
changed: [ws01]

TASK [Enable Apache] *************************************************
changed: [ws01]

PLAY RECAP ***********************************************************
ws01                          : ok=3 changed=2 unreachable=0 failed=0
```

在上述运行中，Ansible 不会在目标主机上进行更改，而是会突出显示在实际运行期间将发生的所有更改。从上述运行中，你可以发现`httpd`服务已经安装在目标主机上。因此，Ansible 对该任务的退出消息是 OK：

```
TASK [Install Apache] ************************************************
changed: [ws01]
```

但是，对于第二个任务，它发现`httpd`服务在目标主机上未运行：

```
TASK [Enable Apache] *************************************************
changed: [ws01]
```

当你再次运行上述的 playbook 时，如果没有启用检查模式，Ansible 会确保服务状态正在运行。

# 使用--diff 指示文件之间的差异

在检查模式下，你可以使用`--diff`选项来显示将应用于文件的更改。为了能够看到`--diff`选项的使用，我们需要创建一个`playbooks/setup_and_config_apache.yaml` playbook，以匹配以下内容：

```
- hosts: all
  tasks: 
    - name: Install Apache 
      yum: 
        name: httpd 
        state: present 
      become: True
    - name: Enable Apache 
      service: 
        name: httpd 
        state: started 
        enabled: True 
      become: True
    - name: Ensure Apache userdirs are properly configured
      template:
        src: ../templates/userdir.conf
        dest: /etc/httpd/conf.d/userdir.conf
      become: True
```

正如你所看到的，我们添加了一个任务，将确保`/etc/httpd/conf.d/userdir.conf`文件处于特定状态。

我们还需要创建一个模板文件，放置在`templates/userdir.conf`中，并包含以下内容（完整文件可在 GitHub 上找到）：

```
#
# UserDir: The name of the directory that is appended onto a user's home
# directory if a ~user request is received.
#
# The path to the end user account 'public_html' directory must be
# accessible to the webserver userid. This usually means that ~userid
# must have permissions of 711, ~userid/public_html must have permissions
# of 755, and documents contained therein must be world-readable.
# Otherwise, the client will only receive a "403 Forbidden" message.
#
<IfModule mod_userdir.c>
    #
    # UserDir is disabled by default since it can confirm the presence
    # of a username on the system (depending on home directory
    # permissions).
    #
    UserDir enabled

  ...

```

在此模板中，我们仅更改了`UserDir enabled`行，默认情况下为`UserDir disabled`。

`--diff`选项与`file`模块不兼容；你必须仅使用`template`模块。

现在我们可以使用以下命令测试此结果：

```
ansible-playbook -i ws01, playbooks/setup_and_config_apache.yaml --diff --check 
```

正如你所看到的，我们正在使用`--check`参数，确保这将是一个干运行。我们将收到以下输出：

```
PLAY [all] ***********************************************************

TASK [Gathering Facts] ***********************************************
ok: [ws01]

TASK [Install Apache] ************************************************
ok: [ws01]

TASK [Enable Apache] *************************************************
ok: [ws01]

TASK [Ensure Apache userdirs are properly configured] ****************
--- before: /etc/httpd/conf.d/userdir.conf
+++ after: /home/fale/.ansible/tmp/ansible-local-6756FTSbL0/tmpx9WVXs/userdir.conf
@@ -14,7 +14,7 @@
 # of a username on the system (depending on home directory
 # permissions).
 #
- UserDir disabled
+ UserDir enabled

 #
 # To enable requests to /~user/ to serve the user's public_html

changed: [ws01]

PLAY RECAP ***********************************************************
ws01                          : ok=4 changed=1 unreachable=0 failed=0 
```

我们可以看到，Ansible 比较远程主机的当前文件与源文件；以 "+" 开头的行表示向文件中添加了一行，而 "-" 表示删除了一行。

你也可以使用 `--diff` 选项而不用 `--check`，这将允许 Ansible 进行指定的更改并显示两个文件之间的差异。

在 CI 测试的一部分中，将 `--diff` 和 `--check` 模式一起使用作为测试步骤，可以断言有多少步骤在运行过程中发生了变化。另一个可以同时使用这些功能的情况是部署过程的一部分，用于检查运行 Ansible 时会发生什么变化。

有时候会出现这样的情况——尽管不应该出现，但有时候确实会出现——你在一台机器上很长一段时间没有运行 playbook，而你担心再次运行会破坏某些东西。使用这些选项可以帮助你了解到这只是你的担忧，还是一个真正的风险。

# Ansible 中的功能测试

维基百科称功能测试是一种**质量保证**（**QA**）过程和一种基于所测试软件组件的规格的黑盒测试。功能测试通过提供输入并检查输出来测试函数；内部程序结构很少被考虑。在基础设施方面，在代码方面同样重要。

从基础设施的角度来看，就功能测试而言，我们在实际机器上测试 Ansible 运行的输出。Ansible 提供了多种执行 playbook 的功能测试的方式，让我们来看看其中一些最常用的方法。

# 使用 `assert` 进行功能测试

仅当你想要检查任务是否会在主机上改变任何内容时，`check` 模式才会起作用。当你想检查你的模块的输出是否符合预期时，这并没有帮助。例如，假设你编写了一个模块来检查端口是开启还是关闭。为了测试这一点，你可能需要检查你的模块的输出，看它是否与期望的输出匹配。为了执行这样的测试，Ansible 提供了一种将模块的输出与期望的输出直接进行比较的方法。

让我们通过创建 `playbooks/assert_ls.yaml` 文件，并使用以下内容来查看它是如何工作的：

```
---
- hosts: all
  tasks: 
    - name: List files in /tmp 
      command: ls /tmp 
      register: list_files 
    - name: Check if file testfile.txt exists 
      assert: 
        that: 
          - "'testfile.txt' in list_files.stdout_lines" 
```

在上述 playbook 中，我们在目标主机上运行 `ls` 命令，并将该命令的输出记录在 `list_files` 变量中。接下来，我们要求 Ansible 检查 `ls` 命令的输出是否具有期望的结果。我们使用 `assert` 模块来实现这一点，该模块使用某些条件检查来验证任务的 `stdout` 值是否符合用户的预期输出。让我们运行上述 playbook，看看 Ansible 返回什么输出，使用以下命令：

```
ansible-playbook -i ws01, playbooks/assert_ls.yaml
```

由于我们没有这个文件，所以我们会收到以下输出：

```
PLAY [all] ***********************************************************

TASK [Gathering Facts] ***********************************************
ok: [ws01]

TASK [List files in /tmp] ********************************************
changed: [ws01]

TASK [Check if file testfile.txt exists] *****************************
fatal: [ws01]: FAILED! => {
 "assertion": "'testfile.txt' in list_files.stdout_lines", 
 "changed": false, 
 "evaluated_to": false, 
 "msg": "Assertion failed"
}
 to retry, use: --limit @/home/fale/Learning-Ansible-2.X-Third-Edition/Ch8/playbooks/assert_ls.retry

PLAY RECAP ***********************************************************
ws01                          : ok=2 changed=1 unreachable=0 failed=1 
```

如果我们在创建预期的文件之后重新运行 playbook，这将是结果：

```
PLAY [all] ***********************************************************

TASK [Gathering Facts] ***********************************************
ok: [ws01]

TASK [List files in /tmp] ********************************************
changed: [ws01]

TASK [Check if file testfile.txt exists] *****************************
ok: [ws01] => {
 "changed": false, 
 "msg": "All assertions passed"
}

PLAY RECAP ***********************************************************
ws01                          : ok=3 changed=1 unreachable=0 failed=0
```

这次，任务通过了 OK 信息，因为`testfile.txt`在`list_files `变量中存在。同样，你可以使用`and`和`or`运算符在变量中匹配多个字符串或多个变量。断言功能非常强大，编写过单元测试或集成测试的用户将非常高兴看到这个功能！

# 使用标签进行测试

标签是一种在不运行整个 playbook 的情况下测试一堆任务的好方法。我们可以使用标签在节点上运行实际测试，以验证用户在 playbook 中所期望的状态。我们可以将其视为在实际框中运行 Ansible 的另一种方法来运行集成测试。标签测试方法可以在实际运行 Ansible 的机器上运行，并且主要在部署期间用于测试终端系统的状态。在本节中，我们首先看一下如何通用地使用`tags`，它们可能会帮助我们，不仅仅是用于测试，而且甚至是用于测试目的。

要在 playbook 中添加标签，请使用`tags`参数，后面跟着一个或多个标签名称，用逗号或 YAML 列表分隔。让我们在`playbooks/tag_example.yaml`中创建一个简单的 playbook，以查看标签如何与以下内容一起工作：

```
- hosts: all
  tasks: 
    - name: Ensure the file /tmp/ok exists 
      file: 
        name: /tmp/ok 
        state: touch 
      tags: 
        - file_present 
    - name: Ensure the file /tmp/ok does not exists 
      file: 
        name: /tmp/ok 
        state: absent 
      tags: 
        - file_absent 
```

如果现在运行 playbook，文件将被创建和销毁。我们可以看到它的运行情况：

```
ansible-playbook -i ws01, playbooks/tags_example.yaml
```

它会给我们这个输出：

```
PLAY [all] ***********************************************************

TASK [Gathering Facts] ***********************************************
ok: [ws01]

TASK [Ensure the file /tmp/ok exists] ********************************
changed: [ws01]

TASK [Ensure the file /tmp/ok does not exists] ***********************
changed: [ws01]

PLAY RECAP ***********************************************************
ws01                          : ok=3 changed=2 unreachable=0 failed=0 
```

由于这不是一个幂等的 playbook，如果我们反复运行它，我们将始终看到相同的结果，因为 playbook 将每次都创建和删除文件。

但是，我们添加了两个标签：`file_present`和`file_absent`。你现在可以仅传递`file_present `标签或`file_absent`标签以执行其中一个动作，就像以下示例中所示：

```
ansible-playbook -i ws01, playbooks/tags_example.yaml -t file_present
```

由于`-t file_present`部分，只有带有`file_present`标签的任务将被执行。事实上，这将是输出：

```
PLAY [all] ***********************************************************

TASK [Gathering Facts] ***********************************************
ok: [ws01]

TASK [Ensure the file /tmp/ok exists] ********************************
changed: [ws01]

PLAY RECAP ***********************************************************
ws01                          : ok=2 changed=1 unreachable=0 failed=0 
```

你还可以使用标签在远程主机上执行一组任务，就像从负载均衡器中取出一个服务器并将其添加回负载均衡器。

你还可以将标签与`--check`选项一起使用。通过这样做，你可以在不实际在主机上运行任务的情况下测试你的任务。这使你能够直接测试一堆个别任务，而不必将任务复制到临时 playbook 并从那里运行。

# 理解--skip-tags 选项

Ansible 还提供了一种跳过 playbook 中某些标签的方法。如果你有一个带有多个标签（例如 10 个）的长 playbook，并且你想要执行它们中的全部，但不包括一个，那么向 Ansible 传递九个标签不是一个好主意。如果你忘了传递一个标签，`ansible-run`命令将失败，情况将变得更加困难。为了克服这种情况，Ansible 提供了一种跳过几个标签而不是传递多个应该运行的标签的方法。它的工作方式非常简单，可以按以下方式触发：

```
ansible-playbook -i ws01, playbooks/tags_example.yaml --skip-tags file_present 
```

输出将类似于以下内容：

```
PLAY [all] ***********************************************************

TASK [Gathering Facts] ***********************************************
ok: [ws01]

TASK [Ensure the file /tmp/ok exists] ********************************
changed: [ws01]

PLAY RECAP ***********************************************************
ws01                          : ok=2 changed=1 unreachable=0 failed=0 
```

正如你所见，除了具有 `file_present` 标签的任务之外，所有任务都已执行。

# 理解调试命令

Ansible 允许我们使用两个非常强大的变量来帮助我们调试。

`ANSIBLE_KEEP_REMOTE_FILES` 变量允许我们告诉 Ansible 保留它在远程机器上创建的文件，以便我们可以回过头来调试它们。

`ANSIBLE_DEBUG` 变量允许我们告诉 Ansible 将所有调试内容打印到 shell。调试输出通常过多，但对于某些非常复杂的问题可能有所帮助。

我们已经看到如何在你的 playbook 中查找问题。有时，你知道特定步骤可能会失败，但没关系。在这种情况下，我们应该适当地管理异常。让我们看看如何做到这一点。

# 管理异常

有很多情况下，出于各种原因，你希望你的 playbook 和角色在一个或多个任务失败的情况下继续执行。一个典型的例子是你想检查软件是否安装。让我们看一个以下示例，在只有当 Java 8 未安装时才安装 Java 11。在 `roles/java/tasks/main.yaml` 文件中，我们将输入以下代码：

```
- name: Verify if Java8 is installed
  command: rpm -q java-1.8.0-openjdk
  args:
    warn: False
  register: java 
  ignore_errors: True 
  changed_when: java is failed 

- name: Ensure that Java11 is installed
  yum:
    name: java-11-openjdk
    state: present
  become: True
  when: java is failed
```

在继续执行此角色所需的其他部分之前，我想在角色任务列表的各个部分上花几句话，因为有很多新东西。

在此任务中，我们将执行一个 `rpm` 命令：

```
- name: Verify if Java8 is installed
  command: rpm -q java-1.8.0-openjdk
  args:
    warn: False
  register: java 
  ignore_errors: True 
  changed_when: java is failed 
```

此代码可能有两种可能的输出：

+   失败

+   返回 JDK 软件包的完整名称

由于我们只想检查软件包是否存在，然后继续执行，我们会记录输出（第五行），并忽略可能的失败（第六行）。

当它失败时，意味着 `Java8` 未安装，因此我们可以继续安装 `Java11`：

```
- name: Ensure that Java11 is installed
  yum:
    name: java-11-openjdk
    state: present
  become: True
  when: java is failed
```

创建角色后，我们将需要包含主机机器的 `hosts` 文件；在我的情况下，它将如下所示：

```
ws01
```

我们还需要一个用于应用角色的 playbook，放置在 `playbooks/hosts/j01.fale.io.yaml` 中，内容如下：

```
- hosts: ws01
  roles: 
    - java 
```

现在我们可以用以下命令执行它：

```
ansible-playbook playbooks/hosts/ws01.yaml 
```

我们将得到以下结果：

```
PLAY [ws01] **********************************************************

TASK [Gathering Facts] ***********************************************
ok: [ws01]

TASK [java : Verify if Java8 is installed] ***************************
fatal: [ws01]: FAILED! => {"changed": true, "cmd": ["rpm", "-q", "java-1.8.0-openjdk"], "delta": "0:00:00.028358", "end": "2019-02-10 10:56:22.474350", "msg": "non-zero return code", "rc": 1, "start": "2019-02-10 10:56:22.445992", "stderr": "", "stderr_lines": [], "stdout": "package java-1.8.0-openjdk is not installed", "stdout_lines": ["package java-1.8.0-openjdk is not installed"]}
...ignoring

TASK [java : Ensure that Java11 is installed] ************************
changed: [ws01]

PLAY RECAP ***********************************************************
ws01 : ok=3 changed=2 unreachable=0 failed=0
```

正如你所见，安装检查失败，因为机器上未安装 Java，因此其他任务已按预期执行。

# 触发失败

有些情况下，您希望直接触发一个失败。这可能发生在多种原因下，即使这样做有一些不利之处，因为当您触发失败时，Playbook 将被强行中断，如果您不小心的话，这可能会导致机器处于不一致的状态。我曾亲眼见过一个情况非常适用的场景，那就是当您运行一个不是幂等的 Playbook 时（例如，构建一个应用的新版本），您需要一个变量（例如，要部署的版本/分支）设置。在这种情况下，您可以在开始运行操作之前检查预期的变量是否正确配置，以确保以后的一切都能正常运行。

让我们将以下代码放入`playbooks/maven_build.yaml`中：

```
- hosts: all
  tasks: 
    - name: Ensure the tag variable is properly set
      fail: 'The version needs to be defined. To do so, please add: --extra-vars "version=$[TAG/BRANCH]"' 
      when: version is not defined 
    - name: Get last Project version 
      git: 
        repo: https://github.com/org/project.git 
        dest: "/tmp" 
        version: '{{ version }}' 
    - name: Maven clean install 
      shell: "cd /tmp/project && mvn clean install" 
```

如您所见，我们期望用户在调用脚本的命令中添加`--extra-vars "version=$[TAG/BRANCH]"`。我们本可以设置一个默认要使用的分支，但这样做太冒险，因为用户可能分心并忘记自己添加正确的分支名称，这将导致编译（和部署）错误的应用程序版本。`fail`模块还允许我们指定将显示给用户的消息。

我认为，在手动运行 Playbook 时，`fail`任务要比失败更加有用，因为自动运行 Playbook 时，管理异常通常比直接失败更好。

使用`fail`模块，一旦发现问题，您就可以退出 Playbooks。

# 摘要

在本章中，我们已经学习了如何使用语法检查、带有和不带`--diff`的检查模式以及功能测试来调试 Ansible Playbooks。

作为功能测试的一部分，我们已经看到如何对系统的最终状态进行断言，如何利用标签进行测试，以及如何使用`--syntax-check`选项和`ANSIBLE_KEEP_REMOTE_FILES`和`ANSIBLE_DEBUG`标志。然后，我们转向了失败的管理，最后，我们学习了如何故意触发失败。

在下一章中，我们将讨论多层环境以及部署方法论。
