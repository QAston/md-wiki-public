## terminals

* most unix software was written against very old hardware for IO - terminals like VT100, VT220, etc
* this is why their configuration refers to very old and archane configuration for input and output
    * it's actually because that configuration is specific to the machines the software was written for at the time
    * this is why readline, vi, emacs and vim are so obscure to configure
* to understand the range of input/output expected/emitted you should look at the manual of machines the software supported
    * in vim/readline you actually bind the handlers to the terminal output sequences of a historical terminal
* terminal emulators like xterm pretend to be that old hardware and provide a way to map the new hardware to the old one, as well as choosing which hardware they pretend to be
* this is why unix software doesn't support modern keyboards - it supports the keyboard output of old terminals
* xterm extens the emulated functions with new stuff by handling new output codes and providing new input as long as the applications are new enough to be accepting and outputting these
    * mouse click escape sequence handling, window labels, etc
* application is told about the terminal providing the io in the config, usually `$TERM` env variable
* most modern terminals try to be xterm-compatible and thus set the TERM env variable to `xterm` (or variant thereof like `xterm-256color`)
    * what terminal supports is defined in terminfo and termcap for the given TERM entry
* apparently most emulators don't emulate most of xterm and lack in features
* windows terminal sequences: https://docs.microsoft.com/en-us/windows/console/console-virtual-terminal-sequences
* explanation of how terminal works on linux <https://tldp.org/HOWTO/pdf/Keyboard-and-Console-HOWTO.pdf>
* guide on keycodes (input sequences) from mintty: <https://github.com/mintty/mintty/wiki/Keycodes>
    * ctrl-q in mintty msys then press shows the keycode (filtered through readline?, which seems to eat C1 control block (ctrl+shift+ascii characters))
* mintty doc on keycodes: <https://github.com/mintty/mintty/wiki/Keycodes>
* clink doc on keycodes on standard pc keyboard <https://chrisant996.github.io/clink/clink.html#miscellaneous> (search for Binding special keys)
* [an old guide on how terminals used to work, largely obsolete](https://tldp.org/HOWTO/Text-Terminal-HOWTO.html#toc16)
* [ncurses terminology faq](https://invisible-island.net/ncurses/ncurses.faq.html#terminology)
* [xterm faq](https://invisible-island.net/xterm/xterm.faq.html)
* notations for keycodes
    * `ESC` a.k.a `^[` a.k.a binary: 0x1b - ESC ASCII control character, nothing to do with the ESC key
        * see <https://en.wikipedia.org/wiki/Control_character> for other named ctrl characters
        * ctrl character is generated by pressing CTRL and a ASCII sign which when binary-anded with 0b11111 gives the control character
        * ctrl-space is the exception, it produces ^"
    * `^[` - ascii control range - caret notation
    * `\e` - same as `ESC`
    * 0x1b - same as `ESC` (also `\u001b`)
    * some named sequences: CSI( `ESC [`), SS3 (`ESC O`), can be found here <https://invisible-island.net/xterm/ctlseqs/ctlseqs.html#h3-Controls-beginning-with-ESC>
    * other characters are usually verbatim ascii bytes separated by space, for example: `CSI 1 1 ~`
    * unicode: ` U+0000???U+001F` (C0 controls), `U+007F` (delete), and `U+0080???U+009F` (C1 controls)`

## terminal emulation libraries

* see [readline](../tools/readline.md) for info on terminal line-editing software
* keyboard/mouse io: [libtermkey](https://www.leonerd.org.uk/code/libtermkey/) and [libtickit](https://www.leonerd.org.uk/code/libtickit/) (used by nvim)
* terminal emulator as a library [libvterm](https://www.leonerd.org.uk/code/libvterm/) (used by nvim)
* terminfo reader/writer library [unibilium](https://github.com/neovim/unibilium) (used by nvim)
* curses - it's an old api for drawing tuis on unixes, originally used termcap term database
* [ncurses](https://invisible-island.net/ncurses/announce-6.3.html#h2-overview) 
    - terminal tui library which uses the terminfo database format, a newer implementation of curses
    - [documentation index](https://invisible-island.net/ncurses/)

## terminal feature databases (termcap and terminfo)

* `infocmp` prints info about current terminal ($TERM variable) - terminfo decompiler
* [tic](https://linux.die.net/man/1/tic) - terminfo compiler
* [tack](https://linux.die.net/man/1/tack) - terminfo action checker
* [toe](https://linux.die.net/man/1/toe) - list terminfo entries
