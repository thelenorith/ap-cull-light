# ap-cull-lights

[![Test](https://github.com/jewzaam/ap-cull-light/actions/workflows/test.yml/badge.svg)](https://github.com/jewzaam/ap-cull-light/actions/workflows/test.yml)
[![Coverage](https://github.com/jewzaam/ap-cull-light/actions/workflows/coverage.yml/badge.svg)](https://github.com/jewzaam/ap-cull-light/actions/workflows/coverage.yml)
[![Lint](https://github.com/jewzaam/ap-cull-light/actions/workflows/lint.yml/badge.svg)](https://github.com/jewzaam/ap-cull-light/actions/workflows/lint.yml)
[![Format](https://github.com/jewzaam/ap-cull-light/actions/workflows/format.yml/badge.svg)](https://github.com/jewzaam/ap-cull-light/actions/workflows/format.yml)
[![Typecheck](https://github.com/jewzaam/ap-cull-light/actions/workflows/typecheck.yml/badge.svg)](https://github.com/jewzaam/ap-cull-light/actions/workflows/typecheck.yml)
[![Python 3.10+](https://img.shields.io/badge/python-3.10+-blue.svg)](https://www.python.org/downloads/)
[![Code style: black](https://img.shields.io/badge/code%20style-black-000000.svg)](https://github.com/psf/black)

A simple tool for culling astrophotography light frames based on HFR and RMS thresholds.

## Overview

This tool evaluates light frames against HFR and RMS thresholds and moves rejected images to a reject directory while preserving the relative directory structure.

## Documentation

This tool is part of the astrophotography pipeline. For comprehensive documentation including workflow guides and integration with other tools, see:

- **[Pipeline Overview](https://github.com/jewzaam/ap-base/blob/main/docs/index.md)** - Full pipeline documentation
- **[Workflow Guide](https://github.com/jewzaam/ap-base/blob/main/docs/workflow.md)** - Detailed workflow with diagrams
- **[ap-common Reference](https://github.com/jewzaam/ap-base/blob/main/docs/tools/ap-common.md)** - API reference for this tool

## Features

- **Simple Threshold Checks**: Reject images based on HFR and RMS thresholds
- **Batch Processing**: Groups files by directory for efficient batch rejection confirmation
- **Auto-Accept Threshold**: Automatically accept rejections below a configurable percentage
- **Dry Run Mode**: Test rules without actually moving files

## Usage

### Basic Culling

```powershell
python -m ap_cull_lights.cull_lights <source_dir> <reject_dir> --max-hfr 2.5 --max-rms 2.0
```

### With Auto-Accept Threshold

```powershell
python -m ap_cull_lights.cull_lights <source_dir> <reject_dir> --max-hfr 2.5 --max-rms 2.0 --auto-accept-percent 5.0
```

This will automatically accept rejections if less than 5% of images in a directory group are rejected.

### Dry Run

```powershell
python -m ap_cull_lights.cull_lights <source_dir> <reject_dir> --max-hfr 2.5 --max-rms 2.0 --dryrun
```

### Debug Mode

```powershell
python -m ap_cull_lights.cull_lights <source_dir> <reject_dir> --max-hfr 2.5 --max-rms 2.0 --debug
```

## Options

- `source_dir`: Source directory containing light frames to cull
- `reject_dir`: Directory to move rejected images to (relative structure preserved)
- `--max-hfr FLOAT`: Maximum HFR value (reject if HFR > max_hfr)
- `--max-rms FLOAT`: Maximum RMS in arcsec (reject if RMS > max_rms)
- `--auto-accept-percent FLOAT`: Auto-accept if rejection percentage is below this (0.0-100.0, default: -1 = always prompt)
- `--skip-regex REGEX`: Regex pattern to skip files/directories (e.g., 'accept' or 'accept|processed' to skip multiple patterns)
- `--debug`: Enable debug output
- `--dryrun`: Perform dry run without moving files
- `--help`: Show help message and exit

## Installation

### From Source (Development)

```powershell
make install-dev
```

This installs the package in editable mode along with all dependencies (including `ap-common` from git) and development tools.

### From Git Repository (One-liner)

```powershell
pip install git+https://github.com/jewzaam/ap-cull-lights.git
```

This installs the package directly from the GitHub repository without requiring a local checkout.

### Uninstallation

```powershell
make uninstall
```

## How It Works

1. Scans `source_dir` recursively for LIGHT files (FITS and XISF formats)
2. Reads FITS headers to extract metadata (HFR, RMS, star count, etc.)
3. Skips files matching the `--skip-regex` regex pattern
4. **Groups files by directory** - Files are grouped by their parent directory path. Each directory group is processed separately with its own rejection confirmation prompt. This means:
   - Files in the same directory are evaluated together as a batch
   - The rejection percentage is calculated per directory group
   - Confirmation prompts are shown per directory group
   - **Important**: Organize your files into directories before running cull (e.g., by target, filter, or session) to ensure proper grouping
5. Evaluates each file against HFR and RMS thresholds
6. Prompts for confirmation before rejecting (unless auto-accept threshold is met)
7. Moves rejected files to `reject_dir`, preserving relative directory structure

## Examples

### Basic Culling

```powershell
python -m ap_cull_lights.cull_lights D:\Astrophotography\Data D:\Astrophotography\Reject --max-hfr 2.5 --max-rms 2.0
```

### With Auto-Accept

```powershell
python -m ap_cull_lights.cull_lights D:\Astrophotography\Data D:\Astrophotography\Reject --max-hfr 2.5 --max-rms 2.0 --auto-accept-percent 3.0
```

This will automatically accept rejections if less than 3% of images are rejected in a directory group.

### Skipping Files with Regex

```powershell
python -m ap_cull_lights.cull_lights D:\Astrophotography\Data D:\Astrophotography\Reject --max-hfr 2.5 --max-rms 2.0 --skip-regex "accept|processed"
```

This will skip any files whose path contains "accept" or "processed" (case-sensitive regex matching).

### Dry Run to Test Rules

```powershell
python -m ap_cull_lights.cull_lights D:\Astrophotography\Data D:\Astrophotography\Reject --max-hfr 2.5 --max-rms 2.0 --dryrun
```

## License

Apache-2.0
