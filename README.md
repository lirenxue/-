# Lrx-MemoryPool
## 项目介绍
本项目是基于 C++ 实现的自定义内存池框架，旨在提高内存分配和释放的效率，特别是在多线程环境下。
该项目中实现的内存池主要分为两个版本，分别是目录 v1 和 v2 ( v3 在 v2 基础上进行了一定优化)，这两个版本的内存池设计思路大不相同。
### v1 介绍
基于哈希映射的多种定长内存分配器，可用于替换 new 和 delete 等内存申请释放的系统调用。包含以下主要功能：
- 内存分配：提供 allocate 方法，从内存池中分配内存块。
- 内存释放：提供 deallocate 方法，将内存块归还到内存池。
- 内存块管理：通过 allocateNewBlock 方法管理内存块的分配和释放。
- 自由链表：使用无锁的自由链表管理空闲内存块，提高并发性能。

项目架构图如下：
![alt text](/memory-pool-main/images/v1.png)

### v2、v3 介绍
该项目包括以下主要功能：
- 线程本地缓存（ThreadCache）：每个线程维护自己的内存块链表，减少线程间的锁竞争，提高内存分配效率。
- 中心缓存（CentralCache）：用于管理多个线程共享的内存块，支持批量分配和回收，优化内存利用率。
- 页面缓存（PageCache）：负责从操作系统申请和释放大块内存，支持内存块的合并和分割，减少内存碎片。
- 自旋锁和原子操作：在多线程环境下使用自旋锁和原子操作，确保线程安全的同时减少锁的开销。
```
+-------------------+
|  应用请求内存     |
+-------------------+
         |
         v
+-------------------+
|   ThreadCache     |
|-------------------|
|  检查本地缓存     |
|  有：直接分配     |
|  无：请求Central  |
+-------------------+
         |
         v
+-------------------+
|   CentralCache    |
|-------------------|
|  检查共享缓存     |
|  有：分配给Thread |
|  无：请求Page     |
+-------------------+
         |
         v
+-------------------+
|    PageCache      |
|-------------------|
|  从操作系统获取    |
|  切分成小块       |
|  返回给Central    |
+-------------------+
```
项目架构图如下：      
![alt text](/memory-pool-main/images/v2.png)

## 编译  
先进入 v1 或 v2 或 v3 项目目录
```
cd v1
```
在项目目录下创建build目录，并进入该目录
```
mkdir build
cd build
```
执行 cmake 命令
```
cmake ..
```
执行 make 命令
```
make
```  
删除编译生成的可执行文件：  
```
make clean
```  
## 运行
```
./可执行文件名
```  
## 测试结果
### v1
#### v1_cmake&make
![alt text](/memory-pool-main/images/v1_cmake&make.png)
#### 1个线程10轮次、3个线程10轮次、3个线程50轮次下的测试情况：
![alt text](/memory-pool-main/images/v1_test.png)
##### 测试代码
[UintTest](/memory-pool-main/v1/tests/UnitTest.cpp)
##### 代码目的
这段代码的目的是测试和比较自定义内存池与标准 new/delete 操作在多线程环境下的性能。通过创建多个线程并发地进行内存分配和释放操作，来评估内存池的效率。
##### 关键组件
类定义：P1, P2, P3, P4 是用于测试的简单类，分别占用不同大小的内存。
BenchmarkMemoryPool 函数：测试自定义内存池的性能。
BenchmarkNew 函数：测试标准 new/delete 的性能。
main 函数：初始化内存池并调用测试函数。
##### 测试流程
初始化：在 main 函数中，首先调用 HashBucket::initMemoryPool() 初始化内存池。
内存池测试：
BenchmarkMemoryPool 函数创建 nworks 个线程。
每个线程执行 rounds 轮次，每轮次进行 ntimes 次内存分配和释放。
使用 newElement 和 deleteElement 函数进行内存操作。
记录并输出总耗时。
标准分配测试：
BenchmarkNew 函数与 BenchmarkMemoryPool 类似，但使用标准 new 和 delete。
记录并输出总耗时。
##### 结果分析
输出格式：%lu个线程并发执行%lu轮次，每轮次newElement&deleteElement %lu次，总计花费：%lu ms
比较：通过比较两次测试的总耗时，评估自定义内存池相对于标准 new/delete 的性能优势。
##### 结论
性能提升：如果自定义内存池的耗时显著低于标准 new/delete，则说明内存池在多线程环境下具有更高的效率。
优化方向：如果内存池性能不佳，可以考虑优化内存池的实现，如减少锁竞争、提高缓存命中率等。
##### 注意事项
线程安全：确保内存池实现是线程安全的，避免数据竞争。
测试环境：在不同硬件和操作系统上测试，可能会得到不同的结果。
参数调整：可以调整 ntimes, nworks, rounds 参数，观察不同负载下的性能表现。
通过上述测试结果可以看出该内存池的性能甚至不如直接使用系统调用 malloc 和 free，所以根据优化方案就有了v2很和v3。

### v2
#### v2_cmake&make
![alt text](/memory-pool-main/images/v2_cmake&make.png)
#### 功能测试结果
![alt text](/memory-pool-main/images/v2_unit_test.png)
##### 测试代码
[UintTest_v2](/memory-pool-main/v2/tests/UnitTest.cpp)
#### 性能测试结果
![alt text](/memory-pool-main/images/v2_perf_test.png)
##### 测试代码
[PerformanceTest-v2](/memory-pool-main/v2/tests/PerformanceTest.cpp)
##### 测试概述
该性能测试套件主要测试内存池在不同场景下与系统默认分配器(new/delete)的性能对比。测试涵盖了以下几个主要方面：<br>
-小对象分配性能<br>
-多线程并发性能<br>
-混合大小分配性能<br>
##### 预期结果
小对象分配：内存池应显著优于系统分配<br>
多线程场景：内存池应有并发优势<br>
混合大小：性能差异取决于具体分配模式<br>
这个测试套件提供了全面的性能评估，帮助了解内存池在不同场景下的性能特征。
### v3
#### v3_cmake&make
![alt text](/memory-pool-main/images/v3_cmake&make.png)
#### 功能测试结果
![alt text](/memory-pool-main/images/v3_unit_test.png)
##### 测试代码
[UintTest_v3](/memory-pool-main/v3/tests/UnitTest.cpp)
#### 性能测试结果
测试结果表明内存池v3的性能要略好于内存池v2。<br>
![alt text](/memory-pool-main/images/v3_perf_test.png)
##### 测试代码
[PerformanceTest-v3](/memory-pool-main/v3/tests/PerformanceTest.cpp)
