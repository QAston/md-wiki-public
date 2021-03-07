# windows

* [how to get more than 256 length paths](https://msdn.microsoft.com/en-us/library/aa365247%28VS.85%29.aspx?f=255&amp;MSPPError=-2147217396#maxpath)
    * `\\?\` prefix isn't supported by the api
    * `Computer Configuration > Administrative Templates > System > Filesystem > Enable Win32 long paths` can enable long paths, but the app needs to be compiled to have longPathAware setting
* [Scott Hanselman's 2014 Ultimate Developer and Power Users Tool List for Windows - Scott Hanselman](http://www.hanselman.com/blog/ScottHanselmans2014UltimateDeveloperAndPowerUsersToolListForWindows.aspx)
* <https://www.burgaud.com/open-command-window-here>
* [window managers](https://old.reddit.com/r/windows/comments/2rn775/best_tiled_window_manager_for_windows/)
* [compatibility toolkit](https://support.microsoft.com/en-us/topic/how-to-use-the-compatibility-administrator-utility-in-windows-9791a045-9b82-d954-3562-2d22ac973a80)
* [context menu](https://www.lopesoft.com/index.php/en/download/filemenu-tools)
* [sysinternals tool list](https://docs.microsoft.com/en-gb/sysinternals/)
* [everything (search)](https://www.voidtools.com/)
* [a lot of free windows tools](https://www.nirsoft.net/utils/index.html)
* [mostly windows ui apps](https://winaero.com/winaero-apps/)
* [new windows app framework merging uwp and desktop?](https://github.com/microsoft/ProjectReunion)
* [msix - new app isolation technology](https://docs.microsoft.com/en-us/windows/msix/) - containers for windows apps
    - contained in program files\windowsapps\<pkgname>
    - virtual registry
        - registry entries for the app are stored in files, which are merged in the registry when app is running
    - virtual appdata
        - reads/writes to <username>\appdata are redirected to username\appdata\local\packages\<appfamilyname>
    - virtual filesystem
        - app a private copy of all system folders: windows, system32, program files
        - enables you to easily deploy an application which depends on a version of .net core and not worry about the version installed on the system
    - Invoke-CommandInDesktopPackage lets you run a command in the virtualised environment
* [windbg](http://www.windbg.org/)
    - <https://stackoverflow.com/questions/105130/why-use-windbg-vs-the-visual-studio-vs-debugger>
    - [new windbg ui, compatible with old scripts and extensions](https://docs.microsoft.com/en-us/windows-hardware/drivers/debugger/debugging-using-windbg-preview)
* fsutil - new tool for adding links and inspecting ntfs
* setx - set env variables for windows shell
* env variables
    - env variables values can use variables themselves, their values will be substituted (but only if the substituted variable value doesn't have variables itself, otherwise substitution won't happen)
    - path editor will switch to path mode if it finds semicolons and path-looking things
    - setx sets variables in windows registry



### posix api implementations

* midpix - a syscall and posix layer for windows that allows using linux's libc on windows
    * [](https://midipix.org/#sec-implementation)
    * [](https://dev.midipix.org/)
* cygwin
* https://github.com/JFLarvoire/SysToolsLib/tree/master/C/MsvcLibX

### shell

* you can run powershell/cmd.exe/wd -d . in the explorer dir by just typing the line in the address bar
* shift right click gives additional options like copy path or open shell
* you can run admin prompt from task manager by ctrl-clicking file -> run new task menu
* [drag and drop does things based on dest and modifier keys](https://www.tenforums.com/tutorials/38097-change-drag-drop-default-action-windows.html)
    - drag with RMB allows specifying exactly if you want to copy, link or symlink
 
### ntfs links

- hardlink 
    - single drive only point to the same data file
    - visible and treated as a regular file from rest of the system
    - if the file data changes through one hard link, the other is changed too
    - `mklink /h`
- softlink 
    - can span any scope
    - a path to another file
    - breaks if the target is no longer there
    - `mklink` - soft link by default
- junction
    - only works for directories and local absolute paths
    - symilar to softlinks
    - `mklink /j`
* [different types of ntfs links](https://en.wikipedia.org/wiki/NTFS_reparse_point)
    - [see also](https://github.com/googleprojectzero/symboliclink-testing-tools)
* [great symlink tools](https://schinagl.priv.at/)
    - includes shell extension and delete/backup tools which are symlink aware

