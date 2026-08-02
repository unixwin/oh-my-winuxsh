# Framework Smoke Notes

Status: living runtime QA notes for the directory-first framework.

## Current Smoke Command

Run with the selected installed Winuxsh binary:

```sh
winuxsh tools/smoke_framework.winux C:/path/to/oh-my-winuxsh
```

The smoke currently verifies:

- default framework load: `prompt-core git`;
- prompt API functions: `winuxsh_prompt_render_left` and
  `winuxsh_prompt_git_info`;
- all official theme plugins: `minimal`, `classic`, `pure`, `compact`,
  `cyberpunk`, `forest`, and `ocean`;
- Git prompt snapshot contract: `prompt-core` sets `WINUXSH_PROMPT_GIT`, and
  `winuxsh_prompt_git_info` reads that snapshot;
- devtool plugin aliases/functions from `git`, `docker`, `kubectl`, and `npm`;
- lifecycle hook execution through the `dotenv` plugin.

Core-side regression tests currently cover:

- `plugins/<name>/plugin.toml` inventory loading from an Oh My Winuxsh bundle;
- directory plugins replacing same-named legacy packs;
- `bridge` plugin inventory display for Reedline/provider-owned surfaces;
- source plugins receiving `WINUXSH` and `WINUXSH_PLUGIN_BUNDLE_DIR`;
- host prompt sync consuming `prompt-core`/theme plugin templates by default
  while explicit TOML prompt configuration remains authoritative;
- host prompt sync consuming `WINUXSH_PROMPT_GIT` snapshots so `{git}` renders
  from prompt-core state instead of recalculating Git during draw;
- `[plugins.git].enabled = false` removing Git aliases and completion
  definitions;
- external bundles not borrowing official compiled aliases when their own
  assets are missing.

## Runtime Findings

- Prompt rendering must be lazy. The prompt renderer should only evaluate
  segments present in the active template. Eagerly evaluating every registered
  segment caused unused `{time}` to spawn `date` and leak visible output on the
  installed Winuxsh build.
- Git prompt helpers should avoid fragile redirection inside command
  substitution on the installed Winuxsh build. The current helper captures Git
  branch/status without putting `2>/dev/null` inside the captured branch
  command.
- Smoke scripts should prefer stable native paths such as `C:/tmp` for temp
  files. The installed build exposes a `/c/Users/.../Temp` value for `TMPDIR`
  that can fail during redirected writes.

## Next Smoke Gates

- Expand daemon-backed prompt snapshot smoke coverage for dirty/chpwd/git-command
  invalidation.
- Add richer prompt API renderer tests once host prompt rendering can call
  plugin segment/provider functions directly instead of only consuming exported
  template variables.
- Add bridge-plugin tests when keybinding and command-not-found host APIs can
  consume directory plugin identity directly.
