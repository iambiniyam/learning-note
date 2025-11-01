# Functions in Bash

[Previous: 06 - Loops](./06-loops.md) • [Next: 08 - Error Handling and Debugging](./08-error-handling-and-debugging.md)

Functions allow you to group commands into reusable blocks of code. They help organize scripts and avoid code duplication.

## Defining Functions

### Basic Function Syntax

```bash
#!/bin/bash

# Method 1: Using 'function' keyword
function greet {
    echo "Hello, World!"
}

# Method 2: Without 'function' keyword (preferred)
say_goodbye() {
    echo "Goodbye!"
}

# Call the functions
greet
say_goodbye
```

### Function with Parameters

```bash
#!/bin/bash

# Function that accepts parameters
greet_user() {
    echo "Hello, $1!"
    echo "Welcome to $2"
}

# Call with arguments
greet_user "Alice" "Bash scripting"
# Output:
# Hello, Alice!
# Welcome to Bash scripting
```

## Function Parameters

### Accessing Parameters

```bash
#!/bin/bash

print_args() {
    echo "First argument: $1"
    echo "Second argument: $2"
    echo "All arguments: $@"
    echo "Number of arguments: $#"
    echo "Script name: $0"
    echo "Function name: ${FUNCNAME[0]}"
}

print_args apple banana cherry
```

### Default Parameter Values

```bash
#!/bin/bash

# Function with default values
greet() {
    local name=${1:-"Guest"}  # Default to "Guest" if no argument
    local greeting=${2:-"Hello"}
    echo "$greeting, $name!"
}

greet                    # Hello, Guest!
greet "Alice"           # Hello, Alice!
greet "Bob" "Hi"        # Hi, Bob!
```

### Processing All Arguments

```bash
#!/bin/bash

# Loop through all arguments
sum_numbers() {
    local total=0
    for num in "$@"
    do
        ((total += num))
    done
    echo "Sum: $total"
}

sum_numbers 10 20 30 40  # Sum: 100
```

## Return Values

### Using Return Statement

```bash
#!/bin/bash

# Return exit status (0-255)
is_even() {
    local num=$1
    if [ $((num % 2)) -eq 0 ]
    then
        return 0  # True (success)
    else
        return 1  # False (failure)
    fi
}

# Check return value
is_even 4
if [ $? -eq 0 ]
then
    echo "Number is even"
else
    echo "Number is odd"
fi
```

### Using Echo for Return Values

```bash
#!/bin/bash

# Return string value using echo
add_numbers() {
    local num1=$1
    local num2=$2
    local sum=$((num1 + num2))
    echo $sum  # Output the result
}

# Capture output
result=$(add_numbers 5 3)
echo "Result: $result"  # Result: 8
```

### Returning Multiple Values

```bash
#!/bin/bash

# Return multiple values as space-separated string
get_user_info() {
    local name="Alice"
    local age=30
    local city="New York"
    echo "$name $age $city"
}

# Capture and parse multiple values
read -r name age city <<< "$(get_user_info)"
echo "Name: $name, Age: $age, City: $city"
```

## Local vs Global Variables

### Local Variables

```bash
#!/bin/bash

global_var="I am global"

my_function() {
    local local_var="I am local"
    global_var="Modified global"

    echo "Inside function:"
    echo "  Local: $local_var"
    echo "  Global: $global_var"
}

my_function
echo "Outside function:"
echo "  Global: $global_var"
# echo "  Local: $local_var"  # This would be empty
```

### Variable Scope Best Practices

```bash
#!/bin/bash

# Always use 'local' for function variables
calculate_area() {
    local width=$1
    local height=$2
    local area=$((width * height))
    echo $area
}

# Global variables for configuration
readonly CONFIG_FILE="/etc/myapp.conf"
readonly MAX_RETRIES=3
```

## Recursive Functions

```bash
#!/bin/bash

# Factorial using recursion
factorial() {
    local num=$1
    if [ $num -le 1 ]
    then
        echo 1
    else
        local prev=$(factorial $((num - 1)))
        echo $((num * prev))
    fi
}

result=$(factorial 5)
echo "Factorial of 5: $result"  # 120

# Fibonacci sequence
fibonacci() {
    local n=$1
    if [ $n -le 1 ]
    then
        echo $n
    else
        local a=$(fibonacci $((n - 1)))
        local b=$(fibonacci $((n - 2)))
        echo $((a + b))
    fi
}
```

## Function Libraries

### Creating a Library File

```bash
# File: lib/string_utils.sh

# String utility functions
to_uppercase() {
    echo "$1" | tr '[:lower:]' '[:upper:]'
}

to_lowercase() {
    echo "$1" | tr '[:upper:]' '[:lower:]'
}

trim() {
    echo "$1" | sed 's/^[[:space:]]*//;s/[[:space:]]*$//'
}
```

### Sourcing Library Files

```bash
#!/bin/bash
# File: main.sh

# Source the library
source ./lib/string_utils.sh
# Or use: . ./lib/string_utils.sh

# Use library functions
text="  Hello World  "
upper=$(to_uppercase "$text")
lower=$(to_lowercase "$text")
trimmed=$(trim "$text")

echo "Upper: $upper"
echo "Lower: $lower"
echo "Trimmed: '$trimmed'"
```

## Advanced Function Techniques

### Functions with Named Parameters

```bash
#!/bin/bash

# Simulate named parameters
create_user() {
    local name=""
    local email=""
    local role="user"

    # Parse named arguments
    while [ $# -gt 0 ]
    do
        case "$1" in
            --name)
                name="$2"
                shift 2
                ;;
            --email)
                email="$2"
                shift 2
                ;;
            --role)
                role="$2"
                shift 2
                ;;
            *)
                echo "Unknown parameter: $1"
                return 1
                ;;
        esac
    done

    echo "Creating user: $name ($email) with role: $role"
}

# Call with named parameters
create_user --name "Alice" --email "alice@example.com" --role "admin"
```

### Function Error Handling

```bash
#!/bin/bash

# Function with error handling
divide() {
    local numerator=$1
    local denominator=$2

    # Validate inputs
    if ! [[ "$numerator" =~ ^[0-9]+$ ]] || ! [[ "$denominator" =~ ^[0-9]+$ ]]
    then
        echo "Error: Both arguments must be numbers" >&2
        return 1
    fi

    if [ $denominator -eq 0 ]
    then
        echo "Error: Division by zero" >&2
        return 2
    fi

    # Perform division
    echo $((numerator / denominator))
    return 0
}

# Use the function with error checking
result=$(divide 10 2)
if [ $? -eq 0 ]
then
    echo "Result: $result"
else
    echo "Operation failed"
fi
```

### Function Documentation

```bash
#!/bin/bash

# Function: backup_file
# Description: Creates a backup of a file with timestamp
# Arguments:
#   $1 - Source file path (required)
#   $2 - Backup directory (optional, defaults to ./backups)
# Returns:
#   0 - Success
#   1 - Source file not found
#   2 - Cannot create backup directory
# Example:
#   backup_file "/etc/config.conf" "/backup/configs"
backup_file() {
    local source_file="$1"
    local backup_dir="${2:-./backups}"
    local timestamp=$(date +%Y%m%d_%H%M%S)

    # Check if source exists
    if [ ! -f "$source_file" ]
    then
        echo "Error: Source file not found: $source_file" >&2
        return 1
    fi

    # Create backup directory
    if ! mkdir -p "$backup_dir"
    then
        echo "Error: Cannot create backup directory: $backup_dir" >&2
        return 2
    fi

    # Create backup
    local filename=$(basename "$source_file")
    local backup_path="$backup_dir/${filename}_${timestamp}"
    cp "$source_file" "$backup_path"

    echo "Backup created: $backup_path"
    return 0
}
```

## Practical Examples

### Menu System with Functions

```bash
#!/bin/bash

show_menu() {
    echo "=== Main Menu ==="
    echo "1. Show date"
    echo "2. Show disk usage"
    echo "3. Show users"
    echo "4. Exit"
    echo "================="
}

show_date() {
    echo "Current date: $(date)"
}

show_disk() {
    echo "Disk usage:"
    df -h | head -5
}

show_users() {
    echo "Logged in users:"
    who
}

main() {
    while true
    do
        show_menu
        read -p "Select option: " choice

        case $choice in
            1) show_date ;;
            2) show_disk ;;
            3) show_users ;;
            4) echo "Goodbye!"; break ;;
            *) echo "Invalid option" ;;
        esac
        echo
    done
}

# Start the program
main
```

### Input Validation Functions

```bash
#!/bin/bash

# Validate email
is_valid_email() {
    local email="$1"
    local regex="^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$"
    [[ $email =~ $regex ]]
}

# Validate IP address
is_valid_ip() {
    local ip="$1"
    local regex="^([0-9]{1,3}\.){3}[0-9]{1,3}$"
    [[ $ip =~ $regex ]]
}

# Validate file exists and is readable
is_readable_file() {
    local file="$1"
    [ -f "$file" ] && [ -r "$file" ]
}

# Usage
if is_valid_email "user@example.com"
then
    echo "Valid email"
fi
```

## Best Practices

1. **Use local variables**: Prevent variable conflicts
2. **Return meaningful exit codes**: 0 for success, non-zero for errors
3. **Document functions**: Add comments explaining purpose and usage
4. **Keep functions focused**: One function, one purpose
5. **Use descriptive names**: Function names should describe what they do
6. **Validate inputs**: Check parameters before processing
7. **Handle errors gracefully**: Return error codes and messages

## Common Patterns

```bash
# Initialization function
init() {
    # Setup code here
    mkdir -p "$WORK_DIR"
    touch "$LOG_FILE"
}

# Cleanup function
cleanup() {
    # Cleanup code here
    rm -rf "$TEMP_DIR"
}

# Trap cleanup on exit
trap cleanup EXIT

# Main execution
main() {
    init
    # Your code here
}

main "$@"
```

## Common Pitfalls

- Forgetting `local` leads to unintended global variable mutations.
- Using `return` for data; `return` is for status codes (0–255). Use `echo`/command substitution to return values.
- Not forwarding arguments correctly; prefer `"$@"` over `$*`.
- Overwriting `IFS` globally; if changed, restore it after use.
- Capturing command output without checking its exit status; inspect `$?` or use `set -e` wisely.

## Exercises

1. Write a function `join_by SEP ...` that joins its arguments with a separator.
2. Write `require_cmd NAME` that checks for a command and exits with a helpful message if missing.
3. Create a library file with two functions and `source` it from a script; verify scoping via `local`.
4. Write a function that returns both a status and data: print data to stdout; use the exit code for status.

## Cross‑links

- [06 - Loops](./06-loops.md)
- [08 - Error Handling and Debugging](./08-error-handling-and-debugging.md)

## References

- GNU Bash Reference Manual: <https://www.gnu.org/software/bash/manual/bash.html>
