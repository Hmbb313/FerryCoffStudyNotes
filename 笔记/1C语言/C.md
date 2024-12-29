# 一 VS下载

![1](./../../操作/1C语言/1.png)

VS的安装与配置

# 二 main

```.c
#include <stdio.h>

int main()
{
	printf("qwer12");

	return 0;
}
```

# 三 include

include只是复制粘贴

使用库函数必须包含对应的头文件

# 四 注释

两种注释

# 五 变量

变量

int nNum = 123;

# 六 常量

常量

#define PI 3.14



const int nNum = 13;

# 七 栈区的const常量

栈区的const常量可以修改

# 八 关键字

关键字

# 九 转义

转义字符

\r\n

# 十 格式化

格式化输出

%d

# 十一 输入

数据的输入

scanf

# 十二 调试

调试

# 十三 数据宽度

整型

short

int

long

long long

# 十四 数据宽度格式化

%hd

%d

%ld

%lld

# 十五 有符号和无符号

有符号和无符号

# 十六 溢出

有符号数溢出

# 十七 float

浮点型

float f = 3.14f;// 不加f,默认是double

# 十八 char

字符型

# 十九 boolen

bool类型

# 二十 sizeof

sizeof

# 二十一 类型转换

类型转换

强转

# 二十二 算数运算

+

-

*

/

# 二十三 %

%

# 二十四 ++

 

# 二十五 --

--

# 二十六 前++和后++汇编

反汇编看前++和后++



```asm
单独的++Num1和Num1++都是这样
MOV EAX, DWORD PTR [Num1]
ADD EAX, 1
MOV DWORD PTR [Num1], EAX



Num1 = 5
Num2 = ++Num1 + 2

MOV EAX, DWORD PTR [Num1]
ADD EAX, 1
MOV DWORD PTR [Num1], EAX
MOV EAX, DWORD PTR [Num1]
ADD EAX, 2
MOV DWORD PTR [Num2], EAX


Num1 = 5
Num2 = Num1++ + 2

MOV EAX, DWORD PTR [Num1]
ADD EAX, 2
MOV DWORD PTR [Num2], EAX
MOV EAX, DWORD PTR [Num1]
ADD EAX, 1
MOV DWORD PTR [Num1], EAX
```

# 二十七 赋值

赋值

=

# 二十八 比较

比较

<

<=

# 二十九 逻辑

逻辑

&&

||

!

# 三十 运算优先级

运算优先级

# 三十一 if

单if

# 三十二 多行if

多行if

# 三十三 多条件if

多条件if

# 三十四 嵌套if

嵌套if

# 三十五 三目运算符

三目运算符

# 三十六 switch

switch

# 三十七 switch效率大于if

switch的效率大于if

```asm
if...else...

cmp dword ptr[ebp-8], 1
jne 地址

就这样一个一个比较匹配。类似数组遍历。
```

```asm
switch...case...

mov eax, dword ptr[ebp-8]
mov dword ptr [ebp+FFFFFF30h], eax
mov ecx, dword ptr [ebp+FFFFFF30h]
sub ecx, 1
mov dword ptr [ebp+FFFFFF30h], ecx
cmp dword ptr [ebp+FFFFFF30h], case的最大值-1
ja 跳转到default
mov edx, dword ptr [ebp+FFFFFF30h]
jmp dword ptr [edx*4 + case表]


先判断case的最大值，大于最大值，直接default
一个判断，就到了case表
```

# 三十八 while

while

# 三十九 while案例

while猜数字

# 四十 do while

do while

# 四十一 for

for

# 四十二 for循环变形

for循环变形

# 四十三 嵌套循环

嵌套循环

# 四十四 break

```txt
break只跳出一层循环
```

# 四十五 continue

```txt
continue只跳出一层循环
```

# 四十六 goto

goto

# 四十七 for反汇编

```asm
for反汇编分析

// 赋初值
mov dword ptr [ebp-8], 0
jmp 条件表达式

// ++i
mov eax, dword ptr [ebp-8]
add eax, 1
mov dword ptr [ebp-8], eax

// 条件表达式
cmp dword ptr [ebp-8], 5
jge 结束

// 循环内部语句
// 调函数什么的
jmp ++i

// 循环结束
xor eax, eax
pop edi
```

# 四十八 数组

数组

# 四十九 一维数组

一维数组

# 五十

```c
// 数组拓展
int nArr[5] = {0};

printf("整个数组占用的空间大小: %d \r\n", sizeof(nArr));
printf("单个元素占用的空间大小: %d \r\n", sizeof(nArr[0]));
printf("数组元素个数为: %d \r\n", sizeof(nArr) / sizeof(nArr[0]));

printf("数组首地址: %08x \r\n", nArr);
```

# 五十一 遍历数组

遍历找数组最大值

只要大于temp，就赋值给temp

# 五十二 数组元素逆置

数组元素逆置

# 五十三 冒泡排序

冒泡排序

# 五十四 冒泡排序实现

冒泡排序实现

# 五十五 二维数组

二维数组介绍

```c
printf("数组的首地址: %08x \r\n", Arr2);

printf("整个数组占用的内存大小: %d \r\n", sizeof(Arr2));
printf("一行占用的内存大小: %d \r\n", sizeof(Arr2[0]));
printf("一个元素占用的内存大小: %d \r\n", sizeof(Arr2[0][0]));

printf("行数: %d \r\n", sizeof(Arr2) / sizeof(Arr2[0]));
printf("列数: %d \r\n", sizeof(Arr2[0]) / sizeof(Arr2[0][0]));
```

# 五十六 二维数组实现

二维数组实现

# 五十七 二维数组拓展

二维数组拓展

# 五十八 二维数组案例

二维数组案例

# 五十九 字符串

字符串

# 六十 字符串的本质

字符串的本质

```c
char szBuf[] = {'H', 'e', 'l', 'l', 'o'};
printf("%s\r\n", szBuf);// 有问题。因为没有'\0'

char szBuf[] = {"Hello"};
printf("%s\r\n", szBuf);// "Hello"
printf("%d\r\n", sizeof(szBuf));// 5
printf("%d\r\n", strlen(szBuf));// 6
```

# 六十一 字符和字符串

字符和字符串

字符的名字是字符，字符串的名字是地址

字符只是ASCII码，字符串末尾还有一个\0

# 六十二 字符串处理函数

字符串处理函数

```c
strcpy
strcat
strcmp
```

# 六十三 字符串和字符数组

字符串和字符数组

```c
// 字符数组
char szBuf1[] = {'H', 'e', 'l', 'l', 'o'};

// 字符串
char szBuf2[] = {'H', 'e', 'l', 'l', 'o', '\0'};
char szBuf3[] = "Hello";
区别就是\0
```

# 六十四 缓冲区溢出(数组越界)

缓冲区溢出

```c
void TestFun()
{
    int i = 0;
    int Arr[5] = {0};
    for(i = 0; i <=7; i++)
    {
        Arr[i] = 0;
        printf("停不下来 %d \r\n", i);
    }
}
```

# 六十五 函数

函数

# 六十六 函数框架

函数框架

# 六十七 函数常见样式

函数常见样式

# 六十八 函数声明

函数声明

# 六十九 函数传参

函数传参

# 七十、七十一 反汇编看函数流程

反汇编看函数流程

```c
int Add(int a, int b)
{
    int nTemp = a + b;
    return nTemp;
}

int main()
{
    int a = 1;
    int b = 2;
    int nRes = Add(a, b);
}
```



![2](./../../操作/1C语言/2.png)

![3](./../../操作/1C语言/3.png)

![4](./../../操作/1C语言/4.png)

# 七十二 指针

指针

# 七十三 指针操作内存

指针操作内存

# 七十四 指针大小

指针大小

# 七十五 空指针和野指针

空指针和野指针

# 七十六 结构体

结构体

# 七十七 结构体初始化

结构体初始化

# 七十八 结构体指针

结构体指针

# 七十九 结构体数组

结构体数组

# 八十 结构体套结构体

结构体套结构体

# 八十一-九十 卡密验证管理系统

卡密验证管理系统
