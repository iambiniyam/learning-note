# Bash Learning Notes (01–13)

A beginner-friendly, structured set of notes to learn Bash step by step. Each chapter is practical, cross-linked, and includes exercises, pitfalls, and references to help you build real skills.

## How to use this guide

- Start at 01 and read in order; each chapter links to the next and related topics.
- Try the examples in your terminal as you read.
- Do the exercises at the end of each chapter; they reinforce the key ideas.
- Keep the GNU Bash Reference Manual handy for deeper details.

## Prerequisites

- Bash 4.4+ recommended (Bash 5.x ideal). Many examples also work on older versions, but arrays, associative arrays, and some expansions require 4.x+.
- Familiarity with a terminal/shell is helpful but not required.

## Windows users

- Recommended: WSL (Windows Subsystem for Linux) with Ubuntu from the Microsoft Store, or Git Bash.
- Save scripts with Unix line endings (LF) to avoid `bad interpreter` errors.
- Some Linux utilities (e.g., `ip`, `ss`, `dig`, `lsof`) aren’t available on plain Windows; prefer WSL.

## Table of Contents

1. 01 – Introduction → `01-introduction.md`
2. 02 – Creating and Running Scripts → `02-creating-and-running-scripts.md`
3. 03 – Variables and Data Types → `03-variables-and-data-types.md`
4. 04 – User Input and Arguments → `04-user-input-and-arguments.md`
5. 05 – Conditionals → `05-conditionals.md`
6. 06 – Loops → `06-loops.md`
7. 07 – Functions → `07-functions.md`
8. 08 – Error Handling and Debugging → `08-error-handling-and-debugging.md`
9. 09 – Text Processing → `09-text-processing.md`
10. 10 – Files and Directories → `10-files-and-directories.md`
11. 11 – Process Management → `11-process-management.md`
12. 12 – Networking and APIs → `12-networking-and-apis.md`
13. 13 – Advanced Topics → `13-advanced-topics.md`

## Conventions

- Code blocks use Bash with quotes and safe patterns.
- Prefer `set -euo pipefail` for scripts; turn on `set -x` for debugging when needed.
- Quote variables ("$var"), use `[[ ... ]]` for tests, avoid UUOC (useless use of `cat`).
- Use `jq` for JSON; avoid parsing JSON with `grep`/`sed`/`awk`.

## Tips for success

- Practice: small scripts daily beat long weekend marathons.
- Keep a scratch folder for experiments.
- Read errors carefully—exit codes and messages are clues.
- Skim References sections for deeper reading.

## Linting and formatting (optional)

If you use a Markdown linter, ensure:

- Headings increment by one level at a time.
- Blank lines around lists and fenced code blocks.
- Fenced code blocks include a language tag (e.g., ```bash).

## License and sources

- Content references the GNU Bash Reference Manual: <https://www.gnu.org/software/bash/manual/bash.html>
- Examples are original and designed for learning purposes.
