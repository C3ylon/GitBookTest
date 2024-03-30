斐波那契的四种实现方法：

```C
#include <stdio.h>
#include <stdlib.h>

int feibo1(int n)
{
    if(n == 1 || n == 2)
        return 1;
    return feibo1(n - 1) + feibo1(n - 2);
}

int feibo2(int n)
{
    int sum = 0;
    int *arr = (int*)malloc(1000*4);
    int p = 0;
    arr[p] = n;
    while(p >= 0)
    {
        if(arr[p] == 1 || arr[p] == 2)
        {
            sum += 1;
            p--;
        }
        else
        {
            n = arr[p];
            arr[p++] = n - 2;
            arr[p] = n - 1;
        }
    }
    return sum;
}
// int feibo1(int n)
//     if(n == 1 || n == 2)
//         return 1;
//     int a = feibo1(n - 1);
//     int b = feibo1(n - 2);
//     int sum = a + b;
//     return sum;

//==========basic stack to loop=============
int feibo3(int n)
{
    int *stack = (int*)malloc(1000*4);
    int p = -1;
    int ifcall = 1;
    int pos = 0;
    int res = 0;
    int a = 0;
    int b = 0;
    int sum;
    while(1)
    {
        while(ifcall)
        {
            while(ifcall)
            {
                ifcall = 0;
                if(!pos && (n == 1 || n == 2))
                {
                    if(p < 0) return 1;
                    pos = stack[p];
                    res = 1;
                    stack[p] = res;
                }
                if(!pos)
                //feibo(n - 1)
                {
                    stack[++p] = a;
                    //stack[++p] = b;
                    stack[++p] = n;     //arg
                    stack[++p] = 1;     //pos
                    n -= 1;
                    ifcall = 1;
                }
                if(pos == 1)
                {
                    pos = 0;
                    a = stack[p--];
                    n = stack[p--];
                    //b = stack[p--];
                    p--;
                }
            }
            if(!pos)
            {
                stack[++p] = a;
                //stack[++p] = b;
                stack[++p] = n;     //arg
                stack[++p] = 2;     //pos
                n -= 2;
                ifcall = 1;
            }
            if(pos == 2)
            {
                pos = 0;
                b = stack[p--];
                n = stack[p--];
                //p--;
                a = stack[p--];
            }
        }
        sum = a + b;
        if(p < 0) return sum;
        pos = stack[p];
        res = sum;
        stack[p] = res;
        ifcall = 1;
    }
}
// most quick solution
int feibo4(int n)
{
    if(n == 1||n == 2)
        return 1;
    int a = 1;
    int b = 1;
    int count = (n + 1) / 2;
    while(--count)
    {
        a += b;
        b += a;
    }
    if(n % 2) return a;
    return b;
}

int main()
{
    printf("%d %d %d %d\n", feibo1(9), feibo2(9), feibo3(9), feibo4(9));
    return 0;
}
```
