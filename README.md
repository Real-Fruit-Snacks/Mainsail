<div align="center">

# Mainsail

A BusyBox-style multi-call binary written in Python.

[![Version](https://img.shields.io/github/v/release/Real-Fruit-Snacks/mainsail?color=89b4fa)](https://github.com/Real-Fruit-Snacks/mainsail/releases)
[![License](https://img.shields.io/github/license/Real-Fruit-Snacks/mainsail?color=cba6f7)](LICENSE)

</div>

## Overview

Mainsail is the **reference implementation** of our multi-call shell toolkits. It provides 84 native POSIX-style utilities (including `jq`, `http`, `dig`, and `nc`), all running off the standard Python library with zero runtime dependencies. 

Because it is written in Python, Mainsail is exceptionally easy to read, extend, and fork. The applet roster, flag conventions, exit codes, and stdout byte semantics defined in this repository are what all sibling projects (`jib` in Rust, `topsail` in Go, `staysail` in Zig, `moonraker` in Lua, and `rill` in NASM) verify against.

Mainsail can be compiled into a ~6 MB statically-linked Nuitka executable or bundled into a ~148 KB portable `.pyz` zipapp, meaning it can run gracefully on nearly any system that supports Python 3.8+ (even locked-down environments, jailbroken routers, and esoteric architectures).

## Key Features

- **Reference Implementation**: Defines the strict `NAME`, `ALIASES`, `HELP`, and `main` API contract used across the entire fleet of our multi-call binaries.
- **84 Native Applets**: Broad coverage spanning file operations, text formatting, JSON processing (with a `jq` clone supporting 47 builtins), network debugging, and hashing.
- **Portable Formats**: Distribute via fully compiled executables (via Nuitka) or a standard `.pyz` zipapp (via zipapp) that executes natively via the standard Python interpreter.
- **Auto-Discovery**: Simply drop a new Python script into `mainsail/applets/` adhering to the four-symbol contract, and Mainsail automatically discovers and registers it.
- **Comprehensive Lifecycle Management**: Built-in self-updaters, alias installation features, and shell completion generation (`bash`, `zsh`, `fish`, `powershell`).

## Getting Started / Installation

### Prerequisites

- Python 3.10+ (if running from source)

### Installation

**From Precompiled Releases (No Python Required):**

```bash
curl -LO https://github.com/Real-Fruit-Snacks/mainsail/releases/latest/download/mainsail-linux-x64
chmod +x mainsail-linux-x64
./mainsail-linux-x64 --list
```

**From Source / pipx:**

```bash
# Clone and install locally
git clone https://github.com/Real-Fruit-Snacks/mainsail.git
cd mainsail
pip install -e .

# Or, use pipx for isolated global install
pipx install git+https://github.com/Real-Fruit-Snacks/mainsail.git
```

## Usage

### Symlinking and Initialization

You can execute Mainsail as a command-line wrapper (`mainsail ls -la`) or via symlink-driven multi-call dispatch.

```bash
# Wire up your shell: generate symlinks and shell completions
mainsail install-aliases ~/.local/bin
mainsail completions bash | sudo tee /etc/bash_completion.d/mainsail

# Self-update from the latest release
mainsail update
```

### Portable Zipapp Execution

The `.pyz` artifact can be directly transferred to and executed on any system with a Python interpreter.

```bash
scp mainsail.pyz host:/tmp/
ssh host 'python3 /tmp/mainsail.pyz find /var/log -mtime -7'
```

### Custom Builds

Build exactly the subset of applets you need using the custom `build.py` orchestrator:

```bash
# Build the portable zipapp
python build.py --pyz

# Build a "slim" binary containing only POSIX coreutils (no archives or hashes)
python build.py --preset slim

# Pick exactly the applets you want for a micro-deployment
python build.py --pyz --applets ls,cat,grep,sed,awk
```

## Architecture / File Structure

```text
mainsail/
├── __main__.py          # Entrypoint passing argv to cli.py
├── cli.py               # Handles symlink multi-call vs explicit subcommand
├── registry.py          # Auto-discovery using pkgutil to load applets
├── usage.py             # Generates --help and documentation formats
├── common.py            # Shared utility functions
├── applets/             # The 84 individual utility implementations
│   ├── ls.py
│   ├── cat.py
│   └── (82 more...)
├── build.py             # Build script to generate Nuitka executables and zipapps
├── scripts/             # Development scripts (stress testing, etc.)
└── tests/               # 402 unit tests verifying core logic
```

Mainsail utilizes a robust auto-discovery system. It loads modules dynamically using `pkgutil.iter_modules` and reads the `NAME`, `ALIASES`, `HELP`, and `main` parameters from each Python file within the `applets` directory.

## Contributing

Review the contribution guidelines in [CONTRIBUTING.md](CONTRIBUTING.md) before submitting pull requests.

## License

This project is licensed under the MIT License. See [LICENSE](LICENSE) for details.
