## windows sdk

### setup 

- run the installer from <https://developer.microsoft.com/en-gb/windows/downloads/windows-10-sdk/>
    - can install components which aren't provided by visual studio (like debugging and profiling)
    - sees components installed by visual studio (they both install to same location) and doesn't conflict/override them
- install [windows performance toolkit](https://docs.microsoft.com/en-us/windows-hardware/test/wpt/) and [debugging tools for windows](https://docs.microsoft.com/en-us/windows-hardware/drivers/debugger/)
- remove windows perf toolkit from system path
- add entries env_windbg and env_winperf to portable_env
- install windbg preview from windows store - it has features not contained in the old debugger 
    - run WinDbgX /I to set as the default postmortem debugger
        - add `.jdinfo 0x%p` to `Computer\HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\AeDebug` `Debug` entry
    - send to shim-msyscommon `C:\Users\qasto\AppData\Local\Microsoft\WindowsApps\WinDbgX.exe`
- configure symbols
    - `file.exe+addres` means symbols are not found for a file
    - `file.exe!methodName()` means symbols are found
    - set env variable: `_NT_SYMBOL_PATH=cache*%TMP%\SymbolCache;srv*https://msdl.microsoft.com/download/symbols`
    - use `_NT_ALT_SYMBOL_PATH` to prepend any symbol dirs if needed, not needed if symbols are in the same dir as executable

### sdk tools docs

- [debugging tools for windows](https://docs.microsoft.com/en-us/windows-hardware/drivers/debugger/)
- [application verifier](https://docs.microsoft.com/en-us/windows-hardware/drivers/devtest/application-verifier)
- [windows performance toolkit](https://docs.microsoft.com/en-us/windows-hardware/test/wpt/)

### usage

#### debuggers

- [configuring a post mortem debugger](https://docs.microsoft.com/en-us/windows-hardware/drivers/debugger/enabling-postmortem-debugging)
    - [configuring a post-mortem debugger](https://docs.microsoft.com/en-us/windows/win32/debug/configuring-automatic-debugging)
    - [when there's no post-mortem debugger, winapi can be configured to write crash dumps](https://docs.microsoft.com/en-us/windows/win32/wer/collecting-user-mode-dumps)
- [configuring symbols](https://docs.microsoft.com/en-us/windows-hardware/drivers/debugger/symbols)
    - [old guide](https://docs.microsoft.com/en-us/windows/win32/dxtecharts/debugging-with-symbols#getting-the-symbols-you-need)
- [support tools](https://docs.microsoft.com/en-us/windows-hardware/drivers/debugger/extra-tools)
    - gflags
        - `gflags /i <executable>` - change debugging flags for the executable
        - `gflags +sls` - enable dll path debugging
        - [flag list](https://docs.microsoft.com/en-us/windows-hardware/drivers/debugger/gflags-flag-table)
    - kill - kill processes
    - [logger](https://docs.microsoft.com/en-us/windows-hardware/drivers/debugger/logger) and [log viewer](https://docs.microsoft.com/en-us/windows-hardware/drivers/debugger/logviewer)
    - remote - remote execution of commands
    - tlist - same as tasklist
    - dbh - views symbols
    - pdbcopy
    - umdh - [user mode dump heap](https://docs.microsoft.com/en-us/windows-hardware/drivers/debugger/umdh)

#### commands

- `!` commands are extensions, `.` commands are directives which controll debugger process itself
- [command index](https://docs.microsoft.com/en-us/windows-hardware/drivers/debugger/commands)
- `x modulename!funname`  - show symbols, allows wildcards
- `bu` - set unresolved breakpoint
- `ba` - breakpoint on memory access
- `bl` - list breakpoints
- `g` - start running, `g-` - start running backwards in the execution recording
- `p` - step, `p-` - step backwards in the execution recording
- `lm` - list modules(execs/ddls)
- `k` - print current stacktrace
- `~` - show threads
    - `~<threadnum>s` - switch to thread
    - `~f` - freeze a thread, `~m` resume
- `|` - show processes
- `qd` - quit and detach
- `!analyze -v` - analyze using microsoft's issues database
- `.srcpath` - set source path, `.srcpath+` adds to the source path
- `dx` - access data model
    - `dx -r1 @$curprocess.TTD.Events` - print time travel events
    - `dx &DisplayGreeting!GetCppConGreeting` - get address of a method in a dll
- `t` - trace `t-` trace backwards in execution recording
- `!tt ` - travel in the execution recording, percentage 0-100 or an absolute location
- `!positions` - shows positions of active threads in the execution recording

#### windbg preview

- `WinDbgX /?` - prints command line help
- `WinDbgX <command>` starts command and attaches the debugger
- `WinDbgX -i <executable> -y <symbolpath if needed> -z <dump file path>` - open a dump file
- to debug mingw builds you should run with env_mingw64, otherwise it'll complain about missing dlls
- sessions are automatically created for each open process and saved to `C:\Users\qasto\AppData\Local\DBG\Targets` with the session settings
- [time travel debugging](https://docs.microsoft.com/en-us/windows-hardware/drivers/debugger/time-travel-debugging-overview)
    - needs to be started as admin - run within an admin terminal
    - kills application on exit
    - no way to start recording from command line - need to go through gui `file -> attach/launch(advanced) -> record time travel` instead

### windows api

- [api choices overview](https://docs.microsoft.com/en-us/cpp/windows/overview-of-windows-programming-in-cpp?view=msvc-160)
- <https://en.wikibooks.org/wiki/Windows_Programming>
- <https://github.com/microsoft/cppwinrt>
- [native api - api below win32](https://www.youtube.com/watch?v=a0KozcRhotM)
- [structured exception handling](https://www.youtube.com/watch?v=COEv2kq_Ht8)
- [windows diagnostics API and config](https://docs.microsoft.com/en-us/windows/win32/diagnostics)
    - contains APIs for writing debuggers and monitoring apps, as well as making apps resilient to failure
    - [windows error reporting](https://docs.microsoft.com/en-us/windows/win32/wer/windows-error-reporting) API configures what happens when an unhandled exception fires
    - [application recovery and restart](https://docs.microsoft.com/en-us/windows/win32/recovery/using-application-recovery-and-restart) can be configured to automatically restart from WER