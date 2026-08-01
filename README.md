# Oh My Winuxsh

Official bundled plugin distribution for Winuxsh.

This repository is not an Oh My Zsh fork and does not provide zsh plugin
compatibility. It contains first-party Winuxsh plugin bundle metadata that the
Winuxsh plugin system can ship, inspect, update, and roll back.

## Status

This branch rebuilds the repository for the current Winuxsh architecture:

- Winuxsh owns the shell frontend, config, plugin registry, and permission model.
- `oh-my-winuxsh` provides official plugin manifests and bundled assets.
- First-party shell plugins can ship `.winux` startup code through
  `kind = "source"`, close to the Oh My Zsh model but gated by manifests and
  permission review.
- Future sandboxed third-party providers can use the same manifest model, with
  WASM/WASI as the preferred runtime once the host API is stable.

## Bundle Model

The bundle is expected to ship with Winuxsh releases, similar to how winuxcmd is
bundled as the default coreutils layer. Users should have a local baseline even
without network access, then update the bundle independently:

```sh
winuxsh plugin update oh-my-winuxsh --from dist\oh-my-winuxsh-1.0.0.zip --checksum-file dist\oh-my-winuxsh-1.0.0.zip.sha256
winuxsh plugin rollback oh-my-winuxsh
```

Plugin state belongs in `~/.winshrc.toml`:

```toml
[plugins]
enabled = true
bundles = ["oh-my-winuxsh"]
load = ["git", "prompts", "keybindings"]

[plugins.git]
enabled = true
permissions = ["shell:source", "cwd:read", "process:run:git"]
```

User shell code belongs in `~/.winshrc`:

```sh
alias ll='ls -la'
export EDITOR=vim
```

## Repository Layout

```text
oh-my-winuxsh/
  bundle.toml
  index.toml
  packs/
    git/plugin.toml
    git/init.winux
    docker/plugin.toml
    docker/init.winux
    kubectl/plugin.toml
    kubectl/init.winux
    npm/plugin.toml
    npm/init.winux
    zoxide/plugin.toml
    direnv/plugin.toml
    dotenv/plugin.toml
    fzf/plugin.toml
    command-not-found/plugin.toml
    last-working-dir/plugin.toml
    thefuck/plugin.toml
    process-echo/plugin.toml
    process-hook/plugin.toml
    wasm-hello/plugin.toml
    keybindings/plugin.toml
    prompts/plugin.toml
  aliases/
    git.toml
    docker.toml
    kubectl.toml
    npm.toml
    README.md
  completions/
    git.toml
    docker.toml
    kubectl.toml
    npm.toml
    README.md
  prompts/
    segments.toml
    README.md
  keybindings/
    common.toml
    emacs.toml
    vi.toml
    README.md
  themes/
    cyberpunk.toml
    forest.toml
    minimal.toml
    ocean.toml
    README.md
  wasm/
    wasm-hello.wasm
    wasm-hello.wat
  docs/
    authoring.md
    design.md
    migration.md
    roadmap.md
  templates/
    builtin/plugin.toml
    source/plugin.toml
    source/init.winux
    process/plugin.toml
    wasm/plugin.toml
```

## First-Party Packs

| Pack | Purpose | Default |
| --- | --- | --- |
| `git` | Git `.winux` helpers, aliases, completions, prompt segment | On |
| `docker` | Docker `.winux` helpers, aliases, completion metadata | Off |
| `kubectl` | Kubernetes `.winux` helpers, aliases, completion metadata | Off |
| `npm` | npm `.winux` helpers, aliases, runtime completion shape | Off |
| `zoxide` | Native `z` command shim and directory tracking | Off |
| `direnv` | Explicit lifecycle hook adapter for `direnv export bash` | Off |
| `dotenv` | Safe `.env` parser for current project directory | Off |
| `fzf` | Directory selector command shims | Off |
| `command-not-found` | Interactive missing-command hints | Off |
| `last-working-dir` | Last working directory cache and restore command | Off |
| `thefuck` | Correction shim for the previous interactive command | Off |
| `process-echo` | Process plugin host contract fixture | Off |
| `process-hook` | Process plugin lifecycle hook contract fixture | Off |
| `wasm-hello` | WASM host API contract fixture | Off |
| `keybindings` | Winuxsh keybinding presets, not ZLE support | On |
| `prompts` | Prompt presets and segment defaults | On |
| `themes` | Official prompt and Git status color themes | On |

## Authoring

Public pack authoring starts in [docs/authoring.md](docs/authoring.md). Use the
copyable templates under `templates/` for source, builtin, process, and WASM
manifests, then run:

```sh
python tools/validate_bundle.py
python tools/package_bundle.py --check
winuxsh plugin review <pack>
winuxsh plugin search devtools
winuxsh plugin themes
winuxsh plugin install git
winuxsh plugin doctor
```

The authoring contract is manifest-first: TOML declares runtime kind,
permissions, exports, and required binaries before Winuxsh runs plugin code.
Source plugins put shell code in bundle-local `.winux` files.

## Legacy

The old repository content was a `.winsh` script framework from a previous
WinSH era. It is preserved through the `legacy-pre-winuxsh-plugin-system`
branch/tag; the active branch describes the official Winuxsh plugin bundle.

See [docs/migration.md](docs/migration.md).

## Roadmap

The synchronized implementation roadmap lives in [docs/roadmap.md](docs/roadmap.md).
Its phase numbers are aligned with Winuxsh's `DOCS/plugin-system-roadmap.md`.
Release compatibility is tracked in [docs/compatibility.md](docs/compatibility.md),
and bundle changes are tracked in [CHANGELOG.md](CHANGELOG.md).

## Validation And Packaging

```sh
python tools/validate_bundle.py
python tools/package_bundle.py --check
# Windows launcher:
py tools\validate_bundle.py
py tools\package_bundle.py --check
```

The validator checks release documents, package index drift, release checksum policy, bundle API and minimum Winuxsh metadata,
bundle inventory drift, manifest required fields, allowed runtime
kinds/categories, exported asset directory presence, parseable alias packs,
parseable completion definitions, prompt preset segment references, and
declarative keybinding metadata and theme TOML assets for exported packs, plus the Phase 9 authoring
guide and templates. Source manifests must declare `shell:source` and a
bundle-local `.winux` entry. Process manifests must be explicit opt-in and
declare protocol, command, timeout, permissions, and required binaries. The package
script builds `dist/oh-my-winuxsh-{version}.zip` plus a `.sha256` checksum when
run without `--check`.
WASM manifests must also declare a bundle-local `.wasm` module path and SHA-256;
the validator checks that the artifact exists, matches the digest, and has a
valid WASM binary header. The current Winuxsh host can execute explicit command
fixtures that export `winuxsh_plugin_main() -> i32`, may write stdout/stderr,
may read simple command arguments, read cwd when `cwd:read` is declared, and
read explicitly permitted env values through `env:read:<NAME>` using the Phase
14-17 `winuxsh:plugin/host` imports; broader WASI and
shell-mutating host APIs remain future work.

Local release smoke test:

```sh
py tools\package_bundle.py
winuxsh plugin update oh-my-winuxsh --from dist\oh-my-winuxsh-1.0.0.zip --checksum-file dist\oh-my-winuxsh-1.0.0.zip.sha256
winuxsh plugin bundle status
winuxsh plugin search workflow
winuxsh plugin doctor
winuxsh plugin review wasm-hello
winuxsh plugin search workflow
```

## License

MIT unless the Unixwin project chooses a different repository license before the
first bundle release.
