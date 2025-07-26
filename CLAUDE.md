# showmethefuckingsizes - Technical Specifications

## Overview
A bash script that analyzes filesystem directory sizes and displays them in descending order. Designed for cross-platform compatibility (Linux/macOS) with special optimizations for macOS system directories.

## Core Functionality

### Primary Features
- **Directory Size Analysis**: Recursively calculates total size of directories including all subdirectories and files
- **Sorted Output**: Displays results sorted by size (largest first)  
- **Human-Readable Formatting**: Converts bytes to appropriate units (B, KB, MB, GB, TB)
- **Pagination**: Built-in pager to navigate through results without overwhelming terminal
- **Cross-Platform**: Works on both Linux and macOS with OS-specific optimizations
- **Progress Tracking**: Real-time progress indicators for long-running scans

### Usage Patterns
```bash
# Scan from root directory (default)
./showmethefuckingsizes

# Scan specific directory  
./showmethefuckingsizes /path/to/directory

# Scan home directory (with warning prompt)
./showmethefuckingsizes /Users/username

# Scan macOS Library folder (optimized mode)
./showmethefuckingsizes /Users/username/Library

# Scan Library subdirectories (now supported)
./showmethefuckingsizes /Users/username/Library/Developer
./showmethefuckingsizes /Users/username/Library/Caches
```

## Architecture

### Main Components

1. **Size Calculation Engine** (`get_dir_size()`)
   - Uses `du -sk` for directory size calculation
   - Platform-specific flags: `--max-depth=0` on Linux, no flag on macOS
   - Error handling for permission denied/inaccessible directories
   - Returns size in kilobytes

2. **Formatting Engine** (`format_size()`)
   - Converts bytes to human-readable units using base-1024 calculation
   - Supports units: B, KB, MB, GB, TB
   - Uses `bc` calculator for floating-point arithmetic
   - Displays one decimal place precision

3. **Pagination System** (`paginate_output()`)
   - Terminal height detection via `tput lines`
   - Space bar for next page, Ctrl+C to exit
   - Automatic line counting and page breaks

4. **Directory Discovery Engine**
   - Uses `find` with dynamic exclusion patterns
   - Context-aware exclusions based on scanning location
   - Limited to max depth of 2 for performance
   - Extensive exclusion list for problematic directories

### Special Handling Modes

#### macOS Library Optimization (`handle_library_folder()`)
- **Trigger**: When scanning `/*/Library` directories
- **Method**: Targeted scanning of known large subdirectories instead of full traversal
- **Target Directories**: 
  - Developer, Containers, Application Support
  - Mobile Documents, Caches, CloudStorage
  - Group Containers, Saved Application State
  - WebKit, Logs, Preferences
- **Performance**: Dramatically faster than full Library scan

#### Home Directory Protection
- **Trigger**: When scanning `/Users/*` directories (except Shared)
- **Behavior**: Interactive warning with alternatives
- **Suggested Alternatives**: Library (optimized), Documents, Downloads
- **User Choice**: Cancel or proceed with full scan

## Technical Implementation

### Dependencies
- **Required**: `bash`, `find`, `du`, `cut`, `grep`, `sort`, `bc`, `tput`
- **Platform**: Linux (any distribution) or macOS
- **Minimum bash version**: 4.0+ (for associative arrays and modern features)

### Performance Optimizations

#### Directory Exclusions
```bash
# Dynamic exclusion patterns based on scanning context:

# When scanning OUTSIDE Library directories:
-path "*/proc/*"           # Linux virtual filesystem
-path "*/sys/*"            # Linux virtual filesystem  
-path "*/dev/*"            # Device files
-path "*/.git/*"           # Git repositories
-path "*/node_modules/*"   # Node.js dependencies
-path "*/.npm/*"           # NPM cache
-path "*/Library"          # macOS Library (handled separately)
-path "*/Library/*"        # macOS Library subdirs (excluded)
-path "*/System/*"         # macOS system directories
-path "*/private/*"        # macOS private directories
-path "*/.Trash/*"         # Trash directories
-path "*/Pictures/Photos Library.photoslibrary*"  # macOS Photos

# When scanning INSIDE Library directories:
# (Library/* exclusion is removed to allow subdirectory scanning)
-path "*/proc/*"           # Linux virtual filesystem
-path "*/sys/*"            # Linux virtual filesystem  
-path "*/dev/*"            # Device files
-path "*/.git/*"           # Git repositories
-path "*/node_modules/*"   # Node.js dependencies
-path "*/.npm/*"           # NPM cache
-path "*/System/*"         # macOS system directories
-path "*/private/*"        # macOS private directories
-path "*/.Trash/*"         # Trash directories
-path "*/Pictures/Photos Library.photoslibrary*"  # macOS Photos
```

#### Performance Features
- **Dynamic Exclusions**: Context-aware patterns for optimal performance
- **Depth Limiting**: `maxdepth 2` prevents infinite recursion
- **Progress Indicators**: Every 100 directories processed
- **Early Termination**: User can cancel at any point
- **Efficient Sorting**: Numeric sort on padded values for performance

### Error Handling

#### Permission Errors
- Redirects stderr to `/dev/null` for inaccessible directories
- Graceful fallback to size "0" for calculation errors
- Continues processing despite individual directory failures

#### Missing Dependencies
- Checks for `bc` calculator availability
- Provides platform-specific installation instructions
- Exits gracefully if requirements not met

#### Invalid Input
- Validates directory existence and readability
- Handles special characters in directory names
- Protects against shell injection via proper quoting

### Output Format

#### Standard Mode
```
Directory sizes (largest first):
================================

45.2GB    /path/to/large/directory
12.7GB    /path/to/another/directory
3.1GB     /path/to/smaller/directory
```

#### Library Mode  
```
Directory sizes in Library (largest first):
=========================================

60.6GB    /Users/username/Library/Developer
12.7GB    /Users/username/Library/Mobile Documents
7.1GB     /Users/username/Library/CloudStorage
```

#### Progress Display
```
Scanning directories and calculating sizes...
This may take a while for large filesystems

Found 1247 directories to scan

Progress: 100/1247 directories processed...
Progress: 200/1247 directories processed...
```

### Color Coding
- **Green**: Headers, progress messages, success states
- **Yellow**: Warnings, size values, interactive prompts  
- **Blue**: Directory paths, informational text
- **Red**: Errors, missing dependencies

## Configuration

### Environment Variables
- **OSTYPE**: Automatically detected for platform-specific behavior
- Terminal capabilities detected via `tput` for pagination

### Customization Points
- Exclusion patterns can be modified in the `find` commands
- Library target directories can be adjusted in `major_dirs` array
- Color codes can be modified in the color constants section
- Pagination behavior can be adjusted in `paginate_output()`

## Testing and Validation

### Test Cases
1. **Basic functionality**: Scan small directory tree
2. **Large directory handling**: Scan /var or /usr on Linux systems
3. **Permission handling**: Scan restricted directories  
4. **macOS Library optimization**: Verify targeted scanning works
5. **Home directory warning**: Verify prompt appears and works
6. **Cross-platform**: Test on both Linux and macOS
7. **Missing dependencies**: Test with `bc` uninstalled

### Performance Benchmarks
- Library scan: ~5-10 seconds vs 5+ minutes for full scan
- Home directory warning prevents multi-hour hang scenarios
- Progress indicators prevent timeout on slow filesystems

## Security Considerations

### Safe Practices
- No write operations - read-only tool
- Proper shell quoting prevents injection attacks
- Error redirection prevents information leakage
- No temporary files created in shared locations

### Permission Handling
- Respects filesystem permissions
- Gracefully handles permission denied errors
- Does not attempt privilege escalation
- Safe for use with restricted user accounts

## Future Enhancement Opportunities

### Potential Features
- Configurable exclusion patterns via command line
- JSON/CSV output format options
- Disk usage threshold highlighting
- Integration with system monitoring tools
- Recursive size caching for faster subsequent runs
- Network filesystem detection and handling

### Performance Improvements  
- Parallel directory processing for multi-core systems
- Smart caching of previously calculated sizes
- Integration with OS-native filesystem APIs
- Memory usage optimization for very large filesystems