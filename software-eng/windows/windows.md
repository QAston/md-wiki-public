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
* [a gnu software package](https://github.com/bmatzelle/gow)
* [new windows app framework merging uwp and desktop?](https://github.com/microsoft/ProjectReunion)
* [windows usability tips](https://lobste.rs/s/zpumlt/tips_for_surviving_windows_desktop_if_you)
* <https://github.com/processhacker/processhacker>
* <https://github.com/stax76/OpenWithPlusPlus>
* <https://github.com/w00zla/WslPathShellExt>
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
    - you can set the variables with name `=name` and they won't show up in `env` prints, and are accessible through winapi, but can't be set from cmd.exe, some people call these "hidden variables"


### posix api implementations

* midpix - a syscall and posix layer for windows that allows using linux's libc on windows
    * [](https://midipix.org/#sec-implementation)
    * [](https://dev.midipix.org/)
* cygwin
* https://github.com/JFLarvoire/SysToolsLib/tree/master/C/MsvcLibX

### shell

* [drag and drop does things based on dest and modifier keys](https://www.tenforums.com/tutorials/38097-change-drag-drop-default-action-windows.html)
    - drag with RMB allows specifying exactly if you want to copy, link or symlink
* extensions: <https://docs.microsoft.com/en-us/windows/win32/shell/shell-exts>
    * see/disable all shell extensions <https://www.nirsoft.net/utils/shexview.html>
    * fuse could be great as a special directory extension
* shortcuts
    * ctrl+alt=altgr (windows specific shortcut, can't be disabled)
        * ctrl+alt+`key` can be used as a different shortcut, which will take priority over altgr letters, so it's fine if you don't need the letter
        * don't use ctrl+alt as shortcut in applications for other people, because it'll block them from typing their altgr combos
        * whether right alt is altgr on a keybord is defined by the keyboard layout (if there's ANY key combinations for ctrl+alt+letter in the layout altgr is automatically enabled)
    * common
        * alt + underline letter - select menu item
        * shift + F10 - show context menu
        * F10 - show/focus app file menu
        * alt + spacebar - open window maximize/close context menu
        * hold left alt to enable underline letter shortcuts help
            * then press the key to activate the shorctut
            * esc backs exits the help
            * can be also activated without holding by just doing alt-`<char>`
        * F11 - fulscreen
        * numlock enabled + alt + numpad keys - alt codes, doesn' t work without a numpad
    * store apps
        - win-shift-enter - fulscreen
    * tools
        * win-g - game bar
        * win-r - run dialog
        * win-p - presentation/screen sharing
        * win-v - clipboard
        * win-l - lock pc
        * win-shift-s - screenshot tool
        * win-. - emoji browser
    * task manager
        * you can start taskmanager without ctrl-alt-del by right click on windows icon
            * also ctrl-shift-esc
        * you can run admin prompt from task manager by ctrl-clicking file -> run new task menu
        * you can debug, create a debug dump, "analyse wait chain" which debugs waits, set priority and affinity and end the process tree, all in the details menu
        * you can add more columns to details tab columns including: gpu memory usage, detailed memory usage, which gpu is running the app, dpi awareness, various flags like virtualisation, num handles, num threads, path, command line, performed io details, memory protection config, etc
    * desktop
        * win-up/down - make current window bigger/smaller, different stages between maximizing and minimizing
        * win-shift-up - stretch window to top and bottom of the screen
        * win-shift-down - demaximize then minimize and deselect
        * win-left/right - snap current window to left/right half of monitor and move between and monitors
            * after snapping the other side will show a selection for the second window to snap (snap assist)
            * this can be configured in the settings -> multitasking and sometimes breaks, which is fixed by restarting explorer 
        * win-shift-left/right - move to another monitor
        * window dragging
            * drag to left/right side of monitor - snap window
            * drag to a corner - snap to a corner quadrant
            * drag to top - maximize
        * win-d - bring desktop to front(doesn't actually minimize windows), another press brings it back
        * win-, - preview desktop while holding win key
        * win-home - minimize all but active window, second press restores minimized
        * win-m - minimize all windows
            * win-shift-m - restore minimized windows to desktop
        * win-tab - show virtual desktops/task view
            * apps which show recent tasks on taskbar-right-click will show their last tasks here so you can reopen them
            * you can right click on windows in this view to pin a window/app to all desktops/move app between desktops
            * virtual desktops are great if you have few distinct groups of windows with clustered interactions and little cross-over
            * settings->multitasking configures the behaviour of taskbar and alt-tab with virtual desktop (by default desktop has a private taskbar/alt-tab)
            * desktops can be renamed by clicking their name in the task view
            * [new virtual desktop features coming](https://blogs.windows.com/windows-insider/2021/03/17/announcing-windows-10-insider-preview-build-21337/)
            * win-ctrl-d - creates a new virtual desktop
            * win-ctrl-f4 - close curr entvirtual desktop
            * win-ctrl-left/right - switches between desktops
        * ctrl-alt-tab - menu to bring a window to front with arrows
            * arrows or tab(next) or shift tab(previous) to choose, esc to cancel, enter to select, delete to close the window
            * alt-tab - press to activate the menu and move 1 pos forward, releasing alt is selects the window, otherwise standard controls
            * alt-shift-tab - press to activate the menu and move 1 pos backward, releasing alt is selects the window, otherwise standard controls
        * alt-esc - cycle through windows in order of opening
    * taskbar
        * right click shows an app specific list of options ("jump list")
            * often it's showing the task history so you can go back to it (and see it in task view)
        * right click on empty part of taskbar to manipulate all windows(cascade/stack/etc)
        * shift-click on taskbar icon opens a new app instance
        * ctrl-shift-click on taskbar icon opens a new instance as admin
        * shift-rightclick on taskbar icon opens the window menu (minimize/maximize/close/etc)
        * shift-rightclick on taskbar group icon opens the group window/arrangement menu (arrange side by side, top-to-bottom, cascade, maximize, etc)
        * ctrl-click on taskbar group cycles through the windows
        * windows-`<1-9>` - taskbar entries 1-9
            * open a window if none are open, select a window to bring to front if there are open windows
            * with ctrl - cycle through windows
            * with shift - open a new instance
            * with ctrl-shift - open as admin
            * with alt - show the app's "jump list", same as right click
        * windows-T - cycle through taskbar apps and taskbars
        * right click -> toolbars -> new toolbar 
            * adds a menu from which you can start applications which are in the specified custom directory
            * also you can right click -> open directory
    * start menu
        * ctrl-shift-click starts with admin priviledges
    * explorer
        * you can run powershell/cmd.exe/wd -d . in the explorer dir by just typing the line in the address bar
            * you can also evaluate env variables there too
            * also work in file selection dialogs!
        * shift right click gives additional options like copy path or open shell
        * interesting new menu options: group by and history
        * alt-D/Ctrl-l/F4 - select address bar
            * tab switches between panes
        * ctrl-E/ctrl-F/F3 - select search
        * ctrl-N - new window
        * ctrl-W - close window
        * F11 - maximize
        * win-e - open explorer
        * alt-p - preview pane
        * alt-shift-p - details pane
        * alt-enter - show properties
        * delete/shift-delete - move to bin/permadelete
        * history: alt-left - back, alt-right - forward
        * alt-up - up a dir
        * backspace - moves between recent locations?
        * navigation pane(tree view):
            * ctrl-shift-E - find current folder in tree view
            * numlock area asterisk - undind subdirs recursively
            * right arrow/numlock area plus - unwind single dir
            * left arrow/numlock area plus - wind-back single dir
            * backspace - go above in tree view
            * home/end - first/last entry in tree view
        * dir view
            * home/end - first/last item in directory
            * ctrl + arrows - move between items, space adds to selection, enter does action
            * shift + arrows - select adjacent items
            * F2 - rename
            * shift + f10 - open context menu
            * ctrl-z/y - undo/redo
    * adding app shortcuts
        * create a windows shortcut, edit file properties, set the key combination

### generic windows shortcuts
* text editing
    * ctrl-left - move to beginning of prev word
    * ctrl-right - move to beginning of next word
    * ctrl-backspace - delete prev word
    * ctrl-delete - delete next word
    * ctrl-up - move to beginning of a paragraph
    * ctrl-down - move to end of paragraph
    * home/end - beginning/end of current line
    * ctrl-home/end - beginning/end of the text
    * pg up/down - move text by a page
    * shift-left/right - select char at a time
    * ctrl-shift-left/right - select word at a time
    * shift-ctrl-home/end - select text towards beginning/end
    * shift-pgup/down - select a page
    * ctrl-a - select all
    * ctrl-z - undo
    * ctrl-y - redo (sometimes ctrl+shift+z)
* browsers
    * 

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

