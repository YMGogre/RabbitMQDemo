# 0、引言

想要实现两个应用程序之间的通信，我们可以借助“消息队列”技术。本文将介绍使用 C# 语言在 .NET 下实现 RabbitMQ 消息队列；当然无论是哪种编程语言或框架，要使用消息队列，都需要完成以下两个基本步骤：

1. 下载并安装相应的消息队列服务器软件，并根据需要进行配置；
2. 在您的应用程序代码中引入相应的消息队列客户端库，并使用客户端库提供的API来建立与消息队列服务器的连接并进行通信。

这些步骤对于大多数消息队列中间件都是适用的。不同的消息队列中间件可能有不同的安装和配置方法，以及不同的客户端库和API，您可以根据自身需要阅读相应的官方文档即可。

> 本文主要参考：
> 
> 1. [RabbitMQ Tutorials](https://www.rabbitmq.com/getstarted.html)
> 2. [RabbitMQ使用教程(超详细)](https://blog.csdn.net/lzx1991610/article/details/102970854)
> 3. [C# 消息队列之 RabbitMQ 基础入门](https://www.cnblogs.com/abeam/p/11872956.html)
> 4. [消息队列常见的几种使用场景介绍！- 知乎](https://zhuanlan.zhihu.com/p/272700109#)
> 5. [分布式之消息队列的特点、选型、及应用场景详解](https://zhuanlan.zhihu.com/p/97179688)
> 6. [消息队列漫谈：什么是消息模型？- 知乎](https://zhuanlan.zhihu.com/p/99791229)

# 1、基础概念

[消息队列](https://zhuanlan.zhihu.com/p/157112243)
: <font color="#dd0000">**消息队列**</font>（<font color="#dd0000">**M**</font>essage <font color="#dd0000">**Q**</font>ueue）是一种通信模式，用于在应用程序之间传递消息。它提供了一种异步的、可靠的通信方式，允许发送者（发送消息的应用程序）和接收者（接收消息的应用程序）能够独立地进行工作，而不需要彼此直接交互。<br><br>在消息队列中，消息是以队列的形式存储和传递的。发送者将消息放入队列的末尾，而接收者从队列的开头获取消息。这种队列的特性确保了消息的有序处理，并且允许多个发送者和接收者之间进行解耦，从而提高了系统的可伸缩性和可靠性。<br>![](./image/%E5%9F%BA%E7%A1%80%E6%A6%82%E5%BF%B5/1-MQschematicdiagram.jpg)<br>消息队列的使用可以带来许多好处。首先，它允许异步处理，即发送者可以在发送消息后继续执行其他任务，而不需要等待接收者的响应。这种方式可以提高应用程序的性能和吞吐量，特别是在处理大量消息或处理延迟较高的操作时。<br><br>其次，消息队列可以解耦发送者和接收者之间的依赖关系。发送者只需将消息发送到队列中，而不需要知道具体的接收者是谁或接收者何时处理消息。这种松耦合的设计允许系统中的不同模块独立开发、部署和扩展，提高了系统的灵活性和可维护性。<br><br>此外，消息队列还提供了消息持久化的能力，确保即使在发送者和接收者之间发生故障或中断时，消息不会丢失。消息可以持久化到磁盘上，并在系统恢复后重新发送。这种可靠性保证了消息的传递不会因为故障而中断，使系统更加健壮和可靠。

[**常见的消息队列软件**](https://blog.csdn.net/u013521220/article/details/104352365)
: * [RabbitMQ](https://www.rabbitmq.com/)
  * [Apache Kafka](https://kafka.apache.org/)
  * [Apache ActiveMQ](https://activemq.apache.org/)
  * [Amazon SQS](https://aws.amazon.com/sqs/)
  * [Microsoft Azure Service Bus](https://azure.microsoft.com/en-us/services/service-bus/)

中间件
: <font color="#dd0000">**中间件**</font>是指位于操作系统和应用程序之间的软件层。它充当了不同软件组件之间的桥梁，提供了通信和协调的功能，以便它们能够相互交互和协作。<br><br>中间件的主要目标是简化分布式系统的开发和管理。它提供了一组通用的功能和服务，使得不同的应用程序和系统可以进行互操作，并能够以可靠、安全和高效的方式进行通信。<br><br>中间件可以实现各种功能，包括但不限于以下几个方面：<br><br><ol><li>消息传递：中间件提供消息传递机制，用于在分布式系统中传递和交换数据。消息队列就是一种常见的消息传递中间件。</li><li>远程过程调用（RPC）：中间件可以实现远程过程调用，使得应用程序能够在不同的计算机或进程之间调用和执行函数或方法。</li><li>数据库连接和访问：中间件可以提供数据库连接和访问的功能，使得应用程序能够方便地操作和管理数据库。</li><li>分布式事务处理：中间件可以支持分布式系统中的事务处理，确保多个操作在不同的计算机或进程之间具有原子性、一致性、隔离性和持久性。</li><li>缓存管理：中间件可以提供缓存管理的功能，加速数据访问和提高系统性能。</li><li>安全认证和授权：中间件可以实现安全认证和授权机制，保护系统免受未经授权的访问和攻击。</li></ol>

[Erlang](https://www.erlang.org/)
: <font color="#dd0000">**Erlang**</font> 是一种通用的编程语言，最初由爱立信（Ericsson）的开发团队在1980年代末创建。它是一种函数式编程语言，专门设计用于构建可扩展、并发和分布式的软实时系统。<font color="#008000">RabbitMQ 服务是使用 Erlang 语言编写的</font>。

# 2、本文使用的相关软件或产品

1. Windows 10 专业版 22H2
2. Visual Studio Community 2022 - 17.6.2
3. Microsoft .NET Framework 版本 4.8.04084
4. 「.NET 桌面开发」工作负荷
5. Erlang 25.3
6. RabbitMQ Server 3.12.0

# 3、🔔 须知

<font color="#C71585">大多数消息队列中间件都需要在计算机上安装服务器软件。安装服务器软件后，您需要启动消息队列服务器并根据需要进行配置。然后，您就可以在您的应用程序中使用相应的客户端库来连接到消息队列服务器并使用消息队列了。</font>

# 4、安装前准备工作 —— 阅读安装向导页面

> 💬 如果您不想阅读安装向导，那么直接跳过第四章就好啦~ [点击](#AnchorPoint-InstallRabbitMQServer)前往第五章

1. 首先前往 RabbitMQ 的[入门指南页面](https://www.rabbitmq.com/#getstarted)，点击“**Download + Installation**”：<br>![](./image/%E5%AE%89%E8%A3%85%20RabbitMQ%20%E6%9C%8D%E5%8A%A1%E5%99%A8/1-getstarted.PNG)
2. 进入到[下载和安装 RabbitMQ 页面](https://www.rabbitmq.com/download.html)，可以看到最新的 RabbitMQ [发行版本](https://github.com/rabbitmq/rabbitmq-server/releases)为 **3.12.0**。由于笔者使用的 Windows 操作系统，故点击“**Windows Installer**”进入 Windows 平台的安装程序的安装向导页面：<br>![](./image/%E5%AE%89%E8%A3%85%20RabbitMQ%20%E6%9C%8D%E5%8A%A1%E5%99%A8/2-downloadandinstall.PNG)

    > 您也可以通过 [Chocolatey 包安装](https://community.chocolatey.org/packages/rabbitmq)（本文不演示）或者通过[构建二进制包手动安装](https://www.rabbitmq.com/install-windows-manual.html)（通常不推荐）。

3. 进入到 [Windows 平台安装程序的向导页面](https://www.rabbitmq.com/install-windows.html)，推荐了两个安装选项：①使用 Chocolatey 安装；②以管理员身份使用官方安装程序安装。这里选择第二项 —— “[Using the official installer](#AnchorPoint-UsingtheInstaller) as an administrative user”：<br>![](./image/%E5%AE%89%E8%A3%85%20RabbitMQ%20%E6%9C%8D%E5%8A%A1%E5%99%A8/3-installonwindows.png)

<a id="AnchorPoint-UsingtheInstaller">

## 4.1、[使用安装程序](https://www.rabbitmq.com/install-windows.html#installer)

官方的 RabbitMQ 安装程序是为[每个 RabbitMQ 版本](https://www.rabbitmq.com/changelog.html)生成的。

与[通过 Chocolatey 安装](https://www.rabbitmq.com/install-windows.html#chocolatey)相比，此选项为 Windows 用户提供了最大的灵活性，但也要求他们了解安装程序中的某些假设和要求：

- 一次只能安装一个 Erlang 版本；
- 必须**以管理员身份**安装 Erlang；
- **强烈建议** RabbitMQ 也是通过管理员身份安装；
- 安装路径只能包含 ASCII 字符，**强烈建议**安装路径的任何目录名称中都不要包含空格；
- 可能需要手动复制 CLI 工具使用的[共享密钥](https://www.rabbitmq.com/cli.html#erlang-cookie)文件；
- CLI 工具要求在 UTF-8 模式下运行 Windows 控制台。

如果不满足这些条件，Windows 服务和 CLI 工具可能需要重新安装或其他手动步骤，以使它们按预期运行。

[Windows 特定问题](https://www.rabbitmq.com/windows-quirks.html)指南中对此进行了更详细的介绍。

### 4.1.1、依赖

RabbitMQ 需要安装 64 位其[支持的 Erlang for Windows](https://www.rabbitmq.com/which-erlang.html) 版本。

[Erlang 25.3](https://www.erlang.org/patches/otp-25.3.2) 是最新的支持版本。您可以在 [Erlang/OTP 版本树](https://erlang.org/download/otp_versions_tree.html)页面获取其他版本（比如：早期版本）的 Erlang for Windows 二进制构建包。

Erlang **必须以管理员身份安装**，否则 RabbitMQ Windows 服务将无法发现它。安装完成支持的 Erlang 版本后，下载 RabbitMQ 安装程序 `rabbitmq-server-{version}.exe` 并运行它。安装程序会将 RabbitMQ 作为 Windows 服务进行安装并使用默认配置启动它。

<a id="AnchorPoint-DirectDownload">

### 4.1.2、[直接下载](https://www.rabbitmq.com/install-windows.html#downloads)

|描述|下载|签名|
|---|---|---|
|Windows 系统的安装程序（来自 [Github](https://github.com/rabbitmq/rabbitmq-server/releases)）|[rabbitmq-server-3.12.0.exe](https://github.com/rabbitmq/rabbitmq-server/releases/download/v3.12.0/rabbitmq-server-3.12.0.exe)|[签名](https://github.com/rabbitmq/rabbitmq-server/releases/download/v3.12.0/rabbitmq-server-3.12.0.exe.asc)|

## 4.2、[运行 RabbitMQ Windows 服务](https://www.rabbitmq.com/install-windows.html#service)

一旦 Erlang 和 RabbitMQ 都被安装了，RabbitMQ 节点就能作为 Windows 服务启动。RabbitMQ 服务会自动启动。RabbitMQ Windows 服务可以从“开始”菜单进行管理。

## 4.3、[CLI 工具](https://www.rabbitmq.com/install-windows.html#cli)

RabbitMQ 节点通常使用 [PowerShell](https://learn.microsoft.com/zh-cn/windows-server/administration/windows-commands/powershell) 中的 [CLI 工具](https://www.rabbitmq.com/cli.html)进行管理、检查和操作。

在 Windows 上，与其他平台相比，CLI 工具具有 `.bat` 后缀。例如：Windows 上的 `rabbitmqctl` 被调用为 `rabbitmqctl.bat`。

为了使这些工具正常工作，它们必须能够使用名为 **Erlang cookie** 的共享密钥文件[向 RabbitMQ 节点进行身份验证](https://www.rabbitmq.com/cli.html#erlang-cookie)。

[CLI 工具向导](https://www.rabbitmq.com/cli.html)主页面涵盖了与命令行工具使用相关的绝大部分话题。

要了解各种 RabbitMQ CLI 工具提供的命令，请使用 `help` 命令：

```shell
# 罗列 rabbitmqctl.bat 提供的命令
rabbitmqctl.bat help

# 罗列 rabbitmq-diagnostics.bat 提供的命令
rabbitmq-diagnostics.bat help

# 罗列 rabbitmq-plugins.bat 提供的命令
rabbitmq-plugins.bat help
```

想要学习特定命令，将命令名称作为 `help` 的参数传递即可：

```shell
rabbitmqctl.bat help add_user
```

## 4.4、[Cookie 文件位置](https://www.rabbitmq.com/install-windows.html#cli-cookie-file-location)

在 Windows 平台下，[cookie 文件位置](https://www.rabbitmq.com/cli.html#cookie-file-locations)取决于是否设置了 `HOMEDRIVE` 和 `HOMEPATH` 环境变量。

如果 RabbitMQ 使用非管理员账户安装，则节点和 CLI 工具使用的[共享密钥](https://www.rabbitmq.com/cli.html#erlang-cookie)文件将不会被放到正确的位置，这会导致使用 `rabbitmqctl.bat` 以及其他 CLI 工具时出现[身份验证失败错误](https://www.rabbitmq.com/cli.html#cli-authentication-failures)。

可以使用下列选项之一化解：

* 使用管理员账户重新安装 RabbitMQ
* 从 `%SystemRoot%` 或 `%SystemRoot%\system32\config\systemprofile` 手动复制 `.erlang.cookie` 文件到 `%HOMEDRIVE%%HOMEPATH%`

---

...（更多内容请自行前往 [Windows 平台的安装程序向导页面](https://www.rabbitmq.com/install-windows.html)了解）

<a id="AnchorPoint-InstallRabbitMQServer">

# 5、正式开始安装 RabbitMQ 服务器

## 5.1、下载安装 Erlang

1. 在安装 RabbitMQ 之前，首先安装 RabbitMQ 依赖 —— Erlang。前往 [Erlang 25.3](https://www.erlang.org/patches/otp-25.3.2)，点击“<font color="#a2003e">**Download Windows installer**</font>”：<br>![](./image/%E5%AE%89%E8%A3%85%20RabbitMQ%20%E6%9C%8D%E5%8A%A1%E5%99%A8/4-downloaderlang.png)
2. 下载完成后，<font color="#ff4500">**以管理员身份运行**</font>：<br>![](./image/%E5%AE%89%E8%A3%85%20RabbitMQ%20%E6%9C%8D%E5%8A%A1%E5%99%A8/5-downloaderlangcompleted.PNG)
3. 运行后进入组件选择页面，一般按默认选择即可，点击下一步：<br>![](./image/%E5%AE%89%E8%A3%85%20RabbitMQ%20%E6%9C%8D%E5%8A%A1%E5%99%A8/6-setup.PNG)
4. 选择安装位置页面，可以按默认也可以根据自身喜好修改；确认后点击下一步：<br>![](./image/%E5%AE%89%E8%A3%85%20RabbitMQ%20%E6%9C%8D%E5%8A%A1%E5%99%A8/7-installlocation.PNG)
5. 选择开始菜单文件夹页面，该页面要求您通过点击选中或者手动输入的方式以选择一个开始菜单文件夹用于创建程序的快捷方式，按默认即可。点击“安装”：<br>![](./image/%E5%AE%89%E8%A3%85%20RabbitMQ%20%E6%9C%8D%E5%8A%A1%E5%99%A8/8-choosestartmenufolder.PNG)

    > 如果按照默认，安装完成后可以在[开始菜单](https://support.microsoft.com/zh-cn/windows/%E6%89%93%E5%BC%80-%E5%BC%80%E5%A7%8B-%E8%8F%9C%E5%8D%95-4ed57ad7-ed1f-3cc9-c9e4-f329822f5aeb)找到一个名为“**Erlang OTP 25 (x64)**”的文件夹，该文件夹存放了 Erlang 的快捷方式：<br>![](./image/%E5%AE%89%E8%A3%85%20RabbitMQ%20%E6%9C%8D%E5%8A%A1%E5%99%A8/10-startmenu.png)

6. 安装完成，点击“关闭”：<br>![](./image/%E5%AE%89%E8%A3%85%20RabbitMQ%20%E6%9C%8D%E5%8A%A1%E5%99%A8/9-installcompleted.PNG)
7. 完成后，就可以删除 Erlang 的安装程序了：<br>![](./image/%E5%AE%89%E8%A3%85%20RabbitMQ%20%E6%9C%8D%E5%8A%A1%E5%99%A8/11-%E5%88%A0%E9%99%A4%E5%AE%89%E8%A3%85%E7%A8%8B%E5%BA%8F.PNG)

## 5.2、下载安装 RabbitMQ Server

1. 安装完 Erlang 后，就可以下载安装 RabbitMQ Server 了，笔者安装的 Erlang 版本为 **25.3.2**，由[该页面](https://www.rabbitmq.com/which-erlang.html)可知可以安装 RabbitMQ 的最新版本 —— **3.12.0**：<br>![](./image/%E4%B8%8B%E8%BD%BD%E5%AE%89%E8%A3%85RabbitMQServer/1-versionsupport.png)
2. 前往[直接下载](#AnchorPoint-DirectDownload)，下载 `rabbitmq-server-3.12.0.exe`：![](./image/%E4%B8%8B%E8%BD%BD%E5%AE%89%E8%A3%85RabbitMQServer/2-download.png)
3. 下载完成后，<font color="#ff4500">**以管理员身份运行**</font>：<br>![](./image/%E4%B8%8B%E8%BD%BD%E5%AE%89%E8%A3%85RabbitMQServer/3-downloadcompleted.PNG)
4. 运行后进入组件选择页面，一般按默认选择即可，点击下一步：<br>![](./image/%E4%B8%8B%E8%BD%BD%E5%AE%89%E8%A3%85RabbitMQServer/4-componentchoose.PNG)
5. 选择安装位置页面，可以按默认也可以根据自身喜好修改；确认后点击“安装”：<br>![](./image/%E4%B8%8B%E8%BD%BD%E5%AE%89%E8%A3%85RabbitMQServer/5-chooseinstalllocation.PNG)

    > 💬 在[这个](https://www.rabbitmq.com/install-windows.html#installer)官方页面提到过“强烈建议安装路径的任何目录名称中都不要包含空格”，但是安装程序默认的安装路径都是有空格的，所以我也不太懂他们了，，，

6. 安装完成后，点击下一步：<br>![](./image/%E4%B8%8B%E8%BD%BD%E5%AE%89%E8%A3%85RabbitMQServer/6-installcompleted.PNG)

    <a id="AnchorPoint-CompletedPage">

7. 完成页面，点击“结束”：<br>![](./image/%E4%B8%8B%E8%BD%BD%E5%AE%89%E8%A3%85RabbitMQServer/7-completed.PNG)

    > 结束后可以在[开始菜单](https://support.microsoft.com/zh-cn/windows/%E6%89%93%E5%BC%80-%E5%BC%80%E5%A7%8B-%E8%8F%9C%E5%8D%95-4ed57ad7-ed1f-3cc9-c9e4-f329822f5aeb)中找到一个名为“**RabbitMQ Server**”的文件夹，可以从这里管理 RabbitMQ 服务器以及使用 CLI 工具：<br>![](./image/%E4%B8%8B%E8%BD%BD%E5%AE%89%E8%A3%85RabbitMQServer/8-startmenu.png)

8. 完成后，就可以删除 RabbitMQ Server 的安装程序了：<br>![](./image/%E4%B8%8B%E8%BD%BD%E5%AE%89%E8%A3%85RabbitMQServer/9-delete.PNG)
9.  由于在[完成页面](#AnchorPoint-CompletedPage)默认勾选了“Start RabbitMQ service”，所以此时我们已经可以在[计算机管理页](https://jingyan.baidu.com/article/e2284b2bab6e35e2e7118d7d.html)找到该 Windows 服务了：<br>![](./image/%E4%B8%8B%E8%BD%BD%E5%AE%89%E8%A3%85RabbitMQServer/10-WindowsService.PNG)

## 5.3、启用服务管理

我们可以通过一些方法来管理 RabbitMQ 服务器，在官网的[端口访问](https://www.rabbitmq.com/install-windows.html#ports)页面介绍了 RabbitMQ 节点绑定到的端口：<br>![](./image/%E5%90%AF%E5%8A%A8%E7%AE%A1%E7%90%86%E7%AB%AF%E5%8F%A3/0-portaccess.PNG)<br>其中我们主要关注 **15672** 号端口：[HTTP API](https://www.rabbitmq.com/management.html) 客户端，[管理 UI](https://www.rabbitmq.com/management.html) 和 [rabbitmqadmin](https://www.rabbitmq.com/management-cli.html)（仅当启用了[管理插件](https://www.rabbitmq.com/management.html)时）。

💬 <font color="#C71585">我们可以通过 HTTP API 使用该管理插件用来管理和[监控](https://www.rabbitmq.com/monitoring.html) RabbitMQ 节点和集群</font>，以及使用基于浏览器的 UI 和命令行工具，[rabbitmqadmin](https://www.rabbitmq.com/management-cli.html)。

1. 在[开始菜单](https://support.microsoft.com/zh-cn/windows/%E6%89%93%E5%BC%80-%E5%BC%80%E5%A7%8B-%E8%8F%9C%E5%8D%95-4ed57ad7-ed1f-3cc9-c9e4-f329822f5aeb)中打开 **RabbitMQ 命令提示符 (sbin dir)**：<br>![](./image/%E5%90%AF%E5%8A%A8%E7%AE%A1%E7%90%86%E7%AB%AF%E5%8F%A3/1-%E6%89%93%E5%BC%80%E5%91%BD%E4%BB%A4%E8%A1%8C%E5%B7%A5%E5%85%B7.png)
2. 使用如下命令以启用 RabbitMQ 管理插件：

    ```shell
    rabbitmq-plugins enable rabbitmq_management
    ```
    ![](./image/%E5%90%AF%E5%8A%A8%E7%AE%A1%E7%90%86%E7%AB%AF%E5%8F%A3/2-%E5%91%BD%E4%BB%A4%E8%A1%8C%E5%90%AF%E5%8A%A8.PNG)

3. 这里提示改动会在重启代理后生效，前往[计算机管理页](https://jingyan.baidu.com/article/e2284b2bab6e35e2e7118d7d.html)找到 RabbitMQ Windows 服务，通过重启该服务以重启代理：<br>![](./image/%E5%90%AF%E5%8A%A8%E7%AE%A1%E7%90%86%E7%AB%AF%E5%8F%A3/3-%E9%87%8D%E5%90%AF%E6%9C%8D%E5%8A%A1.PNG)
4. 打开浏览器，输入如下地址以访问：`http://127.0.0.1:15672/`<br>![](./image/%E5%90%AF%E5%8A%A8%E7%AE%A1%E7%90%86%E7%AB%AF%E5%8F%A3/4-%E7%99%BB%E5%BD%95.PNG)

    > [默认用户访问权限](https://www.rabbitmq.com/install-windows.html#default-user-access)：代理会使用密码 `guest` 创建一个用户 `guest`。未配置的客户端通常会使用这些凭据。**默认情况下，这些凭据只能是在作为本地主机连接到代理时使用**，所以你需要在任何其他设备连接到代理之前采取一些行动。
    >
    > 有关如何创建更多用户以及删除 `guest` 用户的信息，请参阅有关[访问控制](https://www.rabbitmq.com/access-control.html)的文档。
    >
    > 综上，这一步我们使用默认账号 `guest` 和密码 `guest` 登录即可。

5. 登录成功后如下图所示：<br>![](./image/%E5%90%AF%E5%8A%A8%E7%AE%A1%E7%90%86%E7%AB%AF%E5%8F%A3/5-%E8%BF%9B%E5%85%A5.PNG)

# 6、在两个应用程序之间使用 RabbitMQ 消息队列中间件

> ⚠️ 在开始本章的内容之前，您需要确保您已经完成了 RabbitMQ 服务器的安装并正常启动！您可以参考[第五章](#AnchorPoint-InstallRabbitMQServer)了解相关内容。

1. 打开 Visual Studio，创建两个控制台应用程序项目 `RabbitMQProducer` 和 `RabbitMQConsumer`：<br>![](./image/%E5%AE%A2%E6%88%B7%E7%AB%AF%E6%B5%8B%E8%AF%95/1-newproject.PNG)

    > `RabbitMQProducer`：生产者，用于向消息队列中<font color="#008000">**发送消息**</font>；<br>
    > `RabbitMQConsumer`：消费者，<font color="#008000">**接收**</font>来自消息队列中的<font color="#008000">**消息**</font>；

2. 完成后，右击项目名称，点击“管理 NuGet 程序包”：<br>![](./image/%E5%AE%A2%E6%88%B7%E7%AB%AF%E6%B5%8B%E8%AF%95/2-Nuget.PNG)
3. 分别为 `RabbitMQProducer` 和 `RabbitMQConsumer` 两个项目引入 RabbitMQ.Client 库：<br>![](./image/%E5%AE%A2%E6%88%B7%E7%AB%AF%E6%B5%8B%E8%AF%95/3-%E5%AE%89%E8%A3%85Client%E5%BA%93.png)

---

以下是 `RabbitMQProducer` 的代码：
```C#
using RabbitMQ.Client;
using System.Text;

//连接 RabbitMQ 工厂实例
var factory = new ConnectionFactory() { HostName = "localhost",         //主机名，默认为 "localhost"
                                        Port = 5672,                    //要连接的端口，默认为 -1(5672)
                                        UserName = "guest",             //用户名，默认为 "guest"
                                        Password = "guest"};            //密码，默认为 "guest"
//创建连接对象
using (var connection = factory.CreateConnection())
//创建一个新的通道、会话和模型
using(var channel = connection.CreateModel())
{
    //声明一个消息队列
    channel.QueueDeclare(queue: "myTestQueue",      //消息队列名称
                         durable: false,            //队列是否能在代理重启后仍然存在
                         exclusive: false,          //是否应该将此队列限制在其声明的连接中?这样的队列将在其声明连接关闭时被删除。（在一个客户端同时发送和读取消息的应用场景中适用）
                         autoDelete: false,         //当最后一个消费者(如果有的话)退订时，是否应该自动删除这个队列?
                         arguments: null);          //可选的;额外的队列参数，例如:“x-queue-type”
    //定义消息内容
    var message = "Hello RabbitMQ!";
    //转换为字节数组（编码格式UTF-8）
    var body = Encoding.UTF8.GetBytes(message);
    Console.WriteLine("按任意键开始发送消息！");
    Console.ReadKey();
    //循环发送一百次消息
    for(int i = 0; i < 100; i++)
    {
        /*
         * 向 RabbitMQ 服务器发送一条消息
         * 调用 BasicPublish 方法后，指定的消息内容就会被发送到 RabbitMQ 服务器，并根据您指定的路由键被路由到相应的消息队列中
         */
        channel.BasicPublish(exchange: "",                  //指定要将消息发布到哪个交换机（exchange），设置为空字符串以使用默认交换机
                             routingKey: "myTestQueue",     //指定消息的路由键（routing key），路由键用于确定消息应该被发送到哪些消息队列
                             basicProperties: null,         //指定消息属性，可以用来设置消息的优先级、过期时间等属性
                             body: body);                   //消息内容，将要发送的数据转换为字节数组，并传递给这个参数
        //控制台打印发送的消息
        Console.WriteLine($"[{DateTime.Now.ToString("HH:mm:ss fff")}] Send: {message}");
        //当前线程暂停一秒钟
        Thread.Sleep(1000);
    }
}
```

---

以下是 `RabbitMQConsumer` 的代码：
```C#
using RabbitMQ.Client;
using RabbitMQ.Client.Events;
using System.Text;

//连接 RabbitMQ 工厂实例
var factory = new ConnectionFactory() { HostName = "localhost",     //主机名，默认为 "localhost"
                                        Port = 5672,                //要连接的端口，默认为 -1(5672)
                                        UserName = "guest",         //用户名，默认为 "guest"
                                        Password = "guest"};        //密码，默认为 "guest"
//创建连接对象
var connection = factory.CreateConnection();
//创建一个新的通道、会话和模型
var channel = connection.CreateModel();
//声明一个消息队列
channel.QueueDeclare(queue: "myTestQueue",      //消息队列名称
                     durable: false,            //队列是否能在代理重启后仍然存在
                     exclusive: false,          //是否应该将此队列限制在其声明的连接中?这样的队列将在其声明连接关闭时被删除。（在一个客户端同时发送和读取消息的应用场景中适用）
                     autoDelete: false,         //当最后一个消费者(如果有的话)退订时，是否应该自动删除这个队列?
                     arguments: null);          //可选的;额外的队列参数，例如:“x-queue-type”

//创建（将 IBasicConsumer 接口实现为事件的）EventingBasicConsumer 类对象并关联指定的 channel
var consumer = new EventingBasicConsumer(channel);
/* 
 * 启动一个基本的内容类消费者（在当前通道中监听 myTestQueue 消息队列，并进行消费）
 * 调用 BasicConsume 方法后，您的应用程序就可以开始从指定的消息队列中接收消息了。
 * 当从消息队列中接收到一条消息时，消费者对象的 Received 事件会被触发，并执行相应的事件处理程序。
 */
channel.BasicConsume(queue: "myTestQueue",      //消息队列名称
                     autoAck: true,             //是否自动发送确认消息（acknowledgement）给 RabbirMQ 服务器
                     consumer: consumer);       //指定用于接收消息的消费者对象

//使用 Lambda 表达式注册事件处理程序并订阅 Received 事件
consumer.Received += (sender, e) =>
{
    var body = e.Body.ToArray();                                                                        //获取消息字节数组
    var message = Encoding.UTF8.GetString(body);                                                        //UTF-8格式编码消息内容字符串
    Console.WriteLine($"[{DateTime.Now.ToString("HH:mm:ss fff")}] Consumer Received: {message},");      //控制台打印消息内容（您也可以在这里添加其他的消息处理代码）
};
//控制台等待读取输入以避免进程退出
Console.ReadLine();
//关闭连接以及它的所有通道
connection.Close();
```

---

运行效果：<br>![](./image/%E5%AE%A2%E6%88%B7%E7%AB%AF%E6%B5%8B%E8%AF%95/4-%E6%B5%8B%E8%AF%95%E6%95%88%E6%9E%9C.gif)

# 7、Q&A

## Q：Visual Studio 如何同时运行两个项目

**A1：** 
1. 在解决方案资源管理器中，右击解决方案节点，然后选择“属性”；<br>![](./image/Q%26A/1-%E5%B1%9E%E6%80%A7.PNG)
2. 展开“通用属性”节点，然后选择“启动项目”，选择“多个启动项目”选项并设置适当的操作。然后点击应用；<br>![](./image/Q%26A/2-%E8%AE%BE%E7%BD%AE%E5%A4%9A%E4%B8%AA%E5%90%AF%E5%8A%A8%E9%A1%B9%E7%9B%AE.PNG)

**A2：** 或者更简单的，打开两个 Visual Studio 就好啦~

## Q：我可以在第一台计算机上运行消息队列服务器，第二台计算机上运行发送消息的程序，第三台计算机上运行接收消息的程序吗？

**A：** 可以，只要这些平台之间能够相互通信（处于同一局域网内），它们就可以使用消息队列进行通信。同时这也意味着，在发送和接收消息的程序代码中，您需要指定 RabbitMQ 服务器所在的计算机的 IP 地址或主机名，以便连接到 RabbitMQ 服务器。<br>例如，如果您的消息队列服务器运行在 IP 地址为 **192.168.1.100** 的计算机上，您需要使用如下代码来创建 `ConnectionFactory` 对象：
```C#
var factory = new ConnectionFactory() { HostName = "192.168.1.100" };
```
> 💬 有关远程部署 RabbitMQ 服务的相关内容您可以点击[这里](https://www.cnblogs.com/abeam/p/11872956.html)了解更多。