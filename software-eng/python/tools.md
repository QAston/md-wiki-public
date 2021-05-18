# Python installing tools

General guide: <https://packaging.python.org/tutorials/installing-packages/> and <https://packaging.python.org/overview/>

## Python3 venv

See <https://docs.python.org/3/library/venv.html> for details

1. have python 3 in path: `env_python3`
2. run: `python -m venv .` to run venv in a directory
   * this creates directory for a python distribution (by default the one from interpreter) and packages
3. activate by sourcing bin/activate* script 
4. run pip commands as desired for installing modules, examples:
   * `pip install -e .` installs a development version, which will reload changes, use when setup.py is present
   * `pip install -r requirements.txt` if requirements fileis present

## readline

- python can hook a readline library for reading input using PyOS_ReadlineFunctionPointer
   - by default done in Modules/readline.c which is only built on posix systems
- mingw build has readline disabled <https://github.com/msys2/MINGW-packages/blob/master/mingw-w64-python/1850-disable-readline.patch>
   - back out 1850-disable-readline.patch, remote it from PKGBUILD
   - uncomment readline module in src/Python-3.8.9/Modules/readline.c
   - fix build of Modules/readline.c
      - add #undef HAVE_RL_RESIZE_TERMINAL because msvcrt doesn't provide a resize signal
   - run makepkg-mingw -e to build with these changes
   - sadly, this doesn't fully work, the keyboard input isn't read because winsock2's select only works on sockets and not on files
      - todo: fix readline_until_enter_or_signal to properly read files
      - the basic fix is to just make readline_until_enter_or_signal call readline() directly, this will work but you have to use ctrl+d instead of ctrl+c to interrupt
      - todo: look into signal handling and callback api of readline to have a better workaround
         - https://tiswww.case.edu/php/chet/readline/readline.html#SEC44
         - https://tiswww.case.edu/php/chet/readline/readline.html#SEC41
   - it looks like mingw-based python builds fail some tests, but the failures aren't dramatic (+0.0 vs -0.0 for example)
      - python -m test --pgo doesn't pass in msys build(2 tests), and fails in mingw with a couple more of test cases (in lib/ntpath.py we set path separator to '/' when running in bash, this causes some tests to fail, is that even a good idea?)
      - might be better to build the msvc version of python with msvc readline (clink has a working readline build)
- msys build has readline enabled and working
- native windows builds have readline disabled for some reason


## Pip

- `pip freeze` prints requirements.txt with all versions specified