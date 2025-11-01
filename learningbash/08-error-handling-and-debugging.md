# Error Handling and Debugging in Bash

[Previous: 07 - Functions](./07-functions.md) • [Next: 09 - Text Processing](./09-text-processing.md)

Proper error handling and debugging techniques are essential for creating robust and maintainable bash scripts.

## Exit Codes

Every command in bash returns an exit code (0-255). By convention, 0 means success, and non-zero indicates an error.

### Checking Exit Codes

```bash
#!/bin/bash

# Check the last command's exit code
ls /nonexistent
echo "Exit code: $?"  # Will print non-zero (typically 2)

ls /tmp
echo "Exit code: $?"  # Will print 0 (success)
```

### Setting Exit Codes

```bash
#!/bin/bash

# Exit with specific code
if [ ! -f "important.txt" ]
then
    echo "Error: File not found" >&2
    exit 1  # Exit with error code 1
fi

# Success
exit 0
```

### Common Exit Codes

```bash
# Standard exit codes
# 0   - Success
# 1   - General errors
# 2   - Misuse of shell command
# 126 - Command cannot execute
# 127 - Command not found
# 128 - Invalid exit argument
# 130 - Script terminated by Ctrl+C
# 255 - Exit status out of range
```

## Error Handling Techniques

### Set Options for Error Handling

```bash
#!/bin/bash

# Exit on error
set -e  # Exit immediately if a command exits with non-zero status

# Exit on undefined variable
set -u  # Treat unset variables as an error

# Pipe failure detection
set -o pipefail  # Return value of pipeline is status of last command to exit with non-zero

# Combine all options (recommended)
set -euo pipefail

# Alternative: Use at script start
#!/bin/bash -eu
```

### Testing Set Options

```bash
#!/bin/bash
set -e

echo "This will run"
ls /nonexistent  # Script exits here
echo "This will NOT run"
```

### Temporarily Disable Exit on Error

```bash
#!/bin/bash
set -e

# This command might fail, but we want to continue
set +e
ls /nonexistent 2>/dev/null
status=$?
set -e

if [ $status -ne 0 ]
then
    echo "Directory doesn't exist, creating it..."
fi
```

### Using || and && for Error Handling

```bash
#!/bin/bash

# Run command OR handle error
command || echo "Command failed"

# Run command AND continue only if successful
command && echo "Command succeeded"

# Chaining
mkdir /tmp/test && cd /tmp/test && touch file.txt

# Exit on error
command || exit 1

# Provide default value
result=$(command 2>/dev/null || echo "default")
```

### Conditional Execution

```bash
#!/bin/bash

# Check if command succeeds
if command -v python3 &> /dev/null
then
    echo "Python3 is installed"
else
    echo "Python3 is not installed"
    exit 1
fi

# More robust file checking
if [ -f "config.txt" ]
then
    source config.txt
else
    echo "Config file not found" >&2
    exit 1
fi
```

## Try-Catch Pattern

Bash doesn't have native try-catch, but we can simulate it.

```bash
#!/bin/bash

# Simulate try-catch
try() {
    # Save whether -e was set and then disable it
    [[ $- = *e* ]]; SAVED_OPT_E=$?
    set +e
}

catch() {
    # Restore -e if it was previously set and propagate exit code
    local exception_code=$?
    (( SAVED_OPT_E == 0 )) && set -e
    return $exception_code
}

# Usage
try
(
    echo "Attempting risky operation..."
    false  # This will fail
    echo "This won't execute"
)
catch || {
    case $exception_code in
        1)
            echo "Caught error code 1"
            ;;
        *)
            echo "Caught error code: $exception_code"
            ;;
    esac
}
```

## Trap Command

The `trap` command allows you to catch signals and execute cleanup code.

### Basic Trap Usage

```bash
#!/bin/bash

# Cleanup function
cleanup() {
    echo "Cleaning up..."
    rm -f /tmp/temp_file_$$
    echo "Done!"
}

# Set trap to run cleanup on exit
trap cleanup EXIT

# Create temp file
echo "Creating temp file..."
touch /tmp/temp_file_$$

# Your script logic here
echo "Running script..."
sleep 2

# cleanup() will be called automatically when script exits
```

### Trapping Specific Signals

```bash
#!/bin/bash

# Handle Ctrl+C (SIGINT)
trap 'echo "Caught SIGINT, cleaning up..."; exit 130' INT

# Handle termination (SIGTERM)
trap 'echo "Caught SIGTERM, cleaning up..."; exit 143' TERM

# Handle multiple signals
trap 'echo "Caught signal, exiting..."; exit 1' INT TERM HUP

echo "Press Ctrl+C to test..."
while true
do
    sleep 1
done
```

### Trap for Error Handling

```bash
#!/bin/bash

set -e

# Error handler
error_handler() {
    local line_number=$1
    echo "Error on line $line_number" >&2
    echo "Script failed!" >&2
    exit 1
}

# Trap errors
trap 'error_handler ${LINENO}' ERR

# Your code
echo "Step 1"
ls /nonexistent  # This will trigger the error handler
echo "This won't execute"
```

### Advanced Trap with Stack Trace

```bash
#!/bin/bash

set -euo pipefail

# Print stack trace
print_stack_trace() {
    local frame=0
    echo "Stack trace:" >&2
    while caller $frame >&2
    do
        ((frame++))
    done
}

# Error handler with stack trace
error_exit() {
    local line_number=$1
    local bash_lineno=$2
    local exit_code=$3

    echo "Error in line $line_number (exit code: $exit_code)" >&2
    print_stack_trace
    exit $exit_code
}

trap 'error_exit ${LINENO} ${BASH_LINENO} $?' ERR

# Test function
function_with_error() {
    echo "In function_with_error"
    false  # This will trigger error
}

# Main
echo "Starting script"
function_with_error
echo "This won't run"
```

## Debugging Techniques

### Debug Mode

```bash
#!/bin/bash

# Enable debug mode (print commands before execution)
set -x

echo "This command will be printed"
ls /tmp

# Disable debug mode
set +x

echo "This command won't show debug output"
```

### Verbose Mode

```bash
#!/bin/bash

# Print commands as they are read
set -v

echo "Hello"
ls

set +v
```

### Conditional Debugging

```bash
#!/bin/bash

# Debug flag
DEBUG=true

debug() {
    if [ "$DEBUG" = true ]
    then
        echo "[DEBUG] $*" >&2
    fi
}

# Usage
debug "Starting script"
debug "Processing file: $filename"
debug "Current value: $counter"
```

### Debugging Functions

```bash
#!/bin/bash

# Print variable values
debug_var() {
    echo "[DEBUG] $1 = ${!1}" >&2
}

# Usage
username="alice"
debug_var username  # Prints: [DEBUG] username = alice

# Print function call
debug_function() {
    echo "[DEBUG] Calling: ${FUNCNAME[1]} with args: $*" >&2
}

# Trace function execution
trace() {
    echo "[TRACE] ${BASH_SOURCE[1]}:${BASH_LINENO[0]} - ${FUNCNAME[1]}" >&2
}
```

### Shell Check and Lint

```bash
# Use shellcheck for static analysis
# Install: apt-get install shellcheck (Linux)
# or: brew install shellcheck (macOS)

# Check your script
shellcheck myscript.sh

# Ignore specific warnings
# shellcheck disable=SC2034
unused_variable="value"
```

## Logging

### Basic Logging

```bash
#!/bin/bash

LOG_FILE="/var/log/myscript.log"

log() {
    local level=$1
    shift
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] [$level] $*" | tee -a "$LOG_FILE"
}

# Usage
log INFO "Script started"
log ERROR "Failed to connect"
log WARNING "Deprecated function used"
```

### Advanced Logging

```bash
#!/bin/bash

# Log levels
readonly LOG_LEVEL_DEBUG=0
readonly LOG_LEVEL_INFO=1
readonly LOG_LEVEL_WARNING=2
readonly LOG_LEVEL_ERROR=3

# Current log level
LOG_LEVEL=$LOG_LEVEL_INFO
LOG_FILE="app.log"

# Logging function
log() {
    local level=$1
    local level_num=$2
    shift 2

    if [ $level_num -ge $LOG_LEVEL ]
    then
        local timestamp=$(date '+%Y-%m-%d %H:%M:%S')
        echo "[$timestamp] [$level] $*" >> "$LOG_FILE"

        # Also print ERROR to stderr
        if [ $level_num -eq $LOG_LEVEL_ERROR ]
        then
            echo "[$timestamp] [$level] $*" >&2
        fi
    fi
}

# Wrapper functions
log_debug() { log "DEBUG" $LOG_LEVEL_DEBUG "$@"; }
log_info() { log "INFO" $LOG_LEVEL_INFO "$@"; }
log_warning() { log "WARNING" $LOG_LEVEL_WARNING "$@"; }
log_error() { log "ERROR" $LOG_LEVEL_ERROR "$@"; }

# Usage
log_debug "Detailed debug information"
log_info "Application started"
log_warning "Configuration file not found, using defaults"
log_error "Database connection failed"
```

## Assertion and Validation

### Assertions

```bash
#!/bin/bash

# Assert function
assert() {
    if ! "$@"
    then
        echo "Assertion failed: $*" >&2
        exit 1
    fi
}

# Usage
assert [ -f "config.txt" ]
assert [ -n "$USERNAME" ]
assert command -v git
```

### Input Validation

```bash
#!/bin/bash

# Validate number
is_number() {
    [[ "$1" =~ ^[0-9]+$ ]]
}

# Validate not empty
is_not_empty() {
    [ -n "$1" ]
}

# Validate file exists
file_exists() {
    [ -f "$1" ]
}

# Usage with error handling
validate_input() {
    local input=$1

    if ! is_not_empty "$input"
    then
        echo "Error: Input cannot be empty" >&2
        return 1
    fi

    if ! is_number "$input"
    then
        echo "Error: Input must be a number" >&2
        return 1
    fi

    return 0
}

# Use validation
read -p "Enter a number: " user_input
if validate_input "$user_input"
then
    echo "Valid input: $user_input"
else
    exit 1
fi
```

## Error Messages

### Redirecting Errors

```bash
#!/bin/bash

# Send errors to stderr
echo "This is an error" >&2

# Redirect stdout and stderr separately
command > output.txt 2> error.txt

# Redirect stderr to stdout
command 2>&1

# Redirect both to same file
command > all_output.txt 2>&1

# Discard errors
command 2>/dev/null

# Discard all output
command &>/dev/null
```

### Formatted Error Messages

```bash
#!/bin/bash

# Error message function
error() {
    echo "ERROR: $*" >&2
}

warning() {
    echo "WARNING: $*" >&2
}

info() {
    echo "INFO: $*"
}

# Usage
error "Failed to connect to database"
warning "Configuration file not found"
info "Processing complete"
```

## Best Practices

1. **Always check return codes** for critical operations
2. **Use `set -euo pipefail`** at the start of scripts
3. **Implement cleanup with trap EXIT**
4. **Validate input parameters** before processing
5. **Log important operations** for debugging
6. **Write meaningful error messages** to stderr
7. **Use shellcheck** for static analysis
8. **Test error paths** not just success paths

## Debugging Example Script

```bash
#!/bin/bash

# Enable strict mode
set -euo pipefail

# Configuration
DEBUG=${DEBUG:-false}
LOG_FILE="script.log"

# Debug function
debug() {
    if [ "$DEBUG" = true ]
    then
        echo "[DEBUG] $*" >&2
    fi
}

# Error handler
error_exit() {
    echo "Error on line $1" >&2
    exit 1
}

# Cleanup function
cleanup() {
    debug "Running cleanup"
    # Add cleanup code here
}

# Set traps
trap cleanup EXIT
trap 'error_exit $LINENO' ERR

# Main script
main() {
    debug "Script started with args: $*"

    # Your code here
    echo "Running main logic..."

    debug "Script completed successfully"
}

# Run main with all arguments
main "$@"
```

## Common Debugging Commands

```bash
# Run script in debug mode
bash -x script.sh

# Check syntax without executing
bash -n script.sh

# Verbose mode
bash -v script.sh

# Enable debugging for specific sections
set -x  # Enable
# ... code to debug ...
set +x  # Disable

# Show line numbers in PS4 prompt
export PS4='+(${BASH_SOURCE}:${LINENO}): ${FUNCNAME[0]:+${FUNCNAME[0]}(): }'
set -x
```

## Common Pitfalls

- `set -e` doesn’t trigger on all failures (e.g., in `if cmd; then ... fi` or in subshells); know its limits.
- `pipefail` changes pipeline behavior; ensure scripts rely on the correct stage’s status.
- `set -u` (nounset) breaks on missing vars; provide defaults like `${var:-}` when appropriate.
- Forgetting to restore debug flags or temporary state after a “try/catch”-style block.
- Traps don’t automatically propagate to subshells; set them where needed.

## Exercises

1. Create a script with `set -euo pipefail` that runs three commands where the second fails; observe exit behavior.
2. Add a `trap 'echo cleanup; rm -f /tmp/demo.*' EXIT` and verify it runs on error and on success.
3. Demonstrate `pipefail` by making the left command in a pipeline fail; check `$?` with and without `pipefail`.
4. Toggle debugging only for a function using `set -x`/`set +x` and a custom `PS4` prompt.

## Cross‑links

- [07 - Functions](./07-functions.md)
- [09 - Text Processing](./09-text-processing.md)

## References

- GNU Bash Reference Manual: <https://www.gnu.org/software/bash/manual/bash.html>
