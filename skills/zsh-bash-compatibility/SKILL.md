---
name: zsh-bash-compatibility
description: Bash vs Zsh 差異與避坑指南。撰寫 zsh/bash 腳本時自動觸發。
user-invocable: false
---

# Shell Script Compatibility Guide

Bash and Zsh are the two most common interactive shells, but they differ in subtle ways that can break scripts. This guide catalogues the key differences and provides workarounds for writing scripts that run correctly in both.

## Array Indexing

Bash arrays start at 0; Zsh arrays start at 1.

```bash
# Bash
arr=(apple banana cherry)
echo ${arr[0]}  # apple

# Zsh
arr=(apple banana cherry)
echo ${arr[1]}  # apple
```

**Workaround — force consistent indexing**:
```bash
[ -n "$ZSH_VERSION" ] && setopt KSH_ARRAYS
arr=(apple banana cherry)
echo ${arr[0]}  # "apple" in both shells
```

**Workaround — iterate instead of indexing**:
```bash
for item in "${arr[@]}"; do
    echo "$item"
done
```

For positional arguments, extract them into named variables early:
```bash
for item in "$@"; do
    case $item in
        --option)
            option_value="$2"
            shift 2
            ;;
        *)
            positional_args+=("$item")
            shift
            ;;
    esac
done
```

## Detecting the Script's Own Path

```bash
# Bash — BASH_SOURCE is reliable
echo "${BASH_SOURCE[0]}"

# Zsh — BASH_SOURCE doesn't exist
echo "${(%):-%x}"
```

**Workaround**:
```bash
if [ -n "$BASH_SOURCE" ]; then
    SCRIPT_PATH="${BASH_SOURCE[0]}"
elif [ -n "$ZSH_VERSION" ]; then
    SCRIPT_PATH="${(%):-%x}"
else
    SCRIPT_PATH="$0"
fi
SCRIPT_DIR="$(dirname "$SCRIPT_PATH")"
```

## Word Splitting on Unquoted Variables

Bash splits unquoted variables on whitespace; Zsh does not.

```bash
# Bash
var="one two three"
echo $var   # three separate arguments

# Zsh
var="one two three"
echo $var   # single argument (no split)
```

**Workaround**: Always quote variables.
```bash
echo "$var"
```

When you *want* splitting, use arrays:
```bash
read -ra words <<< "$var"
for word in "${words[@]}"; do
    echo "$word"
done
```

## Recursive Globbing

```bash
# Bash — requires opt-in
shopt -s globstar
echo **/*.txt

# Zsh — enabled by default
echo **/*.txt
```

**Workaround — enable globstar conditionally**:
```bash
[ -n "$BASH_VERSION" ] && shopt -s globstar
echo **/*.txt
```

**Workaround — use `find` instead**:
```bash
find . -name "*.txt" -type f
```

## Associative Arrays

```bash
# Bash
declare -A map
map[key]="value"

# Zsh
typeset -A map
map=(key value)
```

**Workaround**:
```bash
declare -A map 2>/dev/null || typeset -A map
map[key]="value"
echo ${map[key]}
```

## The `$path` Trap

In Zsh, `$path` is a special array tied to `$PATH`. Assigning to it changes your search path:

```zsh
local path="oops"
echo $PATH  # now broken
```

In Bash, `$path` is just an ordinary variable.

**Rule**: Never use `path` as a variable name. Always refer to the environment variable as `$PATH`.

## Summary of Best Practices

1. Use `#!/bin/bash` as the shebang for maximum portability
2. Add `setopt KSH_ARRAYS` at the top if the script must also run under Zsh
3. Always quote variables: `"$var"`, not `$var`
4. Expand arrays with `"${arr[@]}"`
5. Use the script-path detection snippet for reliable self-location
6. Prefer `find` over complex glob patterns
7. Never use `path` as a variable name
