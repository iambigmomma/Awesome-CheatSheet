[![返回目录](https://parg.co/UCb)](https://github.com/wxyyxc1992/Awesome-CheatSheet)

# Web 架构缓存概览：从 CPU 到浏览器

# CPU Cache

由于内存的访问速度与 CPU 中寄存器的访问速度相去甚远，当我们需要对同一批数据(譬如数组或循环计数变量)进行多次操作时，CPU 会自动将值放到 CPU 缓存而不是内存中。CPU 缓存的访问速度仅次于 CPU 寄存器。其容量远小于内存，但速度却可以接近处理器的频率。当处理器发出内存访问请求时，会先查看缓存内是否有请求数据。如果存在(命中)，则不经访问内存直接返回该数据；如果不存在(失效)，则要先把内存中的相应数据载入缓存，再将其返回处理器。

![](http://ifeve.com/wp-content/uploads/2013/03/cpu.png)

随着多核的发展，CPU 缓存又被细分为了 L1, L2, L3 这三个级别，级别越小越接近于 CPU，并且容量越小；一般来说，每个核上都会包含用于存放数据的 L1d Cache 与存放指令的 L1i Cache，以及独立的 L2 Cache，而往往同一个 CPU 插槽之间的核会共享 L3 Cache。在 Linux 设备上 `cat /proc/cpuinfo` 或者 `lscpu` 查看 CPU 情况。

| 存储单元 | 体积        | 访问所需 CPU 周期 | 访问所需时间 |
| -------- | ----------- | ----------------- | ------------ |
| 寄存器   |             | 1 cycle           |              |
| L1 Cache | 2KB~64KB    | ~3-4 cycles       | ~0.5-1 ns    |
| L2 Cache | 256KB~512KB | ~10-20 cycles     | ~3-7 ns      |
| L3 Cache | 1MB~8MB     | ~40-45 cycles     | ~15 ns       |
| 跨槽传输 |             |                   | ~20 ns       |
| 内存     |             | ~120-240 cycles   | ~60-120ns    |

结构上，一个直接映射(Direct Mapped)缓存由若干缓存块(Cache Block，或 Cache Line)构成。每个缓存块存储具有连续内存地址的若干个存储单元。在 32 位计算机上这通常是一个双字(dword)，即四个字节。因此，每个双字具有唯一的块内偏移量。每个缓存块有一个索引(Index)，它一般是内存地址的低端部分，但不含块内偏移和字节偏移所占的最低若干位。一个数据总量为 4KB、缓存块大小为 16B 的直接映射缓存一共有 256 个缓存块，其索引范围为 0 到 255。在 Linux 设备上我们可以查看机器缓存行的大小：

```sh
$ cat /sys/devices/system/cpu/cpu0/cache/index0/coherency_line_size
  64
```

# 缓存策略
