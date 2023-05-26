# 刷CMake Practice

## Project_1.1

文件目录：

```bash
.
├── CMakeLists.txt
└── main.cpp
```

`main.cpp`

~~~cpp
#include <iostream>
using namespace std;
int main()
{
    cout << "Hello World from t1 Main!\n";
    return 0;
}
~~~

`CMakeLists.txt`

~~~cmake
PROJECT(HELLO)
SET(SRC_LIST main.cpp)
MESSAGE(STATUS "This is BINARY dir " ${HELLO_BINARY_DIR})
MESSAGE(STATUS "This is SOURCE dir " ${HELLO_SOURCE_DIR})
ADD_EXECUTABLE(hello ${SRC_LIST})
~~~

> 这里我的SET一直报错，不知道为什么
>
> 解决了：引用变量的时候也要加上${}

- `PROJECT`指令 

  语法：`PROJECT(projectname [CXX] [C] [Java])`

  在本例中，`PROJECT(HELLO)`表示工程名称为HELLO，因为是使用C++编写的程序，也可以写为`PROJECT(HELLO CXX)`。需要注意的是，该指令隐式定义了两个cmake变量，分别是：`<projectname>_BINARY_DIR` 以及`<projectname>_SOURCE_DIR`。同时 cmake 系统也帮助我们预定义了 `PROJECT_BINARY_DIR` 和 `PROJECT_SOURCE_DIR`变量，他们的值分别跟`HELLO_BINARY_DIR`与`HELLO_SOURCE_DIR`一致。

  为了统一起见，建议以后直接使用 `PROJECT_BINARY_DIR`，`PROJECT_SOURCE_DIR`，即使修改了工程名称，也不会影响这两个变量。如果使用了`<projectname>_SOURCE_DIR`，修改工程名称后，需要同时修改这些变量。

- `SET`指令 

  语法：`SET(VAR [VALUE] [CACHE TYPE DOCSTRING [FORCE]])`
  显式定义变量：`SET(SRC_LIST main.cpp)`，如果有多个源文件，也可以定义成：`SET(SRC_LIST main.cpp t1.cpp t2.cpp)`

- `MESSAGE`指令

  语法：`MESSAGE([SEND_ERROR | STATUS | FATAL_ERROR] "message to display"...)`

  这个指令用于向终端输出用户定义的信息，包含了三种类型:
  SEND_ERROR，产生错误，生成过程被跳过。
  SATUS，输出前缀为—的信息。
  FATAL_ERROR，立即终止所有 cmake 过程。

- `ADD_EXECUTABLE`指令

  `ADD_EXECUTABLE(hello ${SRC_LIST})`定义了这个工程会生成一个文件名为 hello 的可执行文件，相关的源文件是 SRC_LIST 中定义的源文件列表， 本例中你也可以直接写成 `ADD_EXECUTABLE(hello main.c)`。

清理工程：`make clean`

## Project_2

~~~bash
.
├── build
├── CMakeLists.txt
└── src
    ├── CMakeLists.txt
    └── main.cpp
~~~

根目录下的CMakeLists.txt

```cmake
PROJECT(HELLO CXX)
ADD_SUBDIRECTORY(src bin)
```

src目录下的CMakeLists.txt

```cmake
ADD_EXECUTABLE(hello main.cpp)
```

main.cpp保持Project_1的文件不变。

- `ADD_SUBDIRECTORY` 

  语法：`ADD_SUBDIRECTORY(source_dir [binary_dir] [EXCLUDE_FROM_ALL])`
  向当前工程添加存放源文件的子目录，并可以指定中间二进制和目标二进制存放的位置。`EXCLUDE_FROM_ALL` 参数的含义是将这个目录从编译过程中排除，比如，工程的 example，可能就需要工程构建完成后，再进入 example 目录单独进行构建(当然，你也可以通过定义依赖来解决此类问题)。

  上面的例子定义了将 src 子目录加入工程，并指定编译输出(包含编译中间结果)路径为bin 目录。如果不进行 bin 目录的指定，那么编译结果(包括中间结果)都将存放在build/src 目录(这个目录跟原有的 src 目录对应)，指定 bin 目录后，相当于在编译时将 src 重命名为 bin，所有的中间结果和目标二进制都将存放在 bin 目录。

- `SUBDIRS`

  语法：`SUBDIRS(dir1 dir2...)`

  一次添加多个子目录，并且，即使外部编译，子目录体系仍然会被保存。
  如果我们在上面的例子中将 ADD_SUBDIRECTORY (src bin)修改为 SUBDIRS(src)。
  那么在 build 目录中将出现一个 src 目录，生成的目标代码 hello 将存放在 src 目录中。
  这个指令已经不推荐使用

- 换个地方保存目标二进制

  不论是 `SUBDIRS` 还是 `ADD_SUBDIRECTORY` 指令(不论是否指定编译输出目录)，我们都可以通过 SET 指令重新定义 `EXECUTABLE_OUTPUT_PATH` 和 `LIBRARY_OUTPUT_PATH` 变量来指定最终的目标二进制的位置(指最终生成的 hello 或者最终的共享库，不包含编译生成的中间文件)

  `SET(EXECUTABLE_OUTPUT_PATH ${PROJECT_BINARY_DIR}/bin)`
  `SET(LIBRARY_OUTPUT_PATH ${PROJECT_BINARY_DIR}/lib)`
  在第一节我们提到了`<projectname>_BINARY_DIR` 和 `PROJECT_BINARY_DIR` 变量，他们指的编译发生的当前目录，如果是内部编译，就相当于 PROJECT_SOURCE_DIR 也就是工程代码所在目录，如果是外部编译，指的是外部编译所在目录，也就是本例中的 build目录。

  所以，上面两个指令分别定义了：
  可执行二进制的输出路径为 build/bin 和库的输出路径为 build/lib.

  那在哪个文件添加上述两条指令呢？
  答案是：在哪里 ADD_EXECUTABLE 或 ADD_LIBRARY就在哪里加入上述的定义。
  在本例中，就是指 src 下的 CMakeLists.txt 了

## Project_2大改造

本小节的任务是让前面的 Hello World 更像一个工程，我们需要作的是：

1. 为工程添加一个子目录 src，用来放置工程源代码;
2. 添加一个子目录 doc，用来放置这个工程的文档 hello.txt
3. 在工程目录添加文本文件 COPYRIGHT, README；
4. 在工程目录添加一个 runhello.sh 脚本，用来调用 hello 二进制
5. 将构建后的目标文件放入构建目录的 bin 子目录；
6. 最终安装这些文件：将 hello 二进制与 runhello.sh 安装至/usr/bin，将 doc 目录的内容以及 COPYRIGHT/README 安装到/usr/share/doc/cmake/t2

~~~bash
.
├── build
├── CMakeLists.txt
├── COPYRIGHT
├── doc
│   └── hello.txt
├── README
├── runhello.sh
└── src
    ├── CMakeLists.txt
    └── main.cpp
~~~

- `INSTALL`

  语法：
  ~~~cmake
  INSTALL(TARGETS targets...
  	[[ARCHIVE|LIBRARY|RUNTIME]
  		[DESTINATION <dir>]
  		[PERMISSIONS permissions...]
  		[CONFIGURATIONS
  	[Debug|Release|...]]
  		[COMPONENT <component>]
  		[OPTIONAL]
  		] [...])
  ~~~

  参数中的 TARGETS 后面跟的就是我们通过 ADD_EXECUTABLE 或者 ADD_LIBRARY 定义的目标文件，可能是可执行二进制、动态库、静态库。目标类型也就相对应的有三种，`ARCHIVE` 特指静态库，`LIBRARY` 特指动态库，`RUNTIME`特指可执行目标二进制。

  `DESTINATION` 定义了安装的路径，如果路径以/开头，那么指的是绝对路径，这时候CMAKE_INSTALL_PREFIX 其实就无效了。如果你希望使用 `CMAKE_INSTALL_PREFIX` 来定义安装路径，就要写成相对路径，即不要以/开头，那么安装后的路径就是`${CMAKE_INSTALL_PREFIX}/<DESTINATION 定义的路径>`

  举例：

  ~~~cmake
  INSTALL(TARGETS myrun mylib mystaticlib
  	RUNTIME DESTINATION bin
  	LIBRARY DESTINATION lib
  	ARCHIVE DESTINATION libstatic
  	)
  ~~~

  `myrun`、 `mylib`、`mystaticlib`都是要安装的文件，类型分别为`RUNTIME`、`LIBRARY`、`ARCHIVE`。安装到的路径分别为`${CMAKE_INSTALL_PREFIX}/bin`、`${CMAKE_INSTALL_PREFIX}/lib`、`${CMAKE_INSTALL_PREFIX}/libstatic`。

- 普通文件的安装：

  ~~~cmake
  INSTALL(FILES files... DESTINATION <dir>
  	[PERMISSIONS permissions...]
  	[CONFIGURATIONS [Debug|Release|...]]
  	[COMPONENT <component>]
  	[RENAME <name>] [OPTIONAL])
  ~~~

- 非目标文件的可执行程序安装(比如脚本之类)：

  ~~~cmake
  INSTALL(PROGRAMS files... DESTINATION <dir>
  	[PERMISSIONS permissions...]
  	[CONFIGURATIONS [Debug|Release|...]]
  	[COMPONENT <component>]
  	[RENAME <name>] [OPTIONAL])
  ~~~

- 目录的安装：

  ~~~cmake
  INSTALL(DIRECTORY dirs... DESTINATION <dir>
  	[FILE_PERMISSIONS permissions...]
  	[DIRECTORY_PERMISSIONS permissions...]
  	[USE_SOURCE_PERMISSIONS]
  	[CONFIGURATIONS [Debug|Release|...]]
  	[COMPONENT <component>]
  	[[PATTERN <pattern> | REGEX <regex>]
  	[EXCLUDE] [PERMISSIONS permissions...]] [...])
  ~~~

  这里主要介绍其中的 DIRECTORY、PATTERN 以及 PERMISSIONS 参数。
  `DIRECTORY` 后面连接的是所在 Source 目录的相对路径，但务必注意：
  abc 和 abc/有很大的区别。
  如果目录名不以/结尾，那么这个目录将被安装为目标路径下的 abc，如果目录名以/结尾，
  代表将这个目录中的内容安装到目标路径，但不包括这个目录本身。
  `PATTERN` 用于使用正则表达式进行过滤
  `PERMISSIONS` 用于指定 PATTERN 过滤后的文件权限。

  例子：

  ~~~cmake
  INSTALL(DIRECTORY icons scripts/ DESTINATION share/myproj
  	PATTERN "CVS" EXCLUDE
  	PATTERN "scripts/*"
  	PERMISSIONS OWNER_EXECUTE OWNER_WRITE OWNER_READ GROUP_EXECUTE GROUP_READ)
  ~~~

  这条指令的执行结果是：
  将 icons 目录安装到 `<prefix>/share/myproj`，将 scripts/中的内容安装到`<prefix>/share/myproj`
  不包含目录名为 CVS 的目录，对于 scripts/*文件指定权限为 OWNER_EXECUTE
  OWNER_WRITE OWNER_READ GROUP_EXECUTE GROUP_READ.

- 安装时 CMAKE 脚本的执行：

  `INSTALL([[SCRIPT <file>] [CODE <code>]] [...])`
  `SCRIPT` 参数用于在安装时调用 cmake 脚本文件（也就是`<abc>.cmake` 文件）
  `CODE` 参数用于执行 CMAKE 指令，必须以双引号括起来。比如：
  `INSTALL(CODE "MESSAGE(\"Sample install message.\")")`

~~~cmake
PROJECT(HELLO CXX)
ADD_SUBDIRECTORY(src bin)

# 安装COPYRIGHT README到目录 share/doc/cmake/t2
INSTALL(FILES COPYRIGHT README DESTINATION share/doc/cmake/t2)

# 安装runhello.sh 到目录 bin
INSTALL(PROGRAMS runhello.sh DESTINATION bin)

# 安装 doc目录中的内容到目录 share/doc/cmake/t2
INSTALL(DIRECTORY doc/ DESTINATION share/doc/cmake/t2)
~~~

## 静态库与动态库构建

1. 建立一个静态库和动态库，提供 HelloFunc 函数供其他程序编程使用，HelloFunc向终端输出 Hello World 字符串。
2. 安装头文件与共享库。

文件目录：

~~~bash
.
├── build
├── CMakeLists.txt
└── lib
    ├── CMakeLists.txt
    ├── hello.cpp
    └── hello.h
~~~

库的类型一共有三种：

- SHARED，动态库，xxx.so
- STATIC，静态库，xxx.a
- MODULE，在使用 dyld 的系统有效，如果不支持 dyld，则被当作 SHARED 对待。

生成库的指令

- `ADD_LIBRARY`

  语法：

  ~~~cmake
  ADD_LIBRARY(libname	[SHARED|STATIC|MODULE]
  	[EXCLUDE_FROM_ALL]
  		source1 source2 ... sourceN)
  ~~~

  EXCLUDE_FROM_ALL 参数的意思是这个库不会被默认构建，除非有其他的组件依赖或者手工构建。

  例如：

  ~~~cmake
  SET(LIBHELLO_SRC hello.cpp)
  ADD_LIBRARY(hello SHARED ${LIBHELLO_SRC})
  ~~~

  表示添加动态库，库的名称叫做hello，源文件为hello.cpp，生成的库文件为libhello.so。

添加静态库：

- 例如：

  ~~~cmake
  SET(LIBHELLO_SRC hello.cpp)
  ADD_LIBRARY(hello STATIC ${LIBHELLO_SRC})
  ~~~

  表示添加动态库，库的名称叫做hello，源文件为hello.cpp，生成的库文件为libhello.a。

生成名字相同的静态库和动态库：

- `SET_TARGET_PROPERTIES`

  语法：

  ~~~cmake
  SET_TARGET_PROPERTIES(target1 target2 ...
  	PROPERTIES prop1 value1
  	prop2 value2 ...)
  ~~~

  这条指令可以用来设置输出的名称，对于动态库，还可以用来指定动态库版本和 API 版本。

  在本例中，我们需要作的是向 lib/CMakeLists.txt 中添加一条：

  ~~~cma
  SET_TARGET_PROPERTIES(hello_static PROPERTIES OUTPUT_NAME "hello")
  ~~~


  这样，我们就可以同时得到 libhello.so/libhello.a 两个库了。

  



~~~cmake
SET(LIBHELLO_SRC hello.cpp)
ADD_LIBRARY(hello SHARED ${LIBHELLO_SRC})
ADD_LIBRARY(hello_static STATIC ${LIBHELLO_SRC})
SET_TARGET_PROPERTIES(hello_static PROPERTIES OUTPUT_NAME "hello")
SET_TARGET_PROPERTIES(hello PROPERTIES CLEAN_DIRECT_OUTPUT 1)
SET_TARGET_PROPERTIES(hello_static PROPERTIES CLEAN_DIRECT_OUTPUT 1)
SET_TARGET_PROPERTIES(hello PROPERTIES VERSION 1.2 SOVERSION 1)
~~~

VERSION 指代动态库版本，SOVERSION 指代 API 版本。

~~~cmake
SET(LIBHELLO_SRC hello.cpp)
ADD_LIBRARY(hello SHARED ${LIBHELLO_SRC})
ADD_LIBRARY(hello_static STATIC ${LIBHELLO_SRC})
SET_TARGET_PROPERTIES(hello_static PROPERTIES OUTPUT_NAME "hello")
SET_TARGET_PROPERTIES(hello PROPERTIES CLEAN_DIRECT_OUTPUT 1)
SET_TARGET_PROPERTIES(hello_static PROPERTIES CLEAN_DIRECT_OUTPUT 1)
SET_TARGET_PROPERTIES(hello PROPERTIES VERSION 1.2 SOVERSION 1)

INSTALL(TARGETS hello hello_static
        LIBRARY DESTINATION lib
        ARCHIVE DESTINATION lib)
INSTALL(FILES hello.h DESTINATION include/hello)
~~~

## 如何使用外部共享库和头文件

