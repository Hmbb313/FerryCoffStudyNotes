# 零 win32的数据类型

![1](./../../操作/7Win32/1.jpg)

![2](./../../操作/7Win32/2.jpg)

# 一 OpenProcess

![3](./../../操作/7Win32/3.jpg)

进程的结构体是放在内核中的，OpenProcess返回的句柄就是内核中的一个索引

```cpp
#include <iostream>
#include <windows.h>
#include <winerror.h>

using std::cout;
using std::cin;
using std::endl;

int main()
{
    // 打开一个已经被打开的进程。不是创建进程
    HANDLE hProcess = OpenProcess(PROCESS_ALL_ACCESS, FALSE, 17932);
    if (hProcess == NULL)
    {
        DWORD dwError = GetLastError();

        cout << dwError << endl;
    }

    cout << hProcess << endl;

    return 0;
}
```

# 二 字符串编码

字符串编码

![4](./../../操作/7Win32/4.jpg)

![5](./../../操作/7Win32/5.jpg)

![6](./../../操作/7Win32/6.jpg)

# 三 进程

双击执行，操作系统会从程序入口点启一条线程。没有线程，进程就是一摊死水

![7](./../../操作/7Win32/7.jpg)

# 四 创建进程

创建进程

```cpp
int main()
{
    STARTUPINFOA si = { 0 };
    si.cb = sizeof(si);

    PROCESS_INFORMATION pi = { 0 };

    // 创建进程。这才是打开exe
    BOOL bRet = CreateProcessA(
        ".\\PETool 1.0.0.5.exe",
        NULL,
        NULL,
        NULL,
        NULL,
        CREATE_NEW_CONSOLE,
        NULL,
        NULL,
        &si,
        &pi
        );

    return 0;
}
```

# 五 退出进程

当进程里所有线程都执行完了，进程就退出了

退出进程

```cpp
int main()
{
    STARTUPINFOA si = { 0 };
    si.cb = sizeof(si);

    PROCESS_INFORMATION pi = { 0 };

    // 创建进程。这才是打开exe
    BOOL bRet = CreateProcessA(
        ".\\PETool 1.0.0.5.exe",
        NULL,
        NULL,
        NULL,
        NULL,
        CREATE_NEW_CONSOLE,
        NULL,
        NULL,
        &si,
        &pi
        );

    TerminateProcess(pi.hProcess, 0);

    return 0;
}
```

# 六 HANDLE内核对象

HANDLE是个内核对象，可以在内核里找到一个内核结构体

# 七 遍历系统进程

遍历系统进程

```cpp
#include <iostream>
#include <windows.h>
#include <winerror.h>
#include <TlHelp32.h>

using std::cout;
using std::cin;
using std::endl;

int main()
{
    HANDLE hSnap = CreateToolhelp32Snapshot(TH32CS_SNAPPROCESS, 0);
    if (hSnap == INVALID_HANDLE_VALUE)
    {
        printf("%x \r\n", GetLastError());
    }

    PROCESSENTRY32 pe32 = { 0 };
    pe32.dwSize = sizeof(pe32);

    BOOL bRet = Process32First(hSnap, &pe32);
    while (bRet)
    {
        // 进程名，进程id
        wprintf(L"%s %d \r\n", pe32.szExeFile, pe32.th32ProcessID);

        bRet = Process32Next(hSnap, &pe32);
    }

    return 0;
}
```

# 八 父子进程

父子进程

```cpp
#include <iostream>
#include <windows.h>
#include <winerror.h>
#include <TlHelp32.h>

using std::cout;
using std::cin;
using std::endl;

int main()
{
    STARTUPINFOA si = { 0 };
    si.cb = sizeof(si);

    PROCESS_INFORMATION pi = { 0 };

    CreateProcessA(
        ".\\PETool 1.0.0.5.exe",
        NULL,
        NULL,
        NULL,
        NULL,
        CREATE_NEW_CONSOLE,
        NULL,
        NULL,
        &si,
        &pi
    );

    HANDLE hSnap = CreateToolhelp32Snapshot(TH32CS_SNAPPROCESS, 0);
    if (hSnap == INVALID_HANDLE_VALUE)
    {
        printf("%x \r\n", GetLastError());
    }

    PROCESSENTRY32 pe32 = { 0 };
    pe32.dwSize = sizeof(pe32);

    BOOL bRet = Process32First(hSnap, &pe32);
    while (bRet)
    {
        // 遍历进程，找到进程
        if (wcscmp(pe32.szExeFile, L"PETool 1.0.0.5.exe") == 0)
        {
            // 拿这个进程的父进程ID
            printf("%d \r\n", pe32.th32ParentProcessID);
        }

        bRet = Process32Next(hSnap, &pe32);
    }

    return 0;
}
```

当前进程CreateProcess了一个进程，当前进程就是创建的那个进程的父进程

# 九 挂起和恢复进程

挂起和恢复进程

```cpp
#include <iostream>
#include <windows.h>
#include <winerror.h>
#include <TlHelp32.h>

using std::cout;
using std::cin;
using std::endl;

typedef DWORD(WINAPI* SuspendProcss)(HANDLE);
typedef DWORD(WINAPI* ResumeProcess)(HANDLE);

int main()
{
    HMODULE hModule = LoadLibraryA("ntdll.dll");

    SuspendProcss sus = (SuspendProcss)GetProcAddress(hModule, "ZwSuspendProcess");
    ResumeProcess res = (ResumeProcess)GetProcAddress(hModule, "ZwResumeProcess");

    HANDLE hProcess = OpenProcess(PROCESS_ALL_ACCESS, FALSE, 17292);

    // 挂起进程(假死)
    sus(hProcess);

    // 恢复
    res(hProcess);

    return 0;
}
```

# 十 读写当前进程的内存

读写当前进程的内存

```cpp
#include <iostream>
#include <windows.h>
#include <winerror.h>

using std::cout;
using std::cin;
using std::endl;

int main()
{
    // 当前进程自身的PID
    DWORD pId = GetCurrentProcessId();
    HANDLE hProcess = GetCurrentProcess();

    DWORD dwData = 0;
    DWORD dwRead = 0;

    BOOL bRet = ReadProcessMemory(hProcess, (LPVOID)0x400000, &dwData, 4, &dwRead);
    if (bRet == 0)
    {
        cout << GetLastError() << endl;
    }

    dwData = 0x12345678;

    DWORD dwOld = 0;
    // 这个函数只能修改当前进程的权限
    VirtualProtect((LPVOID)0x400000, 4, PAGE_READWRITE, &dwOld);

    bRet = WriteProcessMemory(hProcess, (LPVOID)0x400000, &dwData, 4, &dwRead);
    if (bRet == 0)
    {
        cout << GetLastError() << endl;
    }

    VirtualProtect((LPVOID)0x400000, 4, dwOld, &dwOld);

    return 0;
}
```

# 十一 跨进程读写内存

跨进程读写内存

```cpp
#include <iostream>
#include <windows.h>
#include <winerror.h>

using std::cout;
using std::cin;
using std::endl;

DWORD g_Data = 0xFFFFFFFF;

int main()
{
    // 获取当前进程的pid
    DWORD dwPid = GetCurrentProcessId();
    cout << dwPid << endl;// 当前进程的pid
    cout << &g_Data << endl;// 当前进程中g_Data数据的地址

    system("pause");

    while (1)
    {
        system("cls");
        cout << g_Data << endl;
        g_Data--;
        Sleep(1000);// 睡1秒
    }

    return 0;
}
```

![8](./../../操作/7Win32/8.jpg)

```cpp
#include <iostream>
#include <windows.h>
#include <winerror.h>

using std::cout;
using std::cin;
using std::endl;

int main()
{
    DWORD dwData = 0;
    DWORD dwRead = 0;

    HANDLE hProcess = OpenProcess(PROCESS_ALL_ACCESS, FALSE, 27988);
    if (hProcess == 0)
    {
        cout << GetLastError() << endl;
    }

    dwData = 100;
    WriteProcessMemory(hProcess, (LPDWORD)0x0041A044, &dwData, 4, &dwRead);

    while (1)
    {
        system("cls");
        ReadProcessMemory(hProcess, (LPDWORD)0x0041A044, &dwData, 4, &dwRead);
        cout << dwData << endl;
        Sleep(1000);
    }
    

    return 0;
}
```

![9](./../../操作/7Win32/9.jpg)

# 十二 CreateFileMappingA

CreateFileMappingA，创建一个带有名字的内核对象。跨进程

```cpp
#include <iostream>
#include <windows.h>
#include <winerror.h>

using std::cout;
using std::cin;
using std::endl;

int main()
{
    // 内核对象.创建文件映射对象
    HANDLE hFileMap = CreateFileMappingA(
        INVALID_HANDLE_VALUE, NULL, PAGE_READWRITE, 0, 256, "Ferry");
    if (hFileMap == NULL)
    {
        cout << GetLastError() << endl;
    }

    // 用FILE_ALL_ACCESS可能会拒绝访问
    // 将文件映射对象映射到当前内存
    LPVOID lpBuffer = MapViewOfFile(hFileMap, FILE_READ_ACCESS | FILE_WRITE_ACCESS, 0, 0, 256);
    if(lpBuffer == NULL)
    { 
        cout << GetLastError() << endl;
    }
    
    memcpy(lpBuffer, "Ferry-Process-Msg", strlen("Ferry-Process-Msg") + 1);

    system("pause");

    UnmapViewOfFile(lpBuffer);

    CloseHandle(hFileMap);

    return 0;
}
```

打开共享内存

```cpp
#include <iostream>
#include <windows.h>
#include <winerror.h>

using std::cout;
using std::cin;
using std::endl;

int main()
{
    // 打开文件映射对象
    HANDLE hFileMap = OpenFileMappingA(FILE_MAP_ALL_ACCESS, FALSE, "Ferry");
    if (hFileMap == NULL)
    {
        cout << GetLastError() << endl;
    }

    // 将文件映射对象映射到当前内存
    LPVOID lpBuffer = MapViewOfFile(hFileMap, FILE_MAP_ALL_ACCESS, 0, 0, 256);
    if (lpBuffer == NULL)
    {
        cout << GetLastError() << endl;
    }

    MessageBoxA(NULL, (LPCSTR)lpBuffer, NULL, NULL);

    // 这里只是使用共享内存，不用删除共享内存
    // UnmapViewOfFile(lpBuffer);

    CloseHandle(hFileMap);

    return 0;
}
```

# 十三 命名管道

命名管道

### 服务端

```cpp
#include <iostream>
#include <windows.h>
#include <winerror.h>

using std::cout;
using std::cin;
using std::endl;

int main()
{
    // 创建一个命名管道
    HANDLE hPipe = CreateNamedPipeA(
        ".\\pipe\\Ferry_1",
        PIPE_ACCESS_DUPLEX,
        PIPE_TYPE_MESSAGE | PIPE_READMODE_MESSAGE | PIPE_WAIT,
        PIPE_UNLIMITED_INSTANCES,
        0,
        0,
        NMPWAIT_USE_DEFAULT_WAIT,
        NULL
    );
    if (hPipe == INVALID_HANDLE_VALUE)
    {
        cout << GetLastError() << endl;
    }

    // 等待客户端连接
    BOOL bRet = ConnectNamedPipe(hPipe, NULL);
    if (!bRet)
    {
        cout << "连接失败" << endl;
    }

    // 接受消息
    char szBuffer[256] = { 0 };
    DWORD dwRead = 0;
    ReadFile(hPipe, szBuffer, 256, &dwRead, NULL);

    // 发送消息
    DWORD dwData = 0x12345678;
    WriteFile(hPipe, &dwData, 4, &dwRead, NULL);

    // 关闭管道
    FlushFileBuffers(hPipe);
    DisconnectNamedPipe(hPipe);
    CloseHandle(hPipe);

    return 0;
}
```

### 客户端

```cpp
#include <iostream>
#include <windows.h>
#include <winerror.h>

using std::cout;
using std::cin;
using std::endl;

int main()
{
    // 等待一个已经创建好了的命名管道
    BOOL bRet = WaitNamedPipeA(".\\pipe\\Ferry_1", NMPWAIT_WAIT_FOREVER);
    
    // 打开一个已经创建好的命名管道
    HANDLE hPipe = CreateFileA(
        ".\\pipe\\Ferry_1",
        GENERIC_READ | GENERIC_WRITE,
        0,
        NULL,
        OPEN_EXISTING,
        FILE_ATTRIBUTE_NORMAL,
        NULL
    );

    // 发送消息
    DWORD dwRead = 0;
    char szBuffer[256] = "Ferry-Process-Msg";
    WriteFile(hPipe, szBuffer, 256, &dwRead, NULL);

    // 接受消息
    ReadFile(hPipe, szBuffer, 256, &dwRead, NULL);

    // 关闭管道
    CloseHandle(hPipe);

    return 0;
}
```

# 十四 匿名管道

匿名管道

### 父进程

```cpp
#include <iostream>
#include <windows.h>
#include <winerror.h>

using std::cout;
using std::cin;
using std::endl;

int main()
{
    SECURITY_ATTRIBUTES sa = { 0 };
    sa.bInheritHandle = TRUE;
    sa.lpSecurityDescriptor = NULL;
    sa.nLength = sizeof(sa);

    HANDLE hRead = 0;
    HANDLE hWrite = 0;

    BOOL bRet = CreatePipe(&hRead, &hWrite, &sa, 0);
    if (!bRet)
    {
        cout << GetLastError() << endl;
    }

    STARTUPINFOA si = { 0 };
    si.cb = sizeof(si);
    si.dwFlags = STARTF_USESTDHANDLES;
    si.hStdInput = hRead;
    si.hStdOutput = hWrite;
    si.hStdError = GetStdHandle(STD_ERROR_HANDLE);

    PROCESS_INFORMATION pi = { 0 };

    CreateProcessA(
        "..\\TestDemo\\Debug\\TestDemo.exe",
        NULL,
        NULL,
        NULL,
        TRUE,
        CREATE_NEW_CONSOLE,
        NULL,
        NULL,
        &si,
        &pi
    );

    char szBuffer[256] = "Ferry-Process-Msg";
    DWORD dwRead = 0;

    while (1)
    {
        WriteFile(hWrite, szBuffer, 256, &dwRead, NULL);
    }


    return 0;
}
```

### 子进程

```cpp
#include <iostream>
#include <windows.h>
#include <winerror.h>

using std::cout;
using std::cin;
using std::endl;

int main()
{
    HANDLE hRead = GetStdHandle(STD_INPUT_HANDLE);
    HANDLE hWrite = GetStdHandle(STD_OUTPUT_HANDLE);

    char szBuffer[256] = { 0 };
    DWORD dwRead = 0;

    while (1)
    {
        ReadFile(hRead, szBuffer, 256, &dwRead, NULL);
        MessageBoxA(NULL, szBuffer, NULL, NULL);
    }

    return 0;
}
```

# 十五 Mailslot

### 服务端

```cpp
#include <iostream>
#include <windows.h>
#include <winerror.h>

using std::cout;
using std::cin;
using std::endl;

int main()
{
    HANDLE hMail = CreateMailslotA(
        "\\\\.\\mailslot\\Ferry_1",
        0,
        MAILSLOT_WAIT_FOREVER,
        NULL
    );
    if (hMail == INVALID_HANDLE_VALUE)
    {
        cout << GetLastError() << endl;
    }

    char szBuff[256] = { 0 };
    DWORD dwRead = 0;

    ReadFile(hMail, szBuff, 256, &dwRead, NULL);
    printf("%s \r\n", szBuff);

    CloseHandle(hMail);

    return 0;
}
```

### 客户端

```cpp
#include <iostream>
#include <windows.h>
#include <winerror.h>

using std::cout;
using std::cin;
using std::endl;

int main()
{
    HANDLE hMail = CreateFileA(
        "\\\\.\\mailslot\\Ferry_1",
        GENERIC_READ | GENERIC_WRITE,
        FILE_SHARE_READ,
        NULL,
        OPEN_EXISTING,
        FILE_ATTRIBUTE_NORMAL,
        NULL
    );
    if (hMail == INVALID_HANDLE_VALUE)
    {
        cout << GetLastError() << endl;
    }

    char szBuffer[256] = "Ferry-Process";
    DWORD dwWrite = 0;

    WriteFile(hMail, szBuffer, 256, &dwWrite, NULL);

    return 0;
}
```

# 十六 CloseHandle的时机

CreateProcess

CreateFileMapping

CreatePipe

CreateNamedPipe

CreateMailslot

这些类似API会让内核对象(HANDLE)内部的引用计数+1

需要CloseHandle()

# 十七 主线程

![10](./../../操作/7Win32/10.jpg)

当前有三条线程



PE文件内容贴到进程中时，操作系统会自动启一条线程，从OPTION_HEADER中的EntryPoint+ImageBase开始，这样才叫进程



TerminateProcess()就是把当前进程的所有线程释放，没有线程，操作系统会释放当前进程

# 十八 每个线程都有自己独立的堆栈

每个线程都有自己独立的堆栈

```cpp
#include <iostream>
#include <windows.h>
#include <winerror.h>

using std::cout;
using std::cin;
using std::endl;

DWORD WINAPI WorkThread(LPVOID lp)
{
    while (1)
    {
        cout << "WorkThread" << endl;
    }
}

int main()
{
    HANDLE hThread = CreateThread(
        NULL,
        0,
        WorkThread,
        NULL,
        0,
        NULL
    );
    if (hThread == NULL)
    {
        cout << GetLastError() << endl;
    }

    while (1)
    {
        cout << "main" << endl;
    }

    CloseHandle(hThread);

    return 0;
}
```

WorkThread和Main是两个不同的线程，有两个不同的堆栈

# 十九 线程退出

![11](./../../操作/7Win32/11.jpg)

主线程里起了一条新线程，然后主线程马上结束了

![12](./../../操作/7Win32/12.jpg)

### 线程执行完，线程返回，线程就自然死亡了

```cpp
#include <iostream>
#include <windows.h>
#include <winerror.h>

using std::cout;
using std::cin;
using std::endl;

DWORD WINAPI WorkThread(LPVOID lp)
{
    for (size_t i = 0; i < 10; i++)
    {
        cout << i << endl;
        Sleep(1000);
    }
    
    return 0;
}

int main()
{
    HANDLE hThread = CreateThread(
        NULL,
        0,
        WorkThread,
        NULL,
        0,
        NULL
    );

    DWORD dwExitCode = 0;
    BOOL bRet = GetExitCodeThread(hThread, &dwExitCode);
    cout << bRet << "\t" << dwExitCode << endl;

    system("pause");

    bRet = GetExitCodeThread(hThread, &dwExitCode);
    cout << bRet << "\t" << dwExitCode << endl;

    CloseHandle(hThread);

    return 0;
}
```

dwExitCode是259，说明线程正在执行，如果线程正常退出，dwExitCode就是线程的返回值

# 二十 退出线程

### ExitThread

```cpp
#include <iostream>
#include <windows.h>
#include <winerror.h>

using std::cout;
using std::cin;
using std::endl;

DWORD WINAPI WorkThread(LPVOID lp)
{
    for (size_t i = 0; i < 10; i++)
    {
        cout << i << endl;
        Sleep(100);

        if (i == 5)
        {
            ExitThread(5);// 线程退出码为5
        }
    }
    
    return 0;
}

int main()
{
    HANDLE hThread = CreateThread(
        NULL,
        0,
        WorkThread,
        NULL,
        0,
        NULL
    );

    system("pause");

    DWORD dwExitCode = 0;
    BOOL bRet = GetExitCodeThread(hThread, &dwExitCode);
    cout << bRet << "\t" << dwExitCode << endl;

    CloseHandle(hThread);

    return 0;
}
```

### TerminateThread

```cpp
#include <iostream>
#include <windows.h>
#include <winerror.h>

using std::cout;
using std::cin;
using std::endl;

DWORD WINAPI WorkThread(LPVOID lp)
{
    for (size_t i = 0; i < 100; i++)
    {
        cout << i << endl;
        Sleep(100);
    }
    
    return 0;
}

int main()
{
    HANDLE hThread = CreateThread(
        NULL,
        0,
        WorkThread,
        NULL,
        0,
        NULL
    );

    system("pause");

    // 给另一条线程发个东西，对方收到了，就自杀了
    TerminateThread(hThread, 3);

    system("pause");

    CloseHandle(hThread);

    return 0;
}
```

### 不推荐TerminateThread

```cpp
#include <iostream>
#include <windows.h>
#include <winerror.h>

using std::cout;
using std::cin;
using std::endl;

DWORD WINAPI WorkThread(LPVOID lp)
{
    int* p = new int;
    *p = 0x12345678;

    cout << std::hex << p << endl;

    for (size_t i = 0; i < 100; i++)
    {
        Sleep(100);
    }
    
    delete p;

    return 0;
}

int main()
{
    HANDLE hThread = CreateThread(
        NULL,
        0,
        WorkThread,
        NULL,
        0,
        NULL
    );

    system("pause");

    // 给另一条线程发个东西，对方收到了，就自杀了
    TerminateThread(hThread, 3);

    system("pause");

    CloseHandle(hThread);

    return 0;
}
```

TerminateThread的时候并不知道另一条线程到底执行到哪了。

### 用标志位

全局变量

```cpp
#include <iostream>
#include <windows.h>
#include <winerror.h>

using std::cout;
using std::cin;
using std::endl;

DWORD dwThreadFlag = 0;

DWORD WINAPI WorkThread(LPVOID lp)
{
    int* p = new int;
    *p = 0x12345678;

    cout << std::hex << p << endl;

    if (dwThreadFlag == 1)
    {
        delete p;
        ExitThread(2);
    }

    return 0;
}

int main()
{
    HANDLE hThread = CreateThread(
        NULL,
        0,
        WorkThread,
        NULL,
        0,
        NULL
    );

    system("pause");

    // 给另一条线程发个东西，对方收到了，就自杀了
    dwThreadFlag = 1;

    system("pause");

    CloseHandle(hThread);

    return 0;
}
```

这也有问题，万一第二个线程已经执行过了if语句块，再设置标志位也没用了

# 二十一 挂起和恢复线程

挂起一个正在运行的线程

恢复一个挂起的线程

```cpp
#include <iostream>
#include <windows.h>
#include <winerror.h>

using std::cout;
using std::cin;
using std::endl;

DWORD WINAPI WorkThread(LPVOID lp)
{
    DWORD dwFlag = 0;
    while (1)
    {
        cout << "WorkThread" << dwFlag << endl;
        dwFlag++;

        Sleep(500);
    }
    
    return 0;
}

int main()
{
    HANDLE hThread = CreateThread(
        NULL,
        0,
        WorkThread,
        NULL,
        0,
        NULL
    );

    DWORD dwFlag = 0;
    while (1)
    {
        cout << "Main" << dwFlag << endl;
        dwFlag++;

        if (dwFlag == 5)
        {
            SuspendThread(hThread);
        }

        if (dwFlag == 10)
        {
            ResumeThread(hThread);
        }

        Sleep(500);
    }

    CloseHandle(hThread);

    return 0;
}
```

```cpp
#include <iostream>
#include <windows.h>
#include <winerror.h>

using std::cout;
using std::cin;
using std::endl;

DWORD dwThreadId = 0;
HANDLE hMain = 0;

DWORD WINAPI WorkThread(LPVOID lp)
{
    // 打开主线程，获得主线程句柄
    HANDLE hThread = OpenThread(THREAD_ALL_ACCESS, FALSE, dwThreadId);

    DWORD dwFlag = 0;
    while (1)
    {
        cout << "WorkThread" << dwFlag << endl;
        dwFlag++;

        if (dwFlag == 5)
        {
            SuspendThread(hThread);
        }

        if (dwFlag == 10)
        {
            ResumeThread(hThread);
        }

        Sleep(500);
    }
    
    return 0;
}

int main()
{
    // 获取当前主线程的线程id
    dwThreadId = GetCurrentThreadId();

    HANDLE hThread = CreateThread(
        NULL,
        0,
        WorkThread,
        NULL,
        0,
        NULL
    );

    DWORD dwFlag = 0;
    while (1)
    {
        cout << "Main" << dwFlag << endl;
        dwFlag++;
        Sleep(500);
    }

    CloseHandle(hThread);

    return 0;
}
```

主线程句柄放全局变量，子线程也可以挂起主线程

# 二十二 遍历线程

线程遍历

```cpp
#include <iostream>
#include <windows.h>
#include <winerror.h>
#include <TlHelp32.h>

using std::cout;
using std::cin;
using std::endl;

int main()
{
    THREADENTRY32 te32 = { 0 };
    te32.dwSize = sizeof(te32);

    HANDLE hSnap = CreateToolhelp32Snapshot(TH32CS_SNAPTHREAD, 0);
    if (hSnap == INVALID_HANDLE_VALUE)
    {
        cout << GetLastError() << endl;
    }

    BOOL bRet = Thread32First(hSnap, &te32);
    while (bRet)
    {
        // 12992是VS2022
        if (te32.th32OwnerProcessID == 12992)
        {
            cout << "ThreadId -> " << te32.th32ThreadID << endl;
        }

        bRet = Thread32Next(hSnap, &te32);
    }

    CloseHandle(hSnap);

    return 0;
}
```

# 二十三 WaitForSingleObject

### 等单个信号

```cpp
#include <iostream>
#include <windows.h>
#include <winerror.h>
#include <TlHelp32.h>

using std::cout;
using std::cin;
using std::endl;

DWORD WINAPI Work1(LPVOID lp)
{
	for (size_t i = 0; i < 5; i++)
	{
		cout << i << endl;
		Sleep(1000);
	}

	return 0;
}

int main()
{
	HANDLE hThread = CreateThread(NULL, 0, Work1, NULL, 0, NULL);
	if (hThread == INVALID_HANDLE_VALUE)
	{
		cout << GetLastError() << endl;
	}

	// 等hThread线程执行完之后，才能继续向下执行
	WaitForSingleObject(hThread, INFINITE);

	// 等hThread线程执行，只等2秒
	// WaitForSingleObject(hThread, 2000);


	system("pause");

    return 0;
}
```

### 同时等多个对象

```cpp
#include <iostream>
#include <windows.h>
#include <winerror.h>
#include <TlHelp32.h>

using std::cout;
using std::cin;
using std::endl;

DWORD WINAPI Work1(LPVOID lp)
{
    for (size_t i = 0; i < 5; i++)
    {
        cout << i << endl;
        Sleep(1000);
    }

    return 0;
}

DWORD WINAPI Work2(LPVOID lp)
{
    for (size_t i = 0; i < 5; i++)
    {
        cout << i << endl;
        Sleep(1000);
    }

    return 0;
}

int main()
{
    HANDLE Arr_Handle[2] = { 0 };


    Arr_Handle[0] = CreateThread(NULL, 0, Work1, NULL, 0, NULL);
    Arr_Handle[1] = CreateThread(NULL, 0, Work1, NULL, 0, NULL);
    // 等两个线程都执行完之后，才能继续向下执行
    DWORD bRet = WaitForMultipleObjects(2, Arr_Handle, TRUE, INFINITE);

    // 等两个线程中只要有一个执行完，就能继续向下执行
    // DWORD bRet = WaitForMultipleObjects(2, Arr_Handle, FALSE, INFINITE);

    system("pause");

    return 0;
} = 
```

# 二十四 临界区

```cpp
#include <iostream>
#include <windows.h>
#include <winerror.h>
#include <TlHelp32.h>

using std::cout;
using std::cin;
using std::endl;

DWORD g_dwData = 0;

DWORD WINAPI Fun1(LPVOID lp)
{
    for (size_t i = 0; i < 100000; i++)
    {
        g_dwData++;
    }

    return 0;
}

DWORD WINAPI Fun2(LPVOID lp)
{
    for (size_t i = 0; i < 100000; i++)
    {
        g_dwData++;
    }

    return 0;
}

int main()
{
    HANDLE hThread1 = CreateThread(NULL, 0, Fun1, NULL, 0, NULL);
    HANDLE hThread2 = CreateThread(NULL, 0, Fun2, NULL, 0, NULL);

    WaitForSingleObject(hThread1, INFINITE);
    WaitForSingleObject(hThread2, INFINITE);

    cout << g_dwData << endl;

    return 0;
}
```

![13](./../../操作/7Win32/13.jpg)

一个线程是有个执行时间的，操作系统分配

如果线程1刚执行完mov eax, dword ptr [g_dwData]，切换到线程2执行了。假设g_dwData为1

线程2执行完mov eax, dword ptr [g_dwData]，g_dwData为2

线程1继续执行mov eax, dword ptr [g_dwData]已经执行过了，结果g_dwData还是2，相当于少执行了一次

### 临界区

```cpp
#include <iostream>
#include <windows.h>
#include <winerror.h>
#include <TlHelp32.h>

using std::cout;
using std::cin;
using std::endl;

DWORD g_dwData = 0;

// 创建临界区结构体
CRITICAL_SECTION cs = { 0 };

DWORD WINAPI Fun1(LPVOID lp)
{
    for (size_t i = 0; i < 100000; i++)
    {
        EnterCriticalSection(&cs);
        g_dwData++;// 必须将这条指令完全执行完，才会切换线程
        LeaveCriticalSection(&cs);
    }

    return 0;
}

DWORD WINAPI Fun2(LPVOID lp)
{
    for (size_t i = 0; i < 100000; i++)
    {
        EnterCriticalSection(&cs);
        g_dwData++;// 必须将这条指令完全执行完，才会切换线程
        LeaveCriticalSection(&cs);
    }

    return 0;
}

int main()
{
    // 初始化临界区结构体
    InitializeCriticalSection(&cs);

    HANDLE hThread1 = CreateThread(NULL, 0, Fun1, NULL, 0, NULL);
    HANDLE hThread2 = CreateThread(NULL, 0, Fun2, NULL, 0, NULL);

    WaitForSingleObject(hThread1, INFINITE);
    WaitForSingleObject(hThread2, INFINITE);

    cout << g_dwData << endl;
    
	// 删除临界区
    DeleteCriticalSection(&cs);

    return 0;
}
```

# 二十五 Event

Event

```cpp
#include <iostream>
#include <windows.h>
#include <winerror.h>
#include <TlHelp32.h>

using std::cout;
using std::cin;
using std::endl;

DWORD g_dwData = 0;

HANDLE g_hEvent = 0;

DWORD WINAPI Fun1(LPVOID lp)
{
    WaitForSingleObject(g_hEvent, INFINITE);
    ResetEvent(g_hEvent);

    for (size_t i = 0; i < 100000; i++)
    {
        g_dwData++;
    }

    SetEvent(g_hEvent);

    return 0;
}

DWORD WINAPI Fun2(LPVOID lp)
{
    WaitForSingleObject(g_hEvent, INFINITE);
    ResetEvent(g_hEvent);

    for (size_t i = 0; i < 100000; i++)
    {
        g_dwData++;
    }

    SetEvent(g_hEvent);

    return 0;
}

int main()
{
    // 创建EVENT
    // 第二个参数是TRUE，就要我们自己在等到信号之后ResetEvent(g_hEvent);，否则就是他自动帮我们释放
    // 第三个参数是TRUE，CreateEvent完成之后，g_hEvent默认有信号。否则还需要先SetEvent(g_hEvent);
    g_hEvent = CreateEvent(NULL, TRUE, TRUE, NULL);

    HANDLE hThread1 = CreateThread(NULL, 0, Fun1, NULL, 0, NULL);
    HANDLE hThread2 = CreateThread(NULL, 0, Fun2, NULL, 0, NULL);

    WaitForSingleObject(hThread1, INFINITE);
    WaitForSingleObject(hThread2, INFINITE);

    cout << g_dwData << endl;

    // 释放EVENT
    CloseHandle(g_hEvent);

    return 0;
}
```

一个线程等到事件了，就把事件释放了，让其他线程不能等到事件，不能执行。只有有事件的线程执行完指令，重新设置事件，其他线程才能等到事件

# 二十六 Semaphore

Semaphore

```cpp
#include <iostream>
#include <windows.h>
#include <winerror.h>
#include <TlHelp32.h>

using std::cout;
using std::cin;
using std::endl;

DWORD g_dwData = 0;
HANDLE g_hSema = 0;

DWORD WINAPI Fun1(LPVOID lp)
{
    // 等到信号量之后，信号量会-1
    WaitForSingleObject(g_hSema, INFINITE);

    for (size_t i = 0; i < 100000; i++)
    {
        g_dwData++;
    }

    // 恢复一个信号量
    ReleaseSemaphore(g_hSema, 1, NULL);

    return 0;
}

DWORD WINAPI Fun2(LPVOID lp)
{
    WaitForSingleObject(g_hSema, INFINITE);

    for (size_t i = 0; i < 100000; i++)
    {
        g_dwData++;
    }

    ReleaseSemaphore(g_hSema, 1, NULL);

    return 0;
}

int main()
{
    // 创建信号量对象
    // 初始信号量1，最大信号量2
    g_hSema = CreateSemaphoreA(NULL, 1, 2, NULL);

    HANDLE hThread1 = CreateThread(NULL, 0, Fun1, NULL, 0, NULL);
    HANDLE hThread2 = CreateThread(NULL, 0, Fun2, NULL, 0, NULL);

    WaitForSingleObject(hThread1, INFINITE);
    WaitForSingleObject(hThread2, INFINITE);

    cout << g_dwData << endl;

    // 释放信号量对象
    CloseHandle(g_hSema);

    return 0;
}
```

# 二十七 Mutex

Mutex

跟临界区基本一致。只是多了一个等待时间

```cpp
#include <iostream>
#include <windows.h>
#include <winerror.h>
#include <TlHelp32.h>

using std::cout;
using std::cin;
using std::endl;

DWORD g_dwData = 0;
HANDLE g_hMutex = 0;

DWORD WINAPI Fun1(LPVOID lp)
{
    for (size_t i = 0; i < 100000; i++)
    {
        // 等待互斥体
        WaitForSingleObject(g_hMutex, INFINITE);

        g_dwData++;

        // 恢复互斥体
        ReleaseMutex(g_hMutex);
    }

    return 0;
}

DWORD WINAPI Fun2(LPVOID lp)
{
    for (size_t i = 0; i < 100000; i++)
    {
        // 等待互斥体
        WaitForSingleObject(g_hMutex, INFINITE);

        g_dwData++;

        // 恢复互斥体
        ReleaseMutex(g_hMutex);
    }

    return 0;
}

int main()
{
    // 创建互斥体对象
    g_hMutex = CreateMutex(NULL, FALSE, NULL);

    HANDLE hThread1 = CreateThread(NULL, 0, Fun1, NULL, 0, NULL);
    HANDLE hThread2 = CreateThread(NULL, 0, Fun2, NULL, 0, NULL);

    WaitForSingleObject(hThread1, INFINITE);
    WaitForSingleObject(hThread2, INFINITE);

    cout << g_dwData << endl;

    // 释放互斥体对象
    CloseHandle(g_hMutex);

    return 0;
}
```

```cpp
DWORD WINAPI Fun1(LPVOID lp)
{
    for (size_t i = 0; i < 100000; i++)
    {
        // 等待互斥体
        WaitForSingleObject(g_hMutex, INFINITE);

        cout << "1" << endl;
        ExitThread(5);

        // 恢复互斥体
        ReleaseMutex(g_hMutex);
    }

    return 0;
}

DWORD WINAPI Fun2(LPVOID lp)
{
    for (size_t i = 0; i < 100000; i++)
    {
        // 等待互斥体
        WaitForSingleObject(g_hMutex, INFINITE);

        g_dwData++;

        // 恢复互斥体
        ReleaseMutex(g_hMutex);
    }

    return 0;
}
```

就算线程意外死亡，另一个线程也能帮意外死亡的线程恢复互斥体。临界区不行

# 二十八 互斥体防止多开

互斥体防止多开

```cpp
#include <iostream>
#include <windows.h>
#include <winerror.h>
#include <TlHelp32.h>

using std::cout;
using std::cin;
using std::endl;

int main()
{
    HANDLE hMutex = OpenMutexA(MUTEX_ALL_ACCESS, FALSE, "Ferry_1");
    if (hMutex == NULL)
    {
        CreateMutexA(NULL, FALSE, "Ferry_1");
    }
    else
    {
        MessageBoxA(NULL, "禁止躲开", NULL, NULL);
    }

    system("pause");

    return 0;
}
```

```cpp
#include <iostream>
#include <windows.h>
#include <winerror.h>
#include <TlHelp32.h>

using std::cout;
using std::cin;
using std::endl;

int main()
{
    HANDLE hMutex = CreateMutexA(NULL, FALSE, "Ferry_1");
    DWORD dwError = GetLastError();
    if (dwError == ERROR_ALREADY_EXISTS)
    {
        cout << "禁止躲开" << endl;
        exit(0);
    }

    system("pause");

    return 0;
}
```

# 二十九 CreateFileA

CreateFileA，创建/打开一个IO文件，文件、邮槽、管道等。

# 三十 创建文件

创建文件

```cpp
#include <iostream>
#include <windows.h>
#include <winerror.h>
#include <TlHelp32.h>

using std::cout;
using std::cin;
using std::endl;

int main()
{
    HANDLE hFile = CreateFileA(
        "\\C:\\User\\Administrato\\Desktop\\123.txt",
        GENERIC_ALL,
        FILE_SHARE_READ,
        NULL,
        CREATE_ALWAYS,// OPEN_EXISTING这两个用的最多
        FILE_ATTRIBUTE_NORMAL,
        NULL
    );

    cout << GetLastError() << endl;

    CloseHandle(hFile);

    return 0;
}
```

# 三十一 文件读写操作

文件读写操作

```cpp
#include <iostream>
#include <windows.h>
#include <winerror.h>
#include <TlHelp32.h>

using std::cout;
using std::cin;
using std::endl;

int main()
{
    HANDLE hFile = CreateFileA(
        "\\C:\\Users\\Administrato\\Desktop\\123.txt",
        GENERIC_ALL,
        FILE_SHARE_READ,
        NULL,
        OPEN_EXISTING,
        FILE_ATTRIBUTE_NORMAL,
        NULL
    );
    if (hFile == INVALID_HANDLE_VALUE)
    {
        cout << GetLastError() << endl;

        return 0;
    }
    
    char szBuffer[] = "Hello \r\nFerry";
    DWORD dwWrite = 0;
    WriteFile(hFile, szBuffer, sizeof(szBuffer), &dwWrite, NULL);

    DWORD dwFileSize = GetFileSize(hFile, NULL);
    PCHAR szBuff = new CHAR[dwFileSize];
    if (szBuff == NULL)
    {
        cout << "New Failed" << endl;
        CloseHandle(hFile);
        return 0;
    }
    memset(szBuff, 0, dwFileSize);

    DWORD dwRead = 0;
    ReadFile(hFile, szBuff, dwFileSize, &dwRead, NULL);

    CloseHandle(hFile);

    return 0;
}
```

# 三十二 文件遍历

文件遍历

```cpp
#include <iostream>
#include <windows.h>
#include <winerror.h>
#include <TlHelp32.h>

using std::cout;
using std::cin;
using std::endl;

int main()
{
    WIN32_FIND_DATAA wfd = { 0 };

    // 第一次返回句柄
    HANDLE hFile = FindFirstFileA("C:\\Users\\Administrator\\Desktop\\fsdownload\\*", &wfd);
    if (hFile == INVALID_HANDLE_VALUE)
    {
        cout << GetLastError() << endl;
        return 0;
    }

    BOOL bRet = FALSE;

    do 
    {
        if (wfd.dwFileAttributes & FILE_ATTRIBUTE_DIRECTORY)
        {
            // 目录
            if (wfd.dwFileAttributes & FILE_ATTRIBUTE_DIRECTORY | FILE_ATTRIBUTE_SYSTEM)
            {
                // 系统目录
            }
            else if (wfd.dwFileAttributes &
                FILE_ATTRIBUTE_DIRECTORY | FILE_ATTRIBUTE_SYSTEM | FILE_ATTRIBUTE_HIDDEN)
            {
                // 系统隐藏目录
            }

            if (strcmp(".", wfd.cFileName) != 0 && strcmp("..", wfd.cFileName) != 0)
            {
                cout << "目录 -> " << wfd.cFileName << endl;
            }
        }
        else
        {
            // 文件
            cout << "文件 -> " << wfd.cFileName << endl;
        }

        bRet = FindNextFileA(hFile, &wfd);// 第二次之后返回BOOL
    } while (bRet);

    FindClose(hFile);

    return 0;
}
```

# 三十三 文件目录常见API

文件目录常见API

```cpp
#include <iostream>
#include <windows.h>
#include <winerror.h>
#include <TlHelp32.h>

#include <Shlwapi.h>
#pragma comment(lib, "shlwapi.lib")

using std::cout;
using std::cin;
using std::endl;

int main()
{
    // 获取当前运行目录
    char szBuff[MAX_PATH] = { 0 };
    GetCurrentDirectoryA(MAX_PATH, szBuff);
    printf("%s \r\n", szBuff);

    // 判断指定的目录是否存在.需要shlwapi
    BOOL bRet = PathIsDirectoryA("D:\\Debug");
    cout << bRet << endl;

    // 判断指定的目录是否为空
    bRet = PathIsDirectoryEmptyA("D:\\WeGameApps");
    cout << bRet << endl;

    // 创建目录
    CreateDirectoryA(".\\Test", NULL);

    // 删除目录
    RemoveDirectoryA(".\\Test");

    // 判断文件是否存在
    bRet = PathFileExistsA("D:\\WeGameApps.exe");
    cout << bRet << endl;

    // 复制文件.覆盖
    CopyFileA("D:\\WeGameApps.exe", "D:\\NEW_WeGameApps.exe", TRUE);
    cout << bRet << endl;

    // 删除文件
    DeleteFileA("D:\\WeGameApps.exe");

    return 0;
}
```

# 三十四 内存属性信息

```cpp
    SYSTEM_INFO si = { 0 };
    GetSystemInfo(&si);
```

![14](./../../操作/7Win32/14.jpg)

# 三十五 VirtualAlloc

VirtualAlloc

```cpp
#include <iostream>
#include <windows.h>
#include <winerror.h>
#include <TlHelp32.h>

#include <Shlwapi.h>
#pragma comment(lib, "shlwapi.lib")

using std::cout;
using std::cin;
using std::endl;

int main()
{
    // 申请新内存，设置该内存的初始权限为读写
    LPVOID pBuffer = VirtualAlloc(NULL, 0x100, MEM_COMMIT, PAGE_READWRITE);
    if (!pBuffer)
    {
        cout << GetLastError() << endl;
        return 1;
    }
    memset(pBuffer, 0, 0x100);

    memcpy(pBuffer, "Ferry", sizeof("Ferry"));

    printf("%s \r\n", pBuffer);

    VirtualFree(pBuffer, 0x100, MEM_DECOMMIT);

    return 0;
}
```

# 三十六 内存属性

内存属性

![15](./../../操作/7Win32/15.jpg)

# 三十七 修改内存属性

修改内存属性

```cpp
const int a = 10;

int main()
{
    int* p = (int*) & a;

    DWORD dwOld1 = 0;
    VirtualProtect((LPVOID)&a, 4, PAGE_READWRITE, &dwOld1);

    *p = 0x12345678;

    DWORD dwOld2 = 0;
    VirtualProtect((LPVOID)&a, 4, dwOld1, &dwOld2);

    return 0;
}
```

# 三十八 跨进程内存操作

跨进程内存管理。VirtualAllocEx，VirtualFreeEx

```cpp
int main()
{
    HANDLE hProcess = OpenProcess(PROCESS_ALL_ACCESS, FALSE, 18348);
    if (hProcess == NULL)
    {
        cout << GetLastError() << endl;
        return 1;
    }

    LPVOID lpBuffer = VirtualAllocEx(hProcess, NULL, USN_PAGE_SIZE, MEM_COMMIT, PAGE_READWRITE);
    if (!lpBuffer)
    {
        return 2;
    }

    // 跨进程，不能用memset

    WriteProcessMemory(hProcess, lpBuffer, "Ferry", sizeof("Ferry"), NULL);

    VirtualFreeEx(hProcess, lpBuffer, sizeof("Ferry"), NULL);

    return 0;
}
```

# 三十九 内存地址，物理地址

内存地址，物理地址

![16](./../../操作/7Win32/16.jpg)

![17](./../../操作/7Win32/17.jpg) 

![18](./../../操作/7Win32/18.jpg)

# 四十 共享内存、物理页

共享内存

0地址不能访问，是因为0地址没有挂物理页

不同进程，相同的线性地址，挂的物理页不同

![19](./../../操作/7Win32/19.jpg)

![20](./../../操作/7Win32/20.jpg)

![21](./../../操作/7Win32/21.jpg)

两个进程挂同一个物理页，就叫共享内存

写拷贝

![22](./../../操作/7Win32/22.jpg)

操作系统也会用到KERNEL32.DLL，挂了一份物理页

如果把它HOOK了，还HOOK出问题了，重装系统就挂了

所以这种dll，会写拷贝，重新分配一个物理页
