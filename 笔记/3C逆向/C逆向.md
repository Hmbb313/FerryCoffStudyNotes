# 一 函数调用过程

```txt
函数调用，跟C语言部分笔记一样
```

# 二 裸函数里实现函数调用过程

```c
// 裸函数里面，编译器不会自动加push ebp, mov ebp, esp那些
// 裸函数里面只能写汇编
void __declspec(naked) Func()
{
    __asm
    {
        PUSH EBP
        MOV EBP, ESP
        SUB ESP, 0x40
            
        PUSH EBX
        PUSH ESI
        PUSH EDI
            
        LEA EDI, DWORD PTR DS:[EBP-0x40]
        MOV EAX, 0xCCCCCCCC
        MOV ECX, 0x10
        REP STOSD
            
        
        POP EDI
        POP ESI
        POP EBX
        
        MOV ESP, EBP
        POP EBP
            
        RET
    }
}
```

# 三 局部变量

call指令占五个字节，所以call指令的下一条指令就在+5的位置

保存EBX,ESI,EDI的原因是可能外部也会使用这几个寄存器，保证进函数和出函数的时候这几个寄存器的值相同



函数的局部变量是通过[EBP-一个值]来存储

# 四 裸函数里实现局部变量

```c
// 写个带有局部变量的裸函数
void __declspec(naked) Fun()
{
    __asm
    {
        PUSH EBP
        MOV EBP, ESP
        SUB ESP, 0x40
           
        PUSH EBX
        PUSH ESI
        PUSH EDI
        
        MOV EAX, 0xCCCCCCCC
        MOV ECX, 0x10
        LEA EDI, DWORD PTR DS:[EBP-0x40]
        REP STOSD
            
        MOV DWORD PTR DS:[EBP-4], 1
        MOV DWORD PTR DS:[EBP-8], 2
        MOV EAX, DWORD PTR DS:[EBP-4]
        ADD EAX, DWORD PTR DS:[EBP-8]
        MOV DWORD PTR DS:[EBP-0xC], EAX
            
        POP EDI
        POP ESI
        POP EBX
            
        ADD ESP, 0x40
        MOV ESP, EBP
        POP EBP
        RET
    }
}
```

# 五 函数返回值

函数的返回值，就是把返回值放到EAX中

# 六 裸函数里实现函数返回值

```c
// 写个带返回值的裸函数

int __declspec(naked) Func()
{
    __asm
    {
        PUSH EBP
        MOV EBP, ESP
        SUB ESP, 0x40
        
        PUSH EBX
        PUSH ESI
        PUSH EDI
            
        LEA EDI, DWORD PTR DS:[EBP-0x40]
        MOV EAX, 0xCCCCCCCC
        MOV ECX, 0x10
        REP STOSD
         
        MOV DWORD PTR DS:[EBP-4], 0xA
        MOV DWORD PTR DS:[EBP-8], 0x5
        MOV EAX, DWORD PTR DS:[EBP-4]
        SUB EAX, DWORD PTR DS:[EBP-8]
        
        POP EDI
        POP ESI
        POP EBX
            
        ADD ESP, 0x40
        MOV ESP, EBP
        POP EBP
        RET
    }
}
```

# 七 函数传参

函数参数的传递

call之前，先push参数



[EBP+4]，返回地址

[EBP+8]，第一个参数

[EBP+0xC]，第二个参数



call之后，add esp 8，平衡堆栈(函数内部pop ebp，ret之后，只改变了ebp和eip，esp在函数外部平栈)

# 八 裸函数里实现函数传参

```c
// 带参数的裸函数
int __declspec(naked) Func(int a, int b)
{
    __asm
    {
        PUSH EBP
        MOV EBP, ESP
        SUB ESP, 0x40
            
        PUSH EBX
        PUSH ESI
        PUSH EDI
        
        LEA EDI, DWORD PTR DS:[EBP-0x40]
        MOV EAX, 0xCCCCCCCC
        MOV ECX, 0x10
        REP STOSD
            
        MOV EAX, DWORD PTR DS:[EBP+8]
        ADD EAX, DWORD PTR DS:[EBP+0xC]
        MOV DWORD PTR DS:[EBP-4], EAX
        MOV EAX, DWORD PTR DS:[EBP-4]
        
        POP EDI
        POP ESI
        POP EBX
        
        ADD ESP, 0x40
        MOV ESP, EBP
        POP EBP
        RET
    }
}
```

# 九 函数嵌套调用

嵌套调用

# 十 __cdelc

__cdelc，C语言的默认调用约定

从右向左依次入栈

调用者平衡堆栈

# 十一 __stdcall

__stdcall

从右向左依次入栈

被调用者自身平衡堆栈



ret 10h(pop eip, sub esp 10h)

# 十二 __fastcall

__fastcall

前两天参数用ECX和EDX传，剩余参数用栈传



保存栈环境(ebx,esi,edi)的时候还push了一个ecx

![1](./../../操作/3C逆向/1.jpg)

![2](./../../操作/3C逆向/2.jpg)

# 十三 函数堆栈总结

函数堆栈总结

# 十四 用EBP来堆栈回溯

堆栈回溯(EBP)

用EBP来找是谁调用的我这个函数

call 一个地址，这个指令占五个字节

![3](./../../操作/3C逆向/3.jpg)

![4](./../../操作/3C逆向/4.jpg)

# 十五 用ESP来堆栈回溯

堆栈回溯(ESP)



函数头刚进去的时候，ESP保存的是返回地址，可以一层一层回溯。找main函数的特征码

![5](./../../操作/3C逆向/5.jpg)

```c
#include <stdio.h>
#include <stdlib.h>

// 地址为41114a
void Fun()
{
}

int main()
{
	printf("%x \r\n", Fun);

	Fun();

	return 0;
}
```

先x32dbg打开，然后ctrl+g到41114a，就是Fun函数的地址，然后用ESP一层层回溯

![6](./../../操作/3C逆向/6.jpg)

![7](./../../操作/3C逆向/7.jpg)

![8](./../../操作/3C逆向/8.jpg)

![9](./../../操作/3C逆向/9.jpg)

![10](./../../操作/3C逆向/10.jpg)

![11](./../../操作/3C逆向/11.jpg)

![12](./../../操作/3C逆向/12.jpg)

![13](./../../操作/3C逆向/13.jpg)

![14](./../../操作/3C逆向/14.jpg)

![15](./../../操作/3C逆向/15.jpg)

![16](./../../操作/3C逆向/16.jpg)

# 十六 局部变量作用域，生命周期

局部变量特性

作用域

不同函数里的局部变量的堆栈都不一样，局部变量是[ebp-值]来访问的



生命周期

不同函数内局部变量生命周期不同，因为函数执行完了会销毁堆栈



块作用域，是编译器限制。都是存在同一个栈的

```c
int Func()
{
    int a = 0;
    int b = 1;
    if (a > 5)
    {
        int b = 10;
        b = 20;
    }
}
```

# 十七 全局变量作用域，生命周期

全局变量是进程作用域

整个进程期间都能使用全局变量

```c
int g_nTemp = 0;

int main()
{
	g_nTemp = 1;// MOV DWORD PTR DS:[0041F23C], 1

	return 0;
}
```

全局变量在编译好的那一刻就已经确定了

后定义的全局变量比先定义的全局变量的内存地址大



# 十八 CE搜索修改全局变量

CE搜全局变量，改全局变量

# 十九 宏常量

宏常量，只是做文本替换

```c
#define Add(x, y) ((x) + (y))
```

# 二十 const修饰局部、全局常量

const修饰的常量

const修饰的局部常量

只是写代码的时候不让写，实际就是[ebp-8]之类的，可以用指针修改



const修饰的全局常量

用指针也无法修改了，因为全局常量内存的地方没有写权限。要修改全局常量内存的权限属性

# 二十一 负数的汇编

整型，存储规则

负数是用补码存储

```asm
mov al, 7
neg al
```

# 二十二 IEEE

IEEE

float，1位符号位，8位指数位(127+指数)，剩下的尾数

double，就是1位符号位，11位指数位(1023+指数)

# 二十三 浮点数不精准

不精准的浮点数

尾数占23位，小数部分一直算到23位就行

# 二十四 浮点寄存器中转

浮点数是用的浮点寄存器(现在很少用了)，普通数据是用的通用寄存器



![17](./../../操作/3C逆向/17.jpg)

```asm
MOVSS XMM0, DWORD PTR DS:[00417BD0h]
MOVSS DWORD PTR DS:[EBP-8], XMM0

MOVSS XMM0, DWORD PTR DS:[00417BCCh]
MOVSS DWORD PTR DS:[EBP-14h], XMM0

MOVSS XMM0, DWORD PTR DS:[EBP-8]
ADDSS XMM0, DWORD PTR DS:[EBP-14h]
MOVSS DWORD PTR DS:[EBP-8], XMM0
```

# 二十五 浮点数作为参数压栈

浮点数作为参数压栈

```asm
push ecx
movss xmm0, dword ptr ds:[ebp-8]
movss dword ptr ds:[esp], xmm0	;先push一个值，然后改esp的值。
```

# 二十六 浮点数做函数返回值

浮点数做函数返回值

```asm
// 函数内
FLD ST(0), DWORD PTR DS:[00417BD0h]// 存到ST，不用存到EAX返回

// 函数外,main内
FSTP DWORD PTR SS:[EBP-8], ST(0)
```

# 二十七 CE搜索浮点数

CE搜索浮点数

# 二十八 ASCII编码(char)

ASCII编码(char)

# 二十九 UNICODE编码(wchar_t)

UNICODE(wchar_t)

```c
#include <locale.h>

int main()
{
	setlocale(LC_ALL, "chs");

	wchar_t wch = L'中';
	wprintf(L"%lc \r\n", wch);   
}
```

# 三十 bool类型

bool

# 三十一 类型转换，无符号数

类型转换，无符号数

```asm
MOV BYTE PTR DS:[EBP-5], 12h
MOVZX AX, BYTE PTR DS:[EBP-5]
MOV WORD PTR DS:[EBP-14h], AX

MOV EAX, 0FFFFh
MOV WORD PTR DS:[EBP-8], AX
MOVZX EAX, WORD PTR DS:[EBP-8]
MOV WORD PTR DS:[EBP-14h], EAX

MOV WORD PTR DS:[EBP-8], 0x12345678h
MOV AX, WORD PTR DS:[EBP-8]
MOV AX, WORD PTR DS:[EBP-14h]
```

# 三十二 类型转换，有符号数

类型转换，有符号数

```asm
MOV BYTE PTR DS:[EBP-5], 12h
MOVSX AX, BYTE PTR DS:[EBP-5]
MOV WORD PTR DS:[EBP-14h], AX

MOV WORD PTR DS:[EBP-8], 0x12345678h
MOV AX, WORD PTR DS:[EBP-8]
MOV AX, WORD PTR DS:[EBP-14h]
```

# 三十三 位运算，与或非

位运算，与或非

```asm
MOVSX EAX, BYTE PTR [ch1]
MOVSX ECX, BYTE PTR [ch2]
AND EAX, ECX
MOV BYTE PTR [ch3], al
```

# 三十四 位运算，左移右移

位运算，左移右移

```asm
// 无符号数左移，补0
MOVZX EAX, BYTE PTR [ch1]
SHL EAX, 1
MOV DWORD PTR [ch1], AL

// 无符号数右移，补0
MOVZX EAX, BYTE PTR [ch1]
SAR EAX, 3
MOV DWORD PTR [ch1], AL

// 有符号数左移，补0.跟无符号数一样

// 有符号数右移，补符号位
```

# 三十五 溢出，CF

```asm
	inc dword ptr [a]


	unsigned int a = 0xFFFFFFFF;
	int b = 0;
	__asm
	{
		add [a], 1// 结果CF为1
		adc [b], 0// b为1
	}
```

# 三十六 IMUL，CDQ，IDIV

```asm
MOV ECX, 2
MOV EAX, 4
IMUL ECX	     // 结果存在EAX

IMUL EAX, ECX    // 结果存在EAX

IMUL EDX, ECX, 4 // 结果存在EDX


num3 = num1 / num2

MOV EAX, DWORD PTR [num1]
CDQ							// 将EAX的符号位填充到EDX，填满
IDIV EAX, DWORD PTR [num2]
MOV DWORD PTR [num3], EAX

num3 = num1 % num2

MOV EAX, DWORD PTR [num1]
CDQ							// 将EAX的符号位填充到EDX，填满
IDIV EAX, DWORD PTR [num2]
MOV DWORD PTR [num3], EDX	// 余数放到EDX中
```

# 三十七 前++和后++

前++和后++，见C语言部分笔记

# 三十八 if的汇编

if条件判断

```asm
CMP DWORD PTR DS:[EBP-8], 5
JNE 00413ECF// 不成立就跳走
PUSH 417BCCH// 成立，就调if里面的函数
CALL 0041141F
```

# 三十九 if..else..的汇编

if..else..

```asm
MOV DWORD PTR DS:[EBP-8], 1
CMP DWORD PTR DS:[EBP-8], 0
JNE 00413ED1// 不相等，跳到else里面
// if内部
// if内部执行完，跳到出去的代码

// else内部
// 跳出的代码
```

# 四十 if...else if...else的汇编

多条件if...else if...else

```asm
MOV DWORD PTR DS:[EBP-8], 1
CMP DWORD PTR DS:[EBP-8], 0
JNE 00413ED1// 不相等，跳到else里面
// if内部
// if内部执行完，跳到else if的代码

// else if内部
// 跳到跳出的代码
// else
// 跳到跳出的代码

// 跳出的代码
```

# 四十一 三目运算符的汇编

三目运算符的汇编

![18](./../../操作/3C逆向/18.jpg)

# 四十二 switch...case...的汇编

switch...case...

![19](./../../操作/3C逆向/19.jpg)

# 四十二 跳转表，case是连续的且5个以上

case是连续的且5个以上，就会有跳转表

![20](./../../操作/3C逆向/20.jpg)

![21](./../../操作/3C逆向/21.jpg)

![22](./../../操作/3C逆向/22.jpg)

sub ecx, 2这里减的是最小的case值和0的差值

# 四十四 索引表，跳转表

![23](./../../操作/3C逆向/23.jpg)

![24](./../../操作/3C逆向/24.jpg)

先从索引表拿索引，再到跳转表去执行每个case里面的内容

# 四十五 while的汇编

while

![25](./../../操作/3C逆向/25.jpg)

# 四十六 do...while...的汇编

do...while...

![26](./../../操作/3C逆向/26.jpg)

do...while...效率最高，因为只有一次跳转

# 四十七 for的汇编

for

![27](./../../操作/3C逆向/27.jpg)

# 四十八 break，continue的汇编

break。跳出

![28](./../../操作/3C逆向/28.jpg)

continue。跳到i++

![29](./../../操作/3C逆向/29.jpg)

# 四十九 数组的汇编

数组



在局部定义五个int类型的变量并赋初值

![30](./../../操作/3C逆向/30.jpg)

在局部定义一个数组

![31](./../../操作/3C逆向/31.jpg)

数组名是数组的首地址，也可以说是数组第一个元素的地址

![32](./../../操作/3C逆向/32.jpg)

全局数组，编译之后就有一个固定的地址了

# 五十 数组下标寻址

数组下标寻址



首地址加偏移

![33](./../../操作/3C逆向/33.jpg)

shl eax, 0	;左移0位

# 五十一 数组作为参数传递

数组作为参数传递给一个函数

![34](./../../操作/3C逆向/34.jpg)

![35](./../../操作/3C逆向/35.jpg)

数组传参，其实就传的数组首地址

# 五十二 多维数组的汇编

多维数组



多维数组和一维数组，在内存中都是一维，反汇编中也都是一维

![36](./../../操作/3C逆向/36.jpg)

多维数组算行数和列数

# 五十三 多维数组寻址

多维数组寻址

![37](./../../操作/3C逆向/37.jpg)

mov eax, 0Ch	;0C的意思是，一维还是二维。0C == 3 * sizeof(int)

![38](./../../操作/3C逆向/38.jpg)

数组首地址 + sizeof(int) * 列数 * 行号 + sizeof(int) * 列号

# 五十四 数组越界实验

```c
#include <stdio.h>
#include <stdlib.h>

void Func1()
{
    int i = 0;
    int Arr[3] = { 0 };

    for (i = 0; i <= 5; i++)
    {
        Arr[i] = 0;
        printf("停不下来 \r\n");
    }
}

void Func2()
{
    printf("Hack Success \r\n");
    system("pause");
    exit(0);
}

void Fun3()
{
    int Arr[5] = { 0 };
}

int main()
{
    Func1();

    return 0;
}
```

逆向Func1为什么停不下来

![39](./../../操作/3C逆向/39.jpg)

![40](./../../操作/3C逆向/40.jpg)

![41](./../../操作/3C逆向/41.jpg)

在Func3中只改一行代码调用Func2

![42](./../../操作/3C逆向/42.jpg)

![43](./../../操作/3C逆向/43.jpg)

修改Func3的返回地址为Func2

# 五十五 字符和字符串的汇编

char ch = 'A';

char ch = 65;

没区别，字符在内存中存的就是字符对应的ASCII码值



char arr[] = "A"; 

字符串最后有个00也就是'\0'



char Arr[] = "Hello";

字符数组赋值为字符串，是先在常量区生成那个字符串，再将字符串拷贝到堆栈的字符数组里面

![44](./../../操作/3C逆向/44.jpg)

这里只用两个mov因为只有6个字节

# 五十六 结构体的汇编

结构体初始化跟int类型的数组初始化为0的反汇编是一样的

![45](./../../操作/3C逆向/45.jpg)

内存对齐

![46](./../../操作/3C逆向/46.jpg)

# 五十七 结构体数组的汇编

 

![48](./../../操作/3C逆向/48.jpg)

![47](./../../操作/3C逆向/47.jpg)

mov eax, 8是因为一个结构体的大小是8

# 五十八 结构体作为参数传递

结构体作为参数传递

![49](./../../操作/3C逆向/49.jpg)

结构体作为函数的参数传递，其实就将结构体里的每个成员当成参数push到函数

![50](./../../操作/3C逆向/50.jpg)

如果结构体里面内容太多，编译器会将结构体成员一个一个push的代码优化成REP MOSD，整段拷贝(还是相当于push)

# 五十九 typedef

typedef

![51](./../../操作/3C逆向/51.jpg)

# 六十 union

union

![52](./../../操作/3C逆向/52.jpg)

![53](./../../操作/3C逆向/53.jpg)

# 六十一 enum

enum

![55](./../../操作/3C逆向/55.jpg)

# 六十二 指针变量的汇编

指针变量

![56](./../../操作/3C逆向/56.jpg)

# 六十三 多级指针的汇编

多级指针

![57](./../../操作/3C逆向/57.jpg)

# 六十四 指针的算数运算，步长

指针的算数运算

![58](./../../操作/3C逆向/58.jpg)

# 六十五 指针用[]来访问

指针和数组



指针也可以用[]来访问，跟数组[]访问一样，多了一个取指针内容的步骤

![59](./../../操作/3C逆向/59.jpg)

![60](./../../操作/3C逆向/60.jpg)

# 六十六 值传递和指针传递

值传递和指针传递

![61](./../../操作/3C逆向/61.jpg)

只是改了参数

![62](./../../操作/3C逆向/62.jpg)

# 六十七 结构体指针的汇编

结构体指针

![63](./../../操作/3C逆向/63.jpg)

也是结构体首地址加偏移

# 六十八 指针数组

指针数组

![64](./../../操作/3C逆向/64.jpg)

# 六十九 指针函数

指针函数，返回值为指针的函数

不能返回函数内部定义的局部变量的地址

# 七十 函数指针

函数指针

![65](./../../操作/3C逆向/65.jpg)

![66](./../../操作/3C逆向/66.jpg)

函数就是一个地址，函数指针就是一个指针变量，只不过存的是一个函数的地址

# 七十一 堆内存的分配和释放

全局变量和局部变量的释放不用我们自己释放



malloc分配的堆内存。我们自己分配和释放

![67](./../../操作/3C逆向/67.jpg)

![68](./../../操作/3C逆向/68.jpg)

不释放可能造成堆内存不够用，当前进程占用过大内存，其他进程无内存可用

# 七十二 堆内存管理的函数

分配的堆内存初始化

![69](./../../操作/3C逆向/69.jpg)

只能用一个字节来初始化



内存拷贝

![70](./../../操作/3C逆向/70.jpg)
