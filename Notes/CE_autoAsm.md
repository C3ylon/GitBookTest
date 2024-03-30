# Cheat Engine AutoAssemble 函数说明

```
一、目前已支持的CE AA功能如下
alloc
label
registersymbol
unregistersymbol
globalalloc
fullaccess (有某种限制, 不是所有内存区都有效)
aobscan (支持使用??或*表示未知字节)
createthread (此命令不管写在哪里, 永远是脚本中最后运行)
define
dealloc
assert (支持使用??或*表示未知字节)
readmem
地址:行可用地址表达式 (请参看 GetAddress 函数说明)
DB, DW, DD, DQ 命令, 分别为单, 双, 四, 八字节数组写入
#123 表示十进制
(int)123 同样表示十进制
(float) 单精度浮点数(4字节)
(double) 双精度浮点数(8字节)
例:
mov [eax],(float)3.5
dd (float)123.5 (float)100
dq (double)200.75
注释方式有三种: //单行 /*多行*/ {多行}
支持调用游戏空间中存在的 Win API 符号
例:
在游戏中调用两个API
先调用MessageBoxA显示一个"BYE BYE"信息
接着调用ExitProcess结束游戏进程
[enable]
alloc(newmem,4096)
label(text)
newmem:
push 0
push 0
push text
push 0
call messageboxA
push 0
call exitprocess
ret
text:
db 42 59 45 20 42 59 45 0 // "BYE BYE"
createthread(newmem)
[disable]
dealloc(newmem)
二、自定义符号表与人造指针
在AA脚本中可用如下方式加入自定义符号
registersymbol(pointer)
alloc(pointer,4)
或者也可只用一行
globalalloc(pointer,4)
以上两种方式将pointer这个符号加入自定义符号表
并且动态分配了4字节的内存空间给pointer
我们不必知道这个4字节空间的正确地址
我们只要用pointer这个名称就可以代表该地址了
接下来可以把pointer当做人造指针, 在其内放入游戏数据的有效地址
由于自定义符号表为所有使用AA引擎的脚本共享
包含所有其他AA脚本, 在本脚本启动以后, 都可以直接以[pointer]当做地址使用
由于pointer是代表着由alloc分配的动态地址
人造指针实际是存放在地址pointer处
因此一般都是以[pointer]的形式来使用
也可以使用GetAddress函数, 直接取得pointer自身的地址, 或其内人造指针[pointer]的地址
当不再使用pointer的时候, 可以如下方式取消pointer符号名及分配的内存空间
unregistersymbol(pointer)
dealloc(pointer)
七、汇编代码编写注意:
由于使用了OD的汇编编译器, 其汇编代码格式较CE严格
不过我已经加入了汇编代码自动修正功能
例如原先必须写为 mov dword ptr [exa+ebx*4+0c],0c8 的代码
已经可以直接写为 mov [eax+ebx*4+c],c8
如果还是出现编译错误, 表示自动修正无法修正, 请依下列方式自行修正代码
1). 出现"Unknown identifier"编译错误时:
可能为汇编编译器无法正确识别寄存器, 指令, 与数值
解决方法为, 所有数值, 如果是A到F等英文字母开头的, 前面加一个0
如c要写成0c, A8B300要写成0A8B300
2). 出现"Command does not support given operands"
或"Unrecognized command"错误时:
可能为数据长度错误
一般情况, 如果没有指名长度, 则将一律视为4字节长度
例如 mov [esi+30],3f800000
将自动转换为 mov dword ptr [esi+30],3f800000 再送给编译器
若出现这个编译错误, 则请自行指定正确的数据长度
数据长度的表示法有以下几种:
单字节 byte ptr
双字节 word ptr
四字节 dwrod ptr
八、目前不支持的CE AA功能:
loadlibrary
loadbinary
include
九、外部代码区:
CE的AA脚本分为三个区段: [ENABLE], [DISABLE], 外部代码(outer codes)区
外部代码区指的是在[ENABLE]之前的区域, 或可称为前置区
放在这里的代码, 为无论选择[ENABLE]或[DISABLE], 都会被运行
会放在这里的一般是aobscan
十、本AA引擎专有功能:
assert2(地址,字节数组,"选填错误信息")
功能同 assert, 可选填错误信息
例: assert2(00081048,12 ff 3c 78,"Wrong Game Version!")
aobscan2(名称,字节数组,起始地址,结束地址,第几个命中目标,"选填错误信息!")
功能同 aobscan, 设定项目较多, 若不要显示错误信息, 则使用 "" 表示无信息
例1: aobscan2(aobstart,12 34 56 78,00400000,7FFFFFFF,1,"Wrong Game Version!")
例2: aobscan2(aobstart,12 ff 3c 78,0,7FFFFFFF,1,"")
上例2等同于aobscan
十一、其他注意事项:
AA引擎以脚本内容分辨不同的脚本
一个脚本运行[ENABLE]之后, 在运行[DISABLE]之前
不能对该脚本进行任何改变, 包含任何空白回车或注释
否则会被视为不同脚本, 则dealloc等命令将无法正确运作
```

## GetAddress 函数说明

接受地址表达式, 返回最终地址

地址表达式目前支持:

1. 基地址+-*运算
2. 指针表达式 [[[xxxxx+xx]+xx]+xx]+xx

    例:[[0040de00]-4]+c

3. 模块基址+偏移, (本功能需要Vista或XP SP1以上支持)

    不分英文大小写, 若要区分大小写可使用 " " 包住模块名

    例:

    ```
    game.exe
    game.exe+27D0
    game.dat+38c
    "Game.exe"+27D0
    kernel32.dll
    ```

4. 可使用由AA脚本生成的符号(人造指针)
    
    在AA脚本中加入一个自定义符号后, 可直接使用

    例:

    ```
    假若该符号为pointer
    pointer // 取得pointer符号自身被alloc分配的地址
    [pointer] // 取得pointer内存放的人造指针
    ```
