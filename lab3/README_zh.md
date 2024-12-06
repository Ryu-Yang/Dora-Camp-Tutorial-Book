# 实验3：视觉语言模型驱动的具身机器人

## 1. 视觉语言模型-VLM介绍

## 2. 实验3-1 横向移动抓取

### 2.1 环境配置

本实验使用`Pi`和`PC`完成实验，操作需要分别在`Pi`和`PC`上完成。

和之前的实验相同，检查完环境配置。

**首先在PC上完成如下操作**

检查`rerun`和dora的node-hub中的`dora-rerun`是否正常安装，在终端中输入：

```bash
cargo install --list
```

若正常安装，将显示版本(安装的版本可能不一)：

```
dora-rerun v0.3.6:
    dora-rerun.exe
rerun-cli v0.20.2:
    rerun.exe
```

若未正常安装，在终端中分别输入下列命令安装（安装过程可能会有些耗时）：

```bash
cargo install rerun-cli --locked
cargo install dora-rerun --locked
```

PS：在windows下安装dora-rerun，如果用上面的命令可能后面会有问题，推荐把dora仓库clone下来，进入到`dora/node-hub/dora-rerun`文件夹,通过`cargo install --path .`命令安装。

然后是通过pip安装大模型推理需要的包（在conda的虚拟环境下安装）(lab0已安装了就跳过这步)

```bash
pip3 install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu124 --default-timeout=3600
pip install transformers
pip install qwen-vl-utils
pip install optimum
pip install auto-gptq
```

linux下可以加上这个包

```bash
pip install flash-attn
```

然后在PC内新建或找一个文件夹（用于存放`模型`和`微调`的文件），切换到该文件夹：

```bash
cd <the path of your folder>
```

然后分别创建`models`和`adapter`文件夹后，切换到models文件夹内，下载qwen2-vl-2B模型

```bash
cd models
```

模型下载地址，[https://www.modelscope.cn/models/Qwen/Qwen2-VL-2B-Instruct/files]。有三种下载方法，这里介绍通过git下载

请确保 lfs 已经被正确安装，然后clone仓库

```bash
git lfs install
```
```bash
git clone https://www.modelscope.cn/Qwen/Qwen2-VL-2B-Instruct.git
```

如果`model-00001-of-00002.safetensors`和`model-00002-of-00002.safetensors`没有下载下来，可以手动在页面上点击下载，下载下来后在拷贝到models/Qwen2-VL-2B-Instruct/目录下。

再切换到adapter文件夹，下载微调后的得到的文件

```bash
cd ../adapter
```

下载地址，[https://www.modelscope.cn/RyuYang/Qwen2-VL-2B-Fine-Tuning-Grab-Bottle]
    

完成上述环境配置后，切换到本实验文件夹

```bash
cd Dora-Camp-Tutorial/lab3
```

修改`Dora-Camp-Tutorial/lab3/qwenvl2_robot_inference.yml`文件内的第65-66行，把路径替换为前面保存模型和adapter的绝对路径：

```yaml
# 替换为自己电脑上的路径
MODEL_NAME_OR_PATH: E:\Dora-Camp-Tutorial-Saves\models\Qwen2-VL-2B-Instruct
ADAPTER_PATH: E:\Dora-Camp-Tutorial-Saves\adapter\Qwen2-VL-2B-Fine-Tuning-Grab-Bottle
```

**然后在Pi上完成如下操作**

检查`opencv`和dora的node-hub中的`opencv-video-capture`是否正常安装，在终端中输入：

```bash
pip show opencv-python
pip show opencv-video-capture
```

若正常安装，将显示其名称和版本等信息


若未正常安装，在终端中输入：

```bash
pip install opencv-python
pip install opencv-video-capture
```

完成上述环境配置后，切换到项目文件夹，并激活环境

```bash
cd Dora-Camp-Tutorial/
conda activate Dora-Camp
```

### 2.2 启动并查看结果

同实验1和2，确认好机械臂正确连接，摄像头正常。

确认正常后在`PC`内启动dora coordinator:

```bash
dora coordinator
```

再新建一个命令行窗口，启动dora daemon:(这里的IP地址替换为前面记下的IP地址)

```bash
dora daemon -c 192.168.3.183
```

然后在`Pi`内启动dora daemon:

```bash
dora daemon -c 192.168.3.183 --machine-id pi
```

> 注意：这里一定是要在`Dora-Camp-Tutorial/`目录下启动

然后回到`PC`上，

```bash
dora start ./qwenvl2_robot_inference.yml
```

> 注意：这里是在`Dora-Camp-Tutorial/lab3/`目录下启动，且在Dora-Camp环境内

正常启动后会机械臂会回到默认位置。然后`PC`上会弹出一个rerun窗口，在上面可以看到两个摄像头的画面（可能需要加载一会），然后便可以等待大模型输出操作命令了，实现自己横向抓取瓶子。

> 注意：摄像头角度和顺序可能需要手动调整，以及篮子的位置

### 2.3 yaml文件解析
