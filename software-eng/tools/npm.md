# NPM

Files: <https://docs.npmjs.com/files/folders>

## setup windows

download zip to portable and add `C:\portable\node` to `PATH_APPEND_WINDOWS`

## setup linux

```
sudo pacman -S nodejs npm
```
add configuration to where install packages
```
PATH="$HOME/.npmconfig/bin:$PATH"
export npm_config_prefix="$HOME/.npmconfig"
```

## usage

### installing packages

Global instals:

- `npm instal -g packagename` always installs in users global path directory (/usr/local/bin in linux, and in %APPDATA%/npm on windows), shared by all npm installs, **don't do this**

Local installs (directory with package.json, a.k.a `$prefix` directory):

- `npm install <package-name> --save-dev` - adds to devDependencies, which is for devtools (like for build), available from `$prefix/node_modules/.bin`
- `npm install <package-name>` (`--save` is implicitly default) adds to dependencies, available from `$prefix/node_modules/.bin`

Npx:

- Runs binaries found in local installs, global installs, or downloads and runs from caches if none are found
- `npx [options] <command>[@version] [command-arg]...`
- see <https://www.npmjs.com/package/npx>

### scripts

- describe shell commands to execute when `npm run <key>` is used
- some of the `<key>`s are reserved to be executed by npm with other commands like `npm install`
    - npm install
        - preinstall
        - install
        - postinstall
        - prepublish
        - preprepare
        - prepare
        - postprepare
    - npm start
        - prestart
        - start
        - poststart
    - npm stop
        - prestop
        - stop
        - poststop
    - npm test
    - npm restart
- default scripts:
```
{"start": "node server.js"} // if there's a server.js
{"install": "node-gyp rebuild"} // if there's a binding.gyp
```

### yarn 

- yarn does the same job as npm, uses the same package.json format and shares most of the commands
- install in the current npm global install directory using (it's fine to install globally for this particular package because it's just as rare for the project to need an old version of yarn as it is for the project to need an old version of npm)
```
npm -g install yarn
```
- npx will use yarn if yarn.lock is present

### nvm

- like pyenv but for selecting a verion of node.js?
- linux/macos <https://github.com/nvm-sh/nvm>
- windows <https://github.com/coreybutler/nvm-windows>