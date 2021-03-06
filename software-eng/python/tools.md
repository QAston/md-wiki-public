# Python installing tools

General guide: <https://packaging.python.org/tutorials/installing-packages/> and <https://packaging.python.org/overview/>

## setup for arch

```
sudo pacman -S python-pip
```

## Python3 venv

See <https://docs.python.org/3/library/venv.html> for details

1. have python 3 in path: `env_python3`
2. run: `python -m venv .` to run venv in a directory
   * this creates directory for a python distribution (by default the one from interpreter) and packages
3. activate by sourcing bin/activate* script 
4. run pip commands as desired for installing modules, examples:
   * `pip install -e .` installs a development version, which will reload changes, use when setup.py is present
   * `pip install -r requirements.txt` if requirements fileis present

## Pip and python distributions

- a python distro has builtin modules in form of shared librarires and python code in `Lib`
   - these modules can be invoked using `python -m <modulename>`
- pip is installed in the builtin modules directory (usually by default), it's bound to a particular python version
- `pip install` installs modules in a python installation subdirectories, `site-packages` (libraries) and `Scripts` (executables pointing at libraries)
   - the `site-packages` are visible to `python -m` (case needs to match) and are globally available for importing when running this python installation
   - `Scripts` allow directly invoking the executables, but need to be added to $PATH for that to work
   - `pip list` lists the site-packages but not the builtin modules

## pip usage

- `pip freeze` prints requirements.txt with all versions specified

### python's "autorun" using siteconfig.py for venv

- <https://docs.python.org/3/library/site.html>

## readline(windows)

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
- python-editline package looks promising, but it's linux only
- it looks like people don't care about builtin repl and use ipython instead (see ipython section below) or ptpython

## ptpython

- [documentation](https://github.com/prompt-toolkit/ptpython)
```
python -m pip install repl
python -m ptpython # start repl
python -m ptipython # start repl with ipython features
```

## ipython

- ipython is a python repl, originally a project for scientific computing that morphed into jupyter, with ipython now just being the python-specific kernel of jupyter
- [ipython repl extensions](https://ipython.readthedocs.io/en/stable/interactive/tutorial.html)
- globally install ipython to use it as a default repl
```
python -m pip install ipython
python -m IPython # start ipython repl
```
- ipython uses prompt-toolkit instead of readline
   - [configuring ipython keybinds](https://ipython.readthedocs.io/en/stable/config/details.html#keyboard-shortcuts)
   - [configuring prompt-toolkit](https://python-prompt-toolkit.readthedocs.io/en/latest/pages/asking_for_input.html#adding-custom-key-bindings)
- [default keybindings](https://ipython.readthedocs.io/en/stable/config/shortcuts/index.html)

## conda

- a cross platform package manager recommended for jupyter
- installs both python and native dependencies as well as deps for other managed languages

### usage

- [installing a package from a file](https://docs.anaconda.com/anaconda/user-guide/tasks/install-packages/#installing-packages-on-a-non-networked-air-gapped-computer)
- [virtual environments](https://conda.io/projects/conda/en/latest/user-guide/concepts/environments.html)
- [build recipes are embedded in packages](https://docs.conda.io/projects/conda-build/en/latest/concepts/recipe.html#conda-build-recipes)
