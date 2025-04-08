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
这个版本实现了一个高效的内存池，旨在优化内存分配和释放的性能，特别是在多线程环境下。
内存池通过分层缓存架构来管理内存，主要包括以下三层：
1. ThreadCache（线程本地缓存）
每个线程独立的内存缓存
无锁操作，快速分配和释放
减少线程间竞争，提高并发性能
2. CentralCache（中心缓存）
管理多个线程共享的内存块
通过自旋锁保护，确保线程安全
批量从PageCache获取内存，分配给ThreadCache
3. PageCache（页缓存）
从操作系统获取大块内存
将大块内存切分成小块，供CentralCache使用
负责内存的回收和再利用。
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


通过上述测试结果可以看出该内存池的性能甚至不如直接使用系统调用 malloc 和 free，所以根据优化方案就有了v2很和v3。

### v2
#### v2_cmake&make
![alt text](/memory-pool-main/images/v2_cmake&make.png)
#### 功能测试结果
![alt text](/memory-pool-main/images/v2_unit_test.png)
#### 性能测试结果
![alt text](/memory-pool-main/images/v2_perf_test.png)

### v3
#### v3_cmake&make
![alt text](/memory-pool-main/images/v3_cmake&make.png)
#### 功能测试结果
![alt text](/memory-pool-main/images/v3_unit_test.png)
#### 性能测试结果
测试结果表明内存池v3的性能要略好于内存池v2。
![alt text](/memory-pool-main/images/v3_perf_test.png)
