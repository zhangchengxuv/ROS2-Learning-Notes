[TOC]



# 一、ROS2的四种通讯方式及区别

## 1-1 ROS 2通信方法

### 1-1-1 话题（Topics）
- **通信模型**：基于发布/订阅模型。
- **特点**：
  - 异步通信。
  - 松耦合，节点间独立。
  - 支持一对多或多对多通信。
- **用途**：适用于高频率的数据流，如传感器数据或状态更新。

### 1-1-2 服务（Services）
- **通信模型**：基于请求/响应模型。
- **特点**：
  - 同步通信，客户端等待响应。
  - 确保了通信的双向性和交互性。
  - 一对一通信。
- **用途**：适用于需要即时反馈的交互，如请求机器人状态或执行特定动作。

### 1-1-3 动作（Actions）
- **通信模型**：基于目标/反馈/结果模型。
- **特点**：
  - 异步请求-反馈-响应。
  - 支持长时间运行的任务。
  - 允许任务取消。
- **用途**：适合需要较长时间完成的任务，如导航或复杂数据处理，提供进度反馈。

### 1-1-4 参数服务器（Parameter Server）
- **通信模型**：基于配置参数的共享模型。
- **特点**：
  - 动态配置，可运行时修改。
  - 结构化参数存储。
  - 同步操作。
- **用途**：用于跨节点的配置管理，实现节点行为的动态调整。

### 适用场景选择
- **话题**：适用于数据流的传输，无需即时响应。
- **服务**：适用于需要快速响应的请求。
- **动作**：适用于长时间运行的任务，需要反馈和取消能力。
- **参数服务器**：适用于跨节点共享配置信息。

## 1-2 通信机制简介

### 1-2-1 节点

通信对象的构建都依赖于节点(Node)，在 ROS2 中，一般情况下每个节点都对应某一单一的功能模块(例如：雷达驱动节点可能负责发布雷达消息，摄像头驱动节点可能负责发布图像消息)。一个完整的机器人系统可能由许多协同工作的节点组成， **ROS2 中的单个可执行文件(C++程序或 Python 程序)可以包含一个或多个节点**

### 1-2-2 话题

话题(Topic)是一个纽带，具有相同话题的节点可以关联在一起，而这正是通信的前提。并且 **ROS2是跨语言的**，有的节点可能是使用 C++实现，有的节点可能是使用 Python 实现的，但是只要二者使用了相同的话题，就可以实现数据的交互

### 1-2-3 通信模型

话题通信：单向通信，数据流单项由发布方传输到订阅方

服务通信：基于请求响应的通信模型，客户端发送请求数据到服务端，服务端响应结果给客户端

动作通信： 连续反馈的通信模型，客户端发送请求数据到服务端，服务端响应结果给客户端，但是在服务端接收到请求到产生最终响应的过程中，会发送连续的反馈信息到客户端

参数服务：是一种基于共享的通信模型，在通信双方中，服务端可以设置数据，而客户端可以连
接服务端并操作服务端数据。

## 1-3 接口

在通信过程中，需要传输数据，就必然涉及到数据载体，也即要以特定格式传输数据。在 ROS2 中，数据载体称之为接口(interfaces)。通信时使用的数据载体一般需要使用接口文件定义。

### 1-3-1 msg文件（话题）

msg 文件是用于定义话题通信中数据载体的接口文件，

```
int 64 num1
int 64 num2
```

### 1-3-2 srv文件（服务）

```
int64 num1

---

int64 sum
```

文件中声明的数据被 --- 分割为两部分，上半部分用于声明请求数据，下半部分用于声明响应数据

### 1-3-3 action文件（动作）

```
int64 num1

---

int64 num2

---

float54 progress


```

文件中声明的数据被 --- 分割为三部分，上半部分用于声明请求数据，中间部分用于声明响应数据，
下半部分用于声明连续反馈数据

### 1-3-4 补充

参数通信的数据无需定义接口文件，参数通信时数据会被封装为参数对象，参数客户端和服务端操作的都是参数对象。

# 二、准备工作

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

## 2-2 用得到的c++中的新特性

### 2-2-1 自动类型推导（auto）

``auto``:自动类型推导

在C++中，`auto` 关键字用于自动推断变量的类型。它允许编译器在编译时确定变量的类型，而不是在编写代码时显式声明。这使得代码更加简洁，并且可以减少一些类型声明的错误。

### 2-2-2 智能指针

在C++中，智能指针是一种自动管理动态分配（使用 `new` 关键字）内存的类模板。它们帮助防止内存泄漏，因为它们会在不再需要时自动释放分配的内存。`std::shared_ptr` 是C++11标准库中引入的一种智能指针，它允许多个指针实例共享对同一个对象的所有权。

`std::shared_ptr` 的特点：

1. **引用计数**：`std::shared_ptr` 使用引用计数机制来跟踪有多少个 `shared_ptr` 实例共享同一个对象。当最后一个引用被销毁或重置时，对象会被自动删除。
2. **复制语义**：当复制 `std::shared_ptr` 时，引用计数会增加，当 `std::shared_ptr` 被销毁或重新赋值时，引用计数会减少。
3. **线程安全**：对引用计数的修改是原子操作，这意味着 `std::shared_ptr` 在多线程环境中是安全的。
4. **异常安全**：`std::shared_ptr` 保证在抛出异常时不会泄漏内存。
5. **自定义删除器**：可以提供一个自定义的删除器，当管理的对象被销毁时，这个删除器会被调用。

# 三、案例复现(猛狮集训营)

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







## 3-4 参数服务

### 3-4-1 概念

在一个节点下保存数据，其他节点可以访问并操作这些数据。

**概念**

参数服务是以共享的方式实现不同节点之间数据交互的一种通信模式。保存参数的节点称之为参数服务端，调用参数的节点称之为参数客户端。参数客户端与参数服务端的交互是基于请求响应的，且参数通信的实现本质是对服务通信的进一步封装。 

**作用**

类似于编程中**全局变量**的概念，可以在不停节点之间共享数据。

**参数数据类型**

参数由键、值和描述符三部分组成，其中键是字符串类型，值可以是 bool、int64、float64、string、byte[]、bool[]、int64[]、float64[]、string[]中的任一类型，描述符默认情况下为空，但是可以设置参数描述、参数数据类型、取值范围或其他约束等信息。

### 3-4-2 案例实现（增、查、改、删）

**服务端**

在src目录下创建功能包

```bash
ros2 pkg create cpp04_param --build-type ament_cmake --dependencies rclcpp --node-name demo00_param
```

实例：

**增加：**

```cpp
// 需求 创建参数服务端并操作参数
// 3-1 增
// 3-2 查
// 3-3 改
// 3-4 删
// 1.包含头文件
#include "rclcpp/rclcpp.hpp"

// 3.自定义节点类；
class Mynode : public rclcpp::Node
{
public:
    // 如果允许删除参数，需要通过 NodeOptions 声明
    Mynode() : Node("mynode_node_cpp", rclcpp::NodeOptions().allow_undeclared_parameters(true))
    {
        RCLCPP_INFO(this->get_logger(), "参数服务端创建");
        // 一个普通的节点就可以作为一个参数服务端存在
    }
    // 3-1 增
    void declare()
    {
        RCLCPP_INFO(this->get_logger(), "----------增----------");
        this->declare_parameter("name","tigger");
        this->declare_parameter("width",155);
        this->declare_parameter("wheels",5);
    }
    // 3-2 查
    void get()
    {
        RCLCPP_INFO(this->get_logger(), "----------查----------");
    }
    // 3-3 改
    void updata()
    {
        RCLCPP_INFO(this->get_logger(), "----------改----------");
    }
    // 3-4 删
    void del()
    {
        RCLCPP_INFO(this->get_logger(), "----------删----------");
    }
};

int main(int argc, char const *argv[])
{
    // 2.初始化ROS2客户端；
    rclcpp::init(argc, argv);
    // 4.调用spain函数，并传入节点对象指针；
    auto node = std::make_shared<Mynode>();
    node->declare();
    node->get();
    node->updata();
    node->del();
    rclcpp::spin(node);
    // 5.资源释放
    rclcpp::shutdown();
    return 0;
}
```

查看是否创建

```bash
ros2 param list
```

查看具体值

```bash
ros2 param get /mynode_node_cpp name
```



![img](https://gitee.com/zhangchenxuv/images/raw/main/image-20240922200739498.png)





**查找：**

**使用 `get_parameter` 方法**：

这是获取单个参数值的方法。你可以指定参数的名称，它将返回该参数的值（如果存在）。

```cpp
rclcpp::Parameter param = this->get_parameter("param_name");
```

**使用 `get_parameter_or` 方法**：

允许你提供一个默认值。如果指定的参数不存在，它将返回默认值。

```cpp
rclcpp::Parameter param = this->get_parameter_or("param_name", rclcpp::Parameter("default_value"));
```

**使用 `get_parameters` 方法**：

这是获取多个参数的方法。你可以提供一个参数名称的列表，它将返回这些参数的值。

```cpp
std::vector<rclcpp::Parameter> params = this->get_parameters({"param1", "param2"});
```

**使用 `get_parameter_names` 方法**：

如果你想获取节点的所有参数名称，可以使用这个方法。

```cpp
std::vector<std::string> param_names = this->get_parameter_names();
```

**使用 `get_node_parameters_interface` 方法**：

需要更高级的参数接口，可以使用这个方法获取 `NodeParametersInterface`。

```cpp
auto params_interface = this->get_node_parameters_interface();
```



```cpp
// 需求 创建参数服务端并操作参数
// 3-1 增
// 3-2 查
// 3-3 改
// 3-4 删
// 1.包含头文件
#include "rclcpp/rclcpp.hpp"

// 3.自定义节点类；
class Mynode : public rclcpp::Node
{
public:
    // 如果允许删除参数，需要通过 NodeOptions 声明
    Mynode() : Node("mynode_node_cpp", rclcpp::NodeOptions().allow_undeclared_parameters(true))
    {
        RCLCPP_INFO(this->get_logger(), "参数服务端创建");
        // 一个普通的节点就可以作为一个参数服务端存在
    }
    // 3-1 增
    void declare()
    {
        RCLCPP_INFO(this->get_logger(), "----------增----------");
        this->declare_parameter("name_my", "tigger");
        this->declare_parameter("width", 155);
        this->declare_parameter("wheels", 5);
    }
    // 3-2 查
    void get()
    {
        RCLCPP_INFO(this->get_logger(), "----------查----------");
        // 获取指定参数
        auto na = this->get_parameter("name_my");
        RCLCPP_INFO(this->get_logger(), "key = %s , value = %s", na.get_name().c_str(), na.as_string().c_str());
        // 获取一些参数
        auto params = this->get_parameters({"name_my", "width", "wheels"});
        for (auto &&param : params)
        {
            RCLCPP_INFO(this->get_logger(), "(%s = %s)", param.get_name().c_str(), param.value_to_string().c_str());
        }

        // 判断是否包含
        RCLCPP_INFO(this->get_logger(),"是否包含name_my %d",this->has_parameter("name_my"));
        RCLCPP_INFO(this->get_logger(),"是否包含name_my %d",this->has_parameter("name_you"));
    }
    // 3-3 改
    void updata()
    {
        RCLCPP_INFO(this->get_logger(), "----------改----------");
    }
    // 3-4 删
    void del()
    {
        RCLCPP_INFO(this->get_logger(), "----------删----------");
    }
};

int main(int argc, char const *argv[])
{
    // 2.初始化ROS2客户端；
    rclcpp::init(argc, argv);
    // 4.调用spain函数，并传入节点对象指针；
    auto node = std::make_shared<Mynode>();
    node->declare();
    node->get();
    node->updata();
    node->del();
    rclcpp::spin(node);
    // 5.资源释放
    rclcpp::shutdown();
    return 0;
}
```

<img src="https://gitee.com/zhangchenxuv/images/raw/main/image/image-20240922203710421.png" alt="image-20240922203710421" style="zoom:67%;" />



**修改**

``set_parameter``同样可以增加参数，但依赖于``rclcpp::NodeOptions().allow_undeclared_parameters(true)``

```cpp
// 需求 创建参数服务端并操作参数
// 3-1 增
// 3-2 查
// 3-3 改
// 3-4 删
// 1.包含头文件
#include "rclcpp/rclcpp.hpp"

// 3.自定义节点类；
class Mynode : public rclcpp::Node
{
public:
    // 如果允许删除参数，需要通过 NodeOptions 声明
    Mynode() : Node("mynode_node_cpp", rclcpp::NodeOptions().allow_undeclared_parameters(true))
    {
        RCLCPP_INFO(this->get_logger(), "参数服务端创建");
        // 一个普通的节点就可以作为一个参数服务端存在
    }
    // 3-1 增
    void declare()
    {
        RCLCPP_INFO(this->get_logger(), "----------增----------");
        this->declare_parameter("name_my", "tigger");
        this->declare_parameter("width", 155);
        this->declare_parameter("wheels", 5);
    }
    // 3-2 查
    void get()
    {
        RCLCPP_INFO(this->get_logger(), "----------查----------");
        // 获取指定参数
        auto na = this->get_parameter("name_my");
        RCLCPP_INFO(this->get_logger(), "key = %s , value = %s", na.get_name().c_str(), na.as_string().c_str());
        // 获取一些参数
        auto params = this->get_parameters({"name_my", "width", "wheels"});
        for (auto &&param : params)
        {
            RCLCPP_INFO(this->get_logger(), "(%s = %s)", param.get_name().c_str(), param.value_to_string().c_str());
        }

        // 判断是否包含
        RCLCPP_INFO(this->get_logger(), "是否包含name_my %d", this->has_parameter("name_my"));
        RCLCPP_INFO(this->get_logger(), "是否包含name_my %d", this->has_parameter("name_you"));
    }
    // 3-3 改
    void updata()
    {
        RCLCPP_INFO(this->get_logger(), "----------改----------");
        this->set_parameter(rclcpp::Parameter("width", 175));
        // 修改功能
        RCLCPP_INFO(this->get_logger(), "修改后的值 %ld", this->get_parameter("width").as_int());
        // 设置新的参数 依赖于“ rclcpp::NodeOptions().allow_undeclared_parameters(true)”
        this->set_parameter(rclcpp::Parameter("height", 2000));
        RCLCPP_INFO(this->get_logger(), "增加的值 %ld", this->get_parameter("height").as_int());
    }
    // 3-4 删
    void del()
    {
        RCLCPP_INFO(this->get_logger(), "----------删----------");
    }
};

int main(int argc, char const *argv[])
{
    // 2.初始化ROS2客户端；
    rclcpp::init(argc, argv);
    // 4.调用spain函数，并传入节点对象指针；
    auto node = std::make_shared<Mynode>();
    node->declare();
    node->get();
    node->updata();
    node->del();
    rclcpp::spin(node);
    // 5.资源释放
    rclcpp::shutdown();
    return 0;
}
```

<img src="https://gitee.com/zhangchenxuv/images/raw/main/image/image-20240922212027117.png" alt="image-20240922212027117" style="zoom:67%;" />



**删除**

直接使用``this->undeclare_parameter("name_my");``会抛出异常

<img src="https://gitee.com/zhangchenxuv/images/raw/main/image/image-20240922212211205.png" alt="image-20240922212211205" style="zoom:67%;" />

但如果参数是通过``set_parameter``设置的参数，则可以删除

<img src="https://gitee.com/zhangchenxuv/images/raw/main/image/image-20240922212509090.png" alt="image-20240922212509090" style="zoom:67%;" />



```cpp
// 需求 创建参数服务端并操作参数
// 3-1 增
// 3-2 查
// 3-3 改
// 3-4 删
// 1.包含头文件
#include "rclcpp/rclcpp.hpp"

// 3.自定义节点类；
class Mynode : public rclcpp::Node
{
public:
    // 如果允许删除参数，需要通过 NodeOptions 声明
    Mynode() : Node("mynode_node_cpp", rclcpp::NodeOptions().allow_undeclared_parameters(true))
    {
        RCLCPP_INFO(this->get_logger(), "参数服务端创建");
        // 一个普通的节点就可以作为一个参数服务端存在
    }
    // 3-1 增
    void declare()
    {
        RCLCPP_INFO(this->get_logger(), "----------增----------");
        this->declare_parameter("name_my", "tigger");
        this->declare_parameter("width", 155);
        this->declare_parameter("wheels", 5);
    }
    // 3-2 查
    void get()
    {
        RCLCPP_INFO(this->get_logger(), "----------查----------");
        // 获取指定参数
        auto na = this->get_parameter("name_my");
        RCLCPP_INFO(this->get_logger(), "key = %s , value = %s", na.get_name().c_str(), na.as_string().c_str());
        // 获取一些参数
        auto params = this->get_parameters({"name_my", "width", "wheels"});
        for (auto &&param : params)
        {
            RCLCPP_INFO(this->get_logger(), "(%s = %s)", param.get_name().c_str(), param.value_to_string().c_str());
        }

        // 判断是否包含
        RCLCPP_INFO(this->get_logger(), "是否包含name_my %d", this->has_parameter("name_my"));
        RCLCPP_INFO(this->get_logger(), "是否包含name_my %d", this->has_parameter("name_you"));
    }
    // 3-3 改
    void updata()
    {
        RCLCPP_INFO(this->get_logger(), "----------改----------");
        this->set_parameter(rclcpp::Parameter("width", 175));
        // 修改功能
        RCLCPP_INFO(this->get_logger(), "修改后的值 %ld", this->get_parameter("width").as_int());
        // 设置新的参数 依赖于“ rclcpp::NodeOptions().allow_undeclared_parameters(true)”
        this->set_parameter(rclcpp::Parameter("height", 2000));
        RCLCPP_INFO(this->get_logger(), "增加的值 %ld", this->get_parameter("height").as_int());
    }
    // 3-4 删
    void del()
    {
        RCLCPP_INFO(this->get_logger(), "----------删----------");
        this->undeclare_parameter("height"); // height 是通过 set_parameter 设置的参数
        RCLCPP_INFO(this->get_logger(), "删除操作后是否包含height %d", this->has_parameter("height"));
    }
};

int main(int argc, char const *argv[])
{
    // 2.初始化ROS2客户端；
    rclcpp::init(argc, argv);
    // 4.调用spain函数，并传入节点对象指针；
    auto node = std::make_shared<Mynode>();
    node->declare();
    node->get();
    node->updata();
    node->del();
    rclcpp::spin(node);
    // 5.资源释放
    rclcpp::shutdown();
    return 0;
}
```



**客户端**





















## 3-5 坐标变换

### 3-5-1 概念

tf(TransForm Frame)是指坐标变换，它允许用户随时间跟踪多个坐标系。 它在时间缓冲的树结构中维护坐标帧之间的关系，并让用户在任何所需的时间点在任意两个坐标帧之间变换点、向量等。

作用：

​	实现不同坐标系之间的点或者向量的转换

实现流程：

​	1、组成结构：广播方+监听方；

​	2、广播方负责发布相对关系数据，一般一个广播方只负责发布一组数据；

​	3、监听方负责订阅（多个）广播方发布的数据，并将数据组织成**坐标树**，进而实现任意坐标帧之间点或向量的变换。

坐标变换库分为``tf``和升级后的``tf2``两个库，建议使用``tf2``库

<img src="https://gitee.com/zhangchenxuv/images/raw/main/image/image-20240913223305742.png" alt="image-20240913223305742" style="zoom:50%;" />

注意：坐标变换时，需要参考消息数据中的时间戳。需要保证参与变换的两个坐标帧的时间差在一定范围内，否则可能会导致误差偏大。

​           坐标变换基于右手坐标系。



安装依赖

```shell
sudo apt-get install ros-humble-turtle-tf2-py ros-humble-tf2-tools ros-humble-tf-transformations 
```

安装``TRANSFORMS3D``的Python包，为``tf_transformations``提供四元数和欧拉角变换功能

```shell
pip3 install transforms3d
```



### 3-5-2 坐标相关消息

常用的两种接口消息：``geometry_msgs/msg/TransformStamped`` 和 ``geometry_msgs/msg/PointStamped``

前者用于描述某一时刻两个坐标系之间相对关系的接口，后者用于描述某一时刻坐标系内某个坐标点的位置。

#### TransformStamped&PointStamped 

可以使用如下命令查看接口消息

```shell
ros2 interface show geometry_msgs/msg/TransformStamped
```

<img src="https://gitee.com/zhangchenxuv/images/raw/main/image/image-20240913225404986.png" alt="image-20240913225404986" style="zoom:67%;" />

```shell
ros2 interface show geometry_msgs/msg/PointStamped
```



<img src="https://gitee.com/zhangchenxuv/images/raw/main/image/image-20240913231140030.png" alt="image-20240913231140030" style="zoom:67%;" />



### 3-5-3 坐标变换广播

坐标系相对关系主要有两种：**静态坐标系相对关系**与**动态坐标系相对关系**

静态坐标系相对关系是指两个坐标系之间的相对位置是固定不变的

动态坐标系相对关系是指两个坐标系之间的相对位置关系是动态改变的



### 3-5-4 坐标变换广播案例

#### 静态广播器

**准备工作**

创建功能包

```shell
ros2 pkg create cpp03_tf_broadcaster --build-type ament_cmake --dependencies rclcpp tf2 tf2_ros geometry_msgs turtlesim --node-name demo01_static_tf_bro
```

关于静态广播器的官方说明

```shell
ros2 run tf2_ros static_transform_publisher
```

```shell
# 需要声明两个坐标系
required arguments:
  --frame-id FRAME_ID parent frame
  --child-frame-id CHILD_FRAME_ID child frame id
# 两个坐标系之间的关系
optional arguments:
  --x X                 x component of translation
  --y Y                 y component of translation
  --z Z                 z component of translation
  --qx QX               x component of quaternion rotation
  --qy QY               y component of quaternion rotation
  --qz QZ               z component of quaternion rotation
  --qw QW               w component of quaternion rotation
  --roll ROLL           roll component Euler rotation
  --pitch PITCH         pitch component Euler rotation
  --yaw YAW             yaw component Euler rotation
[ros2run]: Process exited with failure 1
# 时间戳不用控制
```

声明两个坐标系

```shell
ros2 run tf2_ros static_transform_publisher --frame-id base_link --child-frame-id laser
```

打开rviz2,添加插件

```shell
zhangchenxu@chengxz-pc:~$ export QT_ENABLE_HIGHDPI_SCALING=0 
zhangchenxu@chengxz-pc:~$ rviz2
```

<img src="https://gitee.com/zhangchenxuv/images/raw/main/image/image-20240914223232789.png" alt="image-20240914223232789" style="zoom: 50%;" />

设置一个偏移量

```shell
ros2 run tf2_ros static_transform_publisher --frame-id base_link --child-frame-id laser --x 1.0
```



<img src="https://gitee.com/zhangchenxuv/images/raw/main/image/image-20240914223454434.png" alt="image-20240914223454434" style="zoom:50%;" />

**静态广播器实现**

接下来用cpp实现以上功能

```cpp linenums="1"
// 需求：执行时传入两个坐标系的相对位子关系以及父子级坐标id，程序运行发布静态坐标变换

// ros2 run 包 可执行程序 x y z roll pitch yaw frame child_frame

//3.自定义节点类；
// 创建广播对象 组织并发布对象

//1.包含头文件
#include "rclcpp/rclcpp.hpp"

//3.自定义节点类；
class Mynode: public rclcpp::Node{
public:
    Mynode(char * argv[]):Node("mynode_node_cpp"){
        // 创建广播对象

        // 组织并发布数据

    }
};

int main(int argc, char *argv[])
{
    // 对输入的参数进行判断
    if (argc != 9)
    {
        RCLCPP_ERROR(rclcpp::get_logger("rclcpp"),("传入的参数不合法"));
        return 1;
    }
    
    
    //2.初始化ROS2客户端；
    rclcpp::init(argc,argv);
    //4.调用spain函数，并传入节点对象指针；
    rclcpp::spin(std::make_shared<Mynode>(argv));
    //5.资源释放
    rclcpp::shutdown();
    return 0;
}
```

组织并发布的数据来自于

``22 int main(int argc, char *argv[])``中的``argv[]``，``argv[]``已经封装好了，需要将其传入

`` 14 Mynode(char * argv[]):Node("mynode_node_cpp"){ ``中

同时在创建自定义节点对象指针时也需要将``argv``传入  `` 35 rclcpp::spin(std::make_shared<Mynode>(argv));``

```cpp
// 需求：执行时传入两个坐标系的相对位子关系以及父子级坐标id，程序运行发布静态坐标变换

// ros2 run 包 可执行程序 x y z roll pitch yaw frame child_frame

// 3.自定义节点类；
//  创建广播对象 组织并发布对象

// 1.包含头文件
#include "rclcpp/rclcpp.hpp"
#include "tf2_ros/static_transform_broadcaster.h"
#include "geometry_msgs/msg/transform_stamped.hpp"
#include "tf2/LinearMath/Quaternion.h"

// 3.自定义节点类；
class Mynode : public rclcpp::Node
{
public:
    Mynode(char *argv[]) : Node("mynode_node_cpp")
    {
        // 创建广播对象 #include "tf2_ros/static_transform_broadcaster.h"
        // tf2_ros::StaticTransformBroadcaster
        broadcaster_ = std::make_shared<tf2_ros::StaticTransformBroadcaster>(this);
        // 组织并发布数据
        pub_st_tf(argv);
    }

private:
    // 将对象声明成成员变量
    std::shared_ptr<tf2_ros::StaticTransformBroadcaster> broadcaster_;
    void pub_st_tf(char *argv[])
    {
        // 组织消息 #include "geometry_msgs/msg/transform_stamped.hpp"
        geometry_msgs::msg::TransformStamped transform;
        transform.header.stamp = this->now(); // 时间戳
        transform.header.frame_id = argv[7];  // 父级坐标系
        transform.child_frame_id = argv[8];   // 子级坐标系

        transform.transform.translation.x = atof(argv[1]); // x的变换 设置偏移量
        transform.transform.translation.y = atof(argv[2]); // y的变换
        transform.transform.translation.z = atof(argv[3]); // z的变换

        tf2::Quaternion qtn; // 将欧拉角转换为四元数 #include "tf2/LinearMath/Quaternion.h"
        qtn.setRPY(atof(argv[4]),atof(argv[5]),atof(argv[6]));
        transform.transform.rotation.x = qtn.x();
        transform.transform.rotation.y = qtn.y();
        transform.transform.rotation.z = qtn.z();
        transform.transform.rotation.w = qtn.w();

        // 发布
        broadcaster_->sendTransform(transform);
    }
};

int main(int argc, char *argv[])
{
    // 对输入的参数进行判断
    if (argc != 9)
    {
        RCLCPP_ERROR(rclcpp::get_logger("rclcpp"), ("传入的参数不合法"));
        return 1;
    }

    // 2.初始化ROS2客户端；
    rclcpp::init(argc, argv);
    // 4.调用spain函数，并传入节点对象指针；
    rclcpp::spin(std::make_shared<Mynode>(argv));
    // 5.资源释放
    rclcpp::shutdown();
    return 0;
}
```

编译后安装环境，并执行

```shell
ros2 run cpp03_tf_broadcaster demo01_static_tf_bro 0.4 0.0 0.2 0 0 0 base_link laser
```

打开rviz2

<img src="https://gitee.com/zhangchenxuv/images/raw/main/image/image-20240915000418494.png" alt="image-20240915000418494" style="zoom:50%;" />

#### 动态广播器

```cpp
// 启动 turlesim_node 节点，编写程序，发布乌龟相对于窗体的位姿

// 3.自定义节点类；
//  创建动态广播器
//  创建一个乌龟位姿订阅方
//  回调函数中获取乌龟位姿,并生成相对关系并发布

// 1.包含头文件
#include "rclcpp/rclcpp.hpp"
#include "tf2_ros/transform_broadcaster.h"
#include "turtlesim/msg/pose.hpp"
#include "geometry_msgs/msg/transform_stamped.hpp"
#include "tf2/LinearMath/Quaternion.h"

// 3.自定义节点类；
class Mynode : public rclcpp::Node
{
public:
    Mynode() : Node("mynode_node_cpp")
    {
        // 创建动态广播器  #include "tf2_ros/transform_broadcaster.h"
        broadcaster_ = std::make_shared<tf2_ros::TransformBroadcaster>(this);
        // 创建乌龟订阅方
        pose_sub_ = this->create_subscription<turtlesim::msg::Pose>("/turtle1/pose", 10,
                                                                    std::bind(&Mynode::do_pose, this, std::placeholders::_1));
    }

private:
    std::shared_ptr<tf2_ros::TransformBroadcaster> broadcaster_;
    rclcpp::Subscription<turtlesim::msg::Pose>::SharedPtr pose_sub_;
    // 回调函数中，获取乌龟位姿并生成相对关系然后发布
    void do_pose(const turtlesim::msg::Pose &pose)
    {
        // 组织消息
        geometry_msgs::msg::TransformStamped ts;
        ts.header.stamp = this->now(); // 时间戳
        ts.header.frame_id = "world";  // 父级坐标系
        ts.child_frame_id = "turtle1"; // 子级坐标系

        ts.transform.translation.x = pose.x;
        ts.transform.translation.y = pose.y;
        ts.transform.translation.z = 0.0;

        tf2::Quaternion qtn; // 将欧拉角转换为四元数 #include "tf2/LinearMath/Quaternion.h"
        qtn.setRPY(0, 0, pose.theta);

        ts.transform.rotation.x = qtn.x();
        ts.transform.rotation.y = qtn.y();
        ts.transform.rotation.z = qtn.z();
        ts.transform.rotation.w = qtn.w();
        // 发布
        broadcaster_->sendTransform(ts);
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



![image-20240915173858681](https://gitee.com/zhangchenxuv/images/raw/main/image/image-20240915173858681.png)



发布坐标点

```cpp
// 发布相对于laser坐标系的坐标点数据
// 创建发布方
// 创建定时器
// 组织并发布消息

// 1.包含头文件
#include "rclcpp/rclcpp.hpp"
#include "geometry_msgs/msg/point_stamped.hpp"

using namespace std::chrono_literals;

// 3.自定义节点类；
class Mynode : public rclcpp::Node
{
public:
    Mynode() : Node("mynode_node_cpp"), x(0.0)
    {
        // 创建发布方
        point_pub = this->create_publisher<geometry_msgs::msg::PointStamped>("point", 10); // 话题名称 qos
        // 创建定时器
        timer_ = this->create_wall_timer(1s, std::bind(&Mynode::on_time, this));
        // 组织并发布消息
    }

private:
    rclcpp::Publisher<geometry_msgs::msg::PointStamped>::SharedPtr point_pub;
    rclcpp::TimerBase::SharedPtr timer_;
    double_t x;

    void on_time()
    {
        // 组织消息
        geometry_msgs::msg::PointStamped ps;
        ps.header.stamp = this->now();
        ps.header.frame_id = "laser";
        x = x + 0.05;
        ps.point.x = x;
        ps.point.y = 0.0;
        ps.point.z = -0.1;
        // 发布消息
        point_pub->publish(ps);
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

安装和启动节点

```bash
zhangchenxu@chengxz-pc:~/Documents/ws_learn$ . install/setup.bash 
zhangchenxu@chengxz-pc:~/Documents/ws_learn$ ros2 run cpp03_tf_broadcaster demo03_point_tf_bro 
^C[INFO] [1726415434.302317168] [rclcpp]: signal_handler(signum=2)
```

创建坐标系

```bash
zhangchenxu@chengxz-pc:~/Documents/ws_learn$ ros2 run tf2_ros static_transform_publisher --frame-id base_link --child-frame-id laser --x 0.2 --z 0.1
```

开启rviz2

<img src="https://gitee.com/zhangchenxuv/images/raw/main/image/image-20240915235346285.png" alt="image-20240915235346285" style="zoom:67%;" />



### 3-5-5 坐标变换监听

#### 坐标系变换

```cpp
// 先发布laser到base_link，再发布camera到base_link的坐标关系，求解laser到camera的坐标相对关系

// 3-1.创建缓存对象，融合多个坐标系相对关系为一颗坐标树
// 3-2.创建监听器，绑定缓存对象，会将所有广播器广播的数据写入缓存
// 3-3.编写定时器，循环实现转换
// 1.包含头文件
#include "rclcpp/rclcpp.hpp"
#include "tf2_ros/buffer.h"
#include "tf2_ros/transform_listener.h"

using namespace std::chrono_literals;

// 3.自定义节点类；
class Mynode : public rclcpp::Node
{
public:
    Mynode() : Node("mynode_node_cpp")
    {
        // 3-1.创建缓存对象，融合多个坐标系相对关系为一颗坐标树 #include "tf2_ros/buffer.h"
        buffer_ = std::make_unique<tf2_ros::Buffer>(this->get_clock());
        // 3-2.创建监听器，绑定缓存对象，会将所有广播器广播的数据写入缓存
        listener_ = std::make_shared<tf2_ros::TransformListener>(*buffer_, this);
        // 3-3.编写定时器，循环实现转换
        timer_ = this->create_wall_timer(1s, std::bind(&Mynode::on_timer, this));
    }

private:
    std::unique_ptr<tf2_ros::Buffer> buffer_;
    std::shared_ptr<tf2_ros::TransformListener> listener_;
    rclcpp::TimerBase::SharedPtr timer_;
    void on_timer()
    // 实现坐标系转换

    {
        try
        {
            auto ts = buffer_->lookupTransform("camera", "laser", tf2::TimePointZero); // tf2::TimePointZero转换最新时刻的时间帧
            RCLCPP_INFO(this->get_logger(), "---完成转换的坐标信息---");
            RCLCPP_INFO(this->get_logger(),
                        "新坐标系：父：%s,子：%s,偏移量（%.2f,%.2f,%.2f)",
                        ts.header.frame_id.c_str(), // camera
                        ts.child_frame_id.c_str(),  // laser
                        ts.transform.translation.x,
                        ts.transform.translation.y,
                        ts.transform.translation.z);
        }
        catch (const tf2::LookupException &e)
        {
            RCLCPP_INFO(this->get_logger(), "异常提示：%s", e.what());
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

建立父级坐标系 base_link和子坐标系laser关系

```bash
ros2 run tf2_ros static_transform_publisher --frame-id base_link --child-frame-id laser --x 0.4 --z 0.2
```

建立父级坐标系 base_link和子坐标系camera关系

```bash
ros2 run tf2_ros static_transform_publisher --frame-id base_link --child-frame-id camera --x -0.5 --z 0.4
```

开启转换

```bash
ros2 run cpp04_tf_listener demo01_tf_listener
```

<img src="https://gitee.com/zhangchenxuv/images/raw/main/image/image-20240916174738400.png" alt="image-20240916174738400" style="zoom:67%;" />



#### 坐标点变换

<img src="https://gitee.com/zhangchenxuv/images/raw/main/image/image-20240916195911792.png" alt="image-20240916195911792" style="zoom:67%;" />

```cpp
// 需求：laser到base_link的坐标系相对关系，然后发布point到laser的坐标，求解point到base_link的坐标
// 3-1.创建坐标变换监听器
// 3-2.创建坐标点消息订阅方
// 3-3.创建过滤器，解析数据
// 1.包含头文件
#include "rclcpp/rclcpp.hpp"
#include "tf2_ros/buffer.h"
#include "tf2_ros/transform_listener.h"
#include "tf2_ros/create_timer_ros.h"
#include "message_filters/subscriber.h"
#include "geometry_msgs/msg/point_stamped.hpp"
#include "tf2_ros/message_filter.h"
#include "tf2_geometry_msgs/tf2_geometry_msgs.hpp"

using namespace std::chrono_literals;

// 3.自定义节点类；
class Mynode : public rclcpp::Node
{
public:
    Mynode() : Node("mynode_node_cpp")
    {
        // 3-1.创建坐标变换监听器
        buffer_ = std::make_shared<tf2_ros::Buffer>(this->get_clock());
        timer_ = std::make_shared<tf2_ros::CreateTimerROS>(
            this->get_node_base_interface(),
            this->get_node_timers_interface());
        buffer_->setCreateTimerInterface(timer_);
        listener_ = std::make_shared<tf2_ros::TransformListener>(*buffer_);
        // 3-2.创建坐标点消息订阅方
        point_sub.subscribe(this, "point");
        // 3-3.创建过滤器，解析数据
        filter_ = std::make_shared<tf2_ros::MessageFilter<geometry_msgs::msg::PointStamped>>(
            point_sub,
            *buffer_,
            "base_link",
            10,
            this->get_node_logging_interface(),
            this->get_node_clock_interface(),
            2s);
        // 解析数据
        filter_->registerCallback(&Mynode::transform_point, this);
    }

private:
    // 创建缓存器
    std::shared_ptr<tf2_ros::Buffer> buffer_;
    // 创建监听器
    std::shared_ptr<tf2_ros::TransformListener> listener_;
    //
    std::shared_ptr<tf2_ros::CreateTimerROS> timer_;
    //
    message_filters::Subscriber<geometry_msgs::msg::PointStamped> point_sub;
    // 过滤器 geometry_msgs::msg::PointStamped 过滤的是坐标点
    std::shared_ptr<tf2_ros::MessageFilter<geometry_msgs::msg::PointStamped>> filter_;
    void transform_point(const geometry_msgs::msg::PointStamped &ps)
    {
        // 实现坐标点变换 被转化的点 基坐标系
        // 包含#include "tf2_geometry_msgs/tf2_geometry_msgs.hpp"
        auto out = buffer_->transform(ps, "base_link");
        RCLCPP_INFO(this->get_logger(),"父：%s,坐标：%.2f,%.2f,%.2f",
            out.header.frame_id.c_str(),
            out.point.x,
            out.point.y,
            out.point.z
        );
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

首先发布base_link以及laser的静态坐标

```bash
zhangchenxu@chengxz-pc:~$ ros2 run tf2_ros static_transform_publisher --frame-id base_link --child-frame-id laser --x 0.4 --z 0.2
```

由之前的项目发布一个随时间相对laser移动的点

```bash
zhangchenxu@chengxz-pc:~/Documents/ws_learn$ . install/setup.bash 
zhangchenxu@chengxz-pc:~/Documents/ws_learn$ ros2 run cpp03_tf_broadcaster demo03_point_tf_bro
```

运行坐标点变换的项目

```bash
zhangchenxu@chengxz-pc:~/Documents/ws_learn$ . install/setup.bash 
zhangchenxu@chengxz-pc:~/Documents/ws_learn$ ros2 run cpp04_tf_listener demo02_msg_filter 
```

可得到如下结果

![image-20240916220730083](https://gitee.com/zhangchenxuv/images/raw/main/image/image-20240916220730083.png)

















## 3-6 可视化

### 3-6-1 RVIZ启动

一般在安装桌面版ROS2时，RVIZ2一般已经安装

**启动方式：**终端输入``rviz``或``ros2 run rviz2 rviz2``

**可能遇到的问题：**启动时主界面闪烁

在终端启动rviz前首先设置系统变量 QT_ENABLE_HIGHDPI_SCALING=0 ，或者设置QT_SCREEN_SCALE_FACTORS=1

```shell
export QT_ENABLE_HIGHDPI_SCALING=0 
```

![image-20240821004633796](/home/zhangchenxu/Pictures/Typora/image-20240821004633796.png)

### 3-6-2 插件列表

<img src="/home/zhangchenxu/Pictures/Typora/image-20240821005918309.png" alt="image-20240821005918309" style="zoom:67%;" />

<img src="/home/zhangchenxu/Pictures/Typora/image-20240821005946473.png" alt="image-20240821005946473" style="zoom:67%;" />

### 3-6-3 rviz2集成URDF

创建功能包

```shell
ros2 pkg create cpp06_urdf --build-type ament_cmake
```

功能包下新建 urdf、rviz、launch、meshes 目录以备用，其中 urdf 目录下再新建子目录 urdf 与
xacro，分别用于存储 urdf 文件和 xacro 文件

#### |--案例实现

**1、编辑配置文件**

在package.xml文件中需要添加一些**执行时**的依赖：

```xml
<exec_depend>rviz2</exec_depend>
<exec_depend>xacro</exec_depend>
<exec_depend>robot_state_publisher</exec_depend><exec_depend>joint_state_publisher</exec_depend>
<exec_depend>ros2launch</exec_depend>
```

在CMakeLists.txt中需要为创建的urdf、rviz、launch、meshes 目录配置安装路径

```cmake
install(DIRECTORY launch rviz urdf meshes  # 安装launch rviz urdf meshes四个新建文件夹
  DESTINATION share/${PROJECT_NAME}         # 指定目录为 install中share下的与项目同名的
)
```

**2、URDF文件实现**

```xml
<!-- 创建长方体 -->

<robot name="demo01">
    <link name="base_link">
        <!-- 可视化 -->
        <visual>
            <!-- 几何形状 -->
            <geometry>
                <!-- box 长方体形状 -->
                <box size="1.0 0.5 0.1"/>
            </geometry>
        </visual>      
    </link>   
</robot>

```

单纯的urdf文件是无法启动的，需要使用launch文件

**3、launch文件**

在launch文件夹下新建``display.launch.py``文件

```py
from launch import LaunchDescription
from launch_ros.actions import Node
# ---|封装终端指令相关类
# from launch.actions import ExecuteProcess
# from launch.substitutions import FindExecutable
# ---|参数声明与获取
# from launch.actions import DeclareLaunchArgument
# from launch.substitutions import LaunchConfiguration
# ---|文件包含相关
# from launch.action import IncludeLaunchDescription
# from launch.launch_description_sources import PythonLaunchDescriptionSource
# ---|分组相关
# from launch_ros.actions import PushRosNamespace
# from launch.actions import GroupAction
# ---|事件相关
# from launch.event_handlers import OnProcessStart, OnProcessExit
# from launch.actions import ExecuteProcess, RegisterEventHandler,LogInfo
# ---|获取功能包下 share 目录路径
# from ament_index_python.packages import get_package_share_directory

from launch_ros.parameter_descriptions import ParameterValue
# 封装指令执行的库
from launch.substitutions import Command
# ---|获取功能包下 share 目录路径
from ament_index_python.packages import get_package_share_directory

from launch import LaunchDescription
from launch.actions import ExecuteProcess
from launch.substitutions import EnvironmentVariable

def generate_launch_description():
    """
        需求：加载URDF中的文件至RVIZ2中
        核心：
            1、启动robot_state_publisher节点，该节点要以参数的方式加载URDF文件内容
            2、启动RVIZ2节点
    """  
    # 1、启动robot_state_publisher节点，该节点要以参数的方式加载URDF文件内容
    p_value = ParameterValue(Command(["xacro ",get_package_share_directory("cpp06_urdf") + "/urdf/urdf/demo01.urdf"]))
    robot_state_pub = Node(
        package="robot_state_publisher",
        executable="robot_state_publisher",
        parameters=[{"robot_description":p_value}]
    )
    # 2、启动RVIZ2节点
    rviz2 = Node(package="rviz2",executable="rviz2") 
    return LaunchDescription([robot_state_pub,
                              rviz2])
```

其中``parameters=[{"robot_description":p_value}]``处，``parameters``实际就是urdf中的参数，但如果直接在此处则移植性变差

此处通过``from launch.substitutions import Command``使launch文件能够执行指令

使用``from ament_index_python.packages import get_package_share_directory``使可以直接访问share下的目录

![image-20241022181405826](https://gitee.com/zhangchenxuv/images/raw/main/image/image-20241022181405826.png)

之后编译并启动

```shell
colcon build
. install/setup.bash
export QT_ENABLE_HIGHDPI_SCALING=0 
ros2 launch cpp06_urdf display.launch.py 
```

<img src="https://gitee.com/zhangchenxuv/images/raw/main/image/image-20240903190313935.png" alt="image-20240903190313935" style="zoom: 25%;" />

增加插件后进行相应配置，可以展现相应的模型

#### |--优化（发布活动关节）

```py
from launch import LaunchDescription
from launch_ros.actions import Node
# ---|封装终端指令相关类
# from launch.actions import ExecuteProcess
# from launch.substitutions import FindExecutable
# ---|参数声明与获取
from launch.actions import DeclareLaunchArgument
from launch.substitutions import LaunchConfiguration
# ---|文件包含相关
# from launch.action import IncludeLaunchDescription
# from launch.launch_description_sources import PythonLaunchDescriptionSource
# ---|分组相关
# from launch_ros.actions import PushRosNamespace
# from launch.actions import GroupAction
# ---|事件相关
# from launch.event_handlers import OnProcessStart, OnProcessExit
# from launch.actions import ExecuteProcess, RegisterEventHandler,LogInfo
# ---|获取功能包下 share 目录路径
# from ament_index_python.packages import get_package_share_directory

# ParameterValue，可视化操作需要
from launch_ros.parameter_descriptions import ParameterValue
# 封装指令执行的库
from launch.substitutions import Command
# ---|获取功能包下 share 目录路径
from ament_index_python.packages import get_package_share_directory

from launch import LaunchDescription
from launch.actions import ExecuteProcess
from launch.substitutions import EnvironmentVariable

def generate_launch_description():
    """
        需求：加载URDF中的文件至RVIZ2中
        核心：
            1、启动robot_state_publisher节点，该节点要以参数的方式加载URDF文件内容；
            2、启动RVIZ2节点；
        优化：
            1、添加 joint_state_publisher 节点，机器人涉及非固定关节，必须包含该节点；
            2、设置RVIZ2的默认配置文件；

    """  
    # 1、启动robot_state_publisher节点，该节点要以参数的方式加载URDF文件内容
    # p_value = ParameterValue(Command(["xacro ",get_package_share_directory("cpp06_urdf") + "/urdf/urdf/demo01.urdf"]))
    # 优化实现
    # 可以实现当添加了新的urdf文件后，在终端运行时可以修改需要加载的模型   
    # get_package_share_directory：可以定位到share目录
    model = DeclareLaunchArgument(name="model",default_value=get_package_share_directory("cpp06_urdf") + "/urdf/urdf/demo01.urdf") 
    p_value = ParameterValue(Command(["xacro ",LaunchConfiguration("model")]))
    
    # 1、启动robot_state_publisher节点，该节点要以参数的方式加载URDF文件内容
    robot_state_pub = Node(
        package="robot_state_publisher",
        executable="robot_state_publisher",
        parameters=[{"robot_description":p_value}]
    )
    # 优化1
    # 复杂机器人，非固定关节需要这个节点
    joint_state_pub = Node(
        package="joint_state_publisher",
        executable="joint_state_publisher"
    )
    # 2、启动RVIZ2节点
    rviz2 = Node(
        package="rviz2",
        executable="rviz2",
        # 配置文件目录
        arguments=["-d",get_package_share_directory("cpp06_urdf") + "/rviz/urdf.rviz"]

    )
    return LaunchDescription([model,
                              robot_state_pub,
                              joint_state_pub,
                              rviz2]
                            )
```



#### |--URDF使用语法

##### |--**robot**

urdf 中为了保证 xml 语法的完整性，使用了 robot 标签作为根标签



##### |--**link**

<img src="https://gitee.com/zhangchenxuv/images/raw/main/image/image-20240903224336210.png" alt="image-20240903224336210" style="zoom:25%;" />

属性：

name（必填）：为连杆命名。

子标签:

<visual>（可选）：用于描述 link 的可视化属性，可以设置 link 的形状（立方体、球体、圆柱等）。

<img src="https://gitee.com/zhangchenxuv/images/raw/main/image/image-20240903224730922.png" alt="image-20240903224730922" style="zoom:50%;" />

<collision> ：连杆的碰撞属性.

<img src="https://gitee.com/zhangchenxuv/images/raw/main/image/image-20240903224905097.png" alt="image-20240903224905097" style="zoom:50%;" />

<Inertial >:连杆的惯性矩阵

<img src="https://gitee.com/zhangchenxuv/images/raw/main/image/image-20240903224939356.png" alt="image-20240903224939356" style="zoom:50%;" />

语法示例：

![image-20241022233940395](https://gitee.com/zhangchenxuv/images/raw/main/image/image-20241022233940395.png)

![image-20241023000718160](https://gitee.com/zhangchenxuv/images/raw/main/image/image-20241023000718160.png)



##### |--**joint**

![image-20241023203803202](https://gitee.com/zhangchenxuv/images/raw/main/image/image-20241023203803202.png)



![image-20241023203821139](https://gitee.com/zhangchenxuv/images/raw/main/image/image-20241023203821139.png)

![image-20241023203840507](https://gitee.com/zhangchenxuv/images/raw/main/image/image-20241023203840507.png)



示例：

URDF:

```xml
<!-- 创建一个机器人模型，由底盘和摄像头组成，摄像头可沿着z轴旋转
     1、创建底盘相关的Link
     2、创建摄像头Link
     3、通过joint关联底盘与摄像头
-->
<robot name="demo_joint">
    <!-- 抽取整体颜色 -->
     <material name="yellow">
        <color rgba="0.8 0.8 0.0 0.5"/>       
     </material>
     <material name="red">
        <color rgba="0.9 0.0 0.0 0.5"/>       
     </material>
    <!-- 1、创建底盘相关link -->
     <link name="base_link">
        <visual>
            <geometry>
                <box size="0.5 0.3 0.1"/>
            </geometry>
            <material name="yellow"/>
        </visual>     
     </link>
     <!-- 2、创建摄像头link -->
      <link name="camera">
        <visual>
            <geometry>
            <box size="0.02 0.05 0.05"/>
            </geometry>
            <material name="red"/>
        </visual>
      </link>

      <!-- 3、关联底盘和摄像头 -->
       <joint name="camera2base_link" type="continuous">
            <parent link="base_link"/>
            <child link="camera"/>
            <!-- 设置平移量和旋转度 -->
            <origin xyz="0.2 0.0 0.075"/>
            <!-- 沿哪个坐标轴旋转 -->
            <axis xyz="0 0 1"/>
       </joint>
    
</robot>
```

Launch:

```python
from launch import LaunchDescription
from launch_ros.actions import Node
# ---|封装终端指令相关类
# from launch.actions import ExecuteProcess
# from launch.substitutions import FindExecutable
# ---|参数声明与获取
from launch.actions import DeclareLaunchArgument
from launch.substitutions import LaunchConfiguration
# ---|文件包含相关
# from launch.action import IncludeLaunchDescription
# from launch.launch_description_sources import PythonLaunchDescriptionSource
# ---|分组相关
# from launch_ros.actions import PushRosNamespace
# from launch.actions import GroupAction
# ---|事件相关
# from launch.event_handlers import OnProcessStart, OnProcessExit
# from launch.actions import ExecuteProcess, RegisterEventHandler,LogInfo
# ---|获取功能包下 share 目录路径
# from ament_index_python.packages import get_package_share_directory

# ParameterValue，可视化操作需要
from launch_ros.parameter_descriptions import ParameterValue
# 封装指令执行的库
from launch.substitutions import Command
# ---|获取功能包下 share 目录路径
from ament_index_python.packages import get_package_share_directory

from launch import LaunchDescription
from launch.actions import ExecuteProcess
from launch.substitutions import EnvironmentVariable

def generate_launch_description():
    """
        需求：加载URDF中的文件至RVIZ2中
        核心：
            1、启动robot_state_publisher节点，该节点要以参数的方式加载URDF文件内容；
            2、启动RVIZ2节点；
        优化：
            1、添加 joint_state_publisher 节点，机器人涉及非固定关节，必须包含该节点；
            2、设置RVIZ2的默认配置文件；

    """  
    # 1、启动robot_state_publisher节点，该节点要以参数的方式加载URDF文件内容
    # p_value = ParameterValue(Command(["xacro ",get_package_share_directory("cpp06_urdf") + "/urdf/urdf/demo01.urdf"]))
    # 优化实现
    # 可以实现当添加了新的urdf文件后，在终端运行时可以修改需要加载的模型   
    # get_package_share_directory：可以定位到share目录
    model = DeclareLaunchArgument(name="model",default_value="install/cpp06_urdf/share/cpp06_urdf/urdf/urdf/demo_joint.urdf") 
    p_value = ParameterValue(Command(["xacro ",LaunchConfiguration("model")]))
    
    # 1、启动robot_state_publisher节点，该节点要以参数的方式加载URDF文件内容
    robot_state_pub = Node(
        package="robot_state_publisher",
        executable="robot_state_publisher",
        parameters=[{"robot_description":p_value}]
    )
    # 优化1
    # 复杂机器人，非固定关节需要这个节点
    joint_state_pub = Node(
        package="joint_state_publisher",
        executable="joint_state_publisher"
    )
    # 2、启动RVIZ2节点
    rviz2 = Node(
        package="rviz2",
        executable="rviz2",
        # 配置文件目录
        # arguments=["-d","src/cpp06_urdf/urdf/urdf/demo_joint.urdf"]

    )
    return LaunchDescription([model,
                              robot_state_pub,
                              joint_state_pub,
                              rviz2]
                            )
```

编译之后通过launch文件启动

使用

```bash
ros2 run joint_state_publisher_gui joint_state_publisher_gui
```

开启一个UI窗口

<img src="https://gitee.com/zhangchenxuv/images/raw/main/image/image-20241023214704938.png" alt="image-20241023214704938" style="zoom: 33%;" />

拖动进度条可以实现关节旋转

##### |--joint_state_publisher的作用

1、``joint_state_publisher``与``joint_state_publisher_gui``作用一致，都会发布非固定关节的运动信息；

2、``robot_state_publisher``会订阅关节的运动信息，并生成坐标变换数据广播；

3、``joint_state_publisher``与``joint_state_publisher_gui``有一个存在时，就会发布关节运动信息，进而生成坐标变换，

​	当两个都不启动时，坐标树生成不了，机器人模型显示异常。

​	当两个都启动时，``joint_state_publisher``会一直发布初始关节信息``joint_state_publisher_gui``发布指定关节位姿信息，最终  	显示会产生抖动



##### |--示例：四轮差速小车

**URDF**文件：

```xml
<!-- 创建一个机器人模型，由底盘和摄像头组成，摄像头可沿着z轴旋转
     1、创建底盘相关的Link
     2、创建摄像头Link
     3、通过joint关联底盘与摄像头
-->
<robot name="demo_joint">
    <!-- 抽取整体颜色 -->
     <material name="yellow">
        <color rgba="0.8 0.8 0.0 0.5"/>       
     </material>
     <material name="red">
        <color rgba="0.9 0.0 0.0 0.5"/>       
     </material>
    <!-- 设置初始化Link -->
     <link name="base_footprint">
        <visual>
            <geometry>
                <geometry>
                    <sphere radius="0.001"/>
                </geometry>
            </geometry>
        </visual>
     </link>

    <!-- 1、创建底盘相关link -->
     <link name="base_link">
        <visual>
            <geometry>
                <box size="0.5 0.3 0.1"/>
            </geometry>
            <material name="yellow"/>
        </visual>     
     </link>

     <!-- 初始化Link通过关节连接base_link,设置关节偏移量，让base_link上移 -->
     <joint name="base_footprint2base_link" type="fixed">
        <parent link="base_footprint"/>
        <child link="base_link"/>
        <origin xyz="0.0 0.0 0.05"/>
     </joint>

     <!-- 2、创建摄像头link -->
      <link name="camera">
        <visual>
            <geometry>
            <box size="0.02 0.05 0.05"/>
            </geometry>
            <material name="red"/>
        </visual>
      </link>

      <!-- 3、关联底盘和摄像头 -->
       <joint name="camera2base_link" type="continuous">
            <parent link="base_link"/>
            <child link="camera"/>
            <!-- 设置平移量和旋转度 -->
            <origin xyz="0.2 0.0 0.075"/>
            <!-- 关节沿哪个坐标轴旋转 -->
            <axis xyz="0 0 1"/>
       </joint>
    
</robot>
```



**Launch**文件

```python
from launch import LaunchDescription
from launch_ros.actions import Node
# ---|封装终端指令相关类
# from launch.actions import ExecuteProcess
# from launch.substitutions import FindExecutable
# ---|参数声明与获取
from launch.actions import DeclareLaunchArgument
from launch.substitutions import LaunchConfiguration
# ---|文件包含相关
# from launch.action import IncludeLaunchDescription
# from launch.launch_description_sources import PythonLaunchDescriptionSource
# ---|分组相关
# from launch_ros.actions import PushRosNamespace
# from launch.actions import GroupAction
# ---|事件相关
# from launch.event_handlers import OnProcessStart, OnProcessExit
# from launch.actions import ExecuteProcess, RegisterEventHandler,LogInfo
# ---|获取功能包下 share 目录路径
# from ament_index_python.packages import get_package_share_directory

# ParameterValue，可视化操作需要
from launch_ros.parameter_descriptions import ParameterValue
# 封装指令执行的库
from launch.substitutions import Command
# ---|获取功能包下 share 目录路径
from ament_index_python.packages import get_package_share_directory

from launch import LaunchDescription
from launch.actions import ExecuteProcess
from launch.substitutions import EnvironmentVariable

def generate_launch_description():
    """
        需求：加载URDF中的文件至RVIZ2中
        核心：
            1、启动robot_state_publisher节点，该节点要以参数的方式加载URDF文件内容；
            2、启动RVIZ2节点；
        优化：
            1、添加 joint_state_publisher 节点，机器人涉及非固定关节，必须包含该节点；
            2、设置RVIZ2的默认配置文件；

    """  
    # 1、启动robot_state_publisher节点，该节点要以参数的方式加载URDF文件内容
    # p_value = ParameterValue(Command(["xacro ",get_package_share_directory("cpp06_urdf") + "/urdf/urdf/demo01.urdf"]))
    # 优化实现
    # 可以实现当添加了新的urdf文件后，在终端运行时可以修改需要加载的模型   
    # get_package_share_directory：可以定位到share目录
    model = DeclareLaunchArgument(name="model",default_value="install/cpp06_urdf/share/cpp06_urdf/urdf/urdf/exercise.urdf") 
    p_value = ParameterValue(Command(["xacro ",LaunchConfiguration("model")]))
    
    # 1、启动robot_state_publisher节点，该节点要以参数的方式加载URDF文件内容
    robot_state_pub = Node(
        package="robot_state_publisher",
        executable="robot_state_publisher",
        parameters=[{"robot_description":p_value}]
    )
    # 优化1
    # 复杂机器人，非固定关节需要这个节点
    joint_state_pub = Node(
        package="joint_state_publisher",
        executable="joint_state_publisher"
    )
    # 2、启动RVIZ2节点
    rviz2 = Node(
        package="rviz2",
        executable="rviz2",
        # 配置文件目录
        # arguments=["-d","src/cpp06_urdf/urdf/urdf/demo_joint.urdf"]

    )
    return LaunchDescription([model,
                              robot_state_pub,
                              joint_state_pub,
                              rviz2]
                            )
```



**效果图**

启用

```bash
ros2 run joint_state_publisher_gui joint_state_publisher_gui 
```



<img src="https://gitee.com/zhangchenxuv/images/raw/main/image/image-20241024001448279.png" alt="image-20241024001448279" style="zoom:50%;" />

**完整建模**

```xml
<!-- 
    需求：创建一个四轮差速机器人
    参数：长0.2m 宽0.12m 高0.07m 轮胎半径及宽度为0.025 0.02，离地间距0.15m
    步骤
    1、设置base_footprint;
    2、设置base_link;
    3、使用joint将二者关联；
    4、添加一个车轮link;
    5、将车轮与base_link关联；
    6、其他车轮实现
-->
<robot name="my_car">
    <!-- 0、抽取颜色 -->
    <material name="yellow">
        <color rgba="0.8 0.7 0.0 0.5" />
    </material>
    <material name="black">
        <color rgba="0.0 0.0 0.0 0.5" />
    </material>

    <!-- 1、设置base_footprint -->
    <link name="base_footprint">
        <visual>
            <geometry>
                <sphere radius="0.001" />
            </geometry>
        </visual>
    </link>
    <!-- 2、设置base_link -->
    <link name="base_link">
        <visual>
            <geometry>
                <box size="0.2 0.12 0.07" />
            </geometry>
            <!-- 颜色 -->
            <material name="yellow" />
        </visual>
    </link>
    <!-- 3、使用joint将二者关联 -->
    <joint name="base_footprint2base_link" type="fixed">
        <parent link="base_footprint" />
        <child link="base_link" />
        <!-- z偏移 = 车体高度/2 + 离地间距 -->
        <origin xyz="0.0 0.0 0.05" />
    </joint>
    <!-- 4、添加一个车轮link -->
    <link name="left_front_wheel">
        <visual>
            <geometry>
                <cylinder radius="0.025" length="0.02" />
            </geometry>
            <!-- 颜色 -->
            <material name="black" />
            <!-- 将车轮竖立起来 -->
            <origin rpy="1.57 0.0 0.0" />
        </visual>
    </link>
    <!-- 5、将车轮与base_link关联； -->
    <joint name="left_front_wheel2base_link" type="continuous">
        <parent link="base_link" />
        <child link="left_front_wheel" />
        <!-- 将车轮移到左前方 -->
        <origin xyz="0.08 0.06 -0.025" />
        <!-- 车轮旋转 -->
        <axis xyz="0 1 0" />
    </joint>

    <!-- 4、添加一个车轮link -->
    <link name="right_front_wheel">
        <visual>
            <geometry>
                <cylinder radius="0.025" length="0.02" />
            </geometry>
            <!-- 颜色 -->
            <material name="black" />
            <!-- 将车轮竖立起来 -->
            <origin rpy="1.57 0.0 0.0" />
        </visual>
    </link>
    <!-- 5、将车轮与base_link关联； -->
    <joint name="right_front_wheel2base_link" type="continuous">
        <parent link="base_link" />
        <child link="right_front_wheel" />
        <!-- 将车轮移 -->
        <origin xyz="0.08 -0.06 -0.025" />
        <!-- 车轮旋转 -->
        <axis xyz="0 1 0" />
    </joint>

    <!-- 4、添加一个车轮link -->
    <link name="left_back_wheel">
        <visual>
            <geometry>
                <cylinder radius="0.025" length="0.02" />
            </geometry>
            <!-- 颜色 -->
            <material name="black" />
            <!-- 将车轮竖立起来 -->
            <origin rpy="1.57 0.0 0.0" />
        </visual>
    </link>
    <!-- 5、将车轮与base_link关联； -->
    <joint name="left_back_wheel2base_link" type="continuous">
        <parent link="base_link" />
        <child link="left_back_wheel" />
        <!-- 将车轮移到左 -->
        <origin xyz="-0.08 0.06 -0.025" />
        <!-- 车轮旋转 -->
        <axis xyz="0 1 0" />
    </joint>
    <!-- 4、添加一个车轮link -->
    <link name="right_back_wheel">
        <visual>
            <geometry>
                <cylinder radius="0.025" length="0.02" />
            </geometry>
            <!-- 颜色 -->
            <material name="black" />
            <!-- 将车轮竖立起来 -->
            <origin rpy="1.57 0.0 0.0" />
        </visual>
    </link>
    <!-- 5、将车轮与base_link关联； -->
    <joint name="right_back_wheel2base_link" type="continuous">
        <parent link="base_link" />
        <child link="right_back_wheel" />
        <!-- 将车轮移 -->
        <origin xyz="-0.08 -0.06 -0.025" />
        <!-- 车轮旋转 -->
        <axis xyz="0 1 0" />
    </joint>
</robot>
```

**效果图：**

<img src="https://gitee.com/zhangchenxuv/images/raw/main/image/image-20241024232309205.png" alt="image-20241024232309205" style="zoom:50%;" />

#### |--URDF优化xacro

Xacro 可以声明变量，可以通过数学运算求解；可以使用流程控制控制执行顺序；还可以通过宏封装、复用功能，从而提高代码复用率以及程序的安全性。

#### |--仿真

##### |--ros_control

**ros_control:**是一组软件包，它包含了控制器接口，控制器管理器，传输和硬件接口。ros_control 是一套机器人控制的中间件，是一套规范，不同的机器人平台只要按照这套规范实现，那么就可以保证 与ROS 程序兼容，通过这套规范，实现了一种可插拔的架构设计，大大提高了程序设计的效率与灵活性。

gazebo 已经实现了 ros_control 的相关接口，如果需要在 gazebo 中控制机器人运动，直接调用相关接口即可







## 3-7 Launch

WS_TOOLS

CMakeLists.txt 中添加语句：

```cmake
install(DIRECTORY launch DESTINATION share/${PROJECT_NAME})
```

注意命名规范

示例:启动两个``turtlesim_node``

```py
from launch import LaunchDescription
from launch_ros.actions import Node
# ---|封装终端指令相关类
# from launch.actions import ExecuteProcess
# from launch.substitutions import FindExecutable
# ---|参数声明与获取
# from launch.actions import DeclareLaunchArgument
# from launch.substitutions import LaunchConfiguration
# ---|文件包含相关
# from launch.action import IncludeLaunchDescription
# from launch.launch_description_sources import PythonLaunchDescriptionSource
# ---|分组相关
# from launch_ros.actions import PushRosNamespace
# from launch.actions import GroupAction
# ---|事件相关
# from launch.event_handlers import OnProcessStart, OnProcessExit
# from launch.actions import ExecuteProcess, RegisterEventHandler,LogInfo
# ---|获取功能包下 share 目录路径
# from ament_index_python.packages import get_package_share_directory

def generate_launch_description():
    # 功能包 可执行程序 
    t1 = Node(package="turtlesim",executable="turtlesim_node",name="t1")
    t2 = Node(package="turtlesim",executable="turtlesim_node",name="t2")

    return LaunchDescription([t1,t2])
```

### 3-7-1 节点设置

- `executable`：指定要执行的程序的名称，如果提供了包名，则在该包中查找可执行文件，否则视为可执行文件的路径。
- `package`：可选参数，指定包含节点可执行文件的包名。
- `name`：节点的名称。如果未指定（或为None），则使用节点代码中指定的默认名称。
- `namespace`：节点的ROS命名空间，可以是绝对路径（以'/'开头）或相对路径。如果是相对路径，将在LaunchConfiguration中指定的`ros_namespace`基础上构建。
- `exec_name`：用于标识进程的标签。默认为节点可执行文件的基本名称。
- `parameters`：参数列表，可以是包含参数规则的yaml文件名，或者是指定参数规则的字典。
- `remappings`：有序的'to'和'from'字符串对列表，表示要传递给节点的ROS重映射规则。
- `ros_arguments`：节点的ROS参数，等同于在`arguments`中添加了一个以'--ros-args'为前缀的项。
- `arguments`：节点的额外参数列表。

这个动作使用了`launch_ros.substitutions.ExecutableInPackage`替换来在运行时查找可执行文件，如果包或可执行文件未找到，会抛出异常。参数可以是yaml文件路径或参数字典，这些参数将被写入一个临时yaml文件，并将其路径传递给节点。

此外，如果提供了命名空间，字典中的参数会以前缀wildcard namespace（`/**`）开始，其他具体的参数声明可能会覆盖它。如果未指定`namespace`，则默认命名空间为`/`。

这个动作在执行时，大部分工作委托给`launch.actions.ExecuteProcess`类，但同时也会将一些ROS特定的参数转换为通用的命令行参数。这意味着，虽然很多参数最终会传递给`launch.actions.ExecuteProcess`，但这个动作还负责处理一些ROS特定的逻辑。



**exec_name**:

会使终端输出时，前面的信息发生变化

<img src="https://gitee.com/zhangchenxuv/images/raw/main/image/image-20241019221111097.png" alt="image-20241019221111097" style="zoom:50%;" />

**ros_arguments**:

```py
def generate_launch_description():
    turtle1 = Node(
        package="turtlesim",
        executable="turtlesim_node",
        exec_name="my_label",
        ros_arguments=["--remap","__ns:=/t2"]
        # 等价于在终端输入 ros2 run turtlesim turtlesim_node --ros-args --remap __ns:=/t2 （修改命名空间）
        )
```



**parameters**:

```py
    turtle2 = Node(
        package="turtlesim",
        executable="turtlesim_node",
        name="HHHHHHHH",
        # 背景颜色改为红色 
        # 方式1 直接设置参数
        # parameters=[{"background_r":255,"background_g":0,"background_b":0}],
        # 方式2 参数保存在yaml文件中 通过绝对路径加载或相对路径
        parameters=["install/cpp01_launch/share/cpp01_launch/config/HHHHHHHH.yaml"]

        )
```

如何将参数导出至yaml文件中呢？

```bash
ros2 param dump HHHHHHHH --output-dir src/cpp01_launch/config
```

生成的yaml文件不能直接使用，需要在cmake文件修改，添加``config``

```cmake
install(DIRECTORY launch config DESTINATION share/${PROJECT_NAME})
```



**respawn**:如果程序因为异常关闭，可以自动重启

```py
        package="turtlesim",
        executable="turtlesim_node",
        respawn=True,
        name="HHHHHHHH",
```



### 3-7-2 执行指令

主要与``from launch.actions import ExecuteProcess``有关

```py
from launch import LaunchDescription
from launch_ros.actions import Node
# ---|封装终端指令相关类
from launch.actions import ExecuteProcess
# from launch.substitutions import FindExecutable
# ---|参数声明与获取
# from launch.actions import DeclareLaunchArgument
# from launch.substitutions import LaunchConfiguration
# ---|文件包含相关
# from launch.action import IncludeLaunchDescription
# from launch.launch_description_sources import PythonLaunchDescriptionSource
# ---|分组相关
# from launch_ros.actions import PushRosNamespace
# from launch.actions import GroupAction
# ---|事件相关
# from launch.event_handlers import OnProcessStart, OnProcessExit
# from launch.actions import ExecuteProcess, RegisterEventHandler,LogInfo
# ---|获取功能包下 share 目录路径
# from ament_index_python.packages import get_package_share_directory

"""
    启动turtlesim_node节点，调用指令打印乌龟位姿信息
"""
def generate_launch_description():
    t1 = Node(package="turtlesim",
              executable="turtlesim_node",
              name="t1")
    
    # 封装指令
    cmd = ExecuteProcess(
        cmd=["ros2 topic echo /turtle1/pose"],
        # 同时写入磁盘和终端
        output="both",
        # 当成终端指令执行
        shell=True
    )
    
    return LaunchDescription([t1,cmd])
```

此外亦可以通过``FindExecutable``封装

### 3-7-3 参数设置

主要与``from launch.actions import DeclareLaunchArgument和from launch.substitutions import LaunchConfiguration``有关的API。

```py
from launch import LaunchDescription
from launch_ros.actions import Node
# ---|封装终端指令相关类
# from launch.actions import ExecuteProcess
# from launch.substitutions import FindExecutable
# ---|参数声明与获取
from launch.actions import DeclareLaunchArgument
from launch.substitutions import LaunchConfiguration
# ---|文件包含相关
# from launch.action import IncludeLaunchDescription
# from launch.launch_description_sources import PythonLaunchDescriptionSource
# ---|分组相关
# from launch_ros.actions import PushRosNamespace
# from launch.actions import GroupAction
# ---|事件相关
# from launch.event_handlers import OnProcessStart, OnProcessExit
# from launch.actions import ExecuteProcess, RegisterEventHandler,LogInfo
# ---|获取功能包下 share 目录路径
# from ament_index_python.packages import get_package_share_directory
"""
    动态设置背景颜色
    1、声明launch中的参数（变量）；
    2、调用参数（调用变量）；
    3、执行launch时动态导入参数；
"""
def generate_launch_description():
    # 1.声明参数（变量）
    bg_r = DeclareLaunchArgument("backg_r",default_value="255")
    # 2.调用参数（变量）
    t1 = Node(
        package="turtlesim",
        executable="turtlesim_node",
        parameters=[{"background_r":LaunchConfiguration("backg_r")}]
    )
    
    # 传入列表
    return LaunchDescription([bg_r,t1])
```



### 3-7-4 文件包含

在 launch 文件中可以包含其他 launch 文件，需要使用的 API 为：``launch.actions.IncludeLaunchDescription`` 和
``aunch.launch_description_sources.PythonLaunchDescriptionSource``。

```py
from launch import LaunchDescription
from launch_ros.actions import Node
# ---|封装终端指令相关类
# from launch.actions import ExecuteProcess
# from launch.substitutions import FindExecutable
# ---|参数声明与获取
# from launch.actions import DeclareLaunchArgument
# from launch.substitutions import LaunchConfiguration
# ---|文件包含相关
from launch.actions import IncludeLaunchDescription
from launch.launch_description_sources import PythonLaunchDescriptionSource
# ---|分组相关
# from launch_ros.actions import PushRosNamespace
# from launch.actions import GroupAction
# ---|事件相关
# from launch.event_handlers import OnProcessStart, OnProcessExit
# from launch.actions import ExecuteProcess, RegisterEventHandler,LogInfo
# ---|获取功能包下 share 目录路径
# from ament_index_python.packages import get_package_share_directory

"""
    在当前launch文件包含其他launch文件
"""
def generate_launch_description():
    include1 = IncludeLaunchDescription(
        launch_description_source=PythonLaunchDescriptionSource(
            launch_file_path="install/cpp01_launch/share/cpp01_launch/launch/py/py04_args_launch.py"
        )
        # 也可以直接传参
    )
    include2 = IncludeLaunchDescription(
        launch_description_source=PythonLaunchDescriptionSource(
            launch_file_path="install/cpp01_launch/share/cpp01_launch/launch/py/py03_cmd_launch.py"
        )
    )
        # 也可以直接传参
    
    return LaunchDescription([include1,include2])
```



### 3-7-4  分组设置

使用的API为``from launch_ros.actions import PushRosNamespace``和``from launch.actions import GroupAction``

```py
from launch import LaunchDescription
from launch_ros.actions import Node
# ---|封装终端指令相关类
# from launch.actions import ExecuteProcess
# from launch.substitutions import FindExecutable
# ---|参数声明与获取
# from launch.actions import DeclareLaunchArgument
# from launch.substitutions import LaunchConfiguration
# ---|文件包含相关
# from launch.actions import IncludeLaunchDescription
# from launch.launch_description_sources import PythonLaunchDescriptionSource
# ---|分组相关
from launch_ros.actions import PushRosNamespace
from launch.actions import GroupAction
# ---|事件相关
# from launch.event_handlers import OnProcessStart, OnProcessExit
# from launch.actions import ExecuteProcess, RegisterEventHandler,LogInfo
# ---|获取功能包下 share 目录路径
# from ament_index_python.packages import get_package_share_directory
"""
    创建三个turtlesim_node，将前两个划分为一组，第三个单独一组
"""

def generate_launch_description():
    # 创建三个 turtlesim_node
    t1 = Node(package="turtlesim",executable="turtlesim_node",name="t1")
    t2 = Node(package="turtlesim",executable="turtlesim_node",name="t2")
    t3 = Node(package="turtlesim",executable="turtlesim_node",name="t3")
    # 分组
    # 设置当前组的命名空间，以及包含的节点
    g1 = GroupAction(actions=[PushRosNamespace("g1_nw"),t1,t2])
    g2 = GroupAction(actions=[PushRosNamespace("g2_nw"),t3])
    return LaunchDescription([g1,g2])
```



### 3-7-5  添加事件

主要涉及``from launch.event_handlers import OnProcessStart, OnProcessExit``和``from launch.actions import ExecuteProcess, RegisterEventHandler,LogInfo``

```py
from launch import LaunchDescription
from launch_ros.actions import Node
# ---|封装终端指令相关类
from launch.actions import ExecuteProcess
# from launch.substitutions import FindExecutable
# ---|参数声明与获取
# from launch.actions import DeclareLaunchArgument
# from launch.substitutions import LaunchConfiguration
# ---|文件包含相关
# from launch.actions import IncludeLaunchDescription
# from launch.launch_description_sources import PythonLaunchDescriptionSource
# ---|分组相关
# from launch_ros.actions import PushRosNamespace
# from launch.actions import GroupAction
# ---|事件相关
from launch.event_handlers import OnProcessStart, OnProcessExit
from launch.actions import ExecuteProcess, RegisterEventHandler,LogInfo
# ---|获取功能包下 share 目录路径
# from ament_index_python.packages import get_package_share_directory

"""
    为 turtlesim_node 绑定事件，节点启动时，执行生成新的乌龟的程序，节点关闭执行日志输出
"""
def generate_launch_description():
    turtle = Node(
        package="turtlesim",
        executable="turtlesim_node"
    )
    spawn = ExecuteProcess(
        cmd=["ros2 service call /spawn turtlesim/srv/Spawn \"{'x': 8.0,'y': 3.0}\""],
        output="both",
        shell=True
    )
    # 注册事件1
    event_start = RegisterEventHandler(
        # 事件源 和 操作
        event_handler=OnProcessStart(
            target_action=turtle,
            on_start=spawn
        )
    )
    # 注册事件2 
    enent_exit = RegisterEventHandler(
        event_handler=OnProcessExit(
            target_action=turtle,
            on_exit=[LogInfo(msg="turtlesim_node 退出！")]
        )
    )
    return LaunchDescription([turtle,event_start,enent_exit])
```



## 3-8 通讯补充

### 3-8-1 多线程

https://blog.csdn.net/m0_73800387/article/details/138862061

### 3-8-2 时间相关API



## 3-9 机器人系统仿真









# 四、QT

## 1、安装QT

【QT】Ubuntu22.04 配置 QT6.5 LTShttps://blog.csdn.net/qq_44940689/article/details/138156395?spm=1001.2014.3001.5502

## 2、安装ROSProjectManager插件

【ROS2-QT合并编程】从环境搭建到UI界面编写https://blog.csdn.net/Functioe/article/details/131659829

【QT】ROS2 Humble联合使用QT教程https://blog.csdn.net/qq_44940689/article/details/138165085

【基于 QT5 的 ROS2 GUI 开发教程（一）】话题消息的发布和订阅http://admin.guyuehome.com/43738

## 3、QT creator无法输入中文

【Ubuntu中Qt6 fcitx5输入法中文解决方案】https://blog.csdn.net/m0_46144825/article/details/122462453

## 4、QT 程序启动流程

### 4-1 启动流程

``main.cpp``

```c++
#include "mainwindow.h"

#include <QApplication>

// 程序入口
// argc是命令行参数个数 argv是命令行参数
int main(int argc, char *argv[])
{
    // QApplication管理Qt程序运行，和设置Qt应用程序，针对QWidget应用程序
    QApplication a(argc, argv);
    // MainWindow是自定义的类，W是创建的对象
    MainWindow w;
    // w对象调用了show()方法，不调用不会显示界面
    w.show();
    // 返回一个事件循环，等待输入
    return a.exec();
}

```

``mainwindow.h``


```c++
#ifndef MAINWINDOW_H
#define MAINWINDOW_H

#include <QMainWindow>

QT_BEGIN_NAMESPACE
namespace Ui {
class MainWindow;
}
QT_END_NAMESPACE
    
// MainWindow是自定义的类，公有继承QMainWindow类
class MainWindow : public QMainWindow
{
    // 宏定义，Qt信号槽需要
    Q_OBJECT

public:
    // 此处的MainWindow构造函数在  ``mywindow.cpp``中
    MainWindow(QWidget *parent = nullptr);
    ~MainWindow();

private:
    // 此处的MainWindow与上方的不同，该MainWindow是在UI命名空间下的
    Ui::MainWindow *ui;
};
#endif // MAINWINDOW_H

```

``mainwindow.cpp``


```c++
#include "mainwindow.h"
#include "./ui_mainwindow.h"

MainWindow::MainWindow(QWidget *parent)
    : QMainWindow(parent) //此处定义了一个对象*parent，并交给QMainWindow初始化
    , ui(new Ui::MainWindow) // 实例化ui对象
{
    ui->setupUi(this);
}

MainWindow::~MainWindow()
{
    delete ui;
}
```

### 4-2 如何添加自己的代码

只修改mainwindow.cpp

```c++
#include "mainwindow.h"
#include "./ui_mainwindow.h"
#include <QDebug>

MainWindow::MainWindow(QWidget *parent)
    : QMainWindow(parent)
    , ui(new Ui::MainWindow)
{
    ui->setupUi(this);
    
    qDebug()<<"构造函数执行"<<Qt::endl;
}

MainWindow::~MainWindow()
{
    // 析构函数，只有在项目销毁时才会执行
    delete ui;
}

```

### 4-3 初始化变量

首先需要在``mainwindow.h``中声明

```c++
#ifndef MAINWINDOW_H
#define MAINWINDOW_H

#include <QMainWindow>

QT_BEGIN_NAMESPACE
namespace Ui {
class MainWindow;
}
QT_END_NAMESPACE
    
// MainWindow是自定义的类，公有继承QMainWindow类
class MainWindow : public QMainWindow
{
    // 宏定义，Qt信号槽需要
    Q_OBJECT

public:
    // 此处的MainWindow构造函数在  ``mywindow.cpp``中
    MainWindow(QWidget *parent = nullptr);
    ~MainWindow();
    
    /* ******** 声明变量*/
    int i;
    /* ******** 声明变量*/

private:
    // 此处的MainWindow与上方的不同，该MainWindow是在UI命名空间下的
    Ui::MainWindow *ui;
};
#endif // MAINWINDOW_H
```

之后在``mainwindow.cpp``中初始化

```c++
#include "mainwindow.h"
#include "./ui_mainwindow.h"
#include <QDebug>

MainWindow::MainWindow(QWidget *parent)
    : QMainWindow(parent)
    , i(4) // 相当于 i = 4
    , ui(new Ui::MainWindow)
{
    ui->setupUi(this);
    
    qDebug()<<"构造函数执行"<<Qt::endl;
}

MainWindow::~MainWindow()
{
    // 析构函数，只有在项目销毁时才会执行
    delete ui;
}
```

### 4-4 编程规范

- 文件命名小写

- 类的名字首写大写，单词之间首写大写

- 成员函数首写小写，单词之间首字母大写

- 变量首字母小写，单词与单词之间首字母大写

## 5、UI设计器

QT的坐标以左上角为原点

## 6、QT信号槽

例子：

``学校``-->``通知``-->``学生``-->``上课``

QT中信号与槽连接模型：

``发送者``-->``信号（函数）``-->``接受者``-->``槽（函数）``

 对象 -->函数-->对象-->函数

### 6-1 如何设计？

<img src="/home/zhangchenxu/Pictures/Typora/image-20240822182334625.png" alt="image-20240822182334625" style="zoom: 33%;" />

模型中按钮既是**发送者**，而主窗口则是**接受者**

第一种方式：通过下方功能栏设置

<img src="/home/zhangchenxu/Pictures/Typora/image-20240822183047754.png" alt="image-20240822183047754" style="zoom:33%;" />

第二种方式：通过功能栏进行可视化拖动操作

<img src="/home/zhangchenxu/Pictures/Typora/image-20240822183221577.png" alt="image-20240822183221577" style="zoom:33%;" />

第三种方式：实现自定义函数

<img src="/home/zhangchenxu/Pictures/Typora/image-20240822183505334.png" alt="image-20240822183505334" style="zoom:33%;" />

### 6-2 简介

Qt 信号和槽是 Qt 框架中用于对象间通信的机制。信号和槽机制允许对象之间进行通信，而不需要知道对方的内部实现。当某个事件发生时，对象可以发出信号，其他对象可以接收这些信号，并对信号做出响应，执行相应的槽函数。

- **信号（Signal）**：当某个事件或操作发生时，对象可以发出一个信号。信号可以被连接到一个或多个槽函数上。

- **槽（Slot）**：槽是一个函数，当信号发出时，槽函数会被调用。槽函数可以是任何成员函数，甚至是 lambda 表达式。

```c++
#include "mainwindow.h"
#include "./ui_mainwindow.h"
#include <QDebug>

MainWindow::MainWindow(QWidget *parent)
    : QMainWindow(parent)
    , ui(new Ui::MainWindow)
{
    ui->setupUi(this);
    // connect 按照：发送者 信号 接受者 槽
    connect(ui->pushButton,SIGNAL(clicked()),this,SLOT(close()));
}

MainWindow::~MainWindow()
{
    delete ui;
}

```

### 6-3 自定义信号槽

实现下面一个例子：

<img src="/home/zhangchenxu/Pictures/Typora/image-20240822211551250.png" alt="image-20240822211551250" style="zoom:33%;" />

首先需要建立一个“学校”类和“学生”类：

<img src="/home/zhangchenxu/Pictures/Typora/image-20240822211712759.png" alt="image-20240822211712759" style="zoom: 25%;" />

<img src="/home/zhangchenxu/Pictures/Typora/image-20240822211821741.png" alt="image-20240822211821741" style="zoom: 25%;" />

同理创建学生类；

建立如下：

<img src="/home/zhangchenxu/Pictures/Typora/image-20240822213359744.png" alt="image-20240822213359744" style="zoom:25%;" />



# 五、C++相关

## 5-1 CMake

### 5-1-1 只有源文件

CMake 使用 # 进行行注释，可以放在任何位置。

```cmake
cmake_minimum_required(VERSION 3.0)
project(CALC)
add_executable(app add.c div.c main.c mult.c sub.c)
```

``cmake_minimum_required：``指定使用的 cmake 的最低版本

``project：``定义工程名称，并可指定工程的版本、工程描述、web主页地址、支持的语言（默认情况支持所有语言），如果不需要这些都是可以忽略的，只需要指定出工程名字即可。

```cmake
# PROJECT 指令的语法是：
project(<PROJECT-NAME> [<language-name>...])
project(<PROJECT-NAME>
       [VERSION <major>[.<minor>[.<patch>[.<tweak>]]]]
       [DESCRIPTION <project-description-string>]
       [HOMEPAGE_URL <url-string>]
       [LANGUAGES <language-name>...])
```

``add_executable：``定义工程会生成一个可执行程序

```cmake
add_executable(可执行程序名 源文件名称)
```

**编译**

在cmakelists目录下，执行``cmake``命令 如

```shell
# . 代表在该目录下 ..则是在上一目录
cmake .
```

编译之后会生成``Makefile``文件，之后通过``Makefile``文件生成可执行程序

在``Makefile``文件所在目录下执行

```shell
make
```

即可生成一个可执行程序``app``

运行时则为

```shell
./app
```

### 5-1-2 set 的使用

在使用SET时，其对应的变量值均是字符串类型。

```c++
# SET 指令的语法是：
# [] 中的参数为可选项, 如不需要可以不写
SET(VAR [VALUE] [CACHE TYPE DOCSTRING [FORCE]])
```

- VAR：变量名
- VALUE：变量值

```cmake
# 方式1: 各个源文件之间使用空格间隔
# set(SRC_LIST add.c  div.c   main.c  mult.c  sub.c)

# 方式2: 各个源文件之间使用分号 ; 间隔
set(SRC_LIST add.c;div.c;main.c;mult.c;sub.c)
add_executable(app  ${SRC_LIST})
# 即 将add.c;div.c;main.c;mult.c;sub.c按照字符串储存在SRC_LIST中，通过${SRC_LIST}取变量值
```

在CMake中指定可执行程序输出的路径，也对应一个宏，叫做EXECUTABLE_OUTPUT_PATH，它的值还是通过set命令进行设置:

```cmake
set(HOME /home/robin/Linux/Sort)
set(EXECUTABLE_OUTPUT_PATH ${HOME}/bin)
```



### 5-1-3 指定使用的C++标准

```cmake
#增加-std=c++11
set(CMAKE_CXX_STANDARD 11)
#增加-std=c++14
set(CMAKE_CXX_STANDARD 14)
#增加-std=c++17
set(CMAKE_CXX_STANDARD 17)
```

### 5-1-4 搜索指定路径下的源文件（.cpp）

**第一种方式**

在 CMake 中使用``aux_source_directory`` 命令可以查找某个路径下的所有源文件，命令格式为：

```
aux_source_directory(< dir > < variable >)
```

- ``dir：``要搜索的目录
- ``variable：``将从dir目录下搜索到的源文件列表存储到该变量

```cmake
cmake_minimum_required(VERSION 3.0)
project(CALC)
include_directories(${PROJECT_SOURCE_DIR}/include)
# 搜索 src 目录下的源文件
# PROJECT_SOURCE_DIR 是CMakeLists.txt文件所在的路径
aux_source_directory(${CMAKE_CURRENT_SOURCE_DIR}/src SRC_LIST)
add_executable(app  ${SRC_LIST})
```

**第二种方式**

如果一个项目里边的源文件很多，在编写CMakeLists.txt文件的时候不可能将项目目录的各个文件一一罗列出来，这样太麻烦了。所以，在CMake中为我们提供了搜索文件的命令，他就是``file``（当然，除了搜索以外通过 file 还可以做其他事情）。

```cmake
file(GLOB/GLOB_RECURSE 变量名 要搜索的文件路径和文件类型)
```

- ``GLOB: ``将指定目录下搜索到的满足条件的所有文件名生成一个列表，并将其存储到变量中。
- ``GLOB_RECURSE：``递归搜索指定目录，将搜索到的满足条件的文件名生成一个列表，并将其存储到变量中。

```cmake
# CMAKE_CURRENT_SOURCE_DIR 是CMakeLists.txt文件所在的路径
file(GLOB MAIN_SRC ${CMAKE_CURRENT_SOURCE_DIR}/src/*.cpp)
file(GLOB MAIN_HEAD ${CMAKE_CURRENT_SOURCE_DIR}/include/*.h)
```

``CMAKE_CURRENT_SOURCE_DIR`` 宏表示当前访问的 CMakeLists.txt 文件所在的路径。

关于要搜索的文件路径和类型可加双引号，也可不加:

```cmake
file(GLOB MAIN_HEAD "${CMAKE_CURRENT_SOURCE_DIR}/src/*.h")
```

### 5-1-5  指定头文件路径

如果源文件和头文件在不同的目录下，则需要指定目录

在编译项目源文件的时候，很多时候都需要将源文件对应的头文件路径指定出来，这样才能保证在编译过程中编译器能够找到这些头文件，并顺利通过编译。在CMake中设置要包含的目录也很简单，通过``include_directories``命令就可以搞定。

```cmake
include_directories(headpath)
```

举例说明，有源文件若干，其目录结构如下：

```c++
$ tree
.
├── build
├── CMakeLists.txt
├── include
│   └── head.h
└── src
    ├── add.cpp
    ├── div.cpp
    ├── main.cpp
    ├── mult.cpp
    └── sub.cpp
```

``CMakeLists.txt``文件内容如下:

```cmake
cmake_minimum_required(VERSION 3.0)
project(CALC)
set(CMAKE_CXX_STANDARD 11)
set(HOME /home/robin/Linux/calc)
set(EXECUTABLE_OUTPUT_PATH ${HOME}/bin/)
include_directories(${PROJECT_SOURCE_DIR}/include)
file(GLOB SRC_LIST ${CMAKE_CURRENT_SOURCE_DIR}/src/*.cpp)
add_executable(app  ${SRC_LIST})
```

### 5-1-5 如何制作动态库（.so）或静态库（.a）

**制作静态库**

在Linux中，静态库名字分为三部分：``lib``+``库名字``+``.a``，此处只需要指定出库的名字就可以了，另外两部分在生成该文件的时候会自动填充。

```cmake
add_library(库名称 STATIC 源文件1 [源文件2] ...) 
```

下面有一个目录，需要将src目录中的源文件编译成静态库，然后再使用：

```cpp
.
├── build
├── CMakeLists.txt
├── include           # 头文件目录
│   └── head.h
├── main.cpp          # 用于测试的源文件
└── src               # 源文件目录
    ├── add.cpp
    ├── div.cpp
    ├── mult.cpp
    └── sub.cpp
```

根据上面的目录结构，可以这样编写``CMakeLists.txt``文件:

```cmake
cmake_minimum_required(VERSION 3.0)
project(CALC)
include_directories(${PROJECT_SOURCE_DIR}/include)
file(GLOB SRC_LIST "${CMAKE_CURRENT_SOURCE_DIR}/src/*.cpp")
add_library(calc STATIC ${SRC_LIST})
```

这样最终就会生成对应的静态库文件``libcalc.a``

**制作动态库**

```cmake
add_library(库名称 SHARED 源文件1 [源文件2] ...) 
```

在Linux中，动态库名字分为三部分：lib+库名字+.so，此处只需要指定出库的名字就可以了，另外两部分在生成该文件的时候会自动填充。

```cmake
cmake_minimum_required(VERSION 3.0)
project(CALC)
include_directories(${PROJECT_SOURCE_DIR}/include)
file(GLOB SRC_LIST "${CMAKE_CURRENT_SOURCE_DIR}/src/*.cpp")
add_library(calc SHARED ${SRC_LIST})
```

这样最终就会生成对应的动态库文件libcalc.so。

**指定输出的路径**

在Linux下生成的动态库默认是有执行权限的，所以可以按照生成可执行程序的方式去指定它生成的目录：

```cmake
cmake_minimum_required(VERSION 3.0)
project(CALC)
include_directories(${PROJECT_SOURCE_DIR}/include)
file(GLOB SRC_LIST "${CMAKE_CURRENT_SOURCE_DIR}/src/*.cpp")
# 设置动态库生成路径
set(EXECUTABLE_OUTPUT_PATH ${PROJECT_SOURCE_DIR}/lib)
add_library(calc SHARED ${SRC_LIST})
```

由于在Linux下生成的静态库默认不具有可执行权限，所以在指定静态库生成的路径的时候就不能使用EXECUTABLE_OUTPUT_PATH宏，而应该使用LIBRARY_OUTPUT_PATH，**这个宏对应静态库文件和动态库文件都适用。**

```cmake
cmake_minimum_required(VERSION 3.0)
project(CALC)
include_directories(${PROJECT_SOURCE_DIR}/include)
file(GLOB SRC_LIST "${CMAKE_CURRENT_SOURCE_DIR}/src/*.cpp")
# 设置动态库/静态库生成路径
set(LIBRARY_OUTPUT_PATH ${PROJECT_SOURCE_DIR}/lib)
# 生成动态库
add_library(calc SHARED ${SRC_LIST})
# 生成静态库
add_library(calc STATIC ${SRC_LIST})
```

### 5-1-6 包含静态库和动态库

#### **链接静态库**

```shell
src
├── add.cpp
├── div.cpp
├── main.cpp
├── mult.cpp
└── sub.cpp
```

add.cpp、div.cpp、mult.cpp、sub.cpp编译成一个静态库文件libcalc.a

```shell
$ tree 
.
├── build
├── CMakeLists.txt
├── include
│   └── head.h
├── lib
│   └── libcalc.a     # 制作出的静态库的名字
└── src
    └── main.cpp
```

在cmake中，链接静态库的命令如下：

```cmake
link_libraries(<static lib> [<static lib>...])
```

- 参数1：指定出要链接的静态库的名字可以是全名 libxxx.a也可以是掐头（lib）去尾（.a）之后的名字 xxx

- 参数2-N：要链接的其它静态库的名字

  

如果该静态库不是系统提供的（第三方提供的静态库）可能出现静态库找不到的情况，此时可以将静态库的路径也指定出来：

```cmake
link_directories(<lib path>)
```

```cmake
cmake_minimum_required(VERSION 3.0)
project(CALC)
# 搜索指定目录下源文件
file(GLOB SRC_LIST ${CMAKE_CURRENT_SOURCE_DIR}/src/*.cpp)
# 包含头文件路径
include_directories(${PROJECT_SOURCE_DIR}/include)
# 包含静态库路径
link_directories(${PROJECT_SOURCE_DIR}/lib)
# 链接静态库
link_libraries(calc)
add_executable(app ${SRC_LIST})
```

​         

#### **链接动态库**

在cmake中链接动态库的命令如下:

```cmake
target_link_libraries(
    <target> 
    <PRIVATE|PUBLIC|INTERFACE> <item>... 
    [<PRIVATE|PUBLIC|INTERFACE> <item>...]...)
```

- target：指定要加载的库的文件的名字
  - 该文件可能是一个源文件
  - 该文件可能是一个动态库/静态库文件
  - 该文件可能是一个可执行文件
- PRIVATE|PUBLIC|INTERFACE：动态库的访问权限，默认为PUBLIC。如果各个动态库之间没有依赖关系，无需做任何设置，三者没有没有区别，一般无需指定，使用默认的 PUBLIC 即可。

**区别**

动态库的链接和静态库是完全不同的：

- 静态库会在生成可执行程序的链接阶段被打包到可执行程序中，所以可执行程序启动，静态库就被加载到内存中了。
- 动态库在生成可执行程序的链接阶段**不会被打包到可执行程序**中，当可执行程序被启动并且调用了动态库中的函数的时候，动态库才会被加载到内存

因此，在cmake中指定要链接的动态库的时候，应该**将命令写到生成了可执行文件之后**：

```cmake
cmake_minimum_required(VERSION 3.0)
project(TEST)
file(GLOB SRC_LIST ${CMAKE_CURRENT_SOURCE_DIR}/*.cpp)
# 添加并指定最终生成的可执行程序名
add_executable(app ${SRC_LIST})
# 指定可执行程序要链接的动态库名字
target_link_libraries(app pthread)
```

- app: 对应的是最终生成的可执行程序的名字

- pthread：这是可执行程序要加载的动态库，这个库是系统提供的线程库，全名为libpthread.so，在指定的时候一般会掐头（lib）去尾（.so）。

  

**链接第三方动态库**

```
$ tree 
.
├── build
├── CMakeLists.txt
├── include
│   └── head.h            # 动态库对应的头文件
├── lib
│   └── libcalc.so        # 自己制作的动态库文件
└── main.cpp              # 测试用的源文件

3 directories, 4 files
```

假设在测试文件main.cpp中既使用了自己制作的动态库libcalc.so又使用了系统提供的线程库，此时CMakeLists.txt文件可以这样写：

```cmake
cmake_minimum_required(VERSION 3.0)
project(TEST)
file(GLOB SRC_LIST ${CMAKE_CURRENT_SOURCE_DIR}/*.cpp)
include_directories(${PROJECT_SOURCE_DIR}/include)
add_executable(app ${SRC_LIST})
target_link_libraries(app pthread calc)
```

在第六行中，pthread（系统）、calc（第三方）都是可执行程序app要链接的动态库的名字。当可执行程序app生成之后并执行该文件，会提示有如下错误信息：

```shell
$ ./app 
./app: error while loading shared libraries: libcalc.so: cannot open shared object file: No such file or directory
```

这是因为可执行程序启动之后，去加载calc（第三方动态库）这个动态库，但是不知道这个动态库被放到了什么位置解决动态库无法加载的问题，所以就加载失败了，在 CMake 中可以在生成可执行程序之前，**通过命令指定出要链接的动态库的位置**，指定静态库位置使用的也是这个命令：

```cmake
link_directories(path)
```

修改后应该是这个样子

```cmake
cmake_minimum_required(VERSION 3.0)
project(TEST)
# 搜索指定目录下源文件
file(GLOB SRC_LIST ${CMAKE_CURRENT_SOURCE_DIR}/*.cpp)
# 指定源文件或者动态库对应的头文件路径
include_directories(${PROJECT_SOURCE_DIR}/include)
# 指定要链接的动态库的路径
link_directories(${PROJECT_SOURCE_DIR}/lib)
# 添加并生成一个可执行程序
add_executable(app ${SRC_LIST})
# 指定要链接的动态库
target_link_libraries(app pthread calc)
```

#### **总结**

``target_link_libraries``

- 功能: target_link_libraries 用于指定一个目标（如可执行文件或库）在编译时需要链接哪些库。它支持指定库的名称、路径以及链接库的顺序。
- 优点：
  - 更精确地控制目标的链接库。
  - 可以指定库的不同链接条件（如调试版本、发布版本）。
  - 支持多个目标和多个库之间的复杂关系。
  - 更加灵活和易于维护，特别是在大型项目中。

``link_libraries``

- 功能: link_libraries 用于设置全局链接库，这些库会链接到之后定义的所有目标上。它会影响所有的目标，适用于全局设置，但不如 target_link_libraries 精确。

### 5-1-7 变量操作

**追加**

使用``set``或者``list``进行字符床的拼接，后者可以完成字符串的删除

### 5-1-8 宏定义

```c++
#include <stdio.h>
#define NUMBER  3

int main()
{
    int a = 10;
#ifdef DEBUG
    printf("我是一个程序猿, 我不会爬树...\n");
#endif
    for(int i=0; i<NUMBER; ++i)
    {
        printf("hello, GCC!!!\n");
    }
    return 0;
}
```

如果``DEBUG``这个宏被定义，则代码``ifdef``下的代码生效

```cmake
add_definitions(-D宏名称)
```

```cmake
cmake_minimum_required(VERSION 3.0)
project(TEST)
# 自定义 DEBUG 宏
add_definitions(-DDEBUG)
add_executable(app ./test.c)
```



**预定义宏**

宏	功能
``PROJECT_SOURCE_DIR``	使用cmake命令后紧跟的目录，一般是工程的根目录
``PROJECT_BINARY_DIR	``执行cmake命令的目录
``CMAKE_CURRENT_SOURCE_DIR``	当前处理的CMakeLists.txt所在的路径
``CMAKE_CURRENT_BINARY_DIR``	target 编译目录
``EXECUTABLE_OUTPUT_PATH``	重新定义目标二进制可执行文件的存放位置
``LIBRARY_OUTPUT_PATH	``重新定义目标链接库文件的存放位置
``PROJECT_NAME``	返回通过PROJECT指令定义的项目名称
``CMAKE_BINARY_DIR``	项目实际构建路径，假设在build目录进行的构建，那么得到的就是这个目录的路径

## 5-2 C++核心编程

### 5-2-1 内存分区模型

C++程序在执行时，将内存大方向划分为**4个区域**

- 代码区：存放函数体的二进制代码，由操作系统进行管理的
- 全局区：存放全局变量和静态变量以及常量
- 栈区：由编译器自动分配释放, 存放函数的参数值,局部变量等
- 堆区：由程序员分配和释放,若程序员不释放,程序结束时由操作系统回收

不同区域存放的数据，赋予不同的生命周期, 给我们更大的灵活编程

**代码区：**

​		存放 CPU 执行的机器指令

​		代码区是**共享**的，共享的目的是对于频繁被执行的程序，只需要在内存中有一份代码即可

​		代码区是**只读**的，使其只读的原因是防止程序意外地修改了它的指令

​	**全局区：**

​		全局变量和静态变量存放在此.

​		全局区还包含了常量区, 字符串常量和其他常量也存放在此.

```cpp
//全局变量
int g_a = 10;
int g_b = 10;

//全局常量
const int c_g_a = 10;
const int c_g_b = 10;

int main() {

	//局部变量
	int a = 10;
	int b = 10;

	//打印地址
	cout << "局部变量a地址为： " << (int)&a << endl;
	cout << "局部变量b地址为： " << (int)&b << endl;

	cout << "全局变量g_a地址为： " <<  (int)&g_a << endl;
	cout << "全局变量g_b地址为： " <<  (int)&g_b << endl;

	//静态变量
	static int s_a = 10;
	static int s_b = 10;

	cout << "静态变量s_a地址为： " << (int)&s_a << endl;
	cout << "静态变量s_b地址为： " << (int)&s_b << endl;

	cout << "字符串常量地址为： " << (int)&"hello world" << endl;
	cout << "字符串常量地址为： " << (int)&"hello world1" << endl;

	cout << "全局常量c_g_a地址为： " << (int)&c_g_a << endl;
	cout << "全局常量c_g_b地址为： " << (int)&c_g_b << endl;

	const int c_l_a = 10;
	const int c_l_b = 10;
	cout << "局部常量c_l_a地址为： " << (int)&c_l_a << endl;
	cout << "局部常量c_l_b地址为： " << (int)&c_l_b << endl;

	system("pause");

	return 0;
}
```

总结：

* C++中在程序运行前分为全局区和代码区
* 代码区特点是共享和只读
* 全局区中存放全局变量、静态变量、常量
* 常量区中存放 const修饰的全局常量  和 字符串常量

### 5-2-2 引用

**作用： **给变量起别名

**语法：** `数据类型 &别名 = 原名`

**引用注意事项**

* 引用必须初始化
* 引用在初始化后，不可以改变

```C++
int main() {

	int a = 10;
	int b = 20;
	//int &c; //错误，引用必须初始化
	int &c = a; //一旦初始化后，就不可以更改
	c = b; //这是赋值操作，不是更改引用

	cout << "a = " << a << endl;
	cout << "b = " << b << endl;
	cout << "c = " << c << endl;

	system("pause");

	return 0;
}
```

### 5-2-3 函数高级

#### 函数默认参数

在C++中，函数的形参列表中的形参是可以有默认值的。

语法：` 返回值类型  函数名 （参数= 默认值）{}`

- 如果某个位置参数有默认值，那么从这个位置往后，从左向右，必须都要有默认值
-  如果函数声明有默认值，函数实现的时候就不能有默认参数

#### 函数占位参数

C++中函数的形参列表里可以有占位参数，用来做占位，调用函数时必须填补该位置

**语法：** `返回值类型 函数名 (数据类型){}`

```C++
//函数占位参数 ，占位参数也可以有默认参数
void func(int a, int) {
	cout << "this is func" << endl;
}

int main() {

	func(10,10); //占位参数必须填补

	system("pause");

	return 0;
}
```

#### 函数重载

**作用：**函数名可以相同，提高复用性

**函数重载满足条件：**

* 同一个作用域下
* 函数名称相同
* 函数参数**类型不同**  或者 **个数不同** 或者 **顺序不同**

``` C++
//函数重载需要函数都在同一个作用域下
void func()
{
	cout << "func 的调用！" << endl;
}
void func(int a)
{
	cout << "func (int a) 的调用！" << endl;
}
void func(double a)
{
	cout << "func (double a)的调用！" << endl;
}
void func(int a ,double b)
{
	cout << "func (int a ,double b) 的调用！" << endl;
}
void func(double a ,int b)
{
	cout << "func (double a ,int b)的调用！" << endl;
}

//函数返回值不可以作为函数重载条件
//int func(double a, int b)
//{
//	cout << "func (double a ,int b)的调用！" << endl;
//}


int main() {

	func();
	func(10);
	func(3.14);
	func(10,3.14);
	func(3.14 , 10);
	
	system("pause");

	return 0;
}
```





###  5-2-4 类和对象

C++面向对象的三大特性为：==封装、继承、多态==

C++认为==万事万物都皆为对象==，对象上有其属性和行为

#### 封装

封装是C++面向对象三大特性之一

封装的意义：

* 将属性和行为作为一个整体，表现生活中的事物
* 将属性和行为加以权限控制

**封装意义一：**

​	在设计类的时候，属性和行为写在一起，表现事物

**语法：** `class 类名{   访问权限： 属性  / 行为  };`

**示例1：**设计一个圆类，求圆的周长

**示例代码：**

```C++
//圆周率
const double PI = 3.14;

//1、封装的意义
//将属性和行为作为一个整体，用来表现生活中的事物

//封装一个圆类，求圆的周长
//class代表设计一个类，后面跟着的是类名
class Circle
{
public:  //访问权限  公共的权限

	//属性
	int m_r;//半径

	//行为
	//获取到圆的周长
	double calculateZC()
	{
		//2 * pi  * r
		//获取圆的周长
		return  2 * PI * m_r;
	}
};

int main() {

	//通过圆类，创建圆的对象
	// c1就是一个具体的圆
	Circle c1;
	c1.m_r = 10; //给圆对象的半径 进行赋值操作

	//2 * pi * 10 = = 62.8
	cout << "圆的周长为： " << c1.calculateZC() << endl;

	system("pause");

	return 0;
}
```



类在设计时，可以把属性和行为放在不同的权限下，加以控制

访问权限有三种：

1. public        公共权限  
2. protected 保护权限 被标记为protected的成员可以被同一类中的其他成员（包括成员函数）和其**子类**中的成员访问
3. private      私有权限 被标记为private的成员只能由同一类中的其他成员（包括成员函数）访问。

```C++
//三种权限
//公共权限  public     类内可以访问  类外可以访问
//保护权限  protected  类内可以访问  类外不可以访问
//私有权限  private    类内可以访问  类外不可以访问

class Person
{
	//姓名  公共权限
public:
	string m_Name;

	//汽车  保护权限
protected:
	string m_Car;

	//银行卡密码  私有权限
private:
	int m_Password;

public:
	void func()
	{
		m_Name = "张三";
		m_Car = "拖拉机";
		m_Password = 123456;
	}
};

int main() {

	Person p;
	p.m_Name = "李四";
	//p.m_Car = "奔驰";  //保护权限类外访问不到
	//p.m_Password = 123; //私有权限类外访问不到

	system("pause");

	return 0;
}
```

**struct和class区别**

在C++中 struct和class唯一的**区别**就在于 **默认的访问权限不同**

区别：

* struct 默认权限为公共
* class   默认权限为私有

```cpp
class C1
{
	int  m_A; //默认是私有权限
};

struct C2
{
	int m_A;  //默认是公共权限
};

int main() {

	C1 c1;
	c1.m_A = 10; //错误，访问权限是私有

	C2 c2;
	c2.m_A = 10; //正确，访问权限是公共

	system("pause");

	return 0;
}
```



**成员属性设置为私有**

**优点1：**将所有成员属性设置为私有，可以自己控制读写权限

**优点2：**对于写权限，我们可以检测数据的有效性

```cpp
class Person {
public:

	//姓名设置可读可写
	void setName(string name) {
		m_Name = name;
	}
	string getName()
	{
		return m_Name;
	}


	//获取年龄 
	int getAge() {
		return m_Age;
	}
	//设置年龄
	void setAge(int age) {
		if (age < 0 || age > 150) {
			cout << "你个老妖精!" << endl;
			return;
		}
		m_Age = age;
	}

	//情人设置为只写
	void setLover(string lover) {
		m_Lover = lover;
	}

private:
	string m_Name; //可读可写  姓名
	
	int m_Age; //只读  年龄

	string m_Lover; //只写  情人
};


int main() {

	Person p;
	//姓名设置
	p.setName("张三");
	cout << "姓名： " << p.getName() << endl;

	//年龄设置
	p.setAge(50);
	cout << "年龄： " << p.getAge() << endl;

	//情人设置
	p.setLover("苍井");
	//cout << "情人： " << p.m_Lover << endl;  //只写属性，不可以读取

	system("pause");

	return 0;
}
```



#### 对象的初始化和清理

* 构造函数：主要作用在于创建对象时为对象的成员属性赋值，构造函数由编译器自动调用，无须手动调用。
* 析构函数：主要作用在于对象**销毁前**系统自动调用，执行一些清理工作。



**构造函数语法：**`类名(){}`

1. 构造函数，**没有返回值也不写void**
2. **函数名称与类名相同**
3. 构造函数可以有参数，因此可以发生重载
4. 程序在调用对象时候会自动调用构造，无须手动调用,而且只会调用一次

**析构函数语法：** `~类名(){}`

1. 析构函数，没有返回值也不写void
2. 函数名称与类名相同,在名称前加上符号  ~
3. 析构函数不可以有参数，因此不可以发生重载
4. 程序在对象销毁前会自动调用析构，无须手动调用,而且只会调用一次

```C++
class Person
{
public:
	//构造函数
	Person()
	{
		cout << "Person的构造函数调用" << endl;
	}
	//析构函数
	~Person()
	{
		cout << "Person的析构函数调用" << endl;
	}

};

void test01()
{
	Person p;
}

int main() {
	
	test01();

	system("pause");

	return 0;
}
```



**初始化列表**

**作用：**

C++提供了初始化列表语法，用来初始化属性

**语法：**`构造函数()：属性1(值1),属性2（值2）... {}`

```C++

class Person {
public:

	////传统方式初始化
	//Person(int a, int b, int c) {
	//	m_A = a;
	//	m_B = b;
	//	m_C = c;
	//}

	//初始化列表方式初始化
	Person(int a, int b, int c) :m_A(a), m_B(b), m_C(c) {}
	void PrintPerson() {
		cout << "mA:" << m_A << endl;
		cout << "mB:" << m_B << endl;
		cout << "mC:" << m_C << endl;
	}
private:
	int m_A;
	int m_B;
	int m_C;
};

int main() {

	Person p(1, 2, 3);
	p.PrintPerson();


	system("pause");

	return 0;
}
```



静态成员就是在成员变量和成员函数前加上关键字static，称为静态成员

静态成员分为：

*  静态成员变量
   *  所有对象共享同一份数据
   *  在编译阶段分配内存
   *  类内声明，类外初始化
*  静态成员函数
   *  所有对象共享同一个函数
   *  静态成员函数只能访问静态成员变量

```C++
class Person
{
	
public:

	static int m_A; //静态成员变量

	//静态成员变量特点：
	//1 在编译阶段分配内存
	//2 类内声明，类外初始化
	//3 所有对象共享同一份数据

private:
	static int m_B; //静态成员变量也是有访问权限的
};

// 类外初始化
int Person::m_A = 10;
int Person::m_B = 10;

void test01()
{
	//静态成员变量两种访问方式

	//1、通过对象
	Person p1;
	p1.m_A = 100;
	cout << "p1.m_A = " << p1.m_A << endl;

	Person p2;
	p2.m_A = 200;
	cout << "p1.m_A = " << p1.m_A << endl; //共享同一份数据
	cout << "p2.m_A = " << p2.m_A << endl;

	//2、通过类名
	cout << "m_A = " << Person::m_A << endl;


	//cout << "m_B = " << Person::m_B << endl; //私有权限访问不到
}

int main() {

	test01();

	system("pause");

	return 0;
}
```





### 5-2-5  C++对象模型和this指针

#### 成员变量和成员函数分开存储

在C++中，类内的成员变量和成员函数分开存储

只有非静态成员变量才属于类的对象上

```C++
class Person {
public:
	Person() {
		mA = 0;
	}
	//非静态成员变量占对象空间
	int mA;
	//静态成员变量不占对象空间
	static int mB; 
	//函数也不占对象空间，所有函数共享一个函数实例
	void func() {
		cout << "mA:" << this->mA << endl;
	}
	//静态成员函数也不占对象空间
	static void sfunc() {
	}
};

int main() {

	cout << sizeof(Person) << endl;

	system("pause");

	return 0;
}
```



#### this指针概念

每一个非静态成员函数只会诞生一份函数实例，也就是说多个同类型的对象会共用一块代码

那么问题是：这一块代码是如何区分那个对象调用自己的呢？

c++通过提供特殊的对象指针，this指针，解决上述问题。**this指针指向被调用的成员函数所属的对象**

this指针是隐含每一个非静态成员函数内的一种指针

this指针不需要定义，直接使用即可



this指针的用途：

*  当形参和成员变量同名时，可用this指针来区分
*  在类的非静态成员函数中返回对象本身，可使用return *this



this指针指向的是被调用的成员所属的对象

下面这个为例：

当代码到``Person p1(10)``时，调用了``Person(int age)``这个有参构造函数，``this``指针指向的是被调用的成员函数``p1``,即``p1``在调用这个函数，指向的就是``p1``。

```cpp
class Person
{
public:

	Person(int age)
	{
		//1、当形参和成员变量同名时，可用this指针来区分
		this->age = age;
	}

	int age;
};

void test01()
{
	Person p1(10);
	cout << "p1.age = " << p1.age << endl;
}

int main() {

	test01();

	system("pause");

	return 0;
}
```



```cpp
class Person
{
public:

	Person(int age)
	{
		//1、当形参和成员变量同名时，可用this指针来区分
		this->age = age;
	}

	Person& PersonAddPerson(Person p)
	{
		this->age += p.age;
		//返回对象本身
		return *this;
	}

	int age;
};

void test01()
{
	Person p1(10);
	cout << "p1.age = " << p1.age << endl;

	Person p2(10);
	p2.PersonAddPerson(p1).PersonAddPerson(p1).PersonAddPerson(p1);
	cout << "p2.age = " << p2.age << endl;
}

int main() {

	test01();

	system("pause");

	return 0;
}
```



#### const修饰成员函数

**常函数：**

* 成员函数后加const后我们称为这个函数为**常函数**
* 常函数内不可以修改成员属性
* 成员属性声明时加关键字mutable后，在常函数中依然可以修改

**常对象：**

* 声明对象前加const称该对象为常对象
* 常对象只能调用常函数



```cpp
class Person {
public:
	Person() {
		m_A = 0;
		m_B = 0;
	}

	//this指针的本质是一个指针常量，指针的指向不可修改
	//如果想让指针指向的值也不可以修改，需要声明常函数
	void ShowPerson() const {
		//const Type* const pointer;
		//this = NULL; //不能修改指针的指向 Person* const this;
		//this->mA = 100; //但是this指针指向的对象的数据是可以修改的

		//const修饰成员函数，表示指针指向的内存空间的数据不能修改，除了mutable修饰的变量
		this->m_B = 100;
	}

	void MyFunc() const {
		//mA = 10000;
	}

public:
	int m_A;
	mutable int m_B; //可修改 可变的
};


//const修饰对象  常对象
void test01() {

	const Person person; //常量对象  
	cout << person.m_A << endl;
	//person.mA = 100; //常对象不能修改成员变量的值,但是可以访问
	person.m_B = 100; //但是常对象可以修改mutable修饰成员变量

	//常对象访问成员函数
	person.MyFunc(); //常对象不能调用const的函数

}

int main() {

	test01();

	system("pause");

	return 0;
}
```



#### 继承

<img src="https://gitee.com/zhangchenxuv/images/raw/main/image/image-20240831212350999.png" alt="image-20240831212350999" style="zoom:50%;" />

定义这些类时，下级别的成员除了拥有上一级的共性，还有自己的特性，这个时候我们就可以考虑利用继承的技术，减少重复代码。

**继承的基本语法**

​	**普通实现：**

```cpp
//Java页面
class Java 
{
public:
	void header()
	{
		cout << "首页、公开课、登录、注册...（公共头部）" << endl;
	}
	void footer()
	{
		cout << "帮助中心、交流合作、站内地图...(公共底部)" << endl;
	}
	void left()
	{
		cout << "Java,Python,C++...(公共分类列表)" << endl;
	}
	void content()
	{
		cout << "JAVA学科视频" << endl;
	}
};
//Python页面
class Python
{
public:
	void header()
	{
		cout << "首页、公开课、登录、注册...（公共头部）" << endl;
	}
	void footer()
	{
		cout << "帮助中心、交流合作、站内地图...(公共底部)" << endl;
	}
	void left()
	{
		cout << "Java,Python,C++...(公共分类列表)" << endl;
	}
	void content()
	{
		cout << "Python学科视频" << endl;
	}
};
//C++页面
class CPP 
{
public:
	void header()
	{
		cout << "首页、公开课、登录、注册...（公共头部）" << endl;
	}
	void footer()
	{
		cout << "帮助中心、交流合作、站内地图...(公共底部)" << endl;
	}
	void left()
	{
		cout << "Java,Python,C++...(公共分类列表)" << endl;
	}
	void content()
	{
		cout << "C++学科视频" << endl;
	}
};

void test01()
{
	//Java页面
	cout << "Java下载视频页面如下： " << endl;
	Java ja;
	ja.header();
	ja.footer();
	ja.left();
	ja.content();
	cout << "--------------------" << endl;

	//Python页面
	cout << "Python下载视频页面如下： " << endl;
	Python py;
	py.header();
	py.footer();
	py.left();
	py.content();
	cout << "--------------------" << endl;

	//C++页面
	cout << "C++下载视频页面如下： " << endl;
	CPP cp;
	cp.header();
	cp.footer();
	cp.left();
	cp.content();

}

int main() {

	test01();

	system("pause");

	return 0;
}
```

​	**继承实现：**

```cpp
//公共页面
class BasePage
{
public:
	void header()
	{
		cout << "首页、公开课、登录、注册...（公共头部）" << endl;
	}

	void footer()
	{
		cout << "帮助中心、交流合作、站内地图...(公共底部)" << endl;
	}
	void left()
	{
		cout << "Java,Python,C++...(公共分类列表)" << endl;
	}

};

//Java页面
class Java : public BasePage
{
public:
	void content()
	{
		cout << "JAVA学科视频" << endl;
	}
};
//Python页面
class Python : public BasePage
{
public:
	void content()
	{
		cout << "Python学科视频" << endl;
	}
};
//C++页面
class CPP : public BasePage
{
public:
	void content()
	{
		cout << "C++学科视频" << endl;
	}
};

void test01()
{
	//Java页面
	cout << "Java下载视频页面如下： " << endl;
	Java ja;
	ja.header();
	ja.footer();
	ja.left();
	ja.content();
	cout << "--------------------" << endl;

	//Python页面
	cout << "Python下载视频页面如下： " << endl;
	Python py;
	py.header();
	py.footer();
	py.left();
	py.content();
	cout << "--------------------" << endl;

	//C++页面
	cout << "C++下载视频页面如下： " << endl;
	CPP cp;
	cp.header();
	cp.footer();
	cp.left();
	cp.content();
}

int main() {

	test01();

	system("pause");

	return 0;
}
```

**继承方式一共有三种：**

* 公共继承
* 保护继承
* 私有继承

<img src="https://gitee.com/zhangchenxuv/images/raw/main/image/image-20240831215727196.png" alt="image-20240831215727196" style="zoom:50%;" />

```cpp
class Base1
{
public: 
	int m_A;
protected:
	int m_B;
private:
	int m_C;
};

//公共继承
class Son1 :public Base1
{
public:
	void func()
	{
		m_A; //可访问 public权限
		m_B; //可访问 protected权限
		//m_C; //不可访问
	}
};

void myClass()
{
	Son1 s1;
	s1.m_A; //其他类只能访问到公共权限
}

//保护继承
class Base2
{
public:
	int m_A;
protected:
	int m_B;
private:
	int m_C;
};
class Son2:protected Base2
{
public:
	void func()
	{
		m_A; //可访问 protected权限
		m_B; //可访问 protected权限
		//m_C; //不可访问
	}
};
void myClass2()
{
	Son2 s;
	//s.m_A; //不可访问
}

//私有继承
class Base3
{
public:
	int m_A;
protected:
	int m_B;
private:
	int m_C;
};
class Son3:private Base3
{
public:
	void func()
	{
		m_A; //可访问 private权限
		m_B; //可访问 private权限
		//m_C; //不可访问
	}
};
class GrandSon3 :public Son3
{
public:
	void func()
	{
		//Son3是私有继承，所以继承Son3的属性在GrandSon3中都无法访问到
		//m_A;
		//m_B;
		//m_C;
	}
};
```



# 六、Ros2_contral

## 6-1 概论

官方文档：https://control.ros.org/master/doc/getting_started/getting_started.html

**ros2_contral作用：**

作为ROS2和实体机器人的中间件控制实体机器人

**主要概念**

controller_manager

controller

hardware_interface

resource_manger

## 6-2 调用controllers和hardware_interfaces

下载该项目：

https://github.com/WMGIII/bbot_demo

在工作空间下编译，如果报错

![image-20240926221729008](https://gitee.com/zhangchenxuv/images/raw/main/image/image-20240926221729008.png)

则下载：

```bash 
sudo apt-get install ros-humble-effort-controllers
```

之后运行：

```bash
ros2 launch bbot_description  bbot.launch.py
```

按照下图配置并加入插件：

![image-20240926224112988](https://gitee.com/zhangchenxuv/images/raw/main/image/image-20240926224112988.png)

可以得到：

<img src="https://gitee.com/zhangchenxuv/images/raw/main/image/image-20240926224153991.png" alt="image-20240926224153991" style="zoom:50%;" />

**之后使用ros2_contral**框架构建**gazebo**下的模型

可以参考：https://github.com/ros-controls/roadmap/blob/master/design_drafts/hardware_access.md

**--相关概念--**

在ROS 2中，硬件资源（hardware resources）是物理硬件的抽象，它们实现了与物理硬件的通信，并作为插件被导出。硬件资源可以通过URDF文件来配置，并且在运行时被动态加载，从而允许灵活和动态地组合要控制的设置。

硬件资源主要有三种类型：

1. **Actuator（执行器）**：简单的1自由度（1 DOF）机器人硬件，如电机或阀门。
2. **System（系统）**：复杂的多自由度（multi-DOF）机器人硬件，如工业机器人。
3. **Sensor（传感器）**：用于感知环境的机器人硬件，可以与关节（例如，编码器）或链接（例如，力-扭矩传感器）相关。

执行器和系统组件具有读写能力（使用joint接口），而传感器组件仅具有读取能力（使用sensor接口）。

```xml
<ros2_control name="my_simple_servo_motor" type="actuator">
  <hardware>
    <class>simple_servo_motor_pkg/SimpleServoMotor</class>
    <param name="serial_port">/dev/tty0</param>
    <!-- ... -->
  </hardware>
  <joint name="joint1">
    <command_interface name="position">
      <param name="min">-1.57</param>
      <param name="max">1.57</param>
    </command>
    <state_interface name="position"/>
    <!-- ... -->
  </joint>
</ros2_control>
```

在上述示例中，`<ros2_control>`标签定义了硬件资源的名称和类型，`<hardware>`标签指定了硬件接口的插件和参数，而`<joint>`标签定义了与执行器相关联的关节，包括命令接口和状态接口。

硬件资源的管理是通过资源管理器（Resource Manager）进行的，它使用pluginlib库动态加载组件，并管理它们的生命周期和状态。在控制循环执行中，资源管理器的`read()`和`write()`方法处理与硬件组件的通信。

之后在URDF目录下创建``bbot.ros2_control.xacro``

```xaml
<?xml version="1.0"?>
<robot xmlns:xacro="test">
    <xacro:macro name="bbot_ros2_control">
        <ros2_control name="bbot_hardware_interface" type="system">
            <!-- 在声明hardware_resource时需要在此处声明 -->
             <hardware>
                <plugin>gazebo_ros2_control/GazeboSystem</plugin>
             </hardware>

             <joint name="left_wheel_joint">
                <command_interface name="velocity">
                    <param name="min">-10</param>
                    <param name="max">10</param>                
                </command_interface>


                <state_interface name="position"/>
                <state_interface name="velocity"/>

             </joint>

             <joint name="right_wheel_joint">
                <command_interface name="velocity">
                    <param name="min">-10</param>
                    <param name="max">10</param>
                </command_interface>


                <state_interface name="position"/>
                <state_interface name="velocity"/>

             </joint>

        </ros2_control>

        <!-- 使用gazebo -->
        <gazebo>
        <plugin filename="libgazebo_ros2_control.so" name="gazebo_ros2_control">
            <parameters>$(find bbot_bringup)/config/bbot_controllers.yaml</parameters>
        </plugin>
        </gazebo>
    </xacro:macro>
    
</robot>

```



另外创建一个``bbot_bringup``文件夹，其中创建``config``和``launch``文件夹

``config``文件夹下创建``bbot_controllers.yaml``文件

```yaml
controller_manager:
  ros__parameters:
    update_rate: 30
    use_sim_time: true

    # 自定义的参数
    diff_drive:
      type: diff_drive_controller/DiffDriveController

    joint_state:
      type: joint_state_broadcaster/JointStateBroadcaster


diff_drive:
  ros__parameters:
    publish_rate: 50.0
    base_frame_id: base_link

    left_wheel_names: ['left_wheel_joint']
    right_wheel_names: ['right_wheel_joint']
    # 轮距
    wheel_separation: 0.35
    # 轮子的半径
    wheel_radius: 0.05

    use_stamped_vel: false

```



``launch``文件夹下创建``bbot_bringup.launch.py``文件

```python
import os
# ---|获取功能包下 share 目录路径
from ament_index_python.packages import get_package_share_directory
from launch import LaunchDescription
from launch_ros.actions import Node
# ---|封装终端指令相关类
# from launch.actions import ExecuteProcess
# from launch.substitutions import FindExecutable
# ---|参数声明与获取
# from launch.actions import DeclareLaunchArgument
# from launch.substitutions import LaunchConfiguration
# ---|文件包含相关
from launch.actions import IncludeLaunchDescription
from launch.launch_description_sources import PythonLaunchDescriptionSource
# ---|分组相关
# from launch_ros.actions import PushRosNamespace
# from launch.actions import GroupAction
# ---|事件相关
# from launch.event_handlers import OnProcessStart, OnProcessExit
# from launch.actions import ExecuteProcess, RegisterEventHandler,LogInfo


def generate_launch_description():
    package_name='bbot_description'

    bbot = IncludeLaunchDescription(
                PythonLaunchDescriptionSource([os.path.join(
                    get_package_share_directory(package_name), 'launch', 'bbot.launch.py'    
                    )]), launch_arguments={'use_sim_time': 'true'}.items()
            )

    gazebo = IncludeLaunchDescription(
                PythonLaunchDescriptionSource([os.path.join(
                    get_package_share_directory('gazebo_ros'), 'launch', 'gazebo.launch.py'
                )]),
            )
    spawn_entity = Node(
        package='gazebo_ros', 
        executable='spawn_entity.py',
        arguments=['-topic', 'robot_description', '-entity', 'bbot'],
        output='screen')
    
    joint_broad_spawner = Node(
        package="controller_manager",
        executable="spawner",
        arguments=["joint_state"],
    )
    diff_drive_spawner = Node(
        package="controller_manager",
        executable="spawner",
        arguments=["diff_drive"],
    )
    
    return LaunchDescription([
        bbot,
        gazebo,
        spawn_entity,
        diff_drive_spawner,
        joint_broad_spawner,
    ])
```



编译后启动``launch``文件,如果报错，或者显示无法连接`/controller_manager/list_controllers`服务，则安装以下功能包：

```bash
sudo apt-get install ros-humble-controller-manager
sudo apt install ros-humble-ros2-control
sudo apt-get install ros-humble-gazebo-ros2-control
sudo apt install ros-humble-gazebo-ros-pkgs
```



之后运行

```bash
ros2 control list_hardware_interfaces
```

可以查看所有的``hardware_interfaces``

<img src="https://gitee.com/zhangchenxuv/images/raw/main/image/image-20241006181914872.png" alt="image-20241006181914872" style="zoom: 67%;" />

如果日志中提示：

<img src="https://gitee.com/zhangchenxuv/images/raw/main/image/image-20241006210854519.png" alt="image-20241006210854519" style="zoom:67%;" />

1. **控制器加载失败**：

   - `[ERROR] [controller_manager]: Loader for controller 'diff_drive' (type 'diff_drive_controller/DiffDriveController') not found.`
   - `[ERROR] [controller_manager]: Loader for controller 'joint_state' (type 'joint_state_broadcaster/JointStateBroadcaster') not found.`

   这表明ROS 2无法找到`diff_drive_controller`和`joint_state_broadcaster`控制器的加载器。这通常是因为没有安装相应的ROS 2包或者环境没有配置正确。

2. **过时的QoS设置**：

   - `[spawn_entity.py-6] /opt/ros/humble/local/lib/python3.10/dist-packages/rclpy/qos.py:307: UserWarning: DurabilityPolicy.RMW_QOS_POLICY_DURABILITY_TRANSIENT_LOCAL is deprecated. Use DurabilityPolicy.TRANSIENT_LOCAL instead.`

   这是一个警告，表明你使用的QoS设置已经过时，建议更新为新的设置。

3. **控制器管理器错误**：

   - `[FATAL] [spawner_diff_drive]: Failed loading controller diff_drive`
   - `[FATAL] [spawner_joint_state]: Failed loading controller joint_state`

   这些错误表明控制器加载失败，导致进程退出。

   

   ### 解决方案：

   1. **检查ROS 2包是否安装**： 确保你已经安装了`diff_drive_controller`和`joint_state_broadcaster`包。如果没有安装，可以使用以下命令安装：

      bash

      ```bash
      sudo apt install ros-humble-diff-drive-controller
      sudo apt install ros-humble-joint-state-broadcaster
      ```

   2. **更新QoS设置**： 将代码中的`DurabilityPolicy.RMW_QOS_POLICY_DURABILITY_TRANSIENT_LOCAL`替换为`DurabilityPolicy.TRANSIENT_LOCAL`。

   3. **检查环境配置**： 确保你的环境变量和工作空间配置正确。你可以使用以下命令来检查：

      bash

      ```bash
      source /opt/ros/humble/setup.bash
      source ~/ros2_ws/install/setup.bash
      ```

   4. **检查控制器配置文件**： 确保你的控制器配置文件（如`bbot_controllers.yaml`）中的控制器类型和名称正确。

   5. **重新启动ROS 2节点**： 在解决了上述问题后，重新启动ROS 2节点并再次尝试启动你的项目。

   

**使用键盘控制**

```bash
ros2 run teleop_twist_keyboard teleop_twist_keyboard --ros-args -r /cmd_vel:=/diff_drive/cmd_vel_unstamped
```

命令解析如下：

- `ros2 run`：这是ROS 2中用于运行节点的命令。
- `teleop_twist_keyboard`：这是你要运行的包的名称。
- `teleop_twist_keyboard`：这是你要运行的可执行文件（节点）的名称。
- `--ros-args`：这个选项后面跟着的是ROS 2参数，这些参数将被传递给节点。
- `-r /cmd_vel:=/diff_drive/cmd_vel_unstamped`：这是重映射参数，它将节点内部的主题`/cmd_vel`重映射为`/diff_drive/cmd_vel_unstamped`。这意味着节点将监听`/diff_drive/cmd_vel_unstamped`主题上的订阅者，并将键盘输入的命令发布到这个主题上。

<img src="https://gitee.com/zhangchenxuv/images/raw/main/image/image-20241006212815272.png" alt="image-20241006212815272" style="zoom:50%;" />

- **移动控制**：
  - `u` `i` `o`：分别控制机器人向前、向后和侧向移动。
  - `j` `k` `l`：分别控制机器人向左、向前和向右移动。
  - `m` `,` `.`：在这些键的控制下，机器人会执行更细微的移动。
- **全向移动模式（Holonomic mode）**：
  - 当按住`shift`键时，可以使用大写字母`U` `I` `O` `J` `K` `L` `M` `<` `>`来实现全向移动，也就是可以在不改变方向的情况下向任何方向平移。
- **垂直移动**：
  - `t`：使机器人向上移动（+z轴方向）。
  - `b`：使机器人向下移动（-z轴方向）。
- **停止**：
  - 按除了上述键之外的任何键都会使机器人停止移动。
- **调整速度**：
  - `q` `z`：分别增加或减少最大速度10%。
  - `w` `x`：分别增加或减少只有线性速度10%。
  - `e` `c`：分别增加或减少只有角速度10%。

**总结**

第一步：

定义``hardware_resource``，以及其中的``interface``代码，即文件``bbot.ros2_control.xacro``，并且添加进gazebo模型中去

第二步：

定义了``controller_manager``的参数文件，即文件``bbot_controllers.yaml``

第三步：

编写launch文件，添加需要控制的controller，即``bbot_bringup.launch.py``下的：

```python
......
    joint_broad_spawner = Node(
        package="controller_manager",
        executable="spawner",
        arguments=["joint_state"],
    )
    diff_drive_spawner = Node(
        package="controller_manager",
        executable="spawner",
        arguments=["diff_drive"],
......
```



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

## 6-4 准备rrbot

创建工作空间``rrcot_ws``

下载``https://github.com/ros-controls/roscon2022_workshop/tree/7-robot-hardware-interface/solution``中的项目

运行：

```bash
. install/setup.bash
ros2 launch controlko_bringup rrbot_sim_gazebo_classic.launch.py
```

可能

<img src="https://gitee.com/zhangchenxuv/images/raw/main/image/image-20241007171340397.png" alt="image-20241007171340397" style="zoom: 67%;" />

```bash
sudo apt-get install ros-kinetic-joint-trajectory-controller
```

<img src="https://gitee.com/zhangchenxuv/images/raw/main/image/image-20241007172020182.png" alt="image-20241007172020182" style="zoom:50%;" />

之后运行

```bash
ros2 launch controlko_bringup test_joint_trajectory_controller.launch.py 
```

如果报错

![image-20241007172102489](https://gitee.com/zhangchenxuv/images/raw/main/image/image-20241007172102489.png)

安装：

```bash
sudo apt install ros-humble-ros2-controllers-test-nodes
```

模型开始运动：

<img src="https://gitee.com/zhangchenxuv/images/raw/main/image/image-20241007172241820.png" alt="image-20241007172241820" style="zoom: 50%;" />

第二个即是创建控制位置的控制器：

![image-20241007172528985](https://gitee.com/zhangchenxuv/images/raw/main/image/image-20241007172528985.png)

可在下图中看到``command interfaces``中有速度和位置，``interfaces``中有力、位置和速度

<img src="https://gitee.com/zhangchenxuv/images/raw/main/image/image-20241007172631700.png" alt="image-20241007172631700" style="zoom:80%;" />

## 6-5 Writing a new hardware interface

官方文档：https://control.ros.org/master/doc/ros2_control/hardware_interface/doc/writing_new_hardware_component.html

下载一个模板：

```bash
git clone https://github.com/StoglRobotics/ros_team_workspace.git
```

**编写一个简单的硬件接口（hardware interface）**

首先创建一个新的工作空间

```bash
zhangchenxu@chengxz-pc:~/Documents/ros2_control/rrbot_ws/src$ ros2 pkg create my_hardware_interface --build-type ament_cmake 
```

打开文档

```bash
https://rtw.stoglrobotics.de/master/use-cases/ros2_control/setup_controller.html
```

<img src="https://gitee.com/zhangchenxuv/images/raw/main/image/image-20241009174541308.png" alt="image-20241009174541308" style="zoom:67%;" />

下面这个命令是生成hardware interface模板的命令

```bash
ros2_control_setup-controller-package FILE_NAME [CLASS_NAME]
```

``FILE_NAME``：指定cpp文件名

``[CLASS_NAME]``：实际的类名

之后安装环境

```bash
zhangchenxu@chengxz-pc:~/Documents/ros2_control/rrbot_ws/src$ cd ..
zhangchenxu@chengxz-pc:~/Documents/ros2_control/rrbot_ws$ . install/setup.bash 
zhangchenxu@chengxz-pc:~/Documents/ros2_control/rrbot_ws$ cd ..
zhangchenxu@chengxz-pc:~/Documents/ros2_control$ cd ..
zhangchenxu@chengxz-pc:~/Documents$ cd ros
ros2_can/           ros2_mc/            ros2_ws/
ros2_control/       ros2_mcws/          ros_team_workspace/
zhangchenxu@chengxz-pc:~/Documents$ cd ros
ros2_can/           ros2_mc/            ros2_ws/
ros2_control/       ros2_mcws/          ros_team_workspace/
zhangchenxu@chengxz-pc:~/Documents$ cd ros_team_workspace/
zhangchenxu@chengxz-pc:~/Documents/ros_team_workspace$ . setup.bash 
zhangchenxu@chengxz-pc:~/Documents/ros_team_workspace$ cd ..
zhangchenxu@chengxz-pc:~/Documents$ cd ros2_control/rrbot_ws/
zhangchenxu@chengxz-pc:~/Documents/ros2_control/rrbot_ws$ cd src
zhangchenxu@chengxz-pc:~/Documents/ros2_control/rrbot_ws/src$ cd my_hardware_interface/
```

生成

```bash
zhangchenxu@chengxz-pc:~/Documents/ros2_control/rrbot_ws/src/my_hardware_interface$ ros2_control_setup-hardware-interface-package rrbot_harware_interface RRBotHardwareInterface

Which license-header do you want to use? [1]
(0) None
(1) Apache 2.0 License
(2) Proprietary
1
Insert your company or personal name (copyright): zhang
Which type of ros2_control hardware interface you want to extend? [0]
(0) system
(1) sensor
(2) actuator
0

ATTENTION: Setting up ros2_control hardware interface files with following parameters: file name 'rrbot_harware_interface', class 'RRBotHardwareInterface', package/namespace 'my_hardware_interface' for interface type 'system'. Those will be placed in folder '/home/zhangchenxu/Documents/ros2_control/rrbot_ws/src/my_hardware_interface'.

If correct press <ENTER>, otherwise <CTRL>+C and start the script again from the package folder and/or with correct robot name.

Template files copied.

Template files were adjusted.
fatal: 不是 git 仓库（或者任何父目录）：.git
Can not compile: No ROS_WS variable set. Trying to guess by sourced workspace.
Is "/home/zhangchenxu/Documents/ros2_control/rrbot_ws the correct sourced workspace? [yes no]
yes
Starting >>> my_hardware_interface
Finished <<< my_hardware_interface [5.32s]                     

Summary: 1 package finished [5.66s]
/home/zhangchenxu/Documents/ros2_control/rrbot_ws
Starting >>> my_hardware_interface
--- stderr: my_hardware_interface                   
Errors while running CTest
Output from these tests are in: /home/zhangchenxu/Documents/ros2_control/rrbot_ws/build/my_hardware_interface/Testing/Temporary/LastTest.log
Use "--rerun-failed --output-on-failure" to re-run the failed cases verbosely.
---
Finished <<< my_hardware_interface [3.63s]	[ with test failures ]

Summary: 1 package finished [3.86s]
  1 package had stderr output: my_hardware_interface
  1 package had test failures: my_hardware_interface
/home/zhangchenxu/Documents/ros2_control/rrbot_ws
build/my_hardware_interface/Testing/20241009-1154/Test.xml: 5 tests, 0 errors, 1 failure, 0 skipped
build/my_hardware_interface/test_results/my_hardware_interface/cppcheck.xunit.xml: 4 tests, 0 errors, 0 failures, 4 skipped
build/my_hardware_interface/test_results/my_hardware_interface/lint_cmake.xunit.xml: 1 test, 0 errors, 0 failures, 0 skipped
build/my_hardware_interface/test_results/my_hardware_interface/test_rrbot_harware_interface.gtest.xml: 1 test, 0 errors, 0 failures, 0 skipped
build/my_hardware_interface/test_results/my_hardware_interface/uncrustify.xunit.xml: 4 tests, 0 errors, 2 failures, 0 skipped
build/my_hardware_interface/test_results/my_hardware_interface/xmllint.xunit.xml: 2 tests, 0 errors, 0 failures, 0 skipped

FINISHED: Your package is set and the tests should be finished without any errors. (linter errors possible!)

```









# 七、Navigation2

官方文档：https://fishros.org/doc/nav2/getting_started/index.html

参考：https://blog.csdn.net/l961983207/article/details/134300707#:~:text=%E6%9C%AC%E6%96%87%E8%AF%A6%E7%BB%86%E4%BB%8B%E7%BB%8D%E4%BA%86%E5%9C%A8%E4%B8%8D%E5%90%8C

安装：

```bash
sudo apt install ros-humble-navigation2
sudo apt install ros-humble-nav2-bringup
sudo apt install ros-humble-turtlebot3-gazebo
```





# 附录

## 附录一 C++相关

### |-1 C++对于C的拓展

#### |-1-2命名空间

``::``称为``作用域操作符``

``空间名称``::``成员``

在C++中，命名空间（Namespace）是一种将程序中的实体（例如变量、函数、类等）组织在一起的方法，以避免命名冲突。命名空间是一个声明性区域，它提供了一个作用域，在这个作用域内可以定义变量、函数、类等，而且这些定义不会与同一程序中其他命名空间中的定义冲突。

**命名空间别名**

```c++
namespace foo = original_namespace;
```

**几个例子**

```c++
#include <iostream>

using namespace std;

//声明一个命名空间
namespace namea
{
	int x;
	void func()
	{
		cout << "this is namea" << endl;
	}

}

int main() {
	//访问空间中的成员变量
	namea::x = 1000;
	cout << "namea" << endl;
	//访问空间中的函数
	namea::func();
}
```

**注意：**1、命名空间只能全局定义，不能再函数中定义

​           2、命名空间可以嵌套定义

**当命名空间过多时，会将其放入单独的``.h``文件中**

```C++
// 预处理，防止头文件被重复包含


#ifndef TEST_H
#define TEST_H

#endif
```

**整体如下**

``.h``文件

```c++
#ifndef TEST_H
#define TEST_H

// 声明命名空间
namespace myspace
{
	void func1(int);
}

#endif

```

``.h``文件只是对命名空间进行声明，需要另外新建一个c++文件对函数进行实现

```c++
#include "test.h"
#include <iostream>

using namespace std;

// 实现函数
void myspace::func1(int x)
{
	cout << "void myspace::func1()" << x << endl;
}
```

这样便可以在``main``函数中实现

```c++
#include <iostream>
#include "test.h"

using namespace std;

int main()
{	
	myspace::func1(100000);
}
```

<img src="C:\Users\22857\AppData\Roaming\Typora\typora-user-images\image-20240812010750653.png" alt="image-20240812010750653" style="zoom: 67%;" />



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



## 附录三 Ros常用消息

ROS常用message消息数据 https://mp.weixin.qq.com/s/77Nc7jDjqDDgv987AoQWXQ

从零入门激光SLAM（八）——ROS常用消息 https://blog.csdn.net/HUASHUDEYANJING/article/details/130323325

ROS常用消息https://blog.csdn.net/xhtchina/article/details/119707553



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

<img src="/home/zhangchenxu/.config/Typora/typora-user-images/image-20240818012725059.png" alt="image-20240818012725059" style="zoom:50%;" />

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





## 附录五 什么是ip地址？子关掩码？ip地址段

形式是``x.x.x.x``，每个数字的范围是0-255

即范围是0.0.0.0 - 255.255.255.255

**子网掩码，默认网关。DNS都是干什么的**

``子网掩码``:

划分网段

Ip地址也分段，如何判断访问的目标ip是否同网段？看网络位，ip地址的前多少位是判断该ip在哪个网段。

如：ip:``192.268.1.1`` 和 掩码：`` 255.255.255.0``

从掩码可以看出前三位是网络位，及192.268.1.2也与上述ip同网段

掩码的作用是确认一个ip所在的网段，有了掩码就知道了网络位，进而判断ip间是否同网段



``默认网关``:

访问目标时，访问同网段目标，用一种通信方式，访问不同网段的目标，用另一种通信方式

**访问同网段目标**：直接发数据包--直接通讯

**访问不同网段的目标**：必须找中间人，由中间人做数据转发。中间人即网关（路由器）。 

​								路由功能：帮助不同网段进行数据转发功能

<img src="https://gitee.com/zhangchenxuv/images/raw/main/image/image-20240910232345249.png" alt="image-20240910232345249" style="zoom:33%;" />

``DNS``：

将域名解析成ip

网址既是域名，但实际上域名均是ip地址

DNS可以实现通过域名来访问服务器

实际使用中需要一个服务器来将域名解析成为ip地址，这个服务器的ip地址既是DNS服务器的地址

### |-5-1 ROS2设备间的多机有线通讯

首先需要保证设备处于同一局域网下，且需要处于同一网段下（网络位相同且子网掩码一致）

首先将一台设备的Ip地址设置成如下形式：

<img src="https://gitee.com/zhangchenxuv/images/raw/main/image/image-20240912231526778.png" alt="image-20240912231526778" style="zoom: 67%;" />

另一台设备的Ip地址需按照其网络位及子网掩码进行相同配置

<img src="https://gitee.com/zhangchenxuv/images/raw/main/image/image-20240912231637716.png" alt="image-20240912231637716" style="zoom: 67%;" />

此时两机间便可以进行通讯

<img src="https://gitee.com/zhangchenxuv/images/raw/main/image/image-20240912232303844.png" alt="image-20240912232303844" style="zoom: 67%;" />

<img src="https://gitee.com/zhangchenxuv/images/raw/main/image/image-20240912232325576.png" alt="image-20240912232325576" style="zoom: 67%;" />

如果在同一局域网中存在多台运行ROS2的设备，且并不希望计算机之间存在干扰，需要通过配置各自的域ID实现

不同的DOMAIN ID之间是通过不同端口进行组播发现，由于端口数量有限，因此DOMAIN ID的安全范围在**0~101**之间（默认为0）

通过DOMAIN ID实现隔离：

![image-20240912232930077](https://gitee.com/zhangchenxuv/images/raw/main/image/image-20240912232930077.png)

![image-20240912232945504](https://gitee.com/zhangchenxuv/images/raw/main/image/image-20240912232945504.png)

因此需要配置

```shell
export ROS_DOMAIN_ID=2
```

来实现通过DOMAIN ID实现隔离的作用



### |-5-2 ROS2设备间的多机无线通讯

需要通过路由设备实现，大致内容同有线通讯，但网关需要配置为路由所对应的设备ip。







## 附录六 传感器的使用

### |-6-1 编码器

#### |-6-1-1 参考资料

ubuntu cutecom串口调试工具使用方法（图形界面）https://blog.csdn.net/Dontla/article/details/134557362

C++使用serial串口通信 + ROS2示例IMU串口驱动https://blog.csdn.net/zardforever123/article/details/134227412

Ubuntu22.04下ROS2 Humble串口通信https://blog.csdn.net/qq_50972633/article/details/132837550

ROS2环境下的串口通讯https://blog.csdn.net/weixin_53035484/article/details/128135356

serial库 http://wjwwood.io/serial/doc/1.1.0/classserial_1_1_serial.html

#### |-6-1-2 可能遇到的问题

https://blog.csdn.net/qq_50972633/article/details/132837550

##### |-2-1找不到libserial.so动态链接库

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



##### |-2-2 程序可以编译，但端口无法打开

<img src="/home/zhangchenxu/Pictures/Typora/image-20240809221925321.png" alt="image-20240809221925321" style="zoom:50%;" />

**（1）检查串口是否存在，是否被占用**

```
ls -l /dev/ttyUSB0
```

**（2）开放串口权限**

```
sudo chmod 777 /dev/ttyUSB0
```



##### |-2-3 串口驱动报 Key was rejected by service 需要签名的问题

https://blog.csdn.net/qq_28680277/article/details/129162559



##### |-2-4 Brltty 导致 USB 转串口连接失败

https://blog.csdn.net/qq_27865227/article/details/125538516



#### |-6-1-3 实例

##### |-3-1 实例1 打开串口

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



##### |-3-2 实例2 接受编码器消息

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

###### |-3-2-1 遇到的问题

1、接受消息需要进行处理，直接输出并不是一个7组16进制数字

2、此处的``32768.0f``是通过2的15次方求出，15即是编码器的位数

```c++
// 转换数据格式
float value = combined_value * 360.0f / 32768.0f;
```





## 附录七 N100 IMU的使用

### |-7-0 Serial库的安装

Linux 中一般不需要安装 CP2102 的驱动和 CH9102 的驱动，CH9102 设备别名时用 ttyACM 开头的设备直接别名。

如果使用IMU的串口进行通讯，由于ros2中没有集成serial库，因此需要自己下载源码进行编译安装。

首先使用下面获取

```shell
git clone https://github.com/jinmenglei/serial_ros2
```

之后创建``build``文件夹

```shell
cd build
cmake ..
make
sudo make install 
```

之后将``fdilink_ahrs_ROS2``移动至工作空间的``src``下，在工作空间下进行编译

<img src="https://gitee.com/zhangchenxuv/images/raw/main/image/image-20240912185826682.png" alt="image-20240912185826682" style="zoom:50%;" />

直接运行可能会出现以下问题

![image-20240912205207285](https://gitee.com/zhangchenxuv/images/raw/main/image/image-20240912205207285.png)

这是由于没有固定串口号导致的问题

解决方法为：

1、修改串口号

在 Windows 中需要把 WHEELTEC N 系列上的 CP2102 芯片串口号改为 0003，用 USB 线把惯导模块连接电脑，通过 CP21xxCustomizationUtility 这个 windows上的软件修改并固定,操作如下图：

![image-20240912205633862](https://gitee.com/zhangchenxuv/images/raw/main/image/image-20240912205633862.png)

2、设备创建别名

设对应的串口名一般都是会变化的，为了避免手动选择，这里可以通过给USB 设备创建别名的方式解决。
WHEELTEC 通过脚本文件来为设备创建别名，WHEELTEC N 系列对应的串口号为 0003，对应 ATTRS{serial}=="0003"，脚本文件存放在【资料包\2.ROS_SDK\fdilink_ahrs_ROS1\fdilink_ahrs】文件夹下的 wheeltec_udev.sh 文件中：

![image-20240912205718426](https://gitee.com/zhangchenxuv/images/raw/main/image/image-20240912205718426.png)

依次运行以下两个指令：

```shell
sudo chmod 777 wheeltec_udev.sh
sudo ./wheeltec_udev.sh
```

![image-20240912205825113](https://gitee.com/zhangchenxuv/images/raw/main/image/image-20240912205825113.png)

3、检查

把 WHEELTEC N100 模块连接到 ROS 主控，在终端运行：ll /dev 查看设备

![image-20240912205915959](https://gitee.com/zhangchenxuv/images/raw/main/image/image-20240912205915959.png)

完成以上操作后重新插拔imu

之后编译

![image-20240912230728189](https://gitee.com/zhangchenxuv/images/raw/main/image/image-20240912230728189.png)

可以正常运行。

之后正常通过编译：

<img src="https://gitee.com/zhangchenxuv/images/raw/main/image/image-20240912233802709.png" alt="image-20240912233802709" style="zoom: 67%;" />

启用``Launch``文件

```shell
. install/setup.bash 
ros2 launch fdilink_ahrs ahrs_driver.launch.py
```

![image-20240912235353369](https://gitee.com/zhangchenxuv/images/raw/main/image/image-20240912235353369.png)

echo话题查看数据

<img src="https://gitee.com/zhangchenxuv/images/raw/main/image/image-20240912235723586.png" alt="image-20240912235723586" style="zoom:67%;" />

在新终端执行

```shell
ros2 topic echo imu
```

**补充：**其他用来管理和审查话题的工具和命令

1. **rqt_graph**: 这是一个图形化工具，用于可视化节点和话题以及它们之间的连接。您可以通过在终端中输入 `rqt_graph` 来运行它，或者通过 `rqt` 插件来访问。
2. **ros2 topic list**: 这个命令列出了当前活跃的所有话题。使用 `ros2 topic list -t` 可以显示每个话题的消息类型。
3. **ros2 topic echo**: 使用这个命令可以实时监听某个话题的消息流。例如，`ros2 topic echo /turtle1/cmd_vel` 将显示 `/turtle1/cmd_vel` 话题上的消息。
4. **ros2 topic info**: 这个命令显示某个特定话题的详细信息，包括其类型、发布者和订阅者列表。例如，`ros2 topic info /turtle1/cmd_vel`。
5. **ros2 interface show**: 这个命令用于查看消息类型的详细结构，例如 `ros2 interface show geometry_msgs/msg/Twist` 将显示 `Twist` 消息类型的结构。
6. **ros2 topic pub**: 这个命令允许您直接从命令行向话题发布消息。例如，`ros2 topic pub --once /turtle1/cmd_vel geometry_msgs/msg/Twist "{linear: {x: 2.0, y: 0.0, z: 0.0}, angular: {x: 0.0, y: 0.0, z: 1.8}}"` 将向 `/turtle1/cmd_vel` 话题发布一次消息。
7. **ros2 topic hz**: 使用这个命令可以监控并报告某个话题的消息发布频率。例如，`ros2 topic hz /turtle1/cmd_vel`。



### |-7-1 如何订阅IMU中的数据？

实际订阅并不需要走太多弯路，在提供的``ahrs_driver.cpp``文件中，已经创建好了``imu_pub``发布方

<img src="https://gitee.com/zhangchenxuv/images/raw/main/image/image-20240917232002743.png" alt="image-20240917232002743" style="zoom:50%;" /><img src="https://gitee.com/zhangchenxuv/images/raw/main/image/image-20240917232038872.png" alt="image-20240917232038872" style="zoom:67%;" />

话题名称为``imu_topic``,因此只需要订阅这个话题便可以获取IMU的数据

订阅的过程可以修改``imu_tf.cpp``达到,在整个历程中，有很多参数服务器，个人推测应该是为了提高代码移植性，以及方便通过参数服务器修改对应参数而设计的。

```cpp
#include <memory>
#include <inttypes.h>
#include <rclcpp/rclcpp.hpp>
#include <sensor_msgs/msg/imu.hpp>
#include <tf2_ros/transform_broadcaster.h>
#include <tf2_geometry_msgs/tf2_geometry_msgs.hpp>
#include <tf2/LinearMath/Transform.h>
#include <tf2/LinearMath/Quaternion.h>
#include <string>
#include <geometry_msgs/msg/transform_stamped.hpp>
#include <rclcpp/time.hpp>
using std::placeholders::_1;
using namespace std;

/* 参考ROS wiki
 * http://wiki.ros.org/tf/Tutorials/Writing%20a%20tf%20broadcaster%20%28C%2B%2B%29
 * */

int position_x;
int position_y;
int position_z;
std::string imu_topic;
std::string imu_frame_id, world_frame_id;

static std::shared_ptr<tf2_ros::TransformBroadcaster> br;
rclcpp::Node::SharedPtr nh_ = nullptr;
class imu_data_to_tf : public rclcpp::Node
{
public:
    imu_data_to_tf() : Node("imu_data_to_tf")
    {

        // node.param("/imu_tf/imu_topic", imu_topic, std::string("/imu"));
        // node.param("/imu_tf/position_x", position_x, 0);
        // node.param("/imu_tf/position_y", position_y, 0);
        // node.param("/imu_tf/position_z", position_z, 0);
        RCLCPP_INFO(this->get_logger(), "订阅方创建！");

        this->declare_parameter<std::string>("world_frame_id", "/world");
        this->get_parameter("world_frame_id", world_frame_id);

        this->declare_parameter<std::string>("imu_frame_id", "/imu");
        this->get_parameter("imu_frame_id", imu_frame_id);

        this->declare_parameter<std::string>("imu_topic", "/imu");
        this->get_parameter("imu_topic", imu_topic);

        this->declare_parameter<std::int16_t>("position_x", 1);
        this->get_parameter("position_x", position_x);

        this->declare_parameter<std::int16_t>("position_y", 1);
        this->get_parameter("position_y", position_y);

        this->declare_parameter<std::int16_t>("position_z", 1);
        this->get_parameter("position_z", position_z);
        // br = std::make_shared<tf2_ros::TransformBroadcaster>(this);
        sub_ = this->create_subscription<sensor_msgs::msg::Imu>(imu_topic.c_str(), 10, std::bind(&imu_data_to_tf::ImuCallback, this, _1));
    }

private:
    double r, p, y;
    rclcpp::Subscription<sensor_msgs::msg::Imu>::SharedPtr sub_;

    // rclcpp::Subscriber sub = node.subscribe(imu_topic.c_str(), 10, &ImuCallback);

    void ImuCallback(const sensor_msgs::msg::Imu::SharedPtr imu_data)
    {

        // static tf2_ros::TransformBroadcaster br;//广播器

        // tf2::Transform transform;
        // transform.setOrigin(tf2::Vector3(position_x, position_y, position_z));//设置平移部分

        // 从IMU消息包中获取四元数数据
        tf2::Quaternion q;
        q.setX(imu_data->orientation.x);
        q.setY(imu_data->orientation.y);
        q.setZ(imu_data->orientation.z);
        q.setW(imu_data->orientation.w);
        q.normalized(); // 归一化

        // transform.setRotation(q);//设置旋转部分
        // 广播出去
        // br.sendTransform(tf::StampedTransform(transform, ros::Time::now(), "world", "imu"));
        geometry_msgs::msg::TransformStamped tfs;
        tfs.header.stamp = rclcpp::Node::now();
        tfs.header.frame_id = "world";
        tfs.child_frame_id = "imu";
        tfs.transform.translation.x = position_x;
        tfs.transform.translation.y = position_y;
        tfs.transform.translation.z = position_z;
        tfs.transform.rotation.x = q.getX();
        tfs.transform.rotation.y = q.getY();
        tfs.transform.rotation.z = q.getZ();
        tfs.transform.rotation.w = q.getW();

        tf2::Matrix3x3(q).getRPY(r, p, y);
        r = r*180/M_PI;
        p = p*180/M_PI;
        y = y*180/M_PI;
        RCLCPP_INFO(this->get_logger(), "滚转：%.2f-----俯仰：%.2f-----偏航：%.2f", r,p,y);
        // br->sendTransform(tfs);
        // tf2::(transform, rclcpp::Node::now(), "world", "imu")
    }
};

int main(int argc, char **argv)
{
    rclcpp::init(argc, argv);

    // ros::NodeHandle node;, "imu_data_to_tf"
    auto node = std::make_shared<imu_data_to_tf>();
    rclcpp::spin(node);
    rclcpp::shutdown();
    return 0;
}
```

其中最为主要的是 97 行的转换函数，将四元数转换为RPY角。

```cpp
        double r, p, y;
//.....................................
        tf2::Matrix3x3(q).getRPY(r, p, y); // 将 q 转换为一个3×3的矩阵对象,调用getRPY(),将其转换为欧拉角
        r = r*180/M_PI;
        p = p*180/M_PI;
        y = y*180/M_PI;
        RCLCPP_INFO(this->get_logger(), "滚转：%.2f-----俯仰：%.2f-----偏航：%.2f", r,p,y);
```

启动

```bash
ros2 launch fdilink_ahrs ahrs_driver.launch.py
```

再启动

```bash
ros2 run fdilink_ahrs imu_tf_node
```

![image-20240917233034192](https://gitee.com/zhangchenxuv/images/raw/main/image/image-20240917233034192.png)

可以正常读数，关于偏航角，其 0 位置为IMU上电时的位置，具体可以参考手册中关于相对和绝对偏航角，以及 y角的设置



### |-7-2 如何可视化数据？

首先需要安装IMU的可视化工具，imu-tools

```bash
sudo apt install ros-humble-imu-tools
```

装完毕后，运行节点，并开启rviz2，点击`add`，在`By topic`中添加`Imu`

其中fix frame可以在topic中能看到，具体方式是*ros2 topic echo /imu*，就能看到frame_id，fixed frame替换即可。

<img src="https://gitee.com/zhangchenxuv/images/raw/main/image/image-20240917233525319.png" alt="image-20240917233525319" style="zoom: 33%;" />

<img src="https://gitee.com/zhangchenxuv/images/raw/main/image/image-20240917234025281.png" alt="image-20240917234025281" style="zoom: 33%;" />

## 附录七 为IMU的ROS项目设计QT界面

**需要的学习知识:**1、QT自定义消息；2：QT多线程。

**教程推荐：**

	<iframe src="//player.bilibili.com/player.html?isOutside=true&aid=782930587&bvid=BV1g24y1F7X4&cid=1110670698&p=56" scrolling="no" border="0" frameborder="no" framespacing="0" allowfullscreen="true"></iframe>
	<iframe src="//player.bilibili.com/player.html?isOutside=true&aid=813323036&bvid=BV1N34y1H7x7&cid=771797309&p=14" scrolling="no" border="0" frameborder="no" framespacing="0" allowfullscreen="true"></iframe>；
	<iframe src="//player.bilibili.com/player.html?isOutside=true&aid=813323036&bvid=BV1N34y1H7x7&cid=771797708&p=13" scrolling="no" border="0" frameborder="no" framespacing="0" allowfullscreen="true"></iframe>



首先需要安装并启用``ROSProjectManager``插件

### |-7-1 **在QT中建立ROS2框架**

新建一个文件夹用来放项目文件

<img src="https://gitee.com/zhangchenxuv/images/raw/main/image/image-20240921144117402.png" alt="image-20240921144117402" style="zoom:67%;" />

注意build system选择colcon

<img src="https://gitee.com/zhangchenxuv/images/raw/main/image/image-20240921144147990.png" alt="image-20240921144147990" style="zoom:67%;" />

右键选中，构建一次，如果没有``src``文件，在过滤树形视图中设置显示空文件夹

![image-20240921144737842](https://gitee.com/zhangchenxuv/images/raw/main/image/image-20240921144737842.png)

之后将上文中ROS2项目中``src``目录下的文件均复制到QT项目中，项目结构如下所示

![image-20240921145137655](https://gitee.com/zhangchenxuv/images/raw/main/image/image-20240921145137655.png)

此时可以尝试在QT中选中编译，如何之前ROS2的项目没有问题，此处编译也不会出现问题（下图提示没有用到某些变量，再次编译即可消除）

<img src="https://gitee.com/zhangchenxuv/images/raw/main/image/image-20240921145728533.png" alt="image-20240921145728533" style="zoom: 50%;" />

### |-7-2 **在QT中创建UI**

在``src``上右键，添加新文件，选Qt设计师界面类->选则MainWindow

<img src="https://gitee.com/zhangchenxuv/images/raw/main/image/image-20240921145949899.png" alt="image-20240921145949899" style="zoom:67%;" />

在``src``目录中手动添加``main.cpp``，其内容如下

``main.cpp``:

```cpp
#include "mainwindow.h"
#include <Eigen/Eigen>
#include <QApplication>
#include <ahrs_driver.h>
#include <rclcpp/rclcpp.hpp>

int main(int argc, char *argv[]) {   
  QApplication a(argc, argv);
  rclcpp::init(argc, argv);
  MainWindow w;
  w.show();
  return a.exec();
}
```

删除``ahrs_driver.cpp``中入口

![image-20240921151824797](https://gitee.com/zhangchenxuv/images/raw/main/image/image-20240921151824797.png)

由于后续内容用不到以``imu_tf``开头的launch文件和cpp文件，可以删除，CMakelists中也需要排除编译这两个文件，后面会提到这一点

![image-20240921150639521](https://gitee.com/zhangchenxuv/images/raw/main/image/image-20240921150639521.png)

此时的文件目录如下：

![image-20240921150929908](https://gitee.com/zhangchenxuv/images/raw/main/image/image-20240921150929908.png)

### |-7-3 **修改CMakeLists.txt**

需要修改的内容：

```cmake
#----添加的内容-----
find_package(Qt5  REQUIRED COMPONENTS  Widgets)
set(CMAKE_AUTOMOC ON)
set(CMAKE_AUTOUIC ON)
set(CMAKE_INCLUDE_CURRENT_DIR ON)
#-----------------
```

```cmake
# ———————添加的内容--------
add_executable(ahrs_driver_node
    src/ahrs_driver.cpp
    src/crc_table.cpp
    src/main.cpp
    src/mainwindow.ui
    src/mainwindow.h
    src/mainwindow.cpp
)
# -----------------------
```

```cmake
# ----------增加的内容---------
target_link_libraries(ahrs_driver_node Qt5::Widgets)
# ---------------------------
```

```cmake
# ------删除的内容------
# add_executable(imu_tf_node src/imu_tf.cpp)
# ament_target_dependencies(imu_tf_node rclcpp rclpy std_msgs sensor_msgs serial tf2_ros tf2 tf2_geometry_msgs)
# --------------------
```

```cmake
# ---------删除了imu_tf_node-------
install(TARGETS
  ahrs_driver_node
  DESTINATION lib/${PROJECT_NAME}
)
# --------------------------------
```

整体``CMakeLists.txt``文件如下

```cmake
cmake_minimum_required(VERSION 3.5)
project(fdilink_ahrs)


# Default to C99
if(NOT CMAKE_C_STANDARD)
  set(CMAKE_C_STANDARD 99)
endif()

# Default to C++14
if(NOT CMAKE_CXX_STANDARD)
  set(CMAKE_CXX_STANDARD 14)
endif()

if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  add_compile_options(-Wall -Wextra -Wpedantic)
endif()

find_package(Eigen3 REQUIRED)
set(Eigen3_INCLUDE_DIRS ${EIGEN3_INCLUDE_DIR})

# find dependencies
find_package(ament_cmake REQUIRED)
find_package(rclcpp REQUIRED)
find_package(rclpy REQUIRED)
find_package(sensor_msgs REQUIRED)
find_package(std_msgs REQUIRED)
find_package(tf2 REQUIRED)
find_package(tf2_ros REQUIRED)
find_package(serial REQUIRED)
find_package(nav_msgs REQUIRED)
find_package(geometry_msgs REQUIRED)
find_package(tf2_geometry_msgs REQUIRED)

#----添加的内容-----
find_package(Qt5  REQUIRED COMPONENTS  Widgets)
set(CMAKE_AUTOMOC ON)
set(CMAKE_AUTOUIC ON)
set(CMAKE_INCLUDE_CURRENT_DIR ON)
#-----------------



if(BUILD_TESTING)
  find_package(ament_lint_auto REQUIRED)
  # the following line skips the linter which checks for copyrights
  # uncomment the line when a copyright and license is not present in all source files
  #set(ament_cmake_copyright_FOUND TRUE)
  # the following line skips cpplint (only works in a git repo)
  # uncomment the line when this package is not in a git repo
  #set(ament_cmake_cpplint_FOUND TRUE)
  ament_lint_auto_find_test_dependencies()
endif()

include_directories(
  include
  src
  ${catkin_INCLUDE_DIRS}
  ${Eigen3_INCLUDE_DIRS} 
)

# ———————添加的内容--------
add_executable(ahrs_driver_node
    src/ahrs_driver.cpp
    src/crc_table.cpp
    src/main.cpp
    src/mainwindow.ui
    src/mainwindow.h
    src/mainwindow.cpp
)
# -----------------------

ament_target_dependencies(ahrs_driver_node rclcpp rclpy  std_msgs  sensor_msgs  serial tf2_ros tf2 nav_msgs geometry_msgs)

# ----------增加的内容---------
target_link_libraries(ahrs_driver_node Qt5::Widgets)
# ---------------------------

# ------删除的内容------
# add_executable(imu_tf_node src/imu_tf.cpp)
# ament_target_dependencies(imu_tf_node rclcpp rclpy std_msgs sensor_msgs serial tf2_ros tf2 tf2_geometry_msgs)
# --------------------


# find_package(Boost 1.55.0 REQUIRED COMPONENTS system filesystem)
# include_directories(ahrs_driver_node ${Boost_INCLOUDE_DIRS})
# link_directories(ahrs_driver_node ${Boost_LIBRARY_DIRS})
# target_link_libraries(ahrs_driver_node ${Boost_LIBRSRIES}

# ---------删除了imu_tf_node-------
install(TARGETS
  ahrs_driver_node
  DESTINATION lib/${PROJECT_NAME}
)
# --------------------------------

install(
  DIRECTORY launch 
  DESTINATION share/${PROJECT_NAME}
)

ament_package()

```



编译运行后会提示需要一个可执行程序

<img src="https://gitee.com/zhangchenxuv/images/raw/main/image/image-20240921152340855.png" alt="image-20240921152340855" style="zoom:67%;" />

从CMakeLists.txt中可知，其生成的install文件在``/lib``中,在执行档中选中对应文件

<img src="https://gitee.com/zhangchenxuv/images/raw/main/image/image-20240921153525638.png" alt="image-20240921153525638" style="zoom:67%;" />

再次运行可以弹出UI窗口

<img src="https://gitee.com/zhangchenxuv/images/raw/main/image/image-20240921153622863.png" alt="image-20240921153622863" style="zoom:67%;" />

### |-7-4 UI初步设计

如果想达到一个按钮具有按下和释放均有执行，需要勾选属性中``checkable``选项

<img src="https://gitee.com/zhangchenxuv/images/raw/main/image/image-20240921153959685.png" alt="image-20240921153959685" style="zoom:67%;" />

在``mainwindow.h``中添加头文件,同时定义pushbotton，及pushButton的槽函数

```cpp
#ifndef MAINWINDOW_H
#define MAINWINDOW_H

#include <QMainWindow>
#include <QtWidgets/QPushButton>

namespace Ui {
class MainWindow;
}

class MainWindow : public QMainWindow
{
    Q_OBJECT

public:
    explicit MainWindow(QWidget *parent = nullptr);
    ~MainWindow();
    // 定义pushButton
    QPushButton pushButton;

private:
    Ui::MainWindow *ui;
    
public slots:
    // pushButton的槽函数
    void On_allSelectBtnSlot();
};

#endif // MAINWINDOW_H
```

之后修改``mainwindow.cpp``文件,将槽函数On_allSelectBtnSlot与pushButton连接,并定义On_allSelectBtnSlot具体实现内容

```cpp
#include "mainwindow.h"
#include "ui_mainwindow.h"

MainWindow::MainWindow(QWidget *parent)
    : QMainWindow(parent)
    , ui(new Ui::MainWindow)
{
    ui->setupUi(this);
    // 将槽函数On_allSelectBtnSlot与pushButton连接
    connect(ui->pushButton,SIGNAL(clicked(bool)),this,SLOT(On_allSelectBtnSlot()));
}

MainWindow::~MainWindow()
{
    delete ui;
}
void MainWindow::On_allSelectBtnSlot()
{
    if(ui->pushButton->isChecked())
    {
        // 点击按钮时执行的功能代码
    }
    else
    {
        // 释放按钮时执行的功能代码
    }
}

```

### |-7-5 QT多线程实现

使用多线程的原因：

在ROS2中，`spin`函数的作用是保持节点的运行状态，处理回调函数和事件。具体来说，`spin`函数会进入一个循环，不断检查并调用节点的回调函数，以响应订阅的消息、服务请求等。因此，在主线程执行spin的时候会阻塞，所以通过多线程解决这个问题。

如何设置QT多线程可以在教程中学习，同时此处需要注意的问题有：

1、在其他线程中无法调用UI中的函数，通过继承的方式也不行，因此如果需要将线程中的数据传送到UI界面中，需要使用``自定义信号``来实现，大概来讲就是在其他线程中使用 ``emit()``函数，通过信号槽将数据传出，再通过信号槽将数据显示在ui中；

参考教程：https://www.bilibili.com/video/BV1N34y1H7x7?t=0.6&p=14

2、rclcpp：：init（）与rclcpp：：shutdown（）成对出现，避免重复[初始化](https://so.csdn.net/so/search?q=初始化&spm=1001.2101.3001.7020)以及重复关闭情况；

3、``PI``的定义可能会与一些系统库冲突，如果出现，则将``ahrs_driver.h``中的PI的定义改一下；

<img src="https://gitee.com/zhangchenxuv/images/raw/main/image/image-20240921161019045.png" alt="image-20240921161019045" style="zoom:50%;" />

4、``setText``只能显示字符串，需要转换格式；

另附完整代码文件：

``main.cpp``

```cpp
#include "mainwindow.h"
#include <Eigen/Eigen>
#include <QApplication>
#include <ahrs_driver.h>
#include <rclcpp/rclcpp.hpp>

int main(int argc, char *argv[]) {   
  QApplication a(argc, argv);
  MainWindow w;
  w.show();
  return a.exec();
}
```



``mainwindow.cpp``

```cpp
#include "mainwindow.h"
#include "ui_mainwindow.h"
#include <QDebug>
#include <ahrs_driver.h>
#include <Eigen/Eigen>
#include <rclcpp/rclcpp.hpp>

#include <memory>
#include <inttypes.h>
#include <rclcpp/rclcpp.hpp>
#include <sensor_msgs/msg/imu.hpp>
#include <tf2_ros/transform_broadcaster.h>
#include <tf2_geometry_msgs/tf2_geometry_msgs.hpp>
#include <tf2/LinearMath/Transform.h>
#include <tf2/LinearMath/Quaternion.h>
#include <string>
#include <geometry_msgs/msg/transform_stamped.hpp>
#include <rclcpp/time.hpp>
#include <QWidget>

using std::placeholders::_1;
int position_x;
int position_y;
int position_z;
double r_imu, p_imu, y_imu;

MainWindow::MainWindow(QWidget *parent)
    : QMainWindow(parent)
    , ui(new Ui::MainWindow)
{
    // 开启UI界面
    ui->setupUi(this);
    // 设置imu的输出框无光标且只读
    ui->lineEdit->setStyleSheet("QLineEdit {}");
    ui->lineEdit->setReadOnly(true);
    ui->lineEdit_p->setStyleSheet("QLineEdit {}");
    ui->lineEdit_p->setReadOnly(true);
    ui->lineEdit_y->setStyleSheet("QLineEdit {}");
    ui->lineEdit_y->setReadOnly(true);

    qDebug()<<"读数线程开启!!!"<<Qt::endl;
    // 开启读数线程
    imu_data_Thread = new imu_data_thread(this);
    imu_data_Thread->start();
    // 开启IMU广播线程
    imu_br_Thread = new imu_br_thread(this);

    connect(imu_data_Thread,&imu_data_thread::Data,this,&MainWindow::display_imu);
    connect(ui->pushButton,SIGNAL(clicked(bool)),this,SLOT(On_allSelectBtnSlot()));
}

MainWindow::~MainWindow()
{
    delete ui;
    rclcpp::shutdown();
}

//------------------------------------------------------------------------------------//

void MainWindow::On_allSelectBtnSlot()
{
    if(ui->pushButton->isChecked())
    {

        qDebug()<<"----------IMU使能----------"<<Qt::endl;
        // 开启线程
        imu_br_Thread->start();
    }
    else
    {
        qDebug()<<"----------IMU关闭----------"<<Qt::endl;
        // 关闭线程
        imu_br_Thread->terminate();
        // 清空lineedit中显示的数据
        ui->lineEdit->clear();
        ui->lineEdit_p->clear();
        ui->lineEdit_y->clear();
    }
}

// 开启IMU广播线程任务
void imu_br_thread::run()
{
    qDebug() << "IMU Broadcast Thread Running"<<Qt::endl;
    FDILink::ahrsBringup bp;
}

// 开启读数线程任务
void imu_data_thread::run()
{
    qDebug()<<"读数线程开启"<<Qt::endl;


    int argc=0;
    char **argv=NULL;
    rclcpp::init(argc,argv);
    node=rclcpp::Node::make_shared("imu_data_to_tf");
    sub_ = node->create_subscription<sensor_msgs::msg::Imu>("imu", 10, std::bind(&imu_data_thread::ImuCallback,this, _1));
    RCLCPP_INFO(node->get_logger(), "读数初始化完成");
    rclcpp::spin(node);
    rclcpp::shutdown();
    qDebug()<<"节点关闭"<<Qt::endl;


}


// 读数线程任务回调函数
void imu_data_thread::ImuCallback(const sensor_msgs::msg::Imu::SharedPtr imu_data){

    // static tf2_ros::TransformBroadcaster br;//广播器

    // tf2::Transform transform;
    // transform.setOrigin(tf2::Vector3(position_x, position_y, position_z));//设置平移部分

    // 从IMU消息包中获取四元数数据
    // qDebug()<<"回调函数启动"<<Qt::endl;
    tf2::Quaternion q;
    q.setX(imu_data->orientation.x);
    q.setY(imu_data->orientation.y);
    q.setZ(imu_data->orientation.z);
    q.setW(imu_data->orientation.w);
    q.normalized(); // 归一化

    // transform.setRotation(q);//设置旋转部分
    // 广播出去
    // br.sendTransform(tf::StampedTransform(transform, ros::Time::now(), "world", "imu"));
    geometry_msgs::msg::TransformStamped tfs;
    tfs.header.stamp = rclcpp::Clock().now();
    tfs.header.frame_id = "world";
    tfs.child_frame_id = "imu";
    tfs.transform.translation.x = position_x;
    tfs.transform.translation.y = position_y;
    tfs.transform.translation.z = position_z;
    tfs.transform.rotation.x = q.getX();
    tfs.transform.rotation.y = q.getY();
    tfs.transform.rotation.z = q.getZ();
    tfs.transform.rotation.w = q.getW();

    tf2::Matrix3x3(q).getRPY(r_imu, p_imu, y_imu);
    r_imu = r_imu*180/M_PI;
    p_imu = p_imu*180/M_PI;
    y_imu = y_imu*180/M_PI;

    RCLCPP_INFO(node->get_logger(), "滚转：%.2f-----俯仰：%.2f-----偏航：%.2f-----", r_imu,p_imu,y_imu);
    // 通过信号槽将信号传出
    emit Data(r_imu,p_imu,y_imu);
}

// 将信号槽中的数据显示至Ui中
void MainWindow::display_imu(double dis_r,double dis_p,double dis_y){

    ui->lineEdit->setText(QString::number(dis_r));
    ui->lineEdit_p->setText(QString::number(dis_p));
    ui->lineEdit_y->setText(QString::number(dis_y));
}
```



``main.h``

```cpp
#ifndef MAINWINDOW_H
#define MAINWINDOW_H

#include <QDebug>
#include <QMainWindow>
#include <QThread>
#include <QWidget>
#include <QtWidgets/QPushButton>

#include <rclcpp/rclcpp.hpp>
#include <sensor_msgs/msg/imu.hpp>
// #include "std_msgs/msg/string.hpp"

// 两个线程类的声明
class imu_data_thread;
class imu_br_thread;

namespace Ui {
class MainWindow;
}

class MainWindow : public QMainWindow {
  Q_OBJECT

public:
  explicit MainWindow(QWidget *parent = nullptr);
  ~MainWindow();
  QPushButton pushButton;

public slots:
  // pushButton的槽函数
  void On_allSelectBtnSlot();
  // imu信号
  void display_imu(double dis_r, double dis_p, double dis_y);

private:
  Ui::MainWindow *ui;
  imu_data_thread *imu_data_Thread;
  imu_br_thread *imu_br_Thread;
};

// 订阅线程
class imu_data_thread : public QThread {
  Q_OBJECT
public:
  rclcpp::Subscription<sensor_msgs::msg::Imu>::SharedPtr sub_;
  std::shared_ptr<rclcpp::Node> node;
  void ImuCallback(const sensor_msgs::msg::Imu::SharedPtr imu_data);

  imu_data_thread(QWidget *parent = nullptr) { Q_UNUSED(parent) };
  ~imu_data_thread() {}

  void run();

signals:
  void Data(double r_data, double p_data, double y_data);
};

// 广播线程
class imu_br_thread : public QThread {
  Q_OBJECT
public:
  imu_br_thread(QWidget *parent = nullptr) { Q_UNUSED(parent) };
  ~imu_br_thread() {}
  void run();
};

#endif // MAINWINDOW_H

```



**效果展示：**

<img src="https://gitee.com/zhangchenxuv/images/raw/main/image/image-20240921161529587.png" alt="image-20240921161529587" style="zoom:50%;" />



## 附录八 以ROS2作为上位机控制STM32

## 附录九 ROS2中如何使用MODBUS RS485

### |-9-1 如何安装MODBUS RS485库

首先

```bash
sudo apt-get install libmodbus-dev
```

之后编译，肯会遇到以下错误

<img src="https://gitee.com/zhangchenxuv/images/raw/main/image/image-20241018213750812.png" alt="image-20241018213750812" style="zoom:67%;" />

此时去查找``modbus.h``以及``libmodbus.so``相关文件的位置，之后在``/usr/lib/cmake``创建以下文件夹以及文件

<img src="https://gitee.com/zhangchenxuv/images/raw/main/image/image-20241018214043245.png" alt="image-20241018214043245" style="zoom: 67%;" />

内容如下，相对应的地址可以根据实际情况进行更改

```cmake
FIND_PATH(HIREDIS_INCLUDE_DIR modbus.h
          /usr/local/include
          /usr/include
          )
 
FIND_LIBRARY(MODBUS_LIBRARIES NAMES MODBUS
             PATHS
             /usr/local/lib
             /usr/lib
             /usr/lib/x86_64-linux-gnu
             )
```

### |-9-1 实际使用

实际使用可以参考串口通讯，但发送的信息一定要注意有**校验码**





## 附录十 在ROS2环境下使用相机设备

参考资料：

https://bingda.yuque.com/staff-hckvzc/ai5gkn/nu8kcf

https://zhuanlan.zhihu.com/p/517831485

### |-10-1 设备选择

在ROS中使用摄像头的前提是，摄像头在Linux系统下是可识别的，通常来说UVC协议的USB摄像头都可以正常使用，例如这一类。

<img src="https://cdn.nlark.com/yuque/0/2022/png/29726012/1661767551540-64c6eb70-b818-4b90-9891-dcb7462bc1de.png" alt="img" style="zoom:50%;" />

大部分笔记本电脑自带的摄像头都是UVC协议的，所以也是可以直接使用的。

### |-10-2 如何固定摄像头名称

对于免驱的usb摄像头，其设备号一般可以刷写，这个功能可能需要于卖家沟通

#### |-10-2-1 设备名称

```bash
ls /dev
```

终端命令，确定有几个视频输入

![image-20241118111933460](https://gitee.com/zhangchenxuv/images/raw/main/image/image-20241118111933460.png)

可见有四个输入，而我只使用了两个摄像头，所以需要确认有效的输入。

在安装``v4l2_camera``后https://zhuanlan.zhihu.com/p/517831485，通过与``rqt``工具箱结合，确认有效输入，可能需要用到

```bash 
ros2 run v4l2_camera v4l2_camera_node --ros-args -p video_device:=/dev/video1
```

来选中设备，通过

```bash
ros2 run v4l2_camera v4l2_camera_node --ros-args --remap image_raw:=image_raw/upperright_cam
```

来重映射话题名称

对于我目前的设备，分别使用

```bash
ros2 run v4l2_camera v4l2_camera_node --ros-args -p video_device:=/dev/video0
```

![image-20241118112312956](https://gitee.com/zhangchenxuv/images/raw/main/image/image-20241118112312956.png)

以及

```bash
ros2 run v4l2_camera v4l2_camera_node --ros-args -p video_device:=/dev/video2 --remap image_raw:=image_raw/upperright_cam
```

![image-20241118112403627](https://gitee.com/zhangchenxuv/images/raw/main/image/image-20241118112403627.png)

可以在``rqt``工具箱中获取两个话题

![image-20241118112444915](https://gitee.com/zhangchenxuv/images/raw/main/image/image-20241118112444915.png)

#### |-10-2-2 获取设备ID

USB相机使用固定设备名称的方式，此外还有固定USB口名称的方式

https://blog.csdn.net/weixin_58198422/article/details/137474551

https://blog.csdn.net/qq_37280428/article/details/124960303

https://blog.csdn.net/HuangChen666/article/details/125626570

https://www.cnblogs.com/miaorn/p/14144854.html

使用下面这条命令获取

```bash
udevadm info -a -p /sys/class/video4linux/video0
```

往下翻到的第一个

![image-20241118112900217](https://gitee.com/zhangchenxuv/images/raw/main/image/image-20241118112900217.png)

这个获取方式也可以通过插拔设备，加``lsusb``命令的方式获取

#### |-10-2-3 编译规则

进入`/etc/udev/rules.d/`文件夹下

![image-20241118113336744](https://gitee.com/zhangchenxuv/images/raw/main/image/image-20241118113336744.png)

新建`video.rules`文件，文件内容如下：

```
KERNEL=="video*" , ATTRS{idVendor}== "0bda", ATTRS{idProduct}=="3041", ATTR{index}=="0",MODE:="0777", SYMLINK+="camera_1"
```

重启后执行

```bash
ls /dev/camera*
```

![image-20241118115448227](https://gitee.com/zhangchenxuv/images/raw/main/image/image-20241118115448227.png)

可以找到设备，使用

```bash 
ros2 run v4l2_camera v4l2_camera_node --ros-args -p video_device:=/dev/camera_1 
```

可以使用``rqt``获取视频数据。

#### |-10-2-4  应用

创建以下规则

```bash 
KERNEL=="video*" , ATTRS{idVendor}== "0bda", ATTRS{idProduct}=="3041", ATTR{index}=="0",MODE:="0777", SYMLINK+="camera_1"
KERNEL=="video*" , ATTRS{idVendor}== "0bda", ATTRS{idProduct}=="3042", ATTR{index}=="0",MODE:="0777", SYMLINK+="camera_2"
```

为重映射话题

```bash 
ros2 run v4l2_camera v4l2_camera_node --ros-args -p video_device:=/dev/camera_1 --remap image_raw:=image_raw/camera_1
```

``` bash 
ros2 run v4l2_camera v4l2_camera_node --ros-args -p video_device:=/dev/camera_2 --remap image_raw:=image_raw/camera_2
```





























​		































