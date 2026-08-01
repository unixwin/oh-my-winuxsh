# Externalization Readiness
This document classifies official bundle packs before moving host-owned
behavior out of `builtin`. It records when a pack belongs in `source`,
`process`, `wasm`, or a future asset-only marker.
## Current Bundle Facts
- Static bundle assets already cover aliases, completion tables, prompt presets,
  keybinding metadata, and themes.
- Source packs are reviewed bundle-local `.winux` startup scripts. They are the
  Oh My-style path for first-party shell helpers that intentionally define
  aliases, functions, exports, or startup glue in the current interactive
  shell.
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
| `git` | `source` | Source helper plus mixed declarative/native | `.winux` helpers plus alias/completion assets; native prompt segment until prompt provider ABI | Prompt segment provider ABI | Yes, by trusted startup source | Yes |
| `docker` | `source` | Source helper plus declarative asset | `.winux` helpers plus alias/completion assets | None for current first-party helper scope | Yes, by trusted startup source | Minimal |
| `kubectl` | `source` | Source helper plus declarative asset | `.winux` helpers plus alias/completion assets | None for current first-party helper scope | Yes, by trusted startup source | Minimal |
| `npm` | `source` | Source helper plus mixed declarative/native | `.winux` helpers plus assets; native/dynamic completion until completion provider ABI | Completion/provider ABI | Yes, by trusted startup source | Yes |
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
1. Use `kind = "source"` for reviewed first-party shell helpers whose value is
   current-session shell code.
2. Preserve `builtin` for host-owned native behavior and fallback paths until
   Winuxsh exposes a better runtime/API.
3. Treat `themes`, `keybindings`, static aliases, static completions, and prompt
   presets as declarative assets in docs and reviews even while manifests still
   say `kind = "builtin"`.
4. Use [command-not-found](command-not-found-provider-abi.md) to design the
   first provider migration only after Winuxsh process binding proves deterministic input/output,
   permission review, timeout behavior, and compiled fallback rules.
5. Do not migrate `zoxide`, `direnv`, `dotenv`, `fzf`, `thefuck`, or
   `last-working-dir` until shell effects are part of the host contract.
