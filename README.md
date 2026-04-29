# Oh My Winuxsh!

A framework for managing your [WinSH](https://github.com/caomengxuan666/winuxsh) shell configuration. Inspired by [Oh My Zsh](https://ohmyz.sh/).

## Features

- **5 Beautiful Themes**: Default, Cyberpunk, Ocean, Minimal, Forest
- **Git Plugin**: 50+ git aliases and enhanced prompt info
- **Completion Plugin**: Extended tab completions
- **Core Library**: Color definitions, Unicode symbols, helper functions
- **Easy Configuration**: Simple `.winshrc` integration

## Installation

### Option 1: Git Clone
```bash
git clone https://github.com/caomengxuan666/oh-my-winuxsh.git $HOME/.oh-my-winuxsh
echo 'source $HOME/.oh-my-winuxsh/winshrc' >> $HOME/.winshrc
```

### Option 2: Manual Download
Download the latest release and extract to `$HOME/.oh-my-winuxsh`.

## Themes

| Theme | Preview | Description |
|-------|---------|-------------|
| `default` | `➜ ~` | Clean and professional with git status |
| `cyberpunk` | `⚡ ▶ ~` | Neon-style with cyberpunk aesthetics |
| `ocean` | `❯ 🌊 ~` | Calming blue and green ocean vibes |
| `minimal` | `λ ~` | Minimal with just the essentials |
| `forest` | `↳ 🌲 ~` | Nature-inspired forest colors |

To change theme, set `WINUXSH_THEME` in your `.winshrc`:
```bash
WINUXSH_THEME="cyberpunk"
```

## Plugins

### Git Plugin
Provides 50+ git aliases and enhanced prompt integration.
- `gst` for `git status`, `gco` for `git checkout`, etc.
- Shows branch, ahead/behind, staged/unstaged in prompt

### Completion Plugin
Enhances tab completion with additional contexts.
- Git commands completion
- SSH hostname completion
- Docker, Cargo, NPM command completions

## Configuration

Place configuration in `$HOME/.winshrc`:
```bash
# Set theme
WINUXSH_THEME="ocean"

# Set plugins
plugins=(
    git
    completion
)
```

## Directory Structure
```
~/.oh-my-winuxsh/
├── winshrc              # Main entry point
├── lib/
│   └── core.winsh       # Core utilities
├── themes/
│   ├── default.theme.winsh
│   ├── cyberpunk.theme.winsh
│   ├── ocean.theme.winsh
│   ├── minimal.theme.winsh
│   └── forest.theme.winsh
├── plugins/
│   ├── git.plugin.winsh
│   └── completion.plugin.winsh
└── README.md
```

## License

MIT
