# Design Pattern

## Single Thread Execution mode

### 逻辑描述

保证一个现成在执行

### 几种设计模式

- Guarded Suspension
- Read-Write Lock
- Immutable模式
- Thread-Specific Storage

#### 死锁的问题

- 存在多个SharedResource
- 线程在持有SharedResource角色的锁的同时，还想获取其他SharedResource角色的锁
- 获取SharedResource角色锁的顺序并不固定

#### Synchronized

- 在保护什么
- 该以什么单位来保护
- 是不是所有的保护单位都已经可控了
- 使用哪个锁保护

#### 原子操作

#### 计数信号量+Semaphore

