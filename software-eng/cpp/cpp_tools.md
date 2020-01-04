# c++ tools

## debuggers

* gdb
* lldb
* <https://docs.microsoft.com/en-us/windows-hardware/drivers/debugger/time-travel-debugging-overview>
* visualgdb - płatny :(
    * <http://visualgdb.com/tutorials/mingw/mingw64/>
* <http://wingdb.com/> - paid

## profilers and tracing

* gperf
* amd profiler for cpu/gpu/opengl and stuff
* <http://developer.amd.com/tools-and-sdks/opencl-zone/codexl/>
* valgrind
    * linux only
    * <http://stackoverflow.com/questions/413477/is-there-a-good-valgrind-substitute-for-windows#413842>
* drmemory
* amd tools <http://developer.amd.com/tools-and-sdks/tools-libraries-sdks-index/>
* google performance tools: <https://github.com/gperftools/gperftools>

## clang tools

* <http://clang.llvm.org/docs/ClangTools.html>
* clang-check - static analisys
* clang-tidy - lint, can do static analisys and coding style checks
* clang-format - auto formatter
* modularize -introduce real modules to c++ <http://clang.llvm.org/extra/modularize.html>

## gnu tools

* as – GNU Assembler Command
* ld – GNU Linker Command
* ar – GNU Archive Command
* nm – List Object File Symbols
    * nm -gC - public symbols
* objcopy – Copy and Translate Object Files
* objdump – Display Object File Information
* size – List Section Size and Toal Size
* strings – Display Printable Characters from a File
* strip – Discard Symbols from Object File
* c++filt – Demangle Command
* addr2line – Convert Address to Filename and Numbers
* readelf – Display ELF File Info
* Ldd - show shared libraries for executable

## documentation

* doxygen
* cppdoc