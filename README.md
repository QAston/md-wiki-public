## md-wiki-public

A public mirror of parts my markdown docs.

### Why

I've decided to share some of the private documentation I'm maintaining for my own purposes (I'm too lazy to remember things), so that people working on similar things can hopefully reuse my solutions instead of having to come up with them on their own like I had to. It's a way to give back to the community that's less time consuming than writing blogposts or answers on obscure forums but still gets the job done and the documents themselves are more likely to remain up-to-date.  

Relevant XKCD:

![](https://imgs.xkcd.com/comics/wisdom_of_the_ancients.png)

### Highlights

I've documented the configuration of my development environment which I'm very fond of:
- [windows environment](software-eng/windows/devenv.md#setup-steps)
- [msys2](software-eng/windows/msys2.md)
- [wsl2](software-eng/windows/wsl2_artix.md)
- [native linux host for the wsl2 container](software-eng/linux/native_wsl_host.md)

### Caveats

- the documentation is as opinionated as it gets, to the points of having references to paths on my filesystem, and it's going to stay that way, as making it generic is not the purpose of this repository
- the repo is best browsed within vscode markdown text editor, as I don't really check for how the rendered html looks like because I don't use the html renderer
- the documentation might link to documentation not contained in this repository, or to external files - if you're following a guide and encounter a missing file you want to use, please create a github issue
    - some of the referenced repositories: [wslconfig](https://github.com/QAston/wslconfig), [shimzon](https://github.com/QAston/shimzon), [portable_env](https://github.com/QAston/portable_env)
- if the documentation is out of date or wrong feel free to create a github issue with a correction
- this repository is not maintaining history (that's the job of the master repo), so don't base your work on top of it because the history tree will certainly change from under you
- there's no commit hygiene as there's no need for it in this particular repository
