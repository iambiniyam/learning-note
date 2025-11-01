# Process Management in Bash

[Previous: 10 - Files and Directories](./10-files-and-directories.md) • [Next: 12 - Networking and APIs](./12-networking-and-apis.md)

Process management is a core task for system administrators and developers. Bash provides several tools to monitor, control, and automate processes.

> Note (Windows): Many commands in this chapter (e.g., `ps`, `nice`, `renice`, `lsof`) are Linux/Unix utilities. Use WSL (Windows Subsystem for Linux) or Git Bash to follow along, or look for PowerShell equivalents.

## Viewing Processes

### `ps` - Report a Snapshot of Current Processes

```bash
# Show processes for the current user and terminal
ps

# Show all processes in BSD format
ps aux

# Show all processes in System V format
ps -ef

# Show processes in a tree (hierarchy)
ps axjf

# Custom format
ps -o pid,ppid,user,%cpu,%mem,cmd

# Filter by user
ps -u username

# Filter by process name
ps -C bash
```

### `top` - Display Dynamic Real-Time View of Processes

```bash
# Start top
top

# Useful commands in top:
# k - Kill a process
# r - Renice a process (change priority)
# u - Filter by user
# M - Sort by memory usage
# P - Sort by CPU usage
# q - Quit
```

### `htop` - Interactive Process Viewer (Improved `top`)

```bash
# Start htop (often needs to be installed)
htop

# Features:
# - Colorized display
# - Mouse support
# - Horizontal and vertical scrolling
# - Easier process killing and renicing
```

### `pgrep` - Look Up Processes Based on Name and Other Attributes

```bash
# Find PID of a process
pgrep bash

# Find PID and show command line
pgrep -a bash

# Find processes owned by a user
pgrep -u username

# Find processes NOT matching
pgrep -v root
```

## Controlling Processes

### `kill` - Send a Signal to a Process

```bash
# Terminate a process gracefully (SIGTERM, 15)
kill 12345  # 12345 is the PID

# Force kill a process (SIGKILL, 9)
kill -9 12345
kill -KILL 12345

# Send other signals
kill -HUP 12345  # Reload configuration (SIGHUP, 1)
kill -STOP 12345 # Pause a process (SIGSTOP, 19)
kill -CONT 12345 # Continue a paused process (SIGCONT, 18)

# List all signals
kill -l
```

### `pkill` - Send Signal to Processes Based on Name

```bash
# Kill a process by name
pkill firefox

# Force kill
pkill -9 firefox

# Kill processes for a specific user
pkill -u username
```

### `killall` - Kill Processes by Name

```bash
# Kill all instances of a command
killall firefox

# Interactive mode
killall -i firefox

# Kill older processes
killall -o 10m firefox  # Older than 10 minutes
```

## Job Control

Job control allows you to manage multiple processes within a single shell session.

### Running Processes in the Background

```bash
# Start a process in the background
sleep 300 &

# The shell returns a job ID and PID
# [1] 12345
```

### `jobs` - List Active Jobs

```bash
# List all background jobs
jobs

# List jobs with their PIDs
jobs -l
```

### `fg` - Bring a Job to the Foreground

```bash
# Bring the most recent job to the foreground
fg

# Bring a specific job to the foreground
fg %1  # %1 is the job ID from jobs command
```

### `bg` - Send a Job to the Background

```bash
# If a foreground process is stopped (Ctrl+Z), send it to the background
bg

# Send a specific stopped job to the background
bg %2
```

### `Ctrl+Z` - Suspend a Foreground Process

```bash
# While a command is running...
# Press Ctrl+Z
# [1]+  Stopped                 sleep 300
```

### `disown` - Remove a Job from the Job Table

```bash
# Start a job
sleep 300 &

# Disown the job so it's not killed when the shell closes
disown %1

# Disown and keep it running
disown -h %1
```

### `nohup` - Run a Command Immune to Hangups

```bash
# Run a command that continues after you log out
nohup ./myscript.sh &

# Output is redirected to nohup.out by default
nohup ./myscript.sh > output.log 2>&1 &
```

## Process Priority

### `nice` - Run a Command with Modified Priority

```bash
# Run a command with lower priority (nicer)
# Nice values range from -20 (highest priority) to 19 (lowest)
nice -n 10 ./my_backup_script.sh

# Run with higher priority (requires root)
sudo nice -n -5 ./important_task.sh
```

### `renice` - Alter Priority of Running Processes

```bash
# Change priority of a running process
renice 10 -p 12345  # Set nice value to 10 for PID 12345

# Change priority for a user's processes
renice 5 -u username

# Change priority for a process group
renice -2 -g 54321
```

## Waiting for Processes

### `wait` - Wait for a Process to Complete

```bash
#!/bin/bash

echo "Starting background jobs..."
sleep 5 &
job1_pid=$!
sleep 3 &
job2_pid=$!

echo "Waiting for jobs to finish..."
wait $job1_pid
echo "Job 1 finished."

wait $job2_pid
echo "Job 2 finished."

echo "All done."
```

## System Information

### `uptime` - Show How Long System Has Been Running

```bash
# Display uptime, number of users, and load average
uptime
# 10:30:00 up 5 days,  2:10,  3 users,  load average: 0.05, 0.15, 0.10
```

### `free` - Display Amount of Free and Used Memory

```bash
# Show memory usage
free

# Human-readable format
free -h
```

### `lsof` - List Open Files

```bash
# List all open files (can be very long)
lsof

# List files opened by a specific process
lsof -p 12345

# List processes that have a specific file open
lsof /var/log/syslog

# List processes using a specific port
lsof -i :80
```

## Practical Examples

### Simple Parallel Processing

```bash
#!/bin/bash

# Process multiple files in parallel
for file in *.zip; do
    unzip "$file" &
done

# Wait for all background jobs to finish
wait
echo "All files unzipped."
```

### Watchdog Script

```bash
#!/bin/bash

# Ensure a process is always running
while true; do
    if ! pgrep -x "my_daemon" > /dev/null; then
        echo "Daemon not running. Starting..."
        /usr/local/bin/my_daemon &
    fi
    sleep 10
done
```

### Resource-Intensive Task Management

```bash
#!/bin/bash

# Run a CPU-intensive task with low priority
echo "Starting video encoding with low priority..."
nice -n 19 ffmpeg -i input.mp4 output.mkv
```

### Graceful Shutdown of Services

```bash
#!/bin/bash

PID_FILE="/var/run/myservice.pid"

if [ -f "$PID_FILE" ]; then
    PID=$(cat "$PID_FILE")
    echo "Sending SIGTERM to process $PID..."
    kill "$PID"

    # Wait for process to terminate
    for i in {1..10}; do
        if ! ps -p "$PID" > /dev/null; then
            echo "Process terminated."
            rm "$PID_FILE"
            exit 0
        fi
        sleep 1
    done

    echo "Process did not terminate gracefully. Sending SIGKILL..."
    kill -9 "$PID"
    rm "$PID_FILE"
else
    echo "Service not running."
fi
```

## Best Practices

1. **Prefer `pkill` over `killall`**: `pkill` is more flexible and standard.
2. **Use `SIGTERM` before `SIGKILL`**: Allow processes to clean up gracefully.
3. **Use `nohup` or `disown` for long-running background tasks**: Prevent them from being killed on logout.
4. **Use `wait` for parallel scripts**: Ensure all background tasks complete before proceeding.
5. **Check for process existence before starting**: Avoid running duplicate daemons.
6. **Use PID files for daemons**: A standard way to manage service PIDs.
7. **Be careful with `kill -9`**: It can lead to data corruption.

## Common Pitfalls

- Using `kill -9` (SIGKILL) as the first attempt instead of trying `SIGTERM` and a grace period.
- Relying on partial names with `pkill`/`pgrep` and matching unintended processes; prefer `-x` for exact matches.
- Forgetting `nohup`/`disown`/a multiplexer (screen/tmux) for long-running jobs over SSH, causing them to terminate on disconnect.
- Assuming background jobs inherit the same environment; exported variables are copied at spawn time, later changes don't propagate.
- Parsing `ps` output with fragile text matching; prefer stable `ps -o` formats or `pgrep -f` where suitable.

## Exercises

1. Start two background `sleep` jobs, list them with `jobs -l`, bring one to the foreground with `fg`, then stop it with Ctrl+Z and resume it in the background with `bg`.
2. Launch a command with `nohup` and verify it survives logging out (or closing the terminal); inspect `nohup.out`.
3. Use `pgrep -x` to find the PID of a known process and send it `SIGHUP` with `kill -HUP` (choose a safe test process that handles HUP).
4. Start a CPU-intensive command and lower its priority with `nice`; then adjust it live with `renice`.
5. Write a small watchdog script that restarts a short-lived process if it exits unexpectedly; add a PID file to avoid multiple instances.

## Cross‑links

- [10 - Files and Directories](./10-files-and-directories.md)
- [12 - Networking and APIs](./12-networking-and-apis.md)

## References

- GNU Bash Reference Manual: <https://www.gnu.org/software/bash/manual/bash.html>
