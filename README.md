# claude-auto

A wrapper for the Claude CLI that automatically accepts "Do you want to proceed?" prompts.

## Why?

When using `claude` CLI, it frequently asks for confirmation with "Do you want to proceed?" prompts. This wrapper automatically presses Enter to accept, so you don't have to babysit long-running sessions.

## Installation

```bash
# Clone the repo
git clone https://github.com/HGBTester/claude-auto.git
cd claude-auto

# Make executable and add to PATH
chmod +x claude-auto
sudo ln -sf $(pwd)/claude-auto /usr/local/bin/claude-auto
```

Or manually copy `claude-auto` to any directory in your PATH.

## Usage

```bash
# Auto-accept ON (default) - presses Enter automatically when prompted
claude-auto "your prompt here"

# Auto-accept OFF (manual mode)
claude-auto --no-auto "your prompt"
claude-auto -n "your prompt"

# Pass flags to claude itself
claude-auto -- --dangerously-skip-permissions "your prompt"
```

## How It Works

1. Wraps the `claude` command in a pseudo-terminal
2. Monitors output for "Do you want to proceed?" prompts
3. When detected & auto-accept is ON → automatically presses Enter
4. Your manual input still works - you can type anytime

## Options

| Flag | Description |
|------|-------------|
| `--no-auto`, `-n` | Disable auto-accept (manual mode) |
| `--help`, `-h` | Show help message |

## Requirements

- Python 3.6+
- `claude` CLI installed and in PATH

## License

MIT
