#Makefile
我们会使用makefile来告诉make命令如何编译和链接程序。

在接下来的示例中，我们的工程有8个C文件和3个头文件。我们的规则是：
1. 如果这个工程没有被编译过，那么所有的C文件都要编译并被链接。
2. 如果这个工程的某几个C文件被修改，那么只编译被修改的C文件，并链接目标程序。
3. 如果这个工程的头文件被改变了，那么只重新编译引用了这几个头文件的C文件，并链接目标程序。

只要makefile写得足够好，只需要一个make命令就可以完成这一切。（如CS144）

## makefile的规则

粗略来说，makefile的规则如下：

```Makefile
target ... : prerequisites ...
	command
	...
	...
```

### target
	可以是一个object file（目标文件），也可以是一个执行文件，还可以是一个标签（label）。

### prerequisites
	生成该target所依赖的文件和/或target。

### command
	该target要执行的命令（任意的shell命令）

prerequisites中如果有一个以上的文件比target文件要新的话，command定义的命令就会调用。可以在prerequisites后面用分号隔开加上command，或者另起一行，但command前面必须有一个`Tab`。如果命令太长，可以用反斜杠隔开表示换行。

## make是如何工作的

默认情况下，也就是只输入make命令，那么：
1. make会在当前目录下找名字叫“Makefile”或者“makefile”的文件
2. 如果找到，它会找文件中第一个目标文件（target），并把这个文件作为最终的目标文件。
3. 如果这个目标文件不存在，或是它后面依赖的`.o`文件比这个目标文件新，就执行后面定义的命令来生成这个目标文件。
4. 如果这个目标文件所依赖的文件也不存在，那么make会在当前文件中找目标为`.o`文件的依赖性，如果找到则根据那一个规则生成`.o`文件，如此递归。
5. 如果递归完成，搜到了所有依赖，那么make就会生成`.o`文件，然后再用`.o`文件生成最终目标文件。

如果在递归的过程中搜到底也没有找到依赖，那么make就退出并报错。

通过这些分析，我们可以知道像clean这种没有被第一个目标文件直接或间接关联，那么它后面的命令将不会被自动执行。不过可以显式要求make执行，即命令`make clean`。

## makefile中使用变量

当同一串依赖文件被使用多次，如果其中一个需要修改则所有地方都要改。因此可以把这些文件存在一个变量里，需要修改时只要修改这个变量即可。

```Makefile
objects = main.o kbd.o command.o display.o insert.o search.o files.o utils.o

edit: $(objects)
	cc -o edit $(objects)

...
```

## makefile自动推导

make可以自动推导文件以及文件依赖关系后面的命令，因此没有必要在每一个`.o`文件后面写上类似的命令。

只要make看到一个`.o`文件，它就会自动地把`.c`文件加在依赖关系中，包括编译命令也会被推导出来。

eg：如果make找到一个`whatever.o`文件，那么它就会把`whatever.c`文件添加到`whatever.o`的依赖文件中，同时在command中添加`cc -c whatever.c`。

这也就是make的“隐晦规则”。

> 基本上，在自动推导后，只要在.o文件后写上头文件（函数声明的位置）就行了。

## 另类风格的makefiles

可以写成`多个targets : prerequisites`的风格来收拢重复的依赖。

```Makefile
objects = main.o kbd.o command.o display.o \
    insert.o search.o files.o utils.o

edit : $(objects)
    cc -o edit $(objects)

$(objects) : defs.h
kbd.o command.o files.o : command.h
display.o insert.o search.o files.o : buffer.h

.PHONY : clean
clean :
    rm edit $(objects)
```

虽然可以有效缩短长度，但是这也让依赖关系变得有些凌乱。

## 清空目标文件的规则
每个Makefile中都应该写一个清空目标文件（`.o`和执行文件）的规则，不仅便于重编译，也利于保持文件的清洁。一般的风格都是：
```Makefile
.PHONY : clean
clean :
	-rm edit $(objects)
```

`rm`前的短杠表示，执行过程中遇到问题直接跳过，继续执行。`clean`的规则一般都放在文件的最后。

## Makefile里有什么？

Makefile里主要包含了五个东西：显式规则、隐晦规则、变量定义、文件指示和注释。
1. 显式规则。显式规则说明了如何生存一个或多个目标文件，是需要Makefile的书写者明显指出要生成的文件、文件的依赖文件和生成的命令。
2. 隐晦规则。即make支持的自动推导。
3. 变量的定义。变量一般都是字符串，类似于C语言的宏，当Makefile被执行时，其中的变量都会扩展到相应的引用位置上。
4. 文件指示。其包含了三个部分，一个是类似于C语言的include，在一个Makefile中引用另一个Makefile。另一个是根据某些情况指定Makefile中的有效部分，就像C语言中的预编译#if一样。还有一个是定义一个多行的命令。
5. 注释。Makefile中只有行注释，使用`#`字符。如果需要在Makefile中使用`#`字符，可以用反斜杠进行转义：`\#`

## 引用其它的Makefile

就像C/C++中的`#include`，会把指出的文件原样放在调用的位置。

```Makefile
include <filename>
```

在`include`前面可以有一些空字符，但是绝不能是`Tab`键开始。可以include多个文件，其中用一个或多个空格隔开。支持通配符`*`和变量。

调用`include`时，make首先会在当前目录下寻找，如果没有找到，那么make还会在以下几个目录找：
1. 如果make执行时，有`-I`或`--include-dir`参数，那么make就会去这个参数指定的目录下寻找。
2. 如果目录`<prefix>/include`（一般是：`/usr/local/bin`或`/usr/include`）存在的话，make也会去找。

如果有文件没找到，make会报一个警告，但不会立刻出现致命错误。一旦完成Makefile的读取，make再重试这些没有找到或是不能读取的文件，如果还是不行，make才会报致命错误。如果想要make不理无法读取的文件，可以在include前加一个短杠。

## 环境变量MAKEFILES

如果当前环境定义了这个变量，make会把这个变量中的值做一个类似`include`的动作。这个变量中的值是其他的Makefile，用空格分割。与`include`不同的是，这个环境变量引入的Makefile的target不会起作用，如果引用的文件中发现错误也会忽略。

这个环境变量一旦设置，所有Makefile都会收到影响。如果Makefile出现了怪事，可以看一下这个。

## make的工作方式

GNU的make工作时执行步骤如下：
1. 读入所有的Makefile
2. 读入被include的其它Makefile
3. 初始化文件中的变量
4. 推导隐晦规则，并分析所有规则。
5. 为所有的目标文件创建依赖关系链。
6. 根据依赖关系，决定哪些目标要重新生成。
7. 执行生成命令

其中1-5步为第一个阶段，6-7步为第二个阶段。第一个阶段中，如果定义的变量被使用了，那么make会把其展开在使用的位置。但make不会马上完全展开，而是懒执行。如果变量出现在依赖关系的规则中，只有当这条依赖被决定要使用了，变量才会在其内部展开。

[[书写规则]]