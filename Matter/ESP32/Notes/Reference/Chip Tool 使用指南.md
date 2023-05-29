# 使用 CHIP TOOL

CHIP 工具 ( `chip-tool`) 是一个 Matter 控制器实现，它允许将 Matter 设备委托到网络中并使用 Matter 消息与其通信，该消息可以对数据模型操作进行编码，例如集群命令。

该工具还提供了其他特定于 Matter 的实用程序，例如解析设置有效负载或执行发现操作。

---

## 1. 源文件

您可以在目录中找到 CHIP 工具的源文件`examples/chip-tool` 。

> **注意：** CHIP 工具将配置状态缓存在 `/tmp/chip_tool_config.ini`文件中。删除此文件和目录`.ini`中的其他文件 `/tmp`有时可以解决与陈旧配置相关的问题。

---

## 2. 构建和运行 CHIP 工具

在使用 CHIP 工具之前，您必须在 Linux (amd64/aarch64) 或 macOS 上从源代码编译它。如果你想在 Raspberry Pi 上运行它，它必须使用 64 位操作系统。

> **注意：**为确保兼容性，请始终从存储库的相同版本构建 CHIP 工具和 Matter 设备`connectedhomeip`。

### 1.1 构建 CHIP 工具

要构建和运行 CHIP 工具：

1. 为 Matter 安装所有必需的包并准备源代码和构建系统。阅读 [Building Matter](https://github.com/project-chip/connectedhomeip/blob/master/docs/guides/BUILDING.md)指南以获取说明。

2. 在目录中打开命令提示符`connectedhomeip`。

3. 运行以下命令：

   ```
   ./scripts/examples/gn_build_example.sh examples/chip-tool BUILD_PATH
   ```

   在此命令中，`BUILD_PATH`指定目标二进制文件的放置位置。

### 1.2 运行 CHIP 工具

要检查 CHIP 工具是否正确运行，请从目录中执行以下命令 `BUILD_PATH`：

```
$ ./chip-tool
```

因此，CHIP 工具会打印所有可用的命令。这些在此上下文中称为 *集群*，但并非所有列出的命令都对应于 数据模型中的*集群*（例如，配对或发现命令）。但是，每个列出的命令都可以通过附加子命令成为新的更复杂命令的根。[支持的命令和选项](https://github.com/project-chip/connectedhomeip/blob/master/docs/guides/chip_tool_guide.md#supported-commands-and-options)部分描述了特定命令及其用例的示例 。

---

## 3. 使用 CHIP 工具进行物质设备测试

以下步骤取决于您在设备上实施的应用程序集群。

这些步骤使用 具有蓝牙 LE 调试方法支持的[Matter Lighting 应用示例。](https://github.com/project-chip/connectedhomeip/tree/master/examples/lighting-app)您可以使用其他 Matter 示例并仍然遵循此过程。如果您使用不同的示例，则 [步骤 7](https://github.com/project-chip/connectedhomeip/blob/master/docs/guides/chip_tool_guide.md#step-7-control-application-data-model-clusters)可能会因应用程序中实施的集群而异。

### 3.1 准备 Matter 设备

[按照Matter Lighting Application Example](https://github.com/project-chip/connectedhomeip/tree/master/examples/lighting-app)文档使用 Matter 设备固件构建和编程设备 。

### 3.2 在 Matter 设备上启用蓝牙 LE 广播

一些示例配置为在启动时自动通告。其他示例需要物理触发，例如按下按钮。按照所选平台的 Matter 设备示例文档，了解如何为给定示例启用蓝牙 LE 广播。

### 3.3 设置 IP 网络

要执行后续步骤，IP 网络必须已启动并正在运行。例如，可以使用 [OpenThread Border Router](https://openthread.io/codelabs/openthread-border-router#0)建立 Thread 网络。

### 3.4 确定网络配对凭据

您必须向 CHIP 工具提供网络凭据，这些凭据将在设备调试过程中使用，以使用网络接口（例如 Thread 或 Wi-Fi）配置设备。

Matter 规范没有定义控制器如何获取网络凭证的首选方式。在本指南中，我们将提供获取 Thread 和 Wi-Fi 网络凭证的步骤。

#### 3.4.1 获取 Thread 网络凭证

从 Thread Border Router 获取并存储当前的活动操作数据集。此步骤可能因 Thread Border Router 实现而异。

如果您使用的是 [OpenThread 边界路由器](https://openthread.io/codelabs/openthread-border-router#0) (OTBR)，请使用以下命令之一检索此信息：

- 对于在 Docker 中运行的 OTBR：

  ```
  sudo docker exec -it otbr sh -c "sudo ot-ctl dataset active -x"
  0e080000000000010000000300001335060004001fffe002084fe76e9a8b5edaf50708fde46f999f0698e20510d47f5027a414ffeebaefa92285cc84fa030f4f70656e5468726561642d653439630102e49c0410b92f8c7fbb4f9f3e08492ee3915fbd2f0c0402a0fff8
  Done
  ```

- 对于 OTBR 本机安装：

  ```
  sudo ot-ctl dataset active -x
  0e080000000000010000000300001335060004001fffe002084fe76e9a8b5edaf50708fde46f999f0698e20510d47f5027a414ffeebaefa92285cc84fa030f4f70656e5468726561642d653439630102e49c0410b92f8c7fbb4f9f3e08492ee3915fbd2f0c0402a0fff8
  Done
  ```

对于 Thread，您还可以使用不同的带外方法来获取网络凭证。

#### 3.4.2 获取 Wi-Fi 网络凭证

您必须获得以下 Wi-Fi 网络凭据才能将 Matter 设备调试到 Wi-Fi 网络：

- 无线网络名称
- 无线网络密码

确定 SSID 和密码所需的步骤可能因设置而异。例如，您可能需要联系您当地的 Wi-Fi 网络管理员。

### 3.5 确定 Matter 设备的鉴别器并设置 PIN 码

Matter 使用以下值：

- 鉴别器 - 一个 12 位值，用于区分多个可委托设备广告。
- 设置 PIN 码 - 用于验证设备的 27 位值。

当设备启动时，您可以在设备的日志记录终端（例如 UART）中找到这些值。例如：

```
I: 254 [DL]Device Configuration:
I: 257 [DL] Serial Number: TEST_SN
I: 260 [DL] Vendor Id: 65521 (0xFFF1)
I: 263 [DL] Product Id: 32768 (0x8000)
I: 267 [DL] Hardware Version: 1
I: 270 [DL] Setup Pin Code: 20202021
I: 273 [DL] Setup Discriminator: 3840 (0xF00)
I: 278 [DL] Manufacturing Date: (not set)
I: 281 [DL] Device Type: 65535 (0xFFFF)
```

在此打印输出中，鉴别器是`3840 (0xF00)`，设置 PIN 码是 `20202021`。

### 3.6 将 Matter 设备委托到现有 IP 网络中

在与 Matter 设备通信之前，首先它必须加入现有的 IP 网络。

Matter 设备可以使用不同的调试通道：

- 尚未连接到目标 IP 网络的设备使用低功耗蓝牙作为调试通道。
- 已经加入 IP 网络的设备只需要使用 IP 协议调测到 Matter 网络即可。

#### 3.6.1 通过蓝牙 LE 进行调试

在这种情况下，您的设备可以通过蓝牙 LE 加入现有的 IP 网络，然后进入 Matter 网络。

Thread 和 Wi-Fi 网络可使用不同的场景，如以下小节所述。

通过蓝牙 LE 连接设备后，控制器打印以下日志：

```
Secure Session to Device Established
```

此日志消息表示已建立使用 SPAKE2 + 协议的 PASE（密码验证会话建立）会话。

##### 3.6.2 通过蓝牙 LE 调试到 Thread 网络

要将设备调试到现有的 Thread 网络，请使用以下命令模式：

```
$ ./chip-tool pairing ble-thread <node_id> hex:<operational_dataset> <pin_code> <discriminator>
```

在这个命令中：

- *<node_id>*是用户定义的被委托节点的 ID。
- *<operational_dataset>是在*[步骤 4](https://github.com/project-chip/connectedhomeip/blob/master/docs/guides/chip_tool_guide.md#step-4-determine-network-pairing-credentials)中确定的操作数据集 。
- *<pin_code>*和*<discriminator>是在*[步骤 5](https://github.com/project-chip/connectedhomeip/blob/master/docs/guides/chip_tool_guide.md#step-5-determine-matter-devices-discriminator-and-setup-pin-code)中确定的特定于设备的密钥 。

##### 3.6.3 通过蓝牙 LE 调试到 Wi-Fi 网络

要将设备调试到现有 Wi-Fi 网络，请使用以下命令模式：

```
$ ./chip-tool pairing ble-wifi <node_id> <ssid> <password> <pin_code> <discriminator>
```

在这个命令中：

- *<node_id>*是用户定义的被委托节点的 ID。
- *<ssid>*和*<password>*是在步骤 3 中确定的凭据。
- *<pin_code>*和*<discriminator>是在*[步骤 5](https://github.com/project-chip/connectedhomeip/blob/master/docs/guides/chip_tool_guide.md#step-5-determine-matter-devices-discriminator-and-setup-pin-code)中确定的特定于设备的密钥 。

如果您更喜欢十六进制格式，请使用`hex:`前缀。例如：

```
$ ./chip-tool pairing ble-wifi <node_id> hex:<ssid> hex:<password> <pin_code> <discriminator>
```

> **注意：** <node_id>可以作为带前缀*的*十六进制值提供`0x` 。

#### 3.6.4 通过 IP 调试网络

当 Matter 设备已存在于 IP 网络中但尚未委托给 Matter 网络时，此选项可用。

要调试设备，您可以使用设置 PIN 码或设置 PIN 码和鉴别器，这两者都是您在步骤 [5](https://github.com/project-chip/connectedhomeip/blob/master/docs/guides/chip_tool_guide.md#step-5-determine-matter-devices-discriminator-and-setup-pin-code)中获得的。或者，您也可以使用二维码负载。

##### 3.6.5 使用设置 PIN 码进行调试

要发现设备并尝试使用提供的设置代码与其中一个设备配对，请使用以下命令模式：

```
$ ./chip-tool pairing onnetwork <node_id> <pin_code>
```

该命令不断尝试设备，直到与其中之一成功配对或直到它用完配对可能性。在这个命令中：

- *<node_id>*是用户定义的被委托节点的 ID。
- *<pin_code>*是在 [步骤 5](https://github.com/project-chip/connectedhomeip/blob/master/docs/guides/chip_tool_guide.md#step-5-determine-matter-devices-discriminator-and-setup-pin-code)中确定的设备特定*设置 PIN 码* ，用于发现设备。

##### 3.6.6 长鉴别器调试

要发现具有长鉴别器的设备并尝试使用提供的设置代码与其中一个设备配对，请使用以下命令模式：

```
$ ./chip-tool pairing onnetwork-long <node_id> <pin_code> <discriminator>
```

该命令不断尝试设备，直到与其中之一成功配对或直到它用完配对可能性。在这个命令中：

- *<node_id>*是用户定义的被委托节点的 ID。
- *<pin_code>*和*<discriminator>是在*[步骤 5](https://github.com/project-chip/connectedhomeip/blob/master/docs/guides/chip_tool_guide.md#step-5-determine-matter-devices-discriminator-and-setup-pin-code)中确定的设备特定密钥 。

##### 3.6.7 使用二维码有效负载或手动配对码进行调试

Matter 设备在启动时会记录二维码负载和手动配对代码。

要根据给定的二维码有效负载或手动配对代码发现设备并尝试与其中之一配对，请使用以下命令模式：

```
$ ./chip-tool pairing code <node_id> <qrcode_payload-or-manual_code>
```

该命令不断尝试设备，直到与其中之一成功配对或直到它用完配对可能性。在这个命令中：

- *<node_id>*是用户定义的被委托节点的 ID。
- *<qrcode_payload-or-manual_code>*是二维码有效载荷 ID，例如 `MT:Y.K9042C00KA0648G00`，或手动配对代码，如 `749701123365521327694`。

#### 3.6.8 忘记已经调试的设备

如果需要重新测试调试，以下命令会从已调试的 Matter 设备列表中删除具有给定节点 ID 的设备：

```
$ ./chip-tool pairing unpair <node_id>
```

在此命令中，*<node_id>*是将被 CHIP 工具遗忘的节点的用户定义 ID。

### 3.7 控制应用程序数据模型集群

完成之前的所有步骤后，您已成功将 Matter 设备投入网络。您现在可以通过与数据模型集群交互来测试设备。

#### 3.7.1 示例：物质照明应用示例

对于我们在步骤 1 中引用的物质照明应用示例，该应用程序实现了开/关和电平控制集群。这意味着您可以通过切换灯泡（使用`onoff`集群命令）或操纵其亮度（使用`levelcontrol`集群命令）来测试它：

- 使用以下命令模式切换 OnOff 属性状态（例如通过 LED 状态可视化）：

  ```
  $ ./chip-tool onoff toggle <node_id> <endpoint_id>
  ```

  在这个命令中：

  - *<node_id>*是委托节点的自定义ID。
  - *<endpoint_id>*是实现了 OnOff 集群的端点的 ID。

- 使用以下命令模式更改 CurrentLevel 属性的值（例如通过 LED 亮度可视化）：

  ```
  $ ./chip-tool levelcontrol move-to-level <level> <transition_time> <option_mask> <option_override> <node_id> <endpoint_id>
  ```

  在这个命令中：

  - *<level>*`0`是在和之间编码的亮度级别`254`，除非在集群中配置了自定义范围。
  - *<transition_time>*是过渡时间。
  - *<option_mask>*是选项掩码。
  - *<option_override>*是选项覆盖。
  - *<node_id>*是委托节点的自定义ID。
  - *<endpoint_id>*是实现 LevelControl 集群的端点的 ID。

### 3.8 从Matter设备中读取基本信息

每个 Matter 设备都支持 Basic 集群，该集群维护控制器可以从设备获取的属性集合。这些属性可以包括供应商名称、产品名称或软件版本。

`read`在集群上使用 CHIP 工具的命令`basic`从设备读取这些值：

```
$ ./chip-tool basic read vendor-name <node_id> <endpoint_id>
$ ./chip-tool basic read product-name <node_id> <endpoint_id>
$ ./chip-tool basic read software-version <node_id> <endpoint_id>
```

在这些命令中：

- *<node_id>*是委托节点的自定义ID。
- *<endpoint_id>*是实现了基本集群的端点的 ID。

您还可以使用以下命令列出 Basic 集群的所有可用命令：

```
$ ./chip-tool basic
```

---

## 4. 支持的命令和选项

本节包含各种 CHIP 工具命令和选项的一般列表，不限于调试过程和集群交互。

### 4.1 交互模式与单命令模式

CHIP 工具可以以下列模式之一运行：

- 单命令模式（默认）- 在此模式下，如果任何单个命令未在特定超时期限内完成，CHIP 工具将退出并出现超时错误。

  超时错误类似于以下错误：

  ```
  [1650992689511] [32397:1415601] CHIP: [TOO] Run command failure: ../../../examples/chip-tool/commands/common/CHIPCommand.cpp:392: CHIP Error 0x00000032: Timeout
  ```

  此外，当使用单命令模式时，CHIP 工具将在发送每个命令时建立一个新的 CASE 会话。

- 交互模式 - 在此模式下，如果命令未在超时期限内完成，它将因错误而终止。但是，CHIP 工具不会终止，也不会终止先前命令启动的进程。而且，在使用交互模式时，CHIP Tool只有在没有会话可用时才会建立一个新的CASE会话。在以下命令中，它将使用现有会话。

#### 4.1.1 单命令模式下修改超时时间

可以通过提供可选参数来修改任何命令执行的超时时间 `--timeout`，该参数的值以秒为单位，最大值为 65535 秒。

**命令示例：**

```
$ ./chip-tool otasoftwareupdaterequestor subscribe-event state-transition 5 10 0x1234567890 0 --timeout 65535
```

#### 4.1.2 启动交互模式

对于事件订阅等需要长时间运行的命令，可以先以交互方式启动CHIP Tool，再运行该命令。

**命令示例：**

```
$ ./chip-tool interactive start
otasoftwareupdaterequestor subscribe-event state-transition 5 10 ${NODE_ID} 0
```

### 4.2 打印所有支持的集群

要打印 CHIP 工具支持的所有簇，请运行以下命令：

```
$ ./chip-tool
```

**输出示例：**

```
[1647346057.900626][394605:394605] CHIP:TOO: Missing cluster name
Usage:
  ./chip-tool cluster_name command_name [param1 param2 ...]

  +-------------------------------------------------------------------------------------+
  | Clusters:                                                                           |
  +-------------------------------------------------------------------------------------+
  | * accesscontrol                                                                     |
  | * accountlogin                                                                      |
  | * administratorcommissioning                                                        |
  | * alarms                                                                            |
  | * any                                                                               |
  | * appliancecontrol                                                                  |
  | * applianceeventsandalert                                                           |
  | * applianceidentification                                                           |
  | * appliancestatistics                                                               |
  | * applicationbasic                                                                  |
```

### 4.3 获取特定集群支持的命令列表

要打印特定集群支持的命令列表，请使用以下命令模式：

```
$ ./chip-tool <cluster_name>
```

在这个命令中：

- *<cluster_name>*是可用集群之一（用 列出 `chip-tool`）。

**命令示例：**

```
$ ./chip-tool onoff
```

**输出示例：**

```
[1647417645.182824][404411:404411] CHIP:TOO: Missing command name
Usage:
  ./chip-tool onoff command_name [param1 param2 ...]

  +-------------------------------------------------------------------------------------+
  | Commands:                                                                           |
  +-------------------------------------------------------------------------------------+
  | * command-by-id                                                                     |
  | * off                                                                               |
  | * on                                                                                |
  | * toggle                                                                            |
  | * off-with-effect                                                                   |
  | * on-with-recall-global-scene                                                       |
  | * on-with-timed-off                                                                 |
  | * read-by-id                                                                        |
  | * read                                                                              |
  | * write-by-id                                                                       |
  | * write                                                                             |
  | * subscribe-by-id                                                                   |
  | * subscribe                                                                         |
  | * read-event-by-id                                                                  |
  | * subscribe-event-by-id                                                             |
  +-------------------------------------------------------------------------------------+
[1647417645.183836][404411:404411] CHIP:TOO: Run command failure: ../../examples/chip-tool/commands/common/Commands.cpp:84: Error 0x0000002F
```

### 4.4 获取特定集群支持的属性列表

要获取特定集群的属性列表，请使用以下命令模式：

```
$ ./chip-tool <cluster_name> read
```

在这个命令中：

- *<cluster_name>*是可用集群之一（用 列出 `chip-tool`）。

**命令示例：**

```
$ ./chip-tool onoff read
```

**输出示例：**

```
[1647417857.913942][404444:404444] CHIP:TOO: Missing attribute name
Usage:
  ./chip-tool onoff read attribute-name [param1 param2 ...]

  +-------------------------------------------------------------------------------------+
  | Attributes:                                                                         |
  +-------------------------------------------------------------------------------------+
  | * on-off                                                                            |
  | * global-scene-control                                                              |
  | * on-time                                                                           |
  | * off-wait-time                                                                     |
  | * start-up-on-off                                                                   |
  | * server-generated-command-list                                                     |
  | * client-generated-command-list                                                     |
  | * attribute-list                                                                    |
  | * feature-map                                                                       |
  | * cluster-revision                                                                  |
  +-------------------------------------------------------------------------------------+
[1647417857.914110][404444:404444] CHIP:TOO: Run command failure: ../../examples/chip-tool/commands/common/Commands.cpp:120: Error 0x0000002F
```

### 4.5 获取命令选项列表

要获取特定命令的参数列表，请使用以下命令模式：

```
$ ./chip-tool <cluster_name> <target_command>
```

在这个命令中：

- *<cluster_name>*是可用集群之一（用 列出 `chip-tool`）。
- *<target_command>*是目标命令名称之一。

**命令示例：**

```
$ ./chip-tool onoff on
```

**输出示例：**

```
[1647417976.556313][404456:404456] CHIP:TOO: InitArgs: Wrong arguments number: 0 instead of 2
Usage:
  ./chip-tool onoff on node-id/group-id endpoint-id-ignored-for-group-commands [--paa-trust-store-path] [--commissioner-name] [--trace_file] [--trace_log] [--ble-adapter] [--timedInteractionTimeoutMs] [--suppressResponse]
[1647417976.556362][404456:404456] CHIP:TOO: Run command failure: ../../examples/chip-tool/commands/common/Commands.cpp:135: Error 0x0000002F
```

#### 4.5.1 选定的命令选项

本节列出了可用于配置输入命令的选定选项。

##### 4.5.1.1 选择蓝牙适配器

要选择 CHIP 工具使用的蓝牙适配器，请使用以下命令模式：

```
--ble-adapter <id>
```

在这个命令中：

- *<id>*是 HCI 设备的 ID。

**使用示例：**

```
$ ./chip-tool pairing ble-thread 1 hex:0e080000000000010000000300001335060004001fffe002084fe76e9a8b5edaf50708fde46f999f0698e20510d47f5027a414ffeebaefa92285cc84fa030f4f70656e5468726561642d653439630102e49c0410b92f8c7fbb4f9f3e08492ee3915fbd2f0c0402a0fff8 20202021 3840 --ble-adapter 0
```

##### 4.5.1.2 使用消息跟踪

消息跟踪允许捕获可用于测试自动化的 CHIP 工具安全消息。跟踪使用多种类型的标志来控制跟踪的去向。

以下标志可用：

- 跟踪文件标志：

  ```
  --trace_file <filename>
  ```

  此处，*<filename>*是存储跟踪数据的文件的名称。它可以通过以下方式附加到命令中：

  ```
  $ ./chip-tool pairing <pairing_options> --trace_file <filename>
  ```

- 跟踪日志标志：

  ```
  --trace_log <onoff>
  ```

  此处，*<onoff>*是一个`[0/1]`标志，设置为将`1`带有自动化日志的跟踪数据打印到控制台。

### 4.5.2 针对配对的对等设备运行测试套件

CHIP 工具允许针对配对的 Matter 设备运行一组已在工具中编译的测试。

- 要获取可用测试的列表，请运行以下命令：

  ```
  $ ./chip-tool tests
  ```

- 要对配对设备执行特定测试，请使用以下命令模式：

  ```
  $ ./chip-tool tests <test_name>
  ```

  在这个命令中：

  - *<test_name>*是特定测试的名称。

有关如何从测试套件运行测试的示例，请参阅[示例部分。](https://github.com/project-chip/connectedhomeip/blob/master/docs/guides/chip_tool_guide.md#running-testclusters-test)

### 4.5.3 解析设置负载

CHIP 工具提供了一个实用程序，用于解析 Matter onboarding 设置有效负载并以可读形式打印它。例如，负载可以在启动期间打印在设备控制台上。

要解析设置代码，请使用带有子命令`payload`的命令 `parse-setup-payload`，如以下命令模式所示：

```
$ ./chip-tool payload parse-setup-payload <payload>
```

这里，*<payload>*是要解析的payload的ID。

**命令示例：**

- 设置二维码负载：

  ```
  $ ./chip-tool payload parse-setup-payload MT:6FCJ142C00KA0648G00
  ```

- 手动配对代码：

  ```
  $ ./chip-tool payload parse-setup-payload 34970112332
  ```

### 4.5.4 解析额外的数据负载

要解析额外的数据负载，请使用以下命令模式：

```
$ ./chip-tool parse-additional-data-payload <payload>
```

在这个命令中：

- *<payload>*是带有要解析的附加数据的有效载荷的 ID。

### 4.5.5 发现动作

该`discover`命令可用于解析节点 ID 并发现可用的 Matter 设备。

使用以下命令打印命令的可用子命令`discover` ：

```
$ ./chip-tool discover
```

#### 4.5.5.1 解析节点名称

要解析与给定节点 ID 对应的 DNS-SD 名称并更新设备控制器中节点的地址，请使用以下命令模式：

```
$ ./chip-tool discover resolve <node_id> <fabric_id>
```

在这个命令中：

- *<node_id>*是要解析的节点ID。
- *<fabric_id>*是节点所属的 Matter fabric 的 ID。

#### 4.5.5.2 发现可用的 Matter 设备

要发现操作区域中可用的所有 Matter 佣金，请运行以下命令：

```
$ ./chip-tool discover commissionables
```

#### 4.5.5.3 发现可用的事务专员

要发现操作区域中可用的所有事务专员，请运行以下命令：

```
$ ./chip-tool discover commissioners
```

### 4.5.6 配对

该`pairing`命令支持有关 Matter 设备调试过程的不同方式。

Thread 和 Wi-Fi 调试用例在 [使用 CHIP 工具进行 Matter 设备测试](https://github.com/project-chip/connectedhomeip/blob/master/docs/guides/chip_tool_guide.md#using-chip-tool-for-matter-device-testing) 部分中进行了描述。

要列出所有`pairing`子命令，请运行以下命令：

```
$ ./chip-tool pairing
```

### 4.5.7 与数据模型集群交互

[如使用 CHIP 工具进行 Matter 设备测试](https://github.com/project-chip/connectedhomeip/blob/master/docs/guides/chip_tool_guide.md#using-chip-tool-for-matter-device-testing)部分所述 ，`chip-tool`使用特定集群名称执行命令会列出该集群支持的所有操作，如以下命令模式所示：

```
$ ./chip-tool <cluster_name>
```

**命令示例：**

```
$ ./chip-tool binding
```

**输出示例：**

```
[1647502596.396184][411686:411686] CHIP:TOO: Missing command name
Usage:
  ./chip-tool binding command_name [param1 param2 ...]

  +-------------------------------------------------------------------------------------+
  | Commands:                                                                           |
  +-------------------------------------------------------------------------------------+
  | * command-by-id                                                                     |
  | * read-by-id                                                                        |
  | * read                                                                              |
  | * write-by-id                                                                       |
  | * write                                                                             |
  | * subscribe-by-id                                                                   |
  | * subscribe                                                                         |
  | * read-event-by-id                                                                  |
  | * subscribe-event-by-id                                                             |
  +-------------------------------------------------------------------------------------+
[1647502596.396299][411686:411686] CHIP:TOO: Run command failure: ../../examples/chip-tool/commands/common/Commands.cpp:84: Error 0x0000002F
```

根据这个列表，`binding`集群支持读取或写入等操作。来自该集群的属性也可以由控制器订阅，这意味着 CHIP 工具将收到通知，例如当属性值更改或特定事件发生时。

#### 4.5.7.1 例子

本节列出了专用于特定用例的 CHIP 工具命令示例。

##### 4.5.7.1.1将 ACL 写入`accesscontrol`集群

访问控制列表 (ACL) 概念允许管理所有数据模型交互（例如读取属性、写入属性、调用命令）。有关 ACL 的更多信息，请参阅 [访问控制指南](https://github.com/project-chip/connectedhomeip/blob/master/docs/guides/access-control-guide.md)。

要将 ACL 写入`accesscontrol`集群，请使用以下命令模式：

```
$ ./chip-tool accesscontrol write acl <acl_data> <node_id> <endpoint_id>
```

在这个命令中：

- *<acl_data>*是格式化为 JSON 数组的 ACL 数据。
- *<node_id>*是要接收 ACL 的节点的 ID。
- *<endpoint_id>*`accesscontrol`是 实施集群的端点的 ID 。

##### 4.5.7.7.2`binding`向集群添加绑定表

绑定描述了包含绑定簇的设备与终端设备之间的关系。必须添加正确的 ACL 以允许终端设备接收来自绑定设备的命令。在绑定过程之后，绑定的设备包含有关连接设备的信息，例如 IPv6 地址和到 Matter 网络中端点的路由。

要将绑定表添加到`binding`集群，请使用以下命令模式：

```
$ ./chip-tool binding write binding  <binding_data> <node_id> <endpoint_id>
```

在这个命令中：

- *<binding_data>*是格式化为 JSON 数组的绑定数据。
- *<node_id>*是要接收绑定的节点的 ID。
- *<endpoint_id>*`binding`是实施集群的端点的 ID 。

##### 4.5.7.7.3运行`TestClusters`测试

完成以下步骤以 [从测试套件运行一个测试](https://github.com/project-chip/connectedhomeip/blob/master/docs/guides/chip_tool_guide.md#running-a-test-suite-against-a-paired-peer-device)：

1. 使用以下命令清除状态的初始化：

   ```
   rm -fr /tmp/chip_*
   ```

2. 在 shell 窗口中，启动 DUT 设备：

   ```
   ./out/debug/standalone/chip-all-clusters-app
   ```

3. 在第二个 shell 窗口中，将 DUT 与 CHIP 工具配对：

   ```
   ./out/debug/standalone/chip-tool pairing onnetwork 333221 20202021
   ```

4. 使用以下命令运行测试：

   ```
   ./out/debug/standalone/chip-tool tests TestCluster --nodeId 333221
   ```

阅读[CHIP 测试套件](https://github.com/project-chip/connectedhomeip/blob/master/src/app/tests/suites/README.md)页面，了解有关测试套件结构的更多信息。

### 4.5.8 多管理员场景

多管理员功能允许您将 Matter 设备加入多个 Matter 结构，并让多个不同的 Matter 管理员对其进行管理。

完成以下部分中提到的步骤。

#### 4.5.8.1 委托面料

[按照使用 CHIP 工具进行物质设备测试](https://github.com/project-chip/connectedhomeip/blob/master/docs/guides/chip_tool_guide.md#using-chip-tool-for-matter-device-testing)部分，将物质设备委托给第一个织物 。

#### 4.5.8.2 打开调试窗口

确保第一个结构的管理员为另一个结构的新管理员打开调试窗口。

使用以下命令模式在配对的 Matter 设备上打开调试窗口：

```
$ ./chip-tool pairing open-commissioning-window <node_id> <option> <window_timeout> <iteration> <discriminator>
```

在这个命令中：

- *<node_id>*是应该打开调试窗口的节点的 ID。
- *<option>*对于增强调试方法等于 1，对于基本调试方法等于 0。
- *<window_timeout>*是调试窗口关闭之前的时间（以秒为单位）。
- *<iteration>*是用于派生 PAKE 验证器的 PBKDF 迭代次数。
- *<discriminator>*是在调试期间确定的设备特定鉴别器。

> **注意：如果***<option>*设置为 0，*则*忽略 <iteration>和*<discriminator>值。*

**命令示例：**

```
$ ./chip-tool pairing open-commissioning-window 1 1 300 1000 2365
```

#### 4.5.8.3 保存配对码

记下命令输出中打印的手动配对代码或 QR 代码有效负载，因为第二个 Matter 管理员需要将 Matter 设备加入其结构。

**输出示例：**

```
[1663675289.149337][56387:56392] CHIP:DMG: Received Command Response Status for Endpoint=0 Cluster=0x0000_003C Command=0x0000_0000 Status=0x0
[1663675289.149356][56387:56392] CHIP:CTL: Successfully opened pairing window on the device
[1663675289.149409][56387:56392] CHIP:CTL: Manual pairing code: [36281602573]
[1663675289.149445][56387:56392] CHIP:CTL: SetupQRCode: [MT:4CT91AFN00YHEE7E300]
```

#### 4.5.8.4 将 Matter 设备调试到新织物

完成以下步骤：

1. 打开 CHIP 工具的另一个实例。

2. 在 CHIP 工具的新实例中，使用以下命令模式将 Matter 设备委托给新结构：

   ```
   $ ./chip-tool pairing code <payload> <node_id> --commissioner-name <commissioner_name>
   ```

   在这个命令中：

   - *<payload>*为第一个委托实例打开调试窗口时生成的二维码有效载荷或手动配对码
   - *<node_id>*是用户定义的被委托节点的 ID。它不需要与第一个结构相同的 ID。
   - *<commissioner_name>*是第二个结构的名称。有效值为“alpha”、“beta”、“gamma”和大于或等于 4 的整数。如果未指定，则默认值为“alpha”。

   **命令示例：**

   ```
   $ ./chip-tool pairing code 36281602573 1 --commissioner-name beta
   ```

#### 4.5.8.5 测试命令的接收

完成上述步骤后，Matter 设备应该能够接收和应答在第二个结构中发送的 Matter 命令。例如，您可以使用以下命令模式`OnOff`在支持集群的设备上切换属性状态`OnOff`：

```
$ ./chip-tool onoff toggle <node_id> <endpoint_id> --commissioner-name <commissioner_name>
```

在这个命令中：

- *<node_id>*是委托节点的自定义ID。
- *<endpoint_id>*是实现了 OnOff 集群的端点的 ID。
- *<commissioner_name>*是第二个结构的名称。有效值为“alpha”、“beta”、“gamma”和大于或等于 4 的整数。如果未指定，则默认值为“alpha”。

**命令示例：**

```
$ ./chip-tool onoff toggle 1 1 --commissioner-name beta
```

---

* 原文地址：[chip_tool_guide.md](https://github.com/project-chip/connectedhomeip/blob/master/docs/guides/chip_tool_guide.md)

