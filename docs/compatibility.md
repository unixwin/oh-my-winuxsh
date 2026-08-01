# Compatibility

This document defines the release compatibility contract for the official
oh-my-winuxsh bundle.

## Current Release

- Bundle name: `oh-my-winuxsh`
- Bundle version: `1.0.0`
- Release channel: `stable`
- Minimum host: `min_winuxsh = "0.8.3"`
- Bundle API: `winuxsh:plugin-bundle@0.1.0`
- Pack manifest API: `winuxsh:plugin@0.1.0`
- Source plugin script suffix: `.winux`
- Process plugin protocol: `winuxsh:process-plugin@0.1.0`
- WASM plugin protocol: `winuxsh:wasm-plugin@0.1.0`

Winuxsh refuses bundle updates whose `min_winuxsh` is newer than the running
host. Keep this field at the oldest host version that can validate and run the
bundle safely.

## Runtime Surface

- `source` packs load bundle-local `.winux` scripts into the current
  interactive Winuxsh session during `startup`, `precmd`, `preexec`, and
  `chpwd` lifecycle hooks. They are active for the interactive REPL and `-C`;
  ordinary `-c`, script-file, and stdin execution stay clean by default.
- `builtin` packs load first-party aliases, completions, prompts, keybindings,
  themes, and built-in Winuxsh fallback/native behavior.
- `process` packs are explicit opt-in adapters for native commands. They must
  declare `required_binaries`, `process:run:<command>` permission, protocol,
  command, arguments, and timeout.
- `wasm` packs are explicit opt-in command modules. Current modules export
  `winuxsh_plugin_main() -> i32`; Phase 14-17 add constrained
  `winuxsh:plugin/host.stdout_write`, `stderr_write`, and `arg_*` imports
  for deterministic stdout/stderr bytes, simple command arguments, and
  permission-gated cwd/env reads through exported module memory.
- `exports.providers` is an optional compatibility marker, not a runtime. The
  current bundle may mark `command-not-found` as the first provider ABI
  candidate, and process packs may use the implemented command-not-found
  provider binding when `command:diagnose` is declared. WASM packs cannot
  export providers until Winuxsh defines a separate WASM provider contract.
- Shell-mutating WASM host APIs, arbitrary zsh source, WASI, unrestricted files,
  unrestricted processes, ZLE widgets, DLL plugins, and unbounded native process bridges
  are outside the current compatibility contract.

## Semver Policy

- Patch releases fix manifests, static assets, checksums, docs, or validation
  rules without requiring new permissions or a newer Winuxsh host.
- Minor releases may add packs, add static assets, or expand permissions when
  the permission review surface makes the change explicit.
- Major releases are reserved for breaking manifest/API changes, removed packs,
  incompatible asset layout changes, or a new minimum host that older supported
  Winuxsh releases cannot install.

## Release Artifacts

Every published release must include:

- `oh-my-winuxsh-{version}.zip`
- `oh-my-winuxsh-{version}.zip.sha256`
- `index.toml` with `checksum_required = true` and `signature = "unsupported"`
- `CHANGELOG.md`
- `docs/compatibility.md`

The zip must be produced by `tools/package_bundle.py` so file ordering,
timestamps, and permissions stay deterministic. Winuxsh host update policy
validates `index.toml` against the bundle and pack manifests before install;
checksum-required zip updates must pass SHA-256 verification before the active
bundle lock is switched.
