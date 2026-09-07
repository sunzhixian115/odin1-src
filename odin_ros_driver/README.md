# Odin_ROS_Driver README

Odin 传感器模块（Manifold Tech Ltd.）的 ROS 驱动套件。

Odin1 Wiki：

https://manifoldtechltd.github.io/wiki/Odin1/Cover.html

## Odin_ROS_Driver

### 兼容性

* ROS 1（推荐使用 LTS 版本：Noetic）
* ROS 2（推荐使用 LTS 版本：Humble）

## 重要提示

本驱动包为点云 SLAM 应用提供核心功能，并针对特定使用场景设计。它仅面向进行二次开发的专业技术人员。

最终用户在实际部署环境中，必须根据具体应用场景进行针对性的优化和自定义开发，以满足实际运行需求。

---

# 1. 版本

当前版本：

```text
v0.11.0
```

要求的设备固件版本：

```text
v0.11.11
```

---

# 2. 准备工作

## 2.1 操作系统要求

* Ubuntu 20.04：适用于 ROS Noetic 和 ROS2 Foxy
* Ubuntu 22.04：适用于 ROS2 Humble
* Ubuntu 18.04：目前不支持
* Ubuntu 24.04：官方暂不支持，但经过部分修改后可能可以运行

## 2.2 依赖项

* OpenCV >= 4.2.0
  推荐版本：4.5.5 / 4.8.0
  **请确保系统中只安装一个版本的 OpenCV**
* yaml-cpp
* thread
* OpenSSL
* Eigen3

## 2.3 安装依赖

### 2.3.1 系统依赖

```shell
sudo apt update

sudo apt-get install build-essential cmake git libgtk2.0-dev pkg-config libavcodec-dev libavformat-dev libswscale-dev
```

### 2.3.2 yaml-cpp

```shell
sudo apt update

sudo apt install -y libyaml-cpp-dev
```

### 2.3.3 libusb

```shell
sudo apt update

sudo apt install -y libusb-1.0-0-dev
```

### 2.3.4 OpenCV

```shell
sudo apt update

sudo apt-get install libopencv-dev
```

### 2.3.5 ROS 安装

ROS Noetic 安装请参考：

ROS Noetic installation instructions

ROS2 Foxy 安装请参考：

ROS Foxy installation instructions

ROS2 Humble 安装请参考：

ROS Humble installation instructions

---

# 3. 准备工作

## 3.1 创建 Udev 规则

```shell
sudo vim /etc/udev/rules.d/99-odin-usb.rules
```

在 `99-odin-usb.rules` 文件中添加以下内容：

```shell
SUBSYSTEM=="usb", ATTR{idVendor}=="2207", ATTR{idProduct}=="0019", MODE="0666", GROUP="plugdev"
```

重新加载规则并重新插拔设备：

```shell
sudo udevadm control --reload

sudo udevadm trigger
```

## 3.2 获取源码

```shell
git clone https://github.com/manifoldsdk/odin_ros_driver.git catkin_ws/src/odin_ros_driver
```

注意：

请务必将源码克隆到：

```text
[ros_workspace]/src/
```

目录下，否则可能会出现编译错误。

## 3.3 编译

### 3.3.1 ROS1（以 Noetic 为例）

```shell
source /opt/ros/noetic/setup.bash

./script/build_ros.sh
```

### 3.3.2 ROS2（以 humble 为例）

```shell
source /opt/ros/humble/setup.bash

./script/build_ros2.sh
```

## 3.4 运行

### 3.4.1 ROS1（以 Noetic 为例）

```shell
source [ros_workspace]/devel/setup.bash

roslaunch odin_ros_driver [launch file]
```

其中：

* `odin_ros_driver`：功能包名称
* `launch file`：启动文件
* `ros_workspace`：用户的 ROS 工作空间

示例：

```shell
roslaunch odin_ros_driver odin1_ros1.launch
```

### 3.4.2 ROS2（以 humble 为例）


source install/setup.bash

ros2 launch odin_ros_driver odin1_ros2.launch.py


其中：

* `odin_ros_driver`：功能包名称
* `launch file`：启动文件
* `ros2_workspace`：用户的 ROS2 工作空间

ROS2 Demo 启动示例：

```shell
ros2 launch odin_ros_driver odin1_ros2.launch.py
```

---

# 3.5 工作模式

工作模式可以通过：

```text
config/control_command.yaml
```

中的：

```yaml
custom_map_mode
```

参数进行配置。

---

## 里程计模式（Odometry mode）

设置：

```yaml
custom_map_mode = 0
```

启用里程计模式。

在此模式下：

```text
map frame
```

和：

```text
odom frame
```

具有相同的位姿。

如果发现 `odom` 数据发生漂移，可以使用以下脚本命令动态重置算法：

```shell
./set_param.sh algo_reset 1
```

---

## SLAM 模式

设置：

```yaml
custom_map_mode = 1
```

启用 SLAM 模式。

该模式提供完整的 SLAM 系统，在 Odometry 模式的基础上增加：

* **回环检测（Loop Closure Detection）**
* **地图保存（Map Saving）**

功能。

启动驱动后，Odin1 会自动开始建图并缓存地图数据。

当场景采集完成后，需要在驱动源码目录中执行：

```shell
./set_param.sh save_map 1
```

保存程序启动以来收集的全部地图数据。

地图保存位置由：

```yaml
mapping_result_dest_dir
```

和：

```yaml
mapping_result_file_name
```

两个参数决定，它们位于：

```text
config/control_command.yaml
```

中。

如果没有指定，则使用默认值。

第一次保存后，可以再次执行保存命令生成新的地图。

每一次保存操作都会生成一个新的地图文件。

> 注意：连续两次保存操作之间请至少间隔 5 秒。

地图原点对应于程序启动时：

```text
odom 坐标系的原点
```

---

### 重定位模式（Relocalization mode）

要启用重定位模式，设置：

```yaml
custom_map_mode = 2
```

并且需要通过：

```yaml
relocalization_map_abs_path
```

指定预先构建好的地图的**绝对路径**。

启动后，Odin1 会根据当前视角和指定地图自动开始重定位。

为了保证较高的成功率，建议设备的初始位置满足：

* 距离原 SLAM 轨迹位置：1 米以内
* 姿态误差：±10°

即建议从原来建图时的附近位置和朝向开始启动。

需要注意的是，重定位性能高度依赖环境。

在特征非常明显的环境中，即使超出：

```text
1m / 10°
```

范围，也可能成功匹配。

而在某些环境中，可能需要更加严格的初始位置条件。

建议在实际目标环境中进行测试，以确定实际可接受的误差范围。

如果初始重定位失败：

系统会暂时进入一种**备用 SLAM 模式**。

在此状态下：

* 不会保存地图
* 用户可以自由移动 Odin1
* 系统会在后台持续尝试重定位

一旦重定位成功：

系统将发布：

```text
map → odom
```

之间的 TF。

> 提示：初始化后轻微摇晃或移动设备，有助于提高重定位成功率。

以下 Topic 发布在 `odom` 坐标系下：

```text
/odin1/cloud_slam
/odin1/odom
/odin1/highodom
/odin1/path
```

如果需要获得它们在 `map` 坐标系下的数据，需要应用：

```text
odom → map
```

之间的 TF 变换。

---

# 4. 文件结构和数据格式

## 4.1 文件结构

```text
Odin_ROS_Driver/                // ROS1/ROS2 驱动包

    3rdparty/                   // 第三方库

    src/

        host_sdk_sample.cpp
        // 示例源码

        yaml_parser.cpp
        // YAML 参数读取源码

        rawCloudRender.cpp
        // RenderCloud 源码

        depth_image_ros_node.cpp
        // ROS1 深度图节点

        depth_image_ros2_node.cpp
        // ROS2 深度图节点

        pcd2depth_ros.cpp
        // ROS1 pcd2depth 节点

        pcd2depth_ros2.cpp
        // ROS2 pcd2depth 节点

        pointcloud_depth_converter.cpp
        // 点云转深度图源码

        cloud_reprojection_ros.cpp
        // 点云重投影节点源码（ROS1/ROS2）

        cloud_reprojector.cpp
        // 点云重投影核心逻辑

    lib/

        liblydHostApi_amd.a
        // AMD 平台静态库

        liblydHostApi_arm.a
        // ARM 平台静态库

    include/

        host_sdk_sample.h
        // 示例头文件

        lidar_api_type.h
        // API 数据结构头文件

        lidar_api.h
        // API 函数声明

        yaml_parser.h
        // 参数文件读取头文件

        rawCloudRender.h
        // RenderCloud API

        data_logger.h
        // save_data 日志相关

        depth_image_ros_node.hpp

        depth_image_ros2_node.hpp

        pointcloud_depth_converter.hpp

        cloud_reprojection_ros_node.hpp

        cloud_reprojector.hpp
        // 点云重投影核心类

    config/

        control_command.yaml
        // 驱动控制参数文件

        calib.yaml
        // 设备标定 YAML
        // 每台设备不同，每次连接 ROS 驱动时从设备获取

    launch_ROS1/

        odin1_ros1.launch
        // ROS1 启动文件

    launch_ROS2/

        odin1_ros2.launch.py
        // ROS2 启动文件

    script/

        build_ros1.sh
        // ROS1 编译脚本

        build_ros2.sh
        // ROS2 编译脚本

    recorddata/
        // 保存可导入 MindCloud 的录制数据

    log/
        // 日志文件目录

        Driver_{timestamp}/
        // 每次驱动启动创建一个目录

            Conn_{timestamp}/
            // 每次 Odin1 设备连接创建一个目录

                dev_status.csv
                // 设备状态日志

    README.md
    // 使用说明

    CMakeLists.txt
    // CMake 编译文件

    License
    // 许可证文件
```

---

## 4.2 Launch 文件

| Launch 文件名称            | 描述                          |
| ---------------------- | --------------------------- |
| `odin1_ros1.launch`    | ROS1 - Odin1 基础操作 Demo 启动文件 |
| `odin1_ros2.launch.py` | ROS2 - Odin1 基础操作 Demo 启动文件 |

---

# 4.3 ROS Topics

Odin ROS Driver 的内部参数定义在：

```text
config/control_command.yaml
```

以下是常用 Topic 及对应参数说明。

| Topic                               | control_command.yaml 参数 | 详细说明                                                                                                        |
| ----------------------------------- | ----------------------- | ----------------------------------------------------------------------------------------------------------- |
| `odin1/imu`                         | `sendimu`               | IMU Topic                                                                                                   |
| `odin1/image`                       | `sendrgb`               | RGB 相机 Topic，由设备原始 JPEG 数据解码，格式为 `bgr8`                                                                     |
| `odin1/image_undistort`             | `sendrgbundistort`      | 去畸变 RGB 图像，使用设备提供的 `calib.yaml` 处理                                                                          |
| `odin1/image/compressed`            | `sendrgbcompressed`     | RGB 相机压缩 Topic，直接使用设备原始 JPEG 数据                                                                             |
| `odin1/cloud_raw`                   | `senddtof`              | 原始点云 Topic                                                                                                  |
| `odin1/cloud_render`                | `sendcloudrender`       | 渲染点云 Topic，由原始点云、RGB 图像和 `calib.yaml` 处理得到                                                                  |
| `odin1/cloud_slam`                  | `sendcloudslam`         | SLAM 点云 Topic                                                                                               |
| `odin1/odometry`                    | `sendodom`              | 里程计 Topic                                                                                                   |
| `odin1/odometry_high`               | `sendodom`              | 高频里程计 Topic                                                                                                 |
| `odin1/path`                        | `showpath`              | 里程计轨迹 Topic                                                                                                 |
| `tf`                                | `sendodom`              | TF 坐标树 Topic                                                                                                |
| `odin1/depth_img_competetion`       | `senddepth`             | 稠密深度图 Topic。Demo 功能，计算量较大。与 `odin1/image_undistort` 一一对应。需要使用数据时建议直接订阅该 Topic，不建议 `echo`。原始数值已经是深度数据，无需再次转换 |
| `odin1/depth_img_competetion_cloud` | `senddepth`             | 稠密深度点云 Topic。Demo 功能，计算量较大                                                                                  |
| `odin1/reprojected_image`           | `sendreprojection`      | 将点云重投影到图像的 Topic。使用里程计将 `cloud_slam` 投影到相机图像，由主机端处理                                                         |

---

# 4.4 数据格式

## 1. 原始点云 cloud_raw

原始点云：

```text
cloud_raw
```

具有以下字段：

```text
float32 x
// X 轴坐标，单位：米

float32 y
// Y 轴坐标，单位：米

float32 z
// Z 轴坐标，单位：米

uint8 intensity
// 反射强度，范围 0–255

uint16 confidence
// 点置信度
// 在典型场景中实际范围约为 0–1300
// 数值越大表示该点越可靠
// 推荐过滤阈值：30–35，需要根据实际情况调整

float32 offset_time
// 相对于基础时间戳的时间偏移
// 单位：秒
```

如果需要在 PCL 中使用该自定义格式，首先定义点类型：

```cpp
/*** LS ***/

namespace ls_ros {

    struct EIGEN_ALIGN16 Point {

        float x;

        float y;

        float z;

        uint8_t intensity;

        uint16_t confidence;

        float offset_time;

        EIGEN_MAKE_ALIGNED_OPERATOR_NEW

    };

}  // namespace ls_ros

POINT_CLOUD_REGISTER_POINT_STRUCT(ls_ros::Point,

      (float, x, x)

      (float, y, y)

      (float, z, z)

      (uint8_t, intensity, intensity)

      (uint16_t, confidence, confidence)

      (float offset_time , offset_time)

)
```

然后可以方便地将 ROS 的：

```cpp
sensor_msgs::PointCloud2
```

转换为 PCL 点云：

```cpp
pcl::PointCloud<ls_ros::Point> ls_cloud;

pcl::fromROSMsg(*msg, ls_cloud);
```

---

## 2. SLAM 点云和渲染点云

以下两种点云：

```text
cloud_slam
```

以及：

```text
cloud_render
```

具有以下字段：

```text
float32 x
// X 坐标，单位：米

float32 y
// Y 坐标，单位：米

float32 z
// Z 坐标，单位：米

float32 rgb
// RGB 颜色值
```

---

# 4.5 其他功能参数

| 参数                                                     | 详细说明                                                                                                                                       |
| ------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------ |
| `use_host_ros_time`                                    | 时间同步模式：`0` 使用 Odin 内部系统时间作为数据时间戳（典型且推荐）；`1` 使用主机接收到数据时的 ROS 时间（大多数用户不推荐）；`2` 通过类似 NTP 的同步方式将 Odin1 时间与主机时间对齐，时间戳位于主机时间轴上                   |
| `strict_usb3.0_check`                                  | 严格检查 USB3.0。如果关闭，即使 USB 连接低于 USB3.0 标准也允许连接                                                                                                |
| `recorddata`                                           | 以特定格式录制数据，可导入 MindCloud(TM) 进行后处理。会占用大量磁盘空间。测试显示 10 分钟数据约占用 9.5GB。录制文件中的 IMU / 图像 / 点云 / Pose / Rotate 时间戳与 `use_host_ros_time` 使用相同时间对齐策略 |
| `devstatuslog`                                         | 设备状态日志。记录 SoC 温度、CPU 使用率、RAM 使用率、dToF 传感器温度等，以及数据收发速率，保存到 `log` 文件夹下的 `devstatus.csv`。每次启动驱动都会创建新的日志文件                                     |
| `showcamerapose`                                       | 显示相机位姿和视场角                                                                                                                                 |
| `custom_map_mode`                                      | 工作模式：模式 `0`：Odometry，map 和 odom 位姿相同；模式 `1`：建图模式，带回环检测，支持保存地图；模式 `2`：重定位模式，需要指定地图文件绝对路径，成功后输出 map 和 odom 之间的 TF                            |
| `custom_init_pos`                                      | 初始化位置，目前未使用                                                                                                                                |
| `relocalization_map_abs_path`                          | 地图文件绝对路径，用于重定位模式                                                                                                                           |
| `mapping_result_dest_dir` 和 `mapping_result_file_name` | 建图模式下地图保存路径和文件名，如果未指定则使用默认值                                                                                                                |

---

# 4.6 通过 ROS Service 在线调整 AE/AWB

驱动提供了 4 个 ROS Service，允许用户在不停止主数据流的情况下，从其他终端动态调整：

* 相机自动曝光（AE）
* 相机自动白平衡（AWB）

底层 SDK 调用与驱动主控制路径共享，并通过内部互斥锁进行串行化，因此可以在正常运行过程中安全调用这些 Service。

## Service 列表

| Service 名称       | 类型                           | 用途                   |
| ---------------- | ---------------------------- | -------------------- |
| `/odin1/get_ae`  | `odin_ros_driver/srv/GetAe`  | 查询当前 AE 状态           |
| `/odin1/get_awb` | `odin_ros_driver/srv/GetAwb` | 查询当前 AWB 状态          |
| `/odin1/set_ae`  | `odin_ros_driver/srv/SetAe`  | 设置 AE 模式以及手动曝光/增益    |
| `/odin1/set_awb` | `odin_ros_driver/srv/SetAwb` | 设置 AWB 模式以及手动 R/B 增益 |

---

## 4.6.1 请求字段、范围及物理意义

### SetAe.Request

| 字段              | 范围                     | 含义                                                |
| --------------- | ---------------------- | ------------------------------------------------- |
| `mode`          | `0`（AUTO）或 `1`（MANUAL） | `0`：设备自行运行 AE 自动曝光，下方两个参数忽略；`1`：设备锁定自动曝光，并应用提供的参数 |
| `exposure_time` | `0.0001 ~ 0.033 s`     | 手动模式下的每帧曝光时间。越长画面越亮，但运动模糊越严重                      |
| `gain`          | `1.0 ~ 64.0`           | 手动模式下的模拟增益。越大画面越亮，但信噪比越差                          |

### SetAwb.Request

| 字段      | 范围                     | 含义                                 |
| ------- | ---------------------- | ---------------------------------- |
| `mode`  | `0`（AUTO）或 `1`（MANUAL） | `0`：设备自动运行 AWB；`1`：锁定 AWB 并使用提供的增益 |
| `rgain` | `0.1 ~ 4.0`            | 红色通道增益。相对于 `bgain` 越大，画面越暖、越偏黄红    |
| `bgain` | `0.1 ~ 4.0`            | 蓝色通道增益。相对于 `rgain` 越大，画面越冷、越偏蓝     |

> Gr / Gb 通道增益由设备固定为 `1.0`，无法调整。

---

## 4.6.2 响应字段

所有 4 个 Service 都返回：

```text
success
```

类型：

```text
bool
```

以及：

```text
rc
```

类型：

```text
int32
```

其中 Get 类 Service 还会返回查询到的状态信息。

### GetAe.Response

| 字段              | 典型范围                | 含义            |
| --------------- | ------------------- | ------------- |
| `exposure_time` | `0.0001 ~ 0.033 s`  | 当前曝光时间        |
| `gain`          | `1.0 ~ 64.0`        | 当前模拟增益        |
| `iso`           | `100 ~ 6400`        | 等效 ISO        |
| `brightness`    | `0 ~ 255`           | 平均画面亮度        |
| `is_converged`  | `0` 或 `1`           | `1` 表示 AE 已收敛 |
| `env_lv`        | `0 ~ 15`            | 环境亮度指数，数值越大越亮 |
| `fps`           | `~10 / ~14.5 / ~29` | 当前帧率          |

### GetAwb.Response

| 字段                  | 典型范围            | 含义                 |
| ------------------- | --------------- | ------------------ |
| `rgain` / `bgain`   | `0.1 ~ 4.0`     | R / B 通道增益         |
| `grgain` / `gbgain` | `1.0`           | Gr / Gb 增益，设备固定    |
| `cct`               | `2500 ~ 8000 K` | 相关色温               |
| `ccri`              | `-50 ~ 50`      | 色温偏离指数，0 表示位于普朗克轨迹 |
| `is_converged`      | `0` 或 `1`       | `1` 表示 AWB 已收敛     |

---

## 4.6.3 rc 返回码

| rc           | 含义                                     |
| ------------ | -------------------------------------- |
| `0`          | 成功                                     |
| `400`        | 设备载荷过短                                 |
| `401`        | 设备不支持该 opcode                          |
| `402`        | 参数长度错误                                 |
| `403`        | **参数超出范围**，通常是手动参数超过允许范围               |
| `404`        | 设备端 Socket 错误                          |
| `405`        | 设备端 `ae_control` 没有响应，请确认 `lydapp` 已运行 |
| `255 (0xFF)` | `ae_control` 返回未知 opcode               |
| `-1`         | SDK 未初始化                               |
| `-2 ~ -5`    | USB 传输异常、超时或返回数据格式错误                   |
| `-100`       | **驱动尚未打开设备**，请等待设备连接成功                 |

---

## 4.6.4 使用示例

### ROS2（Humble）

在一个终端启动驱动，然后打开另一个终端：

```bash
source install/setup.bash

# 查询当前状态

ros2 service call /odin1/get_ae odin_ros_driver/srv/GetAe

ros2 service call /odin1/get_awb odin_ros_driver/srv/GetAwb

# 设置 AE 为自动模式

ros2 service call /odin1/set_ae odin_ros_driver/srv/SetAe "{mode: 0}"

# 设置 AE 为手动模式
# 曝光时间 10ms，增益 4.0

ros2 service call /odin1/set_ae odin_ros_driver/srv/SetAe \
  "{mode: 1, exposure_time: 0.010, gain: 4.0}"

# 设置 AWB 为手动模式
# rgain=1.5，bgain=2.0

ros2 service call /odin1/set_awb odin_ros_driver/srv/SetAwb \
  "{mode: 1, rgain: 1.5, bgain: 2.0}"

# 恢复自动模式

ros2 service call /odin1/set_ae odin_ros_driver/srv/SetAe "{mode: 0}"

ros2 service call /odin1/set_awb odin_ros_driver/srv/SetAwb "{mode: 0}"

# 查看 srv 完整定义

ros2 interface show odin_ros_driver/srv/SetAe
```

### ROS1（Noetic）

启动 Driver 后，在新的终端执行：

```bash
source devel/setup.bash

# 查询

rosservice call /odin1/get_ae

rosservice call /odin1/get_awb

# 设置 AE 手动模式

rosservice call /odin1/set_ae "{mode: 1, exposure_time: 0.010, gain: 4.0}"

# 设置 AWB 手动模式

rosservice call /odin1/set_awb "{mode: 1, rgain: 1.5, bgain: 2.0}"

# 恢复自动模式
# ROS1 要求所有字段都必须填写

rosservice call /odin1/set_ae "{mode: 0, exposure_time: 0.0, gain: 0.0}"

rosservice call /odin1/set_awb "{mode: 0, rgain: 0.0, bgain: 0.0}"

# 查看 srv 定义

rossrv show odin_ros_driver/SetAe
```

---

## 4.6.5 不同场景的推荐起始参数

### AE

| 场景   | `exposure_time`   | `gain`        |
| ---- | ----------------- | ------------- |
| 明亮室外 | `0.001 ~ 0.005 s` | `1.0 ~ 2.0`   |
| 普通室内 | `0.008 ~ 0.015 s` | `2.0 ~ 8.0`   |
| 暗光环境 | `0.020 ~ 0.030 s` | `8.0 ~ 32.0`  |
| 极暗环境 | `0.033 s`         | `32.0 ~ 64.0` |

### AWB

| 目标色调       | `rgain`     | `bgain`     |
| ---------- | ----------- | ----------- |
| 暖色（钨丝灯、夕阳） | `2.0 ~ 2.5` | `1.0 ~ 1.2` |
| 中性（D65 日光） | `1.5 ~ 1.7` | `1.8 ~ 2.0` |
| 冷色（阴天、荧光灯） | `1.2 ~ 1.4` | `2.2 ~ 2.6` |
| 极冷色调       | `1.0`       | `3.0 ~ 4.0` |

---

## 4.6.6 注意事项

* Service 最长可能阻塞约 10 秒等待设备响应。
* 正常情况下延迟通常只有几十毫秒。
* 手动模式不会跨 Driver 或设备重启保存。
* 每次重新连接设备后都会恢复 AUTO 自动模式。
* `rc = -100` 表示 Driver 尚未成功打开设备。
* 请等待 Driver 日志出现：

```text
device connected
```

后再调用 Service。

最大可用：

```text
exposure_time
```

受到帧周期：

```text
1 / fps
```

限制。

例如：

```text
dtof_fps = 290
```

表示：

```text
29 Hz
```

帧周期约：

```text
34 ms
```

此时：

```text
0.033 s
```

的曝光时间已经接近一帧的时间边界。

---

# 5. FAQ

## 5.1 重新启动 Host SDK 时出现 Segmentation Fault

### 错误信息

```text
No device connected after 60 seconds
```

### 解决方案

1. 重新给 Odin 模块上电

即：

```text
断开 Odin 电源后重新连接
```

2. 重新初始化 Odin SDK

即在设备重启后重新执行 SDK。

---

## 5.2 编译时库绑定失败

### 错误信息

```text
ld: cannot find -llydHostApi
```

或者：

```text
symbol lookup errors
```

### 解决方法

#### 1. 清理之前的编译文件

ROS1：

```shell
rm -rf devel/ build/
```

ROS2：

```shell
rm -rf devel/ install/ log/
```

#### 2. 重新执行安装脚本

重新编译驱动。

---

## 5.3 Docker GUI 图形界面转发失败

### 错误信息

```text
Unable to open X display
```

或者：

```text
No protocol specified
```

### 解决方法

```shell
xhost +
```

该命令允许 Docker 容器使用图形界面显示。

---

## 5.4 ROS Driver 出现 get version failed

### 错误信息

```shell
<ERROR><api.cpp:lidar_get_version:672>: get device version fail.

get version failed.
```

### 原因及解决方法

设备固件版本过低。

请升级到最新固件版本。

---

## 5.5 RVIZ 长时间无响应

### 错误现象

RViz 无响应，过一段时间后终端打印：

```text
Device disconnected, waiting for reconnection...
```

### 解决方法

重新给 Odin 模块上电。

---

## 5.6 设备没有响应

### 错误信息

```text
Missed ok response from device,probably wrong interaction procedure.
```

### 解决方法

采用 5.1 中的方法：

重新给 Odin 设备断电再上电，然后重新初始化 SDK。

---

## 5.7 设备没有外部标定文件

### 错误信息

```text
ERROR：Missing camera node 'cam_0'
```

### 解决方法

重新插拔 USB。

---

## 5.8 ROS Driver 数据流启动后立即提示设备断开

### 错误信息

```text
Device ready and streams activated

Device detaching...

Wating for device reconnection...

Device disconnected, waiting for reconnection...
```

### 原因

该问题主要发生在：

```text
ROS2
```

环境中，并且设备连接在复杂网络环境时更容易发生，例如：

* 办公室 WiFi
* 有线网络
* 多设备复杂局域网

ROS2 默认使用广播通信。

复杂网络环境可能导致 ROS2 发布操作发生阻塞，最终导致设备连接断开。

### 解决方法

如果不需要跨设备通信，可以限制 ROS2 只使用本机：

```shell
export ROS_LOCALHOST_ONLY=1
```

如果必须跨设备通信：

尽量简化网络环境。

推荐使用一个仅包含必要设备的小型局域网。

---

## 5.9 ROS Driver 数据流启动后立即崩溃

### 错误信息

```text
Device ready and streams activated

[host_sdk_sample-2] process has died ......
```

### 测试方法

在：

```text
control_command.yaml
```

中关闭：

```yaml
sendrgb = 0
```

即禁用：

```text
odin1/image
```

然后重新测试。

如果关闭 RGB 图像后 Driver 可以正常运行，那么很可能是：

```text
系统安装了多个 OpenCV 版本
```

导致的问题。

### 解决方法

删除系统中未使用的 OpenCV 版本。

确保系统只保留：

```text
一个完整的 OpenCV 版本
```

然后重新编译 Driver。

---

## 5.10 ROS Driver 出现 TF_OLD_DATA ignoring data 警告

### 错误信息

```text
[rviz2-3] Warning: TF_OLD_DATA ignoring data from the past for frame odin1_base_link at time 20.547632 according to authority Authority undetectable

[rviz2-3] Possible reasons are listed at http://wiki.ros.org/tf/Errors%20explained

[rviz2-3]          at line 294 in ./src/buffer_core.cpp
```

### 原因

这是 ROS 和 RViz 的一种保护机制。

它提示用户：

由于时间戳冲突，部分旧 TF 数据被忽略。

通常发生在：

1. ROS Driver 一直运行
2. 用户中途给 Odin 设备断电再重新上电
3. Odin 内部系统时间被重新初始化
4. 新设备数据的时间戳与 RViz 中之前保存的数据发生冲突

### 解决方法

RViz GUI 底部有一个：

```text
Reset
```

按钮。

点击它可以重置 RViz 内部状态，并停止该警告。

---

## 5.11 ROS Driver 出现 unknown cmd code: xx

### 错误信息

```shell
<ERROR><api.cpp:cmd_data_deal:418>: unknow command code 21.
```

### 原因

ROS Driver 版本与设备固件版本不匹配。

设备的新固件增加了新的数据格式，但旧版本 ROS Driver 无法解析。

### 解决方法

确保：

* ROS Driver 使用最新版本
* 设备 Firmware 使用最新版本

并保持版本兼容。

---

## 5.12 USB 设备访问错误

错误可能是：

```text
LIBUSB_ERROR_BUSY
```

或者：

```text
LIBUSB_ERROR_ACCESS
```

### 错误示例

```shell
libusb: error [udev_hotplug_event] ignoring udev action bind

LIBUSB_ERROR_BUSY
```

或者：

```shell
libusb: error [_get_usbfs_fd] libusb couldn't open USB device /dev/bus/usb/xxx/xxx, errno=13

LIBUSB_ERROR_ACCESS
```

### 原因

#### LIBUSB_ERROR_BUSY

另一个进程已经占用了 USB 设备。

常见情况：

* 同时运行了多个 ROS Driver
* 之前崩溃的 Driver 进程仍然占用设备
* 其他程序占用了设备句柄

#### LIBUSB_ERROR_ACCESS

当前用户没有权限访问 USB 设备。

通常原因：

* 没有配置 udev 规则
* 用户权限不足

### LIBUSB_ERROR_BUSY 解决方法

检查是否存在其他 Driver：

```shell
ps aux | grep host_sdk_sample
```

结束已有进程：

```shell
killall host_sdk_sample
```

如果问题仍然存在：

重新插拔 USB 设备以重置设备状态。

### LIBUSB_ERROR_ACCESS 解决方法

#### 1. 添加 udev 规则

创建：

```text
/etc/udev/rules.d/99-odin.rules
```

内容：

```shell
SUBSYSTEM=="usb", ATTR{idVendor}=="2207", ATTR{idProduct}=="0019", MODE="0666", GROUP="plugdev"
```

#### 2. 重新加载规则

```shell
sudo udevadm control --reload-rules

sudo udevadm trigger
```

#### 3. 使用 sudo 运行 Driver

> 不建议在正式环境中使用。

```shell
sudo -E ros2 launch odin_ros_driver odin_ros_driver.launch.py
```

#### 4. 确认用户属于 plugdev 组

```shell
sudo usermod -aG plugdev $USER
```

然后：

```text
注销并重新登录
```

使用户组修改生效。

---

## 5.13 ros2 bag 录制高频 Topic 时出现丢帧

### 现象

使用：

```shell
ros2 bag record
```

录制数据时：

低频 Topic：

* cloud
* image
* odometry
* wiwc

基本完整，没有明显丢失。

但是：

```text
/odin1/imu
```

频率：

```text
400 Hz
```

以及：

```text
/odin1/odometry_highfreq
```

频率：

```text
400 Hz
```

会出现消息丢失。

分析脚本可能发现：

消息之间的时间间隔达到正常周期的：

```text
2 倍甚至更多
```

但是：

* SDK 端没有报告丢帧
* 在线订阅者，例如 `ros2 topic hz` 也没有发现丢帧

---

### 原因

Driver 使用：

```text
RELIABLE
```

QoS 发布：

```text
/odin1/imu
```

和：

```text
/odin1/odometry_highfreq
```

而默认情况下：

```shell
ros2 bag record
```

订阅使用：

```text
history = keep_last
depth = 10
```

对于：

```text
400 Hz
```

的数据流来说：

```text
10 个消息
```

只能缓存大约：

```text
25 ms
```

的数据。

一旦录制程序出现短暂延迟，例如：

* 磁盘写入
* mcap/sqlite 写入数据块
* 系统调度抖动

订阅队列就会溢出。

DDS 会在：

**订阅端**

静默丢弃最旧的数据。

因此：

* SDK 不会发现丢帧
* Publisher 不会发现问题
* `ros2 topic hz` 可能正常

真正丢失发生在：

```text
ros2 bag recorder 的订阅端
```

---

### 解决方案

使用仓库提供的 QoS 配置文件：

```text
script/rosbag2_qos.yaml
```

提高高频 Topic 的订阅队列深度。

配置内容：

```yaml
# script/rosbag2_qos.yaml

/odin1/imu:

  reliability: reliable

  history: keep_last

  depth: 4000


/odin1/odometry_highfreq:

  reliability: reliable

  history: keep_last

  depth: 4000
```

录制时使用：

```shell
ros2 bag record -a \
    --qos-profile-overrides-path src/odin_ros_driver/script/rosbag2_qos.yaml \
    -o my_bag
```

---

### 只录制高频 Topic

也可以：

```shell
ros2 bag record \
    --qos-profile-overrides-path src/odin_ros_driver/script/rosbag2_qos.yaml \
    -o my_bag \
    /odin1/imu /odin1/odometry_highfreq /odin1/odometry /odin1/wiwc /odin1/cloud_raw
```

---

## 可选的进一步优化

如果应用上述 QoS 配置后仍然出现丢帧，通常是磁盘速度较慢导致的。

可以进一步进行以下优化。

### 1. 使用 MCAP 后端

并增加内部缓存：

```shell
ros2 bag record -s mcap --max-cache-size 1073741824 \
    --qos-profile-overrides-path src/odin_ros_driver/script/rosbag2_qos.yaml \
    -o my_bag \
    /odin1/imu /odin1/odometry_highfreq ...
```

MCAP 通常比：

```text
sqlite3
```

录制性能更高。

---

### 2. 增大 Linux UDP Socket Buffer

400Hz 的 RELIABLE 数据流可能受到 UDP Socket Buffer 限制。

默认 Buffer 大约只有：

```text
208 KB
```

可以执行：

```shell
sudo sysctl -w net.core.rmem_max=33554432

sudo sysctl -w net.core.wmem_max=33554432
```

增大系统网络缓冲区。

---

## ROS1 是否存在相同问题？

不存在。

ROS1 使用：

```text
TCP
```

进行发布订阅。

发布端和订阅端主要使用：

```text
queue_size
```

参数，不存在 ROS2 中 Publisher 和 Subscriber QoS Profile 不匹配的问题。

本驱动的 ROS1 实现中：

```text
IMU
```

和：

```text
odometry_highfreq
```

已经设置：

```text
queue_size = 4000
```

位于：

```text
include/host_sdk_sample.h
```

中的：

```text
initialize_publishers
```

ROS1 分支。

同时：

```shell
rosbag record
```

使用 TCP 传输，本身提供可靠传输。

因此：

```text
ROS1
```

环境下不会出现这种特定的高频数据录制丢帧模式。

无需额外配置。

---

# 6. 联系方式

可以通过以下邮箱联系技术支持：

```text
support@manifoldtech.cn
```

为了帮助 FAE 工程师诊断问题，请提供以下信息：

## 1. 当前固件版本

例如：

```shell
[device_version_capture]: ros_driver_version: [Version Number]
```

## 2. 当前使用的电源适配器和转换线照片

## 3. 问题是偶发还是稳定复现

## 4. 提供出现问题时的场景图片

## 5. 第五部分 FAQ 中的故障排查方法是否解决了问题

## 6. 希望问题解决的预期时间
