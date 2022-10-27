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
首先项目的CMakeLists.txt已经写好，然后使用如下命令构建:

```CMake
cmake -B build # 指定构建的文件夹
cmake --build build # 执行构建
```

和make一样，是差量编译。

## 库目标
将目标做成可重用的库目标。
```CMake
# 添加 libanswer 库目标， STATIC 指定为静态库
add_library(libanswer STATIC answer.cpp)
# 为 answer 可执行目标链接 libanswer
target_link_libraries(answer libanswer)
```

