# 第五章：使用 OWASP ZAP 进行自动化 Web 应用程序安全测试

OWASP **Zed Attack Proxy**（通常称为**ZAP**）是最受欢迎的网络应用安全测试工具之一。它具有许多功能，使其可用于手动安全测试；在一些调整和配置后，它也很好地适用于**持续集成/持续交付**（**CI/CD**）环境。

有关该项目的更多详细信息可以在[`www.owasp.org/index.php/OWASP_Zed_Attack_Proxy_Project`](https://www.owasp.org/index.php/OWASP_Zed_Attack_Proxy_Project)找到。

**开放式网络应用安全项目**（**OWASP**）是一个致力于改善软件安全性的全球性非营利慈善组织。在[`www.owasp.org`](https://www.owasp.org)了解更多关于 OWASP 项目和资源的信息。

OWASP ZAP 在一个包中包含许多不同的工具和功能。对于负责对 Web 应用程序进行安全测试的渗透测试员，以下功能至关重要：

| **功能** | **用例** |
| --- | --- |
| 拦截代理 | 这允许我们拦截浏览器中的请求和响应 |
| 主动扫描器 | 自动运行针对目标的网络安全扫描 |
| 被动扫描器 | 从使用蜘蛛工具下载的页面中获取有关安全问题的信息 |
| 蜘蛛 | 在 ZAP 可以攻击应用程序之前，它通过遍历所有可能的网页来创建应用程序的站点地图 |
| REST API | 允许 ZAP 在无头模式下运行，并控制运行自动扫描器、蜘蛛和获取结果 |

正如你可能已经猜到的，在本章中，为了进行安全自动化，我们将以无头模式调用 ZAP，并使用其提供的 API 接口来进行扫描和安全测试。

ZAP 是基于 Java 的软件。使用它的典型方式将涉及以下内容：

+   **Java 运行时环境**（**JRE**）7 或更高版本安装在您选择的操作系统中（macOS、Windows、Linux）

+   使用官方下载页面的软件包管理器、安装程序安装 ZAP

你可以在此处找到最新的更新稳定链接：[`github.com/zaproxy/zaproxy/wiki/Downloads`](https://github.com/zaproxy/zaproxy/wiki/Downloads)。

虽然我们可以构建一个执行此操作的操作手册，但开发者世界正在向 CI/CD 和持续安全的概念发展。一个可以在需要时引导稳定版本的 ZAP 的方法将是理想的。

实现这一目标的最佳方法是将 OWASP ZAP 用作容器。事实上，这是 Mozilla 在 CI/CD 管道中使用 ZAP 来验证每个发布的基线安全控制的设置方式。

如果您想知道 Mozilla 和 OWASP ZAP 之间的联系，Simon Bennetts 领导了 OWASP ZAP 项目并在 Mozilla 工作。 阅读他关于 ZAP 基线扫描的博客文章：[`blog.mozilla.org/security/2017/01/25/setting-a-baseline-for-web-security-controls/`](https://blog.mozilla.org/security/2017/01/25/setting-a-baseline-for-web-security-controls/)。

# 安装 OWASP ZAP

在本章中，我们将使用 OWASP ZAP 作为一个容器，这需要在主机操作系统中运行容器运行时。 OWASP ZAP 团队每周通过 Docker Hub 发布 ZAP Docker 镜像。 根据标签拉取 Docker 镜像的方法在现代 DevOps 环境中很受欢迎，因此我们谈论与其相关的自动化是有意义的。

官方 ZAP 现在通过 Docker Hub 的 Docker 容器提供稳定版和每周发布版本：[`github.com/zaproxy/zaproxy/wiki/Docker`](https://github.com/zaproxy/zaproxy/wiki/Docker)。

# 安装 Docker 运行时

**Docker** 是供开发人员和系统管理员构建、发布和运行分布式应用程序的开放平台，无论是在笔记本电脑、数据中心虚拟机还是云中。 要了解有关 Docker 的更多信息，请参阅 [`www.docker.com/what-docker`](https://www.docker.com/what-docker)。

以下 Playbook 将在 Ubuntu 16.04 中安装 Docker Community Edition 软件：

```
- name: installing docker on ubuntu
  hosts: zap
  remote_user: "{{ remote_user_name }}"
  gather_facts: no
  become: yes
  vars:
    remote_user_name: ubuntu
    apt_repo_data: "deb [arch=amd64] https://download.docker.com/linux/ubuntu xenial stable"
    apt_gpg_key: https://download.docker.com/linux/ubuntu/gpg

  tasks:
    - name: adding docker gpg key
      apt_key:
        url: "{{ apt_gpg_key }}"
        state: present

    - name: add docker repository
      apt_repository:
        repo: "{{ apt_repo_data }}"
        state: present

    - name: installing docker-ce
      apt:
        name: docker-ce
        state: present
        update_cache: yes
    - name: install python-pip
      apt:
        name: python-pip
        state: present
    - name: install docker-py
      pip:
        name: "{{ item }}"
        state: present

      with_items:
        - docker-py

```

Docker 需要 64 位版本的操作系统和 Linux 内核版本大于或等于 3.10。 Docker 运行时也适用于 Windows 和 macOS。 对于本章的目的，我们将使用基于 Linux 的容器。 因此运行时可以在 Windows 上，但在其中运行的容器将是基于 Linux 的。 这些是可用于使用的标准 OWASP ZAP 容器。

# OWASP ZAP Docker 容器设置

我们将在这里使用的两个新模块来处理 Docker 容器是 `docker_image` 和 `docker_container`。

这些模块要求您使用 2.1 及更高版本的 Ansible。现在是检查您的 Ansible 版本的好时机，可以使用 `—version` 标志。

如果您需要使用 `pip` 获取最新稳定版本，请运行以下命令：

```
pip install ansible --upgrade 
```

由于需要从互联网下载约 1GB 的数据，以下操作可能需要一些时间才能完成：

```
- name: setting up owasp zap container
  hosts: zap
  remote_user: "{{ remote_user_name }}"
  gather_facts: no
  become: yes
  vars:
    remote_user_name: ubuntu
    owasp_zap_image_name: owasp/zap2docker-weekly

  tasks:
    - name: pulling {{ owasp_zap_image_name }} container
      docker_image:
        name: "{{ owasp_zap_image_name }}"

    - name: running owasp zap container
      docker_container:
        name: owasp-zap
        image: "{{ owasp_zap_image_name }}"
        interactive: yes
        state: started
        user: zap
        command: zap.sh -daemon -host 0.0.0.0 -port 8090 -config api.disablekey=true -config api.addrs.addr.name=.* -config api.addrs.addr.regex=true
        ports:
          - "8090:8090"
</span>
```

在以下配置中，我们说 `api.disablekey=true`，这意味着我们不使用任何 API 密钥。 这可以通过给定特定的 API 密钥来覆盖。 `api.addrs.addr.name=.*` 和 `api.addrs.addr.regex=true` 将允许所有 IP 地址连接到 ZAP API。 有关 ZAP API 密钥设置的更多信息，请参阅 [`github.com/zaproxy/zaproxy/wiki/FAQapikey`](https://github.com/zaproxy/zaproxy/wiki/FAQapikey)。

您可以通过导航到 `http://ZAPSERVERIPADDRESS:8090` 来访问 ZAP API 界面：

![](img/cd7413f7-b766-41af-b24f-9d79ce0dbc07.png)

OWASP ZAP API Web UI

# 用于处理容器的专用工具 - Ansible Container

目前，我们正在使用 Docker 模块执行容器操作。新工具 `ansible-container` 提供了一个 Ansible 中心化的工作流程，用于构建、运行、测试和部署容器。

这使我们能够使用现有的播放构建、推送和运行容器。Dockerfiles 就像编写 shell 脚本一样，因此，`ansible-container` 将允许我们将这些 Dockerfiles 编码化，并使用现有的播放构建它们，而不是编写复杂的脚本。

`ansible-container` 支持各种编排工具，如 Kubernetes 和 OpenShift。它还可以用于将构建的映像推送到私有注册表，如 Google Container Registry 和 Docker Hub。

在[`docs.ansible.com/ansible-container/`](https://docs.ansible.com/ansible-container/)了解更多关于`ansible-container`的信息。

# 配置 ZAP 基线扫描

ZAP 基线扫描是在 ZAP Docker 镜像中可用的脚本。

更多关于 OWASP ZAP 基线扫描的详细信息可以在[`github.com/zaproxy/zaproxy/wiki/ZAP-Baseline-Scan`](https://github.com/zaproxy/zaproxy/wiki/ZAP-Baseline-Scan)找到。

此脚本的功能如下：

+   对指定目标运行 ZAP 蜘蛛一分钟，然后执行被动扫描

+   默认情况下，将所有警报报告为警告

+   此脚本旨在在 CI/CD 环境中运行，甚至针对生产站点也是理想的。

在设置和运行 ZAP 基线扫描之前，我们希望运行一个简单易受攻击的应用程序，以便所有使用 ZAP 的扫描和测试都针对该应用程序运行，而不是针对真实世界的应用程序运行扫描，这是未经许可的。

# 运行一个易受攻击的应用容器

我们将使用**Damn Vulnerable Web Services**（**DVWS**）应用程序（更多信息，请访问[`github.com/snoopysecurity/dvws`](https://github.com/snoopysecurity/dvws)）。它是一个带有多个易受攻击的 Web 服务组件的不安全 Web 应用程序，可用于学习真实世界的 Web 服务漏洞。

以下播放将设置运行 DVWS 应用程序的 Docker 容器：

```
- name: setting up DVWS container
  hosts: dvws
  remote_user: "{{ remote_user_name }}"
  gather_facts: no
  become: yes
  vars:
    remote_user_name: ubuntu
    dvws_image_name: cyrivs89/web-dvws

  tasks:
    - name: pulling {{ dvws_image_name }} container
      docker_image:
        name: "{{ dvws_image_name }}"

    - name: running dvws container
      docker_container:
        name: dvws
        image: "{{ dvws_image_name }}"
        interactive: yes
        state: started
        ports:
          - "80:80"

```

一旦播放成功执行，我们可以导航到 `http://DVWSSERVERIP`：

![](img/938c0a0d-3b89-43b5-9fa4-88a534cf121d.png)

DVWS 应用首页

现在，我们已经准备好对 DVWS 应用程序执行我们的 OWASP ZAP 基线扫描，通过运行基线扫描播放。

# 运行 OWASP ZAP 基线扫描

以下播放会针对给定的网站 URL 运行 Docker 基线扫描。它还将基线扫描的输出存储在主机系统中，以 HTML、Markdown 和 XML 格式进行存储：

```
- name: Running OWASP ZAP Baseline Scan
  hosts: zap
  remote_user: "{{ remote_user_name }}"
  gather_facts: no
  become: yes
  vars:
    remote_user_name: ubuntu
    owasp_zap_image_name: owasp/zap2docker-weekly
    website_url: {{ website_url }}
    reports_location: /zapdata/
    scan_name: owasp-zap-base-line-scan-dvws

  tasks:
    - name: adding write permissions to reports directory
      file:
        path: "{{ reports_location }}"
        state: directory
        owner: root
        group: root
        recurse: yes
        mode: 0770

    - name: running owasp zap baseline scan container against "{{ website_url }}"
      docker_container:
        name: "{{ scan_name }}"
        image: "{{ owasp_zap_image_name }}"
        interactive: yes
        auto_remove: yes
        state: started
        volumes: "{{ reports_location }}:/zap/wrk:rw"
        command: "zap-baseline.py -t {{ website_url }} -r {{ scan_name }}_report.html"

    - name: getting raw output of the scan
      command: "docker logs -f {{ scan_name }}"
      register: scan_output

    - debug:
        msg: "{{ scan_output }}"

```

让我们探索前面播放的参数：

+   `website_url` 是要执行基线扫描的域名（或）URL，我们可以通过 `--extra-vars "website_url: http://192.168.33.111"` 从 `ansible-playbook` 命令传递这个参数

+   `reports_location` 是 ZAP 主机的路径，报告存储在其中。

下面的截图是来自 OWASP ZAP 的扫描报告输出：

![](img/576c1909-a7c7-43a8-90e3-93ced983ca3c.png)

OWASP ZAP 基线扫描 HTML 报告

要生成 Markdown 和 XML 格式的报告，请分别添加`-w report.md`和`-x report.xml`到命令。

# 针对 Web 应用程序和网站进行安全测试

到目前为止，我们已经看到了如何使用 OWASP ZAP 容器运行基线扫描。现在我们将看到如何对 Web 应用程序执行主动扫描。主动扫描可能导致应用程序中的漏洞被利用。此外，这种类型的扫描需要额外的配置，包括身份验证和敏感功能。

# 对 DVWS 运行 ZAP 全面扫描

下面的 playbook 将针对 DVWS 应用程序运行全面扫描。现在我们可以看到 playbook 看起来几乎相似，只是发送给命令的标志不同：

```
- name: Running OWASP ZAP Full Scan
  hosts: zap
  remote_user: "{{ remote_user_name }}"
  gather_facts: no
  become: yes
  vars:
    remote_user_name: ubuntu
    owasp_zap_image_name: owasp/zap2docker-weekly
    website_url: {{ website_url }}
    reports_location: /zapdata/
    scan_name: owasp-zap-full-scan-dvws

  tasks:
    - name: adding write permissions to reports directory
      file:

        path: "{{ reports_location }}"
        state: directory
        owner: root
        group: root
        recurse: yes
        mode: 0777

    - name: running owasp zap full scan container against "{{ website_url }}"
      docker_container:
        name: "{{ scan_name }}"
        image: "{{ owasp_zap_image_name }}"
        interactive: yes
        auto_remove: yes
        state: started
        volumes: "{{ reports_location }}:/zap/wrk:rw"
        command: "zap-full-scan.py -t {{ website_url }} -r {{ scan_name }}_report.html"

    - name: getting raw output of the scan
      raw: "docker logs -f {{ scan_name }}"
      register: scan_output

    - debug:
        msg: "{{ scan_output }}"
```

OWASP ZAP 全面扫描检查了许多漏洞，其中包括 OWASP TOP 10（有关更多信息，请访问[`www.owasp.org/index.php/Category:OWASP_Top_Ten_Project`](https://www.owasp.org/index.php/Category:OWASP_Top_Ten_Project)）和许多其他漏洞。这可能对应用程序造成干扰，并向应用程序发送主动请求。它可能会基于应用程序中存在的漏洞对功能造成损害：

![](img/f1f77783-9cb8-4c72-9f58-15f022c2e90c.png)

DVWS 应用程序的 OWASP ZAP 全面扫描报告

上面的截图是 OWASP ZAP 针对 DVWS 应用程序的全面扫描报告。我们可以清楚地看到基线扫描和全面扫描之间的差异，这是基于漏洞数量、不同类型的漏洞和风险评级。

# 测试 Web API

与 ZAP 基线扫描类似，ZAP 背后的可爱人士们提供了一个脚本作为他们的实时和每周 Docker 镜像的一部分。我们可以使用它来对由 OpenAPI 规范或**简单对象访问协议**（**SOAP**）定义的 API 端点运行扫描。

脚本可以理解 API 规范并导入所有定义。基于此，它对找到的所有 URL 运行主动扫描：

```
- name: Running OWASP ZAP API Scan
  hosts: zap
  remote_user: "{{ remote_user_name }}"
  gather_facts: no
  become: yes
  vars:
    remote_user_name: ubuntu
    owasp_zap_image_name: owasp/zap2docker-weekly
    website_url: {{ website_url }}
    reports_location: /zapdata/
    scan_name: owasp-zap-api-scan-dvws
    api_type: openapi
>
  tasks:
    - name: adding write permissions to reports directory
      file:
        path: "{{ reports_location }}"
        state: directory
        owner: root
        group: root
        recurse: yes
        mode: 0777

    - name: running owasp zap api scan container against "{{ website_url }}"
      docker_container:
        name: "{{ scan_name }}"
        image: "{{ owasp_zap_image_name }}"
        interactive: yes
        auto_remove: yes
        state: started
        volumes: "{{ reports_location }}:/zap/wrk:rw"
        command: "zap-api-scan.py -t {{ website_url }} -f {{ api_type }} -r {{ scan_name }}_report.html"

    - name: getting raw output of the scan
      raw: "docker logs -f {{ scan_name }}"
      register: scan_output

    - debug:
        msg: "{{ scan_output }}"
```

# 使用 ZAP 和 Jenkins 进行持续扫描工作流

Jenkins 是一个开源自动化服务器。它在 CI/CD 流水线中被广泛使用。这些流水线通常指一系列基于触发器发生的自动化步骤，例如提交代码到版本控制软件或创建新版本。

我们已经看到了 ZAP 基线扫描作为 Mozilla 发布周期的一部分的示例。我们可以将 ZAP 与 Jenkins 集成。虽然我们可以有许多方式来实现这一点，但一组有用的步骤将是以下内容：

1.  基于触发器，一个新的 ZAP 实例已经准备好进行扫描

1.  ZAP 实例针对自动部署的应用程序运行

1.  扫描结果以某种格式捕获和存储

1.  如果我们选择，结果也可以在诸如 Atlassian Jira 的缺陷跟踪系统中创建票证

为此，我们将首先设置我们的流水线基础设施：

1.  使用 playbook 设置 Jenkins

1.  添加官方 OWASP ZAP Jenkins 插件

1.  使用另一个 playbook 触发工作流程

官方 OWASP ZAP Jenkins 插件可以在 [`wiki.jenkins.io/display/JENKINS/zap+plugin`](https://wiki.jenkins.io/display/JENKINS/zap+plugin) 找到。

# 设置 Jenkins

在服务器上设置 Jenkins 用作 OWASP ZAP 的 CI/CD 平台。 这将返回 Jenkins 管理员密码，一旦完成，我们就可以安装 Ansible 插件：

```
- name: installing jenkins in ubuntu 16.04
  hosts: jenkins
  remote_user: {{ remote_user_name }}
  gather_facts: False
  become: yes
  vars:
    remote_user_name: ubuntu

  tasks:
    - name: adding jenkins gpg key
      apt_key:
        url: 'https://pkg.jenkins.io/debian/jenkins-ci.org.key'
        state: present

    - name: jeknins repository to system
      apt_repository:
        repo: 'deb http://pkg.jenkins.io/debian-stable binary/'
        state: present
    - name: installing jenkins
      apt:
        name: jenkins
        state: present
        update_cache: yes

    - name: adding jenkins to startup
      service:
        name: jenkins
        state: started
        enabled: yes

    - name: printing jenkins default administration password
      command: cat "/var/lib/jenkins/secrets/initialAdminPassword"
      register: jenkins_default_admin_password

    - debug: 
        msg: "{{ jenkins_default_admin_password.stdout }}"

```

然后，我们可以将 playbook 添加到项目中。 当 Jenkins 构建中发生新的触发时，playbook 将开始扫描网站以执行基线扫描：

![](img/5f4b742a-1eb2-48e3-bc56-962db8f2ad5b.png)

一旦 playbooks 触发，它将针对 URL 执行 playbooks 并返回 ZAP 基线扫描输出：

![](img/0530335e-2a45-4cf4-9d97-14eb53e6e209.png)

# 设置 OWASP ZAP Jenkins 插件

OWASP ZAP 与 Jenkins 配合工作是一个相当常见的设置。我们已经知道如何设置 Jenkins。我们可以使用我们的 playbook 安装官方 ZAP Jenkins 插件。

一旦 playbook 准备就绪，需要进行一些手动配置。 我们在我们的 playbook 安装了 Jenkins 并重新启动服务器后开始，以便插件可用于我们的构建作业。

让我们创建一个新的构建作业，并将其命名为 `ZAP-Jenkins`，如下面的屏幕截图所示：

![](img/532650b3-2529-4ef8-baf2-c5271b1626dc.png)

对我们来说，这将是一个自由风格的项目。 现在我们将向其中添加 ZAP 的魔力：

![](img/c01a9481-e741-4a27-a5f7-81fe43a0427f.png)

我们正在按照 Jenkins 页面上官方插件的说明进行操作： [`wiki.jenkins.io/display/JENKINS/zap+plugin`](https://wiki.jenkins.io/display/JENKINS/zap+plugin)。

# 需要一些组装

指定接口的 IP 地址和 ZAP 应监听的端口号。 通常，此端口为 `8080`，但由于 Jenkins 正在监听该端口，我们选择 `8090`：

![](img/730cfcc9-281b-48be-afb8-c5a4b7b6b73a.png)

对于 JDK，我们选择唯一可用的选项，即从作业继承：

![](img/9f243189-aa25-4418-8fae-a1833c4c4153.png)

对于安装方法，我们选择已经安装在 `/usr/share/owasp-zap` 上的 ZAP。 我们将此值添加到 `/etc/environment` 中的 `ZAPROXY_HOME` 环境变量中。

这样做，我们确保环境变量值在系统重新启动时也能存活：

![](img/4eccf484-5cca-4f5f-b724-4804d0f95444.png)

我们为超时设置了相当小的值，以确保万一出现问题，我们不必等待太久才能看到构建失败或 ZAP 无响应。

我们还指定了一个命令行选项，告诉 Jenkins ZAP 的安装目录是什么。

您可能需要单击“高级”按钮才能查看这些选项。

![](img/590e44b0-85bf-4458-b085-843093366d5a.png)

我们指定了 ZAP 主目录的路径：

![](img/7ca7ac48-5b26-44b3-829a-db3204f326b2.png)

然后我们配置从哪里加载 ZAP 会话：

![](img/af32082a-c625-49a8-b04f-6df0f587185d.png)

这里显示了上下文名称、范围和排除项：

![](img/0e38dc8c-b8a3-4846-95cd-5d4052862e9b.png)

这是要测试的 URL 的起始点。我们计划执行的测试类型是爬虫扫描，默认为主动扫描：

![](img/4480cb66-d930-4888-827e-5930f8108cb9.png)

最后，我们指定将生成的报告的文件名。我们添加`BUILD_ID`变量以确保我们无需担心覆盖报告。

# 触发构建（ZAP 扫描）

作业配置完成后，我们就可以触发构建了。当然，您也可以手动点击立即构建然后开始。

但我们将配置构建作业以远程触发，并同时传递必要的目标信息。

在常规下勾选此项目是参数化的：

![](img/97119a78-d832-4e6e-8063-b7a9e494be69.png)

在其中，我们添加一个带有默认值的`TARGET`参数。

在构建触发器下，我们指定一个身份验证令牌，以便在远程触发构建时作为参数传递：

![](img/0fe9b5c7-ca95-4b00-a6e3-c38d88445237.png)

请尽量确保此令牌足够长且随机，并且不是我们用作示例的简单单词。

在 Linux/macOS 中生成足够随机的字符串的一个好方法是使用 OpenSSL 命令。对于`hex`输出（`20`是输出的长度），使用`**openssl rand -hex 20**`。对于`base64`输出（`24`是输出的长度），使用`**openssl rand -base64 24**`。

在此时，我们所要做的就是注意已登录用户的 API 令牌（从`http://JENKINS-URL/user/admin/configure`）：

![](img/f484ebee-c98d-4148-baba-a06dbcb14ad7.png)

单击“显示 API 令牌”将显示令牌：

![](img/d828b98f-18ee-4aac-8e5a-7c704816525d.png)

我们现在可以使用命令行工具，比如`curl`，来查看是否正常工作。

链接的格式是`curl "http://username:API-TOKEN@JENKINS-URL/job/ZAP-Jenkins/buildWithParameters?TARGET=http://demo.testfire.net&token=ansible2security"`。

这将触发构建，应用程序将被扫描以查找安全问题。

# 使用自动化进行的操作指南

要执行前述触发器，我们可以使用以下 Ansible playbook。这也可以在我们的 Ansible Tower 中使用来安排扫描。

以下操作指南可以使用 Ansible Vault 存储 API 令牌密钥，该功能用于以加密格式存储 playbooks 中的机密数据。我们将在第十一章中学习有关 Ansible Vault 用法的更多信息，*Ansible 安全最佳实践、参考资料和进一步阅读*。

要创建 Ansible Vault 加密变量，请运行以下命令。当它提示输入密码时，给出一个密码以加密此变量，执行 playbook 时需要。

```
echo 'YOURTOKENGOESHERE' | ansible-vault encrypt_string --stdin-name 'jenkins_api_token'
```

执行后，它会返回加密变量，我们可以直接在 playbook 中使用它作为变量：

```
- name: jenkins build job trigger
  hosts: localhost
  connection: local
  vars:
    jenkins_username: username
    jenkins_api_token: !vault |
          $ANSIBLE_VAULT;1.1;AES256
          36636563313932313366313030623232623338333638363465343339636362353534363536366161
          3062666536613764396439326534663237653438616335640a613564643366623462666361633763
          31326161303666653366343931366265333238383937656435663061363665643431336638353436
          3532646434376533390a646332646639653161343165363832616233333232323130623034313032
          66643537336634633263346363313437666262323064386539616333646132353336
    jenkins_host: 192.168.11.111
    jenkins_target_url: 'http://demo.testfire.net'
    jenkins_token: ansible2security
>
  tasks:
    - name: trigger jenkins build
      uri:
        url: "http://{{ jenkins_username }}:{{ jenkins_api_token }}@{{ jenkins_host }}/job/ZAP-Jenkins/buildWithParameters?TARGET={{ jenkins_target_url }}&token={{ jenkins_token }}"
        method: GET
      register: results
    - debug:
        msg: "{{ results.stdout }}"
```

在执行 playbook 时执行 `ansible-vault` 解密，playbook 执行命令如下：

```
$ ansible-playbook --ask-vault-pass main.yml
```

# ZAP Docker 和 Jenkins

Mozilla 的一群人撰写了一篇关于如何配置 ZAP Docker 与 Jenkins 的出色博客系列。与其重复他们的内容，我们觉得把你指向该系列的第一篇文章更有意义。

欲了解更多信息，请查看有趣的博客 *Docker 化，OWASP-ZAP 安全扫描，在 Jenkins 中，第一部分*，链接在：[`blog.mozilla.org/webqa/2016/05/11/docker-owasp-zap-part-one/`](https://blog.mozilla.org/webqa/2016/05/11/docker-owasp-zap-part-one/)。

# 摘要

OWASP ZAP 是任何安全团队工具库中的绝佳选择。它在我们如何使用以及如何将其融入我们的设置中提供了完全的灵活性。通过将 ZAP 与 Jenkins 结合使用，我们可以快速建立一个体面的、适合生产的持续扫描工作流，并围绕它对我们的流程进行调整。Ansible 允许我们使用 playbooks 安装和配置所有这些优秀的工具。这很棒，因为这主要是一次性的努力，然后我们就可以开始看到 ZAP 的结果和报告。

现在我们正在自动化安全工具的过程中，接下来我们将看到最流行的漏洞评估工具 Nessus，以及我们如何为软件和网络的漏洞评估构建类似的工作流程。
