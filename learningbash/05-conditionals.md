# Conditional Statements and Decision Making

[Previous: 04 - User Input and Arguments](./04-user-input-and-arguments.md) • [Next: 06 - Loops](./06-loops.md)

## Introduction

Conditional statements allow scripts to make decisions and execute different code based on conditions. Essential for any DevOps automation.

## Test Command `[ ]` vs `[[ ]]`

### Single Brackets `[ ]` (POSIX)

```bash
# Old style, POSIX-compliant
if [ "$var" = "value" ]; then
    echo "Match"
fi
```

### Double Brackets `[[ ]]` (Bash Extended)

```bash
# Modern bash style (RECOMMENDED)
if [[ "$var" == "value" ]]; then
    echo "Match"
fi
```

**Why use `[[ ]]`?**

- Pattern matching support
- No word splitting
- No pathname expansion
- More operators (&&, ||, <, >)
- Safer with variables

**Comparison:**

```bash
# Problematic with [ ]
var="hello world"
if [ $var == "hello world" ]; then    # ERROR: too many arguments
    echo "Match"
fi

# Works fine with [[ ]]
if [[ $var == "hello world" ]]; then  # OK
    echo "Match"
fi

# Better with quotes in [ ]
if [ "$var" == "hello world" ]; then  # OK
    echo "Match"
fi
```

## If Statement

### Basic If

```bash
#!/bin/bash

if [[ condition ]]; then
    # code to execute
fi
```

### If-Else

```bash
#!/bin/bash

if [[ condition ]]; then
    # code if true
else
    # code if false
fi
```

### If-Elif-Else

```bash
#!/bin/bash

if [[ condition1 ]]; then
    # code if condition1 is true
elif [[ condition2 ]]; then
    # code if condition2 is true
elif [[ condition3 ]]; then
    # code if condition3 is true
else
    # code if all conditions are false
fi
```

### One-Line If

```bash
[[ condition ]] && echo "true" || echo "false"

# But be careful - this can be tricky
[[ condition ]] && command1 && command2 || echo "failed"
```

## String Comparisons

```bash
#!/bin/bash

str1="hello"
str2="world"

# Equality
if [[ "$str1" == "$str2" ]]; then
    echo "Strings are equal"
fi

# Inequality
if [[ "$str1" != "$str2" ]]; then
    echo "Strings are different"
fi

# Less than (alphabetically)
if [[ "$str1" < "$str2" ]]; then
    echo "$str1 comes before $str2"
fi

# Greater than (alphabetically)
if [[ "$str1" > "$str2" ]]; then
    echo "$str1 comes after $str2"
fi

# Check if empty
if [[ -z "$str1" ]]; then
    echo "String is empty"
fi

# Check if not empty
if [[ -n "$str1" ]]; then
    echo "String is not empty"
fi

# Check if string matches pattern
if [[ "$str1" == h* ]]; then
    echo "String starts with h"
fi

# Regex matching
if [[ "$str1" =~ ^[a-z]+$ ]]; then
    echo "String contains only lowercase letters"
fi
```

## Numeric Comparisons

```bash
#!/bin/bash

num1=10
num2=20

# Equal
if [[ $num1 -eq $num2 ]]; then
    echo "Numbers are equal"
fi

# Not equal
if [[ $num1 -ne $num2 ]]; then
    echo "Numbers are different"
fi

# Less than
if [[ $num1 -lt $num2 ]]; then
    echo "$num1 is less than $num2"
fi

# Less than or equal
if [[ $num1 -le $num2 ]]; then
    echo "$num1 is less than or equal to $num2"
fi

# Greater than
if [[ $num1 -gt $num2 ]]; then
    echo "$num1 is greater than $num2"
fi

# Greater than or equal
if [[ $num1 -ge $num2 ]]; then
    echo "$num1 is greater than or equal to $num2"
fi
```

### Arithmetic Comparison (Alternative)

```bash
#!/bin/bash

num1=10
num2=20

# Using (( ))
if ((num1 < num2)); then
    echo "$num1 is less than $num2"
fi

if ((num1 == 10)); then
    echo "num1 equals 10"
fi

if ((num1 != num2)); then
    echo "Numbers are different"
fi
```

## File Tests

```bash
#!/bin/bash

file="/path/to/file"
dir="/path/to/directory"

# File exists
if [[ -e "$file" ]]; then
    echo "File exists"
fi

# Regular file exists
if [[ -f "$file" ]]; then
    echo "File exists and is a regular file"
fi

# Directory exists
if [[ -d "$dir" ]]; then
    echo "Directory exists"
fi

# File is readable
if [[ -r "$file" ]]; then
    echo "File is readable"
fi

# File is writable
if [[ -w "$file" ]]; then
    echo "File is writable"
fi

# File is executable
if [[ -x "$file" ]]; then
    echo "File is executable"
fi

# File is not empty
if [[ -s "$file" ]]; then
    echo "File is not empty"
fi

# File is a symbolic link
if [[ -L "$file" ]]; then
    echo "File is a symbolic link"
fi

# File1 is newer than File2
if [[ "$file1" -nt "$file2" ]]; then
    echo "File1 is newer than File2"
fi

# File1 is older than File2
if [[ "$file1" -ot "$file2" ]]; then
    echo "File1 is older than File2"
fi
```

### Common File Test Examples

```bash
#!/bin/bash

# Check if config file exists before reading
config_file="/etc/myapp/config.conf"
if [[ -f "$config_file" ]]; then
    source "$config_file"
else
    echo "Config file not found: $config_file"
    exit 1
fi

# Check if log directory exists, create if not
log_dir="/var/log/myapp"
if [[ ! -d "$log_dir" ]]; then
    mkdir -p "$log_dir"
    echo "Created log directory: $log_dir"
fi

# Check if script has execute permission
script="deploy.sh"
if [[ ! -x "$script" ]]; then
    chmod +x "$script"
    echo "Made script executable: $script"
fi
```

## Logical Operators

### AND (&&)

```bash
#!/bin/bash

# Both conditions must be true
if [[ condition1 && condition2 ]]; then
    echo "Both conditions are true"
fi

# Alternative: -a (avoid with [[ ]])
if [[ condition1 ]] && [[ condition2 ]]; then
    echo "Both conditions are true"
fi
```

### OR (||)

```bash
#!/bin/bash

# At least one condition must be true
if [[ condition1 || condition2 ]]; then
    echo "At least one condition is true"
fi

# Alternative
if [[ condition1 ]] || [[ condition2 ]]; then
    echo "At least one condition is true"
fi
```

### NOT (!)

```bash
#!/bin/bash

# Negate condition
if [[ ! condition ]]; then
    echo "Condition is false"
fi

# Example
if [[ ! -f "$file" ]]; then
    echo "File does not exist"
fi
```

### Complex Conditions

```bash
#!/bin/bash

environment="production"
service_status="running"
cpu_usage=85

# Multiple AND conditions
if [[ "$environment" == "production" && "$service_status" == "running" && $cpu_usage -lt 90 ]]; then
    echo "Production service is healthy"
fi

# Mixed AND/OR
if [[ "$environment" == "production" || "$environment" == "staging" ]] && [[ $cpu_usage -lt 90 ]]; then
    echo "Environment is acceptable"
fi

# Grouped conditions
if [[ ("$environment" == "production" || "$environment" == "staging") && $cpu_usage -lt 90 ]]; then
    echo "Conditions met"
fi
```

## Case Statement

The case statement is perfect for matching against multiple patterns.

### Basic Case

```bash
#!/bin/bash

case "$variable" in
    pattern1)
        # commands
        ;;
    pattern2)
        # commands
        ;;
    *)
        # default commands
        ;;
esac
```

### Practical Examples

#### Menu Selection

```bash
#!/bin/bash

read -p "Select environment (dev/staging/prod): " env

case "$env" in
    dev|development)
        echo "Deploying to development"
        SERVER="dev.example.com"
        ;;
    staging|stage)
        echo "Deploying to staging"
        SERVER="staging.example.com"
        ;;
    prod|production)
        echo "Deploying to production"
        SERVER="prod.example.com"
        ;;
    *)
        echo "Invalid environment: $env"
        exit 1
        ;;
esac

echo "Server: $SERVER"
```

#### File Type Detection

```bash
#!/bin/bash

filename="script.sh"

case "$filename" in
    *.sh)
        echo "Bash script"
        ;;
    *.py)
        echo "Python script"
        ;;
    *.js)
        echo "JavaScript file"
        ;;
    *.txt)
        echo "Text file"
        ;;
    *)
        echo "Unknown file type"
        ;;
esac
```

#### Pattern Matching

```bash
#!/bin/bash

user_input="yes"

case "$user_input" in
    [yY]|[yY][eE][sS])
        echo "User confirmed"
        ;;
    [nN]|[nN][oO])
        echo "User declined"
        ;;
    *)
        echo "Invalid input"
        ;;
esac
```

#### Service Control

```bash
#!/bin/bash

action="$1"

case "$action" in
    start)
        echo "Starting service..."
        # systemctl start myservice
        ;;
    stop)
        echo "Stopping service..."
        # systemctl stop myservice
        ;;
    restart)
        echo "Restarting service..."
        # systemctl restart myservice
        ;;
    status)
        echo "Checking service status..."
        # systemctl status myservice
        ;;
    *)
        echo "Usage: $0 {start|stop|restart|status}"
        exit 1
        ;;
esac
```

## Real-World DevOps Examples

### Health Check Script

```bash
#!/bin/bash

set -euo pipefail

readonly SERVICE="nginx"
readonly MAX_CPU=80
readonly MAX_MEMORY=85
readonly LOG_FILE="/var/log/health_check.log"

log_message() {
    echo "$(date '+%Y-%m-%d %H:%M:%S') - $*" | tee -a "$LOG_FILE"
}

check_service() {
    if systemctl is-active --quiet "$SERVICE"; then
        log_message "[OK] Service $SERVICE is running"
        return 0
    else
        log_message "[ERROR] Service $SERVICE is not running"
        return 1
    fi
}

check_cpu() {
    local cpu_usage
    cpu_usage=$(top -bn1 | grep "Cpu(s)" | awk '{print $2}' | cut -d'%' -f1 | cut -d'.' -f1)

    if [[ $cpu_usage -lt $MAX_CPU ]]; then
        log_message "[OK] CPU usage: ${cpu_usage}%"
        return 0
    else
        log_message "[WARN] CPU usage high: ${cpu_usage}%"
        return 1
    fi
}

check_memory() {
    local mem_usage
    mem_usage=$(free | grep Mem | awk '{print int($3/$2 * 100)}')

    if [[ $mem_usage -lt $MAX_MEMORY ]]; then
        log_message "[OK] Memory usage: ${mem_usage}%"
        return 0
    else
        log_message "[WARN] Memory usage high: ${mem_usage}%"
        return 1
    fi
}

check_disk() {
    local disk_usage
    disk_usage=$(df -h / | tail -1 | awk '{print $5}' | cut -d'%' -f1)

    if [[ $disk_usage -lt 85 ]]; then
        log_message "[OK] Disk usage: ${disk_usage}%"
        return 0
    else
        log_message "[WARN] Disk usage high: ${disk_usage}%"
        return 1
    fi
}

main() {
    log_message "=== Starting health check ==="

    local exit_code=0

    check_service || exit_code=1
    check_cpu || exit_code=1
    check_memory || exit_code=1
    check_disk || exit_code=1

    if [[ $exit_code -eq 0 ]]; then
        log_message "=== Health check PASSED ==="
    else
        log_message "=== Health check FAILED ==="
    fi

    exit $exit_code
}

main
```

### Deployment Validation

```bash
#!/bin/bash

set -euo pipefail

ENVIRONMENT="${1:-}"
BRANCH="${2:-main}"

# Validation function
validate_deployment() {
    local errors=0

    # Check environment
    if [[ -z "$ENVIRONMENT" ]]; then
        echo "Error: Environment not specified"
        ((errors++))
    elif [[ ! "$ENVIRONMENT" =~ ^(dev|staging|production)$ ]]; then
        echo "Error: Invalid environment: $ENVIRONMENT"
        ((errors++))
    fi

    # Check branch
    if [[ -z "$BRANCH" ]]; then
        echo "Error: Branch not specified"
        ((errors++))
    fi

    # Production specific checks
    if [[ "$ENVIRONMENT" == "production" ]]; then
        if [[ "$BRANCH" != "main" && "$BRANCH" != "master" ]]; then
            echo "Error: Production must deploy from main/master branch"
            ((errors++))
        fi

        # Check if tests passed
        if [[ ! -f ".test_passed" ]]; then
            echo "Error: Tests must pass before production deployment"
            ((errors++))
        fi
    fi

    # Check if Git is clean
    if [[ -n "$(git status --porcelain)" ]]; then
        echo "Error: Git working directory is not clean"
        ((errors++))
    fi

    return $errors
}

# Run validation
if validate_deployment; then
    echo "Validation passed - proceeding with deployment"
    # Actual deployment code here
else
    echo "Validation failed - deployment aborted"
    exit 1
fi
```

### Backup Script with Conditions

```bash
#!/bin/bash

set -euo pipefail

readonly BACKUP_DIR="/backup"
readonly SOURCE_DIR="/var/www"
readonly MAX_BACKUPS=7
readonly TIMESTAMP=$(date +%Y%m%d_%H%M%S)
readonly BACKUP_FILE="${BACKUP_DIR}/backup_${TIMESTAMP}.tar.gz"

# Check if backup directory exists
if [[ ! -d "$BACKUP_DIR" ]]; then
    echo "Creating backup directory: $BACKUP_DIR"
    mkdir -p "$BACKUP_DIR"
fi

# Check if source directory exists
if [[ ! -d "$SOURCE_DIR" ]]; then
    echo "Error: Source directory does not exist: $SOURCE_DIR"
    exit 1
fi

# Check available disk space (need at least 5GB)
available_space=$(df -BG "$BACKUP_DIR" | tail -1 | awk '{print $4}' | sed 's/G//')
if [[ $available_space -lt 5 ]]; then
    echo "Error: Insufficient disk space. Available: ${available_space}GB, Required: 5GB"
    exit 1
fi

# Perform backup
echo "Creating backup: $BACKUP_FILE"
if tar -czf "$BACKUP_FILE" -C "$SOURCE_DIR" .; then
    echo "Backup created successfully"

    # Check backup file size
    if [[ -s "$BACKUP_FILE" ]]; then
        backup_size=$(du -h "$BACKUP_FILE" | cut -f1)
        echo "Backup size: $backup_size"
    else
        echo "Warning: Backup file is empty"
        exit 1
    fi
else
    echo "Error: Backup failed"
    exit 1
fi

# Cleanup old backups
backup_count=$(ls -1 "${BACKUP_DIR}"/backup_*.tar.gz 2>/dev/null | wc -l)
if [[ $backup_count -gt $MAX_BACKUPS ]]; then
    echo "Removing old backups (keeping last $MAX_BACKUPS)"
    ls -1t "${BACKUP_DIR}"/backup_*.tar.gz | tail -n +$((MAX_BACKUPS + 1)) | xargs rm -f
fi

echo "Backup completed successfully"
```

## Practice Exercise

Create a system monitoring script that:

1. Checks if required services are running (nginx, mysql, redis)
2. Validates CPU and memory usage
3. Checks disk space
4. Sends different alerts based on severity
5. Logs all checks
6. Returns appropriate exit codes

**Starter template:**

```bash
#!/bin/bash

set -euo pipefail

# Configuration
readonly SERVICES=("nginx" "mysql" "redis")
readonly CPU_WARNING=70
readonly CPU_CRITICAL=90
readonly MEM_WARNING=75
readonly MEM_CRITICAL=90

check_services() {
    # Implement service checks
    :
}

check_resources() {
    # Implement resource checks
    :
}

send_alert() {
    local severity="$1"
    local message="$2"
    # Implement alerting
    :
}

main() {
    # Implement main logic
    :
}

main "$@"
```

---

## Common Pitfalls

- Using `[` instead of `[[` can trigger globbing and word-splitting surprises.
- Mixing numeric and string operators (`-eq` vs `==`); use the right one for the type.
- Forgetting quotes around variables in tests: `[[ -n "$var" ]]`.
- File tests against the wrong path or a symlink when you meant the target.
- Complex `if` chains that would be cleaner as a `case` statement.

## Exercises

1. Write a script that checks: file exists and is readable; directory exists and is writable.
2. Prompt for a number and print whether it’s negative, zero, or positive using `if/elif/else`.
3. Build a `case` menu that handles `start|stop|status` with a default “unknown command”.
4. Implement guard clauses that exit early on invalid input instead of deep nesting.

## Cross‑links

- [04 - User Input and Arguments](./04-user-input-and-arguments.md)
- [06 - Loops](./06-loops.md)

## References

- GNU Bash Reference Manual: <https://www.gnu.org/software/bash/manual/bash.html>
