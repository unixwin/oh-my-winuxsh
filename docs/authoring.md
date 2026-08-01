# Plugin Authoring

This guide is the public authoring contract for the current Winuxsh-native
plugin model. It is intentionally manifest-first: plugin trust, permissions,
runtime kind, and exported surfaces are declared in TOML before Winuxsh runs
anything.

## Current Scope

- `oh-my-winuxsh` is the official first-party bundle and reference layout. It
  is not the whole future plugin universe; third-party registries should use
  the same manifest/index contract and be validated by Winuxsh host policy.
- The current local update command accepts only the `oh-my-winuxsh` bundle name.
- `source` packs are the Oh My-style shell plugin path. They ship bundle-local
  `.winux` code that Winuxsh sources into the current interactive session after
  plugin review and enablement.
- `builtin` packs are first-party fallback/native adapters because the Rust
  implementation lives in Winuxsh core.
- `process` packs are explicit opt-in adapters for existing native tools.
- `wasm` packs are the preferred third-party direction. Current command modules
  export `winuxsh_plugin_main() -> i32` and may use the Phase 14-17
  `winuxsh:plugin/host` stdout/stderr, args, cwd, and env read imports.
- Shell-mutating WASM APIs, arbitrary zsh source, ZLE widgets, and DLL/FFI
  plugin ABIs are outside the current public contract.

## Authoring Loop

1. Copy one of the templates under `templates/`.
2. Add the pack name to `bundle.toml` under `[packs].available`.
3. Put the manifest at `packs/<name>/plugin.toml`.
4. Add any exported assets under `aliases/`, `completions/`, `prompts/`,
   `keybindings/`, `themes/`, or `wasm/`; source packs also add
   `packs/<name>/init.winux`.
5. Run the CI-safe checks:

```sh
python tools/validate_bundle.py
python tools/package_bundle.py --check
```

6. Smoke the host-side review surface:

```sh
winuxsh plugin update oh-my-winuxsh --from dist\oh-my-winuxsh-1.0.1.zip --checksum-file dist\oh-my-winuxsh-1.0.1.zip.sha256
winuxsh plugin doctor
winuxsh plugin search <pack>
winuxsh plugin review <pack>
winuxsh plugin install <pack>
```

Use `--json` on `doctor`, `list`, `search`, `info`, `review`, and `themes` when
wiring CI or release automation. Discovery, diagnostic, review, and theme JSON include
`trust_source`, so automation can distinguish the official bundled registry
surface, local override bundles, and future external bundle sources without
executing plugin code.
External bundle sources are review-only in the current host; managed
`plugin install` support waits for the third-party registry trust policy.

## Bundle Metadata

`bundle.toml` must declare `api = "winuxsh:plugin-bundle@0.1.0"` and
`min_winuxsh` as a semantic version. Winuxsh refuses bundle updates whose
minimum host version is newer than the running shell.

`index.toml` is the registry-facing release index. It must match `bundle.toml`
and every pack manifest, set `release.checksum_required = true`, use
`release.checksum_algorithm = "sha256"`, and set `release.signature =
"unsupported"` until signing support exists. Zip updates must be installed
with `--checksum` or `--checksum-file`; the host rejects checksum-required zip
updates that skip verification.

## Manifest Schema

Every `packs/<name>/plugin.toml` must include:

```toml
name = "example"
bundle = "oh-my-winuxsh"
version = "1.0.1"
kind = "source" # asset/static data may still use builtin fallback; runtime kinds include source | builtin | process | wasm
api = "winuxsh:plugin@0.1.0"
category = "workflow" # devtools | environment | workflow | hints | ux
summary = "Short user-facing summary."
default = false
permissions = ["shell:source"]
required_binaries = []

[exports]
aliases = false
completions = []
prompt_segments = []
hooks = []
commands = []
keybindings = []
themes = []
providers = []
```

Field rules:

- `name` must match the directory under `packs/` and the entry in
  `bundle.toml`.
- `bundle` must match `bundle.toml` while this repository is the active
  reference bundle.
- `api` must be `winuxsh:plugin@0.1.0` until the host bumps the contract.
- `default = true` is only allowed for first-party packs that are safe without
  surprising startup side effects.
- `permissions` must exactly describe pack behavior.
- `required_binaries` lists native commands the user must have in `PATH`.
- `exports` declares the surfaces Winuxsh may load. Declaring an export also
  requires the matching asset or runtime contract.
- `exports.providers` is optional and currently limited to
  `command-not-found`. Builtin packs may use it as a readiness marker, and
  process packs may use the implemented Winuxsh command-not-found provider
  binding when `command:diagnose` is declared. WASM packs must not export
  providers until a separate WASM provider ABI exists.

## Runtime Contracts

### Source

Use `kind = "source"` when a pack should behave like a traditional shell
plugin. Winuxsh sources the declared `.winux` file into the current interactive
session during REPL startup and declared lifecycle hooks. Ordinary `-c`,
script-file, and stdin execution stay clean unless a future explicit opt-in is
added; the `-C` one-shot REPL path loads the same interactive startup surface
when the host supports it.

```toml
[source]
entry = "packs/example/init.winux"
```

Rules:

- `permissions` must include `shell:source`.
- `source.entry` must be a bundle-local relative path ending in `.winux`.
- `exports.hooks` may contain `startup`, `precmd`, `preexec`, and `chpwd`.
- Source plugins may define aliases, functions, exports, cwd-changing helpers,
  and shell lifecycle glue, but plugin enablement and permissions remain in
  `~/.winshrc.toml`.
- User `~/.winshrc` runs after official source plugins, so user shell code can
  override plugin defaults.

### Builtin

Use `kind = "builtin"` only when Winuxsh core already owns the runtime behavior.
The bundle may own static data such as aliases, completions, prompt presets, or
keybinding metadata, but it does not ship Rust code.

### Process

Process packs must be explicit opt-in:

```toml
[process]
protocol = "winuxsh:process-plugin@0.1.0"
command = "example-tool"
args = ["--json"]
timeout_millis = 1000
```

Rules:

- `default` must be `false`.
- `permissions` must include `process:run:<command>`.
- `required_binaries` must include the same command.
- At least one command or hook must be exported.
- Timeout must be between `1` and `30000` milliseconds.
- Process plugins cannot mutate shell parser or executor internals.

### WASM

WASM packs must also be explicit opt-in:

```toml
[wasm]
protocol = "winuxsh:wasm-plugin@0.1.0"
module = "wasm/example.wasm"
sha256 = "<64 lowercase hex chars>"
wit_world = "winuxsh:plugin/example"
timeout_millis = 1000
max_memory_pages = 16
```

Rules:

- `default` must be `false`.
- `required_binaries` must be empty.
- `permissions` must not include `process:run:*`.
- The module path must stay bundle-local and end in `.wasm`.
- The SHA-256 must match the checked-in artifact.
- Command modules must export `winuxsh_plugin_main() -> i32`.
- Current host imports are limited to:
  - `winuxsh:plugin/host.stdout_write(ptr: i32, len: i32) -> i32`;
  - `winuxsh:plugin/host.stderr_write(ptr: i32, len: i32) -> i32`;
  - `winuxsh:plugin/host.arg_count() -> i32`;
  - `winuxsh:plugin/host.arg_len(index: i32) -> i32`;
  - `winuxsh:plugin/host.arg_read(index: i32, ptr: i32) -> i32`;
  - `winuxsh:plugin/host.cwd_len() -> i32`;
  - `winuxsh:plugin/host.cwd_read(ptr: i32) -> i32`;
  - `winuxsh:plugin/host.env_len(name_ptr: i32, name_len: i32) -> i32`;
  - `winuxsh:plugin/host.env_read(name_ptr: i32, name_len: i32, value_ptr: i32) -> i32`.
- Host imports read from or write to exported module memory and return `-1` for
  missing memory, invalid indexes, invalid pointers, out-of-bounds memory access,
  values over the host cap, cwd reads without `cwd:read`, or env reads without a
  matching `env:read:<NAME>` permission.
- Lifecycle hooks, completions, prompt segments, keybindings, WASI, files,
  processes, env mutation, and shell mutation are not part of the current WASM
  public contract.

## Permission Tokens

| Token | Risk | Meaning |
| --- | --- | --- |
| `cwd:read` | low | Reads the current working directory or path context. |
| `env:read:<NAME>` | low | Reads one explicitly named environment variable. |
| `shell:source` | high | Sources bundle-owned `.winux` code into the current interactive session and lifecycle hooks. |
| `process:run:<cmd>` | high | Runs a native command such as `git` or `zoxide`. |
| `shell:cwd:write` | medium | Requests a native cwd change through a host-owned shim. |
| `env:write` | high | Requests environment variable changes. |
| `fs:read:<path>` | medium | Reads files below a declared path. |
| `fs:write:<path>` | high | Writes files below a declared path. |
| `command:diagnose` | low | Reads command metadata for diagnostics. |

Unknown tokens are treated as manual-review permissions by Winuxsh.

## Asset Rules

- `exports.aliases = true` requires `aliases/<pack>.toml` with a non-empty
  `[aliases]` table.
- `exports.completions = ["tool"]` requires `completions/tool.toml` whose
  `command` field is `tool`.
- `exports.prompt_segments` requires matching segment definitions in
  `prompts/segments.toml`.
- `exports.keybindings` requires matching files in `keybindings/<name>.toml`.
- `exports.themes = ["ocean"]` requires `themes/ocean.toml` using the Winuxsh
  theme style schema.
- `exports.providers = ["command-not-found"]` is a guarded marker for the first
  provider ABI candidate and requires `command:diagnose`.
- WASM modules live under `wasm/` and must match their manifest checksum.

## Release Checklist

- `python tools/validate_bundle.py`
- `python tools/package_bundle.py --check`
- `winuxsh plugin doctor --json`
- `winuxsh plugin review <pack> --json`
- Changelog entry for any permission, default, runtime, or exported asset
  change.
- Compatibility note when a pack requires a newer Winuxsh host API.
