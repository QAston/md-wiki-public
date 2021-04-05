## windows sdk

### setup 

- run the installer from <https://developer.microsoft.com/en-gb/windows/downloads/windows-10-sdk/>
    - can install components which aren't provided by visual studio (like debugging and profiling)
    - sees components installed by visual studio (they both install to same location) and doesn't conflict/override them
- install [windows performance toolkit](https://docs.microsoft.com/en-us/windows-hardware/test/wpt/) and [debugging tools for windows](https://docs.microsoft.com/en-us/windows-hardware/drivers/debugger/)
- remove windows perf toolkit from system path
- add entries env_windbg and env_winperf to portable_env

### debugging tools

- overview: [debugging tools for windows](https://docs.microsoft.com/en-us/windows-hardware/drivers/debugger/)
- [windows diagnostics tools and config](https://docs.microsoft.com/en-us/windows/win32/diagnostics)
    - windows reporting configures what happens when an unhandled exception fires (the send to windows/open debugger menu)
        - can be configured to restart the app
    - tool help library - api for querying state dumps of apps

### performance tools

- [windows performance toolkit](https://docs.microsoft.com/en-us/windows-hardware/test/wpt/)

### application verifier

- [overview](https://docs.microsoft.com/en-us/windows-hardware/drivers/devtest/application-verifier)

### windows api

- [api choices overview](https://docs.microsoft.com/en-us/cpp/windows/overview-of-windows-programming-in-cpp?view=msvc-160)
- <https://en.wikibooks.org/wiki/Windows_Programming>
- <https://github.com/microsoft/cppwinrt>
