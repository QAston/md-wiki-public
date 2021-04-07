## sysinternals

- a big pile of tools for windows

### setup

- copy to c:\portable\sysinternals
- remove 32bit executables
- verify that symbols are working
    - open procmon right click -> stack and the location should show methodname+addr instead of dllname+addr (it can take a few seconds to update)

### guides

- [setting up symbols](https://www.xitalogy.com/windows-internals/2019/08/14/windows-internals-how-to-configure-symbols-in-sysinternals-process-explorer.html)
- [debugging issues using procmon and procexp](https://www.youtube.com/watch?v=pjKNx41Ubxw)