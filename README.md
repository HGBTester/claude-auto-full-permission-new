# claude-auto (Full Permission)

A cross-platform wrapper for the Claude CLI that **natively bypasses all permission prompts**, including when running as **root**.

## Features

- **Full permission bypass** - Uses Claude Code's native `--dangerously-skip-permissions` mechanism
- **Root access support** - Sets `IS_SANDBOX=1` + `CLAUDE_CODE_BUBBLEWRAP=1` to unlock bypass even as root in containers
- **3-layer bypass** - Environment variables + CLI flag + settings.json for maximum reliability
- **Cross-platform** - Works on Linux, macOS, and Windows
- **Auto-finds claude** - Searches PATH and common installation locations
- **Manual override** - Disable bypass anytime with `-n` flag

## How It Works

claude-auto uses a **3-layer approach** to ensure permissions are always bypassed:

1. **Environment variables** - Sets `IS_SANDBOX=1` and `CLAUDE_CODE_BUBBLEWRAP=1` to unlock bypass for root users
2. **CLI flag** - Passes `--dangerously-skip-permissions` to claude
3. **Settings file** - Writes `~/.claude/settings.json` with `bypassPermissions` as default mode and all tools allowed

This means **no permission prompts** ever interrupt your workflow - not for file edits, bash commands, web fetches, or any other tool.

## Installation

### Linux / macOS

```bash
git clone https://github.com/HGBTester/claude-auto-full-permission-new.git
cd claude-auto-full-permission-new
chmod +x claude-auto
sudo ln -sf $(pwd)/claude-auto /usr/local/bin/claude-auto
```

### Windows

```powershell
git clone https://github.com/HGBTester/claude-auto-full-permission-new.git
cd claude-auto-full-permission-new

# Add to PATH or copy to a directory in your PATH
$env:PATH += ";$(pwd)"

# Or copy to a directory already in PATH
copy claude-auto C:\Users\YourName\bin\claude-auto.py
```

For colored output on Windows, install colorama:
```powershell
pip install colorama
```

## Usage

```bash
# Full interactive session - all permissions bypassed (default)
claude-auto

# Interactive session with initial prompt
claude-auto "fix the bug in main.py"

# One-shot prompt mode (non-interactive)
claude-auto -p "fix the bug in main.py"

# Disable bypass (normal claude behavior)
claude-auto --no-bypass
claude-auto -n

# Only write settings.json without launching claude
claude-auto --setup-only

# Skip settings.json (only use env vars + CLI flag)
claude-auto --no-settings

# Pass flags directly to claude
claude-auto -- --resume
claude-auto -- --help
```

## Options

| Flag | Description |
|------|-------------|
| `--no-bypass`, `-n` | Disable permission bypass (run claude normally) |
| `--setup-only` | Only write settings.json and environment, don't launch claude |
| `--no-settings` | Skip writing settings.json (only use env vars + CLI flag) |
| `--help`, `-h` | Show help message |

## Root / Container Support

When running as root (common in Docker containers and CI/CD):

- Claude Code normally **blocks** `--dangerously-skip-permissions` for root users
- claude-auto sets `IS_SANDBOX=1` to signal a sandboxed environment, unlocking the bypass
- `CLAUDE_CODE_BUBBLEWRAP=1` provides an additional sandbox signal
- This makes it work seamlessly in Docker, CI/CD pipelines, and automated environments

## Settings.json Configuration

claude-auto writes the following to `~/.claude/settings.json`:

```json
{
  "permissions": {
    "defaultMode": "bypassPermissions",
    "allow": [
      "Bash(*)", "Edit", "Write", "Read", "Glob", "Grep",
      "WebFetch(*)", "WebSearch(*)", "NotebookEdit", "Task(*)", "Skill(*)"
    ]
  },
  "skipDangerousModePermissionPrompt": true
}
```

## Claude Detection

claude-auto searches for the claude executable in:

**Linux/macOS:**
- PATH
- `/usr/local/bin/claude`
- `/usr/bin/claude`
- `~/.local/bin/claude`
- `~/.npm-global/bin/claude`
- `/opt/claude/claude`
- `~/.claude/claude`
- npm global bin directory

**Windows:**
- PATH
- `%LOCALAPPDATA%\Programs\claude\claude.exe`
- `%APPDATA%\npm\claude.cmd`
- `~\.claude\claude.exe`
- `C:\Program Files\claude\claude.exe`
- `C:\Program Files (x86)\claude\claude.exe`
- npm global bin directory

## Requirements

- Python 3.6+
- `claude` CLI installed
- (Windows optional) `colorama` for colored output

## License

MIT
