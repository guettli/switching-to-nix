# Switching to Nix

I use Linux daily. The basic packages get installed via (in my case) `apt`: Gnome, Chromium, Bash, ripgrep, fdfind, git, ...

But many tools are often outdated in my Ubuntu LTS, so I use `nix profile` to install them in $HOME.

With `nix profile add nixpkgs#PACKAGE` I install binaries into `~/.nix-profile/bin/`.

This is great for the second layer. The three layers are:

1. OS packages (apt)
2. $HOME packages (nix profile)
3. Per directory (flake.nix and direnv)

## Layer $HOME

Common packages in layer $HOME: atuin, direnv, go_1_23, kubectl, nix-direnv, pnpm, starship, yq-go

## Layer Directory

I use [direnv](https://direnv.net/) and [nix-direnv](https://github.com/nix-community/nix-direnv).

Example .envrc file:

```sh
# shellcheck shell=bash
use flake
```

As soon as I enter the directory with `cd mydir`, the packages for this directory get updated (if needed). This just takes a second.

## Python/Go/JS Dependencies via Nix?

Currently, I do not use Nix to install the programming language-specific dependencies. I use the native format.

- Go: go.mod
- Python: pyproject.toml
- JS/TS: pnpm

Auto-updating these packages via `direnv/envrc` did not work well for me. Currently, I do that by hand. Please tell me, if you have a good solution for that!

## Misc

In January 2026, I switched from using [Homebrew on Linux](https://docs.brew.sh/Homebrew-on-Linux) to Nix, and it works well.

Currently, I do not use the Nix tool [Home Manager](https://nix-community.github.io/home-manager/), because it adds complexity.
