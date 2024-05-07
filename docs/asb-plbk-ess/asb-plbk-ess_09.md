# 第八章。使用 Vault 加密数据

使用变量，我们学习了如何分离数据和代码。通常提供的数据是敏感的，例如，用户密码，数据库凭据，API 密钥和其他组织特定信息。Ansible-playbooks 作为源代码，通常存储在版本控制仓库中，如 **git**，这使得在协作环境中保护这些敏感信息变得更加困难。从 1.5 版本开始，Ansible 提供了一个称为 **vault** 的解决方案，用于安全地存储和检索此类敏感信息，使用经过验证的加密技术。使用 vault 的目的是加密数据，然后可以自由地与版本控制系统（如 git）共享，而不会泄露值。

在本章中，我们将学习以下主题：

+   了解 Ansible-vault

+   使用 Ansible-vault 保护数据

+   加密、解密和重新生成密钥操作

# Ansible-vault

Ansible 提供了一个名为 Ansible-vault 的实用程序，顾名思义，让您安全地管理数据。Ansible-vault 实用程序可以让您通过启动编辑器界面创建一个加密文件，或者加密现有文件。在任何一种情况下，它都会要求输入一个 vault 密码，然后使用该密码使用 AES 密码对数据进行加密。加密内容可以存储在版本控制系统中，而不会泄露。由于 AES 基于共享密码，解密时需要提供相同的密码。提供密码有两种选项，一种是在启动 Ansible 时运行 `--ask-vault-pass` 选项以提示输入密码，另一种是使用 `--vault-password-file` 选项提供包含密码的文件路径。

## 高级加密标准

**高级加密标准**（**AES**）是一种基于 **Rijndael** 对称分组密码的加密标准，得名于两位比利时密码学家 —— Vincent Rijmen 和 Joan Daemen，并由美国国家标准与技术研究院（**NIST**）在 2001 年首次建立。AES 是美国政府用来共享机密信息的算法，并且是最流行的对称密钥密码算法。AES 也是第一个由 **国家安全局**（**NSA**）批准的公开可访问的密码。

作为一个开放和流行的标准，Ansible 使用 AES 密码，密钥长度为 256 位，用于使用 vault 加密数据。

## 使用 Vault 加密什么？

Ansible-vault 可以加密任何结构化数据。由于 YAML 本身是一种结构化语言，几乎你为 Ansible 编写的一切都符合这个标准。以下是可以使用 vault 加密的内容的指示：

+   最常见的是，我们加密变量，可能包括以下内容：

    +   角色中的变量文件，例如，`vars` 和 `defaults`

    +   存货变量，例如，`host_vars`，`group_vars`

    +   使用 `include_vars` 或 `vars_files` 包含的变量文件

    +   通过 `-e` 选项传递给 Ansible-playbook 的变量文件，例如，`-e @vars.yml` 或 `-e @vars.json`

+   由于任务和处理程序也是 JSON 数据，因此可以使用 vault 对其进行加密。但这应该很少见。建议您加密变量并在任务和处理程序中引用它们。

以下是不能使用 vault 加密的指针：

+   由于 vault 的加密单位是文件，因此无法加密部分文件或值。您可以加密完整文件或不加密。

+   文件和模板无法加密，因为它们可能与 JSON 或 YML 不同。

以下数据是加密的良好候选对象：

+   凭证，例如，数据库密码和应用凭证

+   API 密钥，例如，AWS 访问密钥和秘密密钥

+   用于 web 服务器的 SSL 密钥

+   部署的私有 SSH 密钥

# 使用 Ansible-vault

以下表列出了 Ansible-vault 实用程序提供的所有子命令：

| 子命令 | 描述 |
| --- | --- |
| `create` | 这将使用编辑器从头开始创建一个加密文件。在运行命令之前，需要设置编辑器环境变量。 |
| `edit` | 这将使用编辑器编辑现有的加密文件，而不解密内容。 |
| `encrypt` | 这将使用结构化数据加密现有文件。 |
| `decrypt` | 这将解密文件。请谨慎使用，并且不要将解密后的文件提交到版本控制中。 |
| `rekey` | 这会更改用于加密或解密的密钥或密码。 |

## 加密数据

让我们使用 Ansible-vault 执行一些操作。我们将从创建一个加密文件开始。要从头开始创建新文件，Ansible-vault 使用 `create` 子命令。在使用此子命令之前，重要的是要在环境中设置一个编辑器，如下所示：

```
# setting up vi as editor
$ export EDITOR=vi
# Generate a encrypted file
$ ansible-vault create aws_creds.yml
Vault password:
Confirm Vault password:

```

运行此命令会打开一个由编辑器环境变量指定的编辑器。下面是您可能创建的用于以访问密钥和秘密密钥形式存储 AWS 用户凭证的 `aws_creds.yml` 文件示例。然后，这些密钥将用于向 Amazon web services 云平台发出 API 调用。保存此文件并退出编辑器将生成一个加密文件：

![加密数据](img/B03800_08_01.jpg)

你可以通过运行以下命令检查创建的文件类型及其内容：

```
# Check file type and content
$ file aws_creds.yml
aws_creds.yml: ASCII text
$ cat aws_creds.yml
$ANSIBLE_VAULT;1.1;AES256
64616236666362376630366435623538336565393331333331663663636237636335313234313134
3337303865323239623436646630336239653864356561640a363966393135316661636562333932
61323932313230383433313735646438623032613635623966646232306433383335326566343333
3136646536316261300a616438643463656263636237316136356163646161313365336239653434
36626135313138343939363635353563373865306266363532386537623463623464376134353863
37646638636231303461343564343232343837356662316262356537653066356465353432396436
31336664313661306630653765356161616266653232316637653132356661343162396331353863
34356632373963663230373866313961386435663463656561373461623830656261636564313464
37383465353665623830623363353161363033613064343932663432653666633538

```

## 更新加密数据

要更新添加到加密文件中的 AWS 密钥，可以稍后使用 Ansible-vault 的 `edit` 子命令，如下所示：

```
$ ansible-vault edit aws_creds.yml
Vault password:

```

`edit` 命令执行以下操作：

1.  提示输入密码

1.  使用 AES 对称密码，即时解密文件

1.  打开编辑器界面，允许您更改文件的内容

1.  将文件保存后再次加密

还有另一种更新文件内容的方法。您可以按如下方式解密文件：

```
$ ansible-vault decrypt aws_creds.yml
Vault password:
Decryption successful

```

更新后，该文件可以像之前学过的那样再次加密。

## 旋转加密密钥

作为良好的安全实践，经常更改 Ansible-vault 使用的加密密钥是个好主意。当这种情况发生时，重新为之前使用 vault 加密的所有文件重新生成密钥是至关重要的。Ansible vault 提供了一个 `rekey` 子命令，可以如下使用：

```
$ ansible-vault rekey aws_creds.yml
Vault password:
New Vault password:
Confirm New Vault password:
Rekey successful

```

它要求输入当前密码，然后允许您指定并确认新密码。请注意，如果您正在使用版本控制管理此文件，则还需要提交更改。即使实际内容未更改，重新生成操作也会更新所创建的结果文件，该文件是我们存储库的一部分。

# 加密数据库凭据

早些时候在创建数据库用户时，我们在 `group_vars` 中以明文提供了密码。这可能是一个潜在的威胁，特别是当提交到版本控制存储库时。让我们加密它。我们将使用 `encrypt` 子命令，因为我们已经有了一个变量文件。

由于我们使用 `group_vars` 组提供数据库凭据，因此我们将如下加密 `group_vars/all` 文件：

```
$ ansible-vault encrypt group_vars/all
Vault password:
Confirm Vault password:
Encryption successful

```

对于加密，Ansible-vault 要求用户输入密码或密钥。使用此密钥，vault 加密数据并用加密内容替换文件。以下图表显示了 `group_vars/all` 文件的左侧明文内容和等效的右侧加密内容：

![加密数据库凭据](img/B03800_08_02.jpg)

现在此文件可以安全地检入版本控制系统并共享。但是，以下是用户应该注意的注意事项：

+   与纯文本不同，结果文件是以加密格式存储的。不可能获得不同的文件格式，例如 `git diff`，以便在提交到版本控制时比较更改。

+   无法直接在此文件上使用 `grep`、`sed` 或任何文本搜索或操作程序。唯一的方法是先解密它，运行文本操作工具，然后再加密回去。

### 提示

确保您为要在一个 Ansible-playbook 运行中解密的所有文件使用相同的密码。Ansible 一次只能接受一个密码值，如果在同一 playbook 中使用不同的密码对文件进行加密，则会失败。

现在让我们使用以下命令运行 Ansible playbook：

```
$ ansible-playbook -i customhosts site.yml
ERROR: A vault password must be specified to decrypt /vagrant/chap8/group_vars/all

```

它以错误失败！这是因为我们正在为 playbook 提供加密数据，而没有解密它的密钥。vault 的主要用途是在 Ansible 存储库中保护数据。最终，在运行 playbook 时需要解密这些值。解密密码可以使用 `--ask-vault-pass` 选项指定，如下所示：

```
$ ansible-playbook -i customhosts site.yml --ask-vault-pass

```

这将提示输入 "Vault 密码"，然后继续像往常一样运行 Ansible 代码。

# 使用密码文件

每次输入密码可能不是理想的。通常情况下，您可能还希望自动化启动 Ansible playbook 运行的过程，在这种情况下，交互式方式是不可行的。可以通过将密码存储在文件中并将文件提供给 Ansible playbook 运行来避免这种情况。密码应作为单行字符串提供在此文件中。

让我们创建一个密码文件并使用正确的权限保护它：

```
$ echo "password" > ~/.vault_pass
(replace password with your own secret)
$ chmod 600 ~/.vault_pass

```

### 提示

当保险库密码存储为明文时，任何访问此文件的人都可以解密数据。确保密码文件受到适当权限的保护，并且不添加到版本控制中。如果决定对其进行版本控制，请使用**gpg**或等效措施。

现在可以将此文件提供给 Ansible playbook，如下所示：

```
$ ansible-playbook -i customhosts site.yml --vault-password-file ~/.vault_pass

```

# 将保险库密码文件选项添加到 Ansible 配置

使用版本 1.7，还可以将`vault_password_file`选项添加到`ansible.cfg`文件的默认部分。

考虑以下：

```
[defaults]
  vault_password_file = ~/.vault_pass
```

上述选项使您可以自由地不需要每次都指定加密密码或密码文件。让我们看一下以下命令：

```
# launch ansible playbook run with encrypted data
# with vault_password_file option set in the config
$ ansible-playbook -i customhosts site.yml
$ ansible-vault encrypt roles/mysql/defaults/main.yml
Encryption successful
$ ansible-vault decrypt roles/mysql/defaults/main.yml
Decryption successful

```

此外，在从版本 1.7 开始时，可以将脚本提供给`vault_password_file`选项，而不是在文件中存储明文密码。使用脚本时，请确保：

+   脚本上启用了执行位

+   调用此脚本会在标准输出上输出密码。

+   如果脚本提示用户输入，则可以将其发送到标准错误

## 在模板中使用加密数据

您之前了解到，由于模板可能不是结构化文件，如 YAML 或 JSON，因此它无法加密。但是，有一种方法可以向模板添加加密数据。请记住，模板最终是即时生成的，动态内容实际上来自变量，这些变量可以加密。让我们讨论如何通过为 Nginx Web 服务器添加 SSL 支持来实现这一点。

## 为 Nginx 添加 SSL 支持

我们已经设置了一个 Nginx Web 服务器，现在让我们通过以下步骤为默认站点添加 SSL 支持：

1.  我们首先添加变量，如下所示：

    ```
    #file: roles/nginx/defaults/main.yml 
    nginx_ssl: true
    nginx_port_ssl: 443
    nginx_ssl_path: /etc/nginx/ssl
    nginx_ssl_cert_file: nginx.crt
    nginx_ssl_key_file: nginx.key
    ```

1.  让我们也创建自签名的 SSL 证书：

    ```
    $ openssl req -x509 -nodes -newkey rsa:2048 -keyout nginx.key -out nginx.crt

    ```

    上述命令将生成两个文件，`nginx.key`和`nginx.crt`。这些是我们将复制到 Web 服务器的文件。

1.  将这些文件的内容添加到变量中，并创建`group_vars/www`文件：

    ```
    # file: group_vars/www
    ---
    nginx_ssl_cert_content: |
        -----BEGIN CERTIFICATE-----
        -----END CERTIFICATE-----
    nginx_ssl_key_content: |
        -----BEGIN PRIVATE KEY-----
        -----END PRIVATE KEY-----
    ```

    在上述示例中，我们只是添加了将要替换为密钥和证书实际内容的占位符。这些密钥和证书不应暴露在版本控制系统中。

1.  让我们使用保险库加密此文件：

    ```
    $ ansible-vault encrypt group_vars/www
    Encryption successful

    ```

    由于我们已经在配置中提供了保险库密码的路径，因此 Ansible-vault 不会询问密码。

1.  现在让我们创建模板，以添加这些密钥：

    ```
    # filename: roles/nginx/templates/nginx.crt.j2
    {{ nginx_ssl_cert_content }}

    # filename: roles/nginx/templates/nginx.key.j2
    {{ nginx_ssl_key_content }}
    ```

1.  还要将一个虚拟主机`config`文件添加到 SSL 中：

    ```
    # filename: roles/nginx/templates/nginx.key.j2
    server {
      listen {{ nginx_port_ssl }};
      server_name {{ ansible_hostname }};
      ssl on;
      ssl_certificate {{ nginx_ssl_path }}/{{ nginx_ssl_cert_file }};
      ssl_certificate_key {{ nginx_ssl_path }}/{{ nginx_ssl_key_file }};

      location / {
        root {{ nginx_root }};
        index {{ nginx_index }};
      }
    }
    ```

1.  我们还需要创建一个任务文件来配置 SSL 站点，该文件将创建所需的目录、文件和配置：

    ```
    ---
    # filename: roles/nginx/tasks/configure_ssl.yml
     - name: create ssl directory
        file: path="{{ nginx_ssl_path }}" state=directory owner=root group=root
     - name: add ssl key 
        template: src=nginx.key.j2 dest="{{ nginx_ssl_path }}/nginx.key" mode=0644
     - name: add ssl cert 
        template: src=nginx.crt.j2 dest="{{ nginx_ssl_path }}/nginx.crt" mode=0644
     - name: create ssl site configurations 
        template: src=default_ssl.conf.j2 dest="{{ nginx_ssl_path }}/default_ssl.conf" mode=0644
        notify:
        - restart nginx service
    ```

1.  最后，让我们根据`nginx_ssl var`参数是否设置为 true 来选择性地调用此任务：

    ```
    # filename: roles/nginx/tasks/main.yml
     - include: configure_ssl.yml
        when: nginx_ssl
    ```

1.  现在，按照以下方式运行 playbook：

    ```
    $ ansible-playbook -i customhosts  site.yml

    ```

这应该配置在端口`443`上运行的默认 SSL 站点，使用自签名证书。现在，您应该能够使用`https`安全协议打开 Web 服务器地址，如下所示：

![为 Nginx 添加 SSL 支持](img/B03800_08_03.jpg)

当然，由于我们的证书是自签名的，而不是由指定的认证机构提供的，应该显示警告。

# 复习问题

你认为你已经足够理解这一章了吗？尝试回答以下问题来测试你的理解：

1.  为什么需要加密提供给 Ansible playbooks 的数据？

1.  AES 是什么，对称密钥密码是什么？

1.  更新之前使用 vault 加密的文件的两种方法是什么？

1.  添加到 Ansible 配置文件的参数，使其了解保险库密码文件的位置是什么？

# 摘要

在本章中，您学习了如何使用 Ansible-vault 对传递给 playbooks 的数据进行安全保护。我们从加密数据的需求开始，讲解了 vault 的工作原理以及它使用的密码。然后，我们开始深入了解 Ansible-vault 实用程序以及创建加密文件、解密、重新密钥等基本操作。您还学习了如何通过在持有数据库凭据的`vars`文件上运行 Ansible-vault 来加密现有文件。最后，我们为 Nginx 添加了 SSL 支持，您学会了如何使用 vault 安全地存储 Web 服务器的私钥和证书，并使用模板将它们复制。请注意，Ansible vault 提供了一种安全地向 Ansible 模块提供数据的方式。除了使用 vault 之外，还建议采取其他系统安全措施，这不在本文的讨论范围内。

在了解了 vault 之后，在下一章中，我们将开始学习使用 Ansible 管理多个环境（如开发、演示和生产）的各种方法。这些环境通常映射到软件开发工作流程。
