# Switching to Nix

I use Linux daily. The basic packages get installed via (in my case) `apt`: Gnome, Chromium, Bash, ripgrep, fdfind, git, ...

But many tools are often outdated in my Ubuntu LTS, so I use `nix profile` to install them in $HOME.

With `nix profile add nixpkgs#PACKAGE` I install binaries into `~/.nix-profile/bin/`.

This is great for the second layer. The three layers are:

1. OS packages (apt)
2. $HOME packages (nix profile)
3. Per directory (flake.nix and direnv)

## Layer $HOME

Common packages in layer $HOME: atuin, direnv, go_1_25, kubectl, nix-direnv, pnpm, starship, yq-go, meld

## Layer Directory (direnv)

I use [direnv](https://direnv.net/) and [nix-direnv](https://github.com/nix-community/nix-direnv).

Example .envrc file:

```sh
# shellcheck shell=bash
use flake
```

As soon as I enter the directory with `cd mydir`, the packages for this directory get updated (if needed). This just takes a second.

If you use vscode, I recommend [vscode Direnv Extension](https://marketplace.visualstudio.com/items?itemName=mkhl.direnv).
With this extensions, tasks of type shell automatically load your `.envrc` file (for example AI coding agents).

More about [How I use Direnv](https://github.com/guettli/How-I-use-direnv/)

Thinks I install via `flake.nix`:

- The programming languages: Go, Python, ...
- Tools for managing depenencies with each language: pip, nodePackages.pnpm, ...
- Shell helpers: ripgrep, shellcheck, ...
- go-task: [Taskfile](https://taskfile.dev/)

BTW, I use the latest stable: `nixpkgs.url = "github:NixOS/nixpkgs/nixos-YY.MM";`. Not `unstable`.

## Python/Go/JS Dependencies via Nix?

Currently, I do not use Nix to install the programming language-specific dependencies. I use the native format.

- Go: go.mod
- Python: pyproject.toml
- JS/TS: pnpm


Auto-updating these packages via `direnv/envrc` did not work well for me. Currently, I do that by hand. Please tell me, if you have a good solution for that!

## vscode

I avoid Snap Packages - I never understood why they exist. But I install `vscode` via Snap.

BTW, I install UI tools I launch via vscode-terminal via nix. Otherwise:

```
❯ meld 
/usr/bin/python3: symbol lookup error: /snap/core20/current/lib/x86_64-linux-gnu/libpthread.so.0: undefined symbol: __libc_pthread_init, version GLIBC_PRIVATE
```

## Taskfile

Nix could be used as an alternative to Makefiles. I tried that, but switched to [Taskfile](https://taskfile.dev/), because Taskfile is easier to
understand. At least for me.

```yaml
version: "3"

tasks:
  default:
    desc: Build everything (WASM, TypeScript, ...)
    deps: [all]

  _nix-check:
    internal: true
    run: once
    preconditions:
      - sh: test "${DIRENV_DIR#-}" = "{{.TASKFILE_DIR}}"
        msg: "Not in nix dev shell. Run via: ./run task"

  all:
    deps: [_nix-check]
    ...
```

The `./run` script:

```sh
#!/usr/bin/env bash
# Bash Strict Mode: https://github.com/guettli/bash-strict-mode
trap 'echo -e "\n🤷 🚨 🔥 Warning: A command has failed. Exiting the script. Line was ($0:$LINENO): $(sed -n "${LINENO}p" "$0" 2>/dev/null || true) 🔥 🚨 🤷 "; exit 3' ERR
set -Eeuo pipefail

# Show usage if no args or first arg starts with -
if [[ $# -eq 0 ]] || [[ "${1:-}" == -* ]]; then
    echo "Usage: run <command> [args...]" >&2
    echo "Example: run tsx scripts/update-ipa.ts phrases-de.yaml" >&2
    exit 1
fi

if [[ "${DIRENV_DIR#-}" != "$(cd "$(dirname "$0")" && pwd)" ]]; then
    # Execute the command with direnv environment loaded
    exec direnv exec . "$@"
fi

exec "$@"
```

This way I am sure that the nix env is active when running Taskfile.

## No NixOS

NixOS is a Linux operating system. I am not interested. Up to now I stick to Ubuntu LTS - the boring way.

## Misc

In January 2026, I switched from using [Homebrew on Linux](https://docs.brew.sh/Homebrew-on-Linux) to Nix, and it works well.

Currently, I do not use the Nix tool [Home Manager](https://nix-community.github.io/home-manager/), because it adds complexity.

## More

[Thomas WOL: Working out Loud](https://github.com/guettli/wol)
