# 四、QT

> 梳理 Qt 程序的启动流程、自定义代码接入、变量初始化与编码规范。

[← 返回总目录](../../README.md) · [↑ 返回本章目录](README.md) · [← 上一节：3、QT creator无法输入中文](03-chinese-input.md) · [下一节：5、UI设计器 →](05-ui-designer.md)

---

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
