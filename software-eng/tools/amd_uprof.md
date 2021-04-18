### links

* <https://github.com/jlgreathouse/AMD_IBS_Toolkit>
* <https://community.amd.com/t5/server-gurus-discussions/amd-uprof-not-showing-performance-counters/td-p/438368>

### windows setup 

- download installer from [amd](https://developer.amd.com/amd-uprof/)
- add to portable env
```
uprof = [
    {command = 'env', key = 'PATH', value = 'C:\Program Files\AMD\AMDuProf\bin', mode = 'PREPEND_PATH'}
]
```
- enable AMD IBS in bios -> peripherals

### wsl2 setup

- download tar.bz2 and unpack
- run `sudo sh -c 'echo -1 >/proc/sys/kernel/perf_event_paranoid'` at startup

### usage

- to enable advanced profiling of an application click the `next` button in the bottom right, it provides more settings
- hyperv/wsl being enabled breaks hardware counter support :( (ibs is disabled message, even though it's enabled)
    - since 3.4 guest vms have support for time based profiling but nothing more
    - to enable full counters in host system disable hyperv and wsl2 features in windows feature dialog
        - to reenable, reenable the settings and run artix.exe