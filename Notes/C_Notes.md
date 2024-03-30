# **C NOTES**

```C
VirtualProtectEx(hProcess, (LPVOID)addr, 1024, 4, &oldprotect);
printf("1 : %d\n", oldprotect);
//  WriteProcessMemory(hProcess, (LPCVOID)addr, buff, 4, 0);
VirtualProtectEx(hProcess, (LPVOID)addr, 1024, oldprotect, &oldprotect);
printf("2 : %d\n", oldprotect);
VirtualProtectEx(hProcess, (LPVOID)addr, 1024, 4, &oldprotect);
printf("3 : %d\n", oldprotect);
VirtualProtectEx(hProcess, (LPVOID)addr, 1024, oldprotect, &oldprotect);
printf("4 : %d\n", oldprotect);
```

VirtualProtectEx函数的最后一个参数切记不能填**0**，不然修改内存页属性时会出错。如果注释`WriteProcessMemory`函数之后输出为2，8，2，8。如果取消`WriteProcessMemory`函数的注释之后输出为2，4，2，4。结果很奇怪。

完整测试函数为：

```C
#define _CRT_SECURE_NO_WARNINGS
#include <stdio.h>
#include <Windows.h>

int main(int argc, char* argv[], char* envp[])
{
    DWORD pid;
    HWND hWnd =  FindWindowW(NULL, L"Tutorial-i386");// Cheat Engine教程程序
    GetWindowThreadProcessId(hWnd, &pid);
    HANDLE hProcess = OpenProcess(PROCESS_ALL_ACCESS, NULL, pid);
    DWORD addr = 0x400000;
    DWORD oldprotect = NULL;
    unsigned char buff[4] = { 0 };

    VirtualProtectEx(hProcess, (LPVOID)addr, 1024, 4, &oldprotect);
    printf("1 : %d\n", oldprotect);
    // WriteProcessMemory(hProcess, (LPCVOID)addr, buff, 4, 0);
    VirtualProtectEx(hProcess, (LPVOID)addr, 1024, oldprotect, &oldprotect);
    printf("2 : %d\n", oldprotect);
    VirtualProtectEx(hProcess, (LPVOID)addr, 1024, 4, &oldprotect);
    printf("3 : %d\n", oldprotect);
    VirtualProtectEx(hProcess, (LPVOID)addr, 1024, oldprotect, &oldprotect);
    printf("4 : %d\n", oldprotect);
    return 0;
}
```

***

printf函数里面，`\n`是将光标移至下一行，`\r`是将光标移至本行行首，`\b`是将光标前移一位（`\b`不能再回退至上一行）。有输出之后会覆盖掉光标后面的内容。比如`printf("123456\b\b\bab");`的输出就是123ab6。

***

字节模型:

+ win32 ILP32模型（即int，long，pointer都为4字节
+ win64 LLP64模型（即long long，pointer为8字节，long还是为4字节）
+ 类unix系统64位 LP64模型（即long，pointer都为8字节）

| Data Type | ILP32 | LP32 | ILP64 | LP64 | LLP64 |
| :---: | :---: | :---: | :---: | :---: | :---: |
| 宏定义 | - | - | - | \_\_LP64\_\_ | \_\_LLP64\_\_ |
| 平台 | Win32 API / Unix和Unix类的系统（Linux, Mac OS X) | Win16 API | \\ | Unix和Unix类的系统（Linux, Mac OS X) | Win64 API |
| char | 8 | 8 | 8 | 8 | 8 |
| short | 16 | 16 | 16 | 16 | 16 |
| int | 32 | 16 | 64 | 32 | 32 |
| long | 32 | 32 | 64 | 64 | 32 |
| long long | 64 | 64 | 64 | 64 | 64 |
| pointer | 32 | 32 | 64 | 64 | 64 |

***

32位天堂之门：

```C
#define _CRT_SECURE_NO_WARNINGS
#include <stdio.h>
#include <Windows.h>
typedef unsigned long long QWORD;
int main(int argc, char* argv[], char* envp[])
{
    unsigned char func[] =
    {
        0x55, 0x89, 0xE5, 0x81, 0xE4, 0xF0, 0xFF, 0xFF, 0xFF,
        //================进入x64==========================
        0x6A, 0x33,                                         //push 0x33
        0xE8, 0x00, 0x00, 0x00, 0x00,                       //call $0
        0x83, 0x04, 0x24, 0x05,                             //add dowrd ptr [esp], 5
        0xCB,                                               //retf
        //================减法函数=========================
        0x53, 0x48, 0x8B, 0x45, 0x08, 0x48, 0x8B, 0x5D, 0x10,
        0x48, 0x2B, 0xC3, 0x5B, 0x50,
        //================退出x64==========================
        0xE8, 0x00, 0x00, 0x00, 0x00,                       //call $0
        0xC7, 0x44, 0x24, 0x04, 0x23, 0x00, 0x00, 0x00,     //mov dword ptr [rsp + 4], 0x23
        0x83, 0x04, 0x24, 0x0D,                             //add dword ptr [rsp], 0xD 
        0xCB,                                               //retf
        //================进入x32==========================
        0x58, 0x5A,
        0x89, 0xEC, 0x5D, 0xC3
    };
    DWORD oldpro = 0;
    VirtualProtect((LPVOID)func, 1, PAGE_EXECUTE_READWRITE, &oldpro);
    QWORD ans = ((QWORD(*)(QWORD, QWORD))func)(0x23456789ABCDEFFF, 0x1111111111111111);
    printf("%llx\n", ans);
    VirtualProtect((LPVOID)func, 1, oldpro, &oldpro);
    return 0;
}
```

***

位移操作(shl, shr, sar)有操作范围, 不论是32位程序还是64位程序，位移范围都在一字节内，即如shr eax, cl。

***

GCC 编码格式及编码转换：

编码场景：

+ 源码文件的编码格式

  `-finput-charset` 该选项指定GBK，UTF-8等编码格式后由GCC调用host的字符转换功能将源文件的编码格式转换为GCC内部编码格式然后再进行处理。

+ 源码中字符在目标文件中的编码

  可以理解为在源码中定义的字符串在编译进目标文件时和在内存中是如何被表示的。又要分为两类:多字节`(multibyte)`和宽字符`(wide characters)`。

  + `-fexec-charset`
  + `-fwide-exec-charset`

> `-finput`和`-fexec`两个选项可以理解为vscode编码选项中的`Reopen with Encoding`和`Save with Encoding`。
>
> 即`-finput`后指定的编码格式要与源码文件原本的编码格式保持一致，`-fexec`后指定的编码格式是希望储存在编译出的程序二进制文件中的编码格式。
>
> 在VS编译器中，使用Unicode字符集和使用多字节字符集的直接区别是：编译器是否增加了宏定义`UNICODE`。而是否增加该宏定义则决定了Windows API函数参数在需要用到字符串时使用的是多字节字符集还是宽字符字符集。

***

添加<Python.h>时遇到的坑：gcc后加上`-I xxxx\Python\Python310\include` 指令，用以明确路径。

***

```C
#include <stdio.h>
#include <stdlib.h>
#include <Windows.h>

unsigned char buff[1024];
unsigned char tmp[6] = { 0x41, 0x42, 0x43, 0x44, 0x45, 0x46 };
int main()
{
    FILE *fp = fopen("test.txt", "rb+");
    if(fp)
    {
        printf("fp start addr: %p\n", _ftelli64(fp));
        fwrite(tmp, 1, 1, fp);
        //_fseeki64(fp, 0, SEEK_CUR);
        fread(buff, 1, 1, fp);
        fclose(fp);
    }
}
```

此处如果没有`fseek()`来重置某个标志，`fwrite`之后再`fread`的话，`fread`就会清除掉文件中对应大小的内容(就像是用"wb"来打开某个文件)。而如果是`fread`之后再`fwrite`的话，会出现`fwrite`无法成功写入的情况，但是会正确返回预计写入的大小，并且`ftell()`查看指针位置也会发现指针后移了对应的距离(就像是用"rb"来打开某个文件)。尚不清楚具体的原理，需要进一步的逆向分析。

***

mingw创建dll时声明可以在不同进程中共享同一内存地址的变量的方法:

```C
HMODULE ghModule __attribute__((section(".shr"), shared));
HHOOK ghHook __attribute__((section(".shr"), shared));
int gStatus __attribute__((section(".shr"), shared));
```

即用`__attribute__((section(".shr"), shared))` 跟在变量名后面来修饰。
如果用vs平台编译可以用：

```C
#pragma data_seg(".shr")
int gData;
#pragma data_seg()
#pragma comment(linker,"/SECTION:.shr,RWS")
```

***

调用API函数时一定要注意[out]属性的指针所对应变量的字节宽度。比如

```C
BOOL ReadProcessMemory(
  [in]  HANDLE  hProcess,
  [in]  LPCVOID lpBaseAddress,
  [out] LPVOID  lpBuffer,
  [in]  SIZE_T  nSize,
  [out] SIZE_T  *lpNumberOfBytesRead
);
```

对于该函数来说，如果第五个参数传入了一个int类型的变量的指针，就会导致该变量所在内存的后四字节被覆盖(64位程序中)，编译器如果在这后四字节分配了其他的变量那么将会导致不可预估的后果。

***

对gcc编译器来说，

`(unsigned int)0xffffffff == (int)-1`

但是

`(unsigned char)0xff != (char)-1`

```asm
00000000004015A2             | mov eax,dword ptr ss:[rbp-4]
00000000004015A5             | movzx edx,al                 unsigned char
00000000004015A8             | mov eax,dword ptr ss:[rbp-8]
00000000004015AB             | movsx eax,al                 signed char
00000000004015AE             | cmp edx,eax
```

所以在使用字节宽度为1的变量时要注意其是否为**unsigned**。

***

若在a.c文件中定义 `char c[10];`在b.c文件中声明 `extern char *c;`则会出现定义与声明矛盾的错误。因为: 假设声明正确，则在a.c中执行`printf("%p\n", c);`会直接打印出标识符c所在的地址，而在b.c中执行`printf("%p\n", c);`则会打印出标识符c所在地址与一个指针长度相等的内存单元的内容，两者相悖。故在b.c中应声明为`extern char c[];`或`extern char c[10]`。

***

```C
#include <stdio.h>

int main()
{
    int a = 123;
    int *b = &a;
    int(*c)[] = (void*)&b;
    printf("%p %p %d %d\n", c, *c, **(int**)*c, **(int**)c);
    return 0;
}
```

这段代码中`c`和`*c`的值是一样的，`**(int**)*c`和`**(int**)c`才会打印出a的值。因为c解引用之后类型为`int []`，是一个数组，c解引用后的值为数组的地址值而不会再从该地址的内存单元取值，因此c如果在第一次解引用时不转换类型则需要3次解引用才能真正取到a的值。

***

struct结构体的标签在单独的命名空间内。比如定义 `struct tag{…};`再定义 `typedef int tag;`这样并不冲突。`struct tag a` 和 `tag b` 依然可以正确表示两种不同类型的变量。

***

`const`关键字在`*`右边表示指针本身是常量，在`*`左边表示指针所指的对象是常量。比如`char * const *a;`表示指针`a`所指的对象是常量，也即`*a`这个指针本身是常量。

***

C语言链接属性有三种，`external`，`internal`，`none`。全局变量和(全局或局部)函数声明的链接属性默认为`external`，其余皆为`none`。

改变默认链接属性的方式有两种：

1. `extern`作用于默认链接属性为`none`的对象上，可以将链接属性改为`external`（若对象链接属性已经为`internal`，则再用`extern`声明不会再改变对象的链接属性）

2. `static`作用于默认链接属性为`external`的对象上，可以将链接属性改为`internal`（若作用于默认链接属性为`none`的对象上则不会更改链接属性，而是将对象的储存类型更改为静态变量）。

> ```C
> static int a = 1;
> extern int a; // extern出现在static后，正确
> /*****************************************/
> extern int a;
> static int a = 1; // static出现在extern后，错误
>
> // 先声明static函数，在之后定义函数的时候缺省static，正确
> // 定义函数时默认缺省extern，但是不影响之前声明的static，该函数链接属性仍为internal
> static void func();
> void func() { }
> ```

***

C语言变量作用域：

1. 代码块作用域（即局部变量）;
2. 文件作用域（即全局变量）;
3. 原型作用域（函数声明中的变量名不能重复）;
4. 函数作用域（同一个函数内的goto跳转标签名不能重复）。

***

几个比较绕的定义：

```C
int *(*(**c[10])(int**))(int, short);
int (*func(char a, short b))(int,long)
{
    void;
}
int (*f(char a, short b))[3]
{
    void;
}
```

+ `c`是一个数组，数组的成员是指向一个对象的指针的指针，该对象是一个函数，接受一个`int**`类型的参数，返回值是一个指针，该指针指向一个接受`int`和`short`两个参数且返回值为`int*`的函数。
+ `func`是一个函数，接受`short a`和`short b`两个参数，返回值是一个指针，该指针指向一个接受`int`和`long`两个参数且返回值为`int`的函数。
+ `f`是一个函数，接受`char a`和`short b`两个参数，返回值是一个指针，该指针指向一个`int [3]`类型的数组。

由以上推出，指针的星号最好和变量名靠在一起写。

***

c实现默认参数的一种宏定义方式:

```C
#include <stdio.h>

#define TESTARG(a1, a2) ((#a1[0])?(a1+0):a2)

int main()
{
    int a = TESTARG(,3);
    printf("%d\n", a);
    return 0;
}
```

***

```C
#include <stdio.h>

int main(int argc, char *argv[])
{
    int a[4][3];
    printf("%d\n", (int)(a[3] - a[1]));
    return 0;
}
```

注意该段代码输出的值是6而不是2。`a[1]`和`a[3]`两个地址的差值除以的量不是`3 * sizeof(int)`而是`sizeof(int)`。因此`a[3] - &a[1][2]`不会出现语义上的错误。`a[3]`的类型是`int [3]`即`int []`，`&a[1][2]`的类型是`int *`,两个类型兼容。

***

结构体中定义一个长度为零的数组用处是alloc结构体内存时可以自定义数组的长度，即实现结构体内的变长数组。

```C
struct node {
    int a;
    char b;
    char c[]; // 或char c[0];
}
```

若`a`地址为0，则`b`地址为4，`c`地址为5(注意若`c`的类型为`short`则仍会受到内存对齐的影响，即`c`的相对地址为a+6)。`sizeof(struct node)`的值为8。

在结构体中除了可变数组之外的最后一个变量无多余内存对齐时分配内存可以使用:

```C
struct node *p = malloc(sizeof(struct node) + sizeof_arr); // sizeof_arr即此处需要定义的可变数组的大小
```

***

C语言里整除为绝对值的向下取整，取余的计算公式是`x % y = x - ((x / y) * y)`。比如`10 % -3 = 1`。

> Python的整除为向下取整。取余公式同上，但相对应的是`10 % -3 = -2`。

***

C中的宏分为两类:

+ 对象宏 `object-like macro`:

    ```C
    #define 宏名 替换列表 换行符
    ```

+ 函数宏 `function-like macro`:

    ```C
    #define 宏名([标识符列表]) 替换列表 换行符
    // 注意宏名和表示参数的括号之间不能有空白符，否则会把函数宏误判为对象宏
    // 另注：
    // #define f\
    // (a) printf("[+]"#a"\n");
    // 以上换行方式是合法的，需要反斜杠另起的下一行起始不为空白符。
    ```

宏的操作符:

+ Stringizing操作符 `#`

    > 注意点:
    >
    > 1. 字符串化操作符只能对函数宏的参数使用。
    > 2. 参数前后的空白符都会被忽略，中间的空白符会被压缩为一个，注释会被忽略并变成一个空白符。

+ Concatenation操作符 `##`

`...`用来接收任意数量的参数，`__VA_ARGS__`用来代替前者接收的参数。如:

```C
#define f(a, ...) a __VA_ARGS__
f(1, 2, 3, 4)

// 1 2, 3, 4
```

***

## 宏的展开

### 一、对象宏的展开

当展开全程仅有对象宏时是最简单的情况，需要满足以下两点:

1. 对象宏的展开是**深度优先**的递归展开。
2. 在展开过程中产生的宏名称不一定都可以展开。
    > 具体来说即是首先建立一个初始为空集的限定集，然后在递归展开的过程中:
    >
    > + 向下层展开时: (*展开后*)把展开的宏名称加入到限定集中
    > + 向上层返回时: (*返回后*)把当前层的宏名称从限定集中移除
    >
    > 在展开时不去展开限定集中的宏名称

```C
#define a b
#define b c
#define c d
#define d b c end

a
// b c end
```

### 二、函数宏的展开

当展开过程中出现函数宏时，相对于展开全过程中只有对象宏的情况处理思路大致相同，但是还有几点需要特殊处理的地方:

1. 不能展开的宏名称
    > + 当函数宏的参数连接着`#`(*Stringizing*)或连接着`##`(*Concatenation*)时不能展开该位置的参数
    > + 当函数宏名称未连接着括号时不能展开该宏名称
    >
    > ```C
    > #define f(x) g x #x FOO_##x
    > #define g() sth
    > #define A expanded
    >
    > f(A)
    > // g expanded "A" FOO_A
    > ```
    >
2. 参数的展开时机
    > 先将所有参数原封不动替换到函数宏表达式中的各个位置，然后再深度优先递归展开所有参数(不去展开连接着`#`或`##`的参数)，待参数全部完全展开之后，该函数宏才算向下一层展开完毕。此时在限定集中加入该函数宏的宏名称，然后开始下一层的展开
    >
    > ```C
    > #define f(x) g(x)
    > #define g(x) FOO_##x
    > #define A expanded
    >
    > f(A)
    > // FOO_expanded
    > g(A)
    > // FOO_A
    >
    > // 下为在C89标准下常见的输出log行数的技巧：
    > #define HERE "here is: " STR(__LINE__)
    > #define STR(A) _STR(A)
    > #define _STR(A) #A
    > HERE
    > // "here is: xxx"
    > ```
    >
3. 参数的限定集选取
    > 参数的限定集即为该函数宏展开前的限定集
    >
    > ```C
    > #define f(x) x #x
    >
    > f(f(f(x)))
    > // x "x" "f(x)" "f(f(x))"
    > /*=====================*/
    > #define f(x) g(f(0),g(1, 2)) x
    > #define g(x, y) x y
    >
    > f(3)
    >// f(0) 1 2 3
    > ```
    >
4. 如果某支路完全展开后的最后一个对象为不带括号的函数宏名称，那么如何处理这种情况
    > + 若该函数名称在限定集中，则将会被标记为不被展开。如果可以和接下来的括号再结合为一个函数宏，那么仍然不会展开该函数宏
    > + 若该函数名称未在限定集中，则将会被标记为可能能够展开。如果可以和接下来的括号再结合为一个函数宏，则新结合形成的函数宏限定集为其括号所在那一层的限定集，如果满足宏名称不在这个限定集中那么将展开此函数宏
    >
    > ```C
    > #define f() f2
    > #define f2() f
    >
    > f()()()
    > // f2
    > /*=====================*/
    > #define f(x) g() x()
    > #define g() z()
    > #define z() g
    >
    > f(g())
    > // g g()
    > ```
    >

黑魔法 宏的延迟展开

```C
#define EMPTY
#define f(func) func EMPTY
#define g() expanded
f(g)()

// g ()
// 若要完整解析，则需要再套一层函数宏，使得"f(g)()"的展开为某函数宏参数的展开
#define EMPTY
#define f(func) func EMPTY
#define g() expanded
#define EXPAND(a) a
EXPAND(f(g)())

// expanded
```

几个展开示例:

```C
#define FOO_ 0
#define FOO_1 1
#define CAT(x, y) x ## y
CAT(FOO_, 1)

-> 1
//********************
#define FOO_ 0
#define FOO_1 1
#define PRIMITIVE_CAT(x, y) x ## y
#define CAT(x, y) PRIMITIVE_CAT(x, y)
CAT(FOO_, 1)

-> 01
//********************
#define f(a) a #a
#define ax MOV
f(ax)

-> MOV "ax"
//********************
#define f(a) a #a
f(f(a))

-> a "a" "f(a)"
//********************
#define f() ff
#define ff() aa
f()()

-> aa
//********************
#define f() f
f()()

-> f()
//********************
#define f() a()
#define a() b()
#define b() c() d()
#define c() a() b()
f()

-> a() b() d()
//********************
#define f(x) x a
#define a(x) x b
#define b(x) x c
#define c(x) a(x) b(x) a
f(i)(j)(k)(l)(m)

-> i j k l b l c m b
//********************
#define FOO_(x) FOO_1(x)
#define FOO_1(x) FOO_2(x) + 1
#define FOO_2(x) FOO_1(x) - 1
#define BAR(x) FOO_ ## x (12) FOO_2
BAR(1) (5)
//********************
-> FOO_1(12) - 1 + 1 FOO_2(5) + 1 - 1
#define cat(a,b) a ## b
#define xcat(x, y) cat(x, y)
xcat(xcat(1, 2), 3)

-> 123
//********************
#define f a b
#define a g
#define b ()
#define g() expanded
#define h(a) a
h(f)

-> expanded
```

***

C11开始可以在`struct`或`union`中嵌套声明匿名的结构或联合。

```C
struct /*或union*/ tag_a {
    int a;
    float b;
    struct /*或union*/ /*tag_b*/ {
        short c;
        char d;
    };
};
```

若定义`struct tag_a a = { 0 };`则用`a.c`可以直接访问到`struct tag_b`中的成员。

> **注意**
>
> + 在C11以前的标准中不支持。(warning: ISO C99 doesn't support unnamed structs/unions)
> + 注释掉的tag_b不能添加，否则访问`a.c`会报错。(error: 'struct tag_a' has no member named 'c')

如果添加tag_b，则`sizeof(struct tag_a) == 8`，如果未添加tag_b，则`sizeof(struct tag_a) == 12`。

```C
typedef struct _IMAGE_IMPORT_DESCRIPTOR {
    __C89_NAMELESS union {
        DWORD Characteristics;
        DWORD OriginalFirstThunk;
    } DUMMYUNIONNAME;
    DWORD TimeDateStamp;
    DWORD ForwarderChain;
    DWORD Name;
    DWORD FirstThunk;
} IMAGE_IMPORT_DESCRIPTOR;
```

比如以上是`Windows.h`中的IID结构体，里面出现了嵌套的匿名联合，为了避免在使用C89和C99时出现报错的情况，添加了`__C89_NAMELESS`宏用以规避编译器报错。

***

Ubuntu 16.4上执行以下代码会出现错误，输出结果为`result: hello world`。原因是经过编译器的优化导致`res`和`tmp`的起始地址为同一处。修正方法是像注释那样初始化`res`的值。

```C
#include<stdio.h>

void test(char *p, int len)
{
    snprintf(p, len, "%s", "hello world");
}
int main()
{
    char res[512];
    // char res[512] = { 0 };
    {
        char tmp[1024];
        test(tmp, 1024);
    }
    printf("result: %s\n", res);
    return 0;
}
```

***

初始化时赋零值是一个特殊处理的情况。比如:

1. 初始化复合类型变量时`= { 0 }`和`= { 非零值 }`不一样，如下是正确的，但是最后一行换成注释的内容后gcc编译时会触发警告：初始化时缺少括号。
    >
    > ```C
    > struct {
    >     struct {
    >         int a;
    >     };
    >     int b; 
    > } a = { 0 };
    > /* } a = { 1 }; */
    > ```
    >
2. 可以用`((void*)0)`赋值给任意类型指针，包括object pointer和function pointer。但如果是非零值则严格意义上只能赋值给object pointer。
    >
    > ```C
    > void *(*p)() = (void*)0; // 正确
    > void *(*p)() = (void*)1; // 在-Wpedantic编译选项下警告: 
    > // warning: ISO C forbids initialization between function pointer and 'void *'
    > ```
    >

***

```C
char *p1 = NULL;
const char *p2 = NULL;
p2 = p1; // 正确
p1 = p2; // 错误，左值缺少const限定符
/*************************/
char **p1 = NULL;
const char **p2 = NULL;
p2 = p1; // 错误，指针所指对象是两个不同类型的指针
p1 = p2; // 错误，同上
```

把一个指针变量赋值给另一个指针变量时，左值必须**至少**具备右值的全部`const`限定符，但是在强制转换的时候括号内的类型不需全部具备这些限定符。

***

指针类型中包含**object pointer**和**function pointer**两种类型，object pointer之间可以互相转化，function pointer之间也可以互相转化，但是在C标准里object pointer不能和function pointer互相转化，即: 既不能通过隐式转化直接赋值，也不能显式强制转化。但是把一个function pointer类型的变量再取值，得到的是pointer to pointer，这个指针是object pointer类型，即可作为function pointer到object pointer之间的桥梁。

```C
void *p1 = NULL;
void *(*p2)() = p1; // 错误
void *(*p3)() = (void*(*)())p1; // 错误
`warning: ISO C forbids conversion of object pointer to function pointer type [-Wpedantic]`
// 将p1赋值给一个函数指针的正确用法: 
void *(*p4)() = NULL; // NULL的类型虽然是void*，但是由于赋零值的特殊性故可直接赋值.
*(void**)&p4 = p1;
```

***

位域中的值由低位至高位排放。

```C
union un {
    struct {
        unsigned a : 1;
        unsigned b : 3;
        unsigned c : 5;
        unsigned d : 7;
    };
    unsigned short val;
};

int main() {
    union un a = { .a = 1, .b = 3, .c = 5, .d = 7 }; // warning: ISO C90 forbids specifying subobject to initialize
    printf("%d %d %d %d\n", a.a, a.b, a.c, a.d); // 1 3 5 7
    printf("%X\n", a.val); // E57
    return 0;
}
```

***

如果在某函数内部有`static int a = (a const val);`,那么在生成程序的时候会将常量值写入全局变量a，在调用该函数前已经发生赋值，调用函数时不会再触发这个赋值。而如果写成另一种形式：`static int a; a = (a val)`，那么生成程序时将0写入a，后续调用函数时每次都会触发赋值语句，并且值可以为变量，不同于前者只能为常量。

***

```C
typedef char *pchar;
const pchar a = NULL;
```

此处不能把a的声明等效为`const char *a`，a实际上是一个常量指针而非一个指向常量char的指针，即不能修改a的值，可以修改\*a的值。

```C
typedef char *pchar;
const pchar *a;
```

此处a是一个指针，指向的对象是一个常量指针，该常量指针指向的对象是char。

```C
typedef int A[2][3];
const A a = { {1, 2, 3}, {4, 5, 6} };
```

此处变量a的类型为`const int (*)[3]`，而非`int (* const)[3]`。

> 第一段代码中a的声明等效于`char *const a = NULL;`
>
> 第二段代码中a的声明等效于`char *const *a`
>
> 第三段代码中a的声明等效于`int const a[2][3] = { {1, 2, 3}, {4, 5, 6} }`

***

同一语句内连续声明多个变量的情况:

同一语句内多个变量的声明由一个 **基础类型(base type)** 和紧随其后的一个 **声明符(declarator)列表** 组成，比如:

```C
const int a = 10, *pa = &a, *const cpa, *i[1], **p = i;
```

上述基础类型为`const int`，声明符列表为`a, *pa, *const cpa, *i[1], **p`。

> 指针所指对象的限定符算在基础类型中，指针本身的限定符算在声明符中。
>
> `int`和`const int`视作两个基础类型，比如`int a, const b;` 就是一种错误的声明方式，因为它把`int`和`const int`两种不同的基础类型放在了同一个声明语句里。

用typedef的类型来声明变量时用到`const`等限定符的情况:

+ 如果typedef的类型是一个指针，则限定符作用于指针本身而不是指针所指的对象
+ 如果typedef的类型是一个数组，则限定符作用于数组的基础成员(不论是一维数组还是多维数组)

以上是数组和指针的一个重要区别。

比如:

```C
typedef int A[3][4];
const A *p = 0;
// p的类型为 const int (*)[3][4]
typedef int (*B)[4];
const B *q = 0;
// q的类型为 int (* const*)[4]
typedef int *C[3][4];
const C *r;
// r的类型为 int* const (*)[3][4]
```

数组类型与数组成员同等地拥有`const`等限定符。

```C
typedef int A[2][3];
const A a = { 0 }; 
void *p = a; // C23 前 OK ； C23 起错误
// initializing 'void *' with an expression of type 'const A' (aka 'const int[2][3]') discards qualifiers
// 正确用法: const void *p = a;

const int (*b)[3] = 0;
void *pb = b;
// initializing 'void *' with an expression of type 'const int (*)[3]' discards qualifiers
// 正确用法: const void *pb = b;
```

***

\#define替换名称只在所在行之后有效，所在行之前的名称不会被替换。

***

使用宏魔法来代替`##__VA_ARGS__`的一种技巧如下。

```C
#include <stdio.h>

#define BAR(...) printf(FIRST(__VA_ARGS__) "\n" REST(__VA_ARGS__))

#define FIRST(...) FIRST_HELPER(__VA_ARGS__, throwaway)
#define FIRST_HELPER(first, ...) first
#define REST(...) REST_HELPER(NUM(__VA_ARGS__), __VA_ARGS__)
#define REST_HELPER(qty, ...) REST_HELPER2(qty, __VA_ARGS__)
#define REST_HELPER2(qty, ...) REST_HELPER_##qty(__VA_ARGS__)
#define REST_HELPER_ONE(first)
#define REST_HELPER_TWOORMORE(first, ...) , __VA_ARGS__
#define NUM(...) \
    SELECT_10TH(__VA_ARGS__, TWOORMORE, TWOORMORE, TWOORMORE, TWOORMORE,\
                TWOORMORE, TWOORMORE, TWOORMORE, TWOORMORE, ONE, throwaway)
#define SELECT_10TH(a1, a2, a3, a4, a5, a6, a7, a8, a9, a10, ...) a10

int main()
{
    BAR("first test");
    BAR("second test: %s", "a string");
    BAR("val:%d", 1);
    return 0;
}
```

***

在开启O2及以上编译优化时需要注意因违背**strict aliasing**而产生的优化代码之后无法按预期执行的问题。

> gcc和clang会出现，msvc暂未发现。推测是msvc采用了更为保守的优化策略。

```C
#include <stdio.h>

size_t *arr[1000];

// func1()期望因访问0地址内存而异常
// 在开启O2及以上优化后，可以成功执行，
// 原因是arr[0]的值只取了一次，在后续需要读取到该值时由于基于strict-aliasing规则的优化，
// 省略了重复读arr地址储存的值的步骤
void func1() {
    for(int i = 0; i < 1000; i++) {
        arr[0][i] = 0;
    }
}

// func2()期望输出1078523331  3.140000
// 在开启O2及以上优化后，输出3  3.140000
void func2(int *a, float *b) {
    *a = 3;
    *b = 3.14;
    printf("%d  %f\n", *a, *b);
}

int main() {
    arr[0] = (size_t*)arr;
    func1();
    int a = 10;
    func2(&a, (float*)&a);
    return 0;
}
```

`strict-aliasing`规则简言之即是：编译器为了提升效率，会假定**不相容**的指针不会指向同一处内存地址，某个指针对某一内存地址进行**读操作**时不需要考虑不相容的另一个指针是否已经对该地址进行了**写操作**。

> 默认`char*`与所有类型的指针相容。

解决strict-aliasing引起的编码执行效果不符合预期的问题有三种方法：

1. 使用union进行合法的type punning。
2. 在编译选项中添加上`-fno-strict-aliasing`。
3. 使用`volatile`关键字，编译器将会对每次**读操作**或**写操作**进行取值。

> 由于使用`volatile`关键字会使编译器每次对读写操作进行取值，所以编译出的汇编代码效率上比直接添加`-fno-strict-aliasing`开关选项之后编译出的代码效率相对更低。

```C
// 使用volatile关键字的实例如下
// 此时gcc开启O3优化也会如实执行读写操作的每一次取值
#include <stdio.h>

size_t *volatile arr[1000];

void func1() {
    for(int i = 0; i < 1000; i++) {
        arr[0][i] = 0;
    }
}

void func2(volatile int *a, volatile float *b) {
    *a = 3;
    *b = 3.14;
    printf("%d  %f\n", *a, *b);
}

int main() {
    arr[0] = (size_t*)arr;
    func1();
    int a = 10;
    func2(&a, (float*)&a);
    return 0;
}
```

***

不定参数函数的传参过程中，不论是32位编译还是64位编译都会将float类型提升为double类型来传参。

```C
.\test2.c:20:23: warning: 'float' is promoted to 'double' when passed through '...'
     sum += va_arg(ap, float);
                       ^
.\test2.c:20:23: note: (so you should pass 'double' not 'float' to 'va_arg')
```

***

关于赋值的一个小trick:

```C
#include <stdio.h>

struct st {
    int i;
    void *p;
} a = {
    .i = 1,
    .p = &a
};

int main() {
    printf("%p\n", a.p);
    int *p = (void*)&p;
    printf("%p\n", p);
    int i = (i = 10) + 9;
    printf("%d\n", i);
    return 0;
}
```

以上操作皆合法，在CPython源码中常被用于初始化某个结构体变量。

***

类型转换表达式的值类型总是非左值。比如:

```C
int *a;
float *b = 0;
// (float*)a = b; 错误。(float*)a非lvalue
a = (int*)b;
// 或:
*(float**)&a = b;

int *c = 0;
float d = 0;
// c = &(int)d; 错误。(int)d非lvalue
c = (int*)&d;
```

***

`struct`和`union`里可以用一个语句同时定义一个类型名(`enum`，`struct`，`union`)并用该类型声明一个变量，定义的变量名只在结构或联合里可见，但定义的类型名可见范围与该结构或联合本身的可见范围相同。

***

C 中没有`enum`的前置声明(不同于`struct`或`union`)

```C
enum e;
enum e {a, b, c};
// ISO C forbids forward references to 'enum' types
```

`enum`如果定义在`struct`或`union`中，则其大括号内枚举常量的变量名作用域与`struct`或`union`自身变量名的作用域相同。

```C
struct Element {
    enum State { SOLID, LIQUID, GAS } state;
} oxygen = { GAS };
// 类型 enum State 与其枚举常量于此保持可见，例如
void foo(void) {
    enum State e = LIQUID; // OK
    printf("%d %d %d ", SOLID, e, oxygen.state); // 打印 0, 1, 2
}
```

***

早期版本gcc和clang的编译器bug:

x86-64 gcc 4.9.4 和 x86-64 clang 5.0.2及其以往版本中，对于使用`= { 0 }` (*universal zero initializer*)初始化第一个成员为结构体或数组的结构体或数组，在开启`-Wall`的情况下会报错缺少括号，在开启`-Wextra`的情况下会额外报错其他变量缺少初始化器。

```C
struct a {
    struct b {
        int c;
        float d;
    } e;
    int f;
    float g;
} a = { 0 };
// warning: missing braces around initializer(near initialization for 'a.e')
// warning: missing initializer for field 'f' of 'struct a'
// 实际上这么使用没有任何问题，相关报错也在后续版本的gcc/clang中得到修正。
```

为防止编译器对以上情况错误报错，可以在静态储存期范围内定义`const struct a zero_st_a;`，在所有需要用到零初始化器的地方使用`struct a foo = zero_st_a`。

***
