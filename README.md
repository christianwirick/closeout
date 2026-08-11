# closeout

The off switch for your workday.

One command gracefully closes the apps on your Mac.

## Features

- Detects normal foreground applications in the current macOS user session.
- Protects Terminal, Passwords, the coordinating terminal app, and Finder by default.
- Uses PID and process start time together to guard against PID reuse.
- Sends all polite quit requests through one batched AppleScript process.
- Escalates only captured PIDs: quit, `SIGTERM`, then `SIGKILL`.
- Uses an in-place, color-aware interface on a TTY and stable output when piped.
- Switches to a compact high-volume mode above 10 applications.
- Runs follow-up sweeps after large sessions to catch applications opened or relaunched during closeout.
- Includes a non-destructive save mode that never sends signals.
- Has no third-party dependencies and supports the Bash version bundled with macOS.

## Install

```bash
git clone https://github.com/christianwirick/closeout.git
cd closeout

mkdir -p "$HOME/.local/bin"
install -m 755 closeout "$HOME/.local/bin/closeout"
ln -sf closeout "$HOME/.local/bin/eod"
```

Make sure `~/.local/bin` is on your `PATH`:

```bash
export PATH="$HOME/.local/bin:$PATH"
```

Add that line to `~/.bashrc` or `~/.zshrc` if it is not already configured.

## Usage

```text
Usage: closeout [OPTIONS]

Gracefully close foreground macOS applications, escalating only when needed.

Options:
  -f, --finder       Include Finder. macOS may relaunch it automatically.
      --save [SEC]   Wait without sending signals (default: 60 seconds).
  -h, --help         Show this help text.
```

Both command names support every option:

```bash
closeout
eod
eod -f
eod --save 90
eod -f --save 90
```

## Escalation policy

| Eligible applications | Clean-quit window | `SIGTERM` window |
|---:|---:|---:|
| 1–10 | 5 seconds | 2 seconds |
| 11–50 | 3 seconds | 1 second |
| 51+ | 2 seconds | 1 second |

After the `SIGTERM` window, matching survivors receive `SIGKILL`. Identity is checked again before every signal.

`--save` is different: it waits for save dialogs and exits with status `1` if applications remain at timeout. It never sends `SIGTERM` or `SIGKILL`.

## Output

Interactive runs keep one line per application and update it in place:

```text
› last call: 3 apps

  ✓ Notes
  ~ Messages                 term · 5s
  ◆ Preview                  kill · 7s

done · 1 quit · 1 term · 1 kill · 7s
```

Above 10 applications, closeout switches to a compact view and keeps only a small set of escalation receipts. Interactive large runs also receive one of eight deserved observations about their application count.

When output is redirected or piped, wording and fields are deterministic:

```text
closing 3 apps
quit    Notes
term    Messages
kill    Preview
done quit=1 term=1 kill=1 gone=0 elapsed=7s
```

Set `NO_COLOR=1` to disable color while retaining the interactive layout.

## Requirements

- macOS
- Bash 3.2 or newer
- Standard macOS tools: `osascript`, `ps`, `kill`, `date`, and `sleep`

## Important behavior

Applications may show save dialogs after receiving the clean quit request. Normal mode escalates after its grace window, so use `--save` when preserving unsaved work matters more than completing the closeout.

Finder is excluded unless `-f` or `--finder` is supplied. macOS may relaunch Finder automatically.
