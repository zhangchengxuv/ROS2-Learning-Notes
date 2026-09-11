# 五、C++相关

> 覆盖 CMake 源文件组织、变量、编译标准、头文件路径及静态库和动态库配置。

[← 返回总目录](../../README.md) · [↑ 返回本章目录](README.md) · [下一节：5-2 C++核心编程 →](02-cpp-core.md)

---

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
