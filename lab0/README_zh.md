# 0. 实验介绍与环境配置

## 0.0 介绍

本实验主要使用两台计算机，分别将其称为`Pi`和`PC`，其中`Pi`负责摄像头的图像采集和机械臂及小车的控制，`PC`负责实现大模型推理。

> 理论上这两项任务都可集成到一台计算机内，基于如下原因暂时分开。
> - 方便理解Dora的跨机器通信
> - 暂时无法在Pi内实现多模态大模型推理
> - Pi的算力不高


## 0.1 实验所需硬件需求

### 0.1.1 Pi

- 硬件: Orange Pi Pro 20T
- 系统：Ubuntu22.04
- 处理器架构：aarch64

理论上也支持使用其他的Pi或aarch64、x86_64的cpu，来完成实验。因为目前本实验暂未用到这个Pi上的昇腾算力。

未来也会尝试只使用`Orange Pi Pro 20T`完成实验，届时使用`Orange Pi Pro 20T`内部的`Ascend`昇腾算力实现多模态大模型推理。

> 参加训练营线下的同学，线下已为您准备

### 0.1.2 PC

推荐使用个人的电脑。目前实验内容可支持在Windows10/11、Ubuntu、WSL2等上进行操作。硬件要求：自带nvidia独立显卡，独立显存大小最好8G往上。硬盘预留20G以上空间。

如若满足不了上述需求，可以尝试在阿里云上租用带GPU的服务器。

> 需要自行准备

> 参加训练营线下的同学，如若个人电脑满足不了需求，可以在交流群内发消息联系助教获取帮助

### 0.1.3 Dora Kit 机器人开发套件

就是机械臂和小车

> 参加训练营线下的同学，线下已为您准备

### 0.1.4 摄像头

总共两个摄像头。需要是linux免驱动的。

> 参加训练营线下的同学，线下已为您准备

## 0.2 PC 安装环境（Windows）

### 0.2.1 Rust 开发环境配置

在Rust官网下载RUSTUP-INIT.EXE（64位）官网链接：[https://www.rust-lang.org/zh-CN/learn/get-started](https://www.rust-lang.org/zh-CN/learn/get-started)

下载下来后，双击安装，会打开一个命令行窗口。

命令行内会显示以下内容：

```
Current installation options:


   default host triple: x86_64-pc-windows-msvc
     default toolchain: stable (default)
               profile: default
  modify PATH variable: yes

1) Proceed with standard installation (default - just press enter)
2) Customize installation
3) Cancel installation
>
```

这里输入1然后回车。在windows下，回车后会自动安装Visual Studio，然后安装MSVC。

等待所有安装完成后。

如果屏幕上出现`Rust is installed now. Great!`等字样， 就说明安装成功了。

接下来添加环境变量。右键桌面`此电脑`图标->`属性`->`高级系统设置`->`高级`->`环境变量`

这里改用户变量或者系统变量都可以。找到Path，双击打开`编辑环境变量`窗口，添加如下：

```
C:\Users\<替换为你的用户名，包括<>号>\.cargo\bin
```

修改完成后，命令行内执行下面的命令：

```bash
cargo --version
rustc --version
```

将看到输出：

```
cargo 1.82.0 (8f40fc59f 2024-08-21)
rustc 1.82.0 (f6e511eec 2024-10-15)
```
### 0.2.2 conda安装和python虚拟环境配置

conda的安装可以参考这篇博客，主要看前2个章节。[https://blog.csdn.net/wh0605/article/details/142979657](https://blog.csdn.net/wh0605/article/details/142979657)

安装成功后，使用如下命令创建一个python3.11的虚拟环境：(安装期间需要二次确认，输入y然后回车即可)

```bash
conda create --name Dora-Camp python=3.11.10
```

然后检查环境是否安装成功

```bash
conda info --envs
```

将看到如下输出：

```
# conda environments:
#
Dora-Camp                C:\Users\Ryu-Yang\.conda\envs\Dora-Camp
base                     D:\Enviroment\anaconda3
```

其中base的路径是根据conda的默认安装路径来的，Dora-Camp虚拟环境的路径则是默认在C盘的用户路径下。

### 0.2.3 PyCharm/VSCode安装

参考网上资料安装即可。

PyCharm安装完成后配置conda的路径为：`D:\Enviroment\anaconda3\Library\bin\conda.bat`，其中`D:\Enviroment\anaconda3`为conda的安装路径

### 0.2.3 CUDA安装

参考这篇博客。[https://blog.csdn.net/weixin_52677672/article/details/135853106](https://blog.csdn.net/weixin_52677672/article/details/135853106)

### 0.2.4 rerun安装

### 0.2.5 dora安装

### 0.2.6 大模型推理环境配置

### 0.2.6 安装Nodes Hub内和实验相关的node



## 0.3 PC 安装环境（Linux）(待完善)

### 0.3.1 Rust 开发环境配置

首先安装 Rust 版本管理器 rustup 和 Rust 包管理器 cargo，这里我们用官方的安装脚本来安装：

```bash
curl https://sh.rustup.rs -sSf | sh
```

如果官方的脚本在运行时出现了网络速度较慢的问题，可选地可以通过修改 rustup 的镜像地址（修改为中国科学技术大学的镜像服务器）来加速：

```bash
export RUSTUP_DIST_SERVER=https://mirrors.ustc.edu.cn/rust-static
export RUSTUP_UPDATE_ROOT=https://mirrors.ustc.edu.cn/rust-static/rustup
curl https://sh.rustup.rs -sSf | sh
```

或者使用tuna源来加速:

```bash
export RUSTUP_DIST_SERVER=https://mirrors.tuna.edu.cn/rustup
export RUSTUP_UPDATE_ROOT=https://mirrors.tuna.edu.cn/rustup/rustup
curl https://sh.rustup.rs -sSf | sh
```

安装完成后，我们可以重新打开一个终端来让之前设置的环境变量生效。我们也可以手动将环境变量设置应用到当前终端，只需要输入以下命令：

```bash
source $HOME/.cargo/env
```

接下来，我们可以确认一下我们正确安装了 Rust 工具链：

```bash
rustc --version
```

可以看到当前安装的工具链的版本。

```bash
rustc 1.82.0 (f6e511eec 2024-10-15)
```

## 0.4 Pi 安装环境

`Pi`内的环境可以直接下载镜像安装。[暂无]()
