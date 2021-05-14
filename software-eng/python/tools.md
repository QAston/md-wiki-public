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

- mingw build has readline disabled <https://github.com/msys2/MINGW-packages/blob/master/mingw-w64-python/1850-disable-readline.patch>
   - back out 1850-disable-readline.patch, remote it from PKGBUILD
   - uncomment readline module in src/Python-3.8.9/Modules/readline.c
   - fix build of Modules/readline.c
      - add #include <winsock.h> for fd_set and select
      - add #undef HAVE_RL_RESIZE_TERMINAL because windows doesn't provide a resize signal
   - sadly, this doesn't fully work, the keyboard input doesn't get processed with this build
      - could use libedit? or try to fix the readline.c to work with windows c runtime?
- msys build has readline enabled and working
- native windows builds have readline disabled for some reason


## Pip

- `pip freeze` prints requirements.txt with all versions specified