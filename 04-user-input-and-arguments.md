# User Input and Command-Line Arguments

[Previous: 03 - Variables and Data Types](./03-variables-and-data-types.md) • [Next: 05 - Conditionals](./05-conditionals.md)

## Reading User Input with `read`

### Basic Input

```bash
#!/bin/bash

echo "What is your name?"
read name
echo "Hello, $name!"
```

### Input with Prompt (inline)

```bash
#!/bin/bash

read -p "Enter your name: " name
echo "Hello, $name!"
```

### Multiple Inputs

```bash
#!/bin/bash

read -p "Enter username and email: " username email
echo "Username: $username"
echo "Email: $email"

# User input: john john@example.com
# username = john
# email = john@example.com
```

### Reading Password (hidden input)

```bash
#!/bin/bash

read -sp "Enter password: " password
echo  # New line after hidden input
echo "Password received (hidden)"
```

### Read with Timeout

```bash
#!/bin/bash

if read -t 5 -p "Enter your name (5 seconds): " name; then
    echo "Hello, $name!"
else
    echo "Timeout! Using default name."
    name="Guest"
fi
```

### Read with Default Value

```bash
#!/bin/bash

read -p "Enter environment [development]: " environment
environment=${environment:-development}
echo "Using environment: $environment"
```

### Read Single Character

```bash
#!/bin/bash

read -n 1 -p "Continue? (y/n): " answer
echo
if [[ $answer == "y" ]]; then
    echo "Continuing..."
else
    echo "Aborted."
fi
```

### Reading from File

```bash
#!/bin/bash

while IFS= read -r line; do
    echo "Line: $line"
done < input.txt
```

### Reading into Array

```bash
#!/bin/bash

echo "Enter three servers:"
read -a servers
echo "You entered: ${servers[@]}"
echo "First server: ${servers[0]}"
```

## Command-Line Arguments

### Basic Arguments

```bash
#!/bin/bash

echo "Script name: $0"
echo "First argument: $1"
echo "Second argument: $2"
echo "Third argument: $3"

# Usage: ./script.sh arg1 arg2 arg3
```

### Checking Number of Arguments

```bash
#!/bin/bash

if [[ $# -eq 0 ]]; then
    echo "Error: No arguments provided"
    echo "Usage: $0 <filename>"
    exit 1
fi

filename=$1
echo "Processing file: $filename"
```

### Looping Through All Arguments

```bash
#!/bin/bash

echo "Processing $# arguments..."

for arg in "$@"; do
    echo "Argument: $arg"
done

# Or with index
for ((i=1; i<=$#; i++)); do
    echo "Argument $i: ${!i}"
done
```

### Difference Between $\* and $@

```bash
#!/bin/bash

# Without quotes - both behave the same
for arg in $*; do
    echo "arg: $arg"
done

# With quotes - different behavior
for arg in "$*"; do
    echo "arg: $arg"  # All args as ONE string
done

for arg in "$@"; do
    echo "arg: $arg"  # Each arg separate (PREFERRED)
done
```

**Example:**

```bash
./script.sh "hello world" "foo bar"

# Using "$*" - ONE iteration: "hello world foo bar"
# Using "$@" - TWO iterations: "hello world", "foo bar"
```

### Shifting Arguments

```bash
#!/bin/bash

echo "Before shift: $1 $2 $3"

shift  # Remove first argument, shift all left

echo "After shift: $1 $2 $3"

shift 2  # Shift by 2 positions

echo "After shift 2: $1"
```

**Example:**

```bash
./script.sh arg1 arg2 arg3 arg4

# Before shift: arg1 arg2 arg3
# After shift: arg2 arg3 arg4
# After shift 2: arg4
```

## Parsing Command-Line Options

### Simple Flag Parsing

```bash
#!/bin/bash

verbose=false
debug=false

for arg in "$@"; do
    case $arg in
        -v|--verbose)
            verbose=true
            ;;
        -d|--debug)
            debug=true
            ;;
        *)
            echo "Unknown option: $arg"
            exit 1
            ;;
    esac
done

echo "Verbose: $verbose"
echo "Debug: $debug"
```

### Options with Values

```bash
#!/bin/bash

# Initialize defaults
environment="development"
config_file=""
port=8080

while [[ $# -gt 0 ]]; do
    case $1 in
        -e|--environment)
            environment="$2"
            shift 2
            ;;
        -c|--config)
            config_file="$2"
            shift 2
            ;;
        -p|--port)
            port="$2"
            shift 2
            ;;
        -h|--help)
            echo "Usage: $0 [OPTIONS]"
            echo "Options:"
            echo "  -e, --environment ENV    Set environment"
            echo "  -c, --config FILE        Config file path"
            echo "  -p, --port PORT          Port number"
            echo "  -h, --help               Show this help"
            exit 0
            ;;
        *)
            echo "Unknown option: $1"
            exit 1
            ;;
    esac
done

echo "Environment: $environment"
echo "Config: $config_file"
echo "Port: $port"
```

### Using getopts (Built-in)

```bash
#!/bin/bash

# For short options only: -a -b -c value

verbose=false
output=""

while getopts ":vho:" opt; do
    case $opt in
        v)
            verbose=true
            ;;
        o)
            output="$OPTARG"
            ;;
        h)
            echo "Usage: $0 [-v] [-o output]"
            exit 0
            ;;
        \?)
            echo "Invalid option: -$OPTARG"
            exit 1
            ;;
        :)
            echo "Option -$OPTARG requires an argument"
            exit 1
            ;;
    esac
done

shift $((OPTIND - 1))  # Remove parsed options

echo "Verbose: $verbose"
echo "Output: $output"
echo "Remaining args: $@"
```

**getopts syntax:**

- `:` at start: Silent error reporting
- `:` after letter: Option requires argument
- No `:`: Option is flag only

```bash
# Examples:
getopts "abc"      # -a, -b, -c (flags)
getopts "a:bc"     # -a value, -b, -c
getopts ":a:b:c"   # -a value, -b value, -c (silent errors)
```

## Complete DevOps Script Example

```bash
#!/bin/bash

set -euo pipefail

# Script metadata
readonly SCRIPT_NAME="$(basename "$0")"
readonly VERSION="1.0.0"

# Default values
ENVIRONMENT="development"
CONFIG_FILE=""
VERBOSE=false
DRY_RUN=false
SERVICE_NAME=""

# Colors
readonly RED='\033[0;31m'
readonly GREEN='\033[0;32m'
readonly YELLOW='\033[1;33m'
readonly NC='\033[0m'

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

log_debug() {
    if [[ $VERBOSE == true ]]; then
        echo -e "[DEBUG] $*"
    fi
}

# Help function
show_help() {
    cat << EOF
Usage: $SCRIPT_NAME [OPTIONS] SERVICE_NAME

Deploy a service to specified environment

Options:
    -e, --environment ENV     Target environment (default: development)
    -c, --config FILE         Configuration file path
    -v, --verbose             Enable verbose output
    -n, --dry-run             Dry run mode (no actual changes)
    -h, --help                Show this help message
    --version                 Show version information

Arguments:
    SERVICE_NAME              Name of the service to deploy

Examples:
    $SCRIPT_NAME -e production web-api
    $SCRIPT_NAME --config deploy.conf --verbose backend-service
    $SCRIPT_NAME --dry-run frontend

EOF
}

# Parse arguments
parse_arguments() {
    while [[ $# -gt 0 ]]; do
        case $1 in
            -e|--environment)
                ENVIRONMENT="$2"
                shift 2
                ;;
            -c|--config)
                CONFIG_FILE="$2"
                if [[ ! -f "$CONFIG_FILE" ]]; then
                    log_error "Config file not found: $CONFIG_FILE"
                    exit 1
                fi
                shift 2
                ;;
            -v|--verbose)
                VERBOSE=true
                shift
                ;;
            -n|--dry-run)
                DRY_RUN=true
                shift
                ;;
            -h|--help)
                show_help
                exit 0
                ;;
            --version)
                echo "$SCRIPT_NAME version $VERSION"
                exit 0
                ;;
            -*)
                log_error "Unknown option: $1"
                show_help
                exit 1
                ;;
            *)
                if [[ -z $SERVICE_NAME ]]; then
                    SERVICE_NAME="$1"
                else
                    log_error "Multiple service names provided"
                    exit 1
                fi
                shift
                ;;
        esac
    done

    # Validate required arguments
    if [[ -z $SERVICE_NAME ]]; then
        log_error "Service name is required"
        show_help
        exit 1
    fi

    # Validate environment
    case $ENVIRONMENT in
        development|staging|production)
            ;;
        *)
            log_error "Invalid environment: $ENVIRONMENT"
            log_error "Valid options: development, staging, production"
            exit 1
            ;;
    esac
}

# Interactive confirmation
confirm_deployment() {
    echo
    log_info "Deployment Summary:"
    echo "  Service: $SERVICE_NAME"
    echo "  Environment: $ENVIRONMENT"
    echo "  Config: ${CONFIG_FILE:-'default'}"
    echo "  Dry Run: $DRY_RUN"
    echo

    if [[ $DRY_RUN == true ]]; then
        log_warn "This is a DRY RUN - no actual changes will be made"
        return 0
    fi

    if [[ $ENVIRONMENT == "production" ]]; then
        log_warn "You are about to deploy to PRODUCTION!"
        read -p "Type 'yes' to continue: " confirmation
        if [[ $confirmation != "yes" ]]; then
            log_info "Deployment cancelled"
            exit 0
        fi
    else
        read -n 1 -p "Continue? (y/n): " answer
        echo
        if [[ $answer != "y" ]]; then
            log_info "Deployment cancelled"
            exit 0
        fi
    fi
}

# Main deployment function
deploy_service() {
    log_info "Starting deployment of $SERVICE_NAME to $ENVIRONMENT"
    log_debug "Verbose mode enabled"

    if [[ $DRY_RUN == true ]]; then
        log_info "[DRY RUN] Would deploy $SERVICE_NAME"
        return 0
    fi

    # Actual deployment logic here
    log_info "Pulling latest image..."
    log_info "Stopping old container..."
    log_info "Starting new container..."
    log_info "Running health checks..."

    log_info "Deployment completed successfully!"
}

# Main function
main() {
    parse_arguments "$@"
    confirm_deployment
    deploy_service
}

# Run main function
main "$@"
```

## Interactive Menus

### Simple Menu

```bash
#!/bin/bash

echo "Select an option:"
echo "1) Start service"
echo "2) Stop service"
echo "3) Restart service"
echo "4) Exit"

read -p "Enter choice [1-4]: " choice

case $choice in
    1)
        echo "Starting service..."
        ;;
    2)
        echo "Stopping service..."
        ;;
    3)
        echo "Restarting service..."
        ;;
    4)
        echo "Exiting..."
        exit 0
        ;;
    *)
        echo "Invalid choice"
        exit 1
        ;;
esac
```

### Loop Menu

```bash
#!/bin/bash

while true; do
    clear
    echo "=== Service Manager ==="
    echo "1) Start"
    echo "2) Stop"
    echo "3) Status"
    echo "4) Exit"
    echo

    read -p "Select option: " choice

    case $choice in
        1)
            echo "Starting service..."
            sleep 2
            ;;
        2)
            echo "Stopping service..."
            sleep 2
            ;;
        3)
            echo "Checking status..."
            sleep 2
            ;;
        4)
            echo "Goodbye!"
            exit 0
            ;;
        *)
            echo "Invalid option"
            sleep 2
            ;;
    esac
done
```

### Select Menu (Built-in)

```bash
#!/bin/bash

PS3="Select environment: "
options=("Development" "Staging" "Production" "Quit")

select opt in "${options[@]}"; do
    case $opt in
        "Development")
            echo "Deploying to development..."
            break
            ;;
        "Staging")
            echo "Deploying to staging..."
            break
            ;;
        "Production")
            echo "Deploying to production..."
            break
            ;;
        "Quit")
            echo "Exiting..."
            exit 0
            ;;
        *)
            echo "Invalid option"
            ;;
    esac
done
```

## Input Validation

```bash
#!/bin/bash

# Validate non-empty input
read -p "Enter username: " username
while [[ -z $username ]]; do
    echo "Username cannot be empty"
    read -p "Enter username: " username
done

# Validate number
read -p "Enter port number: " port
while ! [[ $port =~ ^[0-9]+$ ]]; do
    echo "Port must be a number"
    read -p "Enter port number: " port
done

# Validate range
while ((port < 1 || port > 65535)); do
    echo "Port must be between 1 and 65535"
    read -p "Enter port number: " port
done

# Validate email format
read -p "Enter email: " email
while ! [[ $email =~ ^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$ ]]; do
    echo "Invalid email format"
    read -p "Enter email: " email
done

# Validate yes/no
read -p "Continue? (yes/no): " answer
while [[ $answer != "yes" && $answer != "no" ]]; do
    echo "Please answer yes or no"
    read -p "Continue? (yes/no): " answer
done
```

## Practice Exercise

Create a deployment script that:

1. Takes service name as argument
2. Accepts optional flags: -e (environment), -v (verbose)
3. Asks for confirmation before deploying to production
4. Uses interactive menu if no arguments provided
5. Validates all inputs

**Starter template:**

```bash
#!/bin/bash

set -euo pipefail

# Your solution here

main() {
    # Parse arguments or show menu
    # Validate inputs
    # Confirm deployment
    # Execute deployment
    echo "Deployment script - implement me!"
}

main "$@"
```

---

**Next:** Learn about conditional statements and decision making in bash.

## Common Pitfalls

- Using `$*` instead of `"$@"` loses argument boundaries when spaces are present.
- Using `read` without `-r` allows backslash escapes; prefer `read -r`.
- Forgetting to quote variable expansions when comparing input.
- Writing custom parsers that break on edge cases; prefer `getopts` for simple flags.
- Trusting user input without validation; always check required fields and formats.

## Exercises

1. Prompt the user for a non-empty username and a valid email (basic regex). Re-prompt on invalid input.
2. Implement a CLI with `-v/--verbose` and `-o/--output FILE` using `getopts`; print parsed values.
3. Build a simple interactive menu (1–3) using `select` or a loop with `read`.
4. Parse positional arguments for a mini copy tool: `script.sh SRC DST [-f]` (force overwrite option).

## Cross‑links

- [03 - Variables and Data Types](./03-variables-and-data-types.md)
- [05 - Conditionals](./05-conditionals.md)

## References

- GNU Bash Reference Manual: <https://www.gnu.org/software/bash/manual/bash.html>
