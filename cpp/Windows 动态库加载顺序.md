## Windows 动态库加载顺序

1. 应用程序的加载目录：D:\Test

2. 当前目录（默认为程序加载目录，可以通过SetCurrentDirectory修改，通过GetCurrentDirectory获取）

3. 系统目录（32位系统下通常是，C:\Windows\System32，可以通过GetSystemDirectory获取）

4. 16位系统目录（忽略）

5. Windows目录（通常是，C:\Windows，可以通过GetWindowsDirectory获取）

6. PATH环境变量中列出的所有路径  

