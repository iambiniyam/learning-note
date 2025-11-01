# Variables and Data Types

[Previous: 02 - Creating and Running Scripts](./02-creating-and-running-scripts.md) • [Next: 04 - User Input and Arguments](./04-user-input-and-arguments.md)

## What are Variables?

Variables store data that can be used and manipulated throughout your script. In bash, variables are untyped (no need to declare data type).

## Variable Naming Rules

**Valid:**

```bash
name="John"
_age=30
server1="192.168.1.1"
USER_NAME="admin"
```

**Invalid:**

```bash
1name="John"        # Cannot start with number
my-name="John"      # Hyphens not allowed
my name="John"      # Spaces not allowed
```

**Best Practices:**

- Use lowercase for local variables: `my_var`
- Use UPPERCASE for constants/environment variables: `MAX_RETRIES`
- Use descriptive names: `database_host` not `dbh`
- Use underscores for separation: `user_name` not `userName`

## Declaring Variables

```bash
#!/bin/bash

# Simple assignment (no spaces around =)
name="John Doe"
age=30
is_admin=true

# Using command output
current_date=$(date)
files_count=$(ls | wc -l)

# Using backticks (old style, avoid)
current_dir=`pwd`

# Empty variable
empty_var=""

# Null/unset variable
unset some_var
```

**Important:** No spaces around `=` sign!

```bash
# Correct
name="John"

# Wrong
name = "John"   # This tries to run 'name' as a command
```

## Accessing Variables

```bash
#!/bin/bash

name="Alice"

# Simple form
echo $name

# Preferred form (protects variable name)
echo ${name}

# When concatenating
echo "${name}_backup"  # Alice_backup
echo "$name_backup"    # (empty - looks for variable name_backup)
```

## Variable Scope

### Local Variables (default)

```bash
#!/bin/bash

# Available only in current script
my_var="local value"
```

### Environment Variables (export)

```bash
#!/bin/bash

# Available to child processes
export DATABASE_URL="postgresql://localhost/mydb"

# Or in one line
export API_KEY="abc123"

# Run child script that can access DATABASE_URL
./child_script.sh
```

### Local in Functions

```bash
#!/bin/bash

my_function() {
    local func_var="only in function"
    echo $func_var
}

my_function
echo $func_var  # Empty - not accessible outside function
```

## Special Variables

```bash
#!/bin/bash

echo "Script name: $0"
echo "First argument: $1"
echo "Second argument: $2"
echo "All arguments: $@"
echo "All arguments (as one string): $*"
echo "Number of arguments: $#"
echo "Exit status of last command: $?"
echo "Current process ID: $$"
echo "Last background process ID: $!"
echo "Current shell options: $-"
```

**Example usage:**

```bash
./script.sh hello world 123

# Inside script:
# $0 = ./script.sh
# $1 = hello
# $2 = world
# $3 = 123
# $@ = hello world 123 (as separate items)
# $* = hello world 123 (as single string)
# $# = 3
```

## Data Types (Bash doesn't have strict types)

### Strings

```bash
#!/bin/bash

# Single quotes (literal - no variable expansion)
name='John $USER'  # Literal: John $USER

# Double quotes (allows variable expansion)
name="John $USER"  # Expands to: John admin

# No quotes (word splitting occurs)
name=John          # Works but avoid for values with spaces

# Concatenation
first_name="John"
last_name="Doe"
full_name="${first_name} ${last_name}"

# Multi-line strings
message="Line 1
Line 2
Line 3"

# Or using cat
message=$(cat << EOF
Line 1
Line 2
Line 3
EOF
)
```

### Numbers (Integers)

```bash
#!/bin/bash

# Integer variables
count=42
age=30

# Arithmetic operations
result=$((5 + 3))
result=$((count * 2))
result=$((age - 5))
result=$((10 / 3))    # Integer division: 3
result=$((10 % 3))    # Modulo: 1

# Increment/Decrement
((count++))
((count--))
((count += 5))
((count -= 3))
((count *= 2))
((count /= 2))

# Alternative: let command
let result=5+3
let count++
```

### Arrays (Indexed)

```bash
#!/bin/bash

# Declare array
servers=("web1" "web2" "web3")

# Or assign individually
servers[0]="web1"
servers[1]="web2"
servers[2]="web3"

# Access elements
echo ${servers[0]}      # First element: web1
echo ${servers[2]}      # Third element: web3
echo ${servers[@]}      # All elements
echo ${servers[*]}      # All elements (different in quotes)
echo ${#servers[@]}     # Number of elements: 3

# Add element
servers+=("web4")

# Loop through array
for server in "${servers[@]}"; do
    echo "Server: $server"
done

# Get array indices
echo ${!servers[@]}     # 0 1 2 3

# Slice array
echo ${servers[@]:1:2}  # Elements from index 1, count 2: web2 web3
```

### Associative Arrays (Key-Value pairs) - Bash 4+

```bash
#!/bin/bash

# Declare associative array
declare -A config

# Assign values
config[host]="localhost"
config[port]="5432"
config[database]="mydb"

# Access values
echo ${config[host]}           # localhost
echo ${config[port]}           # 5432

# All keys
echo ${!config[@]}             # host port database

# All values
echo ${config[@]}              # localhost 5432 mydb

# Number of elements
echo ${#config[@]}             # 3

# Loop through associative array
for key in "${!config[@]}"; do
    echo "$key: ${config[$key]}"
done
```

### Boolean (Convention)

Bash doesn't have true boolean type. Use conventions:

```bash
#!/bin/bash

# Convention 1: true/false strings
is_admin="true"
is_active="false"

if [[ $is_admin == "true" ]]; then
    echo "User is admin"
fi

# Convention 2: 0/1 values
is_admin=1  # 1 = true
is_active=0 # 0 = false

if ((is_admin)); then
    echo "User is admin"
fi

# Convention 3: Using exit codes
if true; then
    echo "This runs"
fi

if false; then
    echo "This doesn't run"
fi
```

## Readonly Variables (Constants)

```bash
#!/bin/bash

# Method 1: readonly
readonly MAX_RETRIES=3
readonly API_ENDPOINT="https://api.example.com"

# Method 2: declare -r
declare -r MAX_CONNECTIONS=100

# Attempting to change will cause error
MAX_RETRIES=5  # Error: readonly variable
```

## String Operations

### Length

```bash
string="Hello World"
echo ${#string}  # 11
```

### Substring

```bash
string="Hello World"
echo ${string:0:5}    # Hello (start:length)
echo ${string:6}      # World (from position 6 to end)
echo ${string: -5}    # World (last 5 characters - note the space)
```

### String Replace

```bash
string="Hello World World"

# Replace first occurrence
echo ${string/World/Universe}  # Hello Universe World

# Replace all occurrences
echo ${string//World/Universe} # Hello Universe Universe

# Delete substring
echo ${string//World/}         # Hello

# Replace at beginning
echo ${string/#Hello/Hi}       # Hi World World

# Replace at end
echo ${string/%World/Universe} # Hello World Universe
```

### Case Conversion (Bash 4+)

```bash
string="Hello World"

# To uppercase
echo ${string^^}               # HELLO WORLD

# To lowercase
echo ${string,,}               # hello world

# First character uppercase
echo ${string^}                # Hello World

# First character lowercase
echo ${string,}                # hello World
```

### Trimming

```bash
string="  Hello World  "

# Remove leading/trailing whitespace (using parameter expansion)
trimmed="$(echo -e "${string}" | sed -e 's/^[[:space:]]*//' -e 's/[[:space:]]*$//')"

# Remove prefix pattern
filename="backup_file.txt"
echo ${filename#backup_}       # file.txt

# Remove suffix pattern
echo ${filename%.txt}          # backup_file

# Remove longest prefix match
path="/usr/local/bin/script"
echo ${path##*/}               # script (basename)

# Remove longest suffix match
echo ${path%%/*}               # (everything after first /)
```

## Default Values

```bash
#!/bin/bash

# Use default if unset
echo ${name:-"Guest"}          # Guest (if name is unset)

# Assign default if unset
echo ${name:="Guest"}          # Sets name to Guest if unset

# Use alternative if set
echo ${name:+"Custom"}         # Custom (if name is set)

# Error if unset
echo ${name:?"Name is required"} # Exits with error if name is unset
```

**DevOps Example:**

```bash
#!/bin/bash

# Configuration with defaults
ENVIRONMENT=${ENVIRONMENT:-"development"}
LOG_LEVEL=${LOG_LEVEL:-"info"}
MAX_RETRIES=${MAX_RETRIES:-3}

echo "Environment: $ENVIRONMENT"
echo "Log Level: $LOG_LEVEL"
echo "Max Retries: $MAX_RETRIES"
```

## Command Substitution

```bash
#!/bin/bash

# Modern syntax (preferred)
current_date=$(date +%Y-%m-%d)
file_count=$(ls -1 | wc -l)
docker_containers=$(docker ps -q)

# Old syntax (avoid)
current_date=`date +%Y-%m-%d`

# Nested substitution
outer=$(echo "inner=$(date)")
```

## Arithmetic Operations

```bash
#!/bin/bash

# Method 1: $(( ))
result=$((5 + 3))
result=$((10 * 2))
result=$((100 / 3))
result=$((10 % 3))
result=$((2 ** 8))     # Power: 256

# Method 2: expr (old style)
result=$(expr 5 + 3)

# Method 3: let
let result=5+3

# Method 4: bc for floating point
result=$(echo "scale=2; 10 / 3" | bc)  # 3.33
```

## Environment Variables in DevOps

```bash
#!/bin/bash

# Reading environment variables
echo "User: $USER"
echo "Home: $HOME"
echo "Path: $PATH"
echo "Shell: $SHELL"
echo "Hostname: $HOSTNAME"
echo "PWD: $PWD"

# Setting for current script and children
export DATABASE_URL="postgresql://localhost/mydb"
export API_KEY="secret_key_123"
export LOG_FILE="/var/log/app.log"

# Loading from .env file (common in DevOps)
if [[ -f .env ]]; then
    export $(cat .env | grep -v '^#' | xargs)
fi
```

**Example .env file:**

```ini
DATABASE_URL=postgresql://localhost/mydb
API_KEY=secret_key_123
LOG_LEVEL=debug
```

## Practical DevOps Examples

### Configuration Script

```bash
#!/bin/bash

# Constants
readonly APP_NAME="MyApp"
readonly VERSION="1.0.0"
readonly CONFIG_DIR="/etc/${APP_NAME}"

# Variables with defaults
ENVIRONMENT=${ENVIRONMENT:-"production"}
LOG_LEVEL=${LOG_LEVEL:-"info"}
MAX_WORKERS=${MAX_WORKERS:-4}

# Arrays
readonly REQUIRED_PACKAGES=("docker" "git" "curl" "jq")
declare -A server_ports
server_ports[web]=8080
server_ports[api]=3000
server_ports[db]=5432

echo "=== $APP_NAME v$VERSION ==="
echo "Environment: $ENVIRONMENT"
echo "Log Level: $LOG_LEVEL"
echo "Max Workers: $MAX_WORKERS"

for package in "${REQUIRED_PACKAGES[@]}"; do
    echo "Checking $package..."
done

for service in "${!server_ports[@]}"; do
    echo "$service runs on port ${server_ports[$service]}"
done
```

## Practice Exercise

Create a script that:

1. Defines variables for server name, IP, and port
2. Creates an array of 5 service names
3. Uses an associative array for service status
4. Calculates uptime in hours from seconds
5. Demonstrates string manipulation

**Solution:**

```bash
#!/bin/bash

set -euo pipefail

# Server configuration
readonly SERVER_NAME="prod-web-01"
readonly SERVER_IP="192.168.1.100"
readonly SERVER_PORT=8080

# Services array
services=("nginx" "postgresql" "redis" "app" "monitoring")

# Service status (associative array)
declare -A service_status
service_status[nginx]="running"
service_status[postgresql]="running"
service_status[redis]="stopped"
service_status[app]="running"
service_status[monitoring]="running"

# Calculate uptime
uptime_seconds=86400
uptime_hours=$((uptime_seconds / 3600))

echo "=== Server Information ==="
echo "Server: ${SERVER_NAME}"
echo "IP: ${SERVER_IP}"
echo "Port: ${SERVER_PORT}"
echo "Uptime: ${uptime_hours} hours"
echo

echo "=== Services Status ==="
for service in "${services[@]}"; do
    status=${service_status[$service]}
    # Uppercase first letter of status
    status_formatted="${status^}"
    echo "- ${service}: ${status_formatted}"
done

# String manipulation
server_type=${SERVER_NAME%-*}  # Extract prefix before last -
echo -e "\nServer Type: ${server_type}"
```

---

**Next:** Learn about user input and command-line arguments.

## Common Pitfalls

- Unquoted expansions cause word splitting and globbing. Prefer "${var}" over $var.
- Confusing strings and arrays; `${arr}` is not the same as "${arr[@]}".
- Relying on implicit integer context; use `declare -i` cautiously and prefer explicit arithmetic `(( ))`.
- Exporting variables unintentionally; `export` affects child processes only, not the parent.
- Assuming POSIX `/bin/sh` supports Bash arrays and `[[` tests; stick to `bash` for these features.

## Exercises

1. Create variables for first and last name, then print a full name safely quoted.
2. Use arithmetic expansion to convert minutes to hours and minutes (e.g., 135 → 2h 15m).
3. Create an indexed array of filenames and loop through them, printing their lengths.
4. Create an associative array mapping service→port; print each pair sorted by service name.
5. Load key=value pairs from a `.env`-style file safely (ignore comments) and print selected vars.

## Cross‑links

- [02 - Creating and Running Scripts](./02-creating-and-running-scripts.md)
- [04 - User Input and Arguments](./04-user-input-and-arguments.md)

## References

- GNU Bash Reference Manual: <https://www.gnu.org/software/bash/manual/bash.html>
