# Working with Files and Directories in Bash

[Previous: 09 - Text Processing](./09-text-processing.md) • [Next: 11 - Process Management](./11-process-management.md)

Bash provides a rich set of commands for managing files and directories. This guide covers essential commands for navigation, creation, deletion, and manipulation.

> Note (Windows): To run these commands on Windows, use WSL (Windows Subsystem for Linux) or Git Bash. Native Windows CMD/PowerShell have different commands and semantics.

## Navigation

### `pwd` - Print Working Directory

```bash
# Display the current directory path
pwd
# Output: /home/user/documents
```

### `cd` - Change Directory

```bash
# Change to a specific directory
cd /var/log

# Go to home directory
cd
cd ~

# Go to previous directory
cd -

# Go up one level
cd ..

# Go up two levels
cd ../..
```

### `ls` - List Directory Contents

```bash
# List files in current directory
ls

# List all files (including hidden)
ls -a

# Long format (permissions, owner, size, date)
ls -l

# Human-readable sizes (KB, MB, GB)
ls -lh

# Sort by modification time (newest first)
ls -lt

# Reverse sort order
ls -lr

# Recursive listing
ls -R

# List only directories
ls -d */
```

## File and Directory Creation

### `touch` - Create Empty Files

```bash
# Create a new empty file
touch newfile.txt

# Create multiple files
touch file1.txt file2.txt file3.txt

# Update modification time of existing file
touch existing_file.txt
```

### `mkdir` - Create Directories

```bash
# Create a new directory
mkdir new_directory

# Create multiple directories
mkdir dir1 dir2 dir3

# Create nested directories (parent directories)
mkdir -p project/src/components
```

## File and Directory Manipulation

### `cp` - Copy Files and Directories

```bash
# Copy a file
cp source.txt destination.txt

# Copy file to a directory
cp source.txt /path/to/dir/

# Copy multiple files to a directory
cp file1.txt file2.txt /path/to/dir/

# Copy a directory recursively
cp -r source_dir/ destination_dir/

# Preserve file attributes (permissions, timestamps)
cp -p source.txt destination.txt

# Verbose mode (show what is being copied)
cp -v source.txt destination.txt

# Interactive mode (prompt before overwriting)
cp -i source.txt destination.txt
```

### `mv` - Move or Rename Files and Directories

```bash
# Rename a file
mv old_name.txt new_name.txt

# Move a file to a directory
mv file.txt /path/to/dir/

# Move multiple files
mv file1.txt file2.txt /path/to/dir/

# Move and rename
mv file.txt /path/to/dir/new_name.txt

# Rename a directory
mv old_dir new_dir

# Move a directory
mv my_dir /path/to/other_place/
```

### `rm` - Remove Files and Directories

```bash
# Remove a file
rm file.txt

# Remove multiple files
rm file1.txt file2.txt

# Force removal (no prompt)
rm -f file.txt

# Interactive mode (prompt for each file)
rm -i file.txt

# Remove an empty directory
rmdir empty_dir

# Remove a directory and its contents (use with caution!)
rm -r directory/
rm -rf directory/  # Force recursive removal
```

## File Content Viewing

### `cat` - Concatenate and Display Files

```bash
# Display file content
cat file.txt

# Display multiple files
cat file1.txt file2.txt

# Display with line numbers
cat -n file.txt

# Create a file with cat
cat > newfile.txt << EOL
Line 1
Line 2
EOL
```

### `less` - View File Content Interactively

```bash
# View a file with less
less file.txt

# Useful commands in less:
# /pattern  - Search for pattern
# n         - Next match
# N         - Previous match
# g         - Go to start of file
# G         - Go to end of file
# q         - Quit
```

### `more` - View File Content Page by Page

```bash
# View a file with more
more file.txt
```

### `head` - View Beginning of a File

```bash
# Show first 10 lines (default)
head file.txt

# Show first 5 lines
head -n 5 file.txt

# Show first 100 bytes
head -c 100 file.txt
```

### `tail` - View End of a File

```bash
# Show last 10 lines (default)
tail file.txt

# Show last 5 lines
tail -n 5 file.txt

# Follow a file (watch for new lines)
tail -f /var/log/syslog
```

## File and Directory Properties

### `stat` - Display File or Filesystem Status

```bash
# Show detailed information about a file
stat file.txt

# Show information about a directory
stat /path/to/dir

# Custom format
stat -c "%A %U %G %s %n" file.txt
# %A - Permissions
# %U - Owner name
# %G - Group name
# %s - Size in bytes
# %n - File name
```

### `file` - Determine File Type

```bash
# Identify file type
file script.sh      # script.sh: Bourne-Again shell script, ASCII text executable
file image.jpg      # image.jpg: JPEG image data
file archive.zip    # archive.zip: Zip archive data
```

### `du` - Estimate File and Directory Space Usage

```bash
# Show disk usage for current directory
du

# Human-readable format
du -h

# Summarize usage for a directory
du -sh /path/to/dir

# Show usage for all files and directories
du -ah
```

### `df` - Report Filesystem Disk Space Usage

```bash
# Show disk space for all mounted filesystems
df

# Human-readable format
df -h

# Show specific filesystem type
df -t ext4
```

## Finding Files and Directories

### `find` - Search for Files and Directories

```bash
# Find by name
find . -name "file.txt"

# Case-insensitive find
find . -iname "file.txt"

# Find by type
find . -type f  # Files
find . -type d  # Directories
find . -type l  # Symbolic links

# Find by modification time
find . -mtime -7  # Modified in last 7 days
find . -mtime +7  # Modified more than 7 days ago
find . -mmin -60  # Modified in last 60 minutes

# Find by size
find . -size +10M  # Larger than 10MB
find . -size -1k   # Smaller than 1KB
find . -size 5G    # Exactly 5GB

# Find by permissions
find . -perm 644

# Find empty files or directories
find . -empty

# Execute command on found files
find . -name "*.log" -exec rm {} \;
find . -name "*.txt" -exec grep "pattern" {} +

# Combine criteria
find /var/log -name "*.log" -mtime +30 -delete
```

### `locate` - Find Files by Name (Faster than find)

```bash
# Update the locate database
sudo updatedb

# Find a file
locate file.txt

# Case-insensitive search
locate -i pattern
```

### `which` - Locate a Command

```bash
# Find the path to an executable
which ls      # /bin/ls
which python  # /usr/bin/python
```

## File Permissions

### `chmod` - Change File Permissions

```bash
# Symbolic notation
chmod u+x script.sh      # Add execute for user
chmod g-w file.txt       # Remove write for group
chmod o=r file.txt       # Set read-only for others
chmod a+r file.txt       # Add read for all

# Octal notation
chmod 755 script.sh      # rwxr-xr-x
chmod 644 file.txt       # rw-r--r--
chmod 600 private.key    # rw-------

# Recursive permissions
chmod -R 644 public_html/
```

### `chown` - Change File Owner and Group

```bash
# Change owner
chown newuser file.txt

# Change group
chgrp newgroup file.txt

# Change owner and group
chown newuser:newgroup file.txt

# Recursive change
chown -R user:group /path/to/dir
```

### `umask` - Set Default File Permissions

```bash
# Show current umask
umask  # 0022

# Set umask
umask 0027

# Permissions are calculated as:
# 666 - umask (for files)
# 777 - umask (for directories)
```

## Links

### `ln` - Create Links

```bash
# Create a hard link
ln source.txt hard_link.txt

# Create a symbolic (soft) link
ln -s /path/to/source.txt symlink.txt

# Overwrite existing link
ln -sf /new/target link_name
```

## Practical Examples

### Batch Renaming

```bash
# Rename all .txt files to .md
for file in *.txt; do
    mv -- "$file" "${file%.txt}.md"
done
```

### Backup Script

```bash
#!/bin/bash
BACKUP_DIR="/backups/$(date +%Y-%m-%d)"
SOURCE_DIR="/var/www"
mkdir -p "$BACKUP_DIR"
cp -r "$SOURCE_DIR" "$BACKUP_DIR"
echo "Backup of $SOURCE_DIR created in $BACKUP_DIR"
```

### Find and Delete Old Log Files

```bash
# Find log files older than 30 days and delete them
find /var/log -name "*.log" -type f -mtime +30 -delete
```

### Organize Files by Extension

```bash
# Move files into directories based on their extension
for file in *.*; do
    ext="${file##*.}"
    mkdir -p "$ext"
    mv "$file" "$ext/"
done
```

## Best Practices

1. **Quote variables**: `rm "$filename"` to handle spaces.
2. **Use `-p` with `mkdir`**: Avoids errors if directory exists.
3. **Use `rm -rf` with extreme caution**: Double-check the path.
4. **Prefer `find ... -exec` over `xargs`**: Safer for complex filenames.
5. **Use `less` for large files**: Avoids loading entire file into memory.
6. **Check exit codes**: For critical operations like `cp` and `mv`.
7. **Use absolute paths in scripts**: For reliability.

## Common Pitfalls

- Not quoting paths with spaces or special characters; always use quotes.
- Using `rm -rf` with globs that may expand unexpectedly; prefer explicit paths and dry runs.
- Relying on relative paths in scripts; change directories safely or use absolute paths.
- `find` piping to `xargs` without `-0`; prefer `-print0 | xargs -0` or `-exec`.
- Assuming `cp`/`mv` won’t overwrite; use `-n` or backups as needed.

## Exercises

1. Create a `backup/` directory, copy all `.conf` files into it preserving structure with `rsync`.
2. Find files larger than 100MB under `~/Downloads` and print their sizes (human-readable).
3. Bulk-rename `*.TXT` to `*.txt` safely using a loop and parameter expansion.
4. Move files into subfolders based on extension (e.g., `.jpg` → `images/`).

## Cross‑links

- [09 - Text Processing](./09-text-processing.md)
- [11 - Process Management](./11-process-management.md)

## References

- GNU Bash Reference Manual: <https://www.gnu.org/software/bash/manual/bash.html>
