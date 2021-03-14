#### SOCKET中首先我们要理解如下几个定义概念

ip，

端口号，

连接（通信链路），

三元组（协议，本地地址，本地端口号）：它指定连接的每半部分

一个完整的网间通信需要一个五元组来标识：五元组(协议，本地地址，本地端口号，远地地址，远地端口号)

#### api

 **socket(int af, int type, int protocol);**  

参数af指定通信发生的区域，：AF_UNIX、AF_INET、AF_NS等，而DOS、 WINDOWS中仅支持AF_INET，它是网际网区域。因此，地址族与协议族相同。

参数type 描述要建立的套接字的类型:tcp,udp, socket_raw(icmp)

参数protocol说明该套接字使用的特定协议，如果调用者不希望特别指定使用的协议，则置为0，使用默认的连接模式

**bind()**

bind(SOCKET s, const struct sockaddr FAR * name, intnamelen);

当一个套接字用socket()创建后，存在一个名字空间(地址族),但它没有被命名。bind()将套接字地址(包括本地主机地址和本地端口地址)与所创建的套接字号联系起来，即将名字赋予套接字，以指定本地半相关

在服务器方，无论是否面向连接，均要调用 bind()，若采用面向连接，则可以不调用bind()，而通过connect()自动完成。若采用无连接，客户方必须使用bind()以获得一个唯一的地址。

**connect()与accept()**

服务端：

connect(SOCKET s, const struct sockaddr FAR * name, intnamelen);

客户端：

 accept(SOCKET s, struct sockaddr FAR* addr, int FAR*addrlen);   

四个套接字系统调用，socket()、bind()、 connect()、accept()，可以完成一个完全五元相关的建立

**listen()**

listen(SOCKET s, int backlog);

backlog表示请求连接队列的最大长度，用于限制排队请求的个数

**数据传输───send()与recv()**

send(SOCKET s, const char FAR *buf, int len, int flags);   

flags 指定传输控制方式，如是否发送带外数据等

 recv(SOCKET s, char FAR *buf, int len, int flags);  

**输入/输出多路复用───select()**

 select(int nfds, fd_set FAR* readfds, fd_set FAR * writefds, fd_set FAR * exceptfds, const struct timevalFAR * timeout);

select() 调用用来检测一个或多个套接字的状态

select()返回包含在fd_set结构中已准备好的套接字描述符的总数目，或者是发生错误则返回SOCKET_ERROR

**关闭套接字───closesocket()**

