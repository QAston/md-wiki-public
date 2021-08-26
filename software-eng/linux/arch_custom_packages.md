### set up build tools

* pacman -S base-devel 

### Setup ABS

 * Docs
     * <https://wiki.archlinux.org/index.php/Arch_Build_System>
     * <https://wiki.archlinux.org/index.php/Makepkg>
     * <https://wiki.archlinux.org/index.php/PKGBUILD>
     * <https://wiki.archlinux.org/index.php/Patching_packages>
 * sudo pacman -S asp subversion
 * Add IgnoreGroup = modified to /etc/pacman.conf
 * Add Packages to ignore updates for to /etc/pacman.conf IgnorePkgs, or use --assume-installed flag for pacman
 * Modify `/etc/makepkg.conf`
    * set MAKEFLAGS="-j12"
    * change -march=x86_64 to march=native # note this will break executables if you upgrade hardware
 * Downloading PKGBUILD
     * mkdir abs
     * cd abs
     * `asp export packagename`
 * Building
     * Add groups=(modified) to pkgbuild if you want to ignore the package during updates
     * `makepkg -s --skippgpcheck`
     * `pacman -U <buildpackage>`
 * Debug builds (based on <https://wiki.archlinux.org/index.php/Debug_-_Getting_Traces>)
     * Add options=(debug !strip) to pkgbuild
     * Add -DCMAKE_BUILD_TYPE=RelWithDebInfo (or Debug or debugfull for kde apps)
     * If above doesn’t work try adding to build() function
         * export CFLAGS="$CFLAGS -O0 -fbuiltin -g"
         * export CXXFLAGS="$CXXFLAGS -O0 -fbuiltin -g"

### Setup AUR

 * Docs
     * <https://wiki.archlinux.org/index.php/AUR_helpers>
     * <https://wiki.archlinux.org/index.php/Arch_User_Repository>
     * <https://wiki.archlinux.org/index.php/Aurweb_RPC_interface>
 * Build/Install
     * Mkdir ~/aur
     * Find a package in <https://aur.archlinux.org/>
     * Git clone git url or using auracle
     * `makepkg -s` in the directory
     * `pacman -U` zipfile
 * Use pacaur to automate AUR:
     * First need to install auracle-git and expac-git from AUR
     * Build and install pacaur package
     * add to ~/.bashrc: `export EDITOR=neovim`

### Set up appimages

1. sudo pacman -S fuse
2. wget appimage into portable
3. chmod a+x appimage
4. ./appimage
5. unpacking : <https://github.com/AppImage/AppImageKit/wiki/FUSE#fallback>

### Setup custom repositories - if needed

 * Docs
     * <https://wiki.archlinux.org/index.php/Unofficial_user_repositories#qt-debug>
     * <https://wiki.archlinux.org/index.php/Pacman/Package_signing#Adding_unofficial_keys>
     * <https://sysdfree.wordpress.com/2018/02/25/177/> - artix repositories compared to arch
 * Add an entry to /etc/pacman.conf
 * Import key: sudo pacman-key --recv-keys D6A1C70FE80A0C82
 * Sign key: sudo pacman-key --lsign-key D6A1C70FE80A0C82