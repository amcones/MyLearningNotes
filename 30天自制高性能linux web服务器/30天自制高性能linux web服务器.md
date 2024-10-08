虽然这个项目听说已经被做烂了，但首先我不会，其次我认为真正学懂了linux的人并不多。所以我决定做下去。

因此，这个项目的目的有二：
1. 作为项目写进简历
2. **学习linux编程**

## day1

- 用户程序执行系统调用时，必须执行一个陷阱指令，把控制权移交给操作系统。系统调用与非系统调用的区别，在于一个进入了内核，另一个则没有。
- 系统调用的一般形式，都是调用一个带有若干个参数的库函数，并返回操作的值。如果出错，返回值置为-1，同时设置全局变量`errno`。
- 文件在被读写前首先要被打开，系统打开文件时会生成一个文件描述符（小整数）。而linux中一切都是文件，所以socket连接也会产生一个文件描述符。
- 使用socket的流程：
	- 打开一个socket，获得一个文件描述符
	- 将socket与服务器地址进行绑定。绑定时，需要考虑主机地址序与网络地址序，即小端存储与大端存储的区别。需要使用`htons`（host to network short）等函数进行转换。**任何格式化的数据通过网络传输都应该进行转换。**
	- 监听socket。
- 通用socket地址和专用socket地址，这里通用和专用是相对于协议的。socket唯一表示了使用连接的一端，由于使用协议不同，相同ip:端口的字节流表示可能不同。为了避免复杂的操作，对于不同协议使用专用的地址。但在实际使用时必须都强转成`sockaddr`。

## day2

- 错误处理的逻辑如果不封装，将会变得非常冗余。
- 通过`write`和`read`系统调用向socket中写字节流。

## day3

- epoll IO复用。调试上一章的代码可以发现，当有多个client同时访问server时，server只能够对一个client进行服务。（也许能对多个client进行连接）
	- 复用IO端口，本质上是事件驱动。监听多个文件描述符的行为。
	- `select`,`poll`都是轮询一些文件描述符，检查它们当中有几个发生了感兴趣的事件。把fd从用户空间拷贝到内核空间，然后进行轮询。只是select有最大限制（bitmap），而poll使用链表，几乎没有限制。
	- `epoll`是linux独有的IO复用函数，由一系列函数实现。其次，它将事件存储在内核的一个事件表中，避免重复传递。底层使用红黑树。
	- 通过`epoll_create`创建一个epoll，再使用`epoll_ctl`往里面添加事件，这样就会与网卡驱动程序建立一个回调关系，当对应事件发生时自动调用这些函数，往双链表里添加发生的事件。最后通过`epoll_wait`监听发生的事件。
	- `epoll`的LT模式下，只要一个fd还有数据可读，每次`epoll_wait`都会返回它的事件。而ET模式下，只会提示一次，直到下次有数据流入前都不会触发，所以这种模式下每次read一定要读完buffer或者遇到错误。
	- 这三种本质上都是同步IO，都会阻塞。
- socket的阻塞与非阻塞模式：阻塞模式下，如果socket没有东西可读或者暂时不可读写，就会阻塞等待。而非阻塞模式下，则会直接报错返回。