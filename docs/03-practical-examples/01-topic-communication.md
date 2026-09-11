# 三、案例复现(猛狮集训营)

> 以 C++ 发布者和订阅者为例，演示话题通信的完整实现流程。

[← 返回总目录](../../README.md) · [↑ 返回本章目录](README.md) · [下一节：3-4 参数服务 →](04-parameter-service.md)

---

## 3-1 话题通信

**应用场景举例：**机器人在执行导航功能，使用的传感器是激光雷达，机器人会采集激光雷达感知到的信息并计算，然后生成运动控制信息驱动机器人底盘运动。
--|以激光雷达信息的采集处理为例，在 ROS 中有一个节点**需要时时的发布**当前雷达**采集到的数据**，导航模块中也有节点会**订阅并解析雷达数据**

以此类推，像雷达、摄像头、GPS.... 等等一些**传感器数据的采集**，也都是使用了话题通信，话题通信适用于不断更新的数据传输相关的应用场景

### 3-1-1 案例实现（原生消息）

需求：发布方以某个频率发布一段文本，订阅方订阅消息并输出在终端

```python
# 需要进入src目录，调用以下命令创建c++功能包
ros2 pkg create cpp01_topic --build-type ament_cmake --dependencies rclcpp std_msgs base_interfaces_demo --node-name demo01_talker_str
# 其中包括了创建c++功能包 添加clcpp std_msgs base_interfaces_demo依赖，以及创建demo01_talker_str节点
```

总体内容

1、包含头文件

2、初始化ROS2客户端

3、定义节点类

​	3-1 创建消息发布方

​	3-2 创建定时器

​	3-3 组织并发布消息

4、调用spin函数，并传入节点对象指针

​	在ROS2中，`spin()` 函数通常用于以下目的：

​	**启动事件循环**：`spin()` 函数启动一个事件循环，使得节点能够异步地接收和处理消息。这允许节点在等待消息到来时，同时执行其他任务。

​	**处理回调函数**：当节点订阅了某个主题或者服务时，它会注册相应的回调函数。`spin()` 函数会调用这些回调函数，以响应接收到的消息或服务请求。

​	**保持节点活跃**：`spin()` 函数确保节点保持活跃状态，即使没有消息到来，节点也不会退出。

​	**处理时间延迟**：在某些情况下，`spin()` 函数可以用来处理时间延迟，例如，通过调用`rclcpp::spin(std::chrono::milliseconds(100))`，可以设置节点	每100毫秒检查一次消息。

5、释放资源

**发布方**

```c++
//1.包含头文件
#include "rclcpp/rclcpp.hpp"
#include "std_msgs/msg/string.hpp"

//可以直接使用时间单位
using namespace std::chrono_literals;

//3.自定义节点类
class Talker: public rclcpp::Node{
public:
    Talker():Node("talker_node_cpp"),count(0) {
        RCLCPP_INFO(this->get_logger(),"发布节点创建！");

        //3.1创建信息发布方 消息类型 string
        /*
        <std_msgs::msg::String> 消息类型的模板 
        ("chatter",10) 话题名称 Qos 前置练习发布方与订阅方
        */
        publisher_ = this->create_publisher<std_msgs::msg::String>("chatter",10);
        //3.2创建定时器
        //两个参数，时间和回调函数，进行的操作放在回调函数中
        timer_ = this->create_wall_timer(1s,std::bind(&Talker::on_timer,this));
    }

private:
  //创建回调函数
  void on_timer(){
    //创建message对象
    auto message = std_msgs::msg::String();
    message.data = "hello world!" + std::to_string(count++);
    RCLCPP_INFO(this->get_logger(),"发布方发布消息：%s",message.data.c_str());
    //发布对象
    publisher_->publish(message);
  }
  //定义一个发布方
  rclcpp::Publisher<std_msgs::msg::String>::SharedPtr publisher_;
  //
  rclcpp::TimerBase::SharedPtr timer_;
  size_t count;

};


int main(int argc,char ** argv)
{
    //2.初始化ros2客户端
    rclcpp::init(argc,argv);
    //4.调用spin函数
    rclcpp::spin(std::make_shared<Talker>());
    //5.释放资源
    rclcpp::shutdown();

    return 0;
}
```

**订阅方**

```c++
/*订阅发布方发布的消息

包含头文件
初始化ros2客户端
自定义节点类 --|创建订阅方 --|解析并输出数据
调用spain函数，并传入节点对象指针
资源释放
*/


//包含头文件
#include "rclcpp/rclcpp.hpp"
#include "std_msgs/msg/string.hpp"

//自定义节点类
class Listener:public rclcpp::Node{
public:
    Listener():Node("listener_node_cpp"){
        RCLCPP_INFO(this->get_logger(),"订阅方创建！");

        //创建订阅方实现
        subscrition_= this->create_subscription<std_msgs::msg::String>("chatter",10,std::bind(&Listener::do_cb,this,std::placeholders::_1));
    }
private:
    void do_cb(const std_msgs::msg::String &msg){
        //解析并输出
        RCLCPP_INFO(this->get_logger(),"订阅的消息是：%s",msg.data.c_str());

    }
    rclcpp::Subscription<std_msgs::msg::String>::SharedPtr subscrition_;
};

int main(int argc, char const *argv[])
{
    //初始化ros2客户端
    rclcpp::init(argc,argv);
    //调用spain函数，并传入节点对象指针
    rclcpp::spin(std::make_shared<Listener>());
    //释放资源
    rclcpp::shutdown();
    return 0;
}
```

**CMakeLists.txt中的更改**

需要添加订阅方的add_executable、install

**选择性编译**

```shell
colcon build --packages-select cpp01_topic
```

**执行**

开启两个终端后，按照如下格式进行执行

```shell
. install/setup.bash
ros2 run cpp01_topic demo
```
