# 二、准备工作

> 说明 ROS 2 工作空间、功能包的创建与 colcon 编译流程。

[← 返回总目录](../../README.md) · [↑ 返回本章目录](README.md) · [下一节：2-2 用得到的c++中的新特性 →](02-modern-cpp-basics.md)

---

## 2-1  创建工作空间

以下操作在终端执行

```Python
# 创建工作空间及src文件夹
mkdir -p ws01/src
cd ws01
# 编译
colcon build
# 在src下创建功能包
ros2 pkg create --build-type ament_cmake base_interfaces_demo
```
