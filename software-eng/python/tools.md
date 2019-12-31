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


## Pip

- `pip freeze` prints requirements.txt with all versions specified