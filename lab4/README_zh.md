# 实验4：使用LLaMA-Factory进行模型微调并评估测试

## 1. LLaMA-Factory介绍

TODO

## 2. 实验4-1 横向移动抓取任务数据集采集

### 2.1 环境配置

本实验使用`Pi`和`PC`完成实验，操作需要分别在`Pi`和`PC`上完成。

**首先在PC上完成如下操作**

和实验3相同，检查完环境配置。    

完成上述环境配置后，切换到本实验文件夹

```bash
cd Dora-Camp-Tutorial/lab4
```

修改`Dora-Camp-Tutorial/lab4/qwenvl2_robot_recorder.yml`文件内的第60行，把路径换为`Dora-Camp-Tutorial/recorder`的绝对路径：

```yaml
LLAMA_FACTORY_ROOT_PATH: E:/Repositories/Dora-Camp-Tutorial/recorder # 替换为自己电脑上的路径
```

切换到Dora-Camp环境

```bash
conda activate Dora-Camp
```

**然后在Pi上完成如下操作**

同实验3完成环境配置后，切换到项目文件夹

```bash
cd Dora-Camp-Tutorial/
```

### 2.2 启动并查看结果

确认好机械臂正确连接，摄像头正常。

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
dora start ./qwenvl2_robot_recorder.yml
```

> 注意：这里是在`Dora-Camp-Tutorial/lab4/`目录下启动

正常启动后会机械臂会回到默认位置。然后`PC`上会弹出一个rerun窗口，在上面可以看到两个摄像头的画面（可能需要加载一会）。

然后便可以通过按键控制机械臂了，操作同实验1一样，不同点是此时按下的所有操作和按下时摄像头的图像都会保存到`Dora-Camp-Tutorial/recorder/data/`中，用作训练的数据集。

> 注意：摄像头角度和顺序可能需要手动调整，以及篮子的位置

### 2.3 yaml文件解析

TODO

## 3. 实验4-2 使用采集的数据集进行微调

### 3.1 本地微调（Windows）

首先安装LLaMA-Factory。

```bash
git clone --depth 1 https://github.com/hiyouga/LLaMA-Factory.git
cd LLaMA-Factory
pip install -e ".[torch,metrics]"
```

如果要在 Windows 平台上开启 FlashAttention-2，需要安装预编译的 flash-attn 库，支持 CUDA 12.1 到 12.2，请根据需求到 [flash-attention](https://github.com/bdashore3/flash-attention/releases) 下载对应版本安装。

安装完成后，将`Dora-Camp-Tutorial/lab4/`内的`qwen2vl_lora_sft.yaml`拷贝到`LLaMA-Factory/`内

`qwen2vl_lora_sft.yaml`内的第2行`model_name_or_path: `需要替换为模型的绝对路径。第11行`dataset`改为要训练的数据集。根据情况看是否使用``flash_attn: fa2`

再将实验4-1采集到的在数据集`Dora-Camp-Tutorial/recorder/data`，替换掉原本的`LLaMA-Factory/`中的`data`

开始微调训练：

```bash
llamafactory-cli train ./qwen2vl_lora_sft.yaml
```

等待训练完成后，将在`LLaMA-Factory/saves`目录下保存adapter、检查点、训练误差等信息。

