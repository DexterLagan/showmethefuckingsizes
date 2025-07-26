# showmethefuckingsizes

A straightforward command-line utility that shows you every folder on your system, sorted by size with the largest folders at the top.

## What it does

This tool recursively scans your filesystem and displays directories sorted by their total size (including all subdirectories and files). It's useful for:

- Finding what's eating up your disk space
- Identifying large directories that might be good candidates for cleanup
- Getting a quick overview of storage usage across your system
- Analyzing specific subdirectories like macOS Library folders

## Why it exists

Modern operating systems don't typically pre-compute directory sizes since it would be expensive to maintain in real-time. This means when you need to find large directories, you're often left running manual commands or using GUI tools that can be slow or limited. This utility fills that gap by providing a simple, fast way to see your largest directories at a glance.

The tool automatically paginates output (like `more` or `less`) so you can browse through results without overwhelming your terminal.

## Features

- **Smart Exclusions**: Automatically skips system directories, caches, and other paths that would slow down scanning
- **macOS Optimizations**: Special handling for Library folders with targeted scanning of major subdirectories
- **Library Subdirectory Support**: Can now scan specific Library subdirectories like Developer, Caches, etc.
- **Cross-Platform**: Works on both Linux and macOS
- **Progress Tracking**: Shows real-time progress for long-running scans
- **Safety Warnings**: Prompts before scanning potentially large home directories

## Usage

```bash
# Scan from root directory (default)
./showmethefuckingsizes

# Scan a specific directory
./showmethefuckingsizes /path/to/directory

# Scan your home directory (with safety prompt)
./showmethefuckingsizes /Users/yourname

# Scan macOS Library folder (uses optimized mode)
./showmethefuckingsizes /Users/yourname/Library

# Scan Library subdirectories (now supported!)
./showmethefuckingsizes /Users/yourname/Library/Developer
./showmethefuckingsizes /Users/yourname/Library/Caches
```

Navigate through results:
- Press `Space` to show the next page
- Press `Ctrl+C` to exit

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.