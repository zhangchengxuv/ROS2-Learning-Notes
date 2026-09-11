# 附录

> 记录 CAN 通信依赖、权限问题与电机控制实践。

[← 返回总目录](../../README.md) · [↑ 返回本章目录](README.md) · [← 上一节：附录三 Ros常用消息](03-ros-messages.md) · [下一节：附录五 什么是ip地址？子关掩码？ip地址段 →](05-network-basics.md)

---

## 附录四 Can通讯

### |-4-1遇到的问题

#### |-4-1-1 .h和.so文件

同Can分析仪一同的有两个文件，一个是``controlcan.h``一个是``libcontrolcan.so``,前者是库文件后者是动态库文件，在创建好包后，需要将其放在如下的目录中，需要新建``lib``目录

<img src="/home/zhangchenxu/Pictures/Typora/image-20240816031011887.png" alt="image-20240816031011887" style="zoom:50%;" />

之后需要对``CMakeLisrs.txt``文件中内容进行修改

```bash
cmake_minimum_required(VERSION 3.8)
project(cpp_can)

if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  add_compile_options(-Wall -Wextra -Wpedantic)
endif()

include_directories(include) # 添加项目中的include文件夹路径

# find dependencies
find_package(ament_cmake REQUIRED)
find_package(rclcpp REQUIRED)
find_package(std_msgs REQUIRED)

add_executable(can_demo_node src/can_demo_node.cpp)
target_include_directories(can_demo_node PUBLIC
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
  $<INSTALL_INTERFACE:include>)
target_compile_features(can_demo_node PUBLIC c_std_99 cxx_std_17)  # Require C99 and C++17
ament_target_dependencies(
  can_demo_node
  "rclcpp"
  "std_msgs"
)

# 添加Link
target_link_libraries(can_demo_node 
  ${PROJECT_SOURCE_DIR}/lib/libcontrolcan.so
)
# ---

install(TARGETS can_demo_node
  DESTINATION lib/${PROJECT_NAME})

install(
    DIRECTORY lib/                       # 发现你项目中的lib中所有的文件
    DESTINATION lib/${PROJECT_NAME}      # 拷贝到install目录中
  )
install(TARGETS can_demo_node 
    RUNTIME DESTINATION lib/${PROJECT_NAME}    # 程序运行的时候调用install中的路径
  )

if(BUILD_TESTING)
  find_package(ament_lint_auto REQUIRED)
  # the following line skips the linter which checks for copyrights
  # comment the line when a copyright and license is added to all source files
  set(ament_cmake_copyright_FOUND TRUE)
  # the following line skips cpplint (only works in a git repo)
  # comment the line when this package is in a git repo and when
  # a copyright and license is added to all source files
  set(ament_cmake_cpplint_FOUND TRUE)
  ament_lint_auto_find_test_dependencies()
endif()

ament_package()

```

此时编译可以通过，但会报以下错误

```
error while loading shared libraries: libcontrolcan.so: cannot open shared object file: No such file or directory
```

此时结局方案为：将libcontrolcan.so文件移动至``usr/lib/``文件中，这个地方是标准动态库，需要管理员权限才能修改，因此执行以下命令

```
sudo cp /home/zhangchenxu/Documents/ros2_ws/can_co_ws/src/cpp_can/lib/libcontrolcan.so /usr/lib/

即：

sudo cp libcontrolcan.so文件所在的绝对路径 /usr/lib/

```

此时``libcontrolcan.so``文件已被复制到``/usr/lib/``下，此时可以调用

参考：https://blog.csdn.net/m0_65304012/article/details/125444356  

​	   https://blog.csdn.net/Vingnir/article/details/130154715

#### |-4-1-2 权限问题

1、检测是否有设备

在终端输入
```bash
lsusb
```

<img src="/home/zhangchenxu/.config/Typora/typora-user-../../images/image-20240818012725059.png" alt="image-20240818012725059" style="zoom:50%;" />

2、创建新规则

```bash
sudo gedit /etc/udev/rules.d/99-myusb.rules
```

3、复制以下内容并保存

```bash
ACTION=="add",SUBSYSTEMS=="usb", ATTRS{idVendor}=="04d8", ATTRS{idProduct}=="0053",
GROUP="users", MODE="0777"
```



### |-4-2 控制电机

在进行二次开发时，需要知道其二次开发库的使用说明，官方提供了这些文件

https://www.zhcxgd.com/2.html

在文件中“接口函数库（二次开发库）使用说明书”中进行了详细的解释

电机采用海泰HT-03

```c++
#include "rclcpp/rclcpp.hpp"
#include "std_msgs/msg/string.hpp"
#include "controlcan.h"

using namespace std::chrono_literals;

// 3.自定义节点类；
class Mynode : public rclcpp::Node
{
public:
  Mynode() : Node("mynode_node_cpp")
  {
    // 初始化CAN设备
    device_type = VCI_USBCAN2; // 设备类型修改
    device_index = 0;          // 设备的索引
    can_index = 0;             // CAN控制器的索引 CAN1为0，CAN2为1

    if (VCI_OpenDevice(device_type, device_index, 0) != STATUS_OK)
    {
      RCLCPP_FATAL(this->get_logger(), "无法开启CAN设备");
      return;
    }

    
    if (VCI_ReadBoardInfo(device_type, device_index, &board_info) != STATUS_OK)
    {
      RCLCPP_FATAL(this->get_logger(), "获取设备信息失败");
      VCI_CloseDevice(device_type, device_index);
      return;
    }

    // 设置CAN初始化配置
    init_config.AccCode = 0X80000000; // 过滤验收码
    init_config.AccMask = 0xFFFFFFFF; // 过滤屏蔽码
    init_config.Filter = 2; // 只接受标准帧
    init_config.Timing0 = 0x00;
    init_config.Timing1 = 0x14; // 波特率：1000kps
    init_config.Mode = 0; // 正常模式

    if (VCI_InitCAN(device_type, device_index, can_index, &init_config) != STATUS_OK)
    {
      RCLCPP_FATAL(this->get_logger(), "------初始化CAN参数失败------");
      VCI_CloseDevice(device_type, device_index);
      return;
    }

    // 开始CAN通讯
    if (VCI_StartCAN(device_type, device_index, can_index) != STATUS_OK)
    {
      RCLCPP_FATAL(this->get_logger(), "------开启CAN通道失败------");
      VCI_CloseDevice(device_type, device_index);
      return;
    }
    
    // 启动电机
    BYTE Open[8] = {0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xfc};
    // 输出1NM
    BYTE torque_1[8] = {0x7f, 0xff, 0x7f, 0xf0, 0x00, 0x00, 0x08, 0x71};
    // 输出2NM
    BYTE torque_2[8] = {0x7f, 0xff, 0x7f, 0xf0, 0x00, 0x00, 0x08, 0xe3};
    // 置零
    BYTE close[8] = {0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xfd};
    // 关闭
    BYTE zero[8] = {0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xfe};

    RCLCPP_INFO(this->get_logger(), "------电机已启动------");
    SendData(config_node, 0x00000001, Open);
    std::this_thread::sleep_for(100ms);
    RCLCPP_INFO(this->get_logger(), "------数据已发送------");
    SendData(config_node, 0x00000001, torque_2);
    std::this_thread::sleep_for(5s);
    // SendData(config_node, 0x00000001, zero);
    // std::this_thread::sleep_for(100ms);
    SendData(config_node, 0x00000001,close);
    std::this_thread::sleep_for(100ms);
    RCLCPP_INFO(this->get_logger(), "------电机已关闭------");

    VCI_CloseDevice(device_type, device_index);
    RCLCPP_INFO(this->get_logger(), "------关闭CAN设备------");
  }

private:
  rclcpp::Publisher<std_msgs::msg::String>::SharedPtr publisher_;
  // USB-CAN系列接口卡的设备信息
  VCI_BOARD_INFO board_info;
  // 初始化CAN的配置
  VCI_INIT_CONFIG init_config;
  // CAN帧结构体
  VCI_CAN_OBJ config_node;
  DWORD device_type;
  DWORD device_index;
  DWORD can_index;

  void SendData(VCI_CAN_OBJ &handle_obj, const int id, const BYTE *data)
  {
    // 帧ID
    handle_obj.ID = id;
    // 是否是远程帧。=0时为数据帧，=1时为远程帧（数据段空）。
    handle_obj.RemoteFlag = 0;
    // 是否是扩展帧。=0时为标准帧（11位ID），=1时为扩展帧（29位ID）。
    handle_obj.ExternFlag = 0;
    // 数据长度 DLC (<=8)，即CAN帧Data有几个字节。
    handle_obj.DataLen = 8;
    for (int i = 0; i < 8; i++)
    {
      handle_obj.Data[i] = data[i];
    }
    // VCI_Transmit（发送函数） 设备类型 设备索引 CAN通道索引 要发送的帧结构体 VCI_CAN_OBJ数组的首指针 发送的帧数量
    if (VCI_Transmit(device_type, device_index, can_index, &handle_obj, 1) > 0)
    {
    }
    else
    {
      RCLCPP_FATAL(this->get_logger(), "------USB-CAN设备不存在或USB掉线------");
    }
  }
};

int main(int argc, char const *argv[])
{
  // 2.初始化ROS2客户端；
  rclcpp::init(argc, argv);
  // 4.调用spain函数，并传入节点对象指针；
  rclcpp::spin(std::make_shared<Mynode>());
  // 5.资源释放
  rclcpp::shutdown();
  return 0;
}
```
