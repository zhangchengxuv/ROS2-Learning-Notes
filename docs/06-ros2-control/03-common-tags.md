# 六、Ros2_contral

> 整理机器人描述中常用的 ros2_control 标签及其作用。

[← 返回总目录](../../README.md) · [↑ 返回本章目录](README.md) · [← 上一节：6-2 调用controllers和hardware_interfaces](02-controllers-and-hardware-interfaces.md) · [下一节：6-4 准备rrbot →](04-prepare-rrbot.md)

---

## 6-3 常用ros2_control标签

在``bbot.ros2_control.xacro``中，有一些常用的标签

`<ros2_control>`

定义整个控制系统的主要标签，包含硬件和关节的定义。

```xml
<ros2_control name="Name_of_the_hardware" type="system">
  <!-- 硬件和关节的定义 -->
</ros2_control>
```

`<hardware>`

指定硬件接口的插件。

```xml
<hardware>
  <plugin>library_name/ClassName</plugin>
  <!-- 硬件参数 -->
</hardware>
```

`<joint>`

定义机器人的一个关节，包括命令接口和状态接口。

```xml
<joint name="name_of_the_component">
  <!-- 命令接口和状态接口的定义 -->
</joint>
```

<command_interface>

定义发送到硬件的命令类型，如位置、速度或力。

```xml
<command_interface name="interface_name">
  <!-- 命令接口参数 -->
</command_interface>
```

<state_interface>

定义从硬件读取的状态类型。

```xml
<state_interface name="position"/>
```

`<sensor>`

定义传感器，如力扭矩传感器。

```xml
<sensor name="tcp_fts_sensor">
  <!-- 传感器的状态接口和参数 -->
</sensor>
```

`<param>`

用于为硬件或接口定义参数。

```xml
<param name="example_param">value</param>
```

`<plugin>`

指定硬件接口的插件名称。

```xml
<plugin>ros2_control_demo_hardware/RRBotSystemPositionOnlyHardware</plugin>
```

`<gpio>`

定义通用输入输出（GPIO）接口。

```xml
<gpio name="flange_digital_IOs">
  <!-- GPIO接口的定义 -->
</gpio>
```
