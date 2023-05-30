# 使用chip tool在ESP32-C3上进行matter开发

---

## 前提准备

* 请确保你已经能够完成在esp-matter下的应用程序的烧录及串口监视，可参考此博客[【Matter】esp-matter环境下的应用实践（程序烧录及串口监视）](https://blog.csdn.net/qq_56914146/article/details/130519043?spm=1001.2014.3001.5501)

* ubuntu最好使用20以上的版本，因为matter最低需要python3.8的环境
* PC机需要支持蓝牙4.0及以上版本，如果没有的话需要购买一个USB蓝牙适配器，而且需要支持Linux，可以参考购买这款[蓝牙适配器](https://m.tb.cn/h.UvoJzj4?tk=KpYpdNFRueB)

## 编译 chip-tool

### 1.激活esp-matter环境

```c
cd esp-idf
. ./export.sh
```

```c
cd esp-matter 
. ./export.sh
```

### 2.编译matter所需环境

* step1：首先安装编译所需的依赖包：

```base
sudo apt-get install git gcc g++ pkg-config libssl-dev libdbus-1-dev libglib2.0-dev libavahi-client-dev ninja-build python3-venv python3-dev python3-pip unzip libgirepository1.0-dev libcairo2-dev libreadline-dev
```

* step2：切换到 /matter/connectedhomeip/connectedhomeip 目录下，编译matter环境（如果没显示环境过期，这一步可跳过）

```bash
# 运行引导程序，该脚本负责下载 GN、ninja，并使用用于构建和测试的库设置 Python 环境。如果此脚本显示环境已过期，则可以通过运行以下命令进行更新

source scripts/bootstrap.sh
```

> 对于 MacOS，`gdbgui`python 包不会使用`bootstrap.sh` 脚本安装，因为它仅限于 x64 Linux 平台。它受到限制，因为在 MacOS 上为`gevent`（依赖于`gdbgui`）构建轮子失败。
>
> 对于ARM-based Mac，如果Python3版本大于或等于3.11，则不需要进一步的安装步骤。
>
> 如果 Python3 版本低于 3.11 或者您使用的是 x86（基于英特尔）Mac，那么请在每次引导后运行以下命令以将 gdbgui wheels 安装为二进制文件
>
> ```
> python3 -m pip install -c scripts/setup/constraints.txt --no-cache --prefer-binary gdbgui==0.13.2.0
> deactivate
> ```

* step3：激活编译matter环境

```c
source scripts/activate.sh
```

* step4：启用 Ccache 以加快 IDF 构建速度

```c
$ export IDF_CCACHE_ENABLE=1
```

### 3.构建CHIP TOOL

在 `~/esp/esp-matter/connectedhomeip/connectedhomeip`目录下，执行命令

```shell
./gn_build.sh
```

![image-20230504173815084](https://raw.githubusercontent.com/kurisaW/picbed/main/img2023/202305041738527.png)

执行完之后，会在根目录下生成 `out/debug/standalone/chip-tool`一个二进制文件。

![image-20230504174038993](https://raw.githubusercontent.com/kurisaW/picbed/main/img2023/202305041740040.png)

如果上述命令：`./gn_build.sh`执行失败，也可以执行如下命令：

```c
scripts/examples/gn_build_example.sh examples/chip-tool SOME-PATH/
```

![image-20230504175634584](https://raw.githubusercontent.com/kurisaW/picbed/main/img2023/202305041756762.png)

执行完毕后，在以下路径 `connetedhomeip/connectedhomeip/SOME-PATH`也可以发现生成了 chip-tool 工具

![image-20230504175700807](https://raw.githubusercontent.com/kurisaW/picbed/main/img2023/202305041757853.png)

## chip-tool client 调试设备说明

为了向设备发送命令，必须使用客户端对其进行调试。芯片工具目前**一次只支持调试和记忆一个设备**。配置状态存储在/tmp/chip\_tool\_config.ini中；

另外删除/tmp中的此文件和其他.ini文件有时可以解决由于过时配置导致的问题。

```c
# 获取受支持集群的列表

Usage:
  ./chip-tool cluster_name command_name [param1 param2 ...]

  +-------------------------------------------------------------------------------------+
  | Clusters:                                                                           |
  +-------------------------------------------------------------------------------------+
  | * barriercontrol                                                                    |
  | * basic                                                                             |
  | * colorcontrol                                                                      |
  | * doorlock                                                                          |
  | * groups                                                                            |
  | * iaszone                                                                           |
  | * identify                                                                          |
  | * levelcontrol                                                                      |
  | * onoff                                                                             |
  | * pairing                                                                           |
  | * payload                                                                           |
  | * scenes                                                                            |
  | * temperaturemeasurement                                                            |
  +-------------------------------------------------------------------------------------+
```

![image-20230504180042312](https://raw.githubusercontent.com/kurisaW/picbed/main/img2023/202305041800372.png)

* **有关具体其他命令和使用方法详见 : [https://github.com/project-chip/connectedhomeip/tree/v1.0-branch/examples/chip-tool](https://github.com/project-chip/connectedhomeip/tree/v1.0-branch/examples/chip-tool)**

要向设备发起客户端调试请求，需要运行构建的可执行文件并选择配对模式，具体操作如下：

### 1.基于 BLE 调试

运行构建的可执行文件并将远程设备的鉴别器和配对代码以及要使用的网络凭据传递给它。下面的命令使用硬编码到 ESP32 all-clusters-app 调试版本中的默认值来将其调试到 Wi-Fi 网络：

```c
chip-tool pairing ble-wifi ${NODE_ID_TO_ASSIGN} ${SSID} ${PASSWORD} 20202021 3840
```

- `${NODE_ID_TO_ASSIGN}`（必须是十进制数或`0x`- 前缀的十六进制数）是要分配给正在调试的节点的节点 ID。
- `${SSID} 是 Wi-Fi SSID` 可以是字符串，也可以是`hex:XXXXXXXX` SSID 的字节被编码为两位十六进制数字的形式。
- `${PASSWORD}` 是 Wi-Fi 密码，同样是字符串或十六进制数据

```c
# 例如
chip-tool pairing ble-wifi 0x7283 jetbot jetbotwyq 202021 3840
```

### 2.通过IP与设备配对

下面的命令将发现设备并尝试使用提供的设置代码与它发现的第一个设备配对。

```
chip-tool pairing onnetwork ${NODE_ID_TO_ASSIGN} 20202021
```

下面的命令将发现具有长鉴别器 3840 的设备，并尝试使用提供的设置代码与它发现的第一个设备配对。

```
chip-tool pairing onnetwork-long ${NODE_ID_TO_ASSIGN} 20202021 3840
```

下面的命令将根据给定的二维码（哪些设备在启动时记录）发现设备，并尝试与它发现的第一个配对。

```
chip-tool pairing code ${NODE_ID_TO_ASSIGN} MT:#######
```

在所有这些情况下，将为设备分配节点 ID `${NODE_ID_TO_ASSIGN}` （必须是十进制数或以 0x 为前缀的十六进制数）。

### 3.Trust store

Trust store 将使用默认的 Test Attestation PAA 自动创建。要使用不同的 PAA 集，请在运行构建的可执行文件时使用可选参数 --paa-trust-store-path 传递路径。受信任的 PAA 位于 credentials/development/paa-root-certs/。

下面的命令将选择一组受信任的 PAA，以在证明验证期间使用。它还会发现具有长鉴别器 3840 的设备，并尝试使用提供的设置代码与它发现的第一个设备配对。

```
chip-tool pairing onnetwork-long ${NODE_ID_TO_ASSIGN} 20202021 3840 --paa-trust-store-path path/to/PAAs
```

### 4.忘记当前委托的设备

```c
chip-tool pairing unpair
```

## 使用chip-tool点灯

### 1.matter环境激活

由于每次配置的 esp-idf 和 esp-matter 环境激活仅在当前终端有效，这里我们编写一个脚本文件，每次打开一个终端执行此脚本即可完成matter环境的激活：

* step1：新建一个名为 matter.sh 的脚本文件

```c
vi matter.sh
```

* step2：复制以下内容到 matter.sh

```c
#/bin/bash
# matter.sh

EPS_MATTER_PATH="/home/kurisaw/Desktop/esp/esp-gitee-tools/esp-matter"

if [ $1 -eq 1 ]; then
    export IDF_PATH="/home/kurisaw/Desktop/esp/esp-gitee-tools/esp-idf" 
    source /home/kurisaw/Desktop/esp/esp-gitee-tools/esp-idf/export.sh
    source $EPS_MATTER_PATH/export.sh
    export IDF_CCACHE_ENABLE=1
    
    echo "enter matter dir"
    cd $EPS_MATTER_PATH
fi
```

* step3：执行脚本以激活 matter 环境

```c
source matter.sh 1
```

### 2.固件烧录

* 打开一个新的**终端1**，进入示例目录设置并编译烧写到评估板运行

```c
cd ./esp/esp-matter/examples/light
```

* 设置要构建的 Matter 目标
* 目前所有示例应用程序都支持目标芯片：esp32、esp32s3、esp32c3，一般仅需要使用 命令1 即可。**需要注意的是：如果你使用的设备为ESP32H2，而ESP32H2 仅在 lighting-app 中支持，执行 命令2 将其设置为目标**

```
# 命令1，通用命令，ESP32H2请执行命令2
idf.py set-target (target chip)

# 命令2，ESP32H2专用命令
idf.py --preview set-target esp32h2
```

这里我使用的是 ESP32C3，所以执行以下命令即可

```c
idf.py set-target esp32c3
```

* 配置选项（可遵循默认配置即可，非特定配置可跳过这一步）

要**构建特定配置**（示例`m5stack`）：

```
rm sdkconfig
idf.py -D 'SDKCONFIG_DEFAULTS=sdkconfig_m5stack.defaults' build
```

注意：如果使用特定的设备配置，强烈建议从默认设置之一开始并在此基础上进行自定义。某些配置具有在设备特定配置中自定义的不同约束（例如：主应用程序堆栈大小）。

要自定义配置，请运行 menuconfig，在菜单中可完成自定义配置：

```
idf.py menuconfig
```
* 构建应用程序

```c
idf.py build
```

* 擦除Flash

构建应用程序后，要通过 USB 连接您的设备来闪擦除它。然后运行以下命令擦除整个闪存，将演示应用程序闪存到设备上，然后监控其输出。

请注意，有时您可能必须在设备尝试连接时按住设备上的启动按钮，然后才能刷机。对于 ESP32-DevKitC 设备，这在[functional description diagram](https://docs.espressif.com/projects/esp-idf/en/latest/esp32/hw-reference/esp32/get-started-devkitc.html#functional-description)中有所提及。

```
idf.py -p (PORT) erase_flash
idf.py -p (PORT) flash monitor
```

请替换`(PORT)`为您系统的正确 USB 设备名称（如`/dev/ttyUSB0`在 Linux 或`/dev/tty.usbserial-101`Mac 上）。

查看USB设备，esp32c3设备名为 `ttyUSB0`，因此执行以下命令 ：

```c
idf.py -p /dev/ttyUSB0 erase_flash
idf.py -p /dev/ttyUSB0 flash monitor
```

* 注意此时的设备串口**终端1**暂时先不关闭，后面可使用`CTRL+]`关闭设备串口调试

![image-20230530173001926](https://raw.githubusercontent.com/kurisaW/picbed/main/img2023/202305301730041.png)

注意：某些用户可能必须在设备出现在 /dev/tty 之前安装[VCP 驱动程序。](https://www.silabs.com/products/development-tools/software/usb-to-uart-bridge-vcp-drivers)

提示：在监视器运行时，您可以通过按 Ctrl+t Ctrl+h 来查看各种监视器命令的菜单。

### 3.项目调试

以下四种方式可以用于调试在ESP32上运行应用程序：

- [Python Based Device Controller](https://github.com/project-chip/connectedhomeip/tree/master/src/controller/python)
- [Standalone chip-tool](https://github.com/project-chip/connectedhomeip/tree/master/examples/chip-tool)
- [iOS chip-tool App](https://github.com/project-chip/connectedhomeip/tree/master/src/darwin/CHIPTool)
- [Android chip-tool App](https://github.com/project-chip/connectedhomeip/tree/master/examples/android/CHIPTool)

**注：这里使用 `Standalone chip-tool`进行项目调试**

打开一个新的**终端2**，我们需要运行构建的可执行文件并将远程设备的鉴别器和配对代码以及要使用的网络凭据传递给它，执行命令：

```bash
cd esp-matter/connectedhomeip/connectedhomeip

# 激活matter环境
source scripts/activate.sh
```

![image-20230530172301207](https://raw.githubusercontent.com/kurisaW/picbed/main/img2023/202305301723608.png)

* 调试WIFI设备（ESP32、ESP32C3、ESP32S3）

如果你使用的是Thread设备(ESPH2)或以太网设备(ESP32-Ethernet-Kit)，设备调试具体可以查看[此链接](https://github.com/project-chip/connectedhomeip/blob/master/docs/guides/esp32/build_app_and_commission.md)

执行下面命令将 matter 设备接入现有现有IP网络，这里我们**基于BLE调试**

**需要注意的是，你需要确保你的 Linux 蓝牙可用，如果是使用虚拟机的话需要考虑购买一个蓝牙适配器，可参考这个[购买链接](https://m.tb.cn/h.UvoJzj4?tk=KpYpdNFRueB)**

接下来请按照我的步骤一步步执行：

* step1：安装 blueman 软件

```bash
sudo apt install blueman #安装blueman软件
sudo /etc/init.d/bluetooth restart  # 重启blueman服务
```

* step2：确保你的蓝牙状态处于激活状态

```bash
# 查看蓝牙状态

sudo systemctl status bluetooth
```

![7e8b531f8b4be994ed272cf2e69703c](https://user-images.githubusercontent.com/98592772/236623922-496f12f1-837d-44eb-8cca-a76b5f132e2c.png)

如果未运行，请执行：

```bash
sudo systemctl enable bluetooth
sudo systemctl start bluetooth
```

* step3：确认蓝牙适配器已经被识别并启用

```bash
hciconfig -a
```
![LRHC%H77T8AU FZ_V$F@(Q6](https://user-images.githubusercontent.com/98592772/236629771-b49be4da-0979-45b7-9484-f9bb2f895f29.png)

根据提示信息我们可以得知我的蓝牙适配器名为"hci0"，并且状态为 "DOWN"，因此我们需要启用该蓝牙适配器。

* step4：启用蓝牙适配器

```bash
sudo hciconfig hci0 up
```

* step5：为了让 matter 设备连接蓝牙网络，我们需要让蓝牙适配器在任何时候可见，点击右上角的蓝牙图标，点击`Adapters...--->Visibility Setting--->Always visible`，这一步很关键，**每次基于 BLE 调试都需要检查这一步！！**

![image-20230530174457873](https://raw.githubusercontent.com/kurisaW/picbed/main/img2023/202305301744038.png)

* step6：BLE调试，回到**终端2**，执行如下命令


```bash
cd esp-matter/connectedhomeip/connectedhomeip
    
out/debug/chip-tool pairing ble-wifi 0x7283 jetbot jetbotwyq 20202021 3840
```

注意：本机ip和matter设备ip必须在同一局域网下

> - `0x7283`（必须是十进制数或`0x`- 前缀的十六进制数）是要分配给正在调试的节点的节点 ID，随意填写即可。
> - `jetbot 是 Wi-Fi SSID` 可以是字符串，也可以是`hex:XXXXXXXX` SSID 的字节被编码为两位十六进制数字的形式。
> - `jetbotwyq` 是 Wi-Fi 密码，同样是字符串或十六进制数据

![image-20230530175437844](https://raw.githubusercontent.com/kurisaW/picbed/main/img2023/202305301754997.png)

在**终端1**我们可以看到相关的ip信息：

![image-20230530175633102](https://raw.githubusercontent.com/kurisaW/picbed/main/img2023/202305301756204.png)

* step7：利用 chip tool 控制LED开关

```bash
# open led
out/debug/chip-tool onoff on 0x7896 0x1
# close led
out/debug/chip-tool onoff off 0x7896 0x1
```

> 这里的节点ID：0x7896需要和前面保持一致

![cd20c5fede056bf65b089da69ab9f3a](https://raw.githubusercontent.com/kurisaW/picbed/main/img2023/202305301802687.jpg)

![f40b925710de89f66bf9ecf7ef27d7e](https://raw.githubusercontent.com/kurisaW/picbed/main/img2023/202305301802294.jpg)

## CHIP TOOL基于BLE调试完整过程

![video](https://raw.githubusercontent.com/kurisaW/picbed/main/img2023/202305301920428.gif)

---

## 参考

* [CHIP Reference](https://github.com/project-chip/connectedhomeip/blob/master/docs/guides/chip_tool_guide.md)
* [Setup ESP-IDF and CHIP Environment](https://github.com/project-chip/connectedhomeip/blob/master/docs/guides/esp32/setup_idf_chip.md)
* [building and commissioning](https://github.com/project-chip/connectedhomeip/blob/master/docs/guides/esp32/build_app_and_commission.md)
