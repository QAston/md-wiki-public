## visual studio

### setup

- install visual studio 2019 community
- set up scripts for dev shell and add to portable\bin\cmd & ps
    - take contents from shortcuts to the developer prompt/powershell from start menu
    - env_vs.bat:
```
call "C:\Program Files (x86)\Microsoft Visual Studio\2019\Community\Common7\Tools\VsDevCmd.bat"
```
    - env_vs.ps1
```
Import-Module "C:\Program Files (x86)\Microsoft Visual Studio\2019\Community\Common7\Tools\Microsoft.VisualStudio.DevShell.dll"; Enter-VsDevShell e821c35b
```
- install workloads in visual studio installer:
    - .net desktop development
    - desktop development with c++
    - uwp development
    - gamedev unity
    - gamedev c++
    - linux development c++
    - .net core cross platform development
- install components in visual studio installer
    - clang compiler for windows
    - clang cl
- install extensions
    - no need to install editorconfig, it's built in into vs now
    - gitextensions
    - vsvim - bad because it doesn't work like vscode insert mode pluging and overrides keybinds in all modes
    - resxmanager - editing resource files
    - shell context menu
    - edit project - allows editing slns from visual studio
    - windows template studio
- tools -> options
    - terminal - add entries
        - normal cmd and powershell
        - msys and mingw64
    - vsvim -> let visualstudio handle keys
- view
    - show terminal
    - show code definition
    - show class view
    - show call hierarchy
    - close team explorer
- window -> save window layout

### old stuff
visual studio iot development
[](https://visualstudiogallery.msdn.microsoft.com/35dbae07-8c1a-4f9d-94b7-bac16cad9c01)<https://visualstudiogallery.msdn.microsoft.com/35dbae07-8c1a-4f9d-94b7-bac16cad9c01>
visual studio linux development:
[](https://visualstudiogallery.msdn.microsoft.com/725025cf-7067-45c2-8d01-1e0fd359ae6e)<https://visualstudiogallery.msdn.microsoft.com/725025cf-7067-45c2-8d01-1e0fd359ae6e>
visual studio has android support now
visual studio + clang:
-llvm version - clang with  x86_64-pc-windows-msvc - targets 
     -package on llvm page, can install into visualstudio, compatible with msvc binaries
-ms c2 version -clang with Microsoft CodeGen - clang + ms optimization backend
     -is visual studio addon
-mingw version
     -not compatible with msvc