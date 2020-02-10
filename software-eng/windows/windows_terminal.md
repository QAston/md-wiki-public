# windows terminal

- <https://github.com/microsoft/terminal>
- <https://www.microsoft.com/en-us/p/windows-terminal-preview/9n0dx20hk701?activetab=pivot:overviewtab>

```
        {
            "acrylicOpacity" : 0.5,
            "closeOnExit" : true,
            "colorScheme" : "Campbell",
            "commandline" : "c:\\portable\\cygwin\\Cygwin.bat",
            "cursorColor" : "#FFFFFF",
            "cursorShape" : "bar",
            "fontFace" : "Consolas",
            "fontSize" : 10,
            "guid" : "{153622ff-13d4-4d8e-97fa-fad279fd50af}",
            "historySize" : 9001,
            "icon" : "c:\\portable\\cygwin\\cygwin.ico",
            "name" : "Cygwin bash",
            "padding" : "0, 0, 0, 0",
            "snapOnInput" : true,
            "useAcrylic" : false
        },
        {
            "acrylicOpacity" : 0.5,
            "closeOnExit" : true,
            "colorScheme" : "Campbell",
            "commandline" : "c:\\portable\\msys\\start_msys2.bat --login  -i",
            "cursorColor" : "#FFFFFF",
            "cursorShape" : "bar",
            "fontFace" : "Consolas",
            "fontSize" : 10,
            "guid" : "{79bde085-5e55-4eaa-b3b1-940b320e2d3b}",
            "historySize" : 9001,
            "icon" : "c:\\portable\\msys\\msys2.ico",
            "name" : "Msys2 bash",
            "padding" : "0, 0, 0, 0",
            "snapOnInput" : true,
            "useAcrylic" : false
        },
```
msys2 bat:
```
@echo off

rem To activate windows native symlinks uncomment next line
set MSYS=winsymlinks:nativestrict

rem Set debugging program for errors
rem set MSYS=error_start:%WD%../../mingw32/bin/qtcreator.exe^|-debug^|^<process-id^>

rem To export full current PATH from environment into MSYS2 uncomment next line
set MSYS2_PATH_TYPE=inherit

set MSYS_HOME=C:\portable\msys
set PATH=%MSYS_HOME%/usr/local/bin;%MSYS_HOME%/usr/bin;%MSYS_HOME%/bin;%PATH%

set "WD=%__CD__%"
if NOT EXIST "%WD%msys-2.0.dll" set "WD=%~dp0usr\bin\"

set MSYSCON=
"%WD%bash" %*
```
cygwin.bat:
```
@echo off

C:
chdir C:\portable\cygwin\bin

bash --login -i -c "cd \"`cygpath -u '%CD%'`\";bash"
```