# Retired: Manifest-First Oh My Winuxsh Design

Status: retired. This document captures the old manifest-first bundle design for
historical context only. The active design is `docs/design.md`.

The reason this design is retired is product-level, not test-level: it made the
official bundle behave more like a host-owned asset registry than an Oh My-style
plugin framework. Tests can pass under that model while the user and authoring
experience still feels wrong.

Retired conclusions:

- A pack should not need to be implemented as host `builtin` behavior to feel
  first class.
- Themes are plugins, not a special shell-core asset category.
- Official distribution can bundle plugins, but the shell should not own plugin
  behavior simply because a plugin ships by default.
- The plugin framework should load and compose plugin directories, while Winuxsh
  core should expose shell primitives and safe host APIs.

# Previous Manifest-First Design

`oh-my-winuxsh` is the official bundled plugin distribution for Winuxsh.
The execution sequence lives in [roadmap.md](roadmap.md).

It follows the normal shell-plugin model where trusted plugin code can be
sourced into the current interactive session, while keeping manifests,
permissions, bundle updates, and rollback in Winuxsh-owned metadata.

This is intentionally closer to Oh My Zsh and bash plugin collections than the
earlier TOML-only plan. Static data still lives in TOML assets, but code-bearing
first-party shell plugins now use bundle-local `.winux` files declared through
`kind = "source"`.

## Bundle Layout

- `bundle.toml` declares bundle identity, update source, pack inventory, and
  asset directory names.
- `packs/<name>/plugin.toml` declares each first-party pack's runtime kind,
  permissions, exports, defaults, and required binaries.
- `packs/<name>/init.winux` contains sourced Winuxsh shell code for
  `kind = "source"` packs.
- `aliases/`, `completions/`, `prompts/`, and `keybindings/` are asset
  directories. `aliases/*.toml` owns first-party alias tables,
  `completions/*.toml` owns native completion definitions,
  `prompts/segments.toml` owns prompt segment mappings and preset layouts, and
  `keybindings/*.toml` owns declarative metadata for native editor actions.
- Winuxsh keeps compiled fallbacks so bundled releases work offline even before
  independent bundle updates exist.
- `docs/authoring.md` and `templates/` define the public authoring surface for
  source, builtin, process, and WASM pack manifests.

## Why TOML First

TOML is the declaration format, not a replacement for bash syntax. It answers
questions that should be auditable before code runs:

- what pack is being enabled;
- which bundle shipped it;
- what runtime kind it uses;
- which permissions it asks for;
- which aliases, completions, prompt presets, keybindings, themes, commands, or
  hooks it exports;
- which external binaries or artifacts are required.

Packs that only need static data can be pure TOML. Packs that need shell
behavior should usually start as `source` packs. Process and WASM remain
available for external-tool adapters and sandboxed providers.

Use [Externalization Readiness](externalization-readiness.md) before changing
pack schemas. The bundle should classify asset-only packs, mixed native packs,
provider candidates, process adapters, and shell-effect candidates before
deciding whether Winuxsh needs a new runtime kind or a lighter execution marker.

## Boundaries

- Winuxsh core owns shell execution, config loading, permissions, plugin
  registry, and update/rollback behavior.
- oh-my-winuxsh owns first-party pack manifests and static assets.
- Prompt rendering and editor behavior remain native Winuxsh/Reedline code;
  bundle assets only select safe presets and metadata.
- rubash owns parser, executor, builtins, redirects, functions, pipelines, and
  shell semantics.
- reedline owns interactive editing behavior.

## Control Plane

Plugin control belongs in `~/.winshrc.toml` because it is deterministic and
machine-editable:

```toml
[plugins]
enabled = true
bundles = ["oh-my-winuxsh"]
load = ["git", "prompts", "keybindings"]
```

The shell rc file remains free-form user code. It can use plugin-provided
commands after they are enabled, but it should not be the source of truth for
plugin permissions or bundle versions.

`~/.winshrc` is still the right place for a user's own trusted shell
customizations:

```sh
alias ll='ls -la'
export EDITOR=vim

hello() {
  echo "hello from winuxsh"
}
```

WASM does not replace this rc file. The rc file is the user's personal freedom
surface. WASM and process plugins are for distributable plugin code that should
be reviewed, permissioned, versioned, sandboxed where possible, and reversible
through the bundle update model.

## Runtime Kinds

- `source`: bundle-local `.winux` scripts sourced into the current interactive
  shell session during startup and declared `precmd`, `preexec`, or `chpwd`
  lifecycle hooks.
- `builtin`: first-party Rust implementations inside Winuxsh, mainly fallback
  and native adapters.
- `wasm`: future third-party plugins through WASM/WASI. WASM packs declare a
  protocol, module path, SHA-256, WIT world, timeout, and memory cap in
  `[wasm]`. Current command modules run in the Winuxsh wasmi host when enabled,
  export `winuxsh_plugin_main() -> i32`, may write stdout/stderr, may read
  simple command arguments, may read cwd when `cwd:read` is declared, and may
  read explicitly allowed env values when `env:read:<NAME>` is declared through
  the Phase 14-17 `winuxsh:plugin/host` imports; completions, prompt segments,
  WASI, and shell-mutating WASM APIs remain later host surfaces.
- `process`: compatibility and debugging adapters for existing tools. Process
  packs declare a protocol, command, arguments, timeout, and permissions in
  `[process]`; Winuxsh validates that contract before accepting the bundle and
  executes enabled command or lifecycle hook exports with deterministic IO.

The runtime kind changes how a plugin executes, not the manifest or user-facing
plugin identity.

The current `builtin` packs are a transition layer. They let Winuxsh put
existing first-party behavior behind the same registry, permission review, and
bundle defaults before every host API needed by external plugins exists. They
should not imply that future plugins are limited to aliases or static data.

## Builtin Externalization Direction

New code-bearing packs should prefer `wasm` or `process` once the required host
API exists. Existing builtin packs can move out gradually, but not all builtin
behavior should move.

Good early candidates for WASM are mostly pure providers:

- command-not-found suggestions;
- prompt segment calculators;
- completion providers;
- command suggestion or formatting helpers.

Good source candidates are trusted shell helpers that must mutate the current
session:

- `zoxide`;
- `direnv`;
- `dotenv`;
- `fzf`-style selectors;
- `last-working-dir`;
- `thefuck`.

Good process candidates are adapters around existing executables that do not
need current-shell env/cwd mutation.

Packs that mutate shell state outside trusted source still need more host API
before they can safely become WASM/provider plugins:

- structured env and cwd effects;
- scoped file effects;
- startup/precmd/preexec/chpwd context;
- deterministic failure and rollback behavior.

Keep core shell machinery inside Winuxsh:

- rubash parsing and execution;
- reedline editor primitives;
- Windows path, cwd, env, and process synchronization;
- core prompt rendering;
- native file/process helpers that define the shell contract.

Do not externalize code just to make the bundle look more like a traditional
script plugin repository. Externalize when it improves distribution,
auditability, or third-party extensibility.

## Authoring Surface

Pack authors should start from `docs/authoring.md` and the templates under
`templates/`. The validator treats those files as part of the release surface,
so CI fails if the public authoring entry point disappears or templates stop
parsing as TOML.

Host-side auditing stays in Winuxsh:

```sh
winuxsh plugin doctor [--json]
winuxsh plugin review <pack> [--json]
```

## Update Model

Winuxsh releases should include a baseline copy of this bundle. Later, users can
update the bundle independently:

```sh
winuxsh plugin update oh-my-winuxsh --from dist\oh-my-winuxsh-1.0.1.zip --checksum-file dist\oh-my-winuxsh-1.0.1.zip.sha256
winuxsh plugin rollback oh-my-winuxsh
```

Install state should be versioned and reversible:

```text
%LOCALAPPDATA%/Winuxsh/bundles/oh-my-winuxsh/<version>/
%LOCALAPPDATA%/Winuxsh/bundles/oh-my-winuxsh/current
~/.winuxsh/plugin-lock.toml
```

The lock file should include bundle version, checksum, source, active path, and
previous path for rollback.

## Zsh Migration

Zsh may be an onboarding source, but it is not plugin identity. A migration
command may read `.zshrc` and suggest `oh-my-winuxsh/git` or
`oh-my-winuxsh/zoxide`. It must not claim that Winuxsh supports zsh plugins or
ZLE.
