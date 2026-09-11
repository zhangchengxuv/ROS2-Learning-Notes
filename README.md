# ROS 2 学习笔记

> 一套面向 ROS 2 开发与机器人应用实践的中文学习笔记，覆盖通信机制、工程环境、C++、Qt、ros2_control、Navigation2 及常用硬件接口。

## 项目简介

本仓库由学习记录、代码示例、命令说明和工程排障经验整理而成。为降低长文档的阅读和维护成本，内容按照“章节—专题”两级结构拆分；主页面负责全局导航，各专题页面保留原始笔记并提供章节间跳转。

内容以实践为导向，适合用于 ROS 2 入门复习、功能验证和工程问题检索。示例涉及的系统版本、软件包和硬件环境可能存在差异，实际使用时请结合当前环境验证。

## 内容导航

### [一、ROS2的四种通讯方式及区别](docs/01-ros2-communication/README.md)

系统梳理 ROS 2 的核心通信模型、适用场景与接口定义，为后续节点开发建立统一概念基础。

  - [1-1 ROS 2通信方法](docs/01-ros2-communication/01-communication-methods.md) — 对比话题、服务、动作和参数服务的通信模型、关键特性及适用场景。
  - [1-2 通信机制简介](docs/01-ros2-communication/02-communication-mechanisms.md) — 从节点、话题和数据流角度说明 ROS 2 通信机制的基本组成。
  - [1-3 接口](docs/01-ros2-communication/03-interfaces.md) — 介绍 msg、srv 与 action 接口文件的结构、用途和基本定义方式。

### [二、准备工作](docs/02-preparation/README.md)

完成 ROS 2 开发前的工作空间配置，并回顾示例代码涉及的现代 C++ 基础。

  - [2-1  创建工作空间](docs/02-preparation/01-create-workspace.md) — 说明 ROS 2 工作空间、功能包的创建与 colcon 编译流程。
  - [2-2 用得到的c++中的新特性](docs/02-preparation/02-modern-cpp-basics.md) — 回顾自动类型推导和智能指针等 ROS 2 开发常用的现代 C++ 特性。

### [三、案例复现(猛狮集训营)](docs/03-practical-examples/README.md)

通过通信、坐标变换、可视化、Launch 与仿真案例，串联 ROS 2 应用开发的主要环节。

  - [3-1 话题通信](docs/03-practical-examples/01-topic-communication.md) — 以 C++ 发布者和订阅者为例，演示话题通信的完整实现流程。
  - [3-4 参数服务](docs/03-practical-examples/04-parameter-service.md) — 演示参数的声明、查询、修改与删除，以及参数客户端和服务端的协作方式。
  - [3-5 坐标变换](docs/03-practical-examples/05-coordinate-transforms.md) — 介绍 TF 坐标消息、静态与动态广播，以及坐标系和坐标点变换。
  - [3-6 可视化](docs/03-practical-examples/06-visualization.md) — 涵盖 RViz、URDF、Xacro、关节状态发布及机器人模型仿真的基础实践。
  - [3-7 Launch](docs/03-practical-examples/07-launch.md) — 说明 Launch 文件中的节点、参数、文件包含、分组与事件配置。
  - [3-8 通讯补充](docs/03-practical-examples/08-communication-supplement.md) — 补充多线程执行与时间相关 API 等通信开发知识。
  - [3-9 机器人系统仿真](docs/03-practical-examples/09-robot-simulation.md) — 汇总机器人系统仿真相关的实践入口与后续扩展方向。

### [四、QT](docs/04-qt/README.md)

整理 Qt 开发环境配置、ROS 2 工程集成、界面设计与信号槽机制。

  - [1、安装QT](docs/04-qt/01-install-qt.md) — 记录 Qt 开发环境的安装入口与准备事项。
  - [2、安装ROSProjectManager插件](docs/04-qt/02-ros-project-manager.md) — 说明 ROSProjectManager 插件的安装与用途。
  - [3、QT creator无法输入中文](docs/04-qt/03-chinese-input.md) — 记录 Qt Creator 中文输入问题的处理方法。
  - [4、QT 程序启动流程](docs/04-qt/04-application-startup.md) — 梳理 Qt 程序的启动流程、自定义代码接入、变量初始化与编码规范。
  - [5、UI设计器](docs/04-qt/05-ui-designer.md) — 介绍 Qt UI 设计器相关内容和界面开发入口。
  - [6、QT信号槽](docs/04-qt/06-signals-and-slots.md) — 说明 Qt 信号槽的设计思路、基本机制与自定义实现。

### [五、C++相关](docs/05-cpp/README.md)

汇总工程构建与 C++ 核心语法，为 ROS 2 节点和库开发提供语言基础。

  - [5-1 CMake](docs/05-cpp/01-cmake.md) — 覆盖 CMake 源文件组织、变量、编译标准、头文件路径及静态库和动态库配置。
  - [5-2 C++核心编程](docs/05-cpp/02-cpp-core.md) — 整理内存模型、引用、函数、类与对象、this 指针和继承等核心主题。

### [六、Ros2_contral](docs/06-ros2-control/README.md)

聚焦 ros2_control 的控制器、硬件接口、机器人模型准备与自定义硬件接入。

  - [6-1 概论](docs/06-ros2-control/01-overview.md) — 概述 ros2_control 的定位、核心组件与整体工作方式。
  - [6-2 调用controllers和hardware_interfaces](docs/06-ros2-control/02-controllers-and-hardware-interfaces.md) — 说明控制器与硬件接口的配置、调用和运行流程。
  - [6-3 常用ros2_control标签](docs/06-ros2-control/03-common-tags.md) — 整理机器人描述中常用的 ros2_control 标签及其作用。
  - [6-4 准备rrbot](docs/06-ros2-control/04-prepare-rrbot.md) — 记录 RRBot 示例模型及控制环境的准备过程。
  - [6-5 Writing a new hardware interface](docs/06-ros2-control/05-new-hardware-interface.md) — 梳理自定义硬件接口的设计、实现与接入步骤。

### [七、Navigation2](docs/07-navigation2/README.md)

整理 Navigation2 导航框架相关资料与学习入口，为定位、规划和导航实践提供索引。

### [附录](docs/08-appendices/README.md)

收录 C++ 扩展、串口、CAN、网络、传感器、IMU、STM32、Modbus 与相机等工程参考资料。

  - [附录一 C++相关](docs/08-appendices/01-cpp-extensions.md) — 补充命名空间等 C++ 相对 C 语言的扩展知识。
  - [附录二 serial串口通讯](docs/08-appendices/02-serial-communication.md) — 整理 Serial 库参考资料、常见故障与串口通信示例。
  - [附录三 Ros常用消息](docs/08-appendices/03-ros-messages.md) — 汇总 ROS 开发中常见消息类型的参考入口。
  - [附录四 Can通讯](docs/08-appendices/04-can-communication.md) — 记录 CAN 通信依赖、权限问题与电机控制实践。
  - [附录五 什么是ip地址？子关掩码？ip地址段](docs/08-appendices/05-network-basics.md) — 说明 IP 地址、子网掩码、网段判断及 ROS 2 多机通信配置。
  - [附录六 传感器的使用](docs/08-appendices/06-sensors.md) — 整理编码器等传感器的接入、排障与示例程序。
  - [附录七 N100 IMU的使用](docs/08-appendices/07-n100-imu.md) — 介绍 N100 IMU 的驱动安装、数据订阅与可视化流程。
  - [附录七 为IMU的ROS项目设计QT界面](docs/08-appendices/07-imu-qt-interface.md) — 记录在 Qt 中搭建 ROS 2 框架、设计 IMU 界面与实现多线程的方法。
  - [附录八 以ROS2作为上位机控制STM32](docs/08-appendices/08-ros2-stm32.md) — 提供 ROS 2 作为上位机与 STM32 协同控制的实践入口。
  - [附录九 ROS2中如何使用MODBUS RS485](docs/08-appendices/09-modbus-rs485.md) — 说明 Modbus RS485 库的安装和基础使用方式。
  - [附录十 在ROS2环境下使用相机设备](docs/08-appendices/10-camera-devices.md) — 整理相机设备选择、固定设备名称、设备 ID 获取与规则配置。

## 阅读建议

- 初次学习建议依次阅读通信基础、准备工作和案例复现，再进入 Qt、C++ 与 ros2_control 专题。
- 已有 ROS 2 基础时，可直接通过上方专题链接定位命令、代码示例或排障记录。
- 文档中的图片统一存放在 [images](images/) 目录，专题正文位于 [docs](docs/) 目录。

## 文档结构

```text
ROS2-Learning-Notes/
├── README.md        # 项目说明与全局导航
├── docs/            # 按章节拆分的专题文档
└── images/          # 文档使用的本地图片
```

## 说明

这些内容来源于个人学习与项目实践，重点是保存可复用的过程、示例与问题处理经验。欢迎根据实际使用情况补充说明或修正疏漏。
