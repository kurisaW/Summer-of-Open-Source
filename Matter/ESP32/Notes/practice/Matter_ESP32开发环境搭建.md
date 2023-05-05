

# esp-matter开发环境搭建

---

## 前提准备

### 1.Ubuntu22.04（内存不小于80G）

### 2.科学上网环境

由于后面的 esp-matter 测试的时候需要使用到科学上网环境，所以我们需要提前确保 linux 环境能够使用科学上网。

参考链接：[【经验分享】Linux 环境下v2ray的使用](https://kurisaw.github.io/p/经验分享linux-环境下v2ray的使用/)

## esp-idf 开发环境搭建

由于在克隆官方esp-idf仓库的时候一般会发生如下两个错误：

* Problem1：执行 git submodule 速度慢
* Problem2：执行install.sh 速度慢

所以我们这里特别着重讲解，注意，这里解决问题的顺序与esp-idf环境搭建是一起进行的，读者可以顺着流程走。

### 1.Problem1 solution

首先使用递归克隆命令克隆整个仓库到文件夹下

```c
mkdir /home/kurisaw/Desktop/esp

git clone --recursive https://github.com/espressif/esp-idf.git

git submodule update --init --recursive
```

在执行这段命令的时候会存在有些文件无法克隆到本地，或者出现下载仅有几kb的现象，这里我们参考[国内这位大神写的脚本](https://gitee.com/EspressifSystems/esp-gitee-tools/blob/master/docs/README-submodule-update.md)

将上面下载的文件删除，执行如下命令：

- Step 1：

  ```
  git clone https://gitee.com/EspressifSystems/esp-gitee-tools.git
  ```

- Step 2：

  ```
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

**本人使用上述步骤已完成测试，确实可以很顺畅的完成操作，且中间也没有发生报错。**

### 2.Problem2 solution

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

```c
git clone --recursive https://github.com/espressif/esp-matter.git
```

若过程有报错，请执行下面命令在Git 仓库中获取到所有子模块，并将所有子模块及其下层子模块更新至最新版本。

```shell
git submodule update --init --recursive
```

安装matter所需的依赖项：

```c
sudo apt install git gcc g++ pkg-config libssl-dev libdbus-1-dev
sudo apt-get install libglib2.0-dev libavahi-client-dev ninja-build python3-venv python3-dev
sudo apt-get python3-pip unzip libgirepository1.0-dev libcairo2-dev libreadline-dev
```

执行安装命令：

```c
./install.sh
```

最后看到`All done!`即代表环境安装成功！

![image-20230505105924550](https://raw.githubusercontent.com/kurisaW/picbed/main/img2023/202305051059941.png)

至此，esp-matter开发环境搭建成功！

