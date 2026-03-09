# Ruby to Go Refactoring: colorls

This document describes the full refactoring of the **colorls** Ruby codebase into Go, with file-by-file mapping and verification notes.

## Objective

Produce a Go implementation that is operationally indistinguishable from the Ruby version: same CLI, I/O, config, exit codes, and behavior, without requiring a Ruby runtime.

---

## File-by-File Mapping

| Ruby source | Go implementation | Notes |
|-------------|-------------------|--------|
| `exe/colorls` | `cmd/colorls/main.go` | Entry point: parses args, calls `Flags.Process()`, exits with code. Ruby workaround for `Dir.pwd` and `$loading` omitted (not needed in Go). |
| `lib/colorls.rb` | (no single file) | Require list is replaced by package imports in `internal/colorls/*.go`. |
| `lib/colorls/version.rb` | `internal/colorls/version.go` | `VERSION = '1.5.0'` → `const VERSION = "1.5.0"`. |
| `lib/colorls/yaml.rb` | `internal/colorls/yaml.go` + `yaml_embed.go` | YAML loader: bundled path + `~/.config/colorls/` merge; `load(aliase: true)` supported. Bundled YAMLs embedded via `embed`. |
| `lib/colorls/core.rb` (module + Core class) | `internal/colorls/core.go`, `core_globals.go` | Module-level `file_encoding` / `screen_width` → `core_globals.go` + `terminal.go`. Core class → `Core` struct and methods: `ls_dir`, `ls_files`, `display_report`, layouts, sorting, grouping, long format, tree, git, colors, icons. |
| `lib/colorls/fileinfo.rb` | `internal/colorls/fileinfo.go`, `fileinfo_unix.go`, `fileinfo_other.go` | `FileInfo` with stat, owner/group (cached), symlink handling; platform-specific uid/gid/nlink/ino/mode via build tags. |
| `lib/colorls/flags.rb` | `internal/colorls/flags.go` | Option parsing (all long/short options), `default_opts`, `process_args`, `group_files_and_directories`, help/version, color scheme loading, `--tree` + `--all` → `--almost-all` fix. |
| `lib/colorls/layout.rb` | `internal/colorls/layout.go` | `Layout`, `VerticalLayout`, `HorizontalLayout`, `SingleColumnLayout`; chunk size binary search and `each_line` semantics. |
| `lib/colorls/git.rb` | `internal/colorls/git.go` | `Git.status(repo_path)` → `GitStatus`; `colored_status_symbols` → `ColoredStatusSymbols`; `git -C ... rev-parse --show-prefix` and `status --porcelain -z`, including rename skip. |
| `lib/colorls/monkeys.rb` | `internal/colorls/colors.go` | `String#colorize` / `String#bright` → `Colorize`, `Bright` with ANSI SGR; color names mapped to 256-color codes. |
| (Rainbow, filesize, etc.) | `internal/colorls/colors.go`, `filesize.go` | Color output and human-readable size formatting. |
| (unicode-display_width) | `internal/colorls/displaywidth.go` | `DisplayWidth(s)` using `golang.org/x/text/width`. |
| (clocale) | `internal/colorls/core.go`, `flags.go` | Locale-aware sort via `golang.org/x/text/collate` (language.Und). |
| (addressable) | `internal/colorls/core.go` | `make_link` uses `file://` + `filepath.Abs`. |
| (io/console size) | `internal/colorls/terminal.go` | `golang.org/x/term` for terminal width. |
| `lib/yaml/*.yaml` | `internal/colorls/yaml/*.yaml` (embedded) | Same files copied; loaded via embed + optional `~/.config/colorls/` override. |

---

## Preserved Behavior

- **CLI**: Same options and usage (`colorls [OPTION]... [FILE]...`); `--help`, `--version`; defaults (e.g. TTY → vertical, non-TTY → one-per-line).
- **Input/Output**: Same listing formats (vertical, horizontal, long, single-column, tree); same report format (short/long).
- **Config**: Same YAML files and merge order (bundled then `~/.config/colorls/`); same keys (colors, files, folders, aliases).
- **Environment**: Locale used implicitly via collate (Go default); no extra env vars added.
- **Exit codes**: 0 on success; 2 on parse error or missing path / system error (per Ruby).
- **Logging**: Errors to stderr with same messages (e.g. "colorls: Specified path 'X' doesn't exist."); git warning on failure.
- **Paths**: Bundled YAML under module; user config `~/.config/colorls/`.
- **Network**: Only git subprocess (same `git -C` usage).
- **Sorting**: Name (locale-aware), time, size, extension; reverse; group dirs/files first.

---

## Verification (Behavioral Parity)

1. **Build**  
   - From repo root: `go build -o colorls ./cmd/colorls`  
   - No Ruby runtime required.

2. **CLI**  
   - `./colorls --help` → same option list and examples.  
   - `./colorls --version` → `1.5.0`.  
   - `./colorls` → list current directory (`.`).  
   - `./colorls -l`, `-la`, `-1`, `-x`, `-C`, `--tree`, `--tree=1`, `--sd`, `--sf`, `-t`, `-S`, `-X`, `-U`, `-r`, `-a`, `-A`, `-d`, `-f`, `--gs`, `-i`, `--report`, `--indicator-style`, `--format`, `--without-icons`, `-o`, `-g`, `-G`, `-L`, `--no-hardlinks`, `--non-human-readable`, `--color=never`, `--light`, `--dark`, `--hyperlink` → same semantics as Ruby.

3. **Output**  
   - Compare `colorls` vs `./colorls` in the same directory (same flags).  
   - Compare `colorls -l` vs `./colorls -l` (long format: mode, user, group, size, time, name).  
   - Compare `colorls --tree=1` vs `./colorls --tree=1` (tree depth).  
   - Compare report: `colorls --report=short` vs `./colorls --report=short`.

4. **Errors**  
   - `./colorls does-not-exist` → stderr message, exit 2.  
   - Invalid option → stderr message, exit 2.

5. **Config**  
   - Copy YAML from Ruby gem’s `yaml/` to `~/.config/colorls/` and change a color/icon → Go build should use the same override.

6. **Git**  
   - In a git repo, `./colorls --gs` should show the same status hints as Ruby.

Recommended: run the Ruby test script (`test/checks` or equivalent) with both `colorls` (Ruby) and `./colorls` (Go) and compare exit codes and output where applicable.

---

## Dependencies

- **Go**: 1.21+  
- **Modules**: `golang.org/x/sys`, `golang.org/x/text`, `gopkg.in/yaml.v3`  
- **Build tags**: `unix` for uid/gid/nlink/ino/mode (see `fileinfo_unix.go` / `fileinfo_other.go`).

---

## Notes

- **Encoding**: Ruby’s `file_encoding` (UTF-8 on Windows when `nul` exists) is reflected in `FileEncodingUTF8()`; directory reads use the same idea.  
- **Rainbow**: Replaced by direct ANSI SGR in `colors.go`; `--color=never` disables coloring.  
- **Monkeys**: No global String extension; all color helpers are in package `colorls`.  
- **Icons**: Same YAML and alias resolution; `\uXXXX` in icons left as-is in this pass (decode can be added for full parity).
