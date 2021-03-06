# haskell-language-server

[![License Apache 2.0][badge-license]][license]
[![CircleCI][badge-circleci]][circleci]

[badge-license]: https://img.shields.io/badge/license-Apache2-green.svg?dummy
[license]: https://github.com/haskell/haskell-language-server/blob/master/LICENSE
[badge-circleci]: https://img.shields.io/circleci/project/github/haskell/haskell-language-server/master.svg
[circleci]: https://circleci.com/gh/haskell/haskell-language-server/

Integration point for [ghcide](https://github.com/digital-asset/ghcide) and [haskell-ide-engine](https://github.com/haskell/haskell-ide-engine). One IDE to rule
them all. Read the [project's
background](https://neilmitchell.blogspot.com/2020/01/one-haskell-ide-to-rule-them-all.html).

This is *very* early stage software.

- [Haskell Language Server (HLS)](#haskell-language-server)
  - [Installation](#installation)
    - [Installation from source](#installation-from-source)
      - [Common pre-requirements](#common-pre-requirements)
      - [Linux-specific pre-requirements](#linux-specific-pre-requirements)
      - [Windows-specific pre-requirements](#windows-specific-pre-requirements)
      - [Download the source code](#download-the-source-code)
      - [Building](#building)
        - [Install via cabal](#install-via-cabal)
        - [Install specific GHC Version](#install-specific-ghc-version)
  - [Project Configuration](#project-configuration)
  - [Contributing](#contributing)
    - [It's time to join the project!](#its-time-to-join-the-project)

## Installation

For now only installation from source is supported.

### Installation from source

#### Common pre-requirements

- `stack` or `cabal` must be in your PATH
  - You need stack version >= 2.1.1 or cabal >= 2.4.0.0
- `git` must be in your PATH
- The directory where `stack`or `cabal` put the binaries must be in you PATH:
  - For stack you can get it with `stack path --local-bin`
  - For cabal it is by default `$HOME/.cabal/bin` in linux and `%APPDATA%\cabal\bin` in windows.

Tip: you can quickly check if some command is in your path by running the command.
If you receive some meaningful output instead of "command not found"-like message
then it means you have the command in PATH.

#### Linux-specific pre-requirements

On Linux you will need install a couple of extra libraries (for Unicode ([ICU](http://site.icu-project.org/)) and [NCURSES](https://www.gnu.org/software/ncurses/)):

**Debian 9/Ubuntu 18.04 or earlier**:

```bash
sudo apt install libicu-dev libtinfo-dev libgmp-dev
```

**Debian 10/Ubuntu 18.10 or later**:

```bash
sudo apt install libicu-dev libncurses-dev libgmp-dev
```

**Fedora**:

```bash
sudo dnf install libicu-devel ncurses-devel # also zlib-devel if not already installed
```

#### Windows-specific pre-requirements

In order to avoid problems with long paths on Windows you can do either one of the following:

1. Clone the `haskell-language-server` to a short path, for example the root of your logical drive (e.g. to
   `C:\hls`). Even if you choose `C:\haskell-language-server` you could hit the problem. If this doesn't work or you want to use a longer path, try the second option.

2. If the `Local Group Policy Editor` is available on your system, go to: `Local Computer Policy -> Computer Configuration -> Administrative Templates -> System -> Filesystem` set `Enable Win32 long paths` to `Enabled`. If you don't have the policy editor you can use regedit by using the following instructions [here](https://docs.microsoft.com/en-us/windows/win32/fileio/naming-a-file#enable-long-paths-in-windows-10-version-1607-and-later). You also need to configure git to allow longer paths by using unicode paths. To set this for all your git repositories use `git config --system core.longpaths true` (you probably need an administrative shell for this) or for just this one repository use `git config core.longpaths true`.

In addition make sure `haskell-language-server.exe` is not running by closing your editor, otherwise in case of an upgrade the executable can not be installed.

#### Download the source code

```bash
git clone https://github.com/haskell/haskell-language-server --recurse-submodules
cd haskell-language-server
```

#### Building

Note, on first invocation of the build script with stack, a GHC is being installed for execution.
The GHC used for the `install.hs` can be adjusted in `./install/shake.yaml` by using a different resolver.

Available commands can be seen with:

```bash
stack ./install.hs help
```

Remember, this will take time to download a Stackage-LTS and an appropriate GHC for build
haskell-language-server the first time.

##### Install via cabal

The install-script can be invoked via `cabal` instead of `stack` with the command

```bash
cabal v2-run ./install.hs --project-file install/shake.project <target>
```

or using the existing alias script

```bash
./cabal-hls-install <target>
```

Running the script with cabal on windows requires a cabal version greater or equal to `3.0.0.0`.

For brevity, only the `stack`-based commands are presented in the following sections.

##### Install specific GHC Version

Install haskell-language-server for the latest available and supported GHC version (and hoogle docs):

```bash
stack ./install.hs hls
```

Install haskell-language-server for a specific GHC version (and hoogle docs):

```bash
stack ./install.hs hls-8.8.3
stack ./install.hs data
```

The Haskell Language Server can also be built with `cabal v2-build` instead of `stack build`.
This has the advantage that you can decide how the GHC versions have been installed.
To see what GHC versions are available, the command `cabal-hls-install ghcs` can be used.
It will list all *supported* GHC versions that are on the path for build with their respective installation directory.
If you think, this list is incomplete, you can try to modify the PATH variable, such that the executables can be found.
Note, that the targets `hls` and `data` depend on the found GHC versions.

An example output is:

```bash
> ./cabal-hls-install ghcs
******************************************************************
Found the following GHC paths:
ghc-8.6.5: /opt/bin/ghc-8.6.5
ghc-8.8.3: /opt/bin/ghc-8.8.3

******************************************************************
```

If your desired ghc has been found, you use it to install haskell-language-server.

```bash
./cabal-hls-install hls-8.6.5
./cabal-hls-install data
```

## Project Configuration

**For a full explanation of possible configurations, refer to [hie-bios/README](https://github.com/mpickering/hie-bios/blob/master/README.md).**

haskell-language-server has some limited support via hie-bios to detect automatically
your project configuration and set up the environment for GHC.
The plan is to improve it to handle most use cases.

However, for now, the more reliable way is using a `hie.yaml` file in the root
of the workspace to **explicitly** describe how to setup the environment.
For that you need to know what *components* have your project and the path
associated with each one. So you will need some knowledge about
[stack](https://docs.haskellstack.org/en/stable/build_command/#components) or [cabal](https://cabal.readthedocs.io/en/latest/cabal-commands.html?#cabal-v2-build) components.

You also can use [this utility](https://github.com/Avi-D-coder/implicit-hie
) to generate automatically `hie.yaml` files for
the most common stack and cabal configurations

For example, to state that you want to use `stack` then the configuration file
would look like:

```yaml
cradle:
  stack:
    component: "haskell-language-server:lib"
```

If you use `cabal` then you probably need to specify which component you want
to use.

```yaml
cradle:
  cabal:
    component: "lib:haskell-language-server"
```

If you have a project with multiple components, you can use a cabal-multi
cradle:

```yaml
cradle:
  cabal:
    - path: "./test/functional/"
      component: "haskell-language-server:func-test"
    - path: "./test/utils/"
      component: "haskell-language-server:hls-test-utils"
    - path: "./exe/Main.hs"
      component: "haskell-language-server:exe:haskell-language-server"
    - path: "./exe/Wrapper.hs"
      component: "haskell-language-server:exe:haskell-language-server-wrapper"
    - path: "./src"
      component: "lib:haskell-language-server"
    - path: "./ghcide/src"
      component: "ghcide:lib:ghcide"
    - path: "./ghcide/exe"
      component: "ghcide:exe:ghcide"
```

Equivalently, you can use stack:

```yaml
cradle:
  stack:
    - path: "./test/functional/"
      component: "haskell-language-server:func-test"
    - path: "./exe/Main.hs"
      component: "haskell-language-server:exe:haskell-language-server"
    - path: "./exe/Wrapper.hs"
      component: "haskell-language-server:exe:haskell-language-server-wrapper"
    - path: "./src"
      component: "haskell-language-server:lib"
    - path: "./ghcide/src"
      component: "ghcide:lib:ghcide"
    - path: "./ghcide/exe"
      component: "ghcide:exe:ghcide"
```

Or you can explicitly state the program which should be used to collect
the options by supplying the path to the program. It is interpreted
relative to the current working directory if it is not an absolute path.

```yaml
cradle:
  bios:
    program: ".hie-bios"
```

The complete configuration is a subset of

```yaml
cradle:
  cabal:
    component: "optional component name"
  stack:
    component: "optional component name"
  bios:
    program: "program to run"
    dependency-program: "optional program to run"
  direct:
    arguments: ["list","of","ghc","arguments"]
  default:
  none:

dependencies:
  - someDep
```

## Contributing

### It's time to join the project!

:heart: Haskell tooling dream is near, we need your help! :heart:

- Join [our IRC channel](https://webchat.freenode.net/?channels=haskell-ide-engine) at `#haskell-ide-engine` on `freenode`.
- Fork this repo and hack as much as you can.
- Ask @alanz or @hvr to join the project.

### Hacking on haskell-language-server

Haskell-language-server can be used on its own project.  We have supplied
preset samples of `hie.yaml` files for stack and cabal, simply copy
the appropriate template to `hie.yaml` and it should work.

- `hie.yaml.cbl` for cabal
- `hie.yaml.stack` for stack

Two sample `hie.yaml` files are provided, `hie.yaml.stack` for stack
usage, `hie.yaml.cbl` for cabal. Simply copy the relevant one to be
`hie.yaml` and it should work.

The developers tend to hang out at [our IRC channel](https://webchat.freenode.net/?channels=haskell-ide-engine) at `#haskell-ide-engine` on `freenode`.
