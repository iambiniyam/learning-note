# Creating and Running Bash Scripts

[Previous: 01 - Introduction](./01-introduction.md) • [Next: 03 - Variables and Data Types](./03-variables-and-data-types.md)

## Overview (beginner friendly)

A Bash script is just a text file with commands you’d normally type in the terminal, saved so you can run them again and again. Think of it like a recipe: each line is a step the shell follows.

Why this matters when you’re new:

- You type less and do more by automating steps.
- You avoid mistakes by repeating the same reliable sequence.
- You can share your steps with teammates.

> Windows tip: To follow along on Windows, use WSL (Windows Subsystem for Linux) or Git Bash. Save files with Unix line endings (LF) to avoid “bad interpreter” errors.

## What is a Bash Script?

A bash script is a file containing a series of commands that are executed sequentially by the bash shell. Scripts allow you to automate tasks instead of typing commands manually.

## Creating Your First Script (step by step)

We’ll create a tiny script that prints some info, make it executable, and run it.

### Step 1: Create a File

```bash
touch myscript.sh
```

### Step 2: Add Shebang

The “shebang” (the very first line starting with `#!`) tells the system which interpreter should run this file.

```bash
#!/bin/bash
```

Always make this the first line of your Bash scripts.

**Why shebang matters:**

```bash
#!/bin/bash          # Use bash
#!/usr/bin/env bash  # More portable - finds bash in PATH
#!/bin/sh            # Use sh (older, POSIX-compliant)
#!/usr/bin/python3   # For Python scripts
```

### Step 3: Add Commands (your script’s body)

```bash
#!/bin/bash
# My first bash script

echo "Hello, World!"
echo "Today is: $(date)"
echo "Current user: $USER"
```

### Step 4: Make It Executable (give it run permission)

```bash
chmod +x myscript.sh
```

**Permissions explained:**

- `chmod +x`: Add execute permission
- `chmod 755`: rwxr-xr-x (owner: read/write/execute, others: read/execute)
- `chmod 700`: rwx------ (only owner can read/write/execute)

### Step 5: Run the Script

```bash
# Method 1: Direct execution (requires execute permission)
./myscript.sh

# Method 2: Explicit bash interpreter
bash myscript.sh

# Method 3: Source the script (runs in current shell)
source myscript.sh
# or
. myscript.sh
```

## Difference: ./script vs bash script vs source script (easy comparison)

| Method             | Description                               | Use Case                      |
| ------------------ | ----------------------------------------- | ----------------------------- |
| `./script.sh`      | Runs in new subshell, needs +x permission | Standard script execution     |
| `bash script.sh`   | Runs in new subshell, no +x needed        | Testing scripts               |
| `source script.sh` | Runs in current shell                     | Setting environment variables |

**Example (why `source` is different):**

```bash
#!/bin/bash
# test.sh
export MY_VAR="Hello"

# If you run: ./test.sh
# MY_VAR won't be available after script ends

# If you run: source test.sh
# MY_VAR will be available in your current shell
```

## Script Structure Best Practices (clean and safe)

Keep scripts easy to read and safe by using a few conventions and options.

- Turn on “strict mode” so errors don’t go unnoticed:

```bash
set -euo pipefail
```

- When you need to see what’s running, toggle debug output:

```bash
set -x   # Turn on debug (prints commands)
# ... run some lines you want to inspect ...
set +x   # Turn off debug
```

## Set Options Explained (with short examples)

### `set -e` (Exit on Error)

```bash
#!/bin/bash
set -e

echo "Step 1"
false           # This will fail
echo "Step 2"   # This won't execute
```

### `set -u` (Exit on Undefined Variable)

```bash
#!/bin/bash
set -u

echo $UNDEFINED_VAR  # Script will exit with error
```

### `set -o pipefail` (Pipe Failure)

```bash
#!/bin/bash
set -o pipefail

# Without pipefail: only checks last command (grep)
# With pipefail: checks all commands in pipe
cat nonexistent.txt | grep "pattern"  # Will fail
```

### `set -x` (Debug Mode)

```bash
#!/bin/bash
set -x

echo "This will show the command before executing"
# Output: + echo 'This will show the command before executing'
```

### Combined: Strict Mode

```bash
#!/bin/bash
set -euo pipefail
# Or: set -Eeuo pipefail (also enables ERR trap)
```

## Comments in Bash (explain your intent)

```bash
#!/bin/bash

# This is a single-line comment

: '
This is a
multi-line comment
Everything here is ignored
'

echo "Hello"  # Inline comment
```

## Script Arguments (inputs to your script)

```bash
#!/bin/bash

echo "Script name: $0"
echo "First argument: $1"
echo "Second argument: $2"
echo "All arguments: $@"
echo "Number of arguments: $#"
echo "Exit status of last command: $?"
echo "Process ID: $$"
```

Try it:

```bash
./script.sh arg1 arg2 arg3
```

## Exit Codes (how scripts say success/failure)

```bash
#!/bin/bash

# Successful execution
exit 0

# Error occurred
exit 1

# Custom error codes
exit 2  # Misuse of command
exit 126  # Command cannot execute
exit 127  # Command not found
exit 130  # Script terminated by Ctrl+C
```

Checking exit codes:

```bash
./myscript.sh
echo $?  # Shows exit code of last command
```

## Script Template (easy to adapt)

```bash
#!/bin/bash

set -euo pipefail

# Script metadata
readonly SCRIPT_NAME="$(basename "$0")"
readonly SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
readonly VERSION="1.0.0"

# Colors for output
readonly RED='\033[0;31m'
readonly GREEN='\033[0;32m'
readonly YELLOW='\033[1;33m'
readonly NC='\033[0m' # No Color

# Logging functions
log_info() {
    echo -e "${GREEN}[INFO]${NC} $*"
}

log_error() {
    echo -e "${RED}[ERROR]${NC} $*" >&2
}

log_warn() {
    echo -e "${YELLOW}[WARN]${NC} $*"
}

# Help function
show_help() {
    cat << EOF
Usage: $SCRIPT_NAME [OPTIONS]

Description:
    Brief description of what the script does

Options:
    -h, --help      Show this help message
    -v, --version   Show version information

Examples:
    $SCRIPT_NAME
    $SCRIPT_NAME --help

EOF
}

# Cleanup function (runs on exit)
cleanup() {
    log_info "Cleaning up..."
    # Add cleanup tasks here
}

# Set trap for cleanup
trap cleanup EXIT

# Main function
main() {
    # Parse arguments
    while [[ $# -gt 0 ]]; do
        case $1 in
            -h|--help)
                show_help
                exit 0
                ;;
            -v|--version)
                echo "$SCRIPT_NAME version $VERSION"
                exit 0
                ;;
            *)
                log_error "Unknown option: $1"
                show_help
                exit 1
                ;;
        esac
        shift
    done

    # Main logic here
    log_info "Script started"

    # Your code here

    log_info "Script completed successfully"
}

# Run main function with all arguments
main "$@"
```

## Sourcing Other Scripts (reuse helpers)

```bash
#!/bin/bash

# Source external configuration
source /path/to/config.sh

# Or with error handling
if [[ -f "/path/to/config.sh" ]]; then
    source "/path/to/config.sh"
else
    echo "Config file not found!"
    exit 1
fi
```

## Running Scripts from Anywhere (PATH 101)

### Option 1: Add to PATH

```bash
# Add directory to PATH in ~/.bashrc
export PATH="$PATH:$HOME/scripts"

# Then run from anywhere
myscript.sh
```

### Option 2: Create symlink in /usr/local/bin

```bash
sudo ln -s /full/path/to/myscript.sh /usr/local/bin/myscript
```

### Option 3: Install in standard location

```bash
sudo cp myscript.sh /usr/local/bin/myscript
sudo chmod +x /usr/local/bin/myscript
```

## Common Mistakes to Avoid (newbie pitfalls)

1. **Forgetting shebang**: Script might use wrong interpreter
2. **Not making executable**: `chmod +x` required for `./script`
3. **Wrong path separator**: Use `/` not `\` in scripts
4. **Spaces in filenames**: Use quotes: `"$filename"`
5. **Not checking errors**: Always use `set -e` or check `$?`
6. **Windows line endings (CRLF)**: Save files with Unix line endings (LF). In VS Code, set "End of Line" to LF. CRLF can cause `bad interpreter` errors.
7. **Wrong shebang path**: On many systems Bash is at `/bin/bash`; on others use `#!/usr/bin/env bash`.
8. **Using `source` when you meant to run**: `source` affects your current shell; prefer `./script.sh` for isolation.

## Practice Exercises

Create a script that:

1. Prints a welcome message
2. Shows current date and time
3. Displays current directory
4. Lists files in current directory
5. Takes your name as argument and greets you

**Solution:**

```bash
#!/bin/bash

set -euo pipefail

echo "=== Welcome to DevOps World ==="
echo "Current date and time: $(date)"
echo "Current directory: $(pwd)"
echo -e "\nFiles in this directory:"
ls -lh
echo -e "\nHello, ${1:-Guest}!"
```

---

## Exercises

1. Add a second argument for a file path; print whether it exists and its size.
2. Add a `-v/--verbose` flag that prints extra logs when present.
3. Add a `--dry-run` flag that prints what would be done without changing anything.

---

## Cross‑links

- [01 - Introduction](./01-introduction.md)
- [03 - Variables and Data Types](./03-variables-and-data-types.md)

## References

- GNU Bash Reference Manual: <https://www.gnu.org/software/bash/manual/bash.html>
