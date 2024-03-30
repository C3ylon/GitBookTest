ALU(Arithmetic&logical Unit)  PC(Program Counter)

寄存器文件(register file)

指令集架构/微体系结构

高速缓存储存器(cache memory) 有近似于寄存器文件的读取速度，但是容量比之于主存而言很小。

I/O桥连接系统总线，内存总线，IO总线。

主存-DRAM 高速缓存-SRAM

四个抽象概念：文件(IO设备)->虚拟内存(+主存)->进程(+处理器[指令集架构])->虚拟机(+操作系统)。

阶码：`exponent`  尾数：`significand`(对于规格化来说默认加1，非规格化不加1)

+ 规格化阶码：不全为0&&不全为1。其表示的值E = e – Bias，其中Bias = 2^(k - 1) – 1。

    对于规格化阶码：float：k = 8，阶码值范围在-126~+127；double：k = 11，阶码值范围在-1022~+1023。

+ 阶码全0：非规格化(E = 1 – Bias)。阶码全1：尾数全0：无穷大；尾数不为0：NaN(Not a Number)。
