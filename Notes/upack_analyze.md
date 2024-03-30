# UPACK

```C
esi = 0x010011B0  // RAW 0x1B0
push 0x0100739D   // [0x1B0]
// 将导入表两个函数地址压栈
push LoadLibraryA esi[38]
push GetProcAddress esi[3C]
push 0x01013FFF   // [0x1B4]


edi = 0x0101FE28  // [0x1B8]  位于第二节区
esi = 0x010270F0  // 位于第三节区，对应RAW为0xF0
for( ecx = 0x27; ecx > 0; ecx--) {
    DWORD ptr [edi++] = DWORD ptr [esi++]
}
push 0x132 // [0xF0 + 0xA0]  前面跳过了4字节暂时没用到，那4字节是decode函数的地址

// 第一次循环完后将edi值存入ebx中，这是一个结构体的起始地址
ebx = edi
eax = -1
[edi++] = eax
eax = 0
[edi++] = eax
eax = 1
for(ecx = 4; ecx > 0; ecx--) {
    [edi++] = eax;
}
eax = 0x400
for(ecx = 0x1C00; ecx > 0; ecx--) {
    [edi++] = eax;
}
push 0x1001000  // section 1
eax = 0

// 取ebx结构体位于0x58偏移处的DWORD数组首地址
edx = ebx + 0x58 + 0*4
// 调用decode函数
call [esi]


push eax
push edx
eax = ebx[0]


```
