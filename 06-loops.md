# Loops in Bash

[Previous: 05 - Conditionals](./05-conditionals.md) • [Next: 07 - Functions](./07-functions.md)

Loops allow you to execute a block of commands repeatedly. Bash supports three types of loops: `for`, `while`, and `until`.

## For Loops

### Basic For Loop (List Iteration)

```bash
#!/bin/bash

# Iterate over a list of items
for item in apple banana cherry
do
    echo "Fruit: $item"
done
```

### For Loop with Range

```bash
#!/bin/bash

# Loop through a range of numbers
for i in {1..5}
do
    echo "Number: $i"
done

# With step size
for i in {0..10..2}
do
    echo "Even number: $i"
done
```

### C-style For Loop

```bash
#!/bin/bash

# Traditional C-style for loop
for ((i=1; i<=5; i++))
do
    echo "Iteration: $i"
done
```

### Looping Through Files

```bash
#!/bin/bash

# Loop through files in current directory
for file in *.txt
do
    echo "Processing: $file"
done

# Loop through files recursively
for file in **/*.sh
do
    echo "Shell script: $file"
done
```

### Looping Through Command Output

```bash
#!/bin/bash

# Loop through lines of command output
for user in $(cut -d: -f1 /etc/passwd)
do
    echo "User: $user"
done

# Better approach using while read (see below)
```

### Looping Through Arrays

```bash
#!/bin/bash

# Define an array
colors=("red" "green" "blue" "yellow")

# Loop through array elements
for color in "${colors[@]}"
do
    echo "Color: $color"
done

# Loop with index
for i in "${!colors[@]}"
do
    echo "Index $i: ${colors[$i]}"
done
```

## While Loops

### Basic While Loop

```bash
#!/bin/bash

# While loop with counter
counter=1
while [ $counter -le 5 ]
do
    echo "Count: $counter"
    ((counter++))
done
```

### Reading Files Line by Line

```bash
#!/bin/bash

# Best way to read a file line by line
while IFS= read -r line
do
    echo "Line: $line"
done < input.txt

# Reading command output
ls -l | while read -r line
do
    echo "Entry: $line"
done
```

### Infinite Loop

```bash
#!/bin/bash

# Infinite loop (use with caution)
while true
do
    echo "Press Ctrl+C to stop"
    sleep 1
done

# Alternative syntax
while :
do
    echo "Running forever..."
    sleep 1
done
```

### While Loop with Multiple Conditions

```bash
#!/bin/bash

counter=1
max=10

while [ $counter -le $max ] && [ $counter -ne 7 ]
do
    echo "Counter: $counter"
    ((counter++))
done
```

## Until Loops

The `until` loop executes until a condition becomes true (opposite of while).

### Basic Until Loop

```bash
#!/bin/bash

# Until loop
counter=1
until [ $counter -gt 5 ]
do
    echo "Count: $counter"
    ((counter++))
done
```

### Waiting for a Condition

```bash
#!/bin/bash

# Wait until a file exists
until [ -f "/tmp/ready.txt" ]
do
    echo "Waiting for file..."
    sleep 2
done
echo "File found!"
```

## Loop Control Statements

### Break Statement

Exit the loop immediately.

```bash
#!/bin/bash

# Break example
for i in {1..10}
do
    if [ $i -eq 6 ]
    then
        break  # Exit loop when i equals 6
    fi
    echo "Number: $i"
done
```

### Continue Statement

Skip the rest of the current iteration and continue with the next.

```bash
#!/bin/bash

# Continue example
for i in {1..10}
do
    if [ $i -eq 5 ]
    then
        continue  # Skip when i equals 5
    fi
    echo "Number: $i"
done
```

### Nested Loop Control

```bash
#!/bin/bash

# Breaking out of nested loops
for i in {1..3}
do
    for j in {1..3}
    do
        if [ $j -eq 2 ]
        then
            break  # Breaks inner loop only
        fi
        echo "i=$i, j=$j"
    done
done
```

## Select Loop

The `select` loop creates a simple menu system.

```bash
#!/bin/bash

# Create a menu
PS3="Select an option: "
options=("Option 1" "Option 2" "Option 3" "Quit")

select opt in "${options[@]}"
do
    case $opt in
        "Option 1")
            echo "You selected Option 1"
            ;;
        "Option 2")
            echo "You selected Option 2"
            ;;
        "Option 3")
            echo "You selected Option 3"
            ;;
        "Quit")
            echo "Goodbye!"
            break
            ;;
        *)
            echo "Invalid option"
            ;;
    esac
done
```

## Practical Examples

### Processing Multiple Files

```bash
#!/bin/bash

# Batch rename files
for file in *.txt
do
    newname="${file%.txt}_backup.txt"
    cp "$file" "$newname"
    echo "Backed up: $file -> $newname"
done
```

### Monitoring System Resources

```bash
#!/bin/bash

# Monitor CPU usage
count=0
while [ $count -lt 10 ]
do
    echo "=== Check $((count + 1)) ==="
    top -bn1 | head -5
    sleep 5
    ((count++))
done
```

### User Input Loop

```bash
#!/bin/bash

# Keep asking until valid input
while true
do
    read -p "Enter a number between 1 and 10: " num

    if [[ $num =~ ^[0-9]+$ ]] && [ $num -ge 1 ] && [ $num -le 10 ]
    then
        echo "Valid input: $num"
        break
    else
        echo "Invalid input. Try again."
    fi
done
```

## Best Practices

1. **Use `while read` for file processing**: More reliable than for loops

   ```bash
   while IFS= read -r line; do
       echo "$line"
   done < file.txt
   ```

2. **Quote variables**: Prevent word splitting

   ```bash
   for file in *.txt; do
       echo "$file"  # Always quote
   done
   ```

3. **Use `[[` for conditions**: More robust than `[`

   ```bash
   while [[ $counter -lt 10 ]]; do
       echo "$counter"
       ((counter++))
   done
   ```

4. **Avoid infinite loops**: Always have an exit condition or break statement

5. **Use meaningful variable names**: Make code readable

## Common Pitfalls

```bash
# WRONG: Word splitting issues
for file in $(ls *.txt)  # Breaks on spaces

# RIGHT: Use glob patterns
for file in *.txt

# WRONG: Reading file splits on whitespace
for line in $(cat file.txt)

# RIGHT: Use while read
while read -r line; do
    echo "$line"
done < file.txt
```

## Exercises

1. Loop over all `*.log` files and print the line count for each using `wc -l`.
2. Read `users.txt` line-by-line with `while read -r` and number each line.
3. Use `break` and `continue` to skip blank lines and stop at a sentinel value (e.g., `END`).
4. Write a loop with a timeout using a counter and `sleep` to retry a command up to N times.

## Cross‑links

- [05 - Conditionals](./05-conditionals.md)
- [07 - Functions](./07-functions.md)

## References

- GNU Bash Reference Manual: <https://www.gnu.org/software/bash/manual/bash.html>
