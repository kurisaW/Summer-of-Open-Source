# 如何在Linux平台下测试Matter应用级通信(虚拟设备)

---

## 准备工作

### 1. 递归克隆Matter仓库

执行如下命令：

```bash
git clone --recurse-submodules git@github.com:project-chip/connectedhomeip.git
```

如果克隆过程中发生报错，请执行如下命令来同步子模块：

```bash
git submodule update --init
```

由于我们的环境构建配置均是基于Matter1.0，所以我们需要切换到v1.0分支下

```bash
git checkout v1.0
```

### 2. Matter依赖项安装

Matter 构建依赖于以下软件包及环境库：

```bash
sudo apt-get install git gcc g++ pkg-config libssl-dev libdbus-1-dev \
     libglib2.0-dev libavahi-client-dev ninja-build python3-venv python3-dev \
     python3-pip unzip libgirepository1.0-dev libcairo2-dev libreadline-dev
```

如果通过` build_examples.py` 和 `-with-ui` 变体进行构建，也要安装 SDL2：

```bash
sudo apt-get install libsdl2-dev
```

### 3. Matter环境构建

执行`scripts/activate.sh`脚本。该脚本负责下载 GN、ninja，并使用用于构建和测试的库设置 Python 环境。

```bash
source scripts/activate.sh
```

![image-20230619083303148](https://cdn.jsdelivr.net/gh/kurisaW/picbed/img2023/202306190833624.png)

如果显示环境已过期可执行如下命令进行更新（一般如果没提示环境已过期的提示不建议执行这一步，编译会花一段时间）：

```bash
source scripts/bootstrap.sh
```

### 4. 安装zap

> 注意：zap 包目前不可用`arm64`（比如在 Raspberry PI 上编译时）。

* **Step1：ZAP需要Node.js来运行，请先确保你的计算机上已经安装了Node.js。**可以使用以下命令：

```bash
node -v
```

如果安装的话不出意外会出现版本号。

* **Step2：zap安装**

```bash
cd connectedhomeip/scripts/tools/zap

python3 zap_download.py
```

下面是安装日志：

```bash
root@kurisaw-virtual-machine:/home/kurisaw/Desktop/esp/esp-gitee-tools/esp-matter/connectedhomeip/connectedhomeip/scripts/tools/zap# python3 zap_download.py 
2023-06-19 13:28:22 root INFO    Found required zap version to be: v2023.04.27-nightly
2023-06-19 13:28:22 root INFO    Fetching: https://github.com/project-chip/zap/releases/download/v2023.04.27-nightly/zap-linux.zip
2023-06-19 13:29:20 root INFO    Data downloaded, extracting ...
2023-06-19 13:29:25 root INFO    Done extracting.
export ZAP_INSTALL_PATH=/home/kurisaw/Desktop/esp/esp-gitee-tools/esp-matter/connectedhomeip/connectedhomeip/.zap/zap-v2023.04.27-nightly
```

* **Step3：配置zap环境变量**

我们看上面 zap 安装日志，其中最后导出了zap 的安装路径为`/home/kurisaw/Desktop/esp/esp-gitee-tools/esp-matter/connectedhomeip/connectedhomeip/.zap/zap-v2023.04.27-nightly`，在此目录下有个 zap 脚本，我们这个位置一定要记住！！

设置`ZAP_DEVELOPMENT_PATH`环境变量（这里的路径需要根据上面安装zap后提示的路径进行设置，不能一昧照抄）

```bash
export ZAP_DEVELOPMENT_PATH=/home/kurisaw/Desktop/esp/esp-gitee-tools/esp-matter/connectedhomeip/connectedhomeip/.zap/zap-v2023.04.27-nightly
```

* **Step4：运行zap引导程序**

执行如下代码：

```bash
./run_zaptool.sh
```

效果如下：

![image-20230619134658521](https://cdn.jsdelivr.net/gh/kurisaW/picbed/img2023/202306191346155.png)

* **Step4：为了方便我们后续使用zap，我们设置root终端下自启动：**

```bash
sudo su

vi ~/.bashrc
```

在`.bashrc`文件最末添加如下代码，也就是配置zap环境变量

```bash
export ZAP_DEVELOPMENT_PATH=/home/kurisaw/Desktop/esp/esp-gitee-tools/esp-matter/connectedhomeip/connectedhomeip/.zap/zap-v2023.04.27-nightly
```

保存退出！

## 应用程序构建

在官方文档中提供有两种构建方式：

* 通过脚本构建
* 使用 Gn 和 Ninja 命令构建

### 1. 通过脚本构建

```bash
./build_script.sh EXAMPLE_DIR OUTPUT_DIR [ARGUMENTS]
```

- `build_script.sh` 是脚本的文件名；
- `EXAMPLE_DIR` 是示例项目的目录路径；
- `OUTPUT_DIR` 是构建输出的目录路径；
- `[ARGUMENTS]` 是可选的其他参数，用于设置gn和ninja命令的选项。

#### 1.1 构建示例

```bash
./scripts/examples/gn_build_example.sh examples/placeholder/linux out/debug/simulated/ chip_tests_zap_config=\"app1\"
```

![image-20230619083551820](https://cdn.jsdelivr.net/gh/kurisaW/picbed/img2023/202306190835972.png)

#### 1.2 运行构建

```bash
./out/simulated/chip-app1
```

![image-20230619084309631](https://cdn.jsdelivr.net/gh/kurisaW/picbed/img2023/202306190843752.png)

### 2. 通过 gn 和 ninja 构建应用程序

#### 2.1 构建示例

```bash
source scripts/activate.sh
gn gen --check --root=examples/placeholder/linux out/simulated --args="chip_tests_zap_config=\"app1\""
ninja -C out/simulated
```

#### 2.2 运行构建

```bash
cd 

./out/app1/chip-app1
```

![image-20230619151054483](https://cdn.jsdelivr.net/gh/kurisaW/picbed/img2023/202306191510937.png)

## 测试应用程序

在前面的应用程序构建那一节中我们已经完成了应用程序的构建并且成功运行了构建，同时我们在日志中也可以看到生成了QR码的链接，我们将其复制到浏览器打开即可得到二维码

![image-20230619151353417](https://cdn.jsdelivr.net/gh/kurisaW/picbed/img2023/202306191513515.png)

我们使用chip tool结合生成的QR码进行调试，重新打开一个终端，使用默认的chip tool工具（记住不是之前构建应用程序生成的chip tool），通过QR码可以快捷迅速地将虚拟设备添加到网络中，我们使用chip tool对设备进行调试：

```bash
cd out/debug

./chip-tool onoff on 0x654321 1
./chip-tool onoff off 0x654321 1
./chip-tool onoff read accepted-command-list 0x654321 1
./chip-tool onoff read on-time 0x654321 1
```

![image-20230619153015727](https://cdn.jsdelivr.net/gh/kurisaW/picbed/img2023/202306191530858.png)

具体更多的使用命令可参考：[Chip tool](https://github.com/project-chip/connectedhomeip/blob/master/examples/chip-tool/README.md)

---

## 参考

* [simulated_device_linux](https://github.com/project-chip/connectedhomeip/blob/master/docs/guides/simulated_device_linux.md)
* [zap](https://github.com/project-chip/zap)
