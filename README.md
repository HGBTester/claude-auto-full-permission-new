# claude-auto (Full Permission)

A cross-platform wrapper for **Claude Code**, **Gemini CLI**, and **Codex CLI** that **natively bypasses all permission prompts**, including when running as **root**.

## Features

- **Multi-model support** — Claude Code, Gemini CLI, and Codex CLI from a single launcher
- **Full permission bypass** — Each model's native bypass mechanism, applied automatically
- **Interactive model menu** — Pick a model, run a session, pick another — no restart needed
- **Session capture** — Captures terminal output via PTY for session logging and context handoff
- **Cross-model context handoff** — Switch models mid-workflow and carry the conversation context
- **Session logging** — Saves cleaned session transcripts to `~/.claude-auto/logs/`
- **Root access support** — Sets `IS_SANDBOX=1` + `CLAUDE_CODE_BUBBLEWRAP=1` for Claude as root
- **Cross-platform** — Works on Linux, macOS, and Windows (capture features are Unix-only)
- **Auto-detects CLIs** — Searches PATH and common installation locations
- **Backward compatible** — Single-model installs behave exactly like before

## Bypass Mechanisms

| Model | Bypass Flag | Env Vars | Config File |
|-------|-------------|----------|-------------|
| Claude | `--dangerously-skip-permissions` | `IS_SANDBOX=1`, `CLAUDE_CODE_BUBBLEWRAP=1` | `~/.claude/settings.json` |
| Gemini | `--yolo` | None | None |
| Codex | `--dangerously-bypass-approvals-and-sandbox` | None | None |

## Installation

### Prerequisites

Install one or more AI coding CLIs:

```bash
npm install -g @anthropic-ai/claude-code   # Claude Code
npm install -g @google/gemini-cli           # Gemini CLI
npm install -g @openai/codex                # Codex CLI
```

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
# Interactive model picker (when multiple CLIs installed)
claude-auto

# Launch a specific model directly
claude-auto --model claude        # or -M claude
claude-auto -M gemini
claude-auto -M codex

# Show which models are installed
claude-auto --list-models         # or -L

# Exit after one session (don't loop back to menu)
claude-auto --no-loop

# Disable bypass (run model CLI normally)
claude-auto --no-bypass           # or -n
claude-auto -n -M gemini          # Gemini without --yolo

# Only write settings/environment, don't launch
claude-auto --setup-only

# Skip writing settings.json (only env vars + CLI flag)
claude-auto --no-settings

# Disable PTY capture (plain subprocess, no context handoff)
claude-auto --no-capture

# Disable context handoff between sessions (still saves logs)
claude-auto --no-context

# Custom session log directory
claude-auto --log-dir /tmp/my-logs

# Pass flags through to the underlying CLI
claude-auto -M claude -- --resume
claude-auto -M gemini -- --help
```

## Options

| Flag | Description |
|------|-------------|
| `--model`, `-M` (claude/gemini/codex) | Select model directly — skip menu, no loop |
| `--list-models`, `-L` | Show installed models and exit |
| `--no-loop` | Exit after session ends (don't show menu again) |
| `--no-bypass`, `-n` | Disable permission bypass (run CLI normally) |
| `--setup-only` | Only write settings/environment, don't launch |
| `--no-settings` | Skip writing settings.json (only use env vars + CLI flag) |
| `--no-capture` | Disable PTY capture (plain subprocess, no context handoff) |
| `--no-context` | Disable context handoff between sessions (still saves logs) |
| `--log-dir DIR` | Custom directory for session logs (default: `~/.claude-auto/logs`) |
| `--help`, `-h` | Show help message |

All unknown flags are passed through to the underlying CLI via `parse_known_args()`.

**Note:** `-M` (uppercase) is used instead of `-m` to avoid conflicts with model-specific `-m` flags that pass through to the child CLI.

## How It Works

### Session Loop with Context Handoff

When multiple models are installed:

```
Start → Show menu → Pick model → Run session (PTY capture) → Session ends
  → Post-session menu → Pick next model → "Carry context? [Y/n]"
  → Inject transcript as initial prompt → Run next session → ...
```

When only one model is installed (backward compatible):

```
Start → Auto-detect model → Run session → Exit
```

### Context Handoff

When you exit a session and pick another model, claude-auto offers to carry the conversation context:

1. **PTY Capture** — Terminal output is captured via `pty.fork()` while still displaying normally
2. **ANSI Stripping** — Raw terminal bytes are cleaned of escape sequences and control characters
3. **Context Accumulation** — Each session's transcript is appended with session markers
4. **Handoff Injection** — The accumulated transcript is wrapped in a handoff prompt template and passed to the next CLI as a positional argument (Claude, Codex) or `-i` flag (Gemini)
5. **Truncation** — Context is tail-truncated to 50,000 characters, breaking at session boundaries when possible

Session transcripts are also saved to `~/.claude-auto/logs/` as `YYYYMMDD_HHMMSS_sNN_model.log`.

### Windows Graceful Degradation

PTY capture requires Unix APIs (`pty`, `termios`, `tty`). On Windows, sessions run via plain `subprocess.run()` — identical to previous behavior, with no capture or context handoff.

### Claude Bypass (3-layer approach)

1. **Environment variables** — Sets `IS_SANDBOX=1` and `CLAUDE_CODE_BUBBLEWRAP=1` to unlock bypass for root users
2. **CLI flag** — Passes `--dangerously-skip-permissions` to claude
3. **Settings file** — Writes `~/.claude/settings.json` with `bypassPermissions` as default mode and all tools allowed

### Gemini Bypass

Passes `--yolo` flag which disables all confirmation prompts.

### Codex Bypass

Passes `--dangerously-bypass-approvals-and-sandbox` flag which disables approval prompts and sandboxing.

## Root / Container Support

When running as root (common in Docker containers and CI/CD):

- Claude Code normally **blocks** `--dangerously-skip-permissions` for root users
- claude-auto sets `IS_SANDBOX=1` to signal a sandboxed environment, unlocking the bypass
- `CLAUDE_CODE_BUBBLEWRAP=1` provides an additional sandbox signal
- Gemini and Codex bypass flags work without special root handling

## Settings.json Configuration (Claude only)

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

## Requirements

- Python 3.6+
- At least one supported CLI installed (`claude`, `gemini`, or `codex`)
- (Windows optional) `colorama` for colored output

## License

MIT
