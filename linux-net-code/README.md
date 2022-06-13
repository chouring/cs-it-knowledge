# Linux网络编程
目录：
- Socket编程
- 

## Socket编程
### 基本步骤
示意图<br>
![图片](https://user-images.githubusercontent.com/75822806/173017566-6b1cc5b5-240f-4266-8e20-7e2cab59dd65.png)

```cpp
int sockfd = socket(); // 返回一个socket, 本质是文件描述符

bind(sockfd, ); // socket 《=》本机的某个ip&端口。因为可能有多网卡，因此需要绑定ip和端口 来 接受信息

listen(sockfd, backlog); // 服务端开始监听
/*
问：为什么接口设计要bind + listen, 而不是直接一个接口：绑定+开始监听呢？类似bind_listen(sockfd, addr)
答：API设计的单一原则。接口越单一越好，接口越单一，复用性越强。

问：listen的参数backlog？
答：TCP三次握手中，server第一次握手会把client的相关信息（tcb）存入半连接队列（syn队列/半连接队列）
而在server第三次握手会把client的相关信息存入全连接队列（accept队列/全连接队列）
这两个队列同为 pending queue
不同的内核实现方法不同，同一内核不同时期版本实现也大不相同。可以认为 backlog是pending queue的最大长度，也可以认为backlog是accept队列的长度
backlog要指定合适的值，Linux一般是128
pending queue中存储的是 第一次握手产生，但还未经过accept系统调用被操作系统分配clientfd的tcb
backlog更多详细的描述[UNIX Network Programming](https://link.zhihu.com/?target=https%3A//book.douban.com/subject/1756533/)
*/

clientfd = accept(sockfd, );
/*
问：accept发生在什么时候？
答：发生在三次握手之后, 从全连接队列中取出一个tcb, 再由文件系统找到一个未被分配的fd, 将其分配给这个tcb
追问：那fd作为一个整数数组，是怎么被分配的？
答：socket的fd 和 文件系统是一样的，是依次增长分配的。0，1，2分别是标准输入、标准输出和标准错误。从3开始依次增长，被分配。
当连接断开时, fd会被系统回收
再追问：那如果在一个IM业务中要记录客户端怎么处理？
答：server维护一张映射表：以sockfd为下标存储用户id的数组
*/

recv(); // 从客户端读数据
decode(); // 对数据进行解析
compute(); // 逻辑处理
encode(); // 对数据进行打包

write() 和 send(fd, buffer, len, 0)
/*
问：write和send的数据对方一定能收到吗？
答：不一定，write和send是将buffer中的数据copy到协议栈中。
问：分包和粘包问题？
答：解决方法1.采用分隔符，方法2.添加长度元数据
*/
```
