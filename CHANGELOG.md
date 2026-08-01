# Changelog

All notable changes to the official oh-my-winuxsh bundle are recorded here.
Release entries describe bundle artifacts, host compatibility, permission
changes, and user-visible asset changes.

## 1.0.0 - 2026-07-31

### Added

- Established the repository as the official Winuxsh plugin bundle rather than
  a legacy `.winsh` script framework.
- Added bundle metadata with `api = "winuxsh:plugin-bundle@0.1.0"` and
  `min_winuxsh = "0.8.3"`.
- Added first-party source packs for Git, Docker, kubectl, npm, zoxide,
  direnv, dotenv, fzf, last-working-dir, and thefuck with bundle-local `.winux`
  startup and lifecycle hook code.
- Added first-party builtin/native packs for command-not-found, prompts, and
  keybindings where host-owned provider/editor/prompt behavior still applies.
- Added explicit opt-in process fixture packs and a no-import WASM command
  fixture for host-side runtime validation.
- Added bundle-owned aliases, completions, prompt presets, keybinding metadata,
  official theme assets, authoring templates, and public authoring documentation.
- Added deterministic release packaging for `oh-my-winuxsh-1.0.0.zip` and the
  matching `.sha256` checksum file.

### Compatibility

- Requires Winuxsh 0.8.3 or newer for local bundle install, rollback, doctor,
  review, process plugin, and Phase 8 WASM command execution support.
- Uses pack manifest API `winuxsh:plugin@0.1.0`, process plugin protocol
  `winuxsh:process-plugin@0.1.0`, and WASM plugin protocol
  `winuxsh:wasm-plugin@0.1.0`.
- Keeps source, process, and WASM execution visible through manifest
  permissions because they can execute outside pure static asset loading.

### Release Checklist

- Attach `oh-my-winuxsh-1.0.0.zip` and
  `oh-my-winuxsh-1.0.0.zip.sha256` to the GitHub release.
- Include this changelog entry and `docs/compatibility.md` in the release notes.
- Run `python tools/validate_bundle.py` and
  `python tools/package_bundle.py --check` before publishing.
