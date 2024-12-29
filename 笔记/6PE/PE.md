# 零 .exe、.dll、.sys

PE - Protable Executable File Format - 可执行文件格式

![1](./../../操作/6PE/1.jpg)

.exe	.dll	.sys(内核模块)

![2](./../../操作/6PE/2.jpg)

分别是sys,dll,exe

起始地址是0，起始两字节是4D 5A(MZ)

找偏移为3C的位置，内容分别为D0,E8,60，加上起始地址，还是D0,E8,60

对应的内容是00 00 45 50，00 00 45 50， 00 00 45 50(PE标记)

# 一 PE标记

00地址和3c位置的值，定位到PE Signature，是PE标记

![3](./../../操作/6PE/3.jpg)

# 二 IMAGE_DOS_HEADER

IMAGE_DOS_HEADER

![4](./../../操作/6PE/4.jpg)

![5](./../../操作/6PE/5.jpg)

![6](./../../操作/6PE/6.jpg)

![7](./../../操作/6PE/7.jpg)

# 三 打开、读取、写入文件

![8](./../../操作/6PE/8.jpg)

![9](./../../操作/6PE/9.jpg)

# 四 文件操作

![10](./../../操作/6PE/10.jpg)

### Tools.h

```h
#pragma once
#define _CRT_SECURE_NO_WARNINGS
#include <stdio.h>
#include <Windows.h>

char* FileToMem(char* szFilePath, int* nFileSize);

void MemToFile(char* szFilePath, char* pFileBuffer, int FileSize);
```

### Tools.c

```c
#include "Tools.h"

char* FileToMem(char* szFilePath, int* nFileSize)
{
    // 打开文件
    FILE* pFile = fopen(szFilePath, "rb");
    if (pFile == NULL)
    {
        printf("FileToMem fopen failed \r\n");
        return NULL;
    }

    // 获取文件长度
    fseek(pFile, 0, SEEK_END);
    int nSize = ftell(pFile);
    fseek(pFile, 0, SEEK_SET);

    // 申请内存，用于存文件内容
    char* pFileBuffer = (char*)malloc(nSize);
    if (pFileBuffer == NULL)
    {
        printf("FileToMem malloc failed \r\n");
        free(pFile);
        return NULL;
    }

    // 清理申请的内存
    memset(pFileBuffer, 0, nSize);

    // 读取数据
    fread(pFileBuffer, nSize, 1, pFile);

    fclose(pFile);

    *nFileSize = nSize;

    return pFileBuffer;
}

void MemToFile(char* szFilePath, char* pFileBuffer, int nFileSize)
{
    FILE* pFile = fopen(szFilePath, "wb");
    if (!pFile)
    {
        printf("MemToFile fopen failed \r\n");
        return;
    }

    fwrite(pFileBuffer, nFileSize, 1, pFile);

    fclose(pFile);
}
```

### Main.c

```c
#define _CRT_SECURE_NO_WARNINGS
#include <stdio.h>
#include <Windows.h>

#include "Tools.h"

#define FILE_PATH_IN    ".\\PETool 1.0.0.5.exe"
#define FILE_PATH_OUT   ".\\New_PETool 1.0.0.5.exe"

int main()
{
    int nFileSize = 0;
    char* pFileBuffer = FileToMem(FILE_PATH_IN, &nFileSize);

    // MemToFile(FILE_PATH_OUT, pFileBuffer, nFileSize);

    //PIMAGE_DOS_HEADER pDos = (PIMAGE_DOS_HEADER)pFileBuffer;
    //printf("%04x \r\n", pDos->e_magic);
    //printf("%08x \r\n", pDos->e_lfanew);

    // IMAGE_DOS_HEADER只有这两个地方有用，其他地方随便改，不会影响运行
    memset(pFileBuffer + 2, 0, 64 - 2 - 4);

    MemToFile(FILE_PATH_OUT, pFileBuffer, nFileSize);
}
```

# 五 指针回顾

指针回顾

![11](./../../操作/6PE/11.jpg)

![12](./../../操作/6PE/12.jpg)

![13](./../../操作/6PE/13.jpg)

![14](./../../操作/6PE/14.jpg)

![15](./../../操作/6PE/15.jpg)

![16](./../../操作/6PE/16.jpg)

隐式转换和显示转换的区别

# 六 IMAGE_NT_HEADER

![17](./../../操作/6PE/17.jpg)

```c
char* FileToMem(char* szFilePath, int* nFileSize)
{
    // 打开文件
    FILE* pFile = fopen(szFilePath, "rb");
    if (pFile == NULL)
    {
        printf("FileToMem fopen failed \r\n");
        return NULL;
    }

    // 获取文件长度
    fseek(pFile, 0, SEEK_END);
    int nSize = ftell(pFile);
    fseek(pFile, 0, SEEK_SET);

    // 申请内存，用于存文件内容
    char* pFileBuffer = (char*)malloc(nSize);
    if (pFileBuffer == NULL)
    {
        printf("FileToMem malloc failed \r\n");
        free(pFile);
        return NULL;
    }

    // 清理申请的内存
    memset(pFileBuffer, 0, nSize);

    // 读取数据
    fread(pFileBuffer, nSize, 1, pFile);

    // pFileBuffer是char*类型的，不转为short*就只能拿到一个字节
    // short Flag = *pFileBuffer;

    // 判断MZ标记
    short Flag = *(short*)pFileBuffer;
    if (Flag != IMAGE_DOS_SIGNATURE)// 0x5A4D
    {
        printf("MZ failed \r\n");
        fclose(pFile);
        free(pFileBuffer);
        pFileBuffer = NULL;
        return NULL;
    }

    // 判断PE标记
    DWORD dwOffset = *(PDWORD)(pFileBuffer + 0x3C);// PDWORD就是DWORD*
    PDWORD PeSignature = (PDWORD)(pFileBuffer + dwOffset);
    if (*PeSignature != IMAGE_NT_SIGNATURE)
    {
        printf("PE failed \r\n");
        fclose(pFile);
        free(pFileBuffer);
        pFileBuffer = NULL;
        return NULL;
    }

    fclose(pFile);

    *nFileSize = nSize;

    return pFileBuffer;
}
```

```c
    int nFileSize = 0;
    char* pFileBuffer = FileToMem(FILE_PATH_IN, &nFileSize);

    PIMAGE_DOS_HEADER pDos = (PIMAGE_DOS_HEADER)pFileBuffer;
    
    DWORD dwOffset = pDos->e_lfanew;

    PIMAGE_NT_HEADERS pNts = (PIMAGE_NT_HEADERS)(pFileBuffer + dwOffset);

    printf("%08x\r\n", pNts->Signature);

    MemToFile(FILE_PATH_OUT, pFileBuffer, nFileSize);
```

# 七 标准PE头

IMAGE_FILE_HEADER - 标准PE头



重点

节区个数

OPTION头的大小



占用20个字节

![18](./../../操作/6PE/18.jpg)

![19](./../../操作/6PE/19.jpg)

TimeDateStamp，时间戳

![20](./../../操作/6PE/20.jpg)

![21](./../../操作/6PE/21.jpg)

![22](./../../操作/6PE/22.jpg)

![23](./../../操作/6PE/23.jpg)

# 八 扩展PE头

IMAGE_OPTIONAL_HEADER - 扩展PE头



重点

Magic

EntryPoint

ImageBase

内存对齐

文件对齐

SizeOfImage

SizeOfHeader

![24](./../../操作/6PE/24.jpg)

![25](./../../操作/6PE/25.jpg)

![26](./../../操作/6PE/26.jpg)

![27](./../../操作/6PE/27.jpg)

操作系统给每个进程分配4g的虚拟内存，后2g属于系统，多个进程共用，前2g的前64kb和64kb不能用，中间可以用的内存从ImageBase指定的地方开始，将文件贴到内存中，贴的大小由SizeOfImage指定



基地址(ImageBase)

虚拟内存地址(VA - Virtual Address) == 基地址(ImageBase) + (相对虚拟地址 - Relative Virtual Address)RVA



文件偏移地址(FOA - File Offset Address)：IMAGE_DOS_HEADER + 3c位置的值



AddressOfEntryPoint，程序入口点

![28](./../../操作/6PE/28.jpg)

ImageBse+EntryPoint才是内存中的真正入口地址

# 九 节表

可选PT头后面就是节



Name

内存中的大小

RVA

文件中的大小

FOA

![29](./../../操作/6PE/29.jpg)

![30](./../../操作/6PE/30.jpg)

![31](./../../操作/6PE/31.jpg)

最后一个节之后的40字节必须是全0



```c
VOID PrintfSectionHeader()
{
    // Read File Data
    DWORD dwFileSize = 0;
    PCHAR pFileBuffer = FileToMem(FILE_PATH_IN, &dwFileSize);
    if (!pFileBuffer)
    {
        return;
    }

    PIMAGE_DOS_HEADER       pDos = (PIMAGE_DOS_HEADER)pFileBuffer;
    PIMAGE_NT_HEADERS       pNts = (PIMAGE_NT_HEADERS)(pFileBuffer + pDos->e_lfanew);
    PIMAGE_FILE_HEADER      pFil = (PIMAGE_FILE_HEADER) & (pNts->FileHeader);
    PIMAGE_OPTIONAL_HEADER  pOp = (PIMAGE_OPTIONAL_HEADER) & (pNts->OptionalHeader);

    PIMAGE_SECTION_HEADER   pSec = (PIMAGE_SECTION_HEADER)((PCHAR)pOp + 
        pFil->SizeOfOptionalHeader);

    CHAR szName[9] = { 0 };
    for (size_t i = 0; i < pFil->NumberOfSections; i++)
    {
        memcpy(szName, pSec->Name, sizeof(pSec[i].Name));
        printf("[%d] -> %s \r\n", i, szName);
        printf("[%d] -> %x \r\n", i, pSec[i].Misc.VirtualSize);
        printf("[%d] -> %x \r\n", i, pSec[i].VirtualAddress);
        printf("[%d] -> %x \r\n", i, pSec[i].SizeOfRawData);
        printf("[%d] -> %x \r\n", i, pSec[i].PointerToRawData);
        printf("[%d] -> %x \r\n", i, pSec[i].Characteristics);
    } 
}
```

![32](./../../操作/6PE/32.jpg)

这个截图有点问题

![33](./../../操作/6PE/33.jpg)

# 十-十一 SizeOfHeaders和SizeOfImage

IMAGE_OPTIONAL_HEADDER中的SizeOfHeaders是整个这个图上的所有内容(DOS_HEADER, NT_HEADER, SECTION_HEADER)对齐之后的大小

SizeOfHeaders只是PE头+节表的大小。SizeOfImage是整个PE文件在内存中的大小。

![34](./../../操作/6PE/34.jpg)

![35](./../../操作/6PE/35.jpg)

# 十二 整个PE文件图

![36](./../../操作/6PE/36.jpg)

![38](./../../操作/6PE/38.jpg)

![39](./../../操作/6PE/39.jpg)

# 十三-十四 FileBuffer和ImageBuffer互转

```c
PCHAR FileBufferToImageBuffer(PCHAR pFileBuffer)
{
    // PE结构
    PIMAGE_DOS_HEADER pDos = (PIMAGE_DOS_HEADER)pFileBuffer;
    PIMAGE_NT_HEADERS pNts = (PIMAGE_NT_HEADERS)(pFileBuffer + pDos->e_lfanew);
    PIMAGE_FILE_HEADER pFil = (PIMAGE_FILE_HEADER) & (pNts->FileHeader);
    PIMAGE_OPTIONAL_HEADER pOpo = (PIMAGE_OPTIONAL_HEADER) & (pNts->OptionalHeader);
    PIMAGE_SECTION_HEADER pSec = (PIMAGE_SECTION_HEADER)((PCHAR)pOpo + 
        pFil->SizeOfOptionalHeader);

    // 要分配的内存大小
    DWORD dwImageSie = pOpo->SizeOfImage;// SizeOfImage是整个PE文件在内存中的大小
    PCHAR pImageBuffer = (PCHAR)malloc(dwImageSie);
    if (!pImageBuffer)
    {
        return NULL;
    }
    memset(pImageBuffer, 0, dwImageSie);

    // 拷贝PE头+节表的大小
    // SizeOfHeaders是PE头+节表的大小(包括了节表到第一个节中间的对齐的区域).文件和内存中都是一样的
    memcpy(pImageBuffer, pFileBuffer, pOpo->SizeOfHeaders);

    // 拷贝节区数据
    for (size_t i = 0; i < pFil->NumberOfSections; i++)
    {
        memcpy(pImageBuffer + pSec[i].VirtualAddress,   // 节内容的RVA
            pFileBuffer + pSec[i].PointerToRawData,		// 节内容的FOA
            pSec->SizeOfRawData);						// 节在文件中的大小
    }

    return pImageBuffer;
}

PCHAR ImageBufferToFileBuffer(PCHAR pImageBuffer, int* nFileSize)
{
    // PE结构
    PIMAGE_DOS_HEADER pDos = (PIMAGE_DOS_HEADER)pImageBuffer;
    PIMAGE_NT_HEADERS pNts = (PIMAGE_NT_HEADERS)(pImageBuffer + pDos->e_lfanew);
    PIMAGE_FILE_HEADER pFil = (PIMAGE_FILE_HEADER) & (pNts->FileHeader);
    PIMAGE_OPTIONAL_HEADER pOpo = (PIMAGE_OPTIONAL_HEADER) & (pNts->OptionalHeader);
    PIMAGE_SECTION_HEADER pSec = (PIMAGE_SECTION_HEADER)((PCHAR)pOpo +
        pFil->SizeOfOptionalHeader);

    // 计算FileBuffer大小
    // SizeOfHeaders是PE头+节表的大小(包括了节表到第一个节中间的对齐的区域).文件和内存中都是一样的
    DWORD dwFileSize = pOpo->SizeOfHeaders;		// PE头+节表的大小
    for (size_t i = 0; i < pFil->NumberOfSections; i++)
    {
        dwFileSize += pSec[i].SizeOfRawData;	// 节内容在文件中的大小
    }

    // 分配内存
    PCHAR pFileBuffer = (PCHAR)malloc(dwFileSize);
    if (!pFileBuffer)
    {
        printf("malloc failed \r\n");
        return NULL;
    }
    memset(pFileBuffer, 0, dwFileSize);

    // 拷贝头+节表大小
    memcpy(pFileBuffer, pImageBuffer, pOpo->SizeOfHeaders);

    // 拷贝节区数据
    for (size_t i = 0; i < pFil->NumberOfSections; i++)
    {
        memcpy(pFileBuffer + pSec[i].PointerToRawData,	// FOA
            pImageBuffer + pSec[i].VirtualAddress,		// RVA
            pSec[i].SizeOfRawData);						// 节在文件中的大小
    }

    if (nFileSize != NULL)
    {
        *nFileSize = dwFileSize;
    }

    return pFileBuffer;
}
```

# 十五 RVA转FOA

RVA转FOA

![40](./../../操作/6PE/40.jpg)

VirtualAddress先减去ImageBase，得到RVA

再找RVA是哪个节区里面的，找这个节区对应的FOA

就可以在文件中修改FOA的值来修改全局变量的值

# 十六 RVA和FOA互转

```c
DWORD RvaToFoa(PCHAR pBuffer, DWORD dwRva)
{
    // PE结构
    PIMAGE_DOS_HEADER pDos = (PIMAGE_DOS_HEADER)pBuffer;
    PIMAGE_NT_HEADERS pNts = (PIMAGE_NT_HEADERS)(pBuffer + pDos->e_lfanew);
    PIMAGE_FILE_HEADER pFil = (PIMAGE_FILE_HEADER) & (pNts->FileHeader);
    PIMAGE_OPTIONAL_HEADER pOpo = (PIMAGE_OPTIONAL_HEADER) & (pNts->OptionalHeader);
    PIMAGE_SECTION_HEADER pSec = (PIMAGE_SECTION_HEADER)((PCHAR)pOpo +
        pFil->SizeOfOptionalHeader);

    // 判断是否在文件头内部
    // ImageBase + dwRva < ImageBase + pOpo->SizeOfHeaders
    if (dwRva < pOpo->SizeOfHeaders)
    {
        return dwRva;
    }

    // 遍历节区查找匹配的RVA
    for (size_t i = 0; i < pFil->NumberOfSections; i++)
    {
        if ((dwRva >= pSec[i].VirtualAddress) && (dwRva < pSec[i].VirtualAddress + pSec[i].Misc.VirtualSize))
        {
            return dwRva - pSec[i].VirtualAddress + pSec[i].PointerToRawData;
        }
    }

    return 0;
}

DWORD FoaToRva(PCHAR pBuffer, DWORD dwFoa)
{
    // PE结构
    PIMAGE_DOS_HEADER pDos = (PIMAGE_DOS_HEADER)pBuffer;
    PIMAGE_NT_HEADERS pNts = (PIMAGE_NT_HEADERS)(pBuffer + pDos->e_lfanew);
    PIMAGE_FILE_HEADER pFil = (PIMAGE_FILE_HEADER) & (pNts->FileHeader);
    PIMAGE_OPTIONAL_HEADER pOpo = (PIMAGE_OPTIONAL_HEADER) & (pNts->OptionalHeader);
    PIMAGE_SECTION_HEADER pSec = (PIMAGE_SECTION_HEADER)((PCHAR)pOpo +
        pFil->SizeOfOptionalHeader);

    // 判断是否在文件头内部
    if (dwFoa < pOpo->SizeOfHeaders)
    {
        return dwFoa;
    }

    // 遍历节区查找匹配的RVA
    for (size_t i = 0; i < pFil->NumberOfSections; i++)
    {
        if ((dwFoa >= pSec[i].PointerToRawData) && (dwFoa < pSec[i].PointerToRawData + pSec[i].SizeOfRawData))
        {
            return dwFoa - pSec[i].PointerToRawData + pSec[i].VirtualAddress;
        }
    }

    return 0;
}
```

# 十七 IMAGE_DATA_DIRECTORY

IMAGE_OPTION_HEADER最后一个成员，IMAGE_DATA_DIRECTORY

![41](./../../操作/6PE/41.jpg)

![42](./../../操作/6PE/42.jpg)

![43](./../../操作/6PE/43.jpg)

Option的最后一个成员是一个(RVA, Size)结构体的数组

# 十八 静态库

![44](./../../操作/6PE/44.jpg)

f12一下strlen，发现只有函数声明，没有函数实现

静态库

![45](./../../操作/6PE/45.jpg)

创建一个空项目，配置类型改为静态库

![46](./../../操作/6PE/46.jpg)

lib项目中，写个.h和.c，编译成lib

.h中写Add函数的声明，.c中写Add函数的实现

将.h和.lib复制到Demo项目中

![47](./../../操作/6PE/47.jpg)

![48](./../../操作/6PE/48.jpg)

把lib里的内容编译到了**当前进程的内存**中了

# 十九 dll

创建一个空项目，配置类型改成dll

![49](./../../操作/6PE/49.jpg)

![50](./../../操作/6PE/50.jpg)

![51](./../../操作/6PE/51.jpg)

编译完，将dll拖入DetectItEasy，可以查看导出函数

![52](./../../操作/6PE/52.jpg)

如果把函数改成_stdcall

![53](./../../操作/6PE/53.jpg)

将DIV函数再加一个参数c

![54](./../../操作/6PE/54.jpg)

如果是_stdcall，可以看到参数的大小。

如果再把.c文件改成.cpp

![55](./../../操作/6PE/55.jpg)

在cpp中用extern “C”，以C风格名称粉碎

![56](./../../操作/6PE/56.jpg)

# 二十 dll的反汇编

不需要.h头文件了

![57](./../../操作/6PE/57.jpg)

![58](./../../操作/6PE/58.jpg)

![59](./../../操作/6PE/59.jpg)

这个函数地址已经不在当前进程内了



用LoadLibraryA。就是把PE文件(dll)加载到内存

![60](./../../操作/6PE/60.jpg)

![61](./../../操作/6PE/61.jpg)

# 二十一 def导出

def导出

![62](./../../操作/6PE/62.jpg)

可以指定导出的时候的序号，也可以指定导出的时候不带名字

![63](./../../操作/6PE/63.jpg)

# 二十二 导出表

IMAGE_EXPORT_DIRECTORY

OPTION_HEADER的最后一个结构体数组成员

![64](./../../操作/6PE/64.jpg)

第一个成员是导出表，这个导出表是个RVA，要转成FOA，再去文件中找

![64](./../../操作/6PE/64.png)

Name

Base

NumberOfFunctions

NumberOfNames

所有函数地址表

有名字的函数的函数名字表

有名字的函数的序号表



NumberOfFunctions是不准确的，要用最大的导出函数序号 - 最小的导出函数序号 + 1

![65](./../../操作/6PE/65.jpg)

# 二十三 导出表结构

![66](./../../操作/6PE/66.jpg)

AddressOfFunctions内容的个数由NumberOfFunctions决定

AddressOfNames内容的个数由NumberOfNames决定



AddressOfNameOrdinals内容的个数由NumberOfNames决定，每个元素都是word。

只记录有函数名的导出函数的序号

表中的值是没有加BASE的，还需要加Base的值才是真正导出函数的序号。所以这个图有问题



用这么多RVA是因为多个dll的时候，ImageBase都是10000000，这个地址被一个dll占了之后，系统会给其他分配别的ImageBase

# 二十四 打印导出表

```C
void PrintfExport()
{
    // 读入dll
    PCHAR pFileBuff = FileToMem(".\\TestDll.dll", NULL);
    if (!pFileBuff)
    {
        return;
    }

    // 定位结构
    PIMAGE_DOS_HEADER pDos = (PIMAGE_DOS_HEADER)pFileBuff;
    PIMAGE_NT_HEADERS pNth = (PIMAGE_NT_HEADERS)(pFileBuff + pDos->e_lfanew);

    // 是否有导出表
    if (!pNth->OptionalHeader.DataDirectory[0].VirtualAddress)
    {
        return;
    }

    // RvaToFoa
    DWORD dwExportFoa = RvaToFoa(pFileBuff, 
        pNth->OptionalHeader.DataDirectory[0].VirtualAddress);

    // 定位到导出表
    PIMAGE_EXPORT_DIRECTORY pExp = (PIMAGE_EXPORT_DIRECTORY)(pFileBuff + dwExportFoa);

    // 输出基本内容
    printf("Name [%s] \r\n", pFileBuff + RvaToFoa(pFileBuff, pExp->Name));
    printf("Base [0x%08x] \r\n", pFileBuff + RvaToFoa(pFileBuff, pExp->Base));
    printf("NumberOfFunctions [0x%08x] \r\n", pFileBuff + RvaToFoa(pFileBuff, pExp->NumberOfFunctions));
    printf("NumberOfNames [0x%08x] \r\n", pFileBuff + RvaToFoa(pFileBuff, pExp->NumberOfNames));
    printf("AddressOfFunctions [0x%08x] \r\n", pFileBuff + RvaToFoa(pFileBuff, pExp->AddressOfFunctions));
    printf("AddressOfNames [0x%08x] \r\n", pFileBuff + RvaToFoa(pFileBuff, pExp->AddressOfNames));
    printf("AddressOfNameOrdinals [0x%08x] \r\n", pFileBuff + RvaToFoa(pFileBuff, pExp->AddressOfNameOrdinals));

    // 导出函数名称表
    LPDWORD lpFunName = (LPDWORD)(pFileBuff +
        RvaToFoa(pFileBuff, pExp->AddressOfNames));
    for (size_t i = 0; i < pExp->NumberOfNames; i++)
    {
        printf("%s \r\n", pFileBuff + RvaToFoa(pFileBuff, lpFunName[i]));
    }

    // 导出函数地址表
    LPDWORD lpFuncAddr = (LPDWORD)(pFileBuff +
        RvaToFoa(pFileBuff, pExp->AddressOfFunctions));
    for (size_t i = 0; i < pExp->NumberOfNames; i++)
    {
        printf("%08x \r\n", pFileBuff + RvaToFoa(pFileBuff, lpFuncAddr[i]));
    }

    // 导出函数序号表
    LPWORD lpFunOrder = (LPWORD)(pFileBuff +
        RvaToFoa(pFileBuff, pExp->AddressOfNameOrdinals));
    for (size_t i = 0; i < pExp->NumberOfNames; i++)
    {
        // 要+Base才是真正的导出函数的序号
        printf("%d \r\n", lpFunOrder[i] + pExp->Base);
    }
}
```

![67](./../../操作/6PE/67.jpg)

# 二十五 函数名找函数地址

有一个函数名，如何在dll里找到这个函数的地址

先从Name表找匹配的name，再用这个序号去ordi表里找对应索引，再用这个索引去函数地址表里找地址

```c
PCHAR GetProcAddressByName(PCHAR pFileBuff, PCHAR pFuncName)
{
    // 定位结构
    PIMAGE_DOS_HEADER pDos = (PIMAGE_DOS_HEADER)pFileBuff;
    PIMAGE_NT_HEADERS pNth = (PIMAGE_NT_HEADERS)(pFileBuff + pDos->e_lfanew);

    // 是否有导出表
    if (!pNth->OptionalHeader.DataDirectory[0].VirtualAddress)
    {
        return;
    }

    // RvaToFoa
    DWORD dwExportFoa = RvaToFoa(pFileBuff,
        pNth->OptionalHeader.DataDirectory[0].VirtualAddress);

    // 定位导出表
    PIMAGE_EXPORT_DIRECTORY pExp = (PIMAGE_EXPORT_DIRECTORY)(pFileBuff + dwExportFoa);

    // 定位三张表
    LPDWORD pFuncAddr = (LPDWORD)(pFileBuff + RvaToFoa(pFileBuff, pExp->AddressOfFunctions));
    LPDWORD pFuncName1 = (LPDWORD)(pFileBuff + RvaToFoa(pFileBuff, pExp->AddressOfNames));
    LPWORD pFuncOrdi = (LPWORD)(pFileBuff + RvaToFoa(pFileBuff, pExp->AddressOfNameOrdinals));

    for (size_t i = 0; i < pExp->NumberOfNames; i++)
    {
        if (strcmp(pFuncName, pFileBuff + RvaToFoa(pFileBuff, pFuncName1[i])) == 0)
        {
            // 不用pFuncAddr[i]是因为可能有没有名字的导出函数
            return pFileBuff + RvaToFoa(pFileBuff, FuncAddr[pFuncOrdi[i]]);
        }
    }

    return NULL;
}
```

# 二十六 函数序号找函数地址

通过导出函数的序号来查找函数，不用导出名称表

```c
PCHAR GetProcAddressByOrdi(PCHAR pFileBuff, DWORD dwOrdi)
{
    // 定位结构
    PIMAGE_DOS_HEADER pDos = (PIMAGE_DOS_HEADER)pFileBuff;
    PIMAGE_NT_HEADERS pNth = (PIMAGE_NT_HEADERS)(pFileBuff + pDos->e_lfanew);

    // 是否有导出表
    if (!pNth->OptionalHeader.DataDirectory[0].VirtualAddress)
    {
        return;
    }

    // RvaToFoa
    DWORD dwExportFoa = RvaToFoa(pFileBuff,
        pNth->OptionalHeader.DataDirectory[0].VirtualAddress);

    // 定位导出表
    PIMAGE_EXPORT_DIRECTORY pExp = (PIMAGE_EXPORT_DIRECTORY)(pFileBuff + dwExportFoa);

    // 定位三张表
    LPDWORD pFuncAddr = (LPDWORD)(pFileBuff + RvaToFoa(pFileBuff, pExp->AddressOfFunctions));

    return pFileBuff + RvaToFoa(pFileBuff, pFuncAddr[dwOrdi - pExp->Base]);
}
```

# 二十七 导入表

导入表，OPTION_HEADER的最后一个参数是个结构体(RVA, Size)数组， 数组的第二个元素就是导入表

![68](./../../操作/6PE/68.jpg)

MFC42.dll是NONAME导出

MSVCRT.dll就是正常导出。



INT

NAME

IAT



RVA转FOA找到这个表，这个表是成员为20字节的数组，每个20字节就是个.dll

![69](./../../操作/6PE/69.jpg)

![70](./../../操作/6PE/70.jpg)

OriginalFirstThunk和FirstThunk都指向IMAGE_THUNK_DATA32

IMAGE_THUNK_DATA32就是一个元素为4字节的数组



数组每个元素是个导入函数

IMAGE_THUNK_DATA32的大小需要找到最后00 00 00 00的地方，就结束了



如果IMAGE_THUNK_DATA32最高位是1，低32位就是导出函数序号

否则，指向IMAGE_IMPORT_BY_NAME结构。序号和函数名的结构体

# 二十八 导入表结构

RVA偏移到导入表的位置。因为这个文件内存对齐和文件对齐是一样的

![71](./../../操作/6PE/71.jpg)

找最后的20个字节的0，就是导入表结束的位置，就知道了导入表的大小



随便找了个导入表中的一个20字节(一个dll)，看INT表和IAT表(这个dll里的导出函数)

![72](./../../操作/6PE/72.jpg)

INT表和IAT表，导出序号。最高位为1，就是NONAME导出的情况

![73](./../../操作/6PE/73.jpg)

这里是找到00 00 00 00结束，4字节代表一个函数，一个dll导入了这么多函数

![74](./../../操作/6PE/74.jpg)

导入表中随意的一个20字节，表示一个dll。这里最高位是1，后面31位就是这个dll的导出函数的序号



INT表和IAT表，导出序号。最高位为0

![75](./../../操作/6PE/75.jpg)

这是函数名，函数名前面008F是函数序号(WORD)

![76](./../../操作/6PE/76.jpg)

# 二十九 导入表加载到进程前后的变化

程序运行时，INT不变，IAT改变

因为这个dll被加载后，属于这个dll的imagebase变了

![77](./../../操作/6PE/77.jpg)

![78](./../../操作/6PE/78.jpg)

# 三十 打印导入表

打印导入表

```c
void PrintImport()
{
    // 读入exe到内存
    PCHAR pFileBuff = FileToMem(FILE_PATH_IN, NULL);
    if (!pFileBuff)
    {
        return;
    }

    // 定位结构
    PIMAGE_DOS_HEADER pDos = (PIMAGE_DOS_HEADER)pFileBuff;
    PIMAGE_NT_HEADERS pNth = (PIMAGE_NT_HEADERS)(pFileBuff + pDos->e_lfanew);

    // 导入表是否存在
    if (!pNth->OptionalHeader.DataDirectory[1].VirtualAddress)
    {
        return;
    }

    // RVA转FOA
    DWORD dwImportFoa = RvaToFoa(pFileBuff, 
        pNth->OptionalHeader.DataDirectory[1].VirtualAddress);
    // 定位到导入表
    PIMAGE_IMPORT_DESCRIPTOR pImp = (PIMAGE_IMPORT_DESCRIPTOR)(pFileBuff + dwImportFoa);

    // 导入表中这两个表都要存在
    while (pImp->OriginalFirstThunk && pImp->FirstThunk)
    {
        printf("-----------------------------------------\r\n");

        // dll名字
        printf("DLLNAME:[%s] \r\n", pFileBuff + RvaToFoa(pFileBuff, pImp->Name));

        // 遍历INT
        PIMAGE_THUNK_DATA INT = (PIMAGE_THUNK_DATA)(pFileBuff +
            RvaToFoa(pFileBuff, pImp->OriginalFirstThunk));
        do 
        {
            // 判断最高位是否位1
            if (*(PDWORD)INT & 0x80000000)
            {
                // 取低31位
                printf("Ordinal -> [%x] \r\n", *(PDWORD)INT & 0x7FFFFFFF);
            }
            else
            {
                // 也是RVA。最终指向PIMAGE_IMPORT_BY_NAME结构体
                PIMAGE_IMPORT_BY_NAME pName = (PIMAGE_IMPORT_BY_NAME)(pFileBuff +
                    RvaToFoa(pFileBuff, *(PDWORD)INT));

                printf("HINT -> [%x] NAME -> [%s] \r\n", pName->Hint, pName->Name);
            }

        } while (*(PDWORD)(++INT));// 最后才++的


        // 遍历IAT
        PIMAGE_THUNK_DATA IAT = (PIMAGE_THUNK_DATA)(pFileBuff +
            RvaToFoa(pFileBuff, pImp->FirstThunk));
        do
        {
            // 判断最高位是否位1
            if (*(PDWORD)IAT & 0x80000000)
            {
                // 取低31位
                printf("Ordinal -> [%x] \r\n", *(PDWORD)IAT & 0x7FFFFFFF);
            }
            else
            {
                PIMAGE_IMPORT_BY_NAME pName = (PIMAGE_IMPORT_BY_NAME)(pFileBuff +
                    RvaToFoa(pFileBuff, *(PDWORD)IAT));

                printf("HINT -> [%x] NAME -> [%s] \r\n", pName->Hint, pName->Name);
            }

        } while (*(PDWORD)(++IAT));// 最后才++的

        printf("-----------------------------------------\r\n");

        // 下一个导入的dll
        pImp++;
    }
}
```

# 三十一 重定位表

多个dll不可能同时占用0x10000000这个地址

需要修复

![79](./../../操作/6PE/79.jpg)

重定位表

当前模块没有加载到指定的IMAGEBASE，操作系统分配了一个新的IMAGEBASE，涉及到的一些全局变量，需要修复

# 三十二 重定位表结构

![80](./../../操作/6PE/80.jpg)

RVA

重定位结构的大小



OPTION_HEADER最后一个成员的第六个元素。RVA转FOA定位到这里

![81](./../../操作/6PE/81.jpg)

005C第一个重定位结构的大小

![82](./../../操作/6PE/82.jpg)

00A8是第二个重定位结构的大小

最后找到00 00 00 00    00 00 00 00，就结束了

![83](./../../操作/6PE/83.jpg)

这里每两个字节，加上最开始4字节的VirtualAddress就是真正需要重定向的RVA

# 三十三 重定位表中数据的修复

![84](./../../操作/6PE/84.jpg)

每2字节的高4位是0011(3)，低12位+VirtualAddress才是需要重定位的RVA

![85](./../../操作/6PE/85.jpg)

修复过程

# 三十四 打印重定位表

```c
void PrintfRelocation()
{
    // 读入exe到内存
    PCHAR pFileBuff = FileToMem(".\\TestDll.dll", NULL);
    if (!pFileBuff)
    {
        return;
    }

    // 定位结构
    PIMAGE_DOS_HEADER pDos = (PIMAGE_DOS_HEADER)pFileBuff;
    PIMAGE_NT_HEADERS pNth = (PIMAGE_NT_HEADERS)(pFileBuff + pDos->e_lfanew);

    // 重定位表是否存在
    if (!pNth->OptionalHeader.DataDirectory[5].VirtualAddress)
    {
        return;
    }

    // 重定位表的FOA
    DWORD dwRelocationFoa = RvaToFoa(pFileBuff, 
        pNth->OptionalHeader.DataDirectory[5].VirtualAddress);

    // 定位到重定位表
    PIMAGE_BASE_RELOCATION pRel = (PIMAGE_BASE_RELOCATION)(pFileBuff + dwRelocationFoa);
   
    // 判断前8字节是否为0
    while (*(PLONGLONG)pRel)
    {
        printf("VirtualAddress -> [0x%08x] \r\n", pRel->VirtualAddress);
        printf("SizeOfBlock    -> [0x%08x] \r\n", pRel->SizeOfBlock);

        DWORD dwNumberOfRel= (pRel->SizeOfBlock - 8) / 2;

        PWORD pRelData = (PWORD)((PCHAR)pRel + 8);
        for (size_t i = 0; i < dwNumberOfRel; i++)
        {
            if (pRelData[i] & 0x3000 == 0x3000)
            {
                DWORD dwOffset = pRelData[i] & 0x0FFF;
                DWORD dwRva = dwOffset + pRel->VirtualAddress;
                printf("%x \r\n", pFileBuff + RvaToFoa(pFileBuff, dwRva));
            }
        }

        // 指向下一个重定位结构
        pRel = (PIMAGE_BASE_RELOCATION)((PCHAR)pRel + pRel->SizeOfBlock);
    }
}
```

