# scanimage-handy

A user-friendly CLI wrapper for `scanimage` that makes everyday scanning effortless.

## Why?

Raw `scanimage` requires verbose commands and manual configuration:

```bash
# scanimage-handy: 15 characters, uses configured device
scanimage-handy

# vs raw scanimage: 87 characters, must remember device name
scanimage -d 'epson2:libusb:001:003' --format=png --resolution=600 -o scan.png
```

| Pain Point | scanimage | scanimage-handy |
|------------|-----------|-----------------|
| Device selection | Copy-paste from `scanimage -L` | Interactive picker (`-S`) or env var |
| Output files | Redirect stdout or `-o` each time | Auto-timestamped filenames |
| ADF Duplex | Manual `--batch-double --batch-increment=1` | Just `-s duplex` |
| Scan failures | Start over | Auto-retry with configurable attempts |
| File collisions | Overwrites silently | Waits and generates unique name |

### ADF Duplex: From Arcane to Simple

```bash
# scanimage-handy: just this
scanimage-handy -s duplex

# vs raw scanimage: must know 6 flags and exact strings
scanimage -d 'epson2:libusb:001:003' --format=png --source='ADF Duplex' \
  --batch='doc_%d.png' --batch-double --batch-increment=1
```

The script automatically enables batch mode, adds `--batch-double` and `--batch-increment=1` for correct page ordering.

**Bonus:** If the scanner hiccups mid-batch, raw `scanimage` fails and you restart manually. `scanimage-handy` retries automatically (configurable via `--retries`).

## Quick Start

```bash
# One-time setup: add to PATH and set default device
export PATH="$PATH:/path/to/scanimage-handy"
export SCANIMAGE_DEVICE='your-device-name'  # from scanimage -L

# Scan! Output: scanimage-2024-01-15--14-30-22.png
scanimage-handy
```

## Features

* **Set-and-forget device** - Configure once via `SCANIMAGE_DEVICE`
* **Interactive device picker** - `-S` shows numbered list to choose from
* **Smart filenames** - Automatic timestamps, collision avoidance
* **Source aliases** - `flat`, `bed`, `adf`, `duplex` instead of exact strings
* **Retry logic** - Automatic retries on transient failures
* **ADF Duplex automation** - Correct batch flags added automatically
* **Viewer integration** - `-V viu` to preview scans immediately
* **Sensible defaults** - PNG format, 600 DPI, Color mode

## Installation

1. Clone or download this repository
2. Add to your PATH:

   ```bash
   # Option A: Symlink to a directory in PATH
   ln -s /path/to/scanimage-handy/scanimage-handy ~/.local/bin/

   # Option B: Add repo directory to PATH
   export PATH="$PATH:/path/to/scanimage-handy"
   ```

3. Set your default scanner (recommended):

   ```bash
   # Find your device
   scanimage -L

   # Add to ~/.bashrc or ~/.zshrc
   export SCANIMAGE_DEVICE='epson2:libusb:001:003'
   ```

## Examples

```bash
# Basic scan with defaults (PNG, 600 DPI, Color)
scanimage-handy

# Choose scanner interactively
scanimage-handy -S

# Grayscale PDF at 300 DPI
scanimage-handy -m Gray -r 300 -f pdf

# ADF duplex scan (auto-configures batch mode)
scanimage-handy -s duplex

# Batch scan with viewer
scanimage-handy --batch -V viu

# Custom basename
scanimage-handy -b invoice
# Output: invoice-2024-01-15--14-30-22.png
```

## For AI Agents

This wrapper is designed to be LLM-friendly:

* **Predictable output** - Prints created filename to stdout
* **No prompts** - Runs autonomously when `SCANIMAGE_DEVICE` is set
* **Resilient** - Retry logic handles transient scanner errors
* **Simple flags** - `-s adf`, `-m Gray`, `-f pdf` are easy to generate
* **Pass-through** - Extra args after `--` go directly to scanimage

```bash
# AI agent can reliably:
scanimage-handy -b document -f pdf -s adf
# Outputs: document-2024-01-15--14-30-22.pdf
```

## Documentation

See [scanimage-handy.README.md](scanimage-handy.README.md) for complete reference.

## Contributing

Feature requests and bug reports welcome via GitHub Issues.
