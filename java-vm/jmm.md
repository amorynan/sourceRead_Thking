Java 中的内存模型 —— 是一种屏蔽顶层操作系统对于内存读取的抽象规范

主要是分成了主内存 和 工作内存

主内存 -- 简单理解成物理内存，但是不完全等，因为JAVA VM其实也是一个进程占用的系统中的一部分的物理内存的嘛，这一部分被所有线程共享，主内存中储存着一个共享变量（静态变量或者堆中的实例对象）

工作内存 -- 简单理解成 CPU中的高速缓存，但不完全占有，因为每一个线程都有着自己的内存区，在这里面储存着主内存中的副本对象

线程对共享变量的操作只能在工作内存中，而不能直接对主内存进行操作，因为主内存操作的速度太慢了，但是变量值的传递又只能通过主内存来传，线程之间是没有办法访问彼此的

而在JVM中定义了八种操作内存的原子指令

read : major mem -- 读主内存的变量出来

load : work mem -- 将主内存的变量读出来放到工作内存的副本中

store : work mem -- 传到主内存

write : major mem -- 主内存在store后进行写入

assign : work mem -- 赋值 （工作内存中的赋值）

use :  work mem -- 将此线程中的工作内存的变量值给执行引擎，（当有字节码指令中有use）

lock : major mem

unlock : major mem

所以线程中的工作内存和主内存中之间的交互（普遍变量之间） --  read（读主内存）－》load\(加载到工作内存\) -- \(进行工作内存中的各种操作，由字节码指令决定 assign , use \)  —》store \(工作内存储存准备传至主内存\)  —》write（主内存写入） 





