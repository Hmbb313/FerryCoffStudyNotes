# 零 Hook本质

![1](./../../操作/10Hook/1.jpg)

把这几行指令改了，jmp到另一个函数，在那个函数里，就可以获取ret(谁调用的)，还有参数

![2](./../../操作/10Hook/2.jpg)

0F1B000h 这个地址是 IAT 表中存储 LoadLibraryA 的RVA+IMAGEBASE的值

从自身进程的导入表中查找LoadLibraryA 的RVA，然后算LoadLibraryA 的实际地址



IAT Hook，把导入表中LoadLibraryA 的RVA改成其他函数的RVA，就HOOK成功了

# 一 Inline Hook原理

内联在原始代码里面的hook

![3](./../../操作/10Hook/3.jpg)

![4](./../../操作/10Hook/4.jpg)

![5](./../../操作/10Hook/5.jpg)

```cpp
#include <iostream>
#include <windows.h>

using std::cout;
using std::cin;
using std::endl;

int Add(int a, int b)
{
    return a + b;
}

int main()
{
    LPVOID lpBuffer = VirtualAlloc(NULL, 0x1000, MEM_COMMIT, PAGE_EXECUTE_READWRITE);
    printf("%x \r\n", lpBuffer);

    Add(1, 2);

    return 0;
}
```

就把函数开头几条指令改成jmp或者call，跳到我们申请的内存空间里，保存寄存器信息，我们申请的内存空间里写我们的代码

最后还原他函数原来的指令，返回

# 二 Inline Hook代码

先把增量链接关了，不然有跳转表，就不对了

![6](./../../操作/10Hook/6.jpg)

```cpp
#include <iostream>
#include <windows.h>

using std::cout;
using std::cin;
using std::endl;

CHAR opCode[9] = { 0 };
DWORD dwRetData = 0;
CHAR szBuffer[0x256] = { 0 };

DWORD dwRetAddr = 0;
DWORD dwParam1 = 0;
DWORD dwParam2 = 0;

int Add(int a, int b)
{
    return a + b;
}

_declspec(naked) void HookFunc()
{
    // 保存寄存器环境
    __asm 
    {
        // 8个通用寄存器，一个Flag寄存器。0x24
        PUSHAD
        PUSHFD
    }

    // 实现功能
    __asm
    {
        // 获取返回地址
        MOV EAX, [ESP + 0x24]
        MOV [dwRetAddr], EAX

        // 获取参数1
        MOV EAX, [ESP + 0x28]
        MOV[dwParam1], EAX

        // 获取参数2
        MOV EAX, [ESP + 0x2C]
        MOV[dwParam2], EAX
    }

    printf("%x \r\n", dwRetAddr);
    printf("%x \r\n", dwParam1);
    printf("%x \r\n", dwParam2);

    // 恢复寄存器环境
    __asm 
    {
        POPFD
        POPAD
    }

    // 执行被Hook函数头被覆盖的数据
    __asm
    {
        PUSH EBP
        MOV EBP, ESP
        SUB ESP, 0xC0
    }

    // 跳转到HOOK之前的代码
    __asm
    {
        jmp [dwRetData]
    }
}

VOID SetInlineHook(DWORD dwHookAddr, DWORD dwTargerAddr)
{
    // 修改内存属性
    DWORD dwOldPro;
    DWORD dwOldPro1;
    BOOL bRet = VirtualProtect((LPVOID)dwHookAddr, 0x9, PAGE_EXECUTE_READWRITE, &dwOldPro);
    if (!bRet)
    {
        return;
    }

    // 备份默认数据
    memcpy(opCode, (LPVOID)dwHookAddr, 9);

    // 设置NOP
    memset((LPVOID)dwHookAddr, 0x90, 0x9);

    // JMP的五字节代码
    CHAR szCode[5] = { 0 };
    szCode[0] = 0xE9;

    // JMP后面四字节
    DWORD dwJmpData = dwTargerAddr - dwHookAddr - 5;
    *(PDWORD)&szCode[1] = dwJmpData;

    // 写到函数头
    memcpy((LPVOID)dwHookAddr, szCode, 5);

    // 记录返回地址
    dwRetData = dwHookAddr + 5;

    // 恢复内存属性
    VirtualProtect((LPVOID)dwHookAddr, 0x9, dwOldPro, &dwOldPro1);
}

VOID UnInlineHook(DWORD dwHookAddr)
{
    // 修改内存属性
    DWORD dwOldPro;
    DWORD dwOldPro1;
    BOOL bRet = VirtualProtect((LPVOID)dwHookAddr, 0x9, PAGE_EXECUTE_READWRITE, &dwOldPro);
    if (!bRet)
    {
        return;
    }

    // 将函数头原来的代码恢复
    memcpy((LPVOID)dwHookAddr, opCode, 0x9);

    // 恢复内存属性
    VirtualProtect((LPVOID)dwHookAddr, 0x9, dwOldPro, &dwOldPro1);
}

int main()
{
    SetInlineHook((DWORD)Add, (DWORD)HookFunc);
    Add(1, 2);
    UnInlineHook((DWORD)Add);

    return 0;
}
```

# 三 IAT Hook原理

![7](./../../操作/10Hook/7.jpg)

调用的这些API是在当前进程加载的dll中，调用这些API的时候，去当前进程的导入表中找dll，在dll的IAT表里找API的RVA，算出实际地址，调用



我自己进程有个导入表，指定了我加载了哪些DLL。这些被加载的DLL也应该有个导入表，他们也需要一些DLL做支持。我当前进程LoadLibrary一个dll的时候，我的导入表怎么变化，要导入的dll的导入表会怎么变化

我进程的导入表不变，会判断LoadLibrary目标进程的导入表里的dll是否已经在当前进程加载，目标进程的IAT会改变。



首先遍历要Hook的函数所属dll的INT表，找函数名或者序号，将要Hook的函数的IAT表改成其他函数的RVA

# 四 IAT Hook代码

```cpp
#include <iostream>
#include <windows.h>

using std::cout;
using std::cin;
using std::endl;

DWORD dwOldFunAddr = 0;

int WINAPI MyMessageBoxA(
    _In_opt_ HWND hWnd,
    _In_opt_ LPCSTR lpText,
    _In_opt_ LPCSTR lpCaption,
    _In_ UINT uType)
{
    printf("%d \r\n", hWnd);
    printf("%s \r\n", lpText);
    printf("%s \r\n", lpCaption);
    printf("%d \r\n", uType);

    typedef int(WINAPI* FUN_MESSAGEBOX)(HWND, LPCSTR, LPCSTR, UINT);
    FUN_MESSAGEBOX pMsgBox = (FUN_MESSAGEBOX)dwOldFunAddr;

    return pMsgBox(hWnd, lpText, lpCaption, uType);
}

VOID IAT_Hook(PCHAR szDllName, PCHAR szFunName, DWORD dwFunAddr)
{
    PCHAR hModule = 0;
    PIMAGE_DOS_HEADER pDos = 0;
    PIMAGE_NT_HEADERS pNth = 0;
    PIMAGE_IMPORT_DESCRIPTOR pImp = 0;
    PIMAGE_IMPORT_BY_NAME pName = 0;
    LPDWORD pIat = 0;
    LPDWORD pInt = 0;
    DWORD dwOld1 = 0;
    DWORD dwOld2 = 0;

    // 定位导入表
    hModule = (PCHAR)GetModuleHandle(NULL);// 获取主模块的IMAGEBASE
    if (!hModule)
    {
        return;
    }

    pDos = (PIMAGE_DOS_HEADER)hModule;
    pNth = (PIMAGE_NT_HEADERS)(hModule + pDos->e_lfanew);
    pImp = (PIMAGE_IMPORT_DESCRIPTOR)(
        pNth->OptionalHeader.DataDirectory[IMAGE_DIRECTORY_ENTRY_IMPORT].VirtualAddress);

    // 遍历查找指定模块
    do 
    {
        // 比较导入模块名
        if (strcmp(szDllName, hModule + pImp->Name) == 0)
        {
            pInt = (LPDWORD)(hModule + pImp->OriginalFirstThunk);
            pIat = (LPDWORD)(hModule + pImp->FirstThunk);

            // 遍历指定函数
            do 
            {
                if (*pInt & 0x80000000 == NULL)
                {
                    pName = (PIMAGE_IMPORT_BY_NAME)(hModule + *pInt);
                    if (strcmp(pName->Name, szFunName) == 0)
                    {
                        // 修改内存属性
                        VirtualProtect(pIat, 0x4, PAGE_READWRITE, &dwOld1);

                        // 备份原来IAT里函数的RVA
                        dwOldFunAddr = *pIat;

                        // 替换IAT中函数RVA
                        *pIat = dwFunAddr;

                        // 恢复内存属性
                        VirtualProtect(pIat, 0x4, dwOld1, &dwOld2);

                        break;
                    }
                }

                pInt++;
                pIat++;
            } while (*pInt);
        }

        pImp++;
    } while (pImp->OriginalFirstThunk && pImp->FirstThunk);
}

int main()
{
    // User32.dll
    MessageBoxA(0, 0, 0, 0);

    IAT_Hook((PCHAR)"USER32.dll", (PCHAR)"MessageBoxA", (DWORD)MyMessageBoxA);

    MessageBoxA(0, "Ferry", "IAT Hook", 0);

    IAT_Hook((PCHAR)"USER32.dll", (PCHAR)"MessageBoxA", (DWORD)dwOldFunAddr);

    MessageBoxA(0, "Ferry", "IAT Hook", 0);

    return 0;
}
```

# 五 虚表Hook原理

![8](./../../操作/10Hook/8.jpg)

找到虚表地址，解析虚表里的函数地址，替换为目标函数地址

# 六 虚表Hook代码

```cpp
#include <iostream>
#include <windows.h>

using std::cout;
using std::cin;
using std::endl;

class Base
{
public:
    virtual void Print() = 0;
};

class Son :public Base 
{
public:
    virtual void Print() 
    {
        printf("Son\r\n");
    }
};

VOID Fun()
{
    printf("Hack");
}

int main()
{
    Son s;
    Base* b = &s;
    b->Print();

    // 获取虚表
    DWORD dwVirtualTableAddr = (DWORD)(*(DWORD*)&s);

    DWORD dwOldPro1;
    DWORD dwOldPro2;
    VirtualProtect((LPVOID)dwVirtualTableAddr, 0x4, PAGE_EXECUTE_READWRITE, &dwOldPro1);
    // 替换虚表
    *(DWORD*)dwVirtualTableAddr = (DWORD)Fun;
    b->Print();

    VirtualProtect((LPVOID)dwVirtualTableAddr, 0x4, dwOldPro1, &dwOldPro2);

    return 0;
}
```

# 七 VEH_Hook异常Hook原理

![9](./../../操作/10Hook/9.jpg)

断点异常

![10](./../../操作/10Hook/10.jpg)

常用的API都有MOV EDI, EDI

VEH_HOOK，就是在函数头部下一个0xCC断点，我们自己再写一个异常回调函数，处理这个断点异常

# 八 VEH-Hook代码

```cpp
#include <iostream>
#include <windows.h>

using std::cout;
using std::cin;
using std::endl;

// 钩子地址
DWORD dwFunAddr = (DWORD)MessageBoxA;

// 设置钩子
VOID SetVehHook(DWORD dwHookAddr)
{
    DWORD dwOld = 0;

    // 修改内存属性
    VirtualProtect((LPVOID)dwHookAddr, 2, PAGE_READWRITE, &dwOld);

    // 覆盖原有数据
    *(PSHORT)dwHookAddr = 0x9090;

    // 设置INT3断点
    *(PCHAR)dwHookAddr = 0xCC;

    // 恢复内存属性
    VirtualProtect((LPVOID)dwHookAddr, 2, dwOld, &dwOld);
}

// 取消钩子
VOID UnVehHook(DWORD dwHookAddr)
{
    DWORD dwOld = 0;

    // 修改内存属性
    VirtualProtect((LPVOID)dwHookAddr, 2, MEM_COMMIT, &dwOld);

    // 恢复原有数据
    *(PSHORT)dwHookAddr = 0xFF8B;// MOV EDI, EDI

    // 恢复内存属性
    VirtualProtect((LPVOID)dwHookAddr, 2, dwOld, &dwOld);
}

// 异常处理
LONG NTAPI Handler(struct _EXCEPTION_POINTERS* ExceptionInfo)
{
    // 判断是否为CC断点异常
    if (ExceptionInfo->ExceptionRecord->ExceptionCode == EXCEPTION_BREAKPOINT)
    {
        // 判断是否为钩子地址
        if ((DWORD)ExceptionInfo->ExceptionRecord->ExceptionAddress == dwFunAddr)
        {
            // 替换MessageBoxA的第二个参数
            CHAR* szBuffer = (CHAR*)"VehHook";// 不能用字符数组。这里要一个堆区的地址
            *(PDWORD)(ExceptionInfo->ContextRecord->Esp + 8) = (DWORD)szBuffer;

            // 休整eip
            ExceptionInfo->ContextRecord->Eip += 2;

            return EXCEPTION_CONTINUE_EXECUTION;
        }

        return EXCEPTION_CONTINUE_SEARCH;
    }
}

int main()
{
    SetVehHook(dwFunAddr);

    // 异常处理
    AddVectoredExceptionHandler(1, Handler);

    MessageBoxA(0, 0, 0, 0);

    UnVehHook(dwFunAddr);

    return 0;
}
```

# 九 VEH-Hook代码2

![11](./../../操作/10Hook/11.jpg)

我们自己写的函数开头没有MOV EDI, EDI

```cpp
#include <iostream>
#include <windows.h>

using std::cout;
using std::cin;
using std::endl;

// 挂钩函数
int Fun(int a, int b)
{
    return a + b;
}

// 挂钩地址
DWORD dwFunAddr = (DWORD)Fun;

DWORD dwRetAddr = 0;

// 设置钩子
VOID SetVehHook(DWORD dwHookAddr)
{
    DWORD dwOld = 0;

    // 修改内存包含
    VirtualProtect((LPVOID)dwHookAddr, 0x1, PAGE_EXECUTE_READWRITE, &dwOld);

    // 设置int3断点
    *(PCHAR)dwHookAddr = 0xCC;

    // 恢复内存保护
    VirtualProtect((LPVOID)dwHookAddr, 0x1, dwOld, &dwOld);
}

// 取消钩子
VOID UnVehHook(DWORD dwHookAddr)
{
    DWORD dwOld = 0;

    // 修改内存包含
    VirtualProtect((LPVOID)dwHookAddr, 0x1, PAGE_EXECUTE_READWRITE, &dwOld);

    // 恢复默认数据
    *(PCHAR)dwHookAddr = 0x55;

    // 恢复内存保护
    VirtualProtect((LPVOID)dwHookAddr, 0x1, dwOld, &dwOld);
}

// 中转函数
_declspec(naked) void JmpRet()
{
    __asm
    {
        push ebp
        jmp [dwRetAddr]
    }
}

// 异常处理
LONG NTAPI Handler(struct _EXCEPTION_POINTERS* ExceptionInfo)
{
    // 判断是否为CC断点异常
    if (ExceptionInfo->ExceptionRecord->ExceptionCode == EXCEPTION_BREAKPOINT)
    {
        // 判断是否为钩子那个地址触发
        if ((DWORD)ExceptionInfo->ExceptionRecord->ExceptionAddress == dwFunAddr)
        {
            // 劫持堆栈数据
            printf("Ret: [%x]\r\n", *(LPDWORD)(ExceptionInfo->ContextRecord->Esp + 0));
            printf("Ret: [%x]\r\n", *(LPDWORD)(ExceptionInfo->ContextRecord->Esp + 4));
            printf("Ret: [%x]\r\n", *(LPDWORD)(ExceptionInfo->ContextRecord->Esp + 8));

            // 修正EIP
            dwRetAddr = (CHAR)ExceptionInfo->ContextRecord->Eip + 1;
            ExceptionInfo->ContextRecord->Eip = (DWORD)JmpRet;
            
            return EXCEPTION_CONTINUE_EXECUTION;
        }

        return EXCEPTION_CONTINUE_SEARCH;
    }
}

int main()
{
    SetVehHook(dwFunAddr);

    AddVectoredExceptionHandler(1, Handler);

    Fun(1, 2);

    UnVehHook(dwFunAddr);

    return 0;
}
```

# 十 无痕Hook前置基础

CRC32检测，在一个函数的硬编码中，启一个线程一直检测这个函数的每个字节。

IAT Hook检测，拿LoadLibrary的导出表，和导入表的IAT表里面做比对

![12](./../../操作/10Hook/12.jpg)

# 十一 Veh无痕Hook介绍

在每个线程中都下一个硬件断点

不会改变HOOK函数的硬编码

# 十二-十三 Veh无痕Hook、Veh无痕Hook多个函数代码

```cpp
#include <iostream>
#include <windows.h>
#include <TlHelp32.h>

using std::cout;
using std::cin;
using std::endl;

VOID Fun(int a, int b)
{

}

DWORD dwRet = 0x401009;

_declspec(naked) void JmpData()
{
    __asm
    {
        push ebp
        mov ebp, esp
        sub esp, 0xC0h

        jmp [dwRet]
    }
}

// 设置硬件断点
VOID SetThreadHook(HANDLE hThread)
{
    // 线程上下文结构体
    CONTEXT ctx = { 0 };

    // 设置获取标准
    ctx.ContextFlags = CONTEXT_ALL;

    // 获取线程上下文环境
    GetThreadContext(hThread, &ctx);

    // 修改调试寄存器
    // 0101 0101
    ctx.Dr0 = (DWORD)Fun;
    ctx.Dr7 = 0x1;// 开启Dr0

    ctx.Dr1 = (DWORD)MessageBoxA;
    ctx.Dr7 |= 0x4;

    // 设置线程上下文环境
    SetThreadContext(hThread, &ctx);
}

// 设置HOOK
VOID SetHook()
{
    HANDLE hSnap = 0;
    THREADENTRY32 te32 = { 0 };
    te32.dwSize = sizeof(te32);

    // 拍摄快照
    hSnap = CreateToolhelp32Snapshot(TH32CS_SNAPTHREAD, 0);
    if (hSnap == INVALID_HANDLE_VALUE)
    {
        return;
    }

    // 遍历线程
    if (Thread32First(hSnap, &te32))
    {
        do 
        {
            if (te32.th32OwnerProcessID == GetCurrentProcessId())
            {
                HANDLE hThread = OpenProcess(PROCESS_ALL_ACCESS, FALSE, te32.th32OwnerProcessID);
                if (hThread)
                {
                    SetThreadHook(hThread);
                }
            }
        } while (Thread32Next(hSnap, &te32));
    }
}

LONG NTAPI Handler(struct _EXCEPTION_POINTERS* ExceptionInfo)
{
    // 判断是否为硬件断点
    if (ExceptionInfo->ExceptionRecord->ExceptionCode == EXCEPTION_SINGLE_STEP)
    {
        // 判断是否为挂钩地址
        if ((DWORD)ExceptionInfo->ExceptionRecord->ExceptionAddress == (DWORD)Fun)
        {
            // 劫持数据
            printf("%08x\r\n", *(PDWORD)(ExceptionInfo->ContextRecord->Esp + 0));
            printf("%08x\r\n", *(PDWORD)(ExceptionInfo->ContextRecord->Esp + 4));
            printf("%08x\r\n", *(PDWORD)(ExceptionInfo->ContextRecord->Esp + 8));

            // 修正EIP
            ExceptionInfo->ContextRecord->Eip += 2;

            return EXCEPTION_CONTINUE_EXECUTION;
        }

        // 判断是否为挂钩地址
        if ((DWORD)ExceptionInfo->ExceptionRecord->ExceptionAddress == (DWORD)MessageBoxA)
        {
            // 劫持数据
            printf("%08x\r\n", *(PDWORD)(ExceptionInfo->ContextRecord->Esp + 0));
            printf("%08x\r\n", *(PDWORD)(ExceptionInfo->ContextRecord->Esp + 4));
            printf("%08x\r\n", *(PDWORD)(ExceptionInfo->ContextRecord->Esp + 8));

            // 修正EIP
            ExceptionInfo->ContextRecord->Eip = (DWORD)JmpData;

            return EXCEPTION_CONTINUE_EXECUTION;
        }

        return EXCEPTION_CONTINUE_SEARCH;
    }
}

int main()
{
    // 异常处理
    AddVectoredExceptionHandler(1, (PVECTORED_EXCEPTION_HANDLER)Handler);
    SetHook();
    Fun(1, 2);
    MessageBoxA(0, "1234", 0, 0);

    return 0;
}
```
