# Linux网络编程
目录：
- 阻塞IO和非阻塞IO
- 阻塞IO + 多线程模型
- IO多路复用
- reactor模型
- 水平触发和边缘触发


## 阻塞IO和非阻塞IO
设置：通过fcntl(fd, f_setl, flag, o_nonblock) 设置fd为非阻塞（默认阻塞）。<br>
区别：IO无数据是否立刻返回。<br>
数据会经历了未准备好->已准备好->在用户态三个状态，前两个状态在内核，最后一个状态在用户态，
非阻塞IO会在遇到第一个状态时立刻返回，阻塞IO则会阻塞等待。<br>


## 阻塞IO + 多线程 模型
- 处理方式：每一个线程处理一个fd
- 优点：处理及时
- 缺点：线程数量有限，且高并发场景下线程利用率低，阻塞等待时间长，处理时间短。

```cpp
// server
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

close(); // 关闭连接

// client
socket();
connect();
send();
recv();
close();
```
## IO多路复用
- 处理方式：在数据准备阶段去select / epoll_wait等阻塞等待数据准备，准备好后返回数据已到达，则接下来可以使用阻塞io也可以使用非阻塞io(因为数据拷贝阶段总是阻塞的)。但是水平出发和边缘触发有所区别。
- 优点：监听多个文件描述符
- 缺点：select / poll / epoll 各有特点，需要各自分析
 ### epoll
    Linux下主要用epoll, 为什么epoll更加优秀？因为可能百万TCP连接的状态下，某个时刻可能只有几十或几百个连接是活跃的。
 
 ```cpp
 // epoll interfaces
 
 ```

 ```cpp
// server
 int listenfd = socket();
 bind(listenfd, addr);
 listen(listenfd);
 int efd = epoll_create(0);
 epoll_ctl(efd, epoll_ctl_add, listenfd, &ev);
 while (true) {
    epoll_event ev[SIZE]; // 用户态创建分配内存
    int nevent = epoll_wait(efd, ev, size, timeout); // 为什么需要timeout?因为不仅仅处理io，还有定时事件
    for (int i = 0; i < nevent; i++) {
        // 处理io
        epoll_event* e = ev[i];
        if (e->fd == listenfd) { // 处理连接事件
            int clientfd = accept(listenfd);
            epoll_ctl(efd, epoll_ctl_add, clientfd, ev);
        }
        else {
            if (/*读事件*/) {
                read();
                logic();
                send();
            }
            if (/*写事件*/) {
                send();
            }
            if (/*错误事件*/) {
                close();
            }
        }
    }
    handle_timer(); 
 }
 ```
 
  ```cpp
  // epoll 底层数据结构：主要是红黑树 + 双向链表
 struct eventpoll { 
    struct rb_root rbr; // 红黑树，管理epoll监听的事件
    struct list_head rdlist; // 双向链表，epoll返回的满足条件的事件
 }
 /*
 当epoll_ctl往红黑树增加节点，会和网卡驱动建立回调关系。
 当某个fd有事件发生，与之对应的网卡驱动回调关系，会把fd拷贝到双向链表中。之后进入数据拷贝阶段，双向链表中的数据会被copy到用户态的数组中。
 红黑树的每个节点是个结构体：包含fd, 读缓冲区，写缓冲区。调用read函数是操作fd的读缓冲区，send是操作fd的写缓冲区。
 操作系统控制将写缓冲区发送到客户端，网络编程往往关注到写入缓冲区即可。

 */
 ```
 
 ## reactor模型
    reactor模型 = 非阻塞IO + IO多路复用
    处理方式：基于事件驱动/事件回调的方式来处理也处理业务逻辑。通过协程等技术吧服务端的回调函数集合。
             协程在服务端通过粘合异步的回调，实现同步分阻塞的操作
    具体写法：
     1. 单reactor模型（epoll + 线程）, 
     处理方式：上边的socket基础写法。
     缺点：遇到计算密集型的业务逻辑占据CPU会影响到网络IO。
     应用：redis, 一个内存数据库。为什么redis用单reactor模型？因为redis的业务逻辑主要是针对redis数据结构的操作，也不会有大量连接。因此可以接受单reactor模型。
     2. 单reactor + 任务队列/消息队列 + 线程池 模型。
     处理方式：logic中把任务加入队列中，让线程池去消耗处理。 
     缺点：多线程处理需要加锁，事件很多仍然有延迟。
     应用：skynet, 一个轻量级游戏服务器框架。
     3. 多reactor模型 之 one loop per thread 模型。 每一个线程至多有一个事件循环。
     处理方式：把网络事件和业务逻辑分配到不同的sub-reactor中去处理。listen收到clientfd后，就和一个sub-reactor绑定。
              每个sub-reactor可以处理read/logic/write逻辑。
     应用：memcached, 一个缓存系统。
     4. 多reactor模型 之 one loop per process
     处理方式：每个worker进程都监听一个port，那么谁来操作呢？也就是产生了“惊群”。
              解决方法简单说下，linux内核中对惊群有处理，而用户态我们也可以处理，比如通过“共享内存 + accept锁”来避免惊群。
     应用：nginx, 一个反向代理服务器。 nginx每一个worker进程都会监听一个port。
     5. 多reactor + (消息队列) + 线程池
     处理方式：面对网络密集 + 业务密集。 再加一个线程池去处理业务逻辑。
     
## 水平触发和边缘触发
- 边缘触发必须使用非阻塞IO, 水平触发可以使用非阻塞IO也可以阻塞IO
- 边缘触发：如果读缓冲区从空->非空，epoll_wait才返回，写缓冲区满->非满，epoll_wait才返回。
水平触发：如果读缓冲区有数据读，epoll_wait总会返回；如果写缓冲区可写，epoll_wait也会返回。
    水平触发：只要读缓冲区没有全部读完，写缓冲区没有满，都可以读/写。边缘触发：只有读缓冲区满，写缓冲区空，才可以读/写。
 ```cpp
// 水平触发
read(fd, SIZE);
// 边缘触发
while (1) {
    int res = read(fd, buf);
    if (res == eagain) break;
}
// 区别：边缘触发的epoll_wait次数少，效率更高，而且用户态可以缓存内核状态。
/* 
应用：redis, skynet, memcached采用水平触发，而nginx采用边缘触发。
用水平触发很容易理解，因为水平触发很方便写，那为什么nginx要用边缘触发？
这要从nginx的业务触发，nginx是用来作反向代理和静态web服务器，反向代理是把数据流量反射到不同的web服务器，过程中要尽量减少IO操作，适合边缘触发。
*/
 ```
