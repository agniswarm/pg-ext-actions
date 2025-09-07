# pg-ext-actions

A custom Github actions for testing PostgreSQL extensions with cross-platform support for Windows, macOS, and Linux.

## Cross-Platform Support

This repository now supports all three major operating systems:

- **Linux**: Uses `apt-get` package manager (Ubuntu/Debian)
- **macOS**: Uses Homebrew package manager
- **Windows**: Uses Chocolatey package manager

The actions automatically detect the operating system and use the appropriate installation methods and tools for each platform.

## Actions

### `pg-setup`

Install and run specified PostgreSQL version on any supported platform.
Arguments:

- `version`: major PostgreSQL version, required;
- `install-contrib`: whether to install postgres-contrib package; `'true'` for yes, anything else is considered a no, optional.

**Platform-specific behavior:**

- **Linux**: Installs PostgreSQL via apt repository
- **macOS**: Installs PostgreSQL via Homebrew
- **Windows**: Installs PostgreSQL via Chocolatey

### `install-dependency`

Clone, build and install listed PostgreSQL extensions.
Arguments:

- `repository`: a space separated list of repositories of extensions to be installed, required;
- `host`: git platform hostname, optional; GitHub (github.com) is used by default;
- `auth-token`: authentication token (e.g. GitHub personal access token), optional.

**Platform-specific behavior:**

- **Linux/macOS**: Uses `make` with `sudo` for installation
- **Windows**: Uses `mingw32-make` or `make` without sudo

### `build-check`

Build and install extension and run regression tests. In case of test failure the `regression.diff` is printed out and error code is returned.
Arguments:

- `working-directory`: directory to run script in, optional; by default runs script in current directory.

**Platform-specific behavior:**

- **Linux/macOS**: Uses `make` with `sudo` for installation
- **Windows**: Uses `mingw32-make` or `make` without sudo

### `update-check`

Run update script from the version specified in control file in the main branch (e.g. `master` or `main`) to the current version.
Arguments:

- `main-branch`: main branch to update from, optional; when not specified the default branch is used;
- `working-directory`: directory to run script in, optional; by default runs script in current directory.

**Platform-specific behavior:**

- **Linux/macOS**: Uses standard `psql` command
- **Windows**: Attempts to locate `psql` in common installation paths

## Example for Github Actions

### Cross-Platform Testing

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
          - os: ubuntu-latest
            pg: 13
          - os: macos-latest
            pg: 14
          - os: windows-latest
            pg: 14
    name: PostgreSQL ${{ matrix.pg }} on ${{ matrix.os }}
    runs-on: ${{ matrix.os }}
    steps:
      # Install PostgreSQL
      - uses: adjust/pg-ext-actions/pg-setup@master
        with:
          version: ${{ matrix.pg }}
          install-contrib: 'true'

      # Install dependencies
      - uses: adjust/pg-ext-actions/install-dependency@master
        with:
          repository: <account>/<repo>
          auth-token: ${{ secrets.SECRET_TOKEN }}

      # Clone and build extension, run tests
      - uses: actions/checkout@v2
      - uses: adjust/pg-ext-actions/build-check@master
```

### Linux-Only Testing (Legacy)

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
        pg: [14, 13, 12, 11, 10, 9.6]
    name: PostgreSQL ${{ matrix.pg }}
    runs-on: ubuntu-latest
    steps:
      # Install PostgreSQL
      - uses: adjust/pg-ext-actions/pg-setup@master
        with:
          version: ${{ matrix.pg }}
          install-contrib: 'true'

      # Install dependencies
      - uses: adjust/pg-ext-actions/install-dependency@master
        with:
          repository: <account>/<repo>
          auth-token: ${{ secrets.SECRET_TOKEN }}

      # Clone and build extension, run tests
      - uses: actions/checkout@v2
      - uses: adjust/pg-ext-actions/build-check@master
```

## Requirements

### Linux

- Ubuntu/Debian-based system
- `sudo` access for package installation
- `curl` and `git` installed

### macOS

- Homebrew package manager (will be installed automatically if not present)
- `git` installed

### Windows

- Chocolatey package manager (will be installed automatically if not present)
- Git for Windows
- PowerShell execution policy set to allow script execution
