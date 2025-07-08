<style>
.sdk-table {
  width: 100%; /* 表格总宽度 */
  border-collapse: collapse; /* 合并边框 */
  display: table;
}

.sdk-table th {
  background-color: #f2f2f2;
 }

.sdk-table th, .sdk-table td {
  border: 1px solid #ccc; /* 边框样式 */
  padding: 8px; /* 内边距 */
}

.sdk-table th:first-child,
.sdk-table td:first-child {
  width: 30%; /* 第一列宽度 */
}

.sdk-table th:nth-child(2),
.sdk-table td:nth-child(2) {
  width: 70%; /* 第二列宽度 */
}
</style>

# **MC_SDK 使用指南**

当前文档适用于SDK版本V0.2.0以上，运控版本v0.1.8以上（狗本体0.1.4以上）



# 1. 目录结构
    

```Shell
├── demo                       # demo
│   ├── cpp                    # cpp demo
│   └── python                 # python demo
├── image_1.jpg                # 机器人关节坐标系示
├── image.png                   # ip配置示意图
├── include                     # 头文件
│   └── sdk
│       ├── highlevel.h         # 高级控制头文件
│       └── lowlevel.h          # 低级控制头文件
├── lib                         # 动态链接库
├── py_whl                      # pythoh包
└── README.md                   # 说明文档
```
# 2. 环境依赖
- Ubuntu 22.04
- CMake 3.8+
- GCC 11+
- Eigen3
- boost
- python3
# 3. 网络配置
机器人默认ip: `192.168.234.1`

机器人ip如有变更，需参照配置`export SDK_CLIENT_IP="机器人新ip"`

## 3.1. 外部控制器与本机通信
    
  配置本地 IP 和端口

  在代码中修改 CLIENT_IP 和 CLIENT_PORT：

  示例：
    
```C++
constexpr int CLIENT_PORT = 43988;      // 本地端口  
std::string CLIENT_IP = "192.168.234.15"; // 本地 IP 地址
```
    
      
    
在机器人端配置本地 IP 和端口
    
```Shell
ssh firefly@192.168.234.1 # 密码 firefly
```
    
编辑配置文件： 📌 路径： `/opt/export/config/sdk_config.yaml`

配置示例
    
```YAML
target_ip: "192.168.234.15"
target_port: 43988
```
    
请确保机器人 IP 和端口号匹配，否则无法建立通信。

当设备程序进行更新，或运控程序进行更新后，配置文件会被重置，需要重新配置！
    

## 3.2. 本机内控制器与本机运控通信
    
配置本地 IP 和端口

在代码中修改 CLIENT_IP 和 CLIENT_PORT：

示例：
    
```C++
constexpr int CLIENT_PORT = 43988;      // 本地端口  
std::string CLIENT_IP = "192.168.234.1"; // 机器狗 IP 地址
```
    
在机器人端配置本地 IP 和端口
    
```Shell
ssh firefly@192.168.234.1 # 密码 firefly
```
    
编辑配置文件： 📌 路径： `/opt/export/config/sdk_config.yaml`

配置示例：
    
```YAML
target_ip: "192.168.234.1"
target_port: 43988
```
    
请确保机器人 IP 和端口号匹配，否则无法建立通信。

当设备程序进行更新，或运控程序进行更新后，配置文件会被重置，需要重新配置！
    
---

# 4. 安装运行步骤

```Shell

###编译demo
cd mc_sdk/demo/cpp
mkdir build
cd build
cmake ..
make -j6
###

###运行c++demo
cd mc_sdk/demo/cpp/build
./highlevel_demo
or
./lowlevel_demo

###运行python demo

cd mc_sdk/demo/python/examples
python highlevel_demo.py
or
python lowlevel_demo.py
###
基于cmake使用mc_sdk
、、、cmake
find_library(MC_SDK_LIB mc_sdk_${CMAKE_HOST_SYSTEM_PROCESSOR}
    PATHS 
        ${CMAKE_SOURCE_DIR}/../../lib
    REQUIRED
)

add_executable(highlevel_demo highlevel_demo.cpp)
target_link_libraries(highlevel_demo ${MC_SDK_LIB})
target_include_directories(highlevel_demo PRIVATE ${CMAKE_SOURCE_DIR}/../../include)

add_executable(lowlevel_demo lowlevel_demo.cpp)
target_link_libraries(lowlevel_demo ${MC_SDK_LIB})
target_include_directories(lowlevel_demo PRIVATE ${CMAKE_SOURCE_DIR}/../../include)

基于python使用mc_sdk
、、、
import sys
import os
sys.path.append(os.path.abspath(os.path.join(os.path.dirname(__file__), "../../../"))) # 包含 py_whl文件夹路径

from py_whl import mc_sdk_py
、、、
```

---

# 5. 运动控制接口
    

## 5.1. App 控制接口函数介绍
    

**class AppFunc**

- 该类包含控制机器狗应用功能的方法，用户可以使用这些方法来进行上层控制和获取数据。
    

<table class="sdk-table">
  <tr>
    <th>函数名</th>
    <th>initRobot(const std::string & local_ip, const int local_port, const string& dog_ip = "192.168.234.1")</th>
  </tr>
  <tr>
    <td>功能概述</td>
    <td>与机器狗建立通信,机器狗端ip默认“192.168.234.1”, 如更改机器人ip，需从该接口传入新ip</td>
  </tr>
  <tr>
    <td>参数</td>
    <td>用户主机 ip 和端口</td>
  </tr>
  <tr>
    <td>返回值</td>
    <td>无</td>
  </tr>
  <tr>
    <td>备注</td>
    <td>通信建立失败时，终端输出失败日志</td>
  </tr>
</table>

<table class="sdk-table">
  <tr>
    <th>函数名</th>
    <th>lieDown()</th>
  </tr>
  <tr>
    <td>功能概述</td>
    <td>控制机器狗站低</td>
  </tr>
  <tr>
    <td>参数</td>
    <td>无</td>
  </tr>
  <tr>
    <td>返回值</td>
    <td>进入状态正常返回0，返回0x3012代表电机数据丢失，0x3010代表电机失能， 0x3011代表电机故障，0x3009 代表电机角度超限，0x3007代表状态机切换失败，0x3013代表速度命令过大</td>
  </tr>
  <tr>
    <td>备注</td>
    <td>机器狗站低，关节锁定</td>
  </tr>
</table>

<table class="sdk-table">
  <tr>
    <th>函数名</th>
    <th>standUP()</th>
  </tr>
  <tr>
    <td>功能概述</td>
    <td>控制机器狗站立</td>
  </tr>
  <tr>
    <td>参数</td>
    <td>无</td>
  </tr>
  <tr>
    <td>返回值</td>
    <td>进入状态正常返回0，返回0x3012代表电机数据丢失，0x3010代表电机失能， 0x3011代表电机故障，0x3009 代表电机角度超限，0x3007代表状态机切换失败，0x3013代表速度命令过大</td>
  </tr>
  <tr>
    <td>备注</td>
    <td>机器狗正常站立，关节锁定</td>
  </tr>
</table>

<table class="sdk-table">
  <tr>
    <th>函数名</th>
    <th>passive()</th>
  </tr>
  <tr>
    <td>功能概述</td>
    <td>控制机器狗紧急趴下</td>
  </tr>
  <tr>
    <td>参数</td>
    <td>无</td>
  </tr>
  <tr>
    <td>返回值</td>
    <td>进入状态正常返回0，返回0x3012代表电机数据丢失，0x3010代表电机失能， 0x3011代表电机故障，0x3009 代表电机角度超限，0x3007代表状态机切换失败，0x3013代表速度命令过大</td>
  </tr>
  <tr>
    <td>备注</td>
    <td>机器狗进入阻尼状态，趴下</td>
  </tr>
</table>

<table class="sdk-table">
  <tr>
    <th>函数名</th>
    <th>move(float vx, float vy, float yaw_rate)</th>
  </tr>
  <tr>
    <td>功能概述</td>
    <td>控制机器狗运动</td>
  </tr>
  <tr>
    <td>参数</td>
    <td>前向速度、侧向速度、绕Z轴角速度</td>
  </tr>
  <tr>
    <td>返回值</td>
    <td>进入状态正常返回0，返回0x3012代表电机数据丢失，0x3010代表电机失能， 0x3011代表电机故障，0x3009 代表电机角度超限，0x3007代表状态机切换失败，0x3013代表速度命令过大</td>
  </tr>
  <tr>
    <td>备注</td>
    <td>机器狗站立状态下切入，控制狗按指定速度运动。 vx范围（-3m/s~-0.4m/s; 0.4m/s~3m/s）vy 范围（-2.0m/s~-0.2m/s;0.2m/s~2.0m/s）yaw_rate范围（-2.0rad/s-2.0rad/s）</td>
  </tr>
</table>

<table class="sdk-table">
  <tr>
    <th>函数名</th>
    <th>jump()</th>
  </tr>
  <tr>
    <td>功能概述</td>
    <td>控制机器狗垂直跳跃</td>
  </tr>
  <tr>
    <td>参数</td>
    <td>无</td>
  </tr>
  <tr>
    <td>返回值</td>
    <td>进入状态正常返回0，返回0x3012代表电机数据丢失，0x3010代表电机失能， 0x3011代表电机故障，0x3009 代表电机角度超限，0x3007代表状态机切换失败，0x3013代表速度命令过大</td>
  </tr>
  <tr>
    <td>备注</td>
    <td>垂直跳高，基于站立状态下切入</td>
  </tr>
</table>

<table class="sdk-table">
  <tr>
    <th>函数名</th>
    <th>frontJump()</th>
  </tr>
  <tr>
    <td>功能概述</td>
    <td>控制机器狗向前跳</td>
  </tr>
  <tr>
    <td>参数</td>
    <td>无</td>
  </tr>
  <tr>
    <td>返回值</td>
    <td>进入状态正常返回0，返回0x3012代表电机数据丢失，0x3010代表电机失能， 0x3011代表电机故障，0x3009 代表电机角度超限，0x3007代表状态机切换失败，0x3013代表速度命令过大</td>
  </tr>
  <tr>
    <td>备注</td>
    <td>站立状态下切入</td>
  </tr>
</table>

<table class="sdk-table">
  <tr>
    <th>函数名</th>
    <th>backflip()</th>
  </tr>
  <tr>
    <td>功能概述</td>
    <td>控制机器狗后空翻</td>
  </tr>
  <tr>
    <td>参数</td>
    <td>无</td>
  </tr>
  <tr>
    <td>返回值</td>
    <td>进入状态正常返回0，返回0x3012代表电机数据丢失，0x3010代表电机失能， 0x3011代表电机故障，0x3009 代表电机角度超限，0x3007代表状态机切换失败，0x3013代表速度命令过大</td>
  </tr>
  <tr>
    <td>备注</td>
    <td>站立状态下切入</td>
  </tr>
</table>

<table class="sdk-table">
  <tr>
    <th>函数名</th>
    <th>attitudeControl(float yaw_vel, float roll_vel, float pitch_vel, float height_vel)</th>
  </tr>
  <tr>
    <td>功能概述</td>
    <td>控制机器狗原地扭动和身体高度变化</td>
  </tr>
  <tr>
    <td>参数</td>
    <td>无</td>
  </tr>
  <tr>
    <td>返回值</td>
    <td>进入状态正常返回0，返回0x3012代表电机数据丢失，0x3010代表电机失能， 0x3011代表电机故障，0x3009 代表电机角度超限，0x3007代表状态机切换失败，0x3013代表速度命令过大</td>
  </tr>
  <tr>
    <td>备注</td>
    <td>机器狗站立状态下切入，控制狗按指定速度原地扭动，身体高度变化。 yaw_vel, pitch_vel, roll_vel范围（-0.5rad/s~0.5rad/s）height_vel范围（-0.5m/s-0.5m/s）</td>
  </tr>
</table>

<table class="sdk-table">
  <tr>
    <th>函数名</th>
    <th>getRoll()</th>
  </tr>
  <tr>
    <td>功能概述</td>
    <td>获取机器狗当前翻滚角</td>
  </tr>
  <tr>
    <td>参数</td>
    <td>无</td>
  </tr>
  <tr>
    <td>返回值</td>
    <td>返回当前翻滚角度</td>
  </tr>
  <tr>
    <td>备注</td>
    <td>单位弧度</td>
  </tr>
</table>

<table class="sdk-table">
  <tr>
    <th>函数名</th>
    <th>getPitch()</th>
  </tr>
  <tr>
    <td>功能概述</td>
    <td>获取机器狗当前俯仰角</td>
  </tr>
  <tr>
    <td>参数</td>
    <td>无</td>
  </tr>
  <tr>
    <td>返回值</td>
    <td>返回当前俯仰角度</td>
  </tr>
  <tr>
    <td>备注</td>
    <td>单位弧度</td>
  </tr>
</table>

<table class="sdk-table">
  <tr>
    <th>函数名</th>
    <th>getYaw()</th>
  </tr>
  <tr>
    <td>功能概述</td>
    <td>获取机器狗当前航向角</td>
  </tr>
  <tr>
    <td>参数</td>
    <td>无</td>
  </tr>
  <tr>
    <td>返回值</td>
    <td>返回当前基于开机上电原点z轴的角度</td>
  </tr>
  <tr>
    <td>备注</td>
    <td>单位弧度</td>
  </tr>
</table>

<table class="sdk-table">
  <tr>
    <th>函数名</th>
    <th>getBodyAccX()</th>
  </tr>
  <tr>
    <td>功能概述</td>
    <td>获取机器狗前向加速度</td>
  </tr>
  <tr>
    <td>参数</td>
    <td>无</td>
  </tr>
  <tr>
    <td>返回值</td>
    <td>返回当前前向加速度</td>
  </tr>
  <tr>
    <td>备注</td>
    <td>m/s^2</td>
  </tr>
</table>

<table class="sdk-table">
  <tr>
    <th>函数名</th>
    <th>getBodyAccY()</th>
  </tr>
  <tr>
    <td>功能概述</td>
    <td>获取机器狗侧向加速度</td>
  </tr>
  <tr>
    <td>参数</td>
    <td>无</td>
  </tr>
  <tr>
    <td>返回值</td>
    <td>返回当前侧向加速度</td>
  </tr>
  <tr>
    <td>备注</td>
    <td>m/s^2</td>
  </tr>
</table>

<table class="sdk-table">
  <tr>
    <th>函数名</th>
    <th>getBodyAccZ()</th>
  </tr>
  <tr>
    <td>功能概述</td>
    <td>获取机器狗垂直方向加速度</td>
  </tr>
  <tr>
    <td>参数</td>
    <td>无</td>
  </tr>
  <tr>
    <td>返回值</td>
    <td>返回当前垂直方向加速度</td>
  </tr>
  <tr>
    <td>备注</td>
    <td>m/s^2</td>
  </tr>
</table>

<table class="sdk-table">
  <tr>
    <th>函数名</th>
    <th>getBodyGyroX()</th>
  </tr>
  <tr>
    <td>功能概述</td>
    <td>获取机器狗绕X轴角速度</td>
  </tr>
  <tr>
    <td>参数</td>
    <td>无</td>
  </tr>
  <tr>
    <td>返回值</td>
    <td>返回当前绕X轴角速度</td>
  </tr>
  <tr>
    <td>备注</td>
    <td>单位 rad/s， 机器狗身体前向为X轴正方向</td>
  </tr>
</table>

<table class="sdk-table">
  <tr>
    <th>函数名</th>
    <th>getBodyGyroY()</th>
  </tr>
  <tr>
    <td>功能概述</td>
    <td>获取机器狗绕Y轴角速度</td>
  </tr>
  <tr>
    <td>参数</td>
    <td>无</td>
  </tr>
  <tr>
    <td>返回值</td>
    <td>返回当前绕Y轴角速度</td>
  </tr>
  <tr>
    <td>备注</td>
    <td>单位rad/s， 机器狗身体左向为Y轴正方向</td>
  </tr>
</table>

<table class="sdk-table">
  <tr>
    <th>函数名</th>
    <th>getBodyGyroZ()</th>
  </tr>
  <tr>
    <td>功能概述</td>
    <td>获取机器狗绕Z轴角速度</td>
  </tr>
  <tr>
    <td>参数</td>
    <td>无</td>
  </tr>
  <tr>
    <td>返回值</td>
    <td>返回当前绕Z轴角速度</td>
  </tr>
  <tr>
    <td>备注</td>
    <td>单位rad/s， 机器狗身体垂直向上为Z轴正方向</td>
  </tr>
</table>

<table class="sdk-table">
  <tr>
    <th>函数名</th>
    <th>getPosWorldX()</th>
  </tr>
  <tr>
    <td>功能概述</td>
    <td>获取机器狗相对上电原点X轴方向的位置</td>
  </tr>
  <tr>
    <td>参数</td>
    <td>无</td>
  </tr>
  <tr>
    <td>返回值</td>
    <td>返回当前相对于上电原点X轴方向的位置</td>
  </tr>
  <tr>
    <td>备注</td>
    <td>单位 m, 上电原点为坐标系原点，机器狗前向为X轴，左侧为Y轴，垂直向上为Z轴</td>
  </tr>
</table>

<table class="sdk-table">
  <tr>
    <th>函数名</th>
    <th>getPosWorldY()</th>
  </tr>
  <tr>
    <td>功能概述</td>
    <td>获取机器狗相对上电原点Y轴方向的位置</td>
  </tr>
  <tr>
    <td>参数</td>
    <td>无</td>
  </tr>
  <tr>
    <td>返回值</td>
    <td>返回当前相对于上电原点Y轴方向的位置</td>
  </tr>
  <tr>
    <td>备注</td>
    <td>单位 m, 上电原点为坐标系原点，机器狗前向为X轴，左侧为Y轴，垂直向上为Z轴</td>
  </tr>
</table>

<table class="sdk-table">
  <tr>
    <th>函数名</th>
    <th>getPosWorldZ()</th>
  </tr>
  <tr>
    <td>功能概述</td>
    <td>获取机器狗相对上电原点Z轴方向的位置</td>
  </tr>
  <tr>
    <td>参数</td>
    <td>无</td>
  </tr>
  <tr>
    <td>返回值</td>
    <td>返回当前相对于上电原点Z轴方向的位置</td>
  </tr>
  <tr>
    <td>备注</td>
    <td>单位 m, 上电原点为坐标系原点，机器狗前向为X轴，左侧为Y轴，垂直向上为Z轴</td>
  </tr>
</table>

<table class="sdk-table">
  <tr>
    <th>函数名</th>
    <th>getWorldVelX()</th>
  </tr>
  <tr>
    <td>功能概述</td>
    <td>获取机器狗相对上电原点X轴方向的速度</td>
  </tr>
  <tr>
    <td>参数</td>
    <td>无</td>
  </tr>
  <tr>
    <td>返回值</td>
    <td>返回当前相对于上电原点X轴方向的速度</td>
  </tr>
  <tr>
    <td>备注</td>
    <td>单位 m/s, 上电原点为坐标系原点，机器狗前向为X轴，左侧为Y轴，垂直向上为Z轴</td>
  </tr>
</table>

<table class="sdk-table">
  <tr>
    <th>函数名</th>
    <th>getWorldVelY()</th>
  </tr>
  <tr>
    <td>功能概述</td>
    <td>获取机器狗相对上电原点Y轴方向的速度</td>
  </tr>
  <tr>
    <td>参数</td>
    <td>无</td>
  </tr>
  <tr>
    <td>返回值</td>
    <td>返回当前相对于上电原点Y轴方向的速度</td>
  </tr>
  <tr>
    <td>备注</td>
    <td>单位 m/s, 上电原点为坐标系原点，机器狗前向为X轴，左侧为Y轴，垂直向上为Z轴</td>
  </tr>
</table>

<table class="sdk-table">
  <tr>
    <th>函数名</th>
    <th>getWorldVelZ()</th>
  </tr>
  <tr>
    <td>功能概述</td>
    <td>获取机器狗相对上电原点Z轴方向的速度</td>
  </tr>
  <tr>
    <td>参数</td>
    <td>无</td>
  </tr>
  <tr>
    <td>返回值</td>
    <td>返回当前相对于上电原点Z轴方向的速度</td>
  </tr>
  <tr>
    <td>备注</td>
    <td>单位 m/s, 上电原点为坐标系原点，机器狗前向为X轴，左侧为Y轴，垂直向上为Z轴</td>
  </tr>
</table>

<table class="sdk-table">
  <tr>
    <th>函数名</th>
    <th>getBodyVelX()</th>
  </tr>
  <tr>
    <td>功能概述</td>
    <td>获取机器狗X轴方向的速度</td>
  </tr>
  <tr>
    <td>参数</td>
    <td>无</td>
  </tr>
  <tr>
    <td>返回值</td>
    <td>返回X轴方向的速度</td>
  </tr>
  <tr>
    <td>备注</td>
    <td>单位 m/s, 机器狗身体前向为X轴正方向</td>
  </tr>
</table>

<table class="sdk-table">
  <tr>
    <th>函数名</th>
    <th>getBodyVelY()</th>
  </tr>
  <tr>
    <td>功能概述</td>
    <td>获取机器狗Y轴方向的速度</td>
  </tr>
  <tr>
    <td>参数</td>
    <td>无</td>
  </tr>
  <tr>
    <td>返回值</td>
    <td>返回Y轴方向的速度</td>
  </tr>
  <tr>
    <td>备注</td>
    <td>单位 m/s, 机器狗身体左向为y轴正方向</td>
  </tr>
</table>

<table class="sdk-table">
  <tr>
    <th>函数名</th>
    <th>getBodyVelZ()</th>
  </tr>
  <tr>
    <td>功能概述</td>
    <td>获取机器狗Z轴方向的速度</td>
  </tr>
  <tr>
    <td>参数</td>
    <td>无</td>
  </tr>
  <tr>
    <td>返回值</td>
    <td>返回Z轴方向的速度</td>
  </tr>
  <tr>
    <td>备注</td>
    <td>单位 m/s, 机器狗身体垂直向上为Z轴正方向</td>
  </tr>
</table>

## 5.2. Motor控制接口函数介绍
    

**class MotorFunc**

- 该类用于与电机进行交互，包括发送电机命令和获取电机状态。
    

<table class="sdk-table">
  <tr>
    <th>函数名</th>
    <th>initRobot(const std::string & local_ip, const int local_port， const string& dog_ip = "192.168.234.1")</th>
  </tr>
  <tr>
    <td>功能概述</td>
    <td>与机器狗建立通信,机器狗端ip默认“192.168.234.1”, 如更改机器人ip，需从该接口传入新ip</td>
  </tr>
  <tr>
    <td>参数</td>
    <td>用户主机 ip 和端口</td>
  </tr>
  <tr>
    <td>返回值</td>
    <td>无</td>
  </tr>
  <tr>
    <td>备注</td>
    <td>通信建立失败时，终端输出失败日志</td>
  </tr>
</table>

<table class="sdk-table">
  <tr>
    <th>函数名</th>
    <th>sendMotorCmd(const motorCmd& cmd)</th>
  </tr>
  <tr>
    <td>功能概述</td>
    <td>发布电机控制命令</td>
  </tr>
  <tr>
    <td>参数</td>
    <td>motorCmd</td>
  </tr>
  <tr>
    <td>返回值</td>
    <td>发送成功返回0，失败返回-1</td>
  </tr>
  <tr>
    <td>备注</td>
    <td>腿部顺序： 右前腿为1号腿， 左前腿为2号腿， 右后腿为3号腿， 左后腿为4号腿；控制命令坐标系为机器狗朝前为X，左侧为Y，垂直向上为Z</td>
  </tr>
</table>

- Cmd 数据协议
    

```C++
struct alignas(4) motorCmd  {

  float q_des_abad[4] = {0.0, 0.0, 0.0, 0.0}; // A关节角度指令
  float q_des_hip[4] = {0.0, 0.0, 0.0, 0.0};  // H关节角度指令
  float q_des_knee[4] = {0.0, 0.0, 0.0, 0.0}; // K关节角度指令

  float qd_des_abad[4] = {0.0, 0.0, 0.0, 0.0}; // A关节角速度指令
  float qd_des_hip[4] = {0.0, 0.0, 0.0, 0.0};  // H关节角速度指令
  float qd_des_knee[4] = {0.0, 0.0, 0.0, 0.0}; // K关节角速度指令

  float kp_abad[4] = {0.0, 0.0, 0.0, 0.0}; // A关节Kp值
  float kp_hip[4] = {0.0, 0.0, 0.0, 0.0};  // H关节Kp值
  float kp_knee[4] = {0.0, 0.0, 0.0, 0.0}; // K关节Kp值

  float kd_abad[4] = {0.0, 0.0, 0.0, 0.0}; // A关节Kd值
  float kd_hip[4] = {0.0, 0.0, 0.0, 0.0};  // H关节Kd值
  float kd_knee[4] = {0.0, 0.0, 0.0, 0.0}; // K关节Kd值

  float tau_abad_ff[4] = {0.0, 0.0, 0.0, 0.0}; // A关节扭矩指令
  float tau_hip_ff[4] = {0.0, 0.0, 0.0, 0.0};  // H关节扭矩指令
  float tau_knee_ff[4] = {0.0, 0.0, 0.0, 0.0}; // K关节扭矩指令

};
```

<table class="sdk-table">
  <tr>
    <th>函数名</th>
    <th>getRoll()</th>
  </tr>
  <tr>
    <td>功能概述</td>
    <td>获取机器狗当前翻滚角</td>
  </tr>
  <tr>
    <td>参数</td>
    <td>无</td>
  </tr>
  <tr>
    <td>返回值</td>
    <td>返回当前翻滚角度</td>
  </tr>
  <tr>
    <td>备注</td>
    <td>单位弧度</td>
  </tr>
</table>

<table class="sdk-table">
  <tr>
    <th>函数名</th>
    <th>getPitch()</th>
  </tr>
  <tr>
    <td>功能概述</td>
    <td>获取机器狗当前俯仰角</td>
  </tr>
  <tr>
    <td>参数</td>
    <td>无</td>
  </tr>
  <tr>
    <td>返回值</td>
    <td>返回当前俯仰角度</td>
  </tr>
  <tr>
    <td>备注</td>
    <td>单位弧度</td>
  </tr>
</table>

<table class="sdk-table">
  <tr>
    <th>函数名</th>
    <th>getYaw()</th>
  </tr>
  <tr>
    <td>功能概述</td>
    <td>获取机器狗当前航向角</td>
  </tr>
  <tr>
    <td>参数</td>
    <td>无</td>
  </tr>
  <tr>
    <td>返回值</td>
    <td>返回当前基于开机上电原点z轴的角度</td>
  </tr>
  <tr>
    <td>备注</td>
    <td>单位弧度</td>
  </tr>
</table>

<table class="sdk-table">
  <tr>
    <th>函数名</th>
    <th>getBodyAccX()</th>
  </tr>
  <tr>
    <td>功能概述</td>
    <td>获取机器狗前向加速度</td>
  </tr>
  <tr>
    <td>参数</td>
    <td>无</td>
  </tr>
  <tr>
    <td>返回值</td>
    <td>返回当前前向加速度</td>
  </tr>
  <tr>
    <td>备注</td>
    <td>m/s^2</td>
  </tr>
</table>

<table class="sdk-table">
  <tr>
    <th>函数名</th>
    <th>getBodyAccY()</th>
  </tr>
  <tr>
    <td>功能概述</td>
    <td>获取机器狗侧向加速度</td>
  </tr>
  <tr>
    <td>参数</td>
    <td>无</td>
  </tr>
  <tr>
    <td>返回值</td>
    <td>返回当前侧向加速度</td>
  </tr>
  <tr>
    <td>备注</td>
    <td>m/s^2</td>
  </tr>
</table>

<table class="sdk-table">
  <tr>
    <th>函数名</th>
    <th>getBodyAccZ()</th>
  </tr>
  <tr>
    <td>功能概述</td>
    <td>获取机器狗垂直方向加速度</td>
  </tr>
  <tr>
    <td>参数</td>
    <td>无</td>
  </tr>
  <tr>
    <td>返回值</td>
    <td>返回当前垂直方向加速度</td>
  </tr>
  <tr>
    <td>备注</td>
    <td>m/s^2</td>
  </tr>
</table>

<table class="sdk-table">
  <tr>
    <th>函数名</th>
    <th>getBodyGyroX()</th>
  </tr>
  <tr>
    <td>功能概述</td>
    <td>获取机器狗绕X轴角速度</td>
  </tr>
  <tr>
    <td>参数</td>
    <td>无</td>
  </tr>
  <tr>
    <td>返回值</td>
    <td>返回当前绕X轴角速度</td>
  </tr>
  <tr>
    <td>备注</td>
    <td>单位 rad/s， 机器狗身体前向为X轴正方向</td>
  </tr>
</table>

<table class="sdk-table">
  <tr>
    <th>函数名</th>
    <th>getBodyGyroY()</th>
  </tr>
  <tr>
    <td>功能概述</td>
    <td>获取机器狗绕Y轴角速度</td>
  </tr>
  <tr>
    <td>参数</td>
    <td>无</td>
  </tr>
  <tr>
    <td>返回值</td>
    <td>返回当前绕Y轴角速度</td>
  </tr>
  <tr>
    <td>备注</td>
    <td>单位rad/s， 机器狗身体左向为Y轴正方向</td>
  </tr>
</table>

<table class="sdk-table">
  <tr>
    <th>函数名</th>
    <th>getBodyGyroZ()</th>
  </tr>
  <tr>
    <td>功能概述</td>
    <td>获取机器狗绕Z轴角速度</td>
  </tr>
  <tr>
    <td>参数</td>
    <td>无</td>
  </tr>
  <tr>
    <td>返回值</td>
    <td>返回当前绕Z轴角速度</td>
  </tr>
  <tr>
    <td>备注</td>
    <td>单位rad/s， 机器狗身体垂直向上为Z轴正方向</td>
  </tr>
</table>

<table class="sdk-table">
  <tr>
    <th>函数名</th>
    <th>getMotorState()</th>
  </tr>
  <tr>
    <td>功能概述</td>
    <td>获取机器狗电机数据</td>
  </tr>
  <tr>
    <td>参数</td>
    <td>无</td>
  </tr>
  <tr>
    <td>返回值</td>
    <td>返回各电机数据</td>
  </tr>
  <tr>
    <td>备注</td>
    <td>单位rad</td>
  </tr>
</table>

- MotorState 数据协议
    

```C++
struct alignas(4) motorState {
  // 关节顺序 FL,FR,RL,RR
  float q_abad[4] = {0.0, 0.0, 0.0, 0.0}; // A关节角度
  float q_hip[4] = {0.0, 0.0, 0.0, 0.0};  // H关节角度
  float q_knee[4] = {0.0, 0.0, 0.0, 0.0}; // K关节角度


  float qd_abad[4] = {0.0, 0.0, 0.0, 0.0}; // A关节角速度
  float qd_hip[4] = {0.0, 0.0, 0.0, 0.0};  // H关节角速度
  float qd_knee[4] = {0.0, 0.0, 0.0, 0.0}; // K关节角速度


  float tau_abad_fb[4] = {0.0, 0.0, 0.0, 0.0};// A关节扭矩反馈
  float tau_hip_fb[4] = {0.0, 0.0, 0.0, 0.0};  // H关节扭矩反馈
  float tau_knee_fb[4] = {0.0, 0.0, 0.0, 0.0}; // K关节扭矩反馈
};
```

<table class="sdk-table">
  <tr>
    <th>函数名</th>
    <th>haveMotorData()</th>
  </tr>
  <tr>
    <td>功能概述</td>
    <td>判断是否获取到电机数据</td>
  </tr>
  <tr>
    <td>参数</td>
    <td>无</td>
  </tr>
  <tr>
    <td>返回值</td>
    <td>返回true or false</td>
  </tr>
  <tr>
    <td>备注</td>
    <td>在下发电机命令之前先判断是否收到电机数据</td>
  </tr>
</table>