# Data Archiver

A fast, multi-process file archiver that copies files older than a specified date from a source directory tree to a destination, preserving directory structure.

## Features

- **Flexible date parsing**: Supports natural language dates like "3 years ago", year-only "2021", or ISO format "2025-06-01"
- **Fast hashing**: Uses xxhash (xxh3_128) for extremely fast file comparison
- **Multi-process architecture**: Uses 8+ worker processes for parallel file processing
- **Producer/consumer pattern**: Dedicated crawler process feeds work to hash/copy workers
- **Conflict detection**: Identifies files with same path but different content
- **Dry run mode**: Preview what would be archived without making changes
- **Verbose logging**: Detailed logs with timestamps and file hashes
- **Progress display**: Spinner during crawl, progress bar once file count is known
- **Colorful output**: Rich terminal interface with colored summary tables

## Requirements

- Python 3.10+
- [uv](https://docs.astral.sh/uv/) (recommended) or pip

## Installation

No installation required! The script uses [PEP 723](https://peps.python.org/pep-0723/) inline metadata, so dependencies are automatically managed.

## Usage

The script has a special shebang (`#!/usr/bin/env -S uv run`) that automatically invokes `uv run`, which handles the PEP 723 inline dependencies. Just run it directly:

```bash
# Run directly (uv handles dependencies automatically)
./archive --source /path/to/source --destination /path/to/dest --older-than '3 years ago'

# Dry run to see what would be archived
./archive --source /path/to/source --destination /path/to/dest --older-than '2021' --dry-run
```

### Alternative: Explicit uv run

You can also invoke it explicitly if you prefer:

```bash
uv run ./archive --source /path/to/source --destination /path/to/dest --older-than '2025-06-01'
```

## Command Line Options

```
usage: archive [-h] --source SOURCE --destination DESTINATION --older-than OLDER_THAN
               [--log-directory LOG_DIRECTORY] [--dry-run]

Archive files older than a specified date.

options:
  -h, --help            show this help message and exit
  --source, -s SOURCE   Source directory to archive from
  --destination, -d DESTINATION
                        Destination directory to archive to
  --older-than, -o OLDER_THAN
                        Archive files older than this date
  --log-directory, -l LOG_DIRECTORY
                        Directory for log files (default: current directory)
  --dry-run, -n         Show what would be done without actually copying files
```

## Date Format Examples

The `--older-than` option accepts many formats:

- **Relative dates**: `"3 years ago"`, `"6 months ago"`, `"2 weeks ago"`
- **Year only**: `"2021"`, `"2020"`
- **ISO format**: `"2025-06-01"`, `"2024-12-31"`
- **Natural language**: `"January 2023"`, `"March 15, 2022"`

## Examples

```bash
# Archive files older than 3 years
./archive -s /home/user/documents -d /mnt/archive/documents -o '3 years ago'

# Archive files from before 2022, with custom log directory
./archive -s /data -d /backup/data -o '2022' -l /var/log/archiver

# Dry run to preview archiving files older than June 2024
./archive -s /projects -d /archive/projects -o '2024-06-01' --dry-run

# Archive with all short options
./archive -s /src -d /dst -o '2021-01-01' -l ./logs -n
```

## Output Files

The archiver creates the following files in the log directory:

### Log File: `data-archiver-YYYY-MM-DD-HHMMSS.log`

A detailed log containing:
- Every file processed with its action (COPIED, SKIPPED, MTIME UPDATED, CONFLICT, ERROR)
- File hashes (xxh3_128) for all processed files
- Timestamps for every operation
- Complete summary statistics

### Conflicts File: `conflicts-YYYY-MM-DD-HHMMSS.txt`

Created only if conflicts are detected. Contains the full source paths of files that:
- Exist in the destination with the same relative path
- Have different content (different hash) than the source

## How It Works

1. **Crawler Process**: Walks the source directory tree, finding files with modification times older than the cutoff date
2. **Worker Processes**: 8+ parallel workers that:
   - Check if destination file exists
   - Compare file sizes (quick check)
   - Compute xxhash for content comparison if sizes match
   - Copy new files, preserving directory structure
   - Update modification times to match source
   - Track conflicts for manual review

3. **Progress Display**: Shows a spinner during crawling, then switches to a progress bar once the total file count is known

## Summary Output

After completion, you'll see a summary table including:
- Directories crawled
- Files found (matching age criteria)
- Files copied / skipped / updated
- Conflicts and errors
- Bytes copied (exact and human-readable)
- Total runtime and throughput
