### SetForegroundWindow

There are several implementations which bring a window to foreground, but they are not very reliable and 
they don't work in case if application which in foreground has higher integrity.

Here is described implementattion which always work no matter what integrity of an application which is in foreground.


```C++
void SetForeground(HWND hWnd)
{
    AllocConsole();
    auto hWndConsole = GetConsoleWindow();
    SetWindowPos(hWndConsole, 0, 0, 0, 0, 0, SWP_NOACTIVATE);
    FreeConsole();

    SetForegroundWindow(hWnd);
}
```

To test: run program bellow and within 5sec start `Task Manager`. The program will be in foregorund.

```C++
#include "stdafx.h"
#include <windows.h>

void SetForeground(HWND hWnd)
{
    AllocConsole();
    auto hWndConsole = GetConsoleWindow();
    SetWindowPos(hWndConsole, 0, 0, 0, 0, 0, SWP_NOACTIVATE);
    FreeConsole();

    SetForegroundWindow(hWnd);
}

int APIENTRY wWinMain(HINSTANCE hInstance, HINSTANCE hPrevInstance, LPWSTR lpCmdLine, int nCmdShow)
{
    Sleep(5000);

    WNDCLASSEXW wcex = {};
    wcex.cbSize = sizeof(WNDCLASSEX);
    wcex.lpfnWndProc = [](HWND hWnd, UINT message, WPARAM wParam, LPARAM lParam) 
    { 
        if (WM_DESTROY == message)
        {
            PostQuitMessage(0);
        }

        return DefWindowProc(hWnd, message, wParam, lParam); 
    };
    wcex.hInstance = hInstance;
    wcex.lpszClassName = L"TestSetForegroundWindow";
    RegisterClassExW(&wcex);

    HWND hWnd = CreateWindowW(wcex.lpszClassName, L"", WS_OVERLAPPEDWINDOW, 0, 0, 500, 500, 0, 0, hInstance, 0);
    ShowWindow(hWnd, SW_SHOW);

    SetForeground(hWnd);

    MSG msg;
    while (GetMessage(&msg, 0, 0, 0))
    {
        TranslateMessage(&msg);
        DispatchMessage(&msg);
    }

    return 0;
}
```
