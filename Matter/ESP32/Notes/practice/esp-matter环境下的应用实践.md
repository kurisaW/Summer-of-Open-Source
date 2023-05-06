# esp-matter环境下的应用实践

---

## 前提准备

请确保你本地已经配置好 `esp-idf` 及`esp-matter`环境，可参考此博客[【Matter】esp-matter开发环境搭建](https://blog.csdn.net/qq_56914146/article/details/130484975?spm=1001.2014.3001.5501)

## 设置环境变量

### 1.ESP-IDF

根据官网提示，我们需要设置linux平台下的标准工具链，安装以下软件包：

```c
sudo apt-get install git wget flex bison gperf python3 python3-pip python3-setuptools cmake ninja-build ccache libffi-dev libssl-dev dfu-util libusb-1.0-0
```

使用 ESP-IDF 需要 CMake 3.5 或以上版本。较早的 Linux 发行版可能需要升级自身的软件源仓库，或开启 backports 套件库，或安装 “cmake3” 软件包（不是安装 “cmake”）。

```c
cd ./esp/esp-idf
source export.sh
```

![image-20230504160909004](https://raw.githubusercontent.com/kurisaW/picbed/main/img2023/202305041609281.png)

### 2.ESP-Matter

- [Linux](https://github.com/project-chip/connectedhomeip/blob/master/docs/guides/BUILDING.md#installing-prerequisites-on-linux)
- [macOS](https://github.com/project-chip/connectedhomeip/blob/master/docs/guides/BUILDING.md#installing-prerequisites-on-macos)

由于我们使用的是Linux环境，所以此处仅作Linux下的说明，macOS可详见[此处](https://github.com/project-chip/connectedhomeip/blob/master/docs/guides/BUILDING.md#installing-prerequisites-on-macos)

在基于 Debian 的 Linux 发行版（例如 Ubuntu）上，可以使用以下命令满足这些依赖项：

```bash
sudo apt-get install git gcc g++ pkg-config libssl-dev libdbus-1-dev \
     libglib2.0-dev libavahi-client-dev ninja-build python3-venv python3-dev \
     python3-pip unzip libgirepository1.0-dev libcairo2-dev libreadline-dev
```

准备编译matter所需环境。注：如切换了其他分支需要重新运行

```bash
cd ./esp/esp-matter/connectedhomeip/connectedhomeip
source scripts/bootstrap.sh
```

![image-20230506013329415](https://raw.githubusercontent.com/kurisaW/picbed/main/img2023/202305060133538.png)

激活编译matter环境

```bash
cd ./esp/esp-matter/connectedhomeip/connectedhomeip
source scripts/activate.sh
```

![image-20230504161123505](https://raw.githubusercontent.com/kurisaW/picbed/main/img2023/202305041611578.png)

## Matter Example编译下载

### 1.激活esp-matter环境

```c
cd esp-idf
. ./export.sh
```

```c
cd esp-matter 
. ./export.sh
```

### 2.选择esp设备

```c
cd esp-matter/examples/light

idf.py set-target esp32c3
```

初次执行这个命令发生了如下报错：

```c
...

AttributeError: 'HTTPResponse' object has no attribute 'strict'

...
```

在GitHub上参考此[issue](https://github.com/espressif/esp-idf/issues/11340)，并执行以下命令：

```c
pip install -U "urllib3<2"
```

同时重新执行esp-matter安装脚本：

由于需要重新运行安装脚本命令，此处直接执行的话会报错，参考此[issue](https://github.com/kurisaW/Summer-of-Open-Source/issues/7)

```c
rm -rf esp-matter/connectedhomeip/connectedhomeip/.environment

cd esp-matter

./install.sh
```

```c
pip install -U "urllib3<2"
```

然后回到示例工程下继续执行esp设备选择

```c
cd esp-matter/examples/light

idf.py set-target esp32c3
```

此时发生了新的错误：

![image-20230506022134054](https://raw.githubusercontent.com/kurisaW/picbed/main/img2023/202305060221146.png)

由于示例工程下的build以前遗留的构建文件，而系统在执行程序时并不会覆盖或主动删除旧的构建文件，因此需要用户手动删除，因此正确的操作就是：

```c
sudo rm -r esp-matter/examples/light/build
idf.py set-target esp32c3
```

最后成功解决问题：

![b372338ad9384db034000d7839549b5](https://user-images.githubusercontent.com/98592772/236539480-35af78e1-382f-4092-a25b-fb2a09004d0a.png)

### 3.编译工程

```c
idf.py build
```

![image-20230506025001282](https://raw.githubusercontent.com/kurisaW/picbed/main/img2023/202305060250998.png)

### 4.SDK烧写

第一次烧写 SDK 时，需要擦除整个 flash 再执行烧录命令

```c
idf.py erase_flash
```
![image-20230506025047817](https://raw.githubusercontent.com/kurisaW/picbed/main/img2023/202305060252819.png)

烧录程序并打开串口监视

```c
idf.py flash monitor
```

可以看到烧录进度：

![image-20230506025133178](https://raw.githubusercontent.com/kurisaW/picbed/main/img2023/202305060251334.png)

包括串口监视器的提示信息，同时执行以下命令可退出串口监视：

```c
CTRL + ]
```

![image-20230506025401001](https://raw.githubusercontent.com/kurisaW/picbed/main/img2023/202305060254172.png)

那么esp-matter项目环境的编译下载就先讲到这里，后面再进行详细的使用教程的讲解。

---

参考链接：

[Matter Over Wifi 例程体验（CHIP Over Wifi）](https://blog.csdn.net/hydfxy2018/article/details/122041168?spm=1001.2101.3001.6650.11&utm_medium=distribute.pc_relevant.none-task-blog-2~default~CTRLIST~Rate-11-122041168-blog-127516686.pc_relevant_default&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2~default~CTRLIST~Rate-11-122041168-blog-127516686.pc_relevant_default&utm_relevant_index=12)

[ESP-Matter 环境测试](https://blog.csdn.net/weixin_40209493/article/details/125814311?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522168324979316800211536064%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fall.%2522%257D&request_id=168324979316800211536064&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~first_rank_ecpm_v1~rank_v31_ecpm-4-125814311-null-null.142^v86^control_2,239^v2^insert_chatgpt&utm_term=matter%E6%8A%A5%E9%94%99&spm=1018.2226.3001.4187)

[matter搭建环境](https://blog.csdn.net/puweiqi/article/details/129474079?spm=1001.2101.3001.6650.2&utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7EYuanLiJiHua%7EPosition-2-129474079-blog-125973073.235%5Ev32%5Epc_relevant_default_base3&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7EYuanLiJiHua%7EPosition-2-129474079-blog-125973073.235%5Ev32%5Epc_relevant_default_base3&utm_relevant_index=3)

[https://docs.espressif.com/projects/esp-matter/en/main/esp32/developing.html](https://docs.espressif.com/projects/esp-matter/en/main/esp32/developing.html)