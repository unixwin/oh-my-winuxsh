# Legacy Migration

This repository previously contained a `.winsh` script framework for an older
WinSH-era shell. That content does not match the current Winuxsh architecture.

## Preservation

Do not erase history. Preserve the old state with:

```sh
git tag legacy-pre-winuxsh-plugin-system
git branch legacy-pre-winuxsh-plugin-system
```

Then rebuild `main` as the official Winuxsh plugin bundle.

## What Changed

Old model:

- clone into `~/.oh-my-winuxsh`;
- source `winshrc`;
- source theme and plugin scripts;
- configure plugin arrays in rc.

New model:

- ship the bundle with Winuxsh releases;
- describe packs with `bundle.toml` and `packs/*/plugin.toml`;
- enable packs through `~/.winshrc.toml`;
- keep `~/.winshrc` for user shell code only;
- support bundle update and rollback through Winuxsh plugin commands.

## Zsh and Oh My Zsh

This repository is not an Oh My Zsh fork.

Winuxsh may provide a zsh migration command that detects familiar configuration
intent such as:

```zsh
plugins=(git zoxide)
```

The output should suggest Winuxsh plugins:

```text
oh-my-winuxsh/git
oh-my-winuxsh/zoxide
```

It should not call those zsh plugins, and it should not execute zsh plugin
source.
