# scanimage-handy Reference

## Synopsis

```
scanimage-handy [OPTIONS] [-- SCANIMAGE_ARGS...]
```

## Description

`scanimage-handy` wraps the SANE `scanimage` command with user-friendly defaults,
automatic file naming, retry logic, and simplified source selection.

## Options

| Option | Default | Description |
|--------|---------|-------------|
| `-b, --basename NAME` | `scanimage` | Base name for output file |
| `-f, --format FORMAT` | `png` | Output format: pnm, tiff, png, jpeg, pdf |
| `-m, --mode MODE` | `Color` | Scan mode: Color, Gray |
| `-r, --resolution DPI` | `600` | Scan resolution in DPI |
| `--date POSITION` | `suffix` | Timestamp position: prefix, suffix, no |
| `-v, --verbose` | off | Enable verbose output |
| `-d, --device-name DEV` | env | SANE device (or use `$SCANIMAGE_DEVICE`) |
| `-S, --select-scanner` | off | Interactive scanner selection |
| `-i, --icc-profile FILE` | none | ICC profile for TIFF files |
| `-o, --output-file PATH` | auto | Explicit output filename |
| `-p, --progress` | off | Show progress messages |
| `-A, --all-options` | off | List backend options (no scan) |
| `-V, --view CMD` | none | Viewer command (e.g., `viu`, `feh {}`) |
| `--retries N` | `3` | Retry attempts on failure |
| `--delay SECS` | `1` | Delay between retries |
| `-s, --source SRC` | `Flatbed` | Scan source (see aliases below) |
| `--batch[=FORMAT]` | off | Enable batch mode |
| `--batch-prompt` | off | Prompt before each page |

## Environment Variables

| Variable | Description |
|----------|-------------|
| `SCANIMAGE_DEVICE` | Default scanner device |
| `SCANIMAGE_FORMAT` | Default output format (default: png) |
| `SCANIMAGE_RESOLUTION` | Default resolution (default: 600) |

## Source Aliases

| Alias | Canonical Value |
|-------|-----------------|
| `flatbed`, `flat`, `bed` | `Flatbed` |
| `adf` | `ADF` |
| `duplex`, `adf duplex` | `ADF Duplex` |

## Batch Mode

Batch mode scans multiple pages to numbered files.

```bash
# Explicit batch with custom prefix
scanimage-handy --batch=document_%d.png

# Auto-enabled for ADF Duplex
scanimage-handy -s duplex
# Creates: scanimage-2024-01-15--14-30-22-%d.png
```

## ADF Duplex Scanning

ADF duplex scanning is dramatically simplified:

```bash
# scanimage-handy: just this
scanimage-handy -s duplex

# vs raw scanimage: 6 flags, exact strings required
scanimage -d 'epson2:libusb:001:003' --format=png --source='ADF Duplex' \
  --batch='doc_%d.png' --batch-double --batch-increment=1
```

When `-s duplex` is used, the script automatically:

1. Enables batch mode
2. Adds `--batch-double` flag
3. Adds `--batch-increment=1` flag

This produces correctly ordered pages for double-sided documents.

**Resilience:** If the scanner fails mid-batch (paper jam, communication error), raw `scanimage` exits and you must restart manually. `scanimage-handy` retries automatically up to `--retries` times (default: 3) with `--delay` seconds between attempts.

## Retry Logic

On scan failure, the script retries up to `--retries` times with `--delay` seconds between attempts. Configure with:

```bash
scanimage-handy --retries 5 --delay 2
```

## Viewer Integration

Preview scans immediately after completion:

```bash
# Simple viewer
scanimage-handy -V viu

# Viewer with placeholder
scanimage-handy -V "feh {}"
```

## Filename Generation

Default pattern: `{basename}-{timestamp}.{format}`

* Timestamp format: `YYYY-MM-DD--HH-MM-SS`
* If file exists, waits 1 second and regenerates
* Override with `-o filename.png`

Examples:

```
scanimage-2024-01-15--14-30-22.png  (default)
invoice-2024-01-15--14-30-22.pdf   (-b invoice -f pdf)
2024-01-15--14-30-22-scan.png      (--date prefix)
```

## Pass-through Arguments

Extra arguments after `--` are passed directly to scanimage:

```bash
scanimage-handy -- --brightness 50 --contrast 30
```

## Exit Codes

| Code | Meaning |
|------|---------|
| 0 | Success |
| 1 | Configuration error (no device) |
| N | scanimage exit code on failure |

## Examples

```bash
# Quick scan with all defaults
scanimage-handy

# High-quality archival scan
scanimage-handy -r 1200 -f tiff -b archive

# Office document batch
scanimage-handy -s adf -f pdf -m Gray -r 300 -b contract

# Interactive device selection
scanimage-handy -S

# Debug: see exact scanimage command
scanimage-handy -v
```

## See Also

* `scanimage(1)` - SANE scanner interface
* `sane-find-scanner(1)` - Find SANE devices
