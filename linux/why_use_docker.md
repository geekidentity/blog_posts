1. 下一代虚拟化技术，潮流，要跟上
2. 隔离用户进程，在一台服务器上，利用多进程的方式实现分布式，每个进程作为一台机器
3. docker可以不用安装操作系统，直接通过宿主机的OS虚拟化出应用
4. 更高效利用硬件，使发挥硬件最大价值。
5. 
 - | 传统虚拟化 | 容器虚拟化
---|---|---
创建速度 | 很慢 | 非常快
性能影响 | 通过对于硬件层的模拟，增加了系统调用链路的环节，有性能损耗 | 共享Kernel，几乎没有性能损耗
资源消耗 | 很大 | 很小，一台机器可以轻松创建多个Container
操作系统覆盖 | 支持Linux，Windows，Mac等 | Kernel所支持的OS

5. Docker支持 Windows, Mac
6. Linux 内核基于 namespace的隔离机制和cgroup 的资源控制机制。
7. 只需要开发环境、生产环境就可以了


docker使用aufs 来实现分层的文件系统的管理
只读部分定义为Image，可写部分是Container
Image类似一个单链表系统，每个Image包含一个指向parent image的指针
没有parent image的image是base image

kubernetes