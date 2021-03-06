# 计算机网络
    记录计算机网络的一些常见问题，看这些问题之前要熟悉TCP/IP协议栈。
不熟悉TCP/IP协议栈的朋友可以写一个[用户网络协议栈](https://github.com/chouring/cs-it-knowledge/tree/main/net/tcpip)<br>

---
已完成：
- HTTP
- TCP和UDP

## HTTP

## TCP和UDP
TCP状态迁移图<br>
![图片](https://user-images.githubusercontent.com/75822806/173028003-8678f131-86ea-40dd-bf61-93bb54999870.png)
### TCP和UDP的对比
    数据格式，有无拥塞控制等等
    TCP：流式套接字，
    UDP：数据包格式，实时性较强，主要用在游戏、直播、下载等场景<br>
    那为什么UDP实时性较强？ 因为TCP有延迟确认机制，而UDP没有<br>
### TCP的流怎么理解？ 
    在应用层send和recv符合：先发的先到 且 数据不丢失

### UDP server fd是否针对大量客户端同时发送？
    可以，UDP服务器可以模拟三次握手。 再分配一个fd和客户端交互

### 大量close_wait的原因
    ret = recv(fd, ), ret == 0 和 close(fd) 之间的时间太长，原因往往是业务逻辑写的不对<br>
    把这两行代码之间的业务做成异步的，或者抛给另一个线程去做<br>
    close_wait出现在四次挥手的被动方，往往是服务器是被动者<br>

### TCP的常见机制
    通过滑动窗口实现flow control，通过拥塞窗口实现congestion control，
    flow是发送与接收的速率匹配，congestion则指的是整个网络的情况。<br>
    在Linux中，congestion control相关算法有Reno, bic, cubic, bbr等, flow control相关算法有nagle，clark算法等等。

### UDP怎么实现可靠性？
    参考TCP解决方式：
    通过把包编号 -> 进行顺序控制，解决乱序问题
    通过滑动窗口 -> 进行流量控制
    通过拥塞窗口 -> 进行拥塞控制
    但是udp可靠性设计很难有一个“普适性”的模式。

