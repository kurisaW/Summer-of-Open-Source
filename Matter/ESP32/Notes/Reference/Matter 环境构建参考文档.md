# Matter 环境构建参考文档

---

Matter支持用[GN](https://gn.googlesource.com/gn/)配置构建，一个快速且可扩展的元构建系统，生成输入到[ninja](https://ninja-build.org/)。

## 经过测试的操作系统

该构建系统已经在以下操作系统上进行了测试：

- macOS 10.15
- Debian 11 (64 bit required)
- Ubuntu 22.04 LTS

## 构建系统的特点

Matter构建系统有以下特点：

- 速度非常快，占用空间小
- 跨平台处理： Linux, Darwin, Embedded Arm, 等等
- 多种工具链和跨工具链的依赖性
- 集成了自动测试框架： ninja check
- 自省："gn desc"。
- 自动格式化： `gn格式`。

## 检查Matter的代码

要检查Matter资源库，请运行以下命令：

```
git clone --recurse-submodules git@github.com:project-chip/connectedhomeip.git
```

## 同步子模块

如果你已经签出了Matter的代码，运行下面的命令来同步子模块：

```
git submodule update --init
```

## 先决条件

在构建之前，你必须安装一些操作系统的特定依赖。

### 1.在Linux上安装先决条件

在基于Debian的Linux发行版上，如Ubuntu，这些依赖项可以通过以下命令来满足：

```
sudo apt-get install git gcc g++ pkg-config libssl-dev libdbus-1-dev\
     libglib2.0-dev libavahi-client-dev ninja-build python3-venv python3-dev
     python3-pip unzip libgirepository1.0-dev libcairo2-dev libreadline-dev
```

#### 用户界面的构建

如果通过`build_examples.py`和`with-ui`变体构建，也要安装SDL2：

```
sudo apt-get install libsdl2-dev
```

### 2.在macOS上安装先决条件

在macOS上，从 Mac App Store上安装 Xcode 。

#### 用户界面的构建

如果构建`-with-ui`变体，也要安装 SDL2 ：

```
brew install sdl2
```

### 3.在Raspberry Pi 4上安装先决条件

完成以下步骤：

1. 使用：在 micro SD 卡上`rpi-imager`安装适用于 arm64 架构的 Ubuntu *22.04* 64 位*服务器操作系统。*
1. 启动SD卡。
1. 用默认的用户账户 "ubuntu "和密码 "ubuntu "登录。
1. 继续执行 [在 Linux 上安装先决条件](https://github.com/project-chip/connectedhomeip/blob/master/docs/guides/BUILDING.md#installing-prerequisites-on-linux)。
1. 安装一些Raspberry Pi的特定依赖项：

```
sudo apt-get install pi-Bluetooth avahi-utils
```

6. 安装完 "pi-bluetooth "后，重新启动你的Raspberry Pi。

#### 配置wpa_supplicant以存储永久变化

默认情况下，wpa_supplicant是不允许更新（覆盖）配置的。如果你想让Matter应用程序能够存储配置的变化，您需要进行以下更改：

1. 编辑 `dbus-fi.w1.wpa_supplicant1.service` 文件以使用配置文件来代替，运行以下命令：

```
sudo nano /etc/systemd/system/bus-fi.w1.wpa_supplicant1.service
```

2. 运行以下命令，将wpa_supplicant的启动参数改为提供的值：

```
ExecStart=/sbin/wpa_supplicant -u -s -i wlan0 -c /etc/wpa_supplicant/wpa_supplicant.conf
```

3. 通过运行以下命令添加`wpa-supplicant`配置文件：

```
sudo nano /etc/wpa_supplicant/wpa_supplicant.conf
```

4. 在`wpa-supplicant`文件中添加以下内容：

```
ctrl_interface=DIR=/run/wpa_supplicant
update_config=1
```

5. 重新启动你的Raspberry Pi。

## 安装ZAP工具

`bootstrap.sh`将下载一个兼容的ZAP工具版本并将其设置在`$PATH`。如果你想安装或使用一个不同版本的工具，你可以从ZAP项目的[Release](https://github.com/project-chip/zap/releases) 页面下载。

### 1.Linux ARM

Zap不提供ARM的二进制版本。Rosetta为Darwin解决了这个问题、然而，对于linux arm，你必须使用本地的ZAP，一般通过设置`$ZAP_DEVELOPMENT_PATH`（见下面 `使用哪种ZAP`一节）。

文件`scripts/setup/zap.json`包含CIPD会下载的版本、所以你可以从zap项目中下载一个兼容的版本[Release](https://github.com/project-chip/zap/releases)。要作为源代码签出代码，相应的标签应该存在于zap中[repository tags](https://github.com/project-chip/zap/tags) 列表中。

命令示例：

```sh
RUN set -x \
    && mkdir -p /opt/zap-${ZAP_VERSION} \
    && git clone https://github.com/project-chip/zap.git /opt/zap-${ZAP_VERSION} \
    && cd /opt/zap-${ZAP_VERSION} \
    && git checkout ${ZAP_VERSION} \
    && npm config set user 0 \
    && npm ci
ENV ZAP_DEVELOPMENT_PATH=/opt/zap-${ZAP_VERSION}
```

### 2.使用哪种ZAP

ZAP工具脚本使用以下检测，按重要性排序：

- `$ZAP_DEVELOPMENT_PATH`指向一个ZAP检出。

- 如果你在本地开发ZAP，并希望用你的改动来运行ZAP和你的改动。

- `$ZAP_INSTALL_PATH`指向`zap-linux.zip`或`zap-m

## 为构建做准备

在运行任何其他构建命令之前，`scripts/activate.sh`的环境设置脚本应该在最高层。这个脚本负责下载GN、ninja，并在Python环境中设置用于构建和测试的库来构建和测试。

运行以下命令：

```
source scripts/activate.sh
```

### 1.更新环境

如果脚本说环境已经过期，你可以通过运行下面的命令来更新它：

```
source scripts/bootstrap.sh
```

脚本 `scripts/bootstrap.sh`从头开始重新创建环境，这是很昂贵的，所以避免运行它，除非环境已经过期。

## 为主机操作系统（Linux或macOS）进行构建

运行以下命令，为主机平台构建所有的源代码、库和测试：

```
source scripts/activate.sh

gn gen out/host

ninja -C out/host
```

这些命令生成了一个适合调试的配置。要配置一个构建，请指定`is_debug=false`：

```
gn gen out/host --args='is_debug=false' 。

ninja -C out/host
```

> **注意：**目录名称 "out/host "可以是任何目录，通常是在`out`目录下构建。这个例子使用 `host` 来强调为主机系统构建。不同的构建目录可以用于不同的配置，或者使用一个目录，并在必要时可以根据需要通过`gn args`重新配置。

要运行所有测试，请运行以下命令：

```
ninja -C out/host check
```

要想只运行`src/inet/tests`中的测试，可以运行以下命令：

```
ninja -C out/host src/inet/tests:test_run
```

> **注意：**构建系统会缓存通过的测试，所以你可能会看到以下消息：
> 
>```
> ninja: no work to do
> ```
> 
>这意味着测试在之前的构建中通过了。

## 使用`build_examples.py`

该脚本`./scripts/build/build_examples.py`提供了一个统一的编译构建接口，可以使用`gn`、`cmake`、`ninja`和其他必要的工具来编译各种平台。

使用 `./scripts/build/build_examples.py targets` 来查看支持的目标。

构建命令的例子：

```
# 编译并在主机上运行所有测试：
./scripts/build/build_examples.py --target linux-x64-test build

# 使用 libfuzzer 编译模糊测试标签(模糊测试需要 clang)
./scripts/build/build_examples.py --target linux-x64-test-clang-asan-libfuzzer build

# 编译一个esp32的例子
./scripts/build/build_examples.py --target esp32-m5stack-all-clusters build

# 编译一个 nrf 示例
./scripts/build/build_examples.py --target nrf-nrf5340dk-pump build
```

### 1.`libfuzzer`单元测试

`libfuzzer`单元测试测试只被编译而不被执行（你必须手动执行它们）。为了获得最佳的错误检测，应该使用某种形式的净化器，如`asan`应该被使用。

可执行以下命令：

```
./scripts/build/build_examples.py --target linux-x64-test-lang-asan-libfuzzer build
```

之后，测试应该被定位在`out/linux-x64-tests-lang-asan-libfuzzer/tests/`。

#### `ossfuzz`的配置

`ossfuzz`配置不是独立的模糊测试，而是作为一个与外部模糊测试自动构建的集成点。它们会获取环境变量，如`$CFLAGS`、`$CXXFLAGS`和`$lib_fuzzing_engine`。

你可能需要`libfuzzer`+`asan`的构建来代替本地测试。

## 构建自定义配置

构建是通过设置构建参数来配置的。你可以通过以下方式设置这些参数：

- 将`--args`选项传递给`gn gen`。
- 在输出目录上运行`gn args`。
- 编辑输出目录下的`args.gn`。

要配置一个新的构建或编辑现有构建的参数，请运行以下命令：

```
source scripts/activate.sh

gn args out/custom

ninja -C out/custom
```

两个关键的内置构建参数是 `target_os` 和 `target_cpu`，它们分别控制构建的操作系统和CPU。

要查看所有可用的构建参数的帮助，请运行以下命令：

```
gn gen out/custom
gn args --list out/custom
```

## 构建实例

你可以通过两种方式构建例子。

### 1.将例子作为独立的项目来构建

要把例子作为单独的项目来构建，在Matter的`third_party directory`，运行下面的命令，输入正确的路径到例子的正确路径（这里是 "chip-shell"）：

```
cd examples/shell
gn gen out/debug
ninja -C out/debug
```

### 2.在顶层建立实例

你可以在Matter项目的顶层构建例子。请看下面的`统一构建`一节了解详情。

## 统一构建

要构建一个近似于连续构建集的统一配置，请运行以下命令：

```
source scripts/activate.sh

gn gen out/unified --args='is_debug=true target_os="all"'

ninja -C out/unified all
```

你可以在改变提交配置之前使用这组命令构建，并测试GCC、Clang、MbedTLS和例子的配置。在一个并行的构建中。每个配置都有一个单独的子目录在输出目录中。

这种统一的构建可以用于日常的开发，尽管为每一次编辑而构建所有的东西会更昂贵。构建每一个编辑项目的成本。为了节省时间，你可以将配置来构建：

```
ninja -C out/unified host_gcc
ninja -C out/unified check_host_gcc
```

用配置的名称替换`host_gcc`，它可以在根目录下的 "BUILD.gn "中找到。

你也可以用参数对生成的配置进行微调。比如说

```
gn gen out/unified --args='is_debug=true target_os="all" enable_host_clang_build=false'
```

完整的列表请参见根目录`BUILD.gn`。

在统一的构建中，目标有多个实例，需要通过添加通过添加`(toolchain)`后缀来区分。使用`gn ls out/debug`来列出所有的目标实例。例如：

```
gn desc out/unified '//src/controller(//build/toolchain/host:linux_x64_clang)'
```

> **注意：**有些平台可以作为统一构建的一部分来构建需要下载额外的工具。要将这些工具添加到构建中，必须将其位置
> 必须作为构建参数提供。例如，要添加 `Simplelink cc13x2_26x2` 例子到统一构建中，安装[SysConfig](https://www.ti.com/tool/SYSCONFIG) 并添加以下构建：
> 
> ```
> gn gen out/unified --args="target_os=\"all\" enable_ti_simplelink_builds=true > ti_sysconfig_root=\"/path/to/sysconfig\""
> ```

## 获得帮助

GN集成了帮助，你可以通过`gn help`命令访问。

请确保查看以下推荐的主题：

```
gn帮助执行
gn help 语法
gn help toolchain
```

也可参见 [快速入门指南](https://gn.googlesource.com/gn/+/master/docs/quick_start.md)。

## 自省

GN有各种自省工具来帮助你检查构建配置。下面的例子以`out/host`输出目录为例：

- 显示一个输出目录中的所有目标：

    ```
    gn ls out/host
    ```

- 显示所有将被构建的文件：

    ```
    gn output out/host '*'
    ```

- 显示配置的目标的GN表示：

    ```
    gn desc out/host //src/inet --all
    ```

- 将整个构建的GN表示转为JSON格式：

    ```
    gn desc out/host/ '*' --all --format=json
    ```

- 显示依赖关系树：

    ```
    gn desc out/host //:all deps --tree --all
    ```

- 查找依赖性路径：

    ```
    gn path out/host //src/transport/tests:test //src/system
    ```

- 列出与`libCHIP'连接的有用信息：

    ```
    gn desc out/host //src/lib include_dirs
    gn desc out/host //src/lib defines
    gn desc out/host //src/lib outputs
    
    # 一切都是JSON格式
    gn desc out/host //src/lib --format=json
    ```

## 覆盖范围

代码覆盖率脚本会生成一份报告，其中详细说明了 Matter SDK 源代码的执行量。它还提供了有关 Matter SDK 执行代码段的频率并生成源文件副本的信息，并用执行频率进行了注释。

运行以下命令来启动该脚本：

```
./scripts/build_coverage.sh
```

默认情况下，代码覆盖脚本在单元测试级别执行。单元测试由开发人员创建，因此可以让他们最好地了解单元测试中要包含哪些测试。您可以使用以下参数按范围和执行方式扩展覆盖率测试：

```
  -c, --code 指定收集覆盖数据的范围。
                            core"：从Matter SDK的核心堆栈中收集覆盖数据。--default
                            clusters"：从Matter SDK中的cluster实现中收集覆盖数据。
                            'all'：收集Matter SDK的覆盖数据。
  -t, --tests 指定哪些工具来运行覆盖率检查。
                            'unit'： 运行单元测试来驱动覆盖率检查。--default
                            'yaml'： 运行yaml测试来驱动覆盖率检查。
                            'all'： 运行单元和yaml测试来驱动覆盖率检查。
```

此外，请参阅 Matter SDK 的最新单元测试覆盖率报告（每天收集）： [matter coverage](https://matter-build-automation.ue.r.appspot.com/)。

## 维护事项

如果你对GN构建系统做了任何改变，下一次构建会自动重新生成`ninja`文件。不需要做任何事情。
