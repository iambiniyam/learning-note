# Advanced Bash Topics

[Previous: 12 - Networking and APIs](./12-networking-and-apis.md)

This guide covers more advanced features of bash scripting, including signals, job control, arrays, and other powerful constructs.

> Note (Windows/macOS): Some advanced features (e.g., certain `trap` behaviors, `lsof` in related workflows, or GNU-specific options) can vary by platform and shell version. Use WSL for a GNU/Linux environment on Windows, and ensure your Bash version is recent (4.4+ recommended, 5.x ideal).

## Signals

Signals are software interrupts sent to a program to indicate that an important event has occurred.

### Common Signals

| Signal Name | Number | Description                              |
| ----------- | ------ | ---------------------------------------- |
| `SIGHUP`    | 1      | Hangup (often used to reload configs)    |
| `SIGINT`    | 2      | Interrupt (from Ctrl+C)                  |
| `SIGQUIT`   | 3      | Quit (from Ctrl+\)                       |
| `SIGKILL`   | 9      | Kill (cannot be caught or ignored)       |
| `SIGTERM`   | 15     | Termination (graceful shutdown)          |
| `SIGSTOP`   | 19     | Stop (pause execution, cannot be caught) |
| `SIGTSTP`   | 20     | Terminal Stop (from Ctrl+Z)              |

### `trap` - Catching Signals

The `trap` command allows you to execute code when a signal is received.

```bash
#!/bin/bash

# Function to run on exit
cleanup() {
    echo "Cleaning up temporary files..."
    rm -f /tmp/myscript.*
}

# Trap EXIT signal (runs on any script exit)
trap cleanup EXIT

# Trap specific signals
trap 'echo "Caught Ctrl+C, exiting..."; exit' INT
trap 'echo "Caught TERM signal, exiting..."' TERM

echo "Script running, PID: $$"
echo "Press Ctrl+C to test."
sleep 60
```

### Ignoring Signals

```bash
# Ignore SIGINT
trap '' INT

echo "Ctrl+C is disabled. Use 'kill $$' to stop."
sleep 60
```

### Resetting Traps

```bash
# Reset trap to default behavior
trap - INT
```

## Job Control

Job control refers to the ability to stop, resume, and move processes between the foreground and background.

```bash
# Start a job in the background
sleep 100 &
# [1] 12345

# List jobs
jobs
# [1]+  Running                 sleep 100 &

# Bring job to foreground
fg %1

# Stop a foreground job (Ctrl+Z)
# [1]+  Stopped                 sleep 100

# Send a stopped job to the background
bg %1
# [1]+  sleep 100 &

# Disown a job to prevent it from being killed on shell exit
disown %1
```

## Arrays

Bash supports one-dimensional indexed and associative arrays.

### Indexed Arrays

```bash
# Declare an indexed array
fruits=("Apple" "Banana" "Cherry")

# Access elements
echo ${fruits[0]}  # Apple
echo ${fruits[1]}  # Banana

# Access all elements
echo ${fruits[@]}  # Apple Banana Cherry

# Get array length
echo ${#fruits[@]} # 3

# Get indices
echo ${!fruits[@]} # 0 1 2

# Add an element
fruits+=("Orange")

# Modify an element
fruits[1]="Blueberry"

# Loop through an array
for fruit in "${fruits[@]}"; do
    echo "I like $fruit"
done
```

### Associative Arrays (Bash 4.0+)

```bash
# Declare an associative array
declare -A user
user["name"]="Alice"
user["email"]="alice@example.com"
user["id"]="123"

# Access elements
echo ${user["name"]}  # Alice

# Access all values
echo ${user[@]}

# Access all keys
echo ${!user[@]}

# Add an element
user["city"]="New York"

# Loop through an associative array
for key in "${!user[@]}"; do
    echo "$key: ${user[$key]}"
done
```

## Advanced Variable Expansion

```bash
# Default value
echo ${VAR:-"default"}  # Use "default" if VAR is unset or null

# Assign default value
echo ${VAR:="default"}  # Assign "default" to VAR if unset or null

# Use alternative value
echo ${VAR:+"alternative"} # Use "alternative" if VAR is set

# Error on unset
echo ${VAR:?"VAR is not set"} # Exit with error if VAR is unset or null

# Substring expansion
string="hello world"
echo ${string:6:5}  # world (offset 6, length 5)
echo ${string:6}    # world (offset 6 to end)

# Pattern matching and replacement
file="image.jpg"
echo ${file%.jpg}.png   # image.png (remove shortest suffix)
echo ${file%%.*}.png  # image.png (remove longest suffix)

path="/home/user/file.txt"
echo ${path#*/}       # home/user/file.txt (remove shortest prefix)
echo ${path##*/}      # file.txt (remove longest prefix)

# Search and replace
echo ${path/user/guest} # /home/guest/file.txt (first match)
echo ${path//o/O}      # /hOme/user/file.txt (all matches)
```

## Process Substitution

Process substitution `<()` and `>()` allows you to treat the output of a command as a temporary file.

```bash
# Compare the output of two commands
diff <(ls -l dir1) <(ls -l dir2)

# Pass output to a command that expects a file
grep "pattern" <(curl -s https://example.com)

# Tee output to multiple processes
ls -l | tee >(grep ".txt") >(grep ".md") > /dev/null
```

## Subshells

Commands grouped in `()` run in a separate subshell environment.

```bash
# Variables in subshells are local
(
    subshell_var="I am local"
    echo "In subshell: $subshell_var"
)
echo "Outside subshell: $subshell_var" # This will be empty

# Change directory in a subshell without affecting the parent
pwd
(cd /tmp && pwd)
pwd # Still in the original directory
```

## Command Substitution

```bash
# Old style (backticks)
now=`date`

# New style (preferred)
now=$(date)

# Nesting
echo "Files in home: $(ls $(echo ~))"
```

## Here Documents and Here Strings

### Here Document

```bash
# Pass multi-line string to a command
cat << EOL
This is a multi-line
string that will be
passed to cat.
EOL

# Use with variables
cat << EOL
User: $USER
Home: $HOME
EOL

# Suppress tab expansion
cat <<- EOL
    This line is indented with tabs.
    The tabs will be ignored.
EOL
```

### Here String

```bash
# Pass a single string to a command's stdin
grep "pattern" <<< "This is a string with a pattern"
```

## Indirect Expansion

```bash
# Access a variable whose name is stored in another variable
var_name="my_var"
my_var="hello"

echo ${!var_name}  # hello
```

## Shell Builtins

### `eval`

`eval` concatenates its arguments into a string, then executes that string as a command. Use with extreme caution as it can execute arbitrary code.

```bash
# Dangerous example
command="ls"
options="-l"
eval "$command $options"

# Safer alternatives are usually available
```

### `source` / `.`

`source` executes a script in the current shell context, allowing it to modify the current environment.

```bash
# file: config.sh
export API_KEY="12345"

# file: main.sh
source config.sh
echo "API Key is: $API_KEY"
```

### `exec`

`exec` replaces the current shell process with the specified command.

```bash
# The shell will be replaced by the 'ls' process
exec ls -l

# Useful for "locking down" a script
exec > script.log 2>&1
echo "This will all go to the log file."
```

## Advanced Scripting Techniques

### Creating a PID File

```bash
#!/bin/bash
PID_FILE="/var/run/myscript.pid"

# Check if script is already running
if [ -f "$PID_FILE" ] && ps -p $(cat "$PID_FILE") > /dev/null; then
    echo "Script is already running."
    exit 1
fi

# Create PID file
echo $$ > "$PID_FILE"

# Ensure PID file is removed on exit
trap 'rm -f "$PID_FILE"' EXIT

# Main script logic
echo "Script started with PID $$"
sleep 30
```

## Common Pitfalls

- Misusing `eval`, which executes arbitrary strings—avoid unless absolutely necessary and prefer safer constructs.
- Forgetting to quote array expansions; use "${array[@]}" to preserve items with spaces.
- Assuming traps run in subshells the same as in the parent—`trap` is per-shell; subshells don’t inherit traps unless set inside them.
- Here-documents accidentally performing variable expansion; use single-quoted delimiters (e.g., << 'EOF') when you want literal content.
- Relying on process substitution `<()` on systems/shells that don’t support it; verify availability or provide fallbacks.

## Exercises

1. Write a script that sets a trap for `INT` and `EXIT`, and demonstrates cleanup of a temporary directory.
2. Implement both indexed and associative arrays; iterate and print keys/values, then sort keys and reprint.
3. Use parameter expansion to sanitize filenames (replace spaces with underscores, change extensions).
4. Compare two directories using process substitution with `diff <(ls -1 dirA) <(ls -1 dirB)` and interpret the result.
5. Create a PID file pattern for a script and ensure it correctly refuses to start if another instance is running.

### Creating Temporary Files

```bash
# Create a secure temporary file
temp_file=$(mktemp)
echo "Data" > "$temp_file"

# Create a temporary directory
temp_dir=$(mktemp -d)

# Cleanup
trap 'rm -rf "$temp_file" "$temp_dir"' EXIT
```

### Reading from a Configuration File

```bash
# config.ini
# HOST=example.com
# USER=admin

# script.sh
CONFIG_FILE="config.ini"
if [ -f "$CONFIG_FILE" ]; then
    # Source the config file, filtering out comments
    source <(grep -v '^#' "$CONFIG_FILE")
fi

echo "Host is: ${HOST:-localhost}"
echo "User is: ${USER:-default}"
```

## References

- GNU Bash Reference Manual: <https://www.gnu.org/software/bash/manual/bash.html>

## Cross‑links

- [12 - Networking and APIs](./12-networking-and-apis.md)
