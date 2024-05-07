# 第八章：高级 Ansible 主题

到目前为止，我们已经努力为您提供了 Ansible 的坚实基础，这样无论您想要的自动化任务是什么，您都可以轻松自如地实现它。然而，当您真正开始加速自动化时，您如何确保能够以一种优雅的方式处理任何出现的情况呢？例如，当您必须启动长时间运行的操作时，如何确保您可以异步运行它们，并在稍后可靠地检查结果？或者，如果您正在更新一大群服务器，如何确保如果一小部分服务器出现故障，play 会及早失败？您最不希望做的事情就是在一百台服务器上部署一个有问题的更新（让我们面对现实，每个人的代码都会不时出现问题）——最好的办法是检测到一小部分服务器失败并根据这个基础中止整个 play，而不是试图继续并破坏整个负载均衡集群。

在本章中，我们将看看如何解决这些特定问题，以及使用 Ansible 的一些更高级功能来控制 playbook 流程和错误处理。我们将通过实际示例探讨如何使用 Ansible 执行滚动更新，如何使用代理和跳板主机（这对于安全环境和核心网络配置通常至关重要），以及如何使用本地 Ansible Vault 技术在静态环境中保护敏感的 Ansible 数据。通过本章结束时，您将全面了解如何在小型环境中以及在大型、安全、关键任务的环境中运行 Ansible。

在本章中，我们将涵盖以下主题：

+   异步与同步操作

+   控制滚动更新的 play 执行

+   配置最大失败百分比

+   设置任务执行委托

+   使用`run_once`选项

+   在本地运行 playbooks

+   使用代理和跳板主机

+   配置 playbook 提示

+   在 play 和任务中放置标签

+   使用 Ansible Vault 保护数据

# 技术要求

本章假设您已经按照第一章中详细介绍的方式在控制主机上设置了 Ansible，并且正在使用最新版本。本章的示例是在 Ansible 2.9 版本下测试的。本章还假设您至少有一个额外的主机进行测试，并且最好是基于 Linux 的。虽然本章将给出特定主机名的示例，但您可以自由地用自己的主机名和/或 IP 地址替换它们；如何做到这一点的详细信息将在适当的位置提供。

本章的代码包可以在[`github.com/PacktPublishing/Ansible-2-Cookbook/tree/master/Chapter%208`](https://github.com/PacktPublishing/Ansible-2-Cookbook/tree/master/Chapter%208)上找到。

# 异步与同步操作

到目前为止，我们已经在本书中看到，Ansible play 是按顺序执行的，每个任务在下一个任务开始之前都会完成。虽然这通常有利于流程控制和逻辑顺序，但有时你可能不希望这样。特别是，可能会出现某个任务运行时间长于配置的 SSH 连接超时时间，而 Ansible 在大多数平台上使用 SSH 来执行自动化任务，这可能会成为一个问题。

幸运的是，Ansible 任务可以异步运行，也就是说，任务可以在目标主机上后台运行，并定期轮询。这与同步任务形成对比，同步任务会保持与目标主机的连接，直到任务完成（这会导致超时的风险）。

和往常一样，让我们通过一个实际的例子来探讨这个问题。假设我们在一个简单的 INI 格式清单中有两台服务器：

```
[frontends]
frt01.example.com
frt02.example.com
```

现在，为了模拟一个长时间运行的任务，我们将使用`shell`模块运行`sleep`命令。但是，我们不会让它在`sleep`命令的持续时间内阻塞 SSH 连接，而是会给任务添加两个特殊参数，如下所示：

```
---
- name: Play to demonstrate asynchronous tasks
  hosts: frontends
  become: true

  tasks:
    - name: A simulated long running task
      shell: "sleep 20"
      async: 30
      poll: 5
```

两个新参数是`async`和`poll`。`async`参数告诉 Ansible 这个任务应该异步运行（这样 SSH 连接就不会被阻塞），最多运行`30`秒。如果任务运行时间超过配置的时间，Ansible 会认为任务失败，整个操作也会失败。当`poll`设置为正整数时，Ansible 会以指定的间隔（在这个例子中是每`5`秒）检查异步任务的状态。如果`poll`设置为`0`，那么任务会在后台运行，不会被检查，需要你编写一个任务来手动检查它的状态。

如果你不指定`poll`值，它将被设置为 Ansible 的`DEFAULT_POLL_INTERVAL`配置参数定义的默认值（即`10`秒）。

当你运行这个 playbook 时，你会发现它的运行方式和其他 playbook 一样；从终端输出中，你看不出任何区别。但在幕后，Ansible 会每`5`秒检查任务，直到成功或达到`30`秒的`async`超时值：

```
$ ansible-playbook -i hosts async.yml

PLAY [Play to demonstrate asynchronous tasks] **********************************

TASK [Gathering Facts] *********************************************************
ok: [frt02.example.com]
ok: [frt01.example.com]

TASK [A simulated long running task] *******************************************
changed: [frt02.example.com]
changed: [frt01.example.com]

PLAY RECAP *********************************************************************
frt01.example.com : ok=2 changed=1 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
frt02.example.com : ok=2 changed=1 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
```

如果你想稍后检查任务（也就是说，如果`poll`设置为`0`），你可以在 playbook 中添加第二个任务，使其如下所示：

```
---
- name: Play to demonstrate asynchronous tasks
  hosts: frontends
  become: true

  tasks:
    - name: A simulated long running task
      shell: "sleep 20"
      async: 30
      poll: 0
      register: long_task

    - name: Check on the asynchronous task
      async_status:
        jid: "{{ long_task.ansible_job_id }}"
      register: async_result
      until: async_result.finished
      retries: 30
```

在这个 playbook 中，初始的异步任务与之前一样定义，只是现在我们将`poll`设置为`0`。我们还选择将这个任务的结果注册到一个名为`long_task`的变量中，这样我们在后面检查时就可以查询任务的作业 ID。play 中的下一个（新的）任务使用`async_status`模块来检查我们从第一个任务中注册的作业 ID，并循环直到作业完成或达到`30`次重试为止。在 playbook 中使用这些时，你几乎肯定不会像这样连续添加两个任务——通常情况下，你会在它们之间执行其他任务，但为了保持这个例子简单，我们将连续运行这两个任务。运行这个 playbook 应该会产生类似以下的输出：

```
$ ansible-playbook -i hosts async2.yml

PLAY [Play to demonstrate asynchronous tasks] **********************************

TASK [Gathering Facts] *********************************************************
ok: [frt01.example.com]
ok: [frt02.example.com]

TASK [A simulated long running task] *******************************************
changed: [frt02.example.com]
changed: [frt01.example.com]

TASK [Check on the asynchronous task] ******************************************
FAILED - RETRYING: Check on the asynchronous task (30 retries left).
FAILED - RETRYING: Check on the asynchronous task (30 retries left).
FAILED - RETRYING: Check on the asynchronous task (29 retries left).
FAILED - RETRYING: Check on the asynchronous task (29 retries left).
FAILED - RETRYING: Check on the asynchronous task (28 retries left).
FAILED - RETRYING: Check on the asynchronous task (28 retries left).
FAILED - RETRYING: Check on the asynchronous task (27 retries left).
FAILED - RETRYING: Check on the asynchronous task (27 retries left).
changed: [frt01.example.com]
changed: [frt02.example.com]

PLAY RECAP *********************************************************************
frt01.example.com : ok=3 changed=2 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
frt02.example.com : ok=3 changed=2 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
```

在上面的代码块中，我们可以看到长时间运行的任务一直在运行，下一个任务会轮询其状态，直到满足我们设置的条件。在这种情况下，我们可以看到任务成功完成，整个操作结果也是成功的。异步操作对于大型下载、软件包更新和其他可能需要长时间运行的任务特别有用。在你的 playbook 开发中，特别是在更复杂的基础设施中，你可能会发现它们很有用。

有了这个基础，让我们来看看另一个在大型基础设施中可能有用的高级技术——使用 Ansible 进行滚动更新。

# 控制滚动更新的 play 执行

默认情况下，Ansible 会在多个主机上并行执行任务，以加快大型清单中的自动化任务。这个设置由 Ansible 配置文件中的`forks`参数定义，默认值为`5`（因此，默认情况下，Ansible 尝试在五个主机上同时运行自动化任务）。

在负载均衡环境中，这并不理想，特别是如果你想避免停机时间。假设我们的清单中有五个前端服务器（或者甚至更少）。如果我们允许 Ansible 同时更新所有这些服务器，终端用户可能会遇到服务中断。因此，重要的是考虑在不同的时间更新所有服务器。让我们重用上一节中的清单，其中只有两个服务器。显然，如果这些服务器在负载均衡环境中，我们只能一次更新一个；如果同时取出服务，那么终端用户肯定会失去对服务的访问，直到 Ansible play 成功完成。

答案是在 play 定义中使用`serial`关键字来确定一次操作多少个主机。让我们通过一个实际的例子来演示这一点：

1.  创建以下简单的 playbook，在我们的清单中的两个主机上运行两个命令。在这个阶段，命令的内容并不重要，但如果你使用`command`模块运行`date`命令，你将能够看到每个任务运行的时间，以及当你运行 play 时，如果指定了`-v`来增加详细信息：

```
---
- name: Simple serial demonstration play
  hosts: frontends
  gather_facts: false

  tasks:
    - name: First task
      command: date
    - name: Second task
      command: date
```

1.  现在，如果你运行这个 play，你会发现它在每个主机上同时执行所有操作，因为我们的主机数量少于默认的 fork 数量——`5`。这种行为对于 Ansible 来说是正常的，但并不是我们想要的，因为我们的用户将会遇到服务中断：

```
$ ansible-playbook -i hosts serial.yml

PLAY [Simple serial demonstration play] ****************************************

TASK [First task] **************************************************************
changed: [frt02.example.com]
changed: [frt01.example.com]

TASK [Second task] *************************************************************
changed: [frt01.example.com]
changed: [frt02.example.com]

PLAY RECAP *********************************************************************
frt01.example.com : ok=2 changed=2 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
frt02.example.com : ok=2 changed=2 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
```

1.  现在，让我们修改 play 定义，如下所示。我们将`tasks`部分保持与*步骤 1*中完全相同：

```
---
- name: Simple serial demonstration play
  hosts: frontends
  serial: 1
  gather_facts: false
```

1.  注意`serial: 1`这一行的存在。这告诉 Ansible 在移动到下一个主机之前一次在 1 个主机上完成 play。如果我们再次运行 play，我们可以看到它的运行情况：

```
$ ansible-playbook -i hosts serial.yml

PLAY [Simple serial demonstration play] ****************************************

TASK [First task] **************************************************************
changed: [frt01.example.com]

TASK [Second task] *************************************************************
changed: [frt01.example.com]

PLAY [Simple serial demonstration play] ****************************************

TASK [First task] **************************************************************
changed: [frt02.example.com]

TASK [Second task] *************************************************************
changed: [frt02.example.com]

PLAY RECAP *********************************************************************
frt01.example.com : ok=2 changed=2 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
frt02.example.com : ok=2 changed=2 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
```

太好了！如果你想象一下，这个 playbook 实际上是在负载均衡器上禁用这些主机，执行升级，然后重新启用负载均衡器上的主机，这正是你希望操作进行的方式。如果没有`serial: 1`指令，所有主机将同时从负载均衡器中移除，导致服务中断。

值得注意的是，`serial`指令也可以取百分比而不是整数。当你指定一个百分比时，你告诉 Ansible 一次在百分之多少的主机上运行 play。所以，如果你的清单中有 4 个主机，并指定`serial: 25%`，Ansible 将一次只在一个主机上运行 play。如果你的清单中有 8 个主机，它将一次在两个主机上运行 play。我相信你明白了！

你甚至可以通过将列表传递给`serial`指令来构建这个。考虑以下代码：

```
  serial:
    - 1
    - 3
    - 5
```

这告诉 Ansible 一开始在 1 个主机上运行 play，然后在接下来的 3 个主机上运行，然后一次在 5 个主机上运行，直到清单完成。你也可以在整数主机数的位置指定一个百分比列表。通过这样做，你将建立一个强大的 playbook，可以执行滚动更新而不会导致终端用户服务中断。完成这一点后，让我们进一步通过控制 Ansible 在中止 play 之前可以容忍的最大失败百分比来增进这一知识，这在高可用或负载平衡环境中将会再次非常有用。

# 配置最大失败百分比

在其默认操作模式下，只要库存中有主机并且没有记录失败，Ansible 就会继续在一批服务器上执行播放（批处理大小由我们在前一节中讨论的`serial`指令确定）。显然，在高可用或负载平衡环境中（如我们之前讨论的环境），这并不理想。如果您的播放中有错误，或者可能存在代码问题，您最不希望的是 Ansible 忠实地将其部署到集群中的所有服务器，导致服务中断，因为所有节点都遭受了失败的升级。在这种环境中，最好的做法是尽早失败，并且在某人能够介入并解决问题之前至少保留一些主机不受影响。

对于我们的实际示例，让我们考虑一个扩展的库存，其中有`10`个主机。我们将定义如下：

```
[frontends]
frt[01:10].example.com
```

现在，让我们创建一个简单的 playbook 在这些主机上运行。我们将在 play 定义中将批处理大小设置为`5`，将`max_fail_percentage`设置为`50%`：

1.  创建以下 play 定义来演示`max_fail_percentage`指令的使用：

```
---
- name: A simple play to demonstrate use of max_fail_percentage
  hosts: frontends
  gather_facts: no
  serial: 5
  max_fail_percentage: 50
```

我们在库存中定义了`10`个主机，因此它将以 5 个一组（由`serial: 5`指定）进行处理。如果一批中超过 50%的主机失败，我们将失败整个播放并停止处理。

失败主机的数量必须超过`max_fail_percentage`的值；如果相等，则播放继续。因此，在我们的示例中，如果我们的主机正好有 50%失败，播放仍将继续。

1.  接下来，我们将定义两个简单的任务。第一个任务下面有一个特殊的子句，我们用它来故意模拟一个失败——这一行以`failed_when`开头，我们用它告诉任务，如果它在批次中的前三个主机上运行此任务，那么无论结果如何，它都应该故意失败此任务；否则，它应该允许任务正常运行：

```
  tasks:
    - name: A task that will sometimes fail
      debug:
        msg: This might fail
      failed_when: inventory_hostname in ansible_play_batch[0:3]
```

1.  最后，我们将添加一个始终成功的第二个任务。如果播放被允许继续，则运行此任务，但如果播放被中止，则不运行：

```
    - name: A task that will succeed
      debug:
        msg: Success!
```

因此，我们故意构建了一个将在 10 个主机库存中以 5 个主机一组的方式运行的 playbook，但是如果任何给定批次中超过 50%的主机经历失败，则播放将中止。我们还故意设置了一个失败条件，导致第一批 5 个主机中的三个（60%）失败。

1.  运行 playbook，让我们观察发生了什么：

```
$ ansible-playbook -i morehosts maxfail.yml

PLAY [A simple play to demonstrate use of max_fail_percentage] *****************

TASK [A task that will sometimes fail] *****************************************
fatal: [frt01.example.com]: FAILED! => {
 "msg": "This might fail"
}
fatal: [frt02.example.com]: FAILED! => {
 "msg": "This might fail"
}
fatal: [frt03.example.com]: FAILED! => {
 "msg": "This might fail"
}
ok: [frt04.example.com] => {
 "msg": "This might fail"
}
ok: [frt05.example.com] => {
 "msg": "This might fail"
}

NO MORE HOSTS LEFT *************************************************************

NO MORE HOSTS LEFT *************************************************************

PLAY RECAP *********************************************************************
frt01.example.com : ok=0 changed=0 unreachable=0 failed=1 skipped=0 rescued=0 ignored=0
frt02.example.com : ok=0 changed=0 unreachable=0 failed=1 skipped=0 rescued=0 ignored=0
frt03.example.com : ok=0 changed=0 unreachable=0 failed=1 skipped=0 rescued=0 ignored=0
frt04.example.com : ok=1 changed=0 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
frt05.example.com : ok=1 changed=0 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
```

注意此 playbook 的结果。我们故意使前 5 个批次中的三个失败，超过了我们设置的`max_fail_percentage`的阈值。这立即导致播放中止，并且第二个任务不会在前 5 个批次上执行。您还会注意到，10 个主机中的第二批次从未被处理，因此我们的播放确实被中止。这正是您希望看到的行为，以防止失败的更新在整个集群中传播。通过谨慎使用批处理和`max_fail_percentage`，您可以在整个集群上安全运行自动化任务，而不必担心在出现问题时破坏整个集群。在下一节中，我们将介绍 Ansible 的另一个功能，当涉及到与集群一起工作时，这个功能可能非常有用——任务委派。

# 设置任务执行委托

到目前为止，我们运行的每个 play 都假设所有任务都按顺序在库存中的每个主机上执行。但是，如果您需要在不同的主机上运行一个或两个任务怎么办？例如，我们已经谈到了在集群上自动升级的概念。但是从逻辑上讲，我们希望自动化整个过程，包括从负载平衡器中逐个删除每个主机并在任务完成后将其返回。

尽管我们仍然希望在整个清单上运行我们的 play，但我们肯定不希望从这些主机上运行负载均衡器命令。让我们再次通过一个实际示例详细解释这一点。我们将重用本章前面使用的两个简单主机清单：

```
[frontends]
frt01.example.com
frt02.example.com
```

现在，让我们在与我们的 playbook 相同的目录中创建两个简单的 shell 脚本来处理这个问题。这只是示例，因为设置负载均衡器超出了本书的范围。但是，请想象您有一个可以调用的 shell 脚本（或其他可执行文件），可以将主机添加到负载均衡器中并从中删除：

1.  对于我们的示例，让我们创建一个名为`remove_from_loadbalancer.sh`的脚本，其中包含以下内容：

```
#!/bin/sh
echo Removing $1 from load balancer...
```

1.  我们还将创建一个名为`add_to_loadbalancer.sh`的脚本，其中包含以下内容：

```
#!/bin/sh
echo Adding $1 to load balancer...
```

显然，在实际示例中，这些脚本中会有更多的代码！

1.  现在，让我们创建一个 playbook，执行我们在这里概述的逻辑。我们首先创建一个非常简单的 play 定义（您可以自由地尝试`serial`和`max_fail_percentage`指令），以及一个初始任务：

```
---
- name: Play to demonstrate task delegation
  hosts: frontends

  tasks:
    - name: Remove host from the load balancer
      command: ./remove_from_loadbalancer.sh {{ inventory_hostname }}
      args:
        chdir: "{{ playbook_dir }}"
      delegate_to: localhost
```

请注意任务结构——大部分内容对您来说应该很熟悉。我们使用`command`模块调用我们之前创建的脚本，将从要从负载均衡器中移除的清单中的主机名传递给脚本。我们使用`chdir`参数和`playbook_dir`魔术变量告诉 Ansible 脚本要从与 playbook 相同的目录中运行。

这个任务的特殊之处在于`delegate_to`指令，它告诉 Ansible，即使我们正在遍历一个不包含`localhost`的清单，我们也应该在`localhost`上运行此操作（我们没有将脚本复制到远程主机，因此如果我们尝试从那里运行它，它将不会运行）。

1.  之后，我们添加一个任务，其中进行升级工作。这个任务没有`delegate_to`指令，因此实际上是在清单中的远程主机上运行的（这是我们想要的）：

```
    - name: Deploy code to host
      debug:
        msg: Deployment code would go here....
```

1.  最后，我们使用我们之前创建的第二个脚本将主机重新添加到负载均衡器。这个任务几乎与第一个任务相同：

```
    - name: Add host back to the load balancer
      command: ./add_to_loadbalancer.sh {{ inventory_hostname }}
      args:
        chdir: "{{ playbook_dir }}"
      delegate_to: localhost
```

1.  让我们看看这个 playbook 的运行情况：

```
$ ansible-playbook -i hosts delegate.yml

PLAY [Play to demonstrate task delegation] *************************************

TASK [Gathering Facts] *********************************************************
ok: [frt01.example.com]
ok: [frt02.example.com]

TASK [Remove host from the load balancer] **************************************
changed: [frt02.example.com -> localhost]
changed: [frt01.example.com -> localhost]

TASK [Deploy code to host] *****************************************************
ok: [frt01.example.com] => {
 "msg": "Deployment code would go here...."
}
ok: [frt02.example.com] => {
 "msg": "Deployment code would go here...."
}

TASK [Add host back to the load balancer] **************************************
changed: [frt01.example.com -> localhost]
changed: [frt02.example.com -> localhost]

PLAY RECAP *********************************************************************
frt01.example.com : ok=4 changed=2 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
frt02.example.com : ok=4 changed=2 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
```

请注意，即使 Ansible 正在通过清单（其中不包括`localhost`）工作，与负载均衡器相关的脚本实际上是从`localhost`运行的，而升级任务是直接在远程主机上执行的。当然，这并不是您可以使用任务委派的唯一方式，但这是一个常见的例子，说明了它可以帮助您的一种方式。

事实上，您可以将任何任务委派给`localhost`，甚至是另一个非清单主机。例如，您可以委派一个`rsync`命令给`localhost`，使用类似于前面的任务定义来将文件复制到远程主机。这很有用，因为尽管 Ansible 有一个`copy`模块，但它无法执行`rsync`能够执行的高级递归`copy`和`update`功能。

另外，请注意，您可以选择在您的 playbooks（和 roles）中使用一种速记符号表示`delegate_to`，称为`local_action`。这允许您在单行上指定一个任务，该任务通常会在其下方添加`delegate_to: localhost`来运行。将所有这些内容整合到第二个示例中，我们的 playbook 将如下所示：

```
---
- name: Second task delegation example
  hosts: frontends

  tasks:
  - name: Perform an rsync from localhost to inventory hosts
    local_action: command rsync -a /tmp/ {{ inventory_hostname }}:/tmp/target/
```

上述速记符号等同于以下内容：

```
tasks:
  - name: Perform an rsync from localhost to inventory hosts
    command: rsync -a /tmp/ {{ inventory_hostname }}:/tmp/target/
    delegate_to: localhost
```

如果我们运行这个 playbook，我们可以看到`local_action`确实从`localhost`运行了`rsync`，使我们能够高效地将整个目录树复制到清单中的远程服务器上：

```
$ ansible-playbook -i hosts delegate2.yml

PLAY [Second task delegation example] ******************************************

TASK [Gathering Facts] *********************************************************
ok: [frt02.example.com]
ok: [frt01.example.com]

TASK [Perform an rsync from localhost to inventory hosts] **********************
changed: [frt02.example.com -> localhost]
changed: [frt01.example.com -> localhost]

PLAY RECAP *********************************************************************
frt01.example.com : ok=2 changed=1 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
frt02.example.com : ok=2 changed=1 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
```

这结束了我们对任务委派的讨论，尽管如前所述，这只是两个常见的例子。我相信您可以想出一些更高级的用例来使用这个功能。让我们继续通过在下一节中查看特殊的`run_once`选项来控制 Ansible 代码的流程。

# 使用`run_once`选项

在处理集群时，有时会遇到一项任务，应该只对整个集群执行一次。例如，你可能想要升级集群数据库的模式，或者发出一个重新配置 Pacemaker 集群的命令，这通常会在一个节点上发出，并由 Pacemaker 复制到所有其他节点。当然，你可以通过一个特殊的只有一个主机的清单来解决这个问题，或者甚至通过编写一个特殊的 play 来引用清单中的一个主机，但这是低效的，并且开始使你的代码变得分散。

相反，你可以像平常一样编写你的代码，但是利用特殊的`run_once`指令来运行你想要在你的清单上只运行一次的任务。例如，让我们重用本章前面定义的包含 10 个主机的清单。现在，让我们继续演示这个选项，如下所示：

1.  按照下面的代码块创建简单的 playbook。我们使用一个 debug 语句来显示一些输出，但在现实生活中，你可以插入你的脚本或命令来执行你的一次性集群功能（例如，升级数据库模式）：

```
---
- name: Play to demonstrate the run_once directive
  hosts: frontends

  tasks:
    - name: Upgrade database schema
      debug:
        msg: Upgrading database schema...
      run_once: true
```

1.  现在，让我们运行这个 playbook，看看会发生什么：

```
$ ansible-playbook -i morehosts runonce.yml

PLAY [Play to demonstrate the run_once directive] ******************************

TASK [Gathering Facts] *********************************************************
ok: [frt02.example.com]
ok: [frt05.example.com]
ok: [frt03.example.com]
ok: [frt01.example.com]
ok: [frt04.example.com]
ok: [frt06.example.com]
ok: [frt08.example.com]
ok: [frt09.example.com]
ok: [frt07.example.com]
ok: [frt10.example.com]

TASK [Upgrade database schema] *************************************************
ok: [frt01.example.com] => {
 "msg": "Upgrading database schema..."
}
---

PLAY RECAP *********************************************************************
frt01.example.com : ok=2 changed=0 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
frt02.example.com : ok=1 changed=0 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
frt03.example.com : ok=1 changed=0 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
frt04.example.com : ok=1 changed=0 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
frt05.example.com : ok=1 changed=0 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
frt06.example.com : ok=1 changed=0 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
frt07.example.com : ok=1 changed=0 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
frt08.example.com : ok=1 changed=0 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
frt09.example.com : ok=1 changed=0 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
frt10.example.com : ok=1 changed=0 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
```

请注意，正如所期望的那样，尽管 playbook 在所有 10 个主机上运行了（并且确实从所有 10 个主机收集了信息），我们只在一个主机上运行了升级任务。

1.  重要的是要注意，`run_once`选项适用于每批服务器，因此如果我们在我们的 play 定义中添加`serial: 5`（在我们的 10 台服务器清单上以 5 台服务器的两批运行我们的 play），模式升级任务实际上会运行两次！它按照要求运行一次，但是每批服务器运行一次，而不是整个清单运行一次。在处理这个指令时，要注意这个细微差别。

将`serial: 5`添加到你的 play 定义中，并重新运行 playbook。输出应该如下所示：

```
$ ansible-playbook -i morehosts runonce.yml

PLAY [Play to demonstrate the run_once directive] ******************************

TASK [Gathering Facts] *********************************************************
ok: [frt04.example.com]
ok: [frt01.example.com]
ok: [frt02.example.com]
ok: [frt03.example.com]
ok: [frt05.example.com]

TASK [Upgrade database schema] *************************************************
ok: [frt01.example.com] => {
 "msg": "Upgrading database schema..."
}

PLAY [Play to demonstrate the run_once directive] ******************************

TASK [Gathering Facts] *********************************************************
ok: [frt08.example.com]
ok: [frt06.example.com]
ok: [frt07.example.com]
ok: [frt10.example.com]
ok: [frt09.example.com]

TASK [Upgrade database schema] *************************************************
ok: [frt06.example.com] => {
 "msg": "Upgrading database schema..."
}

PLAY RECAP *********************************************************************
frt01.example.com : ok=2 changed=0 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
frt02.example.com : ok=1 changed=0 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
frt03.example.com : ok=1 changed=0 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
frt04.example.com : ok=1 changed=0 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
frt05.example.com : ok=1 changed=0 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
frt06.example.com : ok=2 changed=0 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
frt07.example.com : ok=1 changed=0 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
frt08.example.com : ok=1 changed=0 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
frt09.example.com : ok=1 changed=0 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
frt10.example.com : ok=1 changed=0 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
```

这就是`run_once`选项设计的工作原理 - 你可以观察到，在前面的输出中，我们的模式升级运行了两次，这可能不是我们想要的！然而，有了这个意识，你应该能够利用这个选项来控制你的 playbook 流程跨集群，并且仍然实现你想要的结果。现在让我们远离与集群相关的 Ansible 任务，来看一下在本地运行 playbook 和在`localhost`上运行 playbook 之间微妙但重要的区别。

# 在本地运行 playbook

重要的是要注意，当我们谈论使用 Ansible 在本地运行 playbook 时，这并不等同于在`localhost`上运行它。如果我们在`localhost`上运行 playbook，Ansible 实际上会建立一个到`localhost`的 SSH 连接（它不区分其行为，也不尝试检测清单中的主机是本地还是远程 - 它只是忠实地尝试连接）。

实际上，我们可以尝试创建一个包含以下内容的`local`清单文件：

```
[local]
localhost
```

现在，如果我们尝试对这个清单运行一个 ad hoc 命令中的`ping`模块，我们会看到以下内容：

```
$ ansible -i localhosts -m ping all --ask-pass
The authenticity of host 'localhost (::1)' can't be established.
ECDSA key fingerprint is SHA256:DUwVxH+45432pSr9qsN8Av4l0KJJ+r5jTo123n3XGvZs.
ECDSA key fingerprint is MD5:78:d1:dc:23:cc:28:51:42:eb:fb:58:49:ab:92:b6:96.
Are you sure you want to continue connecting (yes/no)? yes
SSH password:
localhost | SUCCESS => {
 "ansible_facts": {
 "discovered_interpreter_python": "/usr/bin/python"
 },
 "changed": false,
 "ping": "pong"
}
```

正如你所看到的，Ansible 建立了一个需要主机密钥验证的 SSH 连接，以及我们的 SSH 密码。尽管你可以添加主机密钥（就像我们在前面的代码块中所做的那样），为你的`localhost`添加基于密钥的 SSH 身份验证等等，但有一种更直接的方法来做到这一点。

现在我们可以修改我们的清单，使其如下所示：

```
[local]
localhost ansible_connection=local
```

我们在`localhost`条目中添加了一个特殊的变量 - `ansible_connection`变量 - 它定义了用于连接到这个清单主机的协议。因此，我们告诉它使用直接的本地连接，而不是 SSH 连接（这是默认值）。

应该注意的是，`ansible_connection`变量的这个特殊值实际上覆盖了您在清单中放置的主机名。因此，如果我们将我们的清单更改为如下所示，Ansible 甚至不会尝试连接到名为`frt01.example.com`的远程主机，它将在本地连接到运行 playbook 的机器（不使用 SSH）：

```
[local]
frt01.example.com ansible_connection=local
```

我们可以非常简单地演示这一点。让我们首先检查一下我们本地`/tmp`目录中是否缺少一个测试文件：

```
ls -l /tmp/foo
ls: cannot access /tmp/foo: No such file or directory
```

现在，让我们运行一个临时命令，在我们刚刚定义的新清单中的所有主机上触摸这个文件：

```
$ ansible -i localhosts2 -m file -a "path=/tmp/foo state=touch" all
frt01.example.com | CHANGED => {
 "ansible_facts": {
 "discovered_interpreter_python": "/usr/bin/python"
 },
 "changed": true,
 "dest": "/tmp/foo",
 "gid": 0,
 "group": "root",
 "mode": "0644",
 "owner": "root",
 "size": 0,
 "state": "file",
 "uid": 0
}
```

命令成功运行，现在让我们看看本地机器上是否存在测试文件：

```
$ ls -l /tmp/foo
-rw-r--r-- 1 root root 0 Apr 24 16:28 /tmp/foo
```

是的！因此，临时命令并没有尝试连接到`frt01.example.com`，即使这个主机名在清单中。`ansible_connection=local`的存在意味着这个命令在本地机器上运行，而不使用 SSH。

在本地运行命令而无需设置 SSH 连接、SSH 密钥等，这种能力可能非常有价值，特别是如果您需要在本地机器上快速启动和运行。完成后，让我们看看如何使用 Ansible 处理代理和跳转主机。

# 使用代理和跳转主机

通常，在配置核心网络设备时，这些设备通过代理或跳转主机与主网络隔离。Ansible 非常适合自动化网络设备配置，因为大部分操作都是通过 SSH 执行的：然而，这只在 Ansible 可以安装和从跳转主机操作的情况下才有帮助，或者更好的是可以通过这样的主机操作。

幸运的是，Ansible 确实可以做到这一点。假设您的网络中有两个 Cumulus Networks 交换机（这些基于 Linux 的特殊分发用于交换硬件，非常类似于 Debian）。这两个交换机分别具有`cmls01.example.com`和`cmls02.example.com`的主机名，但都只能从名为`bastion.example.com`的主机访问。

支持我们的`bastion`主机的配置是在清单中进行的，而不是在 playbook 中。我们首先按照正常方式定义一个包含交换机的清单组：

```
[switches]
cmls01.example.com
cmls02.example.com
```

然而，现在我们可以开始变得聪明起来，通过在清单变量中添加一些特殊的 SSH 参数来实现。将以下代码添加到您的清单文件中：

```
[switches:vars]
ansible_ssh_common_args='-o ProxyCommand="ssh -W %h:%p -q bastion.example.com"'
```

这个特殊的变量内容告诉 Ansible 在设置 SSH 连接时添加额外的选项，包括通过`bastion.example.com`主机进行代理。`-W %h:%p`选项告诉 SSH 代理连接，并连接到由`%h`指定的主机（这通常是`cmls01.example.com`或`cmls02.example.com`），在由`%p`指定的端口上（通常是端口`22`）。

现在，如果我们尝试对这个清单运行 Ansible 的`ping`模块，我们可以看到它是否有效：

```
$ ansible -i switches -m ping all
cmls02.example.com | SUCCESS => {
 "ansible_facts": {
 "discovered_interpreter_python": "/usr/bin/python"
 },
 "changed": false,
127.0.0.1 app02.example.com
 "ping": "pong"
}
cmls01.example.com | SUCCESS => {
 "ansible_facts": {
 "discovered_interpreter_python": "/usr/bin/python"
 },
 "changed": false,
 "ping": "pong"
}
```

您会注意到，我们实际上无法在命令行输出中看到 Ansible 行为上的任何差异。表面上，Ansible 的工作方式与平常一样，并且成功连接到了两个主机。然而，在幕后，它通过`bastion.example.com`进行代理。

请注意，这个简单的例子假设您使用相同的用户名和 SSH 凭据（或在这种情况下，密钥）连接到`bastion`主机和`switches`。有方法可以为这两个变量提供单独的凭据，但这涉及到更高级的 OpenSSH 使用，这超出了本书的范围。然而，本节旨在给您一个起点，并演示这种可能性，您可以自行探索 OpenSSH 代理。

现在让我们改变方向，探讨如何设置 Ansible 在 playbook 运行期间提示您输入数据的可能性。

# 配置 playbook 提示

到目前为止，我们所有的 playbook 都是在其中为它们指定的数据在运行时在 playbook 中定义的变量中。然而，如果您在 playbook 运行期间实际上想要从某人那里获取信息怎么办？也许您希望用户选择要安装的软件包的版本？或者，也许您希望从用户那里获取密码，以用于身份验证任务，而不将其存储在任何地方。（尽管 Ansible Value 可以加密静态数据，但一些公司可能禁止在他们尚未评估的工具中存储密码和其他凭据。）幸运的是，对于这些情况（以及许多其他情况），Ansible 可以提示您输入用户输入，并将输入存储在变量中以供将来处理。

让我们重用本章开头定义的两个主机前端清单。现在，让我们通过一个实际的例子演示如何在 playbook 运行期间从用户那里获取数据：

1.  以通常的方式创建一个简单的 play 定义，如下所示：

```
---
- name: A simple play to demonstrate prompting in a playbook
  hosts: frontends
```

1.  现在，我们将在 play 定义中添加一个特殊的部分。我们之前定义了一个`vars`部分，但这次我们将定义一个叫做`vars_prompt`的部分（它使您能够通过用户提示定义变量）。在这个部分，我们将提示两个变量——一个是用户 ID，一个是密码。一个将被回显到屏幕上，而另一个不会，通过设置`private: yes`：

```
  vars_prompt:
    - name: loginid
      prompt: "Enter your username"
      private: no
    - name: password
      prompt: "Enter your password"
      private: yes
```

1.  现在，我们将向我们的 playbook 添加一个任务，以演示设置变量的提示过程：

```
  tasks:
    - name: Proceed with login
      debug:
        msg: "Logging in as {{ loginid }}..."
```

1.  现在，让我们运行 playbook 并看看它的行为：

```
$ ansible-playbook -i hosts prompt.yml
Enter your username: james
Enter your password:

PLAY [A simple play to demonstrate prompting in a playbook] ********************

TASK [Gathering Facts] *********************************************************
ok: [frt01.example.com]
ok: [frt02.example.com]

TASK [Proceed with login] ******************************************************
ok: [frt01.example.com] => {
 "msg": "Logging in as james..."
}
ok: [frt02.example.com] => {
 "msg": "Logging in as james..."
}

PLAY RECAP *********************************************************************
frt01.example.com : ok=2 changed=0 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
frt02.example.com : ok=2 changed=0 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
```

正如您所看到的，我们提示输入两个变量，但密码没有回显到终端，这对于安全原因很重要。然后我们可以在 playbook 中稍后使用这些变量。在这里，我们只是使用一个简单的`debug`命令来演示变量已经设置；然而，您可以在此处实现一个实际的身份验证功能，而不是这样做。

完成后，让我们继续下一节，看看如何通过标记有选择地运行 play 中的任务。

# 在 play 和任务中放置标记

在本书的许多部分，我们已经讨论过，随着您对 Ansible 的信心和经验的增长，您的 playbook 可能会在大小、规模和复杂性上增长。虽然这无疑是一件好事，但有时您可能只想运行 playbook 的一个子集，而不是从头到尾运行它。我们已经讨论了如何根据变量或事实的值有条件地运行任务，但是否有一种方法可以根据在运行 playbook 时做出的选择来运行它们？

Ansible play 中的标记是解决这个问题的方法，在本节中，我们将构建一个简单的 playbook，其中包含两个任务——每个任务都有一个不同的标记，以向您展示标记的工作原理。我们将使用之前使用过的两个简单主机清单：

1.  创建以下简单的 playbook 来执行两个任务——一个是安装`nginx`包，另一个是从模板部署配置文件：

```
---
- name: Simple play to demonstrate use of tags
  hosts: frontends

  tasks:
    - name: Install nginx
      yum:
        name: nginx
        state: present
      tags:
        - install

    - name: Install nginx configuration from template
      template:
        src: templates/nginx.conf.j2
        dest: /etc/nginx.conf
      tags:
        - customize
```

1.  现在，让我们以通常的方式运行 playbook，但有一个区别——这一次，我们将在命令行中添加`--tags`开关。这个开关告诉 Ansible 只运行与指定标记匹配的任务。例如，运行以下命令：

```
$ ansible-playbook -i hosts tags.yml --tags install

PLAY [Simple play to demonstrate use of tags] **********************************

TASK [Gathering Facts] *********************************************************
ok: [frt02.example.com]
ok: [frt01.example.com]

TASK [Install nginx] ***********************************************************
changed: [frt02.example.com]
changed: [frt01.example.com]

PLAY RECAP *********************************************************************
frt01.example.com : ok=2 changed=1 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
frt02.example.com : ok=2 changed=1 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
```

请注意，部署配置文件的任务没有运行。这是因为它被标记为`customize`，而我们在运行 playbook 时没有指定这个标记。

1.  还有一个`--skip-tags`开关，它与前一个开关相反——它告诉 Ansible 跳过列出的标记。因此，如果我们再次运行 playbook 但跳过`customize`标记，我们应该会看到类似以下的输出：

```
$ ansible-playbook -i hosts tags.yml --skip-tags customize

PLAY [Simple play to demonstrate use of tags] **********************************

TASK [Gathering Facts] *********************************************************
ok: [frt02.example.com]
ok: [frt01.example.com]

TASK [Install nginx] ***********************************************************
ok: [frt02.example.com]
ok: [frt01.example.com]

PLAY RECAP *********************************************************************
frt01.example.com : ok=2 changed=0 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
frt02.example.com : ok=2 changed=0 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
```

这个 play 运行是相同的，因为我们跳过了标记为`customize`的任务，而不是只包括`install`标记的任务。

请注意，如果您不指定`--tags`或`--skip-tags`，则所有任务都会运行，而不管它们的标记如何。

关于标签的一些说明——首先，每个任务可以有多个标签，因此我们看到它们以 YAML 列表格式指定。如果使用`--tags`开关，如果任何一个标签与命令行上指定的标签匹配，任务将运行。其次，标签可以被重复使用，因此我们可以有五个被全部标记为`install`的任务，如果您通过`--tags`或`--skip-tags`要求执行它们，所有五个任务都将被执行或跳过。

您还可以在命令行上指定多个标签，运行与任何指定标签匹配的所有任务。尽管标签背后的逻辑相对简单，但可能需要一点时间来适应它，而您最不希望做的事情就是在真实主机上运行 playbook，以检查您是否理解标签！了解这一点的一个很好的方法是在您的命令中添加`--list-tasks`，这样—而不是运行 playbook—会列出 playbook 中将要执行的任务。以下代码块为您提供了一些示例，基于我们刚刚创建的示例 playbook：

```
$ ansible-playbook -i hosts tags.yml --skip-tags customize --list-tasks

playbook: tags.yml

 play #1 (frontends): Simple play to demonstrate use of tags TAGS: []
 tasks:
 Install nginx TAGS: [install]

$ ansible-playbook -i hosts tags.yml --tags install,customize --list-tasks

playbook: tags.yml

 play #1 (frontends): Simple play to demonstrate use of tags TAGS: []
 tasks:
 Install nginx TAGS: [install]
 Install nginx configuration from template TAGS: [customize]

$ ansible-playbook -i hosts tags.yml --list-tasks

playbook: tags.yml

 play #1 (frontends): Simple play to demonstrate use of tags TAGS: []
 tasks:
 Install nginx TAGS: [install]
 Install nginx configuration from template TAGS: [customize]
```

正如您所看到的，`--list-tasks`不仅会显示哪些任务将运行，还会显示与它们关联的标签，这有助于您进一步理解标签的工作方式，并确保您实现了所需的 playbook 流程。标签是一种非常简单但强大的控制 playbook 中哪些部分运行的方法，通常在创建和维护大型 playbook 时，最好能够一次只运行所选部分的 playbook。从这里开始，我们将继续进行本章的最后一部分，我们将看看如何通过使用 Ansible Vault 对变量数据进行加密来保护您的静止状态下的变量数据。

# 使用 Ansible Vault 保护数据

Ansible Vault 是 Ansible 附带的一个工具，允许您在静止状态下加密敏感数据，同时在 playbook 中使用它。通常，需要将登录凭据或其他敏感数据存储在变量中，以允许 playbook 无人值守运行。然而，这会使您的数据面临被恶意使用的风险。幸运的是，Ansible Vault 使用 AES-256 加密在静止状态下保护您的数据，这意味着您的敏感数据不会被窥探。

让我们继续进行一个简单的示例，向您展示如何使用 Ansible Vault：

1.  首先创建一个新的保险库来存储敏感数据；我们将称这个文件为`secret.yml`。您可以使用以下命令创建这个文件：

```
$ ansible-vault create secret.yml
New Vault password:
Confirm New Vault password:
```

在提示时输入您为保险库选择的密码，并通过第二次输入来确认它（本书在 GitHub 上附带的保险库是用`secure`密码加密的）。

1.  当您输入密码后，您将被设置为您的正常编辑器（由`EDITOR` shell 变量定义）。在我的测试系统中，这是`vi`。在这个编辑器中，您应该以正常的方式创建一个`vars`文件，其中包含您的敏感数据：

```
---
secretdata: "Ansible is cool!"
```

1.  保存并退出编辑器（在`vi`中按*Esc*，然后输入`:wq`）。您将退出到 shell。现在，如果您查看文件的内容，您会发现它们已经被加密，对于任何不应该能够读取文件的人来说是安全的：

```
$ cat secret.yml
$ANSIBLE_VAULT;1.1;AES256
63333734623764633865633237333166333634353334373862346334643631303163653931306138
6334356465396463643936323163323132373836336461370a343236386266313331653964326334
62363737663165336539633262366636383364343663396335643635623463626336643732613830
6139363035373736370a646661396464386364653935636366633663623261633538626230616630
35346465346430636463323838613037386636333334356265623964633763333532366561323266
3664613662643263383464643734633632383138363663323730
```

1.  然而，Ansible Vault 的伟大之处在于，您可以在 playbook 中像使用普通的`variables`文件一样使用这个加密文件（尽管显然，您必须告诉 Ansible 您的保险库密码）。让我们创建一个简单的 playbook，如下所示：

```
---
- name: A play that makes use of an Ansible Vault
  hosts: frontends

  vars_files:
    - secret.yml

  tasks:
    - name: Tell me a secret
      debug:
        msg: "Your secret data is: {{ secretdata }}"
```

`vars_files`指令的使用方式与使用未加密的`variables`文件时完全相同。Ansible 在运行时读取`variables`文件的头，并确定它们是否已加密。

1.  尝试在不告诉 Ansible 保险库密码的情况下运行 playbook——在这种情况下，您应该会收到类似于这样的错误：

```
$ ansible-playbook -i hosts vaultplaybook.yml
ERROR! Attempting to decrypt but no vault secrets found
```

1.  Ansible 正确地理解我们正在尝试加载使用`ansible-vault`加密的`variables`文件，但我们必须手动告诉它密码才能继续。有多种指定 vault 密码的方法（稍后会详细介绍），但为了简单起见，请尝试运行以下命令，并在提示时输入您的 vault 密码：

```
$ ansible-playbook -i hosts vaultplaybook.yml --ask-vault-pass
Vault password:

PLAY [A play that makes use of an Ansible Vault] *******************************

TASK [Gathering Facts] *********************************************************
ok: [frt01.example.com]
ok: [frt02.example.com]

TASK [Tell me a secret] ********************************************************
ok: [frt01.example.com] => {
 "msg": "Your secret data is: Ansible is cool!"
}
ok: [frt02.example.com] => {
 "msg": "Your secret data is: Ansible is cool!"
}

PLAY RECAP *********************************************************************
frt01.example.com : ok=2 changed=0 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
frt02.example.com : ok=2 changed=0 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
```

成功！Ansible 解密了我们的 vault 文件，并将变量加载到了 playbook 中，我们可以从我们创建的`debug`语句中看到。当然，这违背了使用 vault 的目的，但这是一个很好的例子。

这是使用 vault 的一个非常简单的例子。您可以指定密码的多种方式；您不必在命令行上提示输入密码-可以通过包含 vault 密码的纯文本文件提供，也可以通过脚本在运行时从安全位置获取密码（考虑一个动态清单脚本，只返回密码而不是主机名）。`ansible-vault`工具本身也可以用于编辑、查看和更改 vault 文件中的密码，甚至解密并将其转换回纯文本。Ansible Vault 的用户指南是获取更多信息的绝佳起点（[`docs.ansible.com/ansible/latest/user_guide/vault.html`](https://docs.ansible.com/ansible/latest/user_guide/vault.html)）。

需要注意的一点是，您实际上不必为敏感数据单独创建 vault 文件；您实际上可以将其内联包含在 playbook 中。例如，让我们尝试重新加密我们的敏感数据，以便包含在一个否则未加密的 playbook 中（再次使用`secure`密码作为 vault 的密码，如果您正在测试本书附带的 GitHub 存储库中的示例，请运行以下命令在您的 shell 中（它应该产生类似于所示的输出）：

```
$ ansible-vault encrypt_string 'Ansible is cool!' --name secretdata
New Vault password:
Confirm New Vault password:
secretdata: !vault |
 $ANSIBLE_VAULT;1.1;AES256
 34393431303339353735656236656130336664666337363732376262343837663738393465623930
 3366623061306364643966666565316235313136633264310a623736643362663035373861343435
 62346264313638656363323835323833633264636561366339326332356430383734653030306637
 3736336533656230380a316364313831666463643534633530393337346164356634613065396434
 33316338336266636666353334643865363830346566666331303763643564323065
Encryption successful
```

您可以将此命令的输出复制并粘贴到 playbook 中。因此，如果我们修改我们之前的例子，它将如下所示：

```
---
- name: A play that makes use of an Ansible Vault
  hosts: frontends

  vars:
    secretdata: !vault |
          $ANSIBLE_VAULT;1.1;AES256
          34393431303339353735656236656130336664666337363732376262343837663738393465623930
          3366623061306364643966666565316235313136633264310a623736643362663035373861343435
          62346264313638656363323835323833633264636561366339326332356430383734653030306637
          3736336533656230380a316364313831666463643534633530393337346164356634613065396434
          33316338336266636666353334643865363830346566666331303763643564323065

  tasks:
    - name: Tell me a secret
      debug:
        msg: "Your secret data is: {{ secretdata }}"
```

现在，当您以与之前完全相同的方式运行此 playbook（使用用户提示指定 vault 密码）时，您应该看到它的运行方式与我们使用外部加密的`variables`文件时一样：

```
$ ansible-playbook -i hosts inlinevaultplaybook.yml --ask-vault-pass
Vault password:

PLAY [A play that makes use of an Ansible Vault] *******************************

TASK [Gathering Facts] *********************************************************
ok: [frt02.example.com]
ok: [frt01.example.com]

TASK [Tell me a secret] ********************************************************
ok: [frt01.example.com] => {
 "msg": "Your secret data is: Ansible is cool!"
}
ok: [frt02.example.com] => {
 "msg": "Your secret data is: Ansible is cool!"
}

PLAY RECAP *********************************************************************
frt01.example.com : ok=2 changed=0 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
frt02.example.com : ok=2 changed=0 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
```

Ansible Vault 是一个强大而多功能的工具，可以在静止状态下加密您的敏感 playbook 数据，并且（只要小心）应该使您能够在不留下密码或其他敏感数据的情况下无人值守地运行大部分 playbook。这就结束了本节和本章；希望对您有所帮助。

# 总结

Ansible 具有许多高级功能，可以让您在各种场景中运行 playbook，无论是在受控方式下升级服务器集群，还是在安全的隔离网络上使用设备，或者通过提示和标记来控制 playbook 流程。Ansible 已被大量且不断增长的用户群体采用，因此它的设计和演变都是围绕解决现实世界问题而展开的。我们讨论的大多数 Ansible 的高级功能都是围绕着解决现实世界问题展开的。

在本章中，您学习了如何在 Ansible 中异步运行任务，然后了解了运行 playbook 升级集群的各种功能，例如在小批量清单主机上运行任务，如果一定比例的主机失败则提前失败，将任务委派给特定主机，甚至无论清单（或批量）大小如何都运行任务一次。您还学习了在本地运行 playbook 与在`localhost`上运行 playbook 之间的区别，以及如何使用 SSH 代理自动化在`堡垒`主机上的隔离网络上的任务。最后，您学习了如何处理敏感数据，而不是以未加密的形式存储它，可以通过在运行时提示用户或使用 Ansible Vault 来实现。您甚至学习了如何使用标记来运行 playbook 任务的子集。

在下一章中，我们将更详细地探讨我们在本章中简要提到的一个主题 - 使用 Ansible 自动化网络设备管理。

# 问题

1.  哪个参数允许您配置在批处理中失败的最大主机数，然后播放被中止？

A）`百分比`

B）`最大失败`

C）`最大失败百分比`

D）`最大百分比`

E）`失败百分比`

1.  真或假 - 您可以使用`--connect=local`参数在本地运行任何 playbooks 而不使用 SSH：

A）真

B）假

1.  真或假 - 为了异步运行 playbook，您需要使用`async`关键字：

A）真

B）假

# 进一步阅读

如果您安装了 Passlib，这是 Python 2 和 3 的密码哈希库，`vars_prompt`将使用任何加密方案（如`descrypt`，`md5crypt`，`sha56_crypt`等）进行加密：

+   [`passlib.readthedocs.io/en/stable/`](https://passlib.readthedocs.io/en/stable/)
