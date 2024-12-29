# 一 编译链接

高级语言----编译->汇编----编译->二进制

# 二 进制的本质

进制的本质就是查表

# 三 不同进制之间的加法

不同进制之间的加法，乘法运算的本质也是查表

# 四、五 进制转换

进制转换

# 六 数据宽度

数据宽度，数据大小的限制

运算的结果超过了数据宽度，多余的数据会被舍弃

```c
char ch = 100 + 200;	// 300是0x12C.剩2C，44
printf("%d\r\n", ch);	// 打印出44
```

```txt
bit				0-1					// 计算机中最小的
byte			0-0xFF				// 内存中最小的
word			0-0xFFFF
dword			0-0xFFFFFFFF
```

# 七 有符号数和无符号数

有符号数和无符号数

dword		0-0x7FFFFFFFF				正

​			  0xFFFFFFFF-0x80000000		负

```c
char ch = 0x7F;
printf("%d\r\n", ch);	// 127
ch += 1;
printf("%d\r\n", ch);	// -128
//  0 ~  127
// -1 ~ -128
```

# 八 整数的编码规则

整数的编码规则

一位符号位，其他数值位



原码，反码，补码

# 九 位运算

位运算



与，或，非，异或

# 十 工具介绍

工具介绍

X64DBG

OD

# 十一 通用寄存器

寄存器----CPU里存储数据的地方

通用寄存器

EAX

ECX

EDX

EBX

ESP

EBP

ESI

EDI



```asm
MOV EAX, 0X11223344
MOV EBX, EAX
```

# 十二 16位寄存器

16位寄存器

AX就是EAX的右半边

# 十三 8位寄存器

8位寄存器

AX分成AH和AL

# 十四 x86每个进程都有独立的4g虚拟内存空间

x86每个进程都有独立的4g虚拟内存空间



Windows和Linux都运行在保护模式，在保护模式下，处理器可以同时运行多个进程，每个进程分配4g虚拟内存。每个进程只能访问自己的内存空间，不能跨进程访问内存(代码和数据)



```asm
MOV DWORD PTR DS:[0x19FF84], 0x12345678
```

# 十五 内存寻址

内存寻址

```asm
[立即数]			  				MOV DWORD PTR DS:[0x19FF84], 0x19FF84					立即数寻址
[reg]							  MOV DWORD PTR DS:[EAX], 0x12345678			  		  寄存器寻址
[reg + 立即数]		  				MOV DWORD PTR DS:[EAX+4], 0X12345678					寄存器相对寻址
[reg + reg*{1, 2, 4, 8}]
[reg + reg*{1, 2, 4, 8} + 立即数]	MOV DWORD PTR DS:[EAX + ECX*4 + 4]						比例因子寻址
```

不是所有地址都能读写。因为虽然进程分配了4g内存，但是不是所有地址都被使用，没被使用就没有映射物理内存，当然不能访问

# 十六 大小端

大小端

```asm
MOV DWORD PTR DS:[0x19FF84], 0x11223344
0x19FF84:44
0x19FF85:33
0x19FF86:22
0x19FF87:11
```

# 十七汇编指令有0-3个操作数

汇编指令可能0-3个操作数

# 十八 LEA

```asm
LEA EAX, DOWRD PTR DS:[0x19FF84]
MOV EAX, DOWRD PTR DS:[0x19FF84]
```

# 十九 ADD

```asm
ADD EAX, 2

MOV AL, 0xFF
ADD AL, 1		// AL:0x00
```

# 二十 SUB

```asm
SUB EAX, ECX

MOV AL, 0
MOV CL, 1
SUB AL, CL		// AL:0xFF
```

# 二十一 Flag寄存器

标志寄存器

存储某条指令执行的结果

# 二十二 CF

```asm
CF，在VS中显示为CY
向最高有效位的前一位发生了进位和借位,CF为1
mov al, 1
mov cl, 1
add al, cl		// CF还是0。最高位没有进位和借位
```

# 二十三 PF

```asm
PF，在VS中显示为PE
计算结果中，最低字节中(低8位)，二进制位1的个数是偶数，PF置1，奇数，PF为0
MOV EAX, 0x2
MOV ECX, 0x2
ADD EAX, ECX
0x0100			// 所以PF为0
```

# 二十四 AF

```asm
AF，在VS中显示为AC
字节操作中，发生了低4位向高4位进位或借位，AF为1，反之为0
MOV AL, 0x1F
ADD AL, 1		// AF为1

MOV AL, 11
ADD AL, 11		// AF为0.不是低4位进位到高4位的

字操作中，发生了低8位向高8位进位或借位，AF为1，反之为0

只有这两种情况。
```

# 二十五 ZF

```asm
ZF，在VS中显示为ZR
运算结果为0，AF为1，反之为0
MOV EAX, 1
SUB EAX, 1		// AF为1
```

# 二十六 SF

```asm
SF,在VS中显示为PL
结果的最高位是什么，SF就是什么
MOV AL, 0x7F
ADD AL, 0X1		// SF为1
```

# 二十七 OF

```asm
OF，在VS中显示为OV
正 + 正 = 负	
MOV EAX, 0X7F
ADD EAX, 1

负 + 负 = 正 
MOV EAX, 0XFF
ADD EAX, 2

正 + 负 永远不会溢出
```

# 二十八 EIP、JMP

```asm
EIP
即将要执行的指令的地址

JMP
修改EIP
```

# 二十九 CMP、TEST

```txt
CMP
目标操作数 - 源操作数，不存结果，用标志寄存器判断

TEST
目标操作数 AND 源操作数，不存结果，用标志寄存器判断
```

# 三十 JZ/JE

```txt
JZ/JE		结果为0就跳转(相等跳转)
```

# 三十一 STOSD

```asm
STOSB
STOSW
STOSD

STOSD即：

如果DF是0
MOV DWORD PTR DS:[EDI], EAX
ADD EDI, 4		;只会让EDI+4执行一次，需要配合rep使用

如果DF是1
MOV DWORD PTR DS:[EDI], EAX
SUB EDI, 4
```

# 三十二 MOVSD

```asm
MOV ESI, 0x19FF84
MOV EDI, 0x19ffA4
MOV EAX, DWORD PTR DS:[ESI]
MOV DWORD PTR DS:[EDI], EAX
ADD ESI, 4
ADD EDI, 4
MOV EAX, DWORD PTR DS:[ESI]
MOV DWORD PTR DS:[EDI], EAX

如果DF是0
MOV ESI, 0x19FF84
MOV EDI, 0x19ffA4
MOVSD			;只会让ESI+4和EDI+4执行一次，需要配合rep使用
```

# 三十三 REP

```asm
MOV ESI, 0x19FF84
MOV EDI, 0x19ffA4
MOV ECX, 0x3
REP MOVSD		// 循环执行ECX次
```

# 三十四 EBP、ESP

```asm
堆栈

ESP
EBP
```

# 三十五 PUSH、POP

```asm
每条线程都有一个自己的栈

SUB ESP, 4
MOV DWORD PTR SS:[ESP], 0x12345678

即
PUSH 0x12345678


MOV ECX, DWORD PTR SS:[ESP]
ADD ESP, 4

即
POP ECX
```

# 三十六 CALL

```asm
CALL 0x00402494

call的下一条指令(返回地址)入栈
JMP 0x00402494


RET
(POP EIP)可以这样理解
```
