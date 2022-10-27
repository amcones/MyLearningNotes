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
使用如下命令构建:
```CMake

```