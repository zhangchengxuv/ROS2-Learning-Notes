# 附录

> 补充命名空间等 C++ 相对 C 语言的扩展知识。

[← 返回总目录](../../README.md) · [↑ 返回本章目录](README.md) · [下一节：附录二 serial串口通讯 →](02-serial-communication.md)

---

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
