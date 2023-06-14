

# Matter学习笔记1

---

在了解Matter之前，可以选择先了解以下前提知识：

* [matter网络基础之—Thread](https://blog.csdn.net/qq_42860989/article/details/118389957)
* [matter网络基础之—Wi-Fi](https://blog.csdn.net/qq_42860989/article/details/118553988)
* [边界路由器，网关和Wi-Fi接入点](https://blog.csdn.net/qq_42860989/article/details/119253287)
* [Thread地址(IPv6 and RLOC16)](https://blog.csdn.net/qq_42860989/article/details/120067170)

> 以上资料来自CSDN博主：[Eagle115](https://blog.csdn.net/qq_42860989)

## 前言

近日，CSA联盟（Connectivity Standards Alliance)正式对外发布了Matter 1.0 标准，并宣布认证计划现已开放。这意味着智能家居品牌可以对其产品进行相关测试和认证，一旦获得认证，公司就可以开始销售带有Matter 标志的设备。

Matter 最初的项目名称是Project Chip(CHIP)，目前由 CSA联盟维护。它是一个**统一标准的物联网通信协议，旨在将繁杂的智能家居设备收归到统一的通信标准**。

Matter 作为一个**应用级的协议**，向下屏蔽了**设备制造商的生态和系统，让各种智能家居设备之间能相互通信**。例如，一个 Matter 认证的智能灯泡可以由另一个厂家生产的同样经过认证的设备来控制。Matter 是基于ip的协议，支持wifi、 Thread、 Internet三种不同的底层协议栈。

Matter 采用不同的通讯协议和技术为未来智能家居行业提供了不同场景下的解决方案：

* **低功耗蓝牙技术**：低功耗蓝牙作为一种专门设计用于低功耗设备之间通信的无线通信技术，它可以在较低的功率下实现较长的通信距离，因此非常适合用于智能家居设备之间的连接。Matter 使用低功耗蓝牙技术进行设备之间的连接和控制。
* **二维码进行配置**：二维码是一种快速扫描的图形码，可以用于快速识别设备身份和配置设备。在 Matter 中，用户可以扫描设备上的二维码，以快速将设备添加到智能家居网络中，而无需手动输入复杂的网络配置信息。
* **Wi-Fi 技术进行高速数据传输**：Wi-Fi 技术是一种通信技术，可以提供高速的无线网络连接，因此非常适合用于传输大量数据，例如高清视频和音频数据。在 Matter 中，设备可以通过 Wi-Fi 进行高速数据传输，以实现高质量的音视频体验。
* **Thread 协议进行低速数据传输**：Thread 协议是一种低功耗、安全、可靠的无线通信协议，它适用于智能家居设备之间的低速数据传输。在 Matter 中，设备可以使用 Thread 协议进行低速数据传输，例如传输传感器数据、控制指令等。

## Matter协议架构

### 1.Matter Over IPV6

该标准建立在一个共同的信念之上，即智能家居设备应该安全、可靠且无缝使用。通过建立在互联网协议 (IP) 之上，Matter 支持智能家居设备、移动应用程序和云服务之间的通信，并为设备认证定义了一组特定的基于 IP 的网络技术。

IPv6（Internet Protocol version 6）是互联网协议的一种，它是 IPv4 协议的后继者，当然并不是说这是一种全新的技术，更多的可以看作是IPV4 协议的扩展。IPv6 提供了更大的地址空间（128位）、更好的安全性（引入IPsec协议作为默认选项）、更高的性能和更多的扩展性，是未来互联网发展的重要基础。

下面是IPV4 和 IPV6 的一些区别：

|     区别     |                 IPV4                 |         IPV6         |
| :----------: | :----------------------------------: | :------------------: |
|   地址长度   |               32 bits                |       128 bits       |
|   地址数量   |             约**4x10^9**             |   约**3.4×10^38**    |
|   地址类型   |          公网地址和私有地址          |  全局地址和本地地址  |
| 地址分配方式 |          静态地址和动态地址          | 通过 DHCPv6 动态分配 |
|    安全性    | IPsec(Internet协议安全标准) 为可选项 |   IPsec 为默认选项   |
|     ---      |                 ---                  |         ---          |

### 2.Matter协议架构

Matter 旨在为智能家居设备构建一个通用的基于 IPv6 的通信协议。该协议定义了将部署在设备上的应用层和不同的链路层，以帮助维护互操作性。

<center>
    <img style="border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="https://cdn.jsdelivr.net/gh/kurisaW/picbed/img2023/202306121952614.png" width = "60%" alt=""/>
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">
      Matter、OSI 和 TCP/IP 层
  	</div>
</center>

为了解决网络通信壁垒，Matter网络层本身基于 IPV6，因此**天生具备IP连接能力**，可以与WIFI、Thread、以太网等通讯协议配合使用，而蓝牙则仅在配网过程使用；

Matter 还支持**桥接**等其他智能家居技术（例如 Zigbee、Bluetooth Mesh 和 Z-Wave）。这也就意味着，基于这些协议的设备可以像使用 Matter 设备一样运行**Bridge**；

由于Matter是基于应用层的协议，也就是说在未来即便有新的网络层协议的出现，Matter也可以很方便的兼容和支持到新协议，从长远发展来看具有很好的前瞻性！

### 3.Matter标准协议架构

<center>
    <img style="border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="https://cdn.jsdelivr.net/gh/kurisaW/picbed/img2023/202306121138795.png" width = "110%" alt=""/>
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">
      Matter标准协议架构总体流程
  	</div>
</center>

**Matter标准协议架构总体流程分析：**

首先使用Interaction Model构建一个Action；在Action Framing这一层中，该Action会被序列化为一份指定的压缩二进制格式，表示可以在设备上执行设备交互的一组操作；处理后的Action帧通过Security层进行加密和签名处理，确保通信双方信息传输的机密性和可靠性；当Action经过序列化、加密和签名后，Message Layer会指定一份必选及可选的头字段构造Payload格式，其中头字段中包含了规定消息的属性及一些逻辑路由信息；当payload被 Message Layer 层构造后, 会使用基于IP的数据传输协议 (TCP协议或Matter的消息可靠协议[Message Reliability Protocol]())；一旦对方设备收到数据后，数据流则沿着协议栈向上移动，即各个层反转发送方对数据执行的操作，最终将消息传递给应用程序。

后面我们会重点讲解设备数据模型（Data Model）和互动模型（Interaction Model），这两部分是Matter互联互通的前提！

## Matter网络拓扑结构

原理上，任何支持IPV6协议的网络都可以部署Matter，我们重点关注三种链路层技术：以太网（Ethernet）、WIFI和 Thread。

在 Matter 协议中，Matter将网络视为共享资源，它不规定独占网络的所有权或访问权。因此我们可以在同一组成IP的网络下覆盖多个Matter网络。

Matter协议还可以在没有公网IPv6基础设施的情况下运行，经资料查询得知，主要是因为**Matter协议也支持Thread网络协议，其底层是基于IEEE 802.15.4的，并使用了6LoWPAN作为IPv6的适配层**。而 **6LoWPAN协议** 提供了一种在低功耗无线传感器网络中使用IPv6的方法，它可以将IPv6数据包压缩到非常小的尺寸，从而使得这些数据包可以在不需要较大的IP地址空间的情况下传输。这使得Matter设备可以使用私有IPv6地址而不需要公共IPv6地址，因此不需要依赖公网IPv6基础设施。

因此，Matter协议不需要依赖公网IPv6基础设施，也不需要依赖互联网服务提供商的支持，可以在与公网断开连接或有防火墙的网络中操作，这使得它可以在更广泛的场景下进行部署和使用。

### Mesh组网

在了解Matter网络拓扑结构之前，我们可以先来了解下 Mesh 组网。

目前最流行的全屋WiFi方案主要有两种：**Mesh路由器组网**和**AC+AP**两种方案。而Mesh路由器组网由于其实惠的价格和较为稳定的链路连接性能以及安装的简便性，目前在全屋智能网络的选择还是比较热门的。

无线Mesh网络是一种新无线局域网类型，与传统WLAN不同的是，**无线Mesh网络**中的**AP**可以采用**无线连接**的方式进行互连，并且**AP间可以建立多跳的无线链路**。简单来说，就是当WIFI覆盖不了的时候，在有WIFI信号的时候放置一个路由器，可以作为Mesh路由的中继节点，透过这个节点，将WIFI信号覆盖到所有需要覆盖的地方；是一个动态的可以不断扩展的网络架构，任意的WIFI节点设备均可以保持无线互联。

<center>
    <img style="border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="https://cdn.jsdelivr.net/gh/kurisaW/picbed/img2023/202306132314987.png" width = "90%" alt=""/>
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">
      Mesh组网
  	</div>
</center>

这个很直观的体现就是大学里每层走廊中间都会架设一台路由，而你每移动一个楼层，你手机的校园网都会重新连接，也就是手机信号会快速自动重连距离你最近的一台路由，这就构成了一个庞大的无线链路网络。下面我们再来了解下Matter 的网络拓扑结构主要分为单一网络拓扑和星形网络拓扑：

### 1.单一网络拓扑

<center>
    <img style="border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="https://cdn.jsdelivr.net/gh/kurisaW/picbed/img2023/202306121601076.png" width = "30%" alt=""/>
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">
      单一Thread网络拓扑
  	</div>
</center>

<center>
    <img style="border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="https://cdn.jsdelivr.net/gh/kurisaW/picbed/img2023/202306121516744.png" width = "30%" alt=""/>
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">
      单一WIFI网络拓扑
  	</div>
</center>


在单一网络拓扑中，所有的 Matter 设备都连接到一个单一的逻辑网络。 它可以是**Thread/802.15.4网络**、**Wi-Fi网络**或**以太网网络**。在 Wi-Fi/以太网的情况下，网络实际上可以跨越多个Wi-Fi和/或以太网段，**前提是所有段都在链路层桥接**。 节点(Node)是Fabric中的 Matter设备的单个实例，可在IP网络上运行。

在单一网络拓扑中的每个节点都通过单个网络接口与Fabric中的每个其他节点进行通信。

在Matter 中，分属不同网络的设备可以进行同端通信，这也就意味着一个WIFI设备可以和一个Thread进行相互的信息转发，而Matter则扮演了一个虚拟网络的身份，并称其为**Fabric**。

> 注：Fabric是共享同一个Trusted Root的Matter设备的集合。Matter中Trusted Root作为根CA，颁发NOC证书，识别节点身份。在一个Fabric内，每个节点都有一个唯一标识Node ID。Fabric作为一个命名空间来管理所有权，在Fabric范围内使用标识符确保资源的分配和选择的唯一性。

### 2.星形网络拓扑

* **AP(Access Point)**：WI-FI无线接入点，AP 负责向 STA 提供 Wi-Fi 信号，并提供连接互联网的网络服务。
* **STA(Station)**：STA 是 Wi-Fi 中的无线客户端，即 Station。STA 可以是智能手机、平板电脑、笔记本电脑等各种设备，它们可以通过 Wi-Fi 连接到无线接入点，访问互联网或者局域网中的资源。
* **BR(Border Router)**：指的是边界路由器，BR 是一种网络设备，可以连接两个或多个 IP 子网，并将它们转换为同一个 Thread 网络，使得不同子网中的设备可以互相通信。BR 是 Thread 网络中的核心设备之一，通常由路由器或者网关设备提供。
* **ED(End device)**：指的是终端设备，ED 是 Thread 网络中的客户端设备，如智能手机、平板电脑、笔记本电脑等。ED 可以直接连接到 BR 或者 R，也可以通过其他设备中继进行通信。
* **R(Router)**：指的是内部路由器。R 是一种网络设备，可以连接多个 ED 和其他 R，负责在 Thread 网络中进行路由选择和数据转发。
* **SED(Sleepy End Device)**：指的是低功耗终端设备。SED 是一种特殊的终端设备，通常采用低功耗的无线技术，可以在不需要进行通信时进入睡眠模式，从而延长电池寿命。SED 可以直接连接到 BR 或者 R，也可以通过其他设备中继进行通信。

<center>
    <img style="border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="https://cdn.jsdelivr.net/gh/kurisaW/picbed/img2023/202306121605090.png" width = "100%" alt=""/>
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">
      星形网络拓扑
  	</div>
</center>
星形网络拓扑由多个外围网络组成，这些网络通过Hub连接在一起。Hub通常是客户家庭网络（Wi-Fi/以太网）中的设备，而外围网络可以是任何支持的网络类型。**外围网络必须始终通过一个或多个边界路由器(Border Router)直接连接到Hub。**

在架构上，任何数量的外围网络可以存在于单个Fabric中，包括相同类型的多个网络。节点可以具有到任何网络（Hub或外围设备）的接口，并且可以直接与同一网络上的其他节点通信。然而，任何必须跨越网络边界才能到达目的地的通信必须通过边界路由器(Border Router)。

该协议对边界路由器提出了一系列要求。这些要求涉及地址分配、路由分配和广播、多播支持和代理发现。

> 注：在现Matter1.0版本规范中，Thread是主要支持的LLN（Low-Power and Lossy Network）。在许多情况下，客户安装将尝试维护一简单的网络拓扑，包括一个Wi-Fi/以太网子网和一个单Thread网络。但是，可以支持多个Thread网络。

## 设备数据模型（Date Model）

在 Matter 中的设备具有明确定义的**数据模型** (**DM**)，这是对设备功能的分层建模。在此层次结构的顶层，有一个**Device**。

### 1.设备和端点（Node、Endpoint）

所有设备（包括智能手机和家居助理）均由**Node（节点）**组成。“节点”是网络中可以标识为唯一且可寻址的资源，用户可以感知到整个功能。Matter 中的网络通信始于和终止节点。

一组节点包含了多组**Endpoint（端点）**。**而每个端点都封装了一个功能集**。例如，端点1可能涉及照明功能，而端点2可能涉及移动侦测，以及其他与实用程序（例如设备 OTA）的处理方式。

<center>
    <img style="border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="https://cdn.jsdelivr.net/gh/kurisaW/picbed/img2023/202306122042737.png" width = "60%" alt=""/>
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">
      设备、节点和端点
  	</div>
</center>
### 2.节点角色（Node roles）

在Matter 中，每一个物理设备都被称之为**Node**，Node 使用**Node ID（64bit）**来进行表示，在Fabric范围内是唯一的！

**Node roles**是一组相关的行为。每个节点可能有一个或多个role。Node roles 包括：

- **Commissioner ：执行**[调试](https://developers.home.google.com/matter/primer/commissioning)的节点 。
- **控制器**：可以控制一个或多个节点的节点。例子包括Google Home app (GHA), Google Assistant, 和Google Nest Hub (2nd gen). 某些设备类型（例如[开/关灯开关](https://developers.home.google.com/matter/supported-devices#onoff_light_switches)）具有控制器角色。
- **Controlee** : 可以被一个或多个节点控制的节点。大多数设备类型都可以是 Controlee，除了一些具有 Controller 角色的设备类型，例如[On/Off Light Switch](https://developers.home.google.com/matter/supported-devices#onoff_light_switches)。开/关灯开关只能*是*控制器。它不能是受控人。
- **OTA Provider** : 可以提供 OTA 软件更新的节点。
- **OTA 请求者**：可以请求 OTA 软件更新的节点。

### 3.集群（Cluster）

在一个**Endpoint**中，一个 Node 有一个或多个**Clusters**。这些是设备层次结构中的另一个步骤，因为它们将特定功能分组，例如 智能插头上的*开/关*集群，或可调光端点上的*电平控制集群。*

一个节点也可能有多个端点，每个端点都创建一个具有相同功能的实例。例如，灯具可能会暴露对单个灯的独立控制，或者电源板可能会暴露对单个插座的控制。

#### 3.1 属性（Attributes）

在最后一层，我们会找到**Attributes**，这是节点持有的状态，表示可以读取或写入的内容，支持多种数据格式，实际中代表了智能设备的相关属性(如门的开关、室内温度等)。

<center>
    <img style="border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="https://cdn.jsdelivr.net/gh/kurisaW/picbed/img2023/202306122111729.png" width = "60%" alt=""/>
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">
      节点、端点、属性和命令
  	</div>
</center>

#### 3.2 命令（Commands）

除了 Attributes 之外，Clusters 还有**Commands**，也就是**触发 Cluster 进行某种行为的指令**。它们等同于Matter远程过程调用的 DM。命令类似于*动词*，例如*Door Lock*集群上的 *lock door*。命令可能会产生响应和结果；在 Matter，这样的响应也被定义为命令，以相反的方向进行。

#### 3.3 事件（Events）

最后，Clusters 也可能有**Events**，它**可以被认为是过去状态转换的记录**。虽然属性代表**当前状态**，但事件是**过去**的日志，包括单调递增的计数器、时间戳和优先级。它们能够捕获状态转换，以及使用属性不容易实现的数据建模。

<center>
    <img style="border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="https://cdn.jsdelivr.net/gh/kurisaW/picbed/img2023/202306122122628.png" width = "100%" alt=""/>
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">
      层次结构示例：Matter设备交互模型
  	</div>
</center>
`Endpoint 0`作为`Utility Clusters`**保留。Utility Clusters 是特定的集群，它包含端点上的服务功能，例如发现、寻址、诊断和软件更新**。另一方面，**Application(应用集群)**支持主要操作，例如开/关或温度测量。

### 4. Cluster分类

cluster可以定义为**工具(Utility) Cluster**或**应用(Application) Cluster**。

#### 4.1 工具(Utility) Cluster

工具cluster不是端点的主要应用程序操作的一部分。它可以用于配置、发现、寻址、诊断、监控设备运行状况、软件更新等。它可能与对应的cluster存在临时关系。

> 作用域为端点的工具cluster示例:标识符、描述符、绑定、组等。 适用于该节点的工具cluster
> 示例:基本信息、诊断等。

#### 4.2 应用(Application) Cluster

应用cluster支持端点的主要操作。应用cluster可以支持和一个或多个应用程序交互，既包括client也包括server。

> 应用cluster示例:
>
> * On/Off cluster —— client向server发送命令
> * Temperature Measurement cluster —— server向client报告数据

应用程序cluster不是工具cluster，即使它本身可能支持实用的工具功能，如校准、操作模式等。但应用程序cluster规范不应该涉及其应用领域之外的层级和过程。

> 示例：一个特定的温度测量cluster可能存在于不同的设备上，或在不同的网络中，每个设备具有不同的安全与配网机制和/或策略。
> 示例：commissioning cluster的范围是配网，而不是测温。

### 5. Clients and Servers

Clusters 可能是**Client Cluster**或**Server Cluster**。服务器是**有状态的**，保存属性、事件和命令；而客户端是 **无状态的**，其职责是启动与远程服务器集群的**交互**，从而执行：

- **读取**和**写入**其远程属性。
- **读取**其远程事件。
- **调用**其远程命令。

虽然 DM 在节点内是分层的，但节点之间的关系不是。Matter中的节点没有`controller/peripheral` 或 `leader/follower`关系。相反，关系是水平的：任何 Cluster 都可以是**Server**或**Client**。因此，**对于不同的集群和功能，节点可能既是服务器又是客户端。**

例如，我们可能有两个台灯：**节点 A**和**节点 B**。两个节点都实现了一个*开/关灯*设备类型。此设备类型包括控制其各自物理光输出的*开/关服务器集群。*

但是，就像典型的台灯一样，我们的物理设备还将包括一个开/关灯 开关设备类型，用于其本地开/关。此设备类型必须实现*开/关客户端*集群，以便它可以控制服务器集群。

<center>
    <img style="border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="https://cdn.jsdelivr.net/gh/kurisaW/picbed/img2023/202306122240843.png" width = "100%" alt=""/>
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">
      客户端与服务端集群
  	</div>
</center>

在此示例中，节点 A 上的开/关客户端集群正在更改节点 A 和节点 B 上的开/关服务器集群的属性，而节点 B 的客户端集群仅更改节点 B 本身上的服务器集群。

在下一节中，我们将详细介绍客户端和服务器集群如何交互： **Interaction Model（交互模型）**。

## 交互模型

### 1.概念

如果我们不能对节点执行操作，那么节点的数据模型 (DM) 就不相关了。**交互模型**（**IM**），定义了一个节点的 DM 与其他节点的 DM 的关系：即 IM 作为 DM 之间通信的通用语言。

**节点通过以下方式相互交互：**

- 读取和订阅属性和事件
- 写入属性
- 调用命令

每当一个节点与另一个节点建立加密通信序列时，它们就构成了**交互**关系。**Interactions 可能由一个或多个Transactions组成，而 Transactions 由一个或多个Action组成**，可以理解为 Node 之间的 IM 级消息。

<center>
    <img style="border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="https://cdn.jsdelivr.net/gh/kurisaW/picbed/img2023/202306140839728.png" width = "100%" alt=""/>
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">
      交互模型的层次结构
  	</div>
</center>

Matter 支持多个操作，例如从另一个节点请求属性或事件的读取请求操作，或其响应，报告数据操作，它将信息从服务器返回到客户端。

#### 1.1 发起者（Initiators ）和目标（Targets）

在Matter中，节点的发起目标被称为**发起者（Initiators ）**，而响应的节点则作为**目标（Target）**。一般来说，发起者是客户端集群，而目标是客户端集群。

#### 1.2 组（Groups）

在Matter中节点可能隶属于某个组。设备组作为一种机制，主要用于在统一操作中同时寻址并向多个设备发送消息。在一个 Group 中，所有的节点共享同一个 Group ID（16位整型）。

为了完成组级通信（群播），Matter 利用IPV6 多播消息，并且让所有的组成员都具有相同的多播地址。

#### 1.3 路径（Path）

**当我们想要与属性、事件或命令进行交互时，我们需要为这种交互指定 Path ，也就是属性、事件和命令在节点的数据模型层次结构中的位置。**

> 注：Path 也可以使用**Groups**或者**统配交互符（Wildcard Operators）**同时处理多个节点或集群，从而减少操作的数量。

Path这种机制对提高通信的响应能力起到很重要的作用。例如：当用户想要关闭所有灯光，语音助手可以与组内多个灯建立单个的交互，而不是传统的一系列单独的交互。

Matter Path 使用规范：

```base
<path> = <node> <endpoint> <cluster> <attribute | event | command>
<path> = <group ID>        <cluster> <attribute | event | command>
```

在这些路径构建块中，端点和集群还可能包括用于选择多个节点实例的通配符运算符。

#### 1.4 定时和非定时（Timed & Untimed）

有两种执行写入或调用 Matter 的方式：定时的和非定时的。定时交易为写入/调用动作的发送建立了一个最大的超时。这个超时的目的是为了防止对交易的拦截攻击。它特别适用于对资产进行门禁的设备，如车库开门器和锁。

### 2. Read Transactions

与 Nodes 交互时的第一个用例 Matter是从另一个节点读取的属性，例如来自传感器的温度值。在此类交互中，必须执行的第一个操作是读取请求操作。

#### 2.1 读取请求操作（Read Request Action）

> **发起者 -> 目标**

在此 Action 中，Initiator 会查询 Target 提供的以下请求：

- **属性请求**：零个或多个目标属性的列表。该列表由零个或多个目标请求属性的路径组成。
- **事件请求**：目标请求事件的零个或多个路径列表。

目标接收到读取请求操作后，它将使用请求的信息组装一个报告数据操作；当目标接收到读取请求操作后，它将使用请求的信息组装一个报告数据操作。详见下图：

<center>
    <img style="border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="https://cdn.jsdelivr.net/gh/kurisaW/picbed/img2023/202306141108121.png" width = "80%" alt=""/>
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">
      Read Transaction
  	</div>
</center>
#### 2.2 报告请求数据（Report Data Action）

> **目标 -> 发起者**

在此 Action 中，Target 响应：

- **属性报告（Attribute Reports）**：读取操作请求中请求的零个或多个报告属性的列表。
- **事件报告（Event Reports）**：零个或多个报告事件的列表。
- **抑制响应（Suppress Response）**：一个标志，用于确定是否应抑制对此操作的**状态响应。**
- **订阅 ID（Subscription ID）**：如果此报告是订阅交易的一部分，它必须包含一个用于识别订阅交易的整数。

#### 2.3 状态响应动作（Status Response Action）

> **目标 -> 发起者 -> 目标**

一旦 Initiator 接收到请求的数据，默认情况下它必须生成一个 Status Response Action。此操作由启动器发送，确认已收到报告的数据。如果设置了 Suppress Status Response 标志，则 Initiator 不得发送 Status Response Action。

一旦启动器发送了状态响应操作，或者启动器接收到启用了抑制响应标志的报告数据操作，读取/报告查询就完成了。

状态响应操作仅包含一个**状态字段**，该字段将确认操作成功或显示失败代码。

### 3. Subscription Transaction

#### 3.1 订阅请求操作（Subscribe Request Action）

> **发起者 -> 目标**

除了单一的读请求动作外，发起者还可以订阅属性或事件的定期更新。因此，同样的报告数据 Action 可以作为订阅交易后的定期数据更新的结果而产生。

订阅交互创建两个节点之间的关系，其中目标定期向发起者生成报告数据操作。 Initiator 是 Subscriber，Target 是 Publisher。

订阅请求操作包含：

- **Min Interval Floor（最小间隔层）**：报告之间的最小间隔。
- **Max Interval Ceiling（最大区间上限）**：报告之间的最大间隔。
- Attribute Reports（属性报告）：读取操作请求中请求的零个或多个报告属性的列表。
- Event Reports（事件报告）：零个或多个报告事件的列表。

<center>
    <img style="border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="https://cdn.jsdelivr.net/gh/kurisaW/picbed/img2023/202306141332135.png" width = "60%" alt=""/>
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">
      Subscription Transaction
  	</div>
</center>
在订阅请求之后，目标用包含第一批报告数据的报告数据操作响应发起者：**Primed Published Data**。

然后，发起者通过发送到目标的状态响应操作来确认报告数据操作。一旦目标接收到一个状态响应动作报告没有错误，它发送一个订阅响应动作。

目标随后将以协商的间隔定期发送报告数据操作，发起者将响应这些操作，直到订阅丢失或取消。

#### 3.2 订阅响应操作（Subscribe Response Action）

> **目标 -> 发起者**

这是订阅交易的最后一个操作，并结束了该过程。这包括：

- **Subscription ID（订阅 ID）**：标识订阅的整数。
- **Min Interval（最小间隔）**：最终确定的报告之间的最小间隔。
- **Max Interval（最大间隔）**：*最终*确定*的*报告之间的最大间隔。

### 4. Write Transactions

#### 4.1 不定时写入事务（Untimed Write Transaction）

##### 4.1.1 写请求操作（Write Request Action）

> **发起者 -> 目标**

与读取请求操作类似，在此操作中，发起者为目标提供：

- **Write Requests（写入请求）**：包含路径和数据的一个或多个元组的列表。
- **Timed Request（定时请求）**：一个标志，指示此操作是否是定时写入事务的一部分。
- **Suppress Response（抑制响应）**：指示是否应抑制响应状态操作的标志。

<center>
    <img style="border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="https://cdn.jsdelivr.net/gh/kurisaW/picbed/img2023/202306141423081.png" width = "80%" alt=""/>
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">
      Untimed Write Transaction
  	</div>
</center>

##### 4.1.2 写响应操作（Write Response Action）

> **目标 -> 发起者**

##### 4.1.3 不定时写入限制（Untimed Write Restrictions）

写入请求动作可以是一个组播，但在这种情况下，必须设置抑制响应标志。其理由是，否则网络可能会被来自一个组的每个成员的同时响应所淹没。

为了启用这种行为，在写请求列表中使用的路径可以包含组，或者它们可以包含通配符，但只在端点字段上。

#### 4.2 定时写入事务（Timed Write Transaction）

在定时写入事务中比非定时写入事务多了几个步骤。

##### 4.2.1 定时请求操作（Timed request action）

> **发起者 -> 目标**

Initiator 启动事务发送此操作，其中包含：

- **Timeout**：此事务可以保持打开状态的毫秒数。在此期间，Initiator 发送的下一个动作将被视为有效。

一旦接收到定时请求操作，目标必须使用状态响应操作确认定时请求操作。一旦 Initiator 收到报告没有错误的 Status Response Action，它将发送 Write Request Action。

##### 4.2.2 写请求操作（Write Request Action）

与前面描述的 **4.1.1 写请求操作** 相同。

##### 4.2.3 写响应操作（Write Response Action）

与前面描述的 **4.1.2 写响应操作** 相同。

##### 4.2.4 定时写入限制（Timed Write Restrictions）

定时请求动作、写请求动作和写响应动作是单播的。

### 5.调用事务

**调用事务**用于在目标节点上调用一个或多个集群命令。它类似于对集群中定义的命令进行的远程过程调用。

与写入事务类似，调用事务支持定时和不定时事务。 有关定时事务的更多信息，请参阅 **交互模型：1.4.定时和非定时**

#### 5.1 不定时调用事务

##### 5.1.1 调用请求操作（Invoke Request Action）

> **发起者 -> 目标**

类似于读请求动作和写请求动作，在这个动作中，发起者为目标提供：

- **Invoke Requests（调用请求）：集群命令**的路径（PATH）列表 ，以及命令的可选参数，名为 **Command Fields**。
- Timed Request（超时请求）：一个标志，指示此操作是否是定时调用事务的一部分。
- Suppress Response（抑制响应）：指示是否应抑制调用响应操作的标志。
- **Interaction ID**：一个整数，用于将 Invoke Request Action 与 Invoke Response Action 匹配。

##### 5.1.2 调用响应操作（Invoke Response Action）

> **目标 -> 发起者**

目标收到调用请求操作后，它将使用包含以下内容的调用响应操作来完成事务：

- **Invoke Responses（调用响应）**：发送的每个调用请求的命令响应或状态列表。
- Interaction ID：一个整数，用于将 Invoke Response Action 与 Invoke Request Action 匹配。

##### 5.1.3 不定时调用限制

Invoke Request Action可以是一个组播，但在这种情况下，必须设置抑制响应标志。其理由是，否则网络可能会被来自一个组的每个成员的同时响应所淹没。

为了启用这种行为，在调用请求列表中使用的路径可以包含组，或者它们可以包含通配符，但仅在端点字段上。此外，如果行动是组播，这个事务就会在没有响应的情况下终止。

#### 5.2 定时调用事务

与定时写入事务类似，定时调用事务也从定时请求操作开始。

##### 5.2.1 定时请求操作

> **发起者 -> 目标**

Initiator 启动事务发送此操作，其中包含：

- **Timeout**：此事务可以保持打开状态的毫秒数。在此期间，Initiator 发送的下一个动作将被视为有效。

一旦接收到定时请求操作，目标必须使用状态响应操作确认定时请求操作。一旦 Initiator  收到状态响应操作报告没有错误，它将发送调用请求操作。

<center>
    <img style="border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="https://cdn.jsdelivr.net/gh/kurisaW/picbed/img2023/202306141539988.png" width = "80%" alt=""/>
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">
      Timed Invoke Transaction
  	</div>
</center>

##### 5.2.2 调用请求操作（Invoke Request Action）

与前面描述的 **5.1.1 调用请求操作** 相同。

##### 5.2.3 调用响应操作（Invoke Response Action）

与前面描述的 **5.1.2 调用响应操作** 相同。

##### 5.2.4 定时调用限制（Timed Invoke Restrictions）

所有的调用命令都可以在定时交互中调用。定时请求动作、调用请求动作和调用响应动作都是单播的，因此不能在定时调用事务上作为群播使用。

Invoke Request Action支持使用带组的路径，以及通配符，但Invoke Response Action不支持通配符的使用。

---

## 参考资料

* [https://developers.home.google.com/matter/primer/device-data-model#parts_list](https://developers.home.google.com/matter/primer/device-data-model#parts_list)
* [Matter技术及关键特性介绍](https://www.bilibili.com/video/BV1NL411T7Kj/?spm_id_from=333.788&vd_source=40334d7415493efea293dacb3c13f0b4)
