# c++基础知识
已完成：
- STL
- 智能指针

---
## STL
STL是C++的标准模板库，是一个封装良好，复用性强的库/软件系统。以下简单介绍它提供的Service，对外暴露的Interface以及各组件的Implementation

### Service
提供一系列低耦合，复用性强的数据结构和算法

### Interface

主要是通过泛型编程，直接实例化使用容器/配接器的function member，把迭代器视为“指针”，通用算法通过迭代器对容器进行操作。以此达到“利用各种算法操作数据结构”的Service。

### Implementation

STL由六大部分组成：
- allocator：空间配置器，用来为所有数据结构分配并管理空间。
- iterator：迭代器，用来封装“指针”，提供通用的类指针功能。
- container：容器，各种数据结构，分为序列式和关联式。序列式有vector, list, deque, stack, queue(stack和queue是由deque而来)等等，关联式有set, map, multi系列，hash系列等等。
- algorithm：算法，各种通用性算法，容器和算法之间通过迭代器结合使用。
- functor：仿函数/函数对象，像函数一样的对象。
- adapter：配接器，STL各个组件的转换器，常用于转换接口。常见的stack和queue其实底层都是通过配接deque实现的。

六大组件的关系：allocator为所有数据结构分配并管理空间，container提供了各种各种的数据结构，这些数据结构本身有一些function member。我们可以通过迭代器去操作容器元素，也可以用迭代器去和某种算法连接，从而修改容器。而仿函数/函数对象充当了一个行为类似函数的对象，配接器为其他STL组件提供了灵活的接口。

具体的实现，可以学习侯捷的《STL源码剖析》（SGI版本）和STL的一些复现。

## 智能指针





