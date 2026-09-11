# 附录

> 整理 Serial 库参考资料、常见故障与串口通信示例。

[← 返回总目录](../../README.md) · [↑ 返回本章目录](README.md) · [← 上一节：附录一 C++相关](01-cpp-extensions.md) · [下一节：附录三 Ros常用消息 →](03-ros-messages.md)

---

## 附录二 serial串口通讯

### |-1 参考资料

ubuntu cutecom串口调试工具使用方法（图形界面）https://blog.csdn.net/Dontla/article/details/134557362

C++使用serial串口通信 + ROS2示例IMU串口驱动https://blog.csdn.net/zardforever123/article/details/134227412

Ubuntu22.04下ROS2 Humble串口通信https://blog.csdn.net/qq_50972633/article/details/132837550

ROS2环境下的串口通讯https://blog.csdn.net/weixin_53035484/article/details/128135356

serial库 http://wjwwood.io/serial/doc/1.1.0/classserial_1_1_serial.html

### |-2 可能遇到的问题

https://blog.csdn.net/qq_50972633/article/details/132837550

#### |-2-1找不到libserial.so动态链接库

<img src="/home/zhangchenxu/Pictures/Typora/image-20240809221206343.png" alt="image-20240809221206343" style="zoom:50%;" />

**（1）临时方案**

（重新打开terminal后失效）在Terminal中输入以下命令

```
export LD_LIBRARY_PATH=/usr/local/lib:$LD_LIBRARY_PATH
```

**（2）长期方案**

(a) 打开 ld.so.conf 文件

```
sudo gedit /etc/ld.so.conf
```

(b) 在下面加入非标准的动态共享库路径：一般为/src/local/lib。 保存ld.so.conf 文件

(c) 执行ldconfig一下，添加的文件夹内容才能在程序运行时被找到。

```
sudo ldconfig
```



#### |-2-2 程序可以编译，但端口无法打开

<img src="/home/zhangchenxu/Pictures/Typora/image-20240809221925321.png" alt="image-20240809221925321" style="zoom:50%;" />

**（1）检查串口是否存在，是否被占用**

```
ls -l /dev/ttyUSB0
```

**（2）开放串口权限**

```
sudo chmod 777 /dev/ttyUSB0
```



#### |-2-3 串口驱动报 Key was rejected by service 需要签名的问题

https://blog.csdn.net/qq_28680277/article/details/129162559



#### |-2-4 Brltty 导致 USB 转串口连接失败

https://blog.csdn.net/qq_27865227/article/details/125538516



### |-3 实例

#### |-3-1 实例1 打开串口

创建工作空间后创建功能包

在src目录下：

```
ros2 pkg create serial_port --build-type ament_cmake --dependencies rclcpp std_msgs serial_interfaces_demo serial --node-name serial_talke
# 注意添加serial依赖，否则需要对cmakelists.txt及package.xml做出相应更改
```

serial_port

```c++
// 1.包含头文件
#include "rclcpp/rclcpp.hpp"
#include "std_msgs/msg/u_int16.hpp"
#include "serial/serial.h"

// 实例化串口对象
serial::Serial ser;

// 3.自定义节点类；
class Mynode : public rclcpp::Node
{
public:
  Mynode() : Node("mynode_node_cpp")
  {
    RCLCPP_INFO(this->get_logger(), "发布方创建！");
  }
};

int main(int argc, char const *argv[])
{
  // 2.初始化ROS2客户端；
  rclcpp::init(argc, argv);

  // 串口设置
  ser.setPort("/dev/ttyUSB0");                                  // 选择开启的串口
  ser.setBaudrate(9600);                                        // 设置波特率
  serial::Timeout _time = serial::Timeout::simpleTimeout(2000); // 超时等待
  ser.setTimeout(_time);
  ser.open(); // 开启串口
  if (ser.isOpen())
  {
    std::cout << "serial port is open" << std::endl;
  }
  else
  {
    std::cout << "serial port error" << std::endl;
  }

  // 4.调用spain函数，并传入节点对象指针；
  rclcpp::spin(std::make_shared<Mynode>());
  // 5.资源释放
  rclcpp::shutdown();
  return 0;
}
```

返回工作空间目录

```
export LD_LIBRARY_PATH=/usr/local/lib:$LD_LIBRARY_PATH
colcon build
. install/setup.sh
sudo chmod 777 /dev/ttyUSB0
ros2 run serial_port serial_talker
```

<img src="/home/zhangchenxu/Pictures/Typora/image-20240809223800011.png" alt="image-20240809223800011" style="zoom:67%;" />



#### |-3-2 实例2 接受编码器消息

**注意事项**:确保给予串口权限，并且打开串口，确认编码器分辨率

```c++
// 1.包含头文件
#include "rclcpp/rclcpp.hpp"
#include "std_msgs/msg/u_int16.hpp"
#include "serial/serial.h"
#include <chrono>
#include <thread>

// 可以直接使用时间单位
using namespace std::chrono_literals;

// 实例化串口对象
serial::Serial ser;

// 3.自定义节点类；
class Mynode : public rclcpp::Node
{
public:
  Mynode() : Node("mynode_node_cpp")
  {
    RCLCPP_INFO(this->get_logger(), "发布方创建！");

    // 创建发布方
    talker_ = this->create_publisher<std_msgs::msg::UInt16>("talker", 10);

    // 创建定时器
    timer_ = this->create_wall_timer(25ms, std::bind(&Mynode::on_timer, this));
  }

private:
  void on_timer()
  {
    // 向编码器发送消息
    uint8_t senddata[8] = {0x01, 0x03, 0x00, 0x00, 0x00, 0x01, 0x84, 0x0A};
    ser.write(senddata, 8);
    // 等待25ms，确保数据可以被接收
    std::this_thread::sleep_for(std::chrono::milliseconds(25));
    // 为接收的数据创造一个空数组
    uint8_t data[7] = {0};
    ser.read(data, 7);

    // // 输出获取的数据
    // for (size_t i = 0; i < 7; ++i)
    // {
    //   RCLCPP_INFO(this->get_logger(), "%02X ", data[i]);
    // }
    // RCLCPP_INFO(this->get_logger(), "\n"); // 添加换行符以结束输出

    // 取出有效数据位 将第三位左移动8位，将第四放在高16位上
    uint16_t combined_value = (data[3] << 8) | data[4];
    // 转换数据格式
    float value = combined_value * 360.0f / 32768.0f;
    // 打印角度
    RCLCPP_INFO(this->get_logger(), "Combined decimal value: %.2f", value);
  }

  rclcpp::Publisher<std_msgs::msg::UInt16>::SharedPtr talker_;
  rclcpp::TimerBase::SharedPtr timer_;
};

int main(int argc, char const *argv[])
{
  // 2.初始化ROS2客户端；
  rclcpp::init(argc, argv);

  // 串口设置
  try
  {
    ser.setPort("/dev/ttyUSB0");                                  // 选择开启的串口
    ser.setBaudrate(9600);                                        // 设置波特率
    serial::Timeout _time = serial::Timeout::simpleTimeout(2000); // 超时等待
    ser.setTimeout(_time);
    ser.open(); // 开启串口
    if (ser.isOpen())
    {
      std::cout << "\n\n ----serial port is open---- \n\n"
                << std::endl;
    }
    else
    {
      std::cout << "serial port error" << std::endl;
    }
  }
  catch (const serial::IOException &e)
  {
    std::cerr << "\n\n----Serial I/O Exception: " << e.what() << "----\n请检查设备是否连接\n是否开放串口权限\n串口是否占用\n"
              << std::endl;
  }

  // ser.setPort("/dev/ttyUSB0");                                  // 选择开启的串口
  // ser.setBaudrate(9600);                                        // 设置波特率 z
  // serial::Timeout _time = serial::Timeout::simpleTimeout(2000); // 超时等待
  // ser.setTimeout(_time);
  // ser.open(); // 开启串口
  // if (ser.isOpen())
  // {
  //   std::cout << "serial port is open" << std::endl;
  // }
  // else
  // {
  //   std::cout << "serial port error" << std::endl;
  // }

  // 4.调用spain函数，并传入节点对象指针；
  rclcpp::spin(std::make_shared<Mynode>());
  // 5.资源释放
  rclcpp::shutdown();
  return 0;
}
```

**效果如下**

<img src="/home/zhangchenxu/Pictures/Typora/image-20240810195745258.png" alt="image-20240810195745258" style="zoom:67%;" />

##### |-3-2-1 遇到的问题

1、接受消息需要进行处理，直接输出并不是一个7组16进制数字

2、此处的``32768.0f``是通过2的15次方求出，15即是编码器的位数

```c++
// 转换数据格式
float value = combined_value * 360.0f / 32768.0f;
```
