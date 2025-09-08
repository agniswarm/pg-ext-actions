# pg-ext-actions

A comprehensive GitHub Actions collection for testing PostgreSQL extensions with cross-platform support for Windows, macOS, and Linux, including specific OS version targeting.

## 🏗️ Repository Structure

```text
pg-ext-actions/
├── bin/                          # Shell scripts (executables)
│   ├── pg-setup                  # PostgreSQL installation script
│   ├── build-check              # Extension build & test script
│   ├── install-dependency       # Extension dependency installer
│   └── update-check             # Extension version update checker
├── pg-setup/                    # GitHub Action for PostgreSQL setup
│   └── action.yml
├── build-check/                 # GitHub Action for building extensions
│   └── action.yml
├── install-dependency/          # GitHub Action for installing dependencies
│   └── action.yml
├── update-check/                # GitHub Action for update verification
│   └── action.yml
└── README.md                    # This file
```

## 🌍 Cross-Platform Support

### Supported Operating Systems

**Linux Distributions:**

- `ubuntu-22.04`, `ubuntu-24.04`
- `debian-11`, `debian-12`
- `centos-7`, `centos-8`
- `rhel-8`, `rhel-9`
- `fedora-38`, `fedora-39`
- `suse-15`, `suse-16`
- `alpine-3.18`, `alpine-3.19`

**macOS Versions:**

- `macos-12`, `macos-13`, `macos-14`, `macos-15`
- `darwin-21`, `darwin-22`, `darwin-23`

**Windows Versions:**

- `windows-2022`, `windows-2025`
- `windows-10`, `windows-11`
- `win-10`, `win-11`

### Package Managers

- **Linux**: Uses `apt-get` package manager (Ubuntu/Debian)
- **macOS**: Uses Homebrew package manager
- **Windows**: Uses Chocolatey package manager

The actions automatically detect the operating system and use the appropriate installation methods and tools for each platform.

## 🚀 Actions

### `pg-setup`

Install and run specified PostgreSQL version on any supported platform.

**Arguments:**

- `version`: PostgreSQL major version number (required)
- `install-contrib`: Install PostgreSQL contrib package - `'true'` for yes, anything else for no (optional, default: `'false'`)
- `operating-system`: Operating system and version to target (optional, default: `'ubuntu-22.04'`)

**Supported OS Values:**

- Linux: `ubuntu-22.04`, `ubuntu-24.04`, `debian-11`, `debian-12`, `centos-7`, `centos-8`, `rhel-8`, `rhel-9`, `fedora-38`, `fedora-39`, `suse-15`, `suse-16`, `alpine-3.18`, `alpine-3.19`
- macOS: `macos-12`, `macos-13`, `macos-14`, `macos-15`, `darwin-21`, `darwin-22`, `darwin-23`
- Windows: `windows-2022`, `windows-2025`, `windows-10`, `windows-11`, `win-10`, `win-11`
- Generic: `linux`, `macos`, `windows` (auto-detects specific version)

**Platform-specific behavior:**

- **Linux**: Installs PostgreSQL via apt repository with version-specific optimizations
- **macOS**: Installs PostgreSQL via Homebrew with proper development environment setup
- **Windows**: Installs PostgreSQL via Chocolatey with Windows-specific configurations

### `build-check`

Build, install, and test PostgreSQL extensions with comprehensive regression testing.

**⚠️ Prerequisites**: This action requires PostgreSQL to be installed first. Run `pg-setup` before using this action.

**Arguments:**

- `working-directory`: Directory to run script in (optional, default: current directory)
- `operating-system`: Operating system and version to target (optional, default: `'ubuntu-22.04'`)

**What it does:**

1. **Build**: Compiles the extension from source (`make`)
2. **Install**: Installs the extension to PostgreSQL (`make install`)
3. **Test**: Runs regression tests (`make installcheck`)

**Platform-specific behavior:**

- **Linux/macOS**: Uses `make` with `sudo` for installation, validates `pg_config` availability
- **Windows**: Uses `nmake` (preferred) or `mingw32-make`/`make` without sudo
- **macOS**: Automatically sets up PostgreSQL development environment (Homebrew paths, `PG_CONFIG`, etc.)

### `install-dependency`

Clone, build, and install PostgreSQL extension dependencies from Git repositories.

**Arguments:**

- `repository`: Space-separated list of Git repositories to install (required)
- `host`: Git platform hostname (optional, default: `'github.com'`)
- `auth-token`: Authentication token for private repositories (optional)
- `operating-system`: Operating system and version to target (optional, default: `'ubuntu-22.04'`)

**Platform-specific behavior:**

- **Linux/macOS**: Uses `make` with `sudo` for installation
- **Windows**: Uses `mingw32-make` or `make` without sudo

### `update-check`

Verify that extension update scripts work correctly by testing version upgrades.

**Arguments:**

- `working-directory`: Directory to run script in (optional, default: current directory)
- `main-branch`: Branch containing version to update from (optional, default: auto-detect)
- `operating-system`: Operating system and version to target (optional, default: `'ubuntu-22.04'`)

**What it does:**

1. Compares current extension version with main branch version
2. Tests upgrading from main branch version to current version
3. Validates that the update process works correctly

**Platform-specific behavior:**

- **Linux/macOS**: Uses standard `psql` command
- **Windows**: Attempts to locate `psql` in common installation paths

## 📋 Usage Examples

### Cross-Platform Testing with Specific OS Versions

```yaml
name: CI

on:
  push:
    branches: ['*']
  pull_request:
    branches: ['*']

jobs:
  test:
    strategy:
      fail-fast: false
      matrix:
        include:
          - os: ubuntu-latest
            pg: 14
            target-os: ubuntu-22.04
          - os: ubuntu-latest
            pg: 16
            target-os: ubuntu-24.04
          - os: macos-latest
            pg: 14
            target-os: macos-14
          - os: windows-latest
            pg: 14
            target-os: windows-2022
    name: PostgreSQL ${{ matrix.pg }} on ${{ matrix.target-os }}
    runs-on: ${{ matrix.os }}
    steps:
      # Install PostgreSQL
      - uses: agniswarm/pg-ext-actions/pg-setup@master
        with:
          version: ${{ matrix.pg }}
          install-contrib: 'true'
          operating-system: ${{ matrix.target-os }}

      # Install dependencies
      - uses: agniswarm/pg-ext-actions/install-dependency@master
        with:
          repository: <account>/<repo>
          auth-token: ${{ secrets.SECRET_TOKEN }}
          operating-system: ${{ matrix.target-os }}

      # Clone and build extension, run tests
      - uses: actions/checkout@v4
      - uses: agniswarm/pg-ext-actions/build-check@master
        with:
          operating-system: ${{ matrix.target-os }}
```

### Comprehensive OS Version Matrix

```yaml
name: Comprehensive Testing

on:
  push:
    branches: ['*']
  pull_request:
    branches: ['*']

jobs:
  test:
    strategy:
      fail-fast: false
      matrix:
        include:
          # Linux distributions
          - os: ubuntu-latest
            target-os: ubuntu-22.04
            pg: 16
          - os: ubuntu-latest
            target-os: ubuntu-24.04
            pg: 16
          - os: ubuntu-latest
            target-os: debian-12
            pg: 16
          # macOS versions
          - os: macos-latest
            target-os: macos-13
            pg: 16
          - os: macos-latest
            target-os: macos-14
            pg: 16
          # Windows versions
          - os: windows-latest
            target-os: windows-2022
            pg: 16
          - os: windows-latest
            target-os: windows-2025
            pg: 16
    name: PostgreSQL ${{ matrix.pg }} on ${{ matrix.target-os }}
    runs-on: ${{ matrix.os }}
    steps:
      # Install PostgreSQL
      - uses: agniswarm/pg-ext-actions/pg-setup@master
        with:
          version: ${{ matrix.pg }}
          install-contrib: 'true'
          operating-system: ${{ matrix.target-os }}

      # Clone and build extension, run tests
      - uses: actions/checkout@v4
      - uses: agniswarm/pg-ext-actions/build-check@master
        with:
          operating-system: ${{ matrix.target-os }}
```

### Simple Single-Platform Testing

```yaml
name: Simple Test

on:
  push:
    branches: ['main']

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      # Install PostgreSQL (uses default ubuntu-22.04)
      - uses: agniswarm/pg-ext-actions/pg-setup@master
        with:
          version: '16'
          install-contrib: 'true'

      # Clone and build extension, run tests (uses default ubuntu-22.04)
      - uses: actions/checkout@v4
      - uses: agniswarm/pg-ext-actions/build-check@master
```

## 🔄 Typical Workflow

1. **Setup PostgreSQL** (`pg-setup`)
   - Install PostgreSQL with specified version
   - Configure development environment
   - Set up authentication and permissions

2. **Install Dependencies** (`install-dependency`) - *Optional*
   - Clone and install required extension dependencies
   - Build dependencies for the target platform

3. **Build & Test** (`build-check`)
   - Compile your extension from source
   - Install extension into PostgreSQL
   - Run comprehensive regression tests

4. **Verify Updates** (`update-check`) - *Optional*
   - Test extension version upgrade scenarios
   - Validate update scripts work correctly

## 📋 Requirements

### Linux

- Ubuntu/Debian-based system (or compatible)
- `sudo` access for package installation
- `curl` and `git` installed
- PostgreSQL development packages (installed automatically)

### macOS

- Homebrew package manager (installed automatically if not present)
- `git` installed
- Xcode Command Line Tools (for compilation)

### Windows

- Chocolatey package manager (installed automatically if not present)
- Git for Windows
- PowerShell execution policy set to allow script execution
- Visual Studio Build Tools (for compilation)

## 🎯 Key Features

- **🔄 Cross-Platform**: Works on Linux, macOS, and Windows
- **🎯 OS Version Targeting**: Specify exact OS versions (e.g., `ubuntu-24.04`, `macos-14`)
- **🤖 CI-Aware**: Automatically detects CI environments and skips service management
- **🔧 Auto-Configuration**: Automatically sets up PostgreSQL development environment
- **🧪 Comprehensive Testing**: Build, install, and test extensions with regression testing
- **📦 Dependency Management**: Install extension dependencies from Git repositories
- **🔄 Update Validation**: Test extension version upgrade scenarios
- **🎨 Rich Logging**: Colored output with clear success/error indicators
- **⚡ Fast Setup**: One-command PostgreSQL installation and configuration

## 🤖 CI Environment Behavior

The actions automatically detect CI environments (GitHub Actions, GitLab CI, etc.) and adjust their behavior:

- **Service Management**: Skips starting/stopping PostgreSQL services in CI
- **Database Initialization**: Skips database creation in CI (assumes CI handles this)
- **Health Checks**: Skips PostgreSQL readiness checks in CI
- **Environment Setup**: Still configures all necessary environment variables and paths

This ensures the actions work seamlessly in CI environments where PostgreSQL is typically managed by the CI system itself.
