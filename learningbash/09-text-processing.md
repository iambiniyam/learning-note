# Text Processing in Bash

[Previous: 08 - Error Handling and Debugging](./08-error-handling-and-debugging.md) • [Next: 10 - Files and Directories](./10-files-and-directories.md)

Text processing is one of bash's strongest capabilities. This guide covers grep, sed, awk, and other text manipulation tools.

## grep - Pattern Searching

`grep` searches for patterns in text and files.

### Basic grep Usage

```bash
# Search for pattern in file
grep "pattern" file.txt

# Case-insensitive search
grep -i "pattern" file.txt

# Show line numbers
grep -n "pattern" file.txt

# Invert match (show non-matching lines)
grep -v "pattern" file.txt

# Count matches
grep -c "pattern" file.txt
```

### Extended grep Options

```bash
# Recursive search in directories
grep -r "pattern" /path/to/dir

# Show only filenames with matches
grep -l "pattern" *.txt

# Show filenames without matches
grep -L "pattern" *.txt

# Show context (lines before and after)
grep -C 2 "pattern" file.txt  # 2 lines before and after
grep -B 3 "pattern" file.txt  # 3 lines before
grep -A 1 "pattern" file.txt  # 1 line after

# Multiple patterns
grep -e "pattern1" -e "pattern2" file.txt

# Read patterns from file
grep -f patterns.txt file.txt
```

### Regular Expressions with grep

```bash
# Extended regex (-E or egrep)
grep -E "pattern1|pattern2" file.txt

# Match beginning of line
grep "^start" file.txt

# Match end of line
grep "end$" file.txt

# Match whole word
grep -w "word" file.txt

# Match any single character
grep "b.g" file.txt  # Matches: bag, big, bug

# Match zero or more
grep "go*d" file.txt  # Matches: gd, god, good, goood

# Match one or more
grep -E "go+d" file.txt  # Matches: god, good, goood

# Match specific number
grep -E "o{2,4}" file.txt  # Matches 2 to 4 o's

# Character classes
grep "[aeiou]" file.txt  # Any vowel
grep "[0-9]" file.txt    # Any digit
grep "[A-Z]" file.txt    # Any uppercase letter
```

### Practical grep Examples

```bash
# Find IP addresses
grep -E "[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}" file.txt

# Find email addresses
grep -E "[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}" file.txt

# Find specific log entries
grep "ERROR" /var/log/app.log

# Search with color highlighting
grep --color=auto "pattern" file.txt

# Quiet mode (just check if pattern exists)
if grep -q "pattern" file.txt
then
    echo "Pattern found"
fi
```

## sed - Stream Editor

`sed` performs text transformations on an input stream.

### Basic sed Usage

```bash
# Substitute (replace first occurrence on each line)
sed 's/old/new/' file.txt

# Substitute all occurrences (global)
sed 's/old/new/g' file.txt

# Edit file in-place
sed -i 's/old/new/g' file.txt

# Create backup before in-place edit
sed -i.bak 's/old/new/g' file.txt

# Case-insensitive substitution
sed 's/old/new/gi' file.txt
```

### sed Line Operations

```bash
# Delete lines
sed '5d' file.txt              # Delete line 5
sed '2,4d' file.txt            # Delete lines 2-4
sed '/pattern/d' file.txt      # Delete lines matching pattern
sed '/^$/d' file.txt           # Delete empty lines
sed '/^#/d' file.txt           # Delete comment lines

# Print specific lines
sed -n '10p' file.txt          # Print line 10
sed -n '5,10p' file.txt        # Print lines 5-10
sed -n '/pattern/p' file.txt   # Print matching lines

# Insert lines
sed '5i\New line text' file.txt     # Insert before line 5
sed '5a\New line text' file.txt     # Insert after line 5
```

### sed Advanced Patterns

```bash
# Multiple operations
sed 's/old/new/g; s/foo/bar/g' file.txt

# Using sed scripts
sed -e 's/old/new/g' -e 's/foo/bar/g' file.txt

# Conditional replacement
sed '/pattern/s/old/new/g' file.txt

# Replace on specific line numbers
sed '5s/old/new/' file.txt

# Use different delimiters (useful for paths)
sed 's|/old/path|/new/path|g' file.txt

# Capture groups
sed 's/\([0-9]\{3\}\)-\([0-9]\{4\}\)/(\1) \2/g' file.txt
```

### Practical sed Examples

```bash
# Remove leading whitespace
sed 's/^[ \t]*//' file.txt

# Remove trailing whitespace
sed 's/[ \t]*$//' file.txt

# Remove both leading and trailing whitespace
sed 's/^[ \t]*//; s/[ \t]*$//' file.txt

# Add line numbers
sed = file.txt | sed 'N;s/\n/\t/'

# Replace tabs with spaces
sed 's/\t/    /g' file.txt

# Convert DOS to Unix line endings
sed 's/\r$//' file.txt

# Convert Unix to DOS line endings
sed 's/$/\r/' file.txt

# Remove HTML tags
sed 's/<[^>]*>//g' file.html

# Extract email domain
echo "user@example.com" | sed 's/.*@//'

# Change file extension
echo "file.txt" | sed 's/\.txt$/.md/'
```

## awk - Pattern Scanning and Processing

`awk` is a powerful text processing language.

### Basic awk Usage

```bash
# Print entire line
awk '{print}' file.txt

# Print specific field (space-delimited by default)
awk '{print $1}' file.txt      # First field
awk '{print $2}' file.txt      # Second field
awk '{print $NF}' file.txt     # Last field

# Print multiple fields
awk '{print $1, $3}' file.txt

# Custom delimiter
awk -F: '{print $1}' /etc/passwd  # Colon delimiter
awk -F',' '{print $2}' data.csv   # Comma delimiter

# Custom output separator
awk -F: 'BEGIN {OFS="\t"} {print $1, $3}' /etc/passwd
```

### awk Patterns and Conditions

```bash
# Pattern matching
awk '/pattern/ {print}' file.txt

# Regex matching
awk '/^[0-9]/ {print}' file.txt

# Print lines matching field condition
awk '$3 > 100 {print}' file.txt

# Multiple conditions
awk '$3 > 100 && $4 < 500 {print}' file.txt

# Negation
awk '$1 != "root" {print}' file.txt

# String matching
awk '$1 ~ /pattern/ {print}' file.txt

# String not matching
awk '$1 !~ /pattern/ {print}' file.txt
```

### awk Built-in Variables

```bash
# NR - Current line number
awk '{print NR, $0}' file.txt

# NF - Number of fields
awk '{print NF, $0}' file.txt

# FS - Field separator
awk 'BEGIN {FS=":"} {print $1}' /etc/passwd

# OFS - Output field separator
awk 'BEGIN {OFS=" | "} {print $1, $2}' file.txt

# FILENAME - Current filename
awk '{print FILENAME, $0}' file1.txt file2.txt

# FNR - Line number in current file
awk '{print FNR, $0}' file1.txt file2.txt
```

### awk BEGIN and END

```bash
# Execute before processing
awk 'BEGIN {print "Starting..."} {print} END {print "Done"}' file.txt

# Initialize variables
awk 'BEGIN {sum=0} {sum+=$1} END {print "Total:", sum}' numbers.txt

# Print header
awk 'BEGIN {print "Name\tAge"} {print $1, $2}' data.txt
```

### awk Mathematical Operations

```bash
# Sum column
awk '{sum+=$1} END {print sum}' numbers.txt

# Average
awk '{sum+=$1; count++} END {print sum/count}' numbers.txt

# Find max value
awk 'BEGIN {max=0} {if($1>max) max=$1} END {print max}' numbers.txt

# Find min value
awk 'NR==1 {min=$1} {if($1<min) min=$1} END {print min}' numbers.txt

# Arithmetic operations
awk '{print $1 + $2}' file.txt
awk '{print $1 * 1.5}' file.txt
```

### awk Functions

```bash
# String length
awk '{print length($1)}' file.txt

# Substring
awk '{print substr($1, 1, 5)}' file.txt

# To uppercase
awk '{print toupper($1)}' file.txt

# To lowercase
awk '{print tolower($1)}' file.txt

# String replacement
awk '{gsub(/old/, "new"); print}' file.txt

# Split string
awk '{split($1, arr, ":"); print arr[1]}' file.txt
```

### Practical awk Examples

```bash
# Print specific columns from CSV
awk -F',' '{print $1, $3}' data.csv

# Calculate disk usage summary
df -h | awk 'NR>1 {print $1, $5}'

# Process log files
awk '/ERROR/ {count++} END {print "Errors:", count}' app.log

# Format output
awk '{printf "%-20s %10s\n", $1, $2}' file.txt

# Remove duplicate lines (like uniq)
awk '!seen[$0]++' file.txt

# Print lines longer than 80 characters
awk 'length > 80' file.txt

# Extract IP addresses from log
awk '{print $1}' access.log | sort | uniq -c

# Process /etc/passwd
awk -F: '{print "User:", $1, "UID:", $3}' /etc/passwd

# Calculate column statistics
awk '{sum+=$1; sumsq+=$1*$1} END {
    print "Mean:", sum/NR;
    print "StdDev:", sqrt(sumsq/NR - (sum/NR)^2)
}' data.txt
```

## cut - Extract Sections

```bash
# Extract characters
cut -c1-5 file.txt           # Characters 1-5
cut -c1,3,5 file.txt         # Characters 1, 3, and 5

# Extract fields
cut -f1 file.txt             # First field (tab-delimited)
cut -f1,3 file.txt           # Fields 1 and 3
cut -f1-3 file.txt           # Fields 1 through 3

# Custom delimiter
cut -d: -f1 /etc/passwd      # First field (colon-delimited)
cut -d',' -f2,4 data.csv     # Fields 2 and 4 (comma-delimited)

# Complement (everything except specified)
cut -d: -f1 --complement /etc/passwd
```

## sort - Sort Lines

```bash
# Basic sort
sort file.txt

# Reverse sort
sort -r file.txt

# Numeric sort
sort -n numbers.txt

# Sort by specific field
sort -k2 file.txt            # Sort by 2nd field
sort -t: -k3 -n /etc/passwd  # Numeric sort by 3rd field, colon-delimited

# Unique sort (remove duplicates)
sort -u file.txt

# Case-insensitive sort
sort -f file.txt

# Month sort
sort -M months.txt

# Human-readable numbers (K, M, G)
du -h | sort -h
```

## uniq - Report or Filter Duplicate Lines

```bash
# Remove consecutive duplicates
uniq file.txt

# Count occurrences
uniq -c file.txt

# Show only duplicates
uniq -d file.txt

# Show only unique lines
uniq -u file.txt

# Ignore case
uniq -i file.txt

# Check specific fields
uniq -f1 file.txt  # Skip first field
```

## tr - Translate or Delete Characters

```bash
# Convert to uppercase
tr 'a-z' 'A-Z' < file.txt

# Convert to lowercase
tr 'A-Z' 'a-z' < file.txt

# Delete characters
tr -d '0-9' < file.txt       # Delete all digits
tr -d ' ' < file.txt         # Delete spaces

# Squeeze repeats
tr -s ' ' < file.txt         # Squeeze multiple spaces to one

# Replace characters
tr ':' ',' < file.txt        # Replace colons with commas

# Complement (everything except)
tr -cd '0-9\n' < file.txt    # Keep only digits and newlines
```

## paste - Merge Lines

```bash
# Merge files side by side
paste file1.txt file2.txt

# Custom delimiter
paste -d',' file1.txt file2.txt

# Serial paste (one file after another)
paste -s file1.txt file2.txt
```

## join - Join Lines Based on Common Field

```bash
# Join files on first field
join file1.txt file2.txt

# Join on specific fields
join -1 2 -2 1 file1.txt file2.txt

# Custom delimiter
join -t: file1.txt file2.txt
```

## Combining Tools - Pipelines

```bash
# Count unique IP addresses in log (avoid UUOC)
awk '{print $1}' access.log | sort | uniq | wc -l

# Top 10 most frequent IP addresses
awk '{print $1}' access.log | sort | uniq -c | sort -rn | head -10

# Extract and sort email addresses
grep -Eo "[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}" file.txt | sort -u

# Process CSV data (skip comments)
grep -v "^#" data.csv | cut -d',' -f2,4 | sort -t',' -k2 -n

# Find most common words
tr -cs '[:alnum:]' '\n' < file.txt | tr '[:upper:]' '[:lower:]' | sort | uniq -c | sort -rn | head -10

# Complex log analysis
grep ERROR app.log | \
    awk '{print $1, $2, $NF}' | \
    sort | \
    uniq -c | \
    sort -rn | \
    head -20
```

## Best Practices

1. **Use appropriate tool**: grep for searching, sed for substitution, awk for field processing
2. **Quote patterns**: Prevent shell expansion
3. **Test on sample data**: Before processing large files
4. **Use pipelines**: Chain tools together for complex operations
5. **Consider performance**: awk is often faster than multiple pipes
6. **Backup important files**: Before in-place editing
7. **Use extended regex**: `-E` flag for easier patterns

## Quick Reference

```bash
# Search
grep "pattern" file              # Search pattern
grep -r "pattern" dir            # Recursive search

# Replace
sed 's/old/new/g' file           # Replace all occurrences
sed -i 's/old/new/g' file        # Edit in-place

# Field extraction
awk '{print $1}' file            # Print first field
cut -d: -f1 file                 # Cut first field

# Sort and unique
sort file | uniq                 # Remove duplicates
sort -n file                     # Numeric sort

# Count
wc -l file                       # Count lines
grep -c "pattern" file           # Count matches

# Transform
tr 'a-z' 'A-Z' < file           # To uppercase
tr -d '0-9' < file              # Delete digits
```

## Common Pitfalls

- Parsing JSON/XML with `grep`/`sed`/`awk` instead of dedicated tools (`jq`, `xmllint`).
- Using `sed -i` without considering BSD vs GNU differences; prefer `-i''` on macOS.
- Locale affecting sort order; use `LC_ALL=C sort` for byte-wise, predictable sorting.
- Useless use of `cat`; feed files directly to commands via redirection.
- Not quoting regex or replacement strings, leading to shell expansion.

## Exercises

1. Extract the second column from a colon-separated file using `cut`.
2. Replace all occurrences of `foo` with `bar` in-place in `*.txt` files (GNU and BSD variants).
3. Print the top 10 most frequent words in a text (normalize case; handle punctuation).
4. From a CSV file, print rows where column 3 > 100 using `awk`.

## Cross‑links

- [08 - Error Handling and Debugging](./08-error-handling-and-debugging.md)
- [10 - Files and Directories](./10-files-and-directories.md)

## References

- GNU Bash Reference Manual: <https://www.gnu.org/software/bash/manual/bash.html>
