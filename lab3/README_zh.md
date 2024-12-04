# 3. 实验3：视觉语言模型驱动的具身机器人（线下实验）
## 3.1 视觉语言模型-VLM介绍
## 3.2 实验3-1 横向移动抓取
### 3.2.1 环境配置

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

然后是通过pip安装大模型推理需要的包（在conda的虚拟环境下安装）

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

然后分别创建`models`和`fine-tuning`文件夹后，切换到models文件夹内，下载qwen2-vl-2B模型


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

再切换到fine-tuning文件夹，下载微调后的得到的文件

```bash
cd ../fine-tuning
```

下载地址，[https://www.modelscope.cn/RyuYang/Qwen2-VL-2B-Fine-Tuning-Grab-Bottle]
    

完成上述环境配置后，切换到本实验文件夹

```bash
cd Dora-Camp-Tutorial/lab3
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

完成上述环境配置后，切换到项目文件夹

```bash
cd Dora-Camp-Tutorial/
```

### 3.2.2 配置yaml文件


### 3.2.4 启动并查看结果

同实验1，确认好机械臂正确连接。

然后确认摄像头正常连接，在终端输入：

```bash
ls /dev/video*
```

正常情况下，应该看到输出：
```
/dev/video0 /dev/video1 /dev/video2 /dev/video3
```

如若未正常显示，检查摄像头是否正确连接，并进入PCCAM状态

> 注意：使用OrangePi上面的USB Hub连接摄像头时，需要把Hub上的所有连接都拔下来，再从摄像头开始依次插入

确认上述正常后，启动dora的coordinator和daemon，在终端中输入：

```bash
dora up
```

成功启动后将看到输出：

```
started dora coordinator
started dora daemon
```

然后再在终端中输入：

```bash
dora start ctrl_opencv.yml
```

启动后可以看到图像界面，如下图：

![追踪](images/1.png)

随后，机械臂自动追踪黄色方块（根据opencv.py里写好的颜色），并停留在在其上。
