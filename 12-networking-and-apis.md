# Networking and APIs in Bash

[Previous: 11 - Process Management](./11-process-management.md) • [Next: 13 - Advanced Topics](./13-advanced-topics.md)

Bash is a powerful tool for network diagnostics, automation, and interacting with web APIs. This guide covers common networking commands and how to use `curl` and `jq` to work with APIs.

## Basic Networking Commands

### `ping` - Check Network Connectivity

```bash
# Ping a host to check if it's reachable
ping google.com

# Ping a specific number of times
ping -c 4 google.com

# Set interval between pings
ping -i 2 google.com
```

### `ifconfig` / `ip` - Configure Network Interfaces

```bash
# Show network interface configuration (older command)
ifconfig

# Show network interface configuration (modern command)
ip addr show
ip a

# Show routing table
ip route show

# Bring an interface up or down
sudo ip link set eth0 up
sudo ip link set eth0 down
```

### `netstat` / `ss` - Network Statistics

```bash
# Show listening ports (older command)
netstat -tuln

# Show listening ports (modern command)
ss -tuln
# -t: TCP, -u: UDP, -l: listening, -n: numeric

# Show all connections
ss -a

# Filter by state
ss -t -a state established
```

### `host` / `dig` / `nslookup` - DNS Lookups

```bash
# Simple DNS lookup
host google.com

# Detailed DNS lookup
dig google.com

# Query for specific record type
dig google.com MX   # Mail exchange records
dig google.com AAAA # IPv6 records

# Reverse DNS lookup
dig -x 8.8.8.8

# Interactive DNS lookup (older tool)
nslookup google.com
```

### `traceroute` - Trace Packet Route

```bash
# Trace the path packets take to a host
traceroute google.com
```

## Transferring Files

### `scp` - Secure Copy

```bash
# Copy file to remote host
scp localfile.txt user@remotehost:/remote/path/

# Copy file from remote host
scp user@remotehost:/remote/path/file.txt /local/path/

# Copy directory recursively
scp -r local_dir/ user@remotehost:/remote/path/
```

### `rsync` - Fast, Versatile File Copying

```bash
# Sync local directory to remote
rsync -avz local_dir/ user@remotehost:/remote/path/
# -a: archive mode, -v: verbose, -z: compress

# Sync remote directory to local
rsync -avz user@remotehost:/remote/path/ local_dir/

# Dry run (show what would be transferred)
rsync -avzn local_dir/ user@remotehost:/remote/path/

# Delete files on destination that are not on source
rsync -avz --delete local_dir/ user@remotehost:/remote/path/
```

### `wget` - Non-interactive Network Downloader

```bash
# Download a file
wget https://example.com/file.zip

# Download and rename
wget -O newname.zip https://example.com/file.zip

# Download in the background
wget -b https://example.com/largefile.iso

# Resume an interrupted download
wget -c https://example.com/largefile.iso

# Mirror a website
wget --mirror -p --convert-links -P ./local-dir https://example.com
```

## Interacting with APIs using `curl`

`curl` is a versatile tool for transferring data with URLs. It's essential for working with REST APIs.

### Basic `GET` Request

```bash
# Simple GET request
curl https://api.github.com/users/torvalds

# Save output to a file
curl -o user.json https://api.github.com/users/torvalds

# Show only HTTP headers
curl -I https://api.github.com

# Include headers in output
curl -i https://api.github.com

# Silent mode (no progress meter)
curl -s https://api.github.com
```

### `POST` Requests

```bash
# Send POST data (form-encoded)
curl -d "name=Alice&email=alice@example.com" https://api.example.com/users

# Send POST data from a file
curl -d @data.txt https://api.example.com/users

# Send JSON data
curl -X POST -H "Content-Type: application/json" \
  -d '{"name":"Bob","email":"bob@example.com"}' \
  https://api.example.com/users
```

### Other HTTP Methods (`PUT`, `DELETE`, etc.)

```bash
# PUT request to update data
curl -X PUT -H "Content-Type: application/json" \
  -d '{"email":"new.email@example.com"}' \
  https://api.example.com/users/123

# DELETE request to remove data
curl -X DELETE https://api.example.com/users/123
```

### Authentication

```bash
# Basic authentication
curl -u "username:password" https://api.example.com/secure

# Bearer token authentication (e.g., OAuth2)
curl -H "Authorization: Bearer YOUR_ACCESS_TOKEN" https://api.example.com/secure
```

### Handling Redirects and Errors

```bash
# Follow redirects
curl -L https://google.com

# Fail silently on server errors
curl -f https://api.example.com/nonexistent
echo $?  # Will be non-zero if error
```

## Processing JSON with `jq`

`jq` is a lightweight and flexible command-line JSON processor.

### Basic Filtering

```bash
# Pretty-print JSON
curl -s 'https://api.github.com/users/torvalds' | jq '.'

# Get a specific field
curl -s '...' | jq '.name'

# Get nested fields
curl -s '...' | jq '.payload.commits[0].author.name'
```

### Array and Object Manipulation

```bash
# Get first element of an array
curl -s '...' | jq '.[0]'

# Get array length
curl -s '...' | jq 'length'

# Iterate over an array
curl -s '...' | jq '.[]'

# Create a new object
curl -s '...' | jq '{user_id: .id, username: .login}'

# Create an array of values
curl -s '...' | jq '[.followers, .following]'
```

### Filtering and Selecting

```bash
# Select objects based on a condition
# Get repos that are not forks
curl -s 'https://api.github.com/users/torvalds/repos' | \
  jq '.[] | select(.fork == false)'

# Combine filters
curl -s '...' | jq '.[] | {name: .name, stars: .stargazers_count}'

# Find repo with most stars
curl -s '...' | jq 'max_by(.stargazers_count)'
```

## Practical Examples

### Check Website Status

```bash
#!/bin/bash
URL="https://example.com"
STATUS_CODE=$(curl -s -o /dev/null -w "%{http_code}" "$URL")

if [ "$STATUS_CODE" -eq 200 ]; then
    echo "$URL is up and running."
else
    echo "$URL is down (HTTP status: $STATUS_CODE)."
fi
```

### Get Public IP Address

```bash
# Using a public API
curl -s https://api.ipify.org
# or
curl -s ifconfig.me
```

### GitHub API Script

```bash
#!/bin/bash
# Get the names of the 5 most recently pushed repos for a user

USER="torvalds"
URL="https://api.github.com/users/$USER/repos?sort=pushed&per_page=5"

echo "Fetching 5 most recent repos for $USER..."
curl -s "$URL" | jq '.[] | .name'
```

### Weather Report Script

```bash
#!/bin/bash
# Get weather for a city using a free weather API (e.g., wttr.in)

CITY="London"
curl -s "https://wttr.in/$CITY?format=3"
```

### Post to Slack

```bash
#!/bin/bash
# Post a message to a Slack channel using a webhook

WEBHOOK_URL="YOUR_SLACK_WEBHOOK_URL"
MESSAGE="Hello from a bash script!"

curl -X POST -H 'Content-type: application/json' \
  --data "{\"text\":\"$MESSAGE\"}" \
  "$WEBHOOK_URL"
```

## Best Practices

- **Quote URLs**: Always quote URLs in `curl` and `wget` to prevent shell interpretation of special characters.
- **Use `-s` with `curl` in scripts**: Avoids the progress meter in your output.
- **Check HTTP status codes**: Don't assume a `curl` request was successful.
- **Use `jq` for JSON**: Avoid parsing JSON with `grep`, `sed`, or `awk`. `jq` is safer and more powerful.
- **Store secrets securely**: Don't hardcode API keys or passwords in scripts. Use environment variables or a secrets management tool.
- **Set User-Agent**: Some APIs require a `User-Agent` header.

  ```bash
  curl -A "MyCoolScript/1.0" https://api.example.com
  ```

- **Handle rate limiting**: Be aware of API rate limits and build in delays (`sleep`) if necessary.

> Note (Windows): Many networking tools here (e.g., `ip`, `ss`, `dig`, `traceroute`) are Linux utilities. On Windows, use WSL (Windows Subsystem for Linux) or Git Bash to access them, or PowerShell equivalents.

## Common Pitfalls

- Parsing JSON with `grep`/`sed`/`awk` instead of `jq` leads to brittle scripts; prefer `jq` for correctness.
- Forgetting to quote URLs or data can cause shell interpretation issues (e.g., `&` in query strings).
- Assuming HTTP 200 means success; some APIs return useful error info with other status codes—always check `%{http_code}`.
- Hardcoding secrets/tokens in scripts checked into version control; use environment variables or a secrets manager.
- Ignoring rate limits and getting blocked; handle `429 Too Many Requests` and backoff with `sleep`.

## Exercises

1. Use `curl -I` to fetch headers for a site and print only the `Server` header.
2. Fetch your public IP with two different services and compare results.
3. Call the GitHub API to list a user’s repos and use `jq` to output `{name, stars}` pairs sorted by stars.
4. Write a script that follows redirects (`-L`) and prints the final URL using `-w "%{url_effective}"`.
5. Post a JSON payload to a test endpoint (e.g., httpbin.org/post) with a custom User-Agent and parse the echoed JSON with `jq`.

## Cross‑links

- [11 - Process Management](./11-process-management.md)
- [13 - Advanced Topics](./13-advanced-topics.md)

## References

- GNU Bash Reference Manual: <https://www.gnu.org/software/bash/manual/bash.html>
