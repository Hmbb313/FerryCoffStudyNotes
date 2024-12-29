# 一 消息队列

创建了一个窗口之后，会有一个消息队列，我们需要写一个消息处理函数，接收用户的消息，处理

只要进程没结束，消息队列会一直循环，持续接收消息

# 二 空项目写窗口程序

创建一个空项目，将链接器，子系统，改为WINDOWS

![1](./../../操作/8MFC/1.jpg)

第一个参数，就是IMAGE_BASE

# 三 简单完整的窗口程序

一个简单完整的窗口程序

```cpp
#include <iostream>
#include <windows.h>

LRESULT CALLBACK WndProc(HWND hWnd, UINT uMsg, WPARAM wParam, LPARAM lParam)
{
    switch (uMsg)
    {
    default:
        return DefWindowProc(hWnd, uMsg, wParam, lParam);
        break;
    }

    return 0;
}

int WINAPI WinMain(
    HINSTANCE hInstance,                    // 窗口实例句柄，就是IMAGE_BASE
    HINSTANCE hPrevInstance,                // 上一个窗口的实例句柄
    LPSTR lpCmdLine,                        // 命令行参数
    int nShowCmd                            // 窗口显示模式
) 
{
    // 设计窗口
    WNDCLASS wnd = { 0 };
    wnd.lpfnWndProc = WndProc;              // 窗口过程函数
    wnd.hInstance = hInstance;              // 窗口实例句柄
    wnd.hbrBackground = (HBRUSH)COLOR_MENU; // 窗口背景颜色
    wnd.lpszClassName = TEXT("MyWnd");      // 窗口类名

    // 注册窗口
    RegisterClass(&wnd);

    // 创建窗口
    HWND hWnd = CreateWindow(
        TEXT("MyWnd"),                      // 窗口类名
        TEXT("Ferry"),                      // 窗口标题
        WS_OVERLAPPEDWINDOW,                // 窗口样式
        CW_USEDEFAULT,                      // 默认位置
        CW_USEDEFAULT,                      // 默认位置
        800,                                // 窗口高度
        600,                                // 窗口宽度
        NULL,                               // 父窗口句柄
        NULL,                               // 菜单句柄
        hInstance,                          // 实例句柄
        NULL
    );

    // 显示窗口
    ShowWindow(hWnd, SW_SHOWNORMAL);

    // 更新窗口
    UpdateWindow(hWnd);

    // 消息处理
    MSG msg;// 消息结构体
    while (GetMessage(&msg, 0, 0, 0))
    {
        TranslateMessage(&msg); // 翻译消息
        DispatchMessage(&msg);  // 分发消息
    }

    return 0;
}
```

如果不实现WndProc，窗口创建就会失败，因为Windows窗口创建过程中会发送各种消息（如WM_CREATE等），即使在消息循环建立之前也会发送。

所以至少要DefWindowProc

CreateWindow不走消息队列，所以发的WM_CREATE消息可以直接到达窗口过程函数

# 四 处理消息

处理消息

```cpp
#include <iostream>
#include <windows.h>

LRESULT CALLBACK WndProc(HWND hWnd, UINT uMsg, WPARAM wParam, LPARAM lParam)
{
    switch (uMsg)
    {
    case WM_LBUTTONDOWN:
    {
        int x = LOWORD(lParam);
        int y = HIWORD(lParam);
        char szBuf[0x256] = { 0 };
        sprintf_s(szBuf, "x -> [%d], y -> [%d]", x, y);

        MessageBoxA(0, szBuf, 0, 0);
        break;
    }
    case WM_KEYDOWN:
    {
        if (wParam == 0x41)
        {
            MessageBoxA(0, "A", 0, 0);
        }
        break;
    }
    case WM_CLOSE:
    {
        PostQuitMessage(0);
        break;
    }
    default:
        return DefWindowProc(hWnd, uMsg, wParam, lParam);
        break;
    }

    return 0;
}

int WINAPI WinMain(
    HINSTANCE hInstance,                    // 窗口实例句柄，就是IMAGE_BASE
    HINSTANCE hPrevInstance,                // 上一个窗口的实例句柄
    LPSTR lpCmdLine,                        // 命令行参数
    int nShowCmd                            // 窗口显示模式
) 
{
    // 设计窗口
    WNDCLASS wnd = { 0 };
    wnd.lpfnWndProc = WndProc;              // 窗口过程函数
    wnd.hInstance = hInstance;              // 窗口实例句柄
    wnd.hbrBackground = (HBRUSH)COLOR_MENU; // 窗口背景颜色
    wnd.lpszClassName = TEXT("MyWnd");      // 窗口类名

    // 注册窗口
    RegisterClass(&wnd);

    // 创建窗口
    HWND hWnd = CreateWindow(
        TEXT("MyWnd"),                      // 窗口类名
        TEXT("Ferry"),                      // 窗口标题
        WS_OVERLAPPEDWINDOW,                // 窗口样式
        CW_USEDEFAULT,                      // 默认位置
        CW_USEDEFAULT,                      // 默认位置
        800,                                // 窗口高度
        600,                                // 窗口宽度
        NULL,                               // 父窗口句柄
        NULL,                               // 菜单句柄
        hInstance,                          // 实例句柄
        NULL
    );

    // 显示窗口
    ShowWindow(hWnd, SW_SHOWNORMAL);

    // 更新窗口
    UpdateWindow(hWnd);

    // 消息处理
    MSG msg;// 消息结构体
    while (GetMessage(&msg, 0, 0, 0))
    {
        TranslateMessage(&msg); // 翻译消息
        DispatchMessage(&msg);  // 分发消息
    }

    return 0;
}
```

# 五 控件

窗口控件

```CPP
#include <iostream>
#include <windows.h>

LRESULT CALLBACK WndProc(HWND hWnd, UINT uMsg, WPARAM wParam, LPARAM lParam)
{
    switch (uMsg)
    {
    case WM_LBUTTONDOWN:
    {
        int x = LOWORD(lParam);
        int y = HIWORD(lParam);
        char szBuf[0x256] = { 0 };
        sprintf_s(szBuf, "x -> [%d], y -> [%d]", x, y);

        MessageBoxA(0, szBuf, 0, 0);
        break;
    }
    case WM_KEYDOWN:
    {
        if (wParam == 0x41)
        {
            MessageBoxA(0, "A", 0, 0);
        }
        break;
    }
    case WM_CLOSE:
    {
        PostQuitMessage(0);
        break;
    }
    default:
        return DefWindowProc(hWnd, uMsg, wParam, lParam);
        break;
    }

    return 0;
}

int WINAPI WinMain(
    HINSTANCE hInstance,                    // 窗口实例句柄，就是IMAGE_BASE
    HINSTANCE hPrevInstance,                // 上一个窗口的实例句柄
    LPSTR lpCmdLine,                        // 命令行参数
    int nShowCmd                            // 窗口显示模式
) 
{
    // 设计窗口
    WNDCLASS wnd = { 0 };
    wnd.lpfnWndProc = WndProc;              // 窗口过程函数
    wnd.hInstance = hInstance;              // 窗口实例句柄
    wnd.hbrBackground = (HBRUSH)COLOR_MENU; // 窗口背景颜色
    wnd.lpszClassName = TEXT("MyWnd");      // 窗口类名

    // 注册窗口
    RegisterClass(&wnd);

    // 创建窗口
    HWND hWnd = CreateWindow(
        TEXT("MyWnd"),                      // 窗口类名
        TEXT("Ferry"),                      // 窗口标题
        WS_OVERLAPPEDWINDOW,                // 窗口样式
        CW_USEDEFAULT,                      // 默认位置
        CW_USEDEFAULT,                      // 默认位置
        800,                                // 窗口高度
        600,                                // 窗口宽度
        NULL,                               // 父窗口句柄
        NULL,                               // 菜单句柄
        hInstance,                          // 实例句柄
        NULL
    );

    // 显示窗口
    ShowWindow(hWnd, SW_SHOWNORMAL);

    // 更新窗口
    UpdateWindow(hWnd);

    // 创建控件
    CreateWindow(
        TEXT("BUTTON"),
        TEXT("按钮"),
        WS_CHILD | WS_VISIBLE | BS_PUSHBUTTON,
        10,
        10,
        80,
        20,
        hWnd,
        NULL,
        hInstance,
        NULL
    );

    CreateWindow(
        TEXT("BUTTON"),
        TEXT("复选框"),
        WS_CHILD | WS_VISIBLE | BS_CHECKBOX,
        10,
        50,
        80,
        20,
        hWnd,
        NULL,
        hInstance,
        NULL
    );

    CreateWindow(
        TEXT("BUTTON"),
        TEXT("单选框"),
        WS_CHILD | WS_VISIBLE | BS_RADIOBUTTON,
        10,
        90,
        80,
        20,
        hWnd,
        NULL,
        hInstance,
        NULL
    );

    // 消息处理
    MSG msg;// 消息结构体
    while (GetMessage(&msg, 0, 0, 0))
    {
        TranslateMessage(&msg); // 翻译消息
        DispatchMessage(&msg);  // 分发消息
    }

    return 0;
}
```

# 六 处理控件消息

控件交互

```cpp
#include <iostream>
#include <windows.h>

LRESULT CALLBACK WndProc(HWND hWnd, UINT uMsg, WPARAM wParam, LPARAM lParam)
{
    switch (uMsg)
    {
    case WM_COMMAND:
    {
        if (wParam == 1234)
        {
            MessageBoxA(0, "按钮", 0, 0);
        }
        if (wParam == 2345)
        {
            MessageBoxA(0, "复选框", 0, 0);
        }
        if (wParam == 3456)
        {
            MessageBoxA(0, "单选框", 0, 0);
        }
        break;
    }
    case WM_LBUTTONDOWN:
    {
        int x = LOWORD(lParam);
        int y = HIWORD(lParam);
        char szBuf[0x256] = { 0 };
        sprintf_s(szBuf, "x -> [%d], y -> [%d]", x, y);

        MessageBoxA(0, szBuf, 0, 0);
        break;
    }
    case WM_KEYDOWN:
    {
        if (wParam == 0x41)
        {
            MessageBoxA(0, "A", 0, 0);
        }
        break;
    }
    case WM_CLOSE:
    {
        PostQuitMessage(0);
        break;
    }
    default:
        return DefWindowProc(hWnd, uMsg, wParam, lParam);
        break;
    }

    return 0;
}

int WINAPI WinMain(
    HINSTANCE hInstance,                    // 窗口实例句柄，就是IMAGE_BASE
    HINSTANCE hPrevInstance,                // 上一个窗口的实例句柄
    LPSTR lpCmdLine,                        // 命令行参数
    int nShowCmd                            // 窗口显示模式
) 
{
    // 设计窗口
    WNDCLASS wnd = { 0 };
    wnd.lpfnWndProc = WndProc;              // 窗口过程函数
    wnd.hInstance = hInstance;              // 窗口实例句柄
    wnd.hbrBackground = (HBRUSH)COLOR_MENU; // 窗口背景颜色
    wnd.lpszClassName = TEXT("MyWnd");      // 窗口类名

    // 注册窗口
    RegisterClass(&wnd);

    // 创建窗口
    HWND hWnd = CreateWindow(
        TEXT("MyWnd"),                      // 窗口类名
        TEXT("Ferry"),                      // 窗口标题
        WS_OVERLAPPEDWINDOW,                // 窗口样式
        CW_USEDEFAULT,                      // 默认位置
        CW_USEDEFAULT,                      // 默认位置
        800,                                // 窗口高度
        600,                                // 窗口宽度
        NULL,                               // 父窗口句柄
        NULL,                               // 菜单句柄
        hInstance,                          // 实例句柄
        NULL
    );

    // 显示窗口
    ShowWindow(hWnd, SW_SHOWNORMAL);

    // 更新窗口
    UpdateWindow(hWnd);

    // 创建控件
    CreateWindow(
        TEXT("BUTTON"),
        TEXT("按钮"),
        WS_CHILD | WS_VISIBLE | BS_PUSHBUTTON,
        10,
        10,
        80,
        20,
        hWnd,
        HMENU(1234),
        hInstance,
        NULL
    );

    CreateWindow(
        TEXT("BUTTON"),
        TEXT("复选框"),
        WS_CHILD | WS_VISIBLE | BS_CHECKBOX,
        10,
        50,
        80,
        20,
        hWnd,
        HMENU(2345),
        hInstance,
        NULL
    );

    CreateWindow(
        TEXT("BUTTON"),
        TEXT("单选框"),
        WS_CHILD | WS_VISIBLE | BS_RADIOBUTTON,
        10,
        90,
        80,
        20,
        hWnd,
        HMENU(3456),
        hInstance,
        NULL
    );

    // 消息处理
    MSG msg;// 消息结构体
    while (GetMessage(&msg, 0, 0, 0))
    {
        TranslateMessage(&msg); // 翻译消息
        DispatchMessage(&msg);  // 分发消息
    }

    return 0;
}
```

# 七 Dialog

Dialog

```cpp
#include <iostream>
#include <windows.h>

#include "resource.h"

INT_PTR CALLBACK DlgProc(HWND hWnd, UINT uMsg, WPARAM wParam, LPARAM lParam)
{
    switch (uMsg)
    {
    default:
        return FALSE;
        break;
    }

    return 0;
}

int WINAPI WinMain(
    HINSTANCE hInstance,                    // 窗口实例句柄，就是IMAGE_BASE
    HINSTANCE hPrevInstance,                // 上一个窗口的实例句柄
    LPSTR lpCmdLine,                        // 命令行参数
    int nShowCmd                            // 窗口显示模式
) 
{
    DialogBox(hInstance, MAKEINTRESOURCE(IDD_DIALOG_MAIN), NULL, DlgProc);
    
    return 0;
}
```

# 八 处理对话框消息

处理对话框消息

```cpp
#include <iostream>
#include <windows.h>

#include "resource.h"

INT_PTR CALLBACK DlgProc(HWND hWnd, UINT uMsg, WPARAM wParam, LPARAM lParam)
{
    switch (uMsg)
    {
    case WM_LBUTTONDOWN:
    {
        int x = LOWORD(lParam);
        int y = HIWORD(lParam);
        char szBuffer[0x256] = { 0 };
        sprintf_s(szBuffer, "x -> %d \r\ny -> %d \r\n", x, y);
        OutputDebugStringA(szBuffer);

        break;
    }
    case WM_KEYDOWN:
    {
        char szBuffer[0x256] = { 0 };
        sprintf_s(szBuffer, "keyval -> %x \r\n", wParam);
        OutputDebugStringA(szBuffer);
        break;
    }
    case WM_INITDIALOG:
    {
        OutputDebugStringA("窗口初始化完毕\r\n");
        // MessageBoxA(0, "初始化完毕", 0, 0);
        break;
    }
    case WM_CLOSE:
    {
        EndDialog(hWnd, 0);
        break;
    }
    default:
        return FALSE;
        break;
    }

    return 0;
}

int WINAPI WinMain(
    HINSTANCE hInstance,                    // 窗口实例句柄，就是IMAGE_BASE
    HINSTANCE hPrevInstance,                // 上一个窗口的实例句柄
    LPSTR lpCmdLine,                        // 命令行参数
    int nShowCmd                            // 窗口显示模式
) 
{
    DialogBox(hInstance, MAKEINTRESOURCE(IDD_DIALOG_MAIN), NULL, DlgProc);
    
    return 0;
}
```

# 九 对话框程序操作控件

对话框程序操作控件

```cpp
#include <iostream>
#include <windows.h>

#include "resource.h"

#include <CommCtrl.h>
#pragma comment(lib, "comctl32.lib")

INT_PTR CALLBACK DlgProc(HWND hWnd, UINT uMsg, WPARAM wParam, LPARAM lParam)
{
    switch (uMsg)
    {
    // 通用控件

    // 标准控件
    case WM_COMMAND:
    {
        if (wParam == IDC_BUTTON1)
        {
            MessageBoxA(0, "IDC_BUTTON1", 0, 0);
        }
        if (wParam == IDC_LIST1)
        {
            MessageBoxA(0, "IDC_LIST1", 0, 0);
        }
        break;
    }
    case WM_LBUTTONDOWN:
    {
        int x = LOWORD(lParam);
        int y = HIWORD(lParam);
        char szBuffer[0x256] = { 0 };
        sprintf_s(szBuffer, "x -> %d \r\ny -> %d \r\n", x, y);
        OutputDebugStringA(szBuffer);

        break;
    }
    case WM_KEYDOWN:
    {
        char szBuffer[0x256] = { 0 };
        sprintf_s(szBuffer, "keyval -> %x \r\n", wParam);
        OutputDebugStringA(szBuffer);
        break;
    }
    case WM_INITDIALOG:
    {
        OutputDebugStringA("窗口初始化完毕\r\n");
        // MessageBoxA(0, "初始化完毕", 0, 0);
        break;
    }
    case WM_CLOSE:
    {
        EndDialog(hWnd, 0);
        break;
    }
    default:
        return FALSE;
        break;
    }

    return 0;
}

int WINAPI WinMain(
    HINSTANCE hInstance,                    // 窗口实例句柄，就是IMAGE_BASE
    HINSTANCE hPrevInstance,                // 上一个窗口的实例句柄
    LPSTR lpCmdLine,                        // 命令行参数
    int nShowCmd                            // 窗口显示模式
) 
{
    // 通用控件
    INITCOMMONCONTROLSEX iccl = { 0 };
    iccl.dwSize = sizeof(INITCOMMONCONTROLSEX);
    iccl.dwICC = ICC_WIN95_CLASSES;
    InitCommonControlsEx(&iccl);

    DialogBox(hInstance, MAKEINTRESOURCE(IDD_DIALOG_MAIN), NULL, DlgProc);
    
    return 0;
}
```

# 十 遍历窗口/控件

遍历窗口/控件。控件就是窗口

```cpp
#include <iostream>
#include <windows.h>

HWND hWndPid = 0;

// 遍历窗口回调函数
BOOL CALLBACK EnumWindowProc(HWND hwnd, LPARAM lParam)
{
    // 获取窗口类名
    char szClass[0x256] = { 0 };
    GetClassNameA(hwnd, szClass, 0x256);

    // 获取窗口标题
    char szTitle[0x256] = { 0 };
    GetWindowTextA(hwnd, szTitle, 0x256);

    // printf("Class -> [%s] Title -> [%s] \r\n", szClass, szTitle);

    if (strcmp(szClass, "#32770") == 0 || strcmp(szTitle, "PEiD v0.95") == 0)
    {
        hWndPid = hwnd;
    }

    return TRUE;
}

BOOL CALLBACK EnumChildProc(HWND hwnd, LPARAM lParam)
{
    // 获取窗口类名
    char szClass[0x256] = { 0 };
    GetClassNameA(hwnd, szClass, 0x256);

    // 获取窗口标题
    if (strcmp(szClass, "Button") == 0 || strcmp(szClass, "Static") == 0 
        || strcmp(szClass, "Edit") == 0)
    {
        char szTitle[0x256] = { 0 };
        GetWindowTextA(hwnd, szTitle, 0x256);

        printf("Hwnd -> [%s] Class -> [%s] Title -> [%s] \r\n", hwnd, szClass, szTitle);
    }

    return TRUE;
}

int main()
{
    // 遍历当前窗口
    EnumWindows(EnumWindowProc, 0);

    // 遍历子窗口
    EnumChildWindows(hWndPid, EnumChildProc, NULL);

    return 0;
}
```

# 十一 窗口句柄和进程ID

窗口句柄和进程ID

```cpp
#include <iostream>
#include <windows.h>
#include <Psapi.h>
#include <TlHelp32.h>

HWND hFindHwnd = 0;

// 通过窗口句柄查找进程信息
VOID FindProcessInfoByHandle(HWND hwnd)
{
    DWORD dwPid = 0;
    WCHAR szProcessPath[MAX_PATH] = { 0 };
    HANDLE hSnap = 0;
    PROCESSENTRY32 pe32 = { 0 };
    pe32.dwSize = sizeof(pe32);

    // 通过窗口句柄查找进程ID
    GetWindowThreadProcessId(hwnd, &dwPid);

    // 通过进程句柄获取进程路径
    GetModuleFileNameEx(
        OpenProcess(PROCESS_ALL_ACCESS, FALSE, dwPid),
        NULL,
        szProcessPath,
        MAX_PATH
    );

    // 遍历进程
    hSnap = CreateToolhelp32Snapshot(TH32CS_SNAPPROCESS, 0);
    if (hSnap == INVALID_HANDLE_VALUE)
    {
        return;
    }

    BOOL bRet = Process32First(hSnap, &pe32);
    while(bRet)
    {
        if (pe32.th32ProcessID == dwPid)
        {
            printf("%x\r\n", pe32.th32ProcessID);
        }

        bRet = Process32Next(hSnap, &pe32);
    }
}

// 通过进程信息查找窗口句柄
// 遍历进程回调函数
BOOL CALLBACK EnumWindowProc(HWND hWnd, LPARAM lParam)
{
    DWORD dwPid = 0;

    // 通过窗口句柄，得到进程PID
    GetWindowThreadProcessId(hWnd, &dwPid);

    // 判断是否是指定进程PID
    if (dwPid == 6752)
    {
        hFindHwnd = hWnd;
        return FALSE;
    }

    return TRUE;
}

int main()
{
    // 通过窗口类名和窗口标题查找对应窗口句柄
    HWND hParentHwnd = FindWindow(TEXT("#32770"), TEXT("PEiD v0.95"));

    // 通过父窗口句柄查找子窗口句柄
    HWND hButtonHwnd = FindWindowEx(
        hParentHwnd, NULL, TEXT("Button"), TEXT("浏览"));

    // 通过窗口句柄查找进程信息
    FindProcessInfoByHandle(hParentHwnd);

    // 通过进程信息查找窗口句柄
    EnumWindows(EnumWindowProc, NULL);

    return 0;
}
```

# 十二 窗口常见操作

窗口常见操作

```cpp
#include <iostream>
#include <windows.h>

int main()
{
    // 查找窗口句柄。通过窗口类名和窗口标题
    HWND hwnd = FindWindow(TEXT("#32770"), TEXT("PEiD v0.95"));

    RECT rect = { 0 };
    // 获取窗口大小
    GetWindowRect(hwnd, &rect);

    // 移动窗口
    MoveWindow(hwnd, 100, 100, 
        rect.right - rect.left, rect.bottom - rect.top, FALSE);

    // 最小化窗口
    CloseWindow(hwnd);

    // 设置窗口最前
    SetForegroundWindow(hwnd);
    SetWindowPos(hwnd, HWND_TOPMOST, 0, 0, 0, 0, SWP_NOMOVE | SWP_NOSIZE);

    // 关闭窗口
    PostMessage(hwnd, WM_CLOSE, 0, 0);

    return 0;
}
```

# 十三 MFC标准的框架

MFC标准的框架

MFC.h

```cpp
#pragma once

// MFC头文件
#include <afxwin.h>

// 应用程序类
class MyApp :public CWinApp
{
public:
    // 入口函数
    virtual BOOL InitInstance();

};

// 框架类
class MyFrame :public CFrameWnd 
{
public:
    // 构造函数
    MyFrame();
};
```

MFC.cpp

```cpp
#include "MFC.h"

// 全局应用程序类对象。有且只有一个
MyApp myApp;

BOOL MyApp::InitInstance()
{
    // 创建框架类类对象
    MyFrame* myFrame = new MyFrame;

    // 显示窗口
    myFrame->ShowWindow(SW_SHOWNORMAL);

    // 更新窗口
    myFrame->UpdateWindow();

    // 保存框架类对象的指针
    m_pMainWnd = myFrame;

    return TRUE;
}

MyFrame::MyFrame()
{
    // 就是创建一个窗口。窗口的参数有默认参
    Create(NULL, TEXT("Ferry"));
}

```

# 十四 MFC消息处理

消息处理

![2](./../../操作/8MFC/2.jpg)

MFC.h

```cpp
#pragma once

// MFC头文件
#include <afxwin.h>

// 应用程序类
class MyApp :public CWinApp
{
public:
    // 入口函数
    virtual BOOL InitInstance();

};

// 框架类
class MyFrame :public CFrameWnd 
{
public:
    // 构造函数
    MyFrame();

    // 声明消息映射
    // 在MyFrame类处理消息，因为是MyFrame类创建的窗口
    DECLARE_MESSAGE_MAP()

    afx_msg void OnLButtonDown(UINT, CPoint);
};
```

MFC.cpp

```cpp
#include "MFC.h"

// 全局应用程序类对象。有且只有一个
MyApp myApp;

BOOL MyApp::InitInstance()
{
    // 创建框架类类对象
    MyFrame* myFrame = new MyFrame;

    // 显示窗口
    myFrame->ShowWindow(SW_SHOWNORMAL);

    // 更新窗口
    myFrame->UpdateWindow();

    // 保存框架类对象的指针
    m_pMainWnd = myFrame;

    return TRUE;
}

// 消息处理
BEGIN_MESSAGE_MAP(MyFrame, CFrameWnd)
    ON_WM_LBUTTONDOWN()

END_MESSAGE_MAP()

MyFrame::MyFrame()
{
    // 就是创建一个窗口。窗口的参数有默认参
    Create(NULL, TEXT("Ferry"));
}

void MyFrame::OnLButtonDown(UINT, CPoint)
{
    OutputDebugString(TEXT("左键按下")); 
}

```

# 十五 编译器自动生成的MFC项目

介绍一下编译器自动生成的MFC项目

# 十六 CMyDialog

拖一个BUTTON到默认的Dialog里面，双击BUTTON生成消息处理函数

消息处理函数里面，CMyDialog创建对象，DoModal



再添加一个Dialog

将Dialog绑定一个类CMyDialog

给这个类写一个OnInitDialog()函数

![3](./../../操作/8MFC/3.jpg)

![4](./../../操作/8MFC/4.jpg)

![5](./../../操作/8MFC/5.jpg)

![6](./../../操作/8MFC/6.jpg)

# 十七 模态和非模态对话框

模态和非模态对话框

弹出了模态对话框之后，无法点击父窗口

弹出了非模态对话框之后，可以点击父窗口



添加两个Dialog，分别绑定一个类

![7](./../../操作/8MFC/7.jpg)

![8](./../../操作/8MFC/8.jpg)

![9](./../../操作/8MFC/9.jpg)

# 十八 默认退出当前消息处理

默认退出当前消息处理

![10](./../../操作/8MFC/10.jpg)

![11](./../../操作/8MFC/11.jpg)

return TRUE表示消息已经被处理。退出当前消息处理函数，去做其他处理

# 十九 字符处理

字符处理

```cpp
void CMFCDlg::OnBnClickedButton1()
{
	// TODO: 在此添加控件通知处理程序代码

	// ::指定使用win32的API
	::MessageBoxA(0, "Ferry-A", 0, 0);
	::MessageBoxW(0, L"Ferry-B", 0, 0);

	// 使用MFC再次封装的API
	MessageBox(TEXT("Ferry-C"));

	// MFC纯正的字符处理
	AfxMessageBox(TEXT("Ferry-D"));

	// MFC提供的字符串类
	CString str(TEXT("123"));

	// 初始化。调拷贝构造
	CString str1 = str;

	// 初始化10个字符A
	CString str3(L'A', 10);

	// 格式化字符串
	CString str4;
	str4.Format(_T("%s %d"), _T("Ferry"), 666);

	// 获取字符长度
	int nLength = str4.GetLength();

	// 是否为空
	BOOL bRet = str4.IsEmpty();
}
```

# 二十 文件对话框

文件对话框

```cpp
void CMFCDlg::OnBnClickedButton2()
{
	// TODO: 在此添加控件通知处理程序代码

	// 创建文件对话框
	// CFileDialog FileDialog(TRUE, NULL, NULL, NULL, NULL, this);

    // 过滤器
	CFileDialog FileDialog(TRUE, NULL, NULL, NULL, 
		TEXT("文本文件|*.txt|应用程序|*.exe|所有文件|*.*||"), this);

	// 显示文件对话框
	if (FileDialog.DoModal() == IDCANCEL)
	{
		return;
	}

	// 获取打开文件完整路径
	CString csFilePath = FileDialog.GetPathName();

	// 获取打开文件名称
	CString csFileName = FileDialog.GetFileName();

	// 获取打开文件后缀
	CString csFileExt = FileDialog.GetFileExt();
}
```

# 二十一 MFC的文件处理

MFC的文件处理

CFILE

```cpp
void CMFCDlg::OnBnClickedButton2()
{
	// TODO: 在此添加控件通知处理程序代码

	// 创建文件对话框
	// CFileDialog FileDialog(TRUE, NULL, NULL, NULL, NULL, this);

	// 过滤器
	CFileDialog FileDialog(TRUE, NULL, NULL, NULL, 
		TEXT("文本文件|*.txt|应用程序|*.exe|所有文件|*.*||"), this);

	// 显示文件对话框
	if (FileDialog.DoModal() == IDCANCEL)
	{
		return;
	}

	// 获取打开文件完整路径
	CString csFilePath = FileDialog.GetPathName();

	// 获取打开文件名称
	CString csFileName = FileDialog.GetFileName();

	// 获取打开文件后缀
	CString csFileExt = FileDialog.GetFileExt();

	// 打开文件
	CFile cFile;
	cFile.Open(csFilePath, CFile::modeReadWrite);

	// 获取文件长度
	ULONGLONG ullFileLenth = cFile.GetLength();

	// 申请内存
	CHAR* szFileBuffer = new CHAR[ullFileLenth];

	// 读取文件数据
	cFile.Read(szFileBuffer, ullFileLenth);

    // 清空文件数据
    cFile.SetLength(0);

	// 修改文件数据
	cFile.Write(L"Ferry666", sizeof("Ferry666"));

	// 释放内存
	delete[] szFileBuffer;

	// 关闭文件
	cFile.Close();
}
```

# 二十二 控件绑定创建消息处理

Button

绑定创建消息处理

![12](./../../操作/8MFC/12.jpg)

![13](./../../操作/8MFC/13.jpg)

绑定创建类对象

![14](./../../操作/8MFC/14.jpg)

![15](./../../操作/8MFC/15.jpg)

![16](./../../操作/8MFC/16.jpg)

```cpp
BOOL CMFCDlg::OnInitDialog()
{
	CDialogEx::OnInitDialog();

	// 设置此对话框的图标。  当应用程序主窗口不是对话框时，框架将自动
	//  执行此操作
	SetIcon(m_hIcon, TRUE);			// 设置大图标
	SetIcon(m_hIcon, FALSE);		// 设置小图标

	// TODO: 在此添加额外的初始化代码

	// 获取按钮标题
	CString csButtonText;
	m_Button_Test.GetWindowText(csButtonText);
	AfxMessageBox(csButtonText);

	// 设置按钮标题
	m_Button_Test.SetWindowText(TEXT("Ferry"));

	// 设置按钮禁用
	m_Button_Test.EnableWindow(FALSE);

	return TRUE;  // 除非将焦点设置到控件，否则返回 TRUE
}
```

# 二十三 RadioBox

GroupBox

RadioBox，单选框

![17](./../../操作/8MFC/17.jpg)

CTRL+D显示控件序号

![18](./../../操作/8MFC/18.jpg)

单人的组设置为TRUE

![19](./../../操作/8MFC/19.jpg)

![20](./../../操作/8MFC/20.jpg)

```cpp
BOOL CMFCDlg::OnInitDialog()
{
	CDialogEx::OnInitDialog();

	// 设置此对话框的图标。  当应用程序主窗口不是对话框时，框架将自动
	//  执行此操作
	SetIcon(m_hIcon, TRUE);			// 设置大图标
	SetIcon(m_hIcon, FALSE);		// 设置小图标

	// TODO: 在此添加额外的初始化代码

	// 获取按钮标题
	CString csButtonText;
	m_Button_Test.GetWindowText(csButtonText);
	// AfxMessageBox(csButtonText);

	// 设置按钮标题
	m_Button_Test.SetWindowText(TEXT("Ferry"));

	// 设置按钮禁用
	m_Button_Test.EnableWindow(FALSE);

	// 初始化单选框.1和2为一组，3和4为一组。默认选中1和3
	CheckRadioButton(IDC_RADIO1, IDC_RADIO2, IDC_RADIO1);
	CheckRadioButton(IDC_RADIO3, IDC_RADIO4, IDC_RADIO3);

	return TRUE;  // 除非将焦点设置到控件，否则返回 TRUE
}
```

```cpp
void CMFCDlg::OnBnClickedButton3()
{
	// TODO: 在此添加控件通知处理程序代码
	::MessageBoxA(0, "OnBnClickedButton3", 0, 0);
}


void CMFCDlg::OnBnClickedButton4()
{
	// TODO: 在此添加控件通知处理程序代码

	if (IsDlgButtonChecked(IDC_RADIO1))
	{
		AfxMessageBox(TEXT("单人"));
	}
    if (IsDlgButtonChecked(IDC_RADIO2))
    {
        AfxMessageBox(TEXT("组队"));
    }
    if (IsDlgButtonChecked(IDC_RADIO3))
    {
        AfxMessageBox(TEXT("定点"));
    }
    if (IsDlgButtonChecked(IDC_RADIO4))
    {
        AfxMessageBox(TEXT("随机"));
    }
}
```

# 二十四 CheckBox

CheckBox，选择框

![21](./../../操作/8MFC/21.jpg)

```CPP
void CMFCDlg::OnBnClickedButton5()
{
	// TODO: 在此添加控件通知处理程序代码

	CString str;
	if (IsDlgButtonChecked(IDC_CHECK1))
	{
		str += "主线";
	}
    if (IsDlgButtonChecked(IDC_CHECK2))
    {
        str += "师门";
    }
    if (IsDlgButtonChecked(IDC_CHECK3))
    {
        str += "帮派";
    }

	AfxMessageBox(str);
}
```

绑定成员变量

![22](./../../操作/8MFC/22.jpg)

```CPP
void CMFCDlg::OnBnClickedButton6()
{
	// TODO: 在此添加控件通知处理程序代码

	CString str;
    // 师门选中
	if (m_CheckSm.GetCheck() == 1)
	{
		AfxMessageBox(TEXT("师门"));
	}
}
```

单机选择框，立马做出反应

![23](./../../操作/8MFC/23.jpg)

```cpp
void CMFCDlg::OnBnClickedCheck4()
{
	// TODO: 在此添加控件通知处理程序代码

	if (IsDlgButtonChecked(IDC_CHECK4))
	{
		AfxMessageBox(TEXT("处理"));
	}
	else {
		AfxMessageBox(TEXT("待机"));
	}
}
```

# 二十五 StaticText

StaticText，静态文本框

```cpp
	// 初始化静态文本框。修改内容
	// GetDlgItem(IDC_STATIC_T1)->SetWindowText(TEXT("Kernel"));
	m_Static_T1.SetWindowText(TEXT("Kernel"));
```

![24](./../../操作/8MFC/24.jpg)

```cpp
BOOL CMFCDlg::OnInitDialog()
{
	CDialogEx::OnInitDialog();

	// 设置此对话框的图标。  当应用程序主窗口不是对话框时，框架将自动
	//  执行此操作
	SetIcon(m_hIcon, TRUE);			// 设置大图标
	SetIcon(m_hIcon, FALSE);		// 设置小图标

	// TODO: 在此添加额外的初始化代码

	// 初始化字体
	pFont = new CFont;
	pFont->CreatePointFont(200, TEXT("黑体"));

	// 获取按钮标题
	CString csButtonText;
	m_Button_Test.GetWindowText(csButtonText);
	// AfxMessageBox(csButtonText);

	// 设置按钮标题
	m_Button_Test.SetWindowText(TEXT("Ferry"));

	// 设置按钮禁用
	m_Button_Test.EnableWindow(FALSE);

	// 初始化单选框.1和2为一组，3和4为一组。默认选中1和3
	CheckRadioButton(IDC_RADIO1, IDC_RADIO2, IDC_RADIO1);
	CheckRadioButton(IDC_RADIO3, IDC_RADIO4, IDC_RADIO3);

	// 初始化静态文本框。修改内容
	// GetDlgItem(IDC_STATIC_T1)->SetWindowText(TEXT("Kernel"));
	m_Static_T1.SetWindowText(TEXT("Hello"));
	m_Static_T1.SetFont(pFont);

	return TRUE;  // 除非将焦点设置到控件，否则返回 TRUE
}
```

# 二十六 EditCtrl

EditCtrl，编辑框

```cpp
void CMFCDlg::OnBnClickedButton7()
{
	// TODO: 在此添加控件通知处理程序代码

	// 绑定类对象的编辑框
	CString str;
	m_Edit.GetWindowText(str);
	AfxMessageBox(str);

	str = TEXT("Hello Ferry");
	m_Edit.SetWindowText(str);
}


void CMFCDlg::OnBnClickedButton8()
{
	// TODO: 在此添加控件通知处理程序代码

	// 绑定CString的编辑框
	UpdateData(TRUE);
	CString str;
	str = m_csEdit2;
	AfxMessageBox(str);

	m_csEdit2 = "12345";
	UpdateData(FALSE);
}
```

编辑框自动换行

![25](./../../操作/8MFC/25.jpg)

# 二十七 ListCtrl

ListCtrl，列表框

![26](./../../操作/8MFC/26.jpg)

![27](./../../操作/8MFC/27.jpg)

```cpp
BOOL CMFCDlg::OnInitDialog()
{
	CDialogEx::OnInitDialog();

	// 设置此对话框的图标。  当应用程序主窗口不是对话框时，框架将自动
	//  执行此操作
	SetIcon(m_hIcon, TRUE);			// 设置大图标
	SetIcon(m_hIcon, FALSE);		// 设置小图标

	// TODO: 在此添加额外的初始化代码

	// 初始化字体
	pFont = new CFont;
	pFont->CreatePointFont(200, TEXT("黑体"));

	// 获取按钮标题
	CString csButtonText;
	m_Button_Test.GetWindowText(csButtonText);
	// AfxMessageBox(csButtonText);

	// 设置按钮标题
	m_Button_Test.SetWindowText(TEXT("Ferry"));

	// 设置按钮禁用
	m_Button_Test.EnableWindow(FALSE);

	// 初始化单选框.1和2为一组，3和4为一组。默认选中1和3
	CheckRadioButton(IDC_RADIO1, IDC_RADIO2, IDC_RADIO1);
	CheckRadioButton(IDC_RADIO3, IDC_RADIO4, IDC_RADIO3);

	// 初始化静态文本框。修改内容
	// GetDlgItem(IDC_STATIC_T1)->SetWindowText(TEXT("Kernel"));
	m_Static_T1.SetWindowText(TEXT("Hello"));
	m_Static_T1.SetFont(pFont);

	// 初始化列表框
	m_List.InsertColumn(0, TEXT("序号"), LVCFMT_LEFT, 100);
	m_List.InsertColumn(1, TEXT("进程"), LVCFMT_LEFT, 100);
	m_List.InsertColumn(2, TEXT("PID"), LVCFMT_LEFT, 100);

	DWORD dwStyle = m_List.GetStyle();
	// LVS_EX_GRIDLINES表格线
	// LVS_EX_FULLROWSELECT整行选中
	m_List.SetExtendedStyle(dwStyle | LVS_EX_GRIDLINES | LVS_EX_FULLROWSELECT);

	return TRUE;  // 除非将焦点设置到控件，否则返回 TRUE
}


void CMFCDlg::OnBnClickedButton9()
{
	// TODO: 在此添加控件通知处理程序代码

	// 进程快照
	HANDLE hSnap = CreateToolhelp32Snapshot(TH32CS_SNAPPROCESS, 0);
	if (hSnap == INVALID_HANDLE_VALUE)
	{
		return;
	}
	PROCESSENTRY32 pe32 = { 0 };
	pe32.dwSize = sizeof(pe32);

	// 遍历进程
	if (Process32First(hSnap, &pe32))
	{
		do 
		{
			// 获取当前列表框表项数
			DWORD dwIndex = m_List.GetItemCount();

			CString str;
			str.Format(TEXT("%d"), dwIndex);

			// 插入表项
			m_List.InsertItem(dwIndex, str);

			str.Format(TEXT("%s"), pe32.szExeFile);
			m_List.SetItemText(dwIndex, 1, str);

			str.Format(TEXT("%d"), pe32.th32ParentProcessID);
			m_List.SetItemText(dwIndex, 2, str);

		} while (Process32Next(hSnap, &pe32));
	}
}


void CMFCDlg::OnNMDblclkList1(NMHDR* pNMHDR, LRESULT* pResult)
{
	LPNMITEMACTIVATE pNMItemActivate = reinterpret_cast<LPNMITEMACTIVATE>(pNMHDR);
	// TODO: 在此添加控件通知处理程序代码
	*pResult = 0;

	CString str;
	str.Format(TEXT("行%d - 列%d"), 
		pNMItemActivate->iItem, pNMItemActivate->iSubItem);
	AfxMessageBox(str);
}
```

# 二十八 ComboBox

ComboBox，下拉框

![28](./../../操作/8MFC/28.jpg)

```cpp
BOOL CMFCDlg::OnInitDialog()
{
	CDialogEx::OnInitDialog();

	// 设置此对话框的图标。  当应用程序主窗口不是对话框时，框架将自动
	//  执行此操作
	SetIcon(m_hIcon, TRUE);			// 设置大图标
	SetIcon(m_hIcon, FALSE);		// 设置小图标

	// TODO: 在此添加额外的初始化代码

	// 初始化下拉列表
	m_ComboBox.AddString(TEXT("123.ini"));
	m_ComboBox.AddString(TEXT("456.ini"));
	m_ComboBox.AddString(TEXT("789.ini"));

	// 默认下拉列表选项
	m_ComboBox.SetCurSel(0);

	return TRUE;  // 除非将焦点设置到控件，否则返回 TRUE
}

void CMFCDlg::OnBnClickedButton10()
{
	// TODO: 在此添加控件通知处理程序代码

	int nSel = m_ComboBox.GetCurSel();

	CString str;
	m_ComboBox.GetLBText(nSel, str);
	AfxMessageBox(str);
}
```

# 二十九 TabCtrl

TabCtrl，选择夹

![29](./../../操作/8MFC/29.jpg)

Dialog需要添加一个类

![30](./../../操作/8MFC/30.jpg)

![31](./../../操作/8MFC/31.jpg)

![32](./../../操作/8MFC/32.jpg)

```cpp
BOOL CMFCDlg::OnInitDialog()
{
	CDialogEx::OnInitDialog();

	// 设置此对话框的图标。  当应用程序主窗口不是对话框时，框架将自动
	//  执行此操作
	SetIcon(m_hIcon, TRUE);			// 设置大图标
	SetIcon(m_hIcon, FALSE);		// 设置小图标

	// TODO: 在此添加额外的初始化代码

	// 初始化选择夹
	m_TabCtrl.InsertItem(0, TEXT("账号信息"));
	m_TabCtrl.InsertItem(1, TEXT("任务信息"));

	// 创建页面。非模态对话框
	m_Page1.Create(IDD_DIALOG1, &m_TabCtrl);
	m_Page2.Create(IDD_DIALOG2, &m_TabCtrl);

    // 修正位置
	CRect rect;
	m_TabCtrl.GetClientRect(&rect);

    // tabctrl放大一点
	rect.top += 40;

	m_Page1.MoveWindow(&rect);
	m_Page2.MoveWindow(&rect);

	// 显示页面
	m_TabCtrl.SetCurSel(0);
	m_Page1.ShowWindow(TRUE);
	m_Page2.ShowWindow(FALSE);

	return TRUE;  // 除非将焦点设置到控件，否则返回 TRUE
}

void CMFCDlg::OnTcnSelchangeTab1(NMHDR* pNMHDR, LRESULT* pResult)
{
	// TODO: 在此添加控件通知处理程序代码
	*pResult = 0;

	int nSel = m_TabCtrl.GetCurSel();
	switch (nSel)
	{
	case 0:
	{
		m_Page1.ShowWindow(TRUE);
		m_Page2.ShowWindow(FALSE);
		break;
	}
    case 1:
    {
		m_Page1.ShowWindow(FALSE);
        m_Page2.ShowWindow(TRUE);
        break;
    }
	default:
		break;
	}
}
```



如果是PAGE页里的控件，在PAGE类里初始化

# 三十 Menu

Menu，菜单

![33](./../../操作/8MFC/33.jpg)

![34](./../../操作/8MFC/34.jpg)

![35](./../../操作/8MFC/35.jpg)

![36](./../../操作/8MFC/36.jpg)

```cpp
void CMFCDlg::On32771()
{
	// TODO: 在此添加命令处理程序代码
	AfxMessageBox(TEXT("打开文件"));
}


void CMFCDlg::On32778()
{
	// TODO: 在此添加命令处理程序代码
	AfxMessageBox(TEXT("DLL隐藏"));
}


void CMFCDlg::OnNMRClickList1(NMHDR* pNMHDR, LRESULT* pResult)
{
	LPNMITEMACTIVATE pNMItemActivate = reinterpret_cast<LPNMITEMACTIVATE>(pNMHDR);
	// TODO: 在此添加控件通知处理程序代码
	*pResult = 0;

	// 加载菜单
	HMENU hMenu = LoadMenu(AfxGetApp()->m_hInstance, MAKEINTRESOURCE(IDR_MENU1));

	// 获取菜单。真正的菜单句柄
	hMenu = GetSubMenu(hMenu, 0);

	POINT pt;
	GetCursorPos(&pt);
	// 弹出这个子菜单
	TrackPopupMenu(hMenu, TPM_CENTERALIGN, pt.x, pt.y, 0, m_hWnd, NULL);
}
```
