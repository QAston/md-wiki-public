## gdb

### overview

- <https://www.youtube.com/watch?v=Yq6g_kvyvPU>
- <https://raw.githubusercontent.com/CppCon/CppCon2016/master/Tutorials/GDB%20-%20a%20lot%20more%20than%20you%20realized/GDB%20-%20a%20lot%20more%20than%20you%20realized%20-%20Greg%20Law%20-%20CppCon%202016.pdf>

### usage

 - ctrl-x a - enable/disable to curses ui
 - ctrl-l - repaint curses
 - ctrl-x 2 - switch contents of the curses window
 - source path/to/python/script.py - can run/load python scripts which can manipulate gdb and provide pretty printing and other stuff
 - break [filename:]fnname
 - break [filename:]linenum
 - rbreak - regex breakpoint - breaks on all fn names mathcing regex
 - `print <expression>` - if expression is complex it will be called as in call
 - `call <expression>` - calls using the stopped stack, the program must be in a good state for this to work
 - watches (different types)
    - watch varinscope
    - watch -l memorylocation # watches even if var goes out of scope
    - rwatch # break on read
    - [dprintf](https://doc.ecoscentric.com/gnutools/doc/gdb/Dynamic-Printf.html) - insert a print at a location
    - all watches/breakpoints can be conditional, be executed only in a given thread, etc
 - threat apply all bt full - backtrace of all threads
 - catch catch - break on all exceptions
 - catch syscall - break on syscalls

### building with better gdb info

```
add_compile_options(
  "-Wall" "-Wpedantic" "-Wextra" "-fexceptions"
  "$<$<CONFIG:DEBUG>:-O0;-g3;-ggdb>"
)
```

### setup

- nano ~/.gdbinit
```
set history save on
set print pretty on
```

### utilities

- <https://github.com/cyrus-and/gdb-dashboard>