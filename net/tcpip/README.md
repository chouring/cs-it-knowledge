# TCP/IP protocol stack
![图片](https://user-images.githubusercontent.com/75822806/174202113-e6a6e412-7b49-4dc2-b860-b7c91be0fd0d.png)
    
    应用层和内核层之间是socket, 物理层和内核层之间是网卡（进行A/D或者D/A转换）
    接收一个数据包时，在系统中经历的顺序是：网卡 --> 协议栈 --> 应用。可以用DMA方式，网卡存储空间映射到内存。
    实现一个TCP/IP协议栈，是学习TCP/IP的最好方法。
    
