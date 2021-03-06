## nix

### Basics

 * [understanding nix](https://nixos.org/nixos/nix-pills/why-you-should-give-it-a-try.html)
 * Other docs:
     * [nix manual](https://nixos.org/manual/nix/unstable/)
     * [using nixpkgs - functions, conventions, derivations, build-system-support, etc](https://nixos.org/manual/nixpkgs/unstable/)
     * [nix learn - a list of official guides on common tasks and links to other docs](https://nixos.org/learn.html)
     * [wiki](https://nixos.wiki/)
     * [opinionated guidelines and guides - pretty good](https://nix.dev/)
     * [new nix shell commands (todo)](https://blog.ysndr.de/posts/guides/2021-12-01-nix-shells/)
     * [nix flakes](https://blog.ysndr.de/posts/internals/2021-01-01-flake-ification/)
     * [nix develop - replacement for standard nix tooling optimized for development](https://gitlab.com/mightybyte/nix-develop)
 * Dependencies between nix packages are hardcoded
     * Updating a lib requires rebuild of dependencies
     * But packages can coexist thanks to this
     * And less reliance on $PATHs
 * Nix profile - a composition of nix components into a single `PATH` entry that has all the software with dependencies properly resolved
     * By default `~/.nix-profile` itself is a symbolic link to `/nix/var/nix/profiles/default`
     * Generation - nix profile directory at a particular point in time
 * Nix derivation - a build action for a component(`.drv` file)
 * Nix expression - 2 meanings: language expression and language expression returning a derivation
 * Nix channel
     * Added to `NIX_PATH`
 * Closure - recursive list of dependencies necessary to use the derivation
 * Differences from regular packaging systems:
     * nix-build is deterministic, uses sandboxing
     * nix doesn't use standard paths and executables, it uses non-standard environments and CC-wrappers
         * this has consequences for tooling: [nix_editors](../cpp/nix_meson_dev_env.md)
         * supposedly CC-wrappers are just for badly defined packages and can be disabled in favour of standard pkgconfig mechanisms
* configuration: <https://www.mankier.com/5/nix.conf> in `~/.config/nix/nix.conf`

### Usage

 * `ls -l ~/.nix-profile/` - inspect current profile, find it in store
    * root of a mini-unix system that has installed packages, with bin, etc and other directories
 * `source ~/.nix-profile/etc/profile.d/nix.sh` - import default profile into bash
 * `nix-env -i` - install in current profile
 * `nix-env --list-generations` -list versions of current profile
 * `nix-env -q` - list installed derivations (packages)
 * `nix-env -qa` - list available packages from the current channel
 * `nix-env --rollback` - switch to previous version
 * `nix-env -G 3` - switch to specified version
 * `nix-shell expr` - switch to shell needed to build the derivation specified by expression
   * `nix-shell -A pkgname <nixpkgs>` - nix-shell for building packages - selects pkgname from map returned by `<nixpkgs>`
   * `nix-shell --pure` - environment which tries to pull as little as possible
   * `nix-shell -p package1 package2` - not a derivation development shell - instead a shell with listed packages from nix packages
 * `nix-store -q --references $(which hello)` - query which derivations a derivation (follows links to find which derivation the file is part of) uses 
 * `nix-store -q --referrers $(which hello)` - query which derivations use contents of derivation
 * `nix-store -q --references path.drv` - build time dependencies of derivation
 * `nix-store -q --references $(nix-store --realise path.drv)` - runtime dependencies of a package built with path.drv derivation
 * `nix-store -qR $(which hello)` - list derivations in a closure
 * `nix-store -q --tree $(which hello)` - show a tree of derivations in a closure
 * `nix-store -q --tree ~/.nix-profile` - show currently used tree
 * `nix repl` - repl
    * `expr`        Evaluate and print expression
    * `x = expr`  Bind expression to variable
    * `:a expr`     Add attributes from resulting set to scope
    * `:b expr`     Build derivation
    * `:i expr`     Build derivation, then install result into current profile
    * `:l <path>`     Load Nix expression and add it to scope
    * `:p expr`     Evaluate and print expression recursively
    * `:q`            Exit nix-repl
    * `:r`            Reload all files
    * `:s expr`     Build dependencies of derivation, then start nix-shell
    * `:t expr`     Describe result of evaluation
    * `:u expr`     Build derivation, then start nix-shell
 * `nix show-derivation` - pretty print a .drv file
 * `nix-channel --update` - update the channel
 * `nix-channel --update; nix-env -u` - update channel and default profile with latest versions of packages

#### utilities

* <https://github.com/nix-community/lorri> - nix-shell env outside of nix shell
* <https://github.com/nix-community/nix-direnv> - alternative to lorri, but without a daemon
* <https://github.com/numtide/devshell> - nix-shell alternative?
* cached-nix-shell
  * stored cached env values in ~/.cache/cached-nix-shell
  * use latest .env file to reload the environment

#### nix garbage and updates

* nix gc roots are listed in `/nix/var/nix/gcroots/` (anything under that directory)
    * `profiles` symlink links to profiles and profile generations
    * `auto/` subdir, where roots are symlinks to `./result` subdir on nix-build
        * the preferred way to clean those is to remove the `./result symlink`
        * alternatively you can remove from auto, but the symlink will be dangling
* complete cleanup:
```
nix-channel --update # update nix-channel
nix-env -u --always # update all derivations in profile
rm /nix/var/nix/gcroots/auto/* # remove all autogenerated roots
nix-collect-garbage -d # remove old generations of all profiles from roots and run gc
```

#### Nix search path

* `NIX_PATH` is used when searching for expression files
* `nix*` commands accept -I which prepends to nix_path on startup
* `<name>` will search for files/dirs with this name in `$NIX_PATH` and return the path
* `<name>` will also search for `name=/mypath` in `$NIX_PATH`, and return `/mypath`
* In general, prefer using relative paths when possible
* `Nix-env` doesn???t use `$NIX_PATH` variable, it uses `~/.nix-defexpr` instead
    * This can cause different behaviour between nix-env and nix-build
    * Why is nix-env having this different behavior? I don't know specifically by myself either, but the answers could be:
      * `nix-env` tries to be generic, thus it does not look for nixpkgs in `NIX_PATH`, rather it looks in ~/.nix-defexpr.
      * `nix-env` is able to merge multiple trees in `~/.nix-defexpr` by looking at all the possible derivations
      * It may also happen to you that you cannot match a derivation name when installing, because of the derivation name vs `[-A]` switch described above. Maybe `nix-env` wanted to be more friendly in this case for default user setups.
      * It may or may not make sense for you, or it's like that for historical reasons, but that's how it works currently, unless somebody comes up with a better idea.

### Language

```nix
# expressions
5*10
> 50
6/3 # relative path
> /home/dariusza/6/3
6/ 3 
> 2

a-b  # identifier
a - b # subtraction
2-3 # subtraction

# strings
''string''
"string" # exactly the same
foo="strval" # repl only syntax, foo available for next expressions
"${foo}" # interpolated string
''${foo}'' # interpolated string

"\${foo}" # escaping
> "${foo}"
''test ''${foo} test'' # escaping
> "test ${foo} test"

#paths
/tmp  # a path must contain at least one slash.
./builder.sh # relative paths are made absolute at parse time relative to the directory of the nix expression that contained it (or relative to working dir within repl)
~ # refers to user home directory
<nixpkgs> # refers to the directory nixpackages in NIX_PATH

# collections
[ 2 "foo" true (2+3) ] # list
> [ 2 "foo" true 5 ]

s = { foo = "bar"; a-b = "baz"; "123" = "num"; } # map (only string keys allowed)
s.a-b 
> "baz"
s."123"
> "num"
s ? foo # check if contains foo
> true

rec { a = 3; b = a+4; } # recursive attribute set - can refer to other keys in a map
> { a = 3; b = 7; }

Attrset1 // attrset2 - merge 2 sets with set 2 taking precedence

builtins.toString { outPath = "foo"; } # Nix does the "set to string conversion" as long as there is the outPath attribute

# if
if 4 > 3 then "yes" else "no" # if expression
> "yes"
# let
let a = "foo"; in a # let in expression, in specifies the return value
> "foo"
let a = 3; in let b = 4; in a + b # can be nester
> 7
let a = 4; b = a + 5; in b # can refer to other variables in let expression, can't refer to let variables outside expression
> 9

# with
longName = { a = 3; b = 4; }
with longName; a + b
> 7
let a = 10; in with longName; a + b # with doesn't shadow variables in scope
> 14
let a = 10; in with longName; longName.a + b # can refer to with scope explicitly
> 7

# inherit
with longName; { inherit a b } # declare a=a and b=b within the set
{ inherit (longName) a b } # declare a=a and b=b, where longName is a source set

let a = builtins.div 4 0; b = 6; in b  # nix is lazy, so a is never evaluated here because it's unused
6

# functions
double = x: x*2 # lambda funcion assigned to a variable
double 3 # no parens on fn call
> 6
mul = a: (b: a*b) # multi arg fn
mul = a: b: a*b # equivalent
mul 3 # partial application
mul 3 4 # multi arg fn call
mul (6+7) (8+9) # parens - eval order

# functions - argsets 
mul = s: s.a*s.b # fn taking an attrset as a single arg
mul { a = 3; b = 4; }
> 12
mul = { a, b }: a*b # equivalent fn, pattern matching (destructuring) over the set taken as first and only argument, this is the only place with commas in the language
mul { a = 3; b = 4; } # can only call mull with the exactly a,b provided, not more, not less, can't use mul 3 4 syntax to call the function
> 12
mul = { a, b ? 2 }: a*b # default arg, allows calling with just a
mul = { a, b, ... }: a*b # allow other args, skip them in fn implementation
mul = s@{ a, b, ... }: a*b*s.c # get the map with all the optional arguments, refer to them with @-pattern

# imports
a.nix contents: 3
b.nix contents: 4
mul.nix contents: a: b: a*b
a = import ./a.nix # evaluate file as an expression (including function)
b = import ./b.nix # scope of the imported file doesn't inherit the scope of the importer
mul = import ./mul.nix
mul a b
> 12

import a.nix {} # import always takes 1 arg, so the result of this is calling a.nix on an empty set
```

#### builtin functions 

 * <https://nixos.org/nix/manual/#ssec-builtins>
 * `builtin.trace message: value: # builtin fn for debugging` - prints a msg and returns a value

### derivation fundamentals

* build process: 
    * nix-expressions - source code in `.nix` files or repl
    * `.drv` files - created with `nix-instantiate` (evaluation of derivation function)
    * outPaths with the built packaged - created with `nix-store --realise`
    * `nix-build` does both `nix-instantiate` and `nix-store --realise` and creates a symlink `./result` to the outPath
* A derivation from a Nix language view point is simply a set, with some attributes. Therefore you can pass the derivation around with variables like anything else. 
* The derivation function receives a set as first argument. This set requires at least the following three attributes:
   * name: the name of the derivation. In the nix store the format is hash-name, that's the name.
   * system: is the name of the system in which the derivation can be built. For example, x86_64-linux.
   * builder: it is the binary program that builds the derivation. 
      * builder receives env variables which are passed in as the remaining arguments to derivation
      * builder doesn't have access to many usual env variables as they're overridden to prevent build process from depending on nondeterministic environment, like $HOME, $PATH, etc
      * $PWD and $TMP point to current temporary dir where derivation is built
   * every attribute is passed as an env variable to the builder
     * if arguments are paths, they'll be copied into nix store and that path is passed as a value and added as a dependency of the derivation
     * if the argument is a derivation, that derivation is built and added as a dependency
     * true is passed as string 1, false as empty string
   * returns a derivation set
     * `builtins.attrNames derivation { name = "myname"; builder = "mybuilder"; system = "builtins.currentSystem"; }`
     * `> [ "all" "builder" "drvAttrs" "drvPath" "name" "out" "outPath" "outputName" "system" "type" ]`
   * more info: <https://nixos.org/nix/manual/#ssec-derivation>
* loading derivation objects: `:l <nixpkgs>`, you can then refer to derivations by name
    * depending on other derivations - just use their path via derivation object (which is implicitly converted to their path) `derivation { name = "myname"; builder = "${coreutils}/bin/true"; system = builtins.currentSystem; }`
* `.drv` format <https://nixos.org/~eelco/pubs/phd-thesis.pdf> summary:
   * The output paths (there can be multiple ones). By default nix creates one out path called "out".
   * The list of input derivations. It's empty because we are not referring to any other derivation. Otherwise, there would a list of other .drv files.
   * The system and the builder executable.
   * Then a list of environment variables passed to the builder.
* executing derivation builder to realize (nix-store --realise) `.drv`:
   * if output path already exists, it's removed
   * each output path is a hash of all the inputs (includes source files, etc)
   * A log of the combined standard output and error is written to `/nix/var/log/nix`.
   * If the build was successful, Nix scans each output path for references to input paths by looking for the hash parts of the input paths. Since these are potential runtime dependencies, Nix registers them as dependencies of the output paths.
   * the build dir is removed once the build is done, and neutral groups and ownership settings are applied to the generated files
* nix-shell
    * a tool which builds all the dependencies of a derivation and sets the environmnent for running of the builder script
    * notably, it's not pure by default and also it runs .bashrc and /env/bashrc
    * it's apparently a good idea to split up the setup part of build script so that it can be easily run in nix-shell


#### simple derivation example

simple.nix
```
with (import <nixpkgs> {}); # load expressions from nixpkgs
derivation {
  name = "simple";
  builder = "${bash}/bin/bash";
  args = [ ./simple_builder.sh ]; # run our script in bash
  inherit gcc coreutils; # pass gcc and coreutils variables to script
  src = ./simple.c; # pass the source file
  system = builtins.currentSystem;
}
```

simple_build.sh
```
export PATH="$coreutils/bin:$gcc/bin"
mkdir $out
gcc -o $out/simple $src
```


output - /nix/store/2xwdcfnf4157fqxcf7bnjsbdr6pfc2v3-simple.drv
```
{
  "/nix/store/2xwdcfnf4157fqxcf7bnjsbdr6pfc2v3-simple.drv": {
    "outputs": {
      "out": {
        "path": "/nix/store/iyv05vhvm3yrvrw04a6l5dnq6d1vllp1-simple"
      }
    },
    # inputs of derivation, copied into store
    "inputSrcs": [
      "/nix/store/4wpv68jkvw9nbnh4qcblnfvdfnn74kbn-simple_builder.sh",
      "/nix/store/9sll05wmn9b05f0pm1pwfmmq92fg0syh-simple.c"
    ],
    # dependencies on other derivations
    "inputDrvs": {
      "/nix/store/chvalchsi9g4zk3np302003630vb45qh-gcc-wrapper-8.3.0.drv": [
        "out"
      ],
      "/nix/store/ij7yr26mjpnpj78min707x88cbg35sl8-bash-4.4-p23.drv": [
        "out"
      ],
      "/nix/store/pvhv8cm3g8hn8prjjiw3agqbycr6jbr6-coreutils-8.31.drv": [
        "out"
      ]
    },
    "platform": "x86_64-linux",
    "builder": "/nix/store/xb062l4b76zyhq6grqf4iyfdikkpg8fl-bash-4.4-p23/bin/bash",
    # arguments to builder
    "args": [
      "/nix/store/4wpv68jkvw9nbnh4qcblnfvdfnn74kbn-simple_builder.sh"
    ],
    # env set for builder
    "env": {
      "builder": "/nix/store/xb062l4b76zyhq6grqf4iyfdikkpg8fl-bash-4.4-p23/bin/bash",
      "coreutils": "/nix/store/d7hlkykjjqs3f200jnmjm1y2hzgvbqa8-coreutils-8.31",
      "gcc": "/nix/store/47hi526v96skkwgdj7d9c31ccd1xivrx-gcc-wrapper-8.3.0",
      "name": "simple",
      "out": "/nix/store/iyv05vhvm3yrvrw04a6l5dnq6d1vllp1-simple",
      "src": "/nix/store/9sll05wmn9b05f0pm1pwfmmq92fg0syh-simple.c",
      "system": "x86_64-linux"
    }
  }
}
```

### Nixpkgs

* Nixpkgs `default.nix` (what???s returned for import `<nixpkgs>`) is a function that accepts some parameters and returns a set of all packages
* Parameters:
    * System, default to current system
        * Enables cross compiling, example:
        * `nix-build -A psmisc --argstr system i686-linux`
    * Config, by default null, will be read from `~/.nixpkgs/config.nix`
        * `config.packageOverrides` - configures fixed point overriding
* Structure
    * The `all-packages.nix` is then the file that composes all the packages. 
    * `pkgs/` dir with package functions or expressions (if file is a function, it???ll be evaluated with default params)
* installing a package with pinned nixpkgs:
    * `nix-env -if https://github.com/xzfc/cached-nix-shell/archive/refs/tags/v0.1.5.tar.gz --arg pkgs 'import ./cached/nix/packages.nix {}'`

#### Nix packaging patterns

* `nix*` command will run execute `default.nix` by default
* `nix*` -A attrname will refer to a attr in attribute set (map) returned by executed nix expression, useful for the single repository pattern (when expr returns an attrset)
* Single repository pattern (repository expression):
    * Nix has a single repository `<nixpkgs>` with all descriptions of all packages
    * Top level expression is in default.nix and it returns an attrset of name->package pairs
* The inputs pattern
    * Define packages as parametrized functions in order to allow customizability
    * Make package expression independent of the repository, so that they can be called individually
    * Example: <https://gist.github.com/lethalman/734b168a0258b8a38ca2/revisions>
* The `callPackage` pattern
    * Simplifies the single repository/input pattern by simplifying the repeating dependency declarations of packages
    * `callPackage` function, which:
        * Imports given expression, which in return returns a funciton
        * Determines names of it???s arguments
        * Passes the default arguments from the repository set and lets us override those arguments
    * Example: `callPackage mypackage.nix {repo-overrides}`
* The override pattern:
    * A way of overriding repository package definition, without having to redefine the whole thing from scratch and without a risk of unintentionally diverging
    * Example `mypackage.override {package-overrides}`
    * Implementation simply overrides the args and puts them in `.override` field
    * With some standard arg conventions we can create predefined overrides like:
        * `debugVersion (applyPatches [ ./patch1.patch ./patch2.patch ] drv)` which enable debug flags and apply patches
    * Example: <https://nixos.wiki/wiki/Debug_Symbols>
    * in nix packages lib.makeOverridable defines the methods for overrriding
    * mkDerivation uses lib.makeOverridable
* Package setup hooks pattern:
    * Define reusable scripts for use in packages dependant on particular package, to abstract away the repeated work the dependencies would have to do
    * Setup hooks in nixpkgs: <https://nixos.org/nixpkgs/manual/#ssec-setup-hooks>
    * Most important wrapper: cc wrapper
        * Handles dependency finding for nix packages using `NIX_CFLAGS_COMPILE, NIX_LDFLAGS`
* Overriding packages using fixed point
    * [overrides in nixpkgs](https://nixos.org/manual/nixpkgs/stable/#chap-overrides)
    * definition - change how the package derivation is defined
```
helloWithDebug = pkgs.hello.overrideAttrs (oldAttrs: rec {
  separateDebugInfo = true;
});
```
    * Enables overriding a package in a way that it will be picked as a dependency instead of the original
    * Explanation of the implementation: 
        * Function returning an overridable map, taking a single arg - map of packages `pkgs = self: { a = 3; b = 4; c = self.a+self.b; }`
        * Function taking a function and applying it to itself recursively -`fix = f: let result = f result; in result`
        * Fix pkgs will stabilise at `pkgs = { a = 3; b = 4; c = 7; }`
        * <http://r6.ca/blog/20140422T142911Z.html>
        * <https://nixos.org/nixos/nix-pills/nixpkgs-overriding-packages.html#idm140737315550816>
* [Overlays](https://nixos.org/manual/nixpkgs/unstable/#chap-overlays)
    * [wiki](https://nixos.wiki/wiki/Overlays)
    * definiton - change all the packages having a dependency to a different dependency
```
# overlay-def.nix
self: super:
{
  boost = super.boost.override {
    python = self.python3;
  };
  rr = super.callPackage ./pkgs/rr {
    stdenv = self.stdenv_32bit;
  };
}
```
    * hooking it in:
```
pkgs = import nixpkgs {
  overlays = [
    (import ./nix/overlay-def.nix) # () make sure the import is interpreted as a single list element
  ];
};
```
* packaging functions
  * [pkgs.buildEnv](https://github.com/NixOS/nixpkgs/blob/master/pkgs/build-support/buildenv/default.nix) 
    * creates a tree of symlinks which make a "nix environment"
    * [example usage](https://nixos.org/manual/nixpkgs/stable/#sec-building-environment)
  * stdenv
    * c++ build environment context (gnu, compiler, etc)
    * Docs: <https://nixos.org/nixpkgs/manual/#chap-stdenv>
    * [pill](https://nixos.org/guides/nix-pills/fundamentals-of-stdenv.html)

### Setup 

* install nix
```
curl -L https://nixos.org/nix/install | sh
# set up configuration
mkdir -p ~/.config/nix
echo "require-sigs = false" >> ~/.config/nix/nix.conf
echo "max-jobs = auto" >> ~/.config/nix/nix.conf
# initialize channel
nix-channel --update nixpkgs
```
* add to bashrc:
```
# nix if present
if [ -f /home/dariusza/.nix-profile/etc/profile.d/nix.sh ]; then
    source /home/dariusza/.nix-profile/etc/profile.d/nix.sh
fi
```

### Rust

packages.nix:
```
# default evaluation of nix-packages for the project
{ sources ? import ./sources.nix
, mozilla ? import sources.nixpkgs-mozilla
, pkgs ? import sources.nixpkgs {
  overlays = [
    mozilla
    (self: super: let pinned = (super.rustChannelOf { date = "2022-01-12"; channel = "nightly"; }) in {
      rustc = pinned.rust; # todo: why not rustc?
      rust-src = pinned.rust-src;
      cargo = pinned.cargo;
    })
  ];
}
}: pkgs
```

shell.nix
```
{ pkgs ? import ./nix/packages.nix {}
, sources ? import ./nix/sources.nix
}:
let
  crate2nix = import sources.crate2nix {inherit pkgs;};
in
pkgs.mkShell {
  nativeBuildInputs = (
    with pkgs; [
      rustc
      rust-src
      cargo
    ]
    );
  RUST_SRC_PATH="${pkgs.rust-src}/lib/rustlib/src/rust/library/";
}
```
