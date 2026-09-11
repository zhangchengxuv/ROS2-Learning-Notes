# 四、QT

> 说明 Qt 信号槽的设计思路、基本机制与自定义实现。

[← 返回总目录](../../README.md) · [↑ 返回本章目录](README.md) · [← 上一节：5、UI设计器](05-ui-designer.md)

---

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
