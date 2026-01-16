# claude-auto

A cross-platform wrapper for the Claude CLI that automatically accepts prompts with the most aggressive option.

## Features

- **Cross-platform**: Works on Linux, macOS, and Windows
- **Auto-finds claude**: Searches PATH and common installation locations
- **Aggressive auto-accept**: Prefers "Yes, and don't ask again" over simple "Yes"
- **Manual override**: Disable auto-accept anytime with `-n` flag

## How It Works

When claude asks "Do you want to proceed?":

```
Do you want to proceed?
❯ 1. Yes
  2. Yes, and don't ask again for [command] in [directory]
  3. No
```

**claude-auto** automatically selects **Option 2** (don't ask again) when available, or **Option 1** (Yes) otherwise.

## Installation

### Linux / macOS

```bash
git clone https://github.com/HGBTester/claude-auto.git
cd claude-auto
chmod +x claude-auto
sudo ln -sf $(pwd)/claude-auto /usr/local/bin/claude-auto
```

### Windows

```powershell
git clone https://github.com/HGBTester/claude-auto.git
cd claude-auto

# Add to PATH or copy to a directory in your PATH
# Option 1: Add current directory to PATH
$env:PATH += ";$(pwd)"

# Option 2: Copy to a directory already in PATH
copy claude-auto C:\Users\YourName\bin\claude-auto.py
```

For colored output on Windows, install colorama:
```powershell
pip install colorama
```

## Usage

```bash
# Auto-accept ON (default) - aggressive mode
claude-auto "your prompt here"

# Auto-accept OFF (manual mode)
claude-auto --no-auto "your prompt"
claude-auto -n "your prompt"

# Pass flags to claude itself
claude-auto -- --help
claude-auto -- --dangerously-skip-permissions "your prompt"
```

## Options

| Flag | Description |
|------|-------------|
| `--no-auto`, `-n` | Disable auto-accept (manual mode) |
| `--help`, `-h` | Show help message |

## Claude Detection

claude-auto searches for the claude executable in:

**Linux/macOS:**
- PATH
- `/usr/local/bin/claude`
- `/usr/bin/claude`
- `~/.local/bin/claude`
- `~/.npm-global/bin/claude`
- npm global bin directory

**Windows:**
- PATH
- `%LOCALAPPDATA%\Programs\claude\claude.exe`
- `%APPDATA%\npm\claude.cmd`
- `C:\Program Files\claude\claude.exe`
- npm global bin directory

## Requirements

- Python 3.6+
- `claude` CLI installed
- (Windows optional) `colorama` for colored output

## License

MIT
