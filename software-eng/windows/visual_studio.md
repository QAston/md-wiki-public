## visual studio

### usage

#### cmake projects

- right click in directory -> open in visual studio
- right click in solution explorer -> add cmake configuration (or open cmake settings)
    - creates CMakeSettings.json if doesn't exist
    - [reference](https://docs.microsoft.com/en-us/cpp/build/cmakesettings-reference?view=msvc-160)
- click `+` in the window and select a build configuration to add, some useful ones
    - x64 debug/release - uses msvc
    - x64-clang-debug/release - uses msvc clang
    - wsl-gcc-debug/release - connects to wsl default distro
    - wsl-clang-debug/release - connects to wsl default distro
        - uses linux gdb debugger
        - needs `"gdbpath": "/home/dariusza/bin/gdb-login-env"` to set env variables properly for sound/gui/path to work
    - mingw64-debug/release - mingw gcc (how does it find mingw?)
        - visual studio debugger works when using `dynamic` triplet but doesn't work for static, even though in neither case pdbs are produced
    - linux-gcc/clang-debug/release - remote linux instance over ssh
- the configurations can now be selected from run configuration menu and edited in CMakeSettings.json
    - [reference](https://docs.microsoft.com/en-us/cpp/build/cmakesettings-reference?view=msvc-160)
- solution explorer -> switch between available views -> cmake targets view
    - shows cmake targets, right clicking allows customizing target options like debugging
- you can invoke custom tasks(launch command or build) from right click menu in the solution explorer, [reference](https://docs.microsoft.com/en-us/cpp/build/tasks-vs-json-schema-reference-cpp?view=msvc-160)
    - 3 types of tasks: local launch of command, remote launch of command and running msbuild
- debug settings for current debug target are in debug->debug and launch settings 
- debug settings references:
    - [base](https://docs.microsoft.com/en-us/cpp/build/launch-vs-schema-reference-cpp?view=msvc-160)
    - [cmake additions](https://docs.microsoft.com/en-us/cpp/build/configure-cmake-debugging-sessions?view=msvc-160#reference-keys-in-cmakesettingsjson)
- when running gdb you can access gdb command line by typing in immediate window:
```
>Debug.MIDebugExec <gdb command here>
```

#### debugging

- advanced stuff (like time travel debugging) is only available in enterprise
- useful windows (debug -> window -> )
    - disassembly - show disassembled code
    - immediate window 
        - evaluate expression in current context
        - `>Debug.<command>` executes a visual studio command
    - autos - local variables in the open editor file
        - shows registers in disassembly view
        - shows variables when parent stack frame is opened
    - breakpoints
        - function breakpoint - breaks when function name is executed
        - data breakpoint - breaks when data is modified
        - code breakpoint - breaks when code is reached
        - all breakpoints can have a condition attached, expression to evaluate
        - breakpoints can be saved to a file and restored later
    - threads
        - allows switching between threads right click -> switch to thread, and freezing
    - memory - show heap/stack at a given address
        - you can get address of variable using &var_name in the immediate window
    - modules - shows dlls, you can see where symbols are coming from and get the base address of a dll
- useful options
    - right click -> run to cursor
    - **right click -> set next statement!** - manual unconditional jump
    - right click -> show disassembly
- can save a dump on breakpoint
- can load a dump, but must have a pdb file to be able to see symbols (mingw builds don't have it)

#### other tools

* [code sanitizers](https://docs.microsoft.com/en-us/cpp/sanitizers/?view=msvc-160) now
    * currently only address sanitizer
* [profiler](https://docs.microsoft.com/en-us/visualstudio/profiling/?view=vs-2019)
* [static analysis](https://docs.microsoft.com/en-us/cpp/code-quality/?view=msvc-160)
   
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
    - fonts and colors -> show settings for terminal -> select delugia mono
- view
    - show terminal
    - show code definition
    - show class view
    - show call hierarchy
    - close team explorer
- window -> save window layout

### old stuff
visual studio iot development
<https://visualstudiogallery.msdn.microsoft.com/35dbae07-8c1a-4f9d-94b7-bac16cad9c01>
visual studio linux development:
<https://visualstudiogallery.msdn.microsoft.com/725025cf-7067-45c2-8d01-1e0fd359ae6e>
visual studio has android support now
visual studio + clang:
-llvm version - clang with  x86_64-pc-windows-msvc - targets 
     -package on llvm page, can install into visualstudio, compatible with msvc binaries
-ms c2 version -clang with Microsoft CodeGen - clang + ms optimization backend
     -is visual studio addon
-mingw version
     -not compatible with msvc