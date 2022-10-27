现代构建C/C++项目最流行的方式，十分高端，十分蛋疼。

## 基本结构
首先是经典三行：

```CMake
cmake_minimum_required(VERSION 3.9) # 指定最小CMake版本要求
project(answer) # 设置项目名称
add_executable(answer main.cpp answer.cpp) # 添加target和依赖的cpp文件
# CMake会自动找到依赖的头文件，不需要特别指定
```

已经可以发现CMake的指令又臭又长了。

## 构建方法
首先项目的`CMakeLists.txt`已经写好，然后使用如下命令构建:

```CMake
cmake -B build # 指定构建的文件夹
cmake --build build # 执行构建
```

和make一样，是差量编译。

## 模块化

**在模块化中，需要注意控制可见域，依赖的耦合会影响到很多东西。**

### 库目标
将目标做成可重用的库目标。

```CMake
# 添加 libanswer 库目标， STATIC 指定为静态库
add_library(libanswer STATIC answer.cpp)
# 为 answer 可执行目标链接 libanswer
target_link_libraries(answer libanswer)
```

### 子目录
适应项目目录，在每个需要构建的子目录下都建一个`CMakeLists.txt`，然后添加。

```CMake
add_subdirectory(answer)
```

### 添加include path
告诉构建系统include的文件位置。

```CMake
# 给 libanswer 库添加 include path，PUBLIC 使这个 include path 能被外部看到
# 当链接 libanswer 库目标时，include路径会被自动添加到使用此库的 target 的 include 路径里。
target_include_directories(libanswer PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}/include)
```

### 查找系统第三方库

```CMake
# find_package 用于在系统中寻找已经安装的第三方库的头文件和库文件的位置，并创建一个名为 CURL::libcurl 的库目标，以供链接。
find_package(CURL REQUIRED)
target_link_libraries(libanswer PRIVATE CURL::libcurl)
```

### 传递敏感值

```CMake
# 通过-D传入WOLFRAM_APPID变量
set(WOLFRAM_APPID
	""
	CACHE STRING "WolframAlpha APPID"
)

# 判断是否传入参数
if(WOLFRAM_APPID STREQUAL "")
	message(SEND_ERROR "WOLFRAM_APPID must not be empty")
endif()

# 在编译时给target添加一个宏定义，以获取传入的值
target_compile_definitions(libanswer PRIVATE WOLFRAM="${WOLFRAM_APPID}")
```

> ccmake提供一个带UI的cmake

## 编译选项

```CMake
# 设置CMAKE_CXX_STANDARD的话，从设置它的那一级开始 include 的 subdirectory 都会继承这个变量，且应用于所有能看到这个变量的target，而 target_compile_features 只应用于单个 target。
# target_compile_features 可以指定更细粒度的 C++ 特性，例如 cxx_auto_type , cxx_lambda 等。
target_compile_features(libanswer INTERFACE cxx_std_20)

# 设置C++编译器
set (CMAKE_CXX_COMPILER "/usr/bin/g++")

# 对所有编译器有效
add_compile_options(-std=c++11 -Wall -Werror)

# 也可以通过-D参数指定编译参数
```

## 单元测试

关于CTest的内容，目前不是很懂，以后再补。

