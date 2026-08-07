# Vouru – Void User Repository Helper

Vouru is a Bash script to build and install packages from `void-packages` and the community **VUR** overlay.

- Clones `void-packages`, runs `binary-bootstrap`
- Searches, compiles and installs packages
- Detects whether a package is in the official repos or only available as a template
- Heavy operations run with a spinner and log to `~/.vouru.log`

## Requirements

- `xtools`, `git`, `curl` (installed by the automatic installer)
- `doas` or `sudo` for privileged operations

## Installation

```bash
# Rolling (recommended: always the latest version)
curl -sSL https://github.com/javiercplus/vouru/raw/refs/heads/main/vouru-install-rolling | bash

# Manual
curl -sSL -o vouru https://github.com/javiercplus/vouru/raw/refs/heads/main/vouru
chmod +x vouru && sudo mv vouru /usr/local/bin/

# Without installing (from the web)
curl -sSL https://github.com/javiercplus/vouru/raw/refs/heads/main/vouru | bash -s -- search firefox
```

> The `-s --` after `bash` is required: `-s` reads the script from stdin and `--` prevents bash from interpreting vouru's own options (`-i`, `-s`, `-c`...) as its own.

## Commands

| Command | Description |
|---------|-------------|
| `vouru add <git-url>` | Clones a repo and copies its templates to `srcpkgs` (supports subdirectories and a template at the root; skips existing ones) |
| `vouru search <term>` | Searches for packages in `srcpkgs` matching `<term>` |
| `vouru install <term>` | Installs a package: choose between `[repo]` (xbps-install, binary) or `[source]` (build with xbps-src) |
| `vouru remove <pkg>` | Removes a package with `xbps-remove -y -o` (includes orphans) |
| `vouru update` | Updates the repo, `xbps-src` and the system |
| `vouru clean` | Deletes `void-packages` and cleans the cache with `xbps-remove -O` |
| `vouru set-repo [path]` | Changes the repo location (saved in `~/.voururc`) |

**Aliases:** `-s` (search), `-i` (install), `-r` (remove), `-c` (clean), `-a` (add)

When installing a package, vouru shows its template before building it. A warning is shown before the template letting you know that pressing `q` closes the template view and returns to the prompt.

## Examples

```bash
vouru add https://codeberg.org/Neko-Void/zig-nk.git
vouru search firefox
vouru install vlc
vouru remove cmuspp-void
vouru update
vouru clean
```

## Configuration

- **Repo:** `~/.voururc` or `vouru set-repo <path>`
- **Log:** `~/.vouru.log` (check if cloning/bootstrap fails)

## Troubleshooting

| Problem | Solution |
|---------|----------|
| `'xtools' is not installed` | `sudo xbps-install -S xtools` |
| `Neither 'doas' nor 'sudo' found` | Install one of them |
| Compilation fails | Check the full output and the `template` dependencies |
| `binary-bootstrap` fails | Run it manually in `void-packages` and check the error |
| Spinner hangs | `Ctrl+C`, then check `~/.vouru.log` |

## License

BSD 3-Clause
