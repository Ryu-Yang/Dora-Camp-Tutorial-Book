# 实验1：使用Dora操作机器人

## 1. 基于Rust的机器人开发框架-Dora
### 1.1 Dora 总体架构
### 1.2 Dora 数据流
### 1.3 Dora CLI
### 1.4 Dora 数据格式（Apache Arrow）
### 1.5 Dora 安装使用

## 2. 机器人基础
### 2.1 硬件组成及电机驱动控制
### 2.2 坐标系与位姿变换
### 2.3 轨迹规划
### 2.4 GEN72机械臂

## 3. 实验1-1 在OrangePi内实现Dora操作机械臂

### 3.1 环境配置

本实验只使用`Pi`完成实验，所有操作均在`Pi`上完成。

假定您已准备好OrangePi并为其烧录了ubuntu22.04系统。

> 训练营已为各位线下学员准备好环境，可跳过这步

![PrangePi](./images/1.png)

启动OrangePi，并为其连接好屏幕键盘。连接显示器线时，需要注意，只能使用上面的HDMI接口：

![PrangePi-HDMI](./images/2.png)

更多信息可以参考这：[Orange Pi AI Pro 20T 用户手册](documents/OrangePi_AI_Pro_20T_昇腾_用户手册_v0.5.1.pdf)

然后使用密码登录。如果您使用的是训练营已配好的环境，密码是`gosim1234`

登录后，鼠标左键单击桌面左上角`Application`，打开菜单栏后再左键单击`Terminal Emulator`，启动终端。

检查dora是否安装，在终端中输入：

```bash
dara -V
```

如果正确安装，将看到输出：(版本号可能会不一样)

```
dora-cli 0.3.7
```

如若未正常安装，请参考[实验介绍与环境配置](../lab0/README_zh.md)进行安装。

接下来的实验操作所需要的代码和文件已经准备好了，可以直接clone下来（已clone下来可以跳过这步）。在终端中输入：

```bash
git clone https://github.com/Ryu-Yang/Dora-Camp-Tutorial.git
```

克隆下来后，切换到本实验文件夹。在终端中输入：

```bash
cd Dora-Camp-Tutorial/lab1/Single-Pi
```

### 3.2 启动并查看结果

首先确认好机械臂已正常启动，信号指示灯处于绿色闪烁状态，如下图所示：

![arm_connect](images/3.png)

然后确认好已配置好网络。具体的配置流程可以参考[睿尔曼机器人硬件准备](https://develop.realman-robotics.com/robot/quickUseManual/)

> 训练营线下环境已配置好，只需确保正确连接到右边的网口就行

![arm_connect_net](images/4.png)

接下来，启动dora的coordinator和daemon，在终端中输入：

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
dora start ctrl_arm.yml
```

成功启动后机械臂将会到初始位置。接下来便可以通过按键`t`、`g`、`f`、`h`、`y`、`r`，控制机械臂的`前`、`后`、`左`、`右`、`上`、`下`了。通过`q`、`e`，控制机械爪的`抓取`和`放下`。`x`保存轨迹点，`c`清除所有轨迹点，`b`执行保存的轨迹点。

### 3.3 yaml文件解析

在`Dora-Camp-Tutorial/lab1/Single-Pi`文件夹中已写好一个`ctrl_arm.yml`文件，内容如下：

```yaml
# Run on Pi

nodes:
  - id: key-listener
    path: ../../src/key_listener.py
    inputs:
      tick: dora/timer/millis/10
    outputs:
      - char

  - id: key-text
    path: ../../src/key_text.py
    inputs:
      keyboard: key-listener/char
    outputs:
      - text

  - id: trans-cmd
    path: ../../src/trans_cmd.py
    inputs:
      key-keyboard: key-text/text
    outputs:
      - movec
      - claw
      - save
      - clear
      - begin
      - stop
      - goto
  
  - id: arm
    path: ../../src/gen72.py
    inputs:
      movec: trans-cmd/movec
      claw: trans-cmd/claw
      save: trans-cmd/save
      clear: trans-cmd/clear
      begin: trans-cmd/begin
      stop: trans-cmd/stop
      goto: trans-cmd/goto
    env:
      ROBOT_IP: 192.168.1.18  # gen72 robotic arm default IP address
      SAVED_POSE_PATH: ../../recorder/pose_library.json # The relative path of pose_library.json that relative to this yaml file.

```

### 3.4 键盘监听节点

在`Dora-Camp-Tutorial/src`文件夹中已写好`key_listener.py`文件作为键盘监听节点，内容如下：

```python
from pynput import keyboard
from pynput.keyboard import Events
import pyarrow as pa
from dora import Node
import time

node = Node()

with keyboard.Events() as events:
    while True:
        event = events.get(1.0)
        if event is not None and isinstance(event, Events.Press):
            if hasattr(event.key, "char"):
                if event.key.char == "p":
                    break
                if event.key.char == "z":
                    while True:
                        node.send_output("char", pa.array(["b"]))
                        time.sleep(5)
                if event.key.char is not None:
                    node.send_output("char", pa.array([event.key.char]))

```

### 3.5 按键识别节点

在`Dora-Camp-Tutorial/src`文件夹中已写好`key_text.py`文件作为按键识别节点，内容如下：

```python
from dora import Node
import pyarrow as pa

node = Node()

for event in node:
    if event["type"] == "INPUT":
        if event["id"] == "keyboard":
            char = event["value"][0].as_py()
            print(f"""Keyboard recieved: {char}""")
            if   char == "w":
                node.send_output("text", pa.array(["forward"]))
            elif char == "s":
                node.send_output("text", pa.array(["backward"]))
            elif char == "d":
                node.send_output("text", pa.array(["right"]))
            elif char == "a":
                node.send_output("text", pa.array(["left"]))
            elif char == "q":
                node.send_output("text", pa.array(["claw close"]))
            elif char == "e":
                node.send_output("text", pa.array(["claw open"]))
            elif char == "t":
                node.send_output("text", pa.array(["arm forward"]))
            elif char == "g":
                node.send_output("text", pa.array(["arm backward"]))
            elif char == "f":
                node.send_output("text", pa.array(["arm left"]))
            elif char == "h":
                node.send_output("text", pa.array(["arm right"]))
            elif char == "r":
                node.send_output("text", pa.array(["arm up"]))
            elif char == "y":
                node.send_output("text", pa.array(["arm down"]))
            elif char == "x":
                node.send_output("text", pa.array(["save"]))
            elif char == "c":
                node.send_output("text", pa.array(["clear"]))
            elif char == "b":
                node.send_output("text", pa.array(["begin"]))
            elif char == "n":
                node.send_output("text", pa.array(["stop"]))
            elif char == "m":
                node.send_output("text", pa.array(["goto"]))

```
### 3.6 命令识别节点
在`Dora-Camp-Tutorial/src`文件夹中已写好`trans_cmd.py`文件作为命令识别节点，内容如下：

```python
from dora import Node
import pyarrow as pa
from enum import Enum


PID_X = 0.0004
PID_Y = -0.0003

class Action(Enum):
    FORWARD = ("arm forward",   "movec", [0.02, 0, 0, 0, 0, 0, 0.1])
    BACK    = ("arm backward",  "movec", [-0.02, 0, 0, 0, 0, 0, 0.1])
    LEFT    = ("arm left",      "movec", [0, -0.02, 0, 0, 0, 0, 0.1])
    RIGHT   = ("arm right",     "movec", [0, 0.02, 0, 0, 0, 0, 0.1])
    UP      = ("arm up",        "movec", [0, 0, -0.02, 0, 0, 0, 0.1])
    DOWN    = ("arm down",      "movec", [0, 0, 0.02, 0, 0, 0, 0.1])
    CLOSE   = ("claw close",    "claw", [0])
    OPEN    = ("claw open",     "claw", [100])
    SAVE    = ("save",          "save", [0])
    CLEAR   = ("clear",         "clear", [0])
    BEGIN   = ("begin",         "begin", [0])
    STOP    = ("stop",          "stop", [0])
    GOTO    = ("goto",          "goto", [0])


node = Node()

for event in node:
    if event["type"] == "INPUT":
        event_id = event["id"]
        if event_id == "key-keyboard" or event_id == "key-qwenvl":
            text = event["value"][0].as_py()
            text = text.replace(".", "")
            text = text.replace(".", "")

            for action in Action:
                if action.value[0] in text:
                    node.send_output(action.value[1], pa.array(action.value[2]))
                    print(f"""recieved:{action.value[0]}""")

        if event["id"] == "error":
            [error_x, error_y] =  event["value"].tolist()
            move_error = [
                PID_Y*error_y,
                PID_X*error_x,
                0,
                0,
                0,
                0,
                0
            ]
            print(f"""move_error:{move_error}""")
            node.send_output("movec", pa.array(move_error))

```

### 3.7 GEN72机械臂驱动
我们使用`Dora-Camp-Tutorial/src`文件夹中的`robotic_arm_package`作为驱动。这是GEN72机械臂官方提供的python驱动包。这里我们主要使用其中的`robotic_arm`模块

### 3.8 机械臂操作节点

在`Dora-Camp-Tutorial/src`文件夹中已写好`gen72.py`文件作为机械臂操作节点，内容如下：
```python
# import
import numpy as np
from dora import Node
import json
import os
import time
from robotic_arm_package.robotic_arm import *
import sys


# define
SPEED = 30
SAVED_POSE_PATH = os.getenv("SAVED_POSE_PATH", "./recorder/pose_library.json")
ROBOT_IP = os.getenv("ROBOT_IP", "192.168.1.18")
MIN_Z = float(os.getenv("MIN_Z", "0.0"))

assert ROBOT_IP is not None, "ROBOT_IP environment variable must be set"


# function
def load_json_file(file_path):
    """Load JSON file and return the dictionary."""
    with open(file_path, "r") as file:
        data = json.load(file)
        return data

def save_json_file(file_path, data):
    """Save the dictionary back to the JSON file."""
    with open(file_path, "w") as file:
        json.dump(data, file, indent=4)


# main
robot = Arm(72, ROBOT_IP)  # 创建实例
robot.Set_Tool_Voltage(3)
robot.Set_Modbus_Mode(1, 115200, 2, 2)

joint_d = [0, 40, 0, -100, 0, 45, 0]
robot.Movej_Cmd(joint_d, SPEED)
robot.Write_Single_Register(1, 40000, 100, 1, 1)
time.sleep(2)

data = robot.Get_Current_Arm_State()  # 获取机械臂运动数据
[x, y, z, rx, ry, rz] = list(data[2])  # 位置position
joint_angle= data[1]  # 当前的关节angle

logger_.info(
    f"Cartesian Pose: x={x}, y={y}, z={z}, rx={rx}, ry={ry}, rz={rz}"
)
logger_.info(f"joint_angle: {joint_angle}")

node = Node()
pose_library = load_json_file(SAVED_POSE_PATH)
logger_.info(f"pose_library: {pose_library}")

for event in node:
    if event["type"] == "INPUT":
        if event["id"] == "movec":
            [dx, dy, dz, drx, dry, drz, t] = event["value"].tolist()

            cartesian_pose = {
                "x": x + dx,
                "y": y + dy,
                "z": z + dz,
                "rx": rx + drx,
                "ry": ry + dry,
                "rz": rz + drz,
            }

            x=x + dx
            y=y + dy
            z=z + dz
            rx=rx + drx
            ry=ry + dry
            rz=rz + drz

            cart_pose = [
                cartesian_pose[key] for key in ["x", "y", "z", "rx", "ry", "rz"]
            ]
            robot.Movel_Cmd(cart_pose, SPEED, block=False)

        elif event["id"] == "claw":
            [claw] = event["value"].tolist()
            robot.Write_Single_Register(1, 40000, claw, 1, 1)

        elif event["id"] == "save":
            id = pose_library["num"]
            robot.Move_Stop_Cmd()
            data = robot.Get_Current_Arm_State()
            [x, y, z, rx, ry, rz] = list(data[2])
            joint_angle = data[1]
            tag, claw = robot.Get_Read_Holding_Registers(1, 40000, 1)
            pose_library["pose"][id] = list(joint_angle)
            pose_library["claw"][id] = claw
            pose_library["num"] = pose_library["num"] + 1

        elif event["id"] == "clear":
            pose_library["num"] = 0
            pose_library["pose"].clear()
            pose_library["claw"].clear()

        elif event["id"] == "begin":
            robot.Move_Stop_Cmd()
            for i in range(pose_library["num"]):
                retrieved_pose = pose_library["pose"].get(i)
                retrieved_claw = pose_library["claw"].get(i)
                if retrieved_pose is not None:
                    joint_angle = retrieved_pose
                    robot.Movej_Cmd(joint_angle, SPEED, block=True)
                    robot.Write_Single_Register(1, 40000, retrieved_claw, 1, 1)
                    time.sleep(1)
                    data = robot.Get_Current_Arm_State()
                    [x, y, z, rx, ry, rz] = list(data[2])  # 末端工具位置
                    joint_angle = data[1]  # 当前的关节位置

        elif event["id"] == "stop":
            robot.Move_Stop_Cmd()
            data = robot.Get_Current_Arm_State()
            [x, y, z, rx, ry, rz] = list(data[2])  # 末端工具位置
            joint_angle = data[1]  # 当前的关节位置

        elif event["id"] == "goto":
            name = event["value"][0].as_py()
            robot.Move_Stop_Cmd()
            retrieved_pose = pose_library["pose"].get(name)
            retrieved_claw = pose_library["claw"].get(name)
            if retrieved_pose is not None:
                joint_angle = retrieved_pose
                robot.Movej_Cmd(joint_angle, SPEED, block=False)
                data = robot.Get_Current_Arm_State()
                [x, y, z, rx, ry, rz] = list(data[2])  # 末端工具位置
                joint_angle = data[1]  # 当前的关节位置

    save_json_file(SAVED_POSE_PATH, pose_library)

```

## 4. 实验1-2 多机器完成驱动

### 4.1 环境配置

本实验使用`Pi`和`PC`完成实验，操作需要分别在`Pi`和`PC`上完成。

和上一个实验一样，在两台计算机上都检查一下dora是否正常安装。

```bash
dara -V
```

在`Pi`内，切换到Dora-Camp-Tutorial目录：

```bash
cd Dora-Camp-Tutorial/
```

在`PC`内，切换到Dora-Camp-Tutorial/lab1/Multiple-PC目录：

```bash
cd Dora-Camp-Tutorial/lab1/Multiple-PC
```

确保`Pi`和`PC`在同一个局域网内

> 线下训练营可以通过实验场地的wifi来实现，wifi名称`HUAWEI-1F24RW`,wifi密码`gosim!@#$%`

然后在`PC`内点击wifi，点击连接的wifi的属性，在打开的页面的属性一项内查看IPv4地址并记录下来。

或者使用命令查看（windows）：

```bash
ipconfig
```

使用命令查看（linux）：

```bash
ifconfig
```

### 4.2 启动并查看结果

首先在`PC`内启动dora coordinator:

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
dora start ./ctrl_arm.yml
```

> 注意：这里是在`Dora-Camp-Tutorial/lab1/Multiple-PC/`目录下启动

成功启动后机械臂将会到初始位置。接下来便可以通过按键`t`、`g`、`f`、`h`、`y`、`r`，控制机械臂的`前`、`后`、`左`、`右`、`上`、`下`了。通过`q`、`e`，控制机械爪的`抓取`和`放下`。`x`保存轨迹点，`c`清除所有轨迹点，`b`执行保存的轨迹点。

### 4.3 yaml文件解析

在`Dora-Camp-Tutorial/lab1/Multiple-PC`文件夹中已写好一个`ctrl_arm.yml`文件，内容如下：

```yaml
# Run on PC

nodes:
  - id: key-listener
    path: ../../src/key_listener.py
    inputs:
      tick: dora/timer/millis/10
    outputs:
      - char

  - id: key-text
    path: ../../src/key_text.py
    inputs:
      keyboard: key-listener/char
    outputs:
      - text

  - id: trans-cmd
    path: ../../src/trans_cmd.py
    inputs:
      key-keyboard: key-text/text
    outputs:
      - movec
      - claw
      - save
      - clear
      - begin
      - stop
      - goto

  - id: arm
    path: ./src/gen72.py  # The relative path of gen72.py that relative to the path where you started the dora daemon on pi
    _unstable_deploy:
      machine: pi
    inputs:
      movec: trans-cmd/movec
      claw: trans-cmd/claw
      save: trans-cmd/save
      clear: trans-cmd/clear
      begin: trans-cmd/begin
      stop: trans-cmd/stop
      goto: trans-cmd/goto
    env:
      ROBOT_IP: 192.168.1.18  # gen72 robotic arm default IP address
      SAVED_POSE_PATH: ./recorder/pose_library.json # The relative path of pose_library.json that relative to the path where you started the dora daemon on pi

```

相比于实验1-1，实验1-2的大部分节点都是运行在`PC`上的，如 `key-listener`、`key-text`、`trans-cmd`，只有`arm`节点是运行在`Pi`上的。在实验1-1中，dora的coordinater和daemon都运行在`Pi`中，而实验1-2中是`PC`运行：coordinater+daemon，`Pi`只运行一个daemon。

这里详细解析arm节点。由于该节点内加上了下面这个键和键值：

```yaml
_unstable_deploy:
    machine: pi
```

故表示该节点将运行在，名为pi的机器上。在`dora start`该yaml文件后，coordinator便会去尝试找到该机器上启动的daemon。pi机器上的daemon再根据path来找到pi上的代码或包，然后启动该节点。

该节点的path路径和代码内打开文件的路径都将根据在`Pi`上启动daemon的路径。