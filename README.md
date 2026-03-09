# colorls (Go)

A **Go** port of [colorls](https://github.com/athityakumar/colorls): list directory contents with colors and icons. No Ruby required — a single binary, fast startup.

## Features

- **Colored output** — Files, directories, and file types get distinct colors
- **Icons** — Nerd Font / Font Awesome style icons (when your terminal supports them)
- **Long format** — `-l` with permissions, owner, group, size, date
- **Tree view** — `--tree` or `--tree=2` for directory depth
- **Git status** — `--gs` to show git status hints next to files
- **Sorting** — By name, time (`-t`), size (`-S`), extension (`-X`); directories first (`--sd`)
- **Combined flags** — Use `-lA`, `-la`, `-lA --sd` like the Ruby version

Same CLI and behavior as the original colorls; same config files in `~/.config/colorls/`.

---

## Prerequisites

- **Go 1.21 or later** (only needed to build from source)
- **Linux or macOS** (full support; Windows can build but with limited features)

---

## Installation

### Option A: Build from source (Linux and Mac)

1. **Clone the repo**
   ```bash
   git clone https://github.com/athityakumar/colorls-go.git
   cd colorls-go
   ```

2. **Build**
   ```bash
   go build -o colorls ./cmd/colorls
   ```

3. **Run from the project directory**
   ```bash
   ./colorls
   ./colorls -l
   ```

4. **(Optional) Install to your PATH**

   **Linux**
   ```bash
   sudo mv colorls /usr/local/bin/
   # or for your user only:
   mkdir -p ~/.local/bin
   mv colorls ~/.local/bin/
   # add to PATH in ~/.bashrc or ~/.profile: export PATH="$HOME/.local/bin:$PATH"
   ```

   **macOS**
   ```bash
   sudo mv colorls /usr/local/bin/
   # or with Homebrew's path:
   mkdir -p /opt/homebrew/bin   # Apple Silicon
   # or  mkdir -p /usr/local/bin  # Intel
   mv colorls /opt/homebrew/bin/
   ```

   Then run `colorls` from any directory.

### Option B: Download a release (when available)

From the [Releases](https://github.com/athityakumar/colorls-go/releases) page:

- **Linux**: Download the `colorls-go_*_linux_amd64.tar.gz` (or `arm64` for ARM), extract, and move the `colorls` binary to a directory on your PATH (e.g. `~/.local/bin` or `/usr/local/bin`).
- **macOS**: Download the `colorls-go_*_darwin_amd64.tar.gz` (Intel) or `darwin_arm64` (Apple Silicon), extract, and move the `colorls` binary to `/usr/local/bin` or `/opt/homebrew/bin`.

---

## Usage

```bash
colorls [OPTION]... [FILE]...
```

| Option | Description |
|--------|-------------|
| `-l`, `--long` | Long listing (permissions, owner, size, date) |
| `-a`, `--all` | Show hidden files (including `.` and `..`) |
| `-A`, `--almost-all` | Show hidden files but not `.` and `..` |
| `-1` | One file per line |
| `-t` | Sort by modification time (newest first) |
| `-S` | Sort by size (largest first) |
| `-r`, `--reverse` | Reverse sort order |
| `--sd`, `--sort-dirs` | Directories first |
| `--sf`, `--sort-files` | Files first |
| `--tree[=DEPTH]` | Tree view (default depth 3) |
| `--gs`, `--git-status` | Show git status per file |
| `--help` | Show help |
| `--version` | Show version |

**Examples**

```bash
colorls
colorls -l
colorls -lA --sd          # long, almost-all, directories first
colorls --tree=1
colorls -1
colorls --help
```

**Useful alias (works on Linux and Mac)**

Add to `~/.bashrc`, `~/.zshrc`, or your shell config:

```bash
alias ll="colorls -lA --sd"
```

Then run `ll` for a long, colored listing with directories first.

---

## Configuration

Same as the [Ruby colorls](https://github.com/athityakumar/colorls): copy the bundled YAML files into `~/.config/colorls/` and edit them.

1. **Create the config directory**
   ```bash
   mkdir -p ~/.config/colorls
   ```

2. **Copy and edit (example: colors)**
   ```bash
   # From the repo after cloning:
   cp internal/colorls/yaml/dark_colors.yaml ~/.config/colorls/
   # Edit ~/.config/colorls/dark_colors.yaml to change colors
   ```

   **Files you can override**
   - `dark_colors.yaml` / `light_colors.yaml` — color scheme
   - `files.yaml` — file-extension → icon
   - `file_aliases.yaml` — extension aliases
   - `folders.yaml` — folder name → icon
   - `folder_aliases.yaml` — folder aliases

3. **Light theme**
   ```bash
   colorls --light
   ```

---

## Building from source (all platforms)

```bash
git clone https://github.com/athityakumar/colorls-go.git
cd colorls-go
go build -o colorls ./cmd/colorls
```

- **Linux (64-bit)**  
  Same as above; the binary is a static executable for your architecture.

- **macOS (Intel or Apple Silicon)**  
  Same as above; Go will build for your current architecture.

- **Cross-compile (from Linux or Mac)**  
  ```bash
  GOOS=linux GOARCH=amd64 go build -o colorls-linux-amd64 ./cmd/colorls
  GOOS=darwin GOARCH=arm64 go build -o colorls-darwin-arm64 ./cmd/colorls
  ```

---

## Project layout

```
colorls-go/
├── cmd/colorls/main.go    # Entry point
├── internal/colorls/       # Core logic (flags, layout, git, colors, etc.)
├── go.mod
├── README.md
├── REFACTORING.md          # Ruby → Go mapping and verification
└── LICENSE
```

---

## License

MIT. See [LICENSE](LICENSE).  
Original [colorls](https://github.com/athityakumar/colorls) by [Athitya Kumar](https://github.com/athityakumar/); this Go port keeps the same CLI and behavior.
