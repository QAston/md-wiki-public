# c++ tools

## toolchain

* [vcpkg](../vcpkg.md) for downloading libraries
* cmake for generating build files
* ninja/msbuild for building

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
* lldb

## gnu tools

* as – GNU Assembler Command
* ld – GNU Linker Command
    * https://wiki.osdev.org/Linker_Scripts
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
* ldd - show shared libraries for executable
* [gdb](../tools/gdb.md)

## xplatform tools

* [drmemory memory issues debugger](https://github.com/DynamoRIO/drmemory)
    * includes [strace for windows - drstrace](https://drmemory.org/page_drstrace.html) for tracing syscalls
    * includes [symbol query tool](https://drmemory.org/page_symquery.html)
    * includes [dr fuzz](https://drmemory.org/page_fuzzer.html)
    * also shadow memory (seeing the memory contents) and symbol caching
* google performance tools: <https://github.com/gperftools/gperftools>
    * also golang profiler impl <https://github.com/google/pprof> (doesn't include instrumentation)

## linux tools

* [strace](https://strace.io/) - trace all syscalls of a process
    * `strace <cmd>`
* [ltrace](https://ltrace.org/) - trace all dllcalls of a process
    * `ltrace <cmd>`
* [linux perf](https://perf.wiki.kernel.org/index.php/Main_Page)
    * `pacman -S perf`
    * `perf trace` - trace evey syscall
        * `perf trace <cmd>` - trace every syscall by cmd
        * faster than strace, more flexible, but requires more priviledges
    * [perf intel pt](https://man7.org/linux/man-pages/man1/perf-intel-pt.1.html) - processor trace - shows branches taken in the program
```
perf record -e intel_pt//u ./a.out
perf script
```
* [ftrace](https://www.kernel.org/doc/Documentation/trace/ftrace.txt) - function tracer
    * a fast way to trace various kernel functions
    * controlled through /sys/kernel/debug/tracing or [trace-cmd](https://github.com/rostedt/trace-cmd) (pacman -S trace-cmd)
* [systemtap](https://sourceware.org/systemtap/wiki)
* [dtrace4linux - looks dead? last update 2019](https://github.com/dtrace4linux/linux)
* [lttng](https://lttng.org/docs/v2.12/)
* [other tracing resources](https://elinux.org/Kernel_Trace_Systems)
* [record replay debugger](https://rr-project.org/)
    * [similar software](https://github.com/rr-debugger/rr/wiki/Related-work)
* [amd profiler for cpu/gpu/opengl and stuff](https://developer.amd.com/amd-uprof/)
* [sysprof](http://www.sysprof.com/)
* valgrind
    * has multiple tools
        * memcheck - leaks/oob
        * cachegrind - profiling cache
        * massif - heap profiler
        * helgrind - multithreaded programs issues
        * data race detector - multithreading checker
* [eBPF - run userspace code in kernel](https://ebpf.io/projects)
    * pacman -S bpf - [tools for building bpf software](https://archlinux.org/packages/community/x86_64/bpf/)
    * can be used to add tracing code
        * [bpftrace](https://bpftrace.org/)
        * bcc [toolkit](https://github.com/iovisor/bcc) and [tools](https://github.com/iovisor/bcc#tools) for creating kernel tracing
        * [ply - a dynamic tracer for linux](https://wkz.github.io/ply/)
        * <https://github.com/aquasecurity/tracee>


## windows tools

* [everything you want to know about dlls  - how dlls work and how to inspect them](https://www.youtube.com/watch?v=JPQWQfDhICA)
    * gflags +sls
    * dumpbin
* [additional visual studio tools](https://docs.microsoft.com/en-us/cpp/build/reference/c-cpp-build-tools?view=msvc-160)
    * dumpbin - print info on exe/dll/lib files
    * editbin - change options of a exe/dll/lib file
    * lib
    * nmake - makefile execution
    * errlook - print string error based on error code
* visual studio
    * [code sanitizers](https://docs.microsoft.com/en-us/cpp/sanitizers/?view=msvc-160) now
        * currently only address sanitizer
    * [profiler](https://docs.microsoft.com/en-us/visualstudio/profiling/?view=vs-2019)
    * [static analysis](https://docs.microsoft.com/en-us/cpp/code-quality/?view=msvc-160)
* [drmingw](https://github.com/jrfonseca/drmingw)
* [nttrace - strace for windows](https://github.com/rogerorr/NtTrace)
* [stracent - strace for windows - not updated since 2015, x86 only](https://intellectualheaven.com/?BH=StraceNT)
* [sysinternals](../windows/sysinternals.md)
* windows valgrind alternatives:
    * <http://stackoverflow.com/questions/413477/is-there-a-good-valgrind-substitute-for-windows#413842>
    * <https://kinddragon.github.io/vld/>
* [tools in the windows sdk](../windows/windows_sdk.md)
* [hypevisor debugger](https://github.com/HyperDbg/HyperDbg)

## documentation

* doxygen
* cppdoc

## frameworks and libraries

- [nice list on cppreference](https://en.cppreference.com/w/cpp/links/libs)
- [cross platform async io framework](https://github.com/libuv/libuv)
- [seastar - high perf c++ io framework](http://seastar.io/)
- [a large collection of advanced libraries](https://github.com/oneapi-src)
    - [thread building blocks - framework for writing parallel programs](https://github.com/oneapi-src/oneTBB)
- [a multimedia framework that glues a lot of libraries like opengl, openvs, etc](https://openframeworks.cc/about/)
- [mingw llvm setup](https://github.com/valtron/llvm-stuff/wiki/Set-up-Windows-dev-environment-with-MSYS2)
- [guidelines support library](https://github.com/Microsoft/GSL)
-   [alternative, smaller gsl implementation](https://github.com/gsl-lite/gsl-lite)
- [type_safe - utilities for write safer/more-strongly-typed c++](https://github.com/foonathan/type_safe)
- [kvasir - standard library extensions for microcontrollers](https://github.com/kvasir-io/mpl)
- [etl - standard library extensions for microcontrollers](https://github.com/ETLCPP/etl)
- [google standard library](https://abseil.io/)
- [bloomberg standard libraries](https://github.com/bloomberg/bde)
- [eastl - ea standard library](https://github.com/electronicarts/EASTL)
- [facebook folly - fb standard library](https://github.com/facebook/folly)
    - includes a tag_invoke impl
- [persistent data structures](https://github.com/arximboldi/immer)
- [result + throw error handling proposal](https://github.com/TartanLlama/expected)
- boost
    - [a gigantic library with lots of stuff](https://www.boost.org/doc/libs/)
- [brigand](https://github.com/edouarda/brigand) - a replacement for boost.mpl - metaprogramming
- [metric units library](https://github.com/mpusz/units)
- [physical metrics library with validation](https://github.com/jansende/benri)
- [compile time big num library](https://github.com/niekbouman/ctbignum)
- [fmt - typesafe formatting library](https://fmt.dev/latest/index.html)
- [text encoding utilities](https://github.com/soasis/text)
- [entt - entity component system framework](https://github.com/skypjack/entt)
- [hsfm - hierarchical finite state machine](https://github.com/andrew-gresyk/HFSM2)
- [data flow architecture for c++](https://github.com/arximboldi/lager)
- [rx cpp - reactive programming (data flow?)](https://github.com/ReactiveX/RxCpp)
- [transducers for c++](https://github.com/arximboldi/zug)
- [error handling without exceptions](https://ned14.github.io/outcome/)
- [quantstack - scientific algorithms](https://xtl.readthedocs.io/en/latest/#)
- [simd intrinsics abstraction](https://xsimd.readthedocs.io/en/latest/)
- [vectorclass - simd intrinsics abstraction](https://github.com/vectorclass)
- [enoki - vectorization abstractions](https://github.com/mitsuba-renderer/enoki)
- [simd everywhere - polyfill for simd intrinsics](https://github.com/simd-everywhere/simde)
- [numerical analysis with multi-dim array expressions](https://xtensor.readthedocs.io/en/latest/)
- [numerical analysis with multi-dim array expressions](https://github.com/romeric/Fastor)
- [machine learning algorithms and toolkit](http://dlib.net/)
- [civil/absolute time utilities](https://github.com/google/cctz)
- [fixed precision numeric classes](https://github.com/johnmcfarlane/cnl)
- [c++ math library](https://bitbucket.org/blaze-lib/blaze/src/master/)
- [compile time regexp impl](https://github.com/hanickadot/compile-time-regular-expressions)
- [maths for opengl](https://github.com/g-truc/glm)
- [multithreaded programming library](https://github.com/copperspice/cs_libguarded)
- [coroutine utilities for c++20 coroutines](https://github.com/lewissbaker/cppcoro)
- [sender/receiver async multithreading programming proposal](https://github.com/facebookexperimental/libunifex)
    - includes a tag_invoke impl
- [constexpr based feature detection](https://github.com/jfalcou/spy)
- [abstraction over compiler features](https://nemequ.github.io/hedley/user-guide.html)
- [pipes - alternative to ranges/transducers](https://github.com/joboccara/Pipes)
- [named type - strong typedef implementation](https://github.com/joboccara/NamedType)
- [c++20 ranges for c++17](https://github.com/tcbrindle/NanoRange)
- [basis for c++20 ranges proposal](https://github.com/ericniebler/range-v3)
- [eigen - linear algebra library](http://eigen.tuxfamily.org/index.php?title=Main_Page)
- [unit testing for c++](https://github.com/catchorg/Catch2)
- [unit testing for c++, allows writing tests near library code](https://github.com/onqtam/doctest)
- [googletest](https://github.com/google/googletest)
- [tts - testing library](https://github.com/jfalcou/tts)
- [trompleoeil - header only mocking framework](https://github.com/rollbear/trompeloeil)
- [google benchmark framework](https://github.com/google/benchmark)
- [spdlog - logging library](https://github.com/gabime/spdlog)
- [jemalloc - faster default allocator](https://github.com/jemalloc/jemalloc)
- [utility for tag_invoke](https://github.com/bfgroup/duck_invoke)
- [utility for enum reflection](https://github.com/krabicezpapundeklu/smart_enum)
- [utility for enum reflection](https://github.com/aantron/better-enums)
- [utility for multimethods](https://github.com/jll63/yomm2)