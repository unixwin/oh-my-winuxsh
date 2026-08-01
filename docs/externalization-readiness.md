# Externalization Readiness
This document classifies the official bundle packs before moving any `builtin`
pack into a code-bearing runtime. It is not a schema change and does not decide
whether the future manifest should use a new runtime kind, `execution = "none"`,
`asset_only = true`, or another marker.
## Current Bundle Facts
- Static bundle assets already cover aliases, completion tables, prompt presets,
  keybinding metadata, and themes.
- WASM packs are command-only today. They can read args, cwd, and explicitly
  allowed env values, but they cannot act as completion, prompt, hook, or
  command-not-found or other providers yet.
- Process packs can wrap native commands, lifecycle hooks, and Winuxsh command-not-found provider binding, but they do not
  provide a structured shell-effect protocol for env/cwd/history changes.
- Shell-mutating packs must keep their native Winuxsh fallback until host
  effects are explicit, permissioned, and tested.
- Winuxsh plugin CLI review surfaces expose derived `execution_model`,
  `externalization_class`, and readiness profile values for these rows. They
  are not bundle manifest fields.
- `exports.providers` is a guarded provider marker. In current Winuxsh, process packs may use it for command-not-found with command:diagnose; it is
  not a new runtime kind and does not make command-style WASM act as a
  provider.
## Pack Matrix
| Pack | Current runtime | Classification | Target runtime / execution model | Missing host API or decision | Shell-mutating | Fallback needed |
| --- | --- | --- | --- | --- | --- | --- |
| `git` | `builtin` | Mixed declarative/native | Declarative alias/completion assets plus native prompt segment until prompt provider ABI | Prompt segment provider ABI | No | Yes |
| `docker` | `builtin` | Declarative asset | Asset-only/declarative; schema marker TBD | Asset-only schema decision | No | Minimal |
| `kubectl` | `builtin` | Declarative asset | Asset-only/declarative; schema marker TBD | Asset-only schema decision | No | Minimal |
| `npm` | `builtin` | Mixed declarative/native | Declarative assets plus native/dynamic completion until completion provider ABI | Completion/provider ABI | No | Yes |
| `zoxide` | `builtin` | Shell-effect candidate | Native builtin now; future effect runtime only after cwd effects are explicit | `shell:cwd:write`, lifecycle context, rollback behavior | Yes | Yes |
| `direnv` | `builtin` | Shell-effect candidate | Native builtin now; future effect runtime only after env writes are structured | `env:write`, lifecycle context, rollback behavior | Yes | Yes |
| `dotenv` | `builtin` | Shell-effect candidate | Native builtin now; future effect runtime only after fs/env effects are structured | Scoped file read, `env:write`, lifecycle context | Yes | Yes |
| `fzf` | `builtin` | External-tool adapter plus shell effect | Native builtin now; future process/effect only after interactive policy | Interactive process policy, `shell:cwd:write` | Yes | Yes |
| `command-not-found` | `builtin` | Pure provider candidate | Winuxsh process provider binding exists; official bundle can stay builtin until migration; future WASM provider must use separate ABI | Bundle migration decision plus future WASM provider entrypoint | No | Yes |
| `last-working-dir` | `builtin` | Shell-effect candidate | Native builtin now; future effect runtime only after cache/cwd protocol | Scoped cache read/write, startup/chpwd effect protocol | Yes | Yes |
| `thefuck` | `builtin` | External-tool adapter plus shell effect | Native/process adapter plus future effect protocol; hold now | `history:read`, suggested command review/execute protocol | Yes | Yes |
| `keybindings` | `builtin` | Declarative asset | Asset-only/declarative; schema marker TBD | Asset-only schema decision; reedline actions stay native | No | Minimal |
| `prompts` | `builtin` | Mixed declarative/native | Declarative prompt presets plus native segments until prompt provider ABI | Prompt segment provider ABI | No | Yes |
| `themes` | `builtin` | Declarative asset | Asset-only/declarative; schema marker TBD | Asset-only schema decision; renderer stays native | No | Minimal |
| `process-echo` | `process` | Fixture | Process command fixture | None for fixture scope | No | No |
| `process-hook` | `process` | Fixture | Process hook fixture; not a generic effect runtime | Structured hook effects before normal use | No in fixture | No |
| `wasm-hello` | `wasm` | Fixture | WASM command fixture; provider/effect ABI is separate | Provider/effect ABI is separate future work | No | No |
## Next Bundle Work
1. Preserve existing `kind` values until Winuxsh chooses an asset-only schema.
2. Treat `themes`, `keybindings`, static aliases, static completions, and prompt
   presets as declarative assets in docs and reviews even while manifests still
   say `kind = "builtin"`.
3. Use [command-not-found](command-not-found-provider-abi.md) to design the
   first provider migration only after Winuxsh process binding proves deterministic input/output,
   permission review, timeout behavior, and compiled fallback rules.
4. Do not migrate `zoxide`, `direnv`, `dotenv`, `fzf`, `thefuck`, or
   `last-working-dir` until shell effects are part of the host contract.
