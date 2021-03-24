# c++ tools

## toolchain

* [vcpkg](../vcpkg.md) for downloading libraries
* cmake for generating build files
* ninja/msbuild for building

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
* clang compiler has sanitizer tools:(-fsanitize flag)
    -fsanitize=address: AddressSanitizer, a memory error detector.
    -fsanitize=thread: ThreadSanitizer, a data race detector.
    -fsanitize=memory: MemorySanitizer, a detector of uninitialized reads. Requires instrumentation of all program code.
    -fsanitize=undefined: UndefinedBehaviorSanitizer, a fast and compatible undefined behavior checker.
    -fsanitize=dataflow: DataFlowSanitizer, a general data flow analysis.
    -fsanitize=cfi: control flow integrity checks. Requires -flto.
    -fsanitize=safe-stack: safe stack protection against stack-based memory corruption errors.
* compiler static analizer
    -Xanalyzer
* compiler coverage:
    -fcoverage-* options
* <https://llvm.org/docs/XRay.html> - tracing



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

## frameworks

* <https://github.com/libuv/libuv>
- [a multimedia framework that glues a lot of libraries like opengl, openvs, etc](https://openframeworks.cc/about/)
- [mingw llvm setup](https://github.com/valtron/llvm-stuff/wiki/Set-up-Windows-dev-environment-with-MSYS2)