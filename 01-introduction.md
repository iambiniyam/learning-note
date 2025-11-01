# Bash Scripting - Introduction

[Next: 02 - Creating and Running Scripts](./02-creating-and-running-scripts.md)

## What is Bash?

**Bash** (Bourne Again Shell) is a Unix shell and command language written as a free software replacement for the Bourne shell. It's the default shell on most Linux distributions and macOS.

## Why Learn Bash?

- **Automation**: Automate repetitive tasks
- **DevOps Essential**: Critical for CI/CD pipelines, deployment scripts
- **System Administration**: Manage servers, users, processes
- **Productivity**: Chain commands together efficiently
- **Universal**: Available on almost every Unix/Linux system

## Shell vs Terminal vs Console

- **Shell**: Command-line interpreter (bash, zsh, sh)
- **Terminal**: Application that runs a shell (Windows Terminal, iTerm2)
- **Console**: Physical device (keyboard + screen)

## Checking Your Bash Version

```bash
bash --version
```

Current stable version (2025): Bash 5.2+

> Windows tip: If you’re on Windows, the easiest way to use Bash is via WSL (Windows Subsystem for Linux) or Git Bash. For WSL, install Ubuntu from the Microsoft Store and open “Ubuntu” to get a Bash shell. Prefer LF line endings when saving scripts.

## Your First Bash Command

```bash
echo "Hello, DevOps World!"
```

## Common Bash Locations

- Linux/Mac: `/bin/bash` or `/usr/bin/bash`
- Check with: `which bash`

## Interactive vs Non-Interactive Shell

- **Interactive**: You type commands directly (terminal session)
- **Non-Interactive**: Scripts running without user input

## Login vs Non-Login Shell

- **Login Shell**: Initial shell on system login
  - Reads: `/etc/profile`, `~/.bash_profile`, `~/.bash_login`, `~/.profile`
- **Non-Login Shell**: New terminal window
  - Reads: `~/.bashrc`

## Key Concepts for DevOps

1. **Idempotency**: Scripts should produce same result when run multiple times
2. **Error Handling**: Always check for failures
3. **Logging**: Track what your scripts do
4. **Security**: Never hardcode credentials
5. **Portability**: Write scripts that work across environments

## Basic Shell Navigation

```bash
pwd                    # Print working directory
ls                     # List files
ls -la                 # List all files with details
cd /path/to/directory  # Change directory
cd ~                   # Go to home directory
cd -                   # Go to previous directory
cd ..                  # Go up one directory
```

## Getting Help

```bash
man command            # Manual pages
command --help         # Help option
info command           # Info documentation
type command           # Show command type
which command          # Show command location
whereis command        # Locate binary, source, manual
```

## Shell Configuration Files

```bash
~/.bashrc              # User-specific bash configuration
~/.bash_profile        # User login configuration
~/.bash_logout         # Executed when logout
/etc/bash.bashrc       # System-wide bash configuration
/etc/profile           # System-wide login configuration
```

## Quick Tips

- Use **Tab** for auto-completion
- Use **Ctrl + C** to cancel current command
- Use **Ctrl + D** to exit shell
- Use **Ctrl + L** to clear screen (or `clear` command)
- Use **↑/↓** arrows for command history
- Use **Ctrl + R** for reverse search in history

## Next Steps

Once you're comfortable with basic navigation and commands, move on to:

1. Creating and running bash scripts
2. Variables and data types
3. Control structures
4. Functions and more

---

**Remember**: Practice is key! Try each command as you learn it.

## Exercises

1. Open a terminal and run each command in “Basic Shell Navigation.” Note what each does.
2. Run `type`, `which`, and `whereis` on the commands `bash`, `ls`, and `echo`. Compare the outputs.
3. Check your Bash version and locate where Bash is installed on your system.

## Cross‑links

- [02 - Creating and Running Scripts](./02-creating-and-running-scripts.md)

## References

- GNU Bash Reference Manual: <https://www.gnu.org/software/bash/manual/bash.html>
