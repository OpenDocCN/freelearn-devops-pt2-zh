# 第三章：深入了解 Ansible 模块

已经学习了基础知识，本章将带您深入了解 Ansible 的更高级主题，例如：

+   使模块支持在干扰模式下安全执行

+   了解如何在 Ansible 模块中解析参数

+   处理复杂参数和数据结构

+   一个现实生活场景，您可以利用 Ansible 的强大功能创建一个定制模块以满足您的需求

# 干扰（检查模式）

因此，您决定编写自己的模块，根据用户输入对系统进行一些配置更改。考虑到代码必须在生产环境中运行，能够运行尚未发布的配置的模拟非常重要。您不仅可能希望在应用它们之前知道您的配置是否正确，而且您可能还想了解 Playbook 执行将涉及哪些更改。

由于 Ansible 不知道模块执行的后果，它只是按照 Playbook 的指示。在干扰模式下，它将简单地打印出所有它将执行的模块并跳过实际执行。如果模块不支持检查模式，则在执行期间检查模式下简单地跳过该模块。

模块对系统状态或目标机器的任何更改的详细信息很有用。但是，Ansible 只能通过要求模块执行模拟并返回状态更改确认来了解这一点。您的 Ansible Playbook 中可能有一些任务使用一些返回输出的模块。这些可能存储在变量中，并且后续模块执行取决于它们。

为了告诉 Ansible 模块支持检查模式并且可以在干扰模式下安全运行，只需在 Ansible 模块中将`supports_check_mode`标志设置为 true。可以按如下方式完成：

```
module = AnsibleModule(
    argument_spec = dict(
        # List of arguments
    ),  
    supports_check_mode = True
)
```

模块中的前述代码使模块能够以干扰模式执行。您可以使用以下命令在检查模式下运行您的 Ansible Playbook：

```
**ansible-playbook playook.yml --check**

```

这将对所有支持检查模式的模块进行干扰，并在不实际进行更改的情况下报告目标机器上可能进行的任何更改。

# 加载模块

在深入编写 Ansible 模块之前，有必要了解 Ansible 在运行时如何加载模块。了解模块如何加载到 Ansible 中可以让你理解代码流程并调试可能在运行时发生的问题。要理解这一点，你必须了解 Ansible playbook 是如何执行的。

正如你已经知道的，Ansible playbooks 是使用`ansible-playbook`二进制文件执行的，它接受一些参数，如清单文件和要运行的 Ansible play。如果你查看`ansible-playbook`的源代码，你会注意到以下导入：

```
import ansible.constants as C
```

`constants.py`文件是将配置加载到 Ansible 中的主要文件之一。它包含各种配置，如模块和插件将被加载到 Ansible 的默认路径。

这个文件负责定义 Ansible 加载配置的顺序。配置加载到 Ansible 中的默认顺序是：

1.  **ENV**：环境变量。

1.  **CWD**：当前工作目录（执行 Ansible playbook 的目录）。

1.  **主页**：然后从用户的主目录中的配置文件中加载配置。此配置文件名为`~/.ansible.cfg`。

1.  **全局配置文件**：Ansible 将全局配置文件放在`/etc/ansible/ansible.cfg`中。

Ansible 使用在前述顺序中找到的配置。

该文件还设置了一些默认配置值，这些值对于 Ansible 执行 playbook 是必需的。其中一些默认配置值是：

+   `forks`：默认的 forks 数量设置为`5`

+   `remote_user`：这个值设置为控制节点上的活动用户

+   `private_key_file`：设置要用于与目标主机通信的默认私钥

+   `Timeout`：默认值设置为`10`

# 利用 Ansible

上一章介绍了`AnsibleModule`的样板，它允许你编写自己的 Ansible 模块，接受参数并返回结果。在继续开发 Ansible 模块之前，本节将从代码层面详细探讨`AnsibleModule`的样板。

## 深入了解 AnsibleModule 样板

正如在前一章中讨论的那样，`AnsibleModule`的样板可以通过简单地导入`ansible.module_utils.basic`语句来使用。

一旦为`AnsibleModule`类创建了对象，对象的一些属性就会被设置，包括在创建`AnsibleModule`对象时指定的`argument_spec`属性。默认情况下，`supports_check_mode`属性设置为`false`，`check_invalid_arguments`设置为`true`。

`AnsibleModule`类使用`load_params`方法将参数和参数加载到`params`变量中。以下是`load_params`方法的源代码：

```
def _load_params(self):
    ''' read the input and return a dictionary and the arguments string '''
    args = MODULE_ARGS
    items   = shlex.split(args)
    params = {}
    for x in items:
        try:
            (k, v) = x.split("=",1)
        except Exception, e:
            self.fail_json(msg="this module requires key=value arguments (%s)" % (items))
        if k in params:
            self.fail_json(msg="duplicate parameter: %s (value=%s)" % (k, v))
        params[k] = v
    params2 = json_dict_unicode_to_bytes(json.loads(MODULE_COMPLEX_ARGS))
    params2.update(params)
    return (params2, args)
```

正如您所看到的，`params`是一个字典。Python 允许您使用`get`方法从字典中读取与键对应的值。因此，如果您需要访问任何参数，可以简单地在`params`字典变量上使用`get`方法。这就是 Ansible 读取和接受模块中的参数的方式。

现在您已经学会了如何开发模块，接受参数并处理错误，让我们在实际情况中实现这些知识。

所以，假设您拥有一个庞大的基础架构，一切都运行良好。您已经有一个很好的配置管理系统，以及一个监控系统，可以跟踪所有机器的情况，并在发生故障时通知您。一切都很顺利，直到有一天，您需要审计您的基础架构。您需要每台机器的详细信息，比如 BIOS 详细信息、制造商和序列号等系统规格。

一种简单的解决方案是在每台机器上运行`dmidecode`并整理收集到的数据。嗯，在单独的机器上运行`dmidecode`并整理细节是一件麻烦的事情。让我们利用 Ansible 的力量来处理这种情况。

学会了如何创建模块后，您可以使用 Python 库`dmidecode`并编写自己的模块，然后在整个基础架构上运行。额外的好处是您可以将数据以机器可解析的形式，比如 JSON，保存下来，以后可以用来生成报告。

让我们将模块命名为`dmidecode`，并将其放在 Ansible 剧本根目录中的`library`目录中。以下是`dmidecode`模块的源代码：

```
import dmidecode
import json

def get_bios_specs():
    BIOSdict = {}
    BIOSlist = []
    for item in dmidecode.bios().values():
        if type(item) == dict and item['dmi_type'] == 0:
            BIOSdict["Name"] = str((item['data']['Vendor']))
            BIOSdict["Description"] = str((item['data']['Vendor']))
            BIOSdict["BuildNumber"] = str((item['data']['Version']))
            BIOSdict["SoftwareElementID"] = str((item['data']['BIOS Revision']))
            BIOSdict["primaryBIOS"] = "True"
            BIOSlist.append(BIOSdict)
    return BIOSlist

def get_proc_specs():
    PROCdict = {}
    PROClist = []
    for item in dmidecode.processor().values():
        if type(item) == dict and item['dmi_type'] == 4:
            PROCdict['Vendor'] = str(item['data']['Manufacturer']['Vendor'])
            PROCdict['Version'] = str(item['data']['Version'])
            PROCdict['Thread Count'] = str(item['data']['Thread Count'])
            PROCdict['Characteristics'] = str(item['data']['Characteristics'])
            PROCdict['Core Count'] = str(item['data']['Core Count'])
            PROClist.append(PROCdict)
    return PROClist

def get_system_specs():
    SYSdict = {}
    SYSlist = []
    for item in dmidecode.system().values():
        if item['dmi_type'] == 1:
            SYSdict['Manufacturer'] = str(item['data']['Manufacturer'])
            SYSdict['Family'] = str(item['data']['Family'])
            SYSdict['Serial Number'] = str(item['data']['Serial Number'])
            SYSlist.append(SYSdict)
    return SYSlist

def main():
    module = AnsibleModule(
        argument_spec = dict(
            save = dict(required=False, default=False, type='bool'),
        )
    )
    # You can record all data you want. For demonstration purpose, the #example records only the first record.
    dmi_data = json.dumps({
        'Hardware Specs' : {
            'BIOS' : get_bios_specs()[0],
            'Processor' : get_proc_specs()[0],
            'System' : get_system_specs()[0]
        }
    })
    save = module.params.get('save')
    if save:
        with open('dmidecode.json', 'w') as dfile:
            dfile.write(str(dmi_data))
    module.exit_json(changed=True, msg=str(dmi_data))

from ansible.module_utils.basic import *
main()
```

正如您所看到的，我们正在收集处理器规格、BIOS 规格和系统规格等数据；您可以根据个人需求随时扩展模块。

该模块接受来自用户的布尔参数`save`，如果设置为`true`，将把结果写入远程机器上的 JSON 文件。

您可能会注意到模块在开头有一个导入行`import dmidecode`。该语句导入了`dmidecode` Python 库。该库由`python-dmidecode`软件包提供。由于模块依赖于`dmidecode` Python 库，因此需要在目标机器上安装该库。这可以在 Ansible playbook 中处理。

依赖项可以在`global_vars`文件中指定，并且可以在 Ansible playbook 中使用变量名。这样做是为了防止在依赖项发生更改时对 Ansible play 进行更改。可以在`global_vars`目录中指定如下：

`global_vars/all`

```
# Dependencies
dependencies:
    - python-dmidecode
    - python-simplejson
```

因此，Ansible 模块已准备就绪，并且依赖项已得到处理。现在，您需要创建 Ansible play，该 play 将在目标机器上执行`dmidecode`模块。让我们将 Ansible 命名为`play dmidecode.yml`。

```
---
- hosts: remote
  user: root

  tasks:
    - name: Install dependencies
      yum: name={{ item }} state=latest
      with_items:
        - "{{ dependencies }}"

    - name: Test dmidecode
      action: dmidecode save=True
      register: dmi_data

   - debug: var=dmi_data['msg']
```

执行 Ansible playbook 将在远程主机组上运行`dmidecode`模块。由于`save`设置为`true`，这将在远程主机上创建一个包含所需信息的`dmidecode.json`文件。

# 复杂参数

由于 Ansible 模块只是另一种可以接受和解析参数的代码，可能会有一个问题，即它是否能够处理复杂的变量集。尽管 Ansible 用作部署、编排和配置管理工具，但它设计用于处理简单参数，仍然能够处理复杂变量。这是一个高级主题，由于通常不使用，本节将简要介绍它。

您已经学会了如何向 Ansible 模块传递参数。但是，复杂参数的处理方式不同。

## 读取复杂参数

让我们以复杂变量`complex_var`为例，通常情况下，我们在`group_vars/all`中定义它。

```
# Complex Variable
complex_var:
    key0: value0
    key1:
      - value1
      - value2
```

前面的变量是字典类型（即键值对）。为了使 Ansible 模块解析这种类型的参数，我们需要对模块中传递复杂变量的方式和它们的解析方式进行一些更改。我们编写一个自定义模块，接受这个复杂变量作为参数，并打印与关联键对应的值。我们将该模块命名为`complex`。

以下是`complex.py`模块的代码：

**Ansible 模块：** `library/complex.py`

```
#!/usr/bin/python

def main():
    module = AnsibleModule(
        argument_spec = dict(
            key0 = dict(required=True),
            key1 = dict(required=False, default=[])
        )
    )
    module.exit_json(changed=False, out='%s, %s' %
        (module.params['key0'], module.params['key1']))

from ansible.module_utils.basic import *
main()
```

前面的模块接受复杂变量并打印它们对应的键的关联值。复杂变量如何传递给 Ansible 模块在 Ansible play 中指定。

以下是 Ansible playbook，它接受复杂参数并将它们传递给复杂模块：

**Ansible play:** `complex.yaml`

```
---
- hosts: localhost
  user: rdas

  tasks:
    - name: print complex variable key values
      action: complex
      args: '{{ complex_var }}'
      register: res

    - debug: msg='{{res.out}}'
```

当执行 Ansible playbook 时，分别打印与键`key0`和`key1`关联的值。

# 总结

在本章中，您了解了如何通过引入`supports_check_mode`标志使您的模块支持干运行。您还了解了在 Ansible 中如何处理参数。本章涵盖了一个真实场景，其中使用自定义 Ansible 模块对基础设施进行硬件审计。本章还简要介绍了如何在 Ansible 中处理复杂变量。

在下一章中，您将了解 Ansible 插件，为什么它们是必需的，以及它们如何适用于一般的 Ansible 结构。本章还将介绍 Python 插件 API。
