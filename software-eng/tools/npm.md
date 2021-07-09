# NPM

Files: <https://docs.npmjs.com/files/folders>

## setup

download zip to portable and add `C:\portable\node` to `PATH_APPEND_WINDOWS`

## usage

### installing packages

Global instals:

- `npm instal -g packagename` always installs in users global path directory (/usr/local/bin in linux, and in %APPDATA%/npm on windows), shared by all npm installs, **don't do this**

Local installs (directory with package.json, a.k.a `$prefix` directory):

- `npm install <package-name> --save-dev` - adds to devDependencies, which is for devtools (like for build), available from $prefix/node_modules/.bin
- `npm install <package-name>` adds to dependencies, available from $prefix/node_modules/.bin

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