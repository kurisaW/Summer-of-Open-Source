

# esp-matter开发环境搭建

---

## 前提准备

### 1.Ubuntu22.04（磁盘容量不小于80G）

### 2.科学上网环境

由于后面的 esp-matter 测试的时候需要使用到科学上网环境，所以我们需要提前确保 linux 环境能够使用科学上网。

参考链接：[【经验分享】Linux 环境下v2ray的使用](https://kurisaw.github.io/p/经验分享linux-环境下v2ray的使用/)

## esp-idf 开发环境搭建

### 1.ESP-IDF 依赖环境安装

> 参考https://docs.espressif.com/projects/esp-idf/en/v4.4.3/esp32/get-started/linux-setup.html

```c
sudo apt-get install git wget flex bison gperf python3 python3-pip python3-setuptools cmake ninja-build ccache libffi-dev libssl-dev dfu-util libusb-1.0-0
```

由于在克隆官方esp-idf仓库的时候一般会发生如下两个错误：

* Problem1：执行 git submodule 速度慢
* Problem2：执行install.sh 速度慢

所以我们这里特别着重讲解，注意，这里解决问题的顺序与esp-idf环境搭建是一起进行的，读者可以顺着流程走。

### 2.Problem1 solution

首先使用递归克隆命令克隆整个仓库到文件夹下

```c
mkdir /home/kurisaw/Desktop/esp

git clone --recursive https://github.com/espressif/esp-idf.git

git submodule update --init --recursive
```

由于 esp-idf 仓库下有很多递归的下游仓库，一般使用 GitHub 下载的话也会导致递归下载失败，所以乐鑫官方提供了两种解决方案，包括镜像仓库使用、submodule 更新、开发工具安装等，可加速环境的搭建。解决方案如下：

* jihu-mirror 使用（推荐）
* submodule-update 使用（不推荐）

#### 2.1  jihu-mirror 使用（推荐）

* Step 1：

```c
git clone https://gitee.com/EspressifSystems/esp-gitee-tools.git
cd esp-gitee-tools
```

* Step 2：

```c
// 使用如下命令将仓库的 URL 进行替换：

git config --global url.https://jihulab.com/esp-mirror/espressif/esp-idf.insteadOf https://github.com/espressif/esp-idf
```

当我们使用命令 `git clone https://github.com/espressif/esp-idf` 时，默认的 URL `https://github.com/espressif/esp-idf` 将被自动替换成 `https://jihulab.com/esp-mirror/espressif/esp-idf`。

* Step 3：

```c
// 启用镜像URL

./jihu-mirror.sh set
```

使用命令 `./jihu-mirror.sh unset` 恢复，不使用镜像的 URL。

* Step 4：当使用镜像 URL 之后，再递归克隆 esp-idf 仓库

```
git clone --recursive https://github.com/espressif/esp-idf.git
```

当然如果不想使用镜像的URL可以使用如下命令进行恢复：

```c
./jihu-mirror.sh unset
```

#### 2.2  submodule-update 使用（不推荐）


- Step 1：

  ```
  git clone https://gitee.com/EspressifSystems/esp-gitee-tools.git
  ```

- Step 2：

  ```c
  // 仅克隆 esp-idf，不包含子模块
  
  git clone https://gitee.com/EspressifSystems/esp-idf.git
  ```

* Step 3：

可以有两种方式来更新 submodules。

- 方式一

  进入 esp-gitee-tools 目录，export submodule-update.sh 所在路径，方便后期使用，如：

  ```
  cd esp-gitee-tools
  export EGT_PATH=$(pwd)
  ```

  进入 esp-idf 目录执行 submodule-update.sh 脚本：

  ```
  cd esp-idf
  $EGT_PATH/submodule-update.sh
  ```

- 方式二

  `submodule-update.sh` 脚本支持将待更新 submodules 的工程路径作为参数传入，例如：`submodule-update.sh PATH_OF_PROJ`。

  假如 Step 2 中 clone 的 esp-idf 位于 ~/git/esp32-sdk/esp-idf 目录，可使用以下方式来更新：

  ```
  cd esp-gitee-tools
  ./submodule-update.sh ~/git/esp32-sdk/esp-idf
  ```

  如果要更新其他工程，可以同样方式。

> 值得吐槽的是， submodule-update 这种方法还需要保持上游代码分支的提交历史一致，如果官方未及时更新则会导致该脚本暂时失效，不推荐使用，避坑！！

### 3.Problem2 solution

下面说第二个问题：执行./install.sh速度慢的问题

在 Espressif Systems 的 esp-idf 开发框架中，某些组件的构建过程需要从 GitHub 的 release 页面下载预编译的二进制文件。然而，在中国大陆访问 GitHub 的速度往往较慢并且不稳定，为了改善这个问题，Espressif Systems 将这些预编译的二进制文件托管在国内的服务器上，并提供了一个名为 `IDF_GITHUB_ASSETS` 的环境变量来指定这个地址。在设置了 `IDF_GITHUB_ASSETS` 变量之后，构建过程将会从这个指定的地址下载预编译的二进制文件

```c
export IDF_GITHUB_ASSETS="dl.espressif.com/github_assets"
```

然后再执行安装命令

```c
./install.sh
```

在这还报了一个错误

![image-20230504124717772](https://raw.githubusercontent.com/kurisaW/picbed/main/img2023/202305041247286.png)

我们根据提示安装`python3.10-venv`，并再次执行安装命令：

```c
apt install python3.10-venv

./install.sh
```

![image-20230504124913620](https://raw.githubusercontent.com/kurisaW/picbed/main/img2023/202305041249721.png)

至此，esp-idf 的安装工具就告一段落了。

## esp-matter开发环境搭建

> 参考：[【乐鑫 Matter SDK GitHub】](https://github.com/espressif/esp-matter)

**注意：如果上面的 esp-idf 开发环境的搭建使用的是  jihu-mirror 方式，那么你需要取消esp镜像，按理说这部分错误不应该发生，但实际上确实存在这部分问题，请执行命令：`./jihu-mirror.sh unset`取消esp镜像！！ **

```c
git clone --recursive https://github.com/espressif/esp-matter.git
```

若过程有报错，请执行下面命令在Git 仓库中获取到所有子模块，并将所有子模块及其下层子模块更新至最新版本。

```shell
git submodule update --init --recursive
```

执行安装命令：

```c
./install.sh
```

本以为到这就结束了，但不出意外的话意外发生了，在安装过程中发生了报错...

```
  Building wheel for pycryptodome (setup.py): started
  error: subprocess-exited-with-error
  
  × python setup.py bdist_wheel did not run successfully.
  │ exit code: 1
  ╰─> See above for output.
  
  note: This error originates from a subprocess, and is likely not a problem with pip.
  Building wheel for pycryptodome (setup.py): finished with status 'error'
  ERROR: Failed building wheel for pycryptodome
  Running setup.py clean for pycryptodome
  Building wheel for gevent (pyproject.toml): started
  
  ......
```

我们查看`install.sh`文件

```
#!/usr/bin/env bash

set -e

basedir=$(dirname "$0")
ESP_MATTER_PATH=$(cd "${basedir}"; pwd)
MATTER_PATH=${ESP_MATTER_PATH}/connectedhomeip/connectedhomeip
export ESP_MATTER_PATH

echo ""
echo "Running Matter Setup"
echo ""
source ${MATTER_PATH}/scripts/bootstrap.sh

echo ""
echo "Installing zap-cli"
echo ""
# Run the zap_download.py and extract the path of installed binary
# eg output before cut: "export ZAP_INSTALL_PATH=zap/zap-v2023.03.06-nightly"
# output after cut: zap/zap-v2023.03.06-nightly
# TODO: Remove the zap-version after https://github.com/project-chip/connectedhomeip/pull/25727 merged
zap_path=`python3 ${ESP_MATTER_PATH}/connectedhomeip/connectedhomeip/scripts/tools/zap/zap_download.py \
    --sdk-root ${ESP_MATTER_PATH}/connectedhomeip/connectedhomeip --zap RELEASE --zap-version v2023.03.27-nightly \
    --extract-root .zap 2>/dev/null | cut -d= -f2`
# Check whether the download is successful.
if [ -z $zap_path ]; then
    echo "Failed to install zap-cli"
    deactivate
    exit 1
fi

# Move files to one directory up, so that binaries will be in $ESP_MATTER_PATH/.zap/ directory and export.sh can leverage the fixed path
if [ -d "${ESP_MATTER_PATH}/.zap" ]; then
    rm -r ${ESP_MATTER_PATH}/.zap
fi
mkdir ${ESP_MATTER_PATH}/.zap
mv $zap_path/* ${ESP_MATTER_PATH}/.zap/
rm -r $zap_path
chmod +x ${ESP_MATTER_PATH}/.zap/zap-cli

echo ""
echo "Building host tools"
echo ""
gn --root="${MATTER_PATH}" gen ${MATTER_PATH}/out/host
ninja -C ${MATTER_PATH}/out/host
echo ""
echo "Host tools built at: ${MATTER_PATH}/out/host"
echo ""

echo ""
echo "Exit Matter environment"
echo ""
deactivate

echo ""
echo "Installing python dependencies for mfg_tool"
echo ""
python3 -m pip install -r ${ESP_MATTER_PATH}/tools/mfg_tool/requirements.txt

echo ""
echo "Installing python dependencies for Matter"
echo ""
python3 -m pip install -r ${ESP_MATTER_PATH}/requirements.txt

echo "All done! You can now run:"
echo ""
echo "  . ${basedir}/export.sh"
echo ""
```

发现问题出在第10到13行，我尝试安装系统必要的依赖项来解决这个问题，成功解决！命令如下：

```
sudo apt install build-essential python3-dev

sudo apt-get install pkg-config

sudo apt-get install libglib2.0-dev libglib2.0-dev-bin libgio2.0-cil-dev
```

[![image-20230504145216015](https://raw.githubusercontent.com/kurisaW/picbed/main/img2023/202305041452447.png)](https://raw.githubusercontent.com/kurisaW/picbed/main/img2023/202305041452447.png)

接着在安装`zap-cli`的时候再次发生报错，需要安装以下依赖库，并再次运行安装脚本命令，等待编译

```
sudo apt-get install libssl-dev

sudo apt-get install pip

./install.sh
```

[![image-20230504150238105](https://raw.githubusercontent.com/kurisaW/picbed/main/img2023/202305041502605.png)](https://raw.githubusercontent.com/kurisaW/picbed/main/img2023/202305041502605.png)

最后看到`All done!`即代表环境安装成功！

[![image-20230504153243388](https://raw.githubusercontent.com/kurisaW/picbed/main/img2023/202305041535612.png)](https://raw.githubusercontent.com/kurisaW/picbed/main/img2023/202305041535612.png)

至此，esp-matter开发环境搭建成功！
