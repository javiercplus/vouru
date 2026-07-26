# Vouru – Void User Repository Helper

**Vouru** (formerly `vuru`) is a Bash script that streamlines building and installing packages from the **Void Linux** source repository (`void-packages`) and the community **VUR** (Void User Repository) overlay.

It automates:

- Cloning `void-packages` (with `--depth 1`)
- Running `binary-bootstrap`
- Searching, reviewing, compiling, and installing packages
- Checking if a package is already installed or available in the official repos
- Offering a choice between direct `xbps-install` (from official repos) or source compilation
- **Cleaning** the entire repository to free up space
- **Setting a custom repository location**

All heavy operations (clone, bootstrap, updates) are shown with a **spinner** and logged to `~/.vouru.log` for later inspection.

<img width="1280" height="768" alt="image" src="https://github.com/user-attachments/assets/fa7976fe-fa06-44ec-93d1-fda7c18308e8" />

---

## Requirements

- **`xtools`** – for `xbps-query -Rs` and other XBPS utilities  
  **Installed automatically by the rolling installer.**
- **`git`** – to clone repositories
- **`curl`** – to fetch the latest script (for the rolling installer)
- **`doas`** or **`sudo`** – for privileged operations (only needed when installing `xtools` or updating the system)

All dependencies are checked automatically when you run the script.

---

## Quick Start (Online One‑Liner)

You can run Vouru directly from the web without any installation:

```bash
curl -sSL https://github.com/javiercplus/vouru/raw/refs/heads/main/vouru | bash -s search firefox
```

Replace `search firefox` with any command shown below.

> **Important:** The `-s` after `bash` is required to pass arguments to the script.

---

## Rolling Installation (Recommended)

The **rolling installer** creates a tiny launcher in `~/.local/bin/vouru` that always fetches the latest version of Vouru from GitHub. This way you never have to update manually.

```bash
curl -sSL https://github.com/javiercplus/vouru/raw/refs/heads/main/vouru-install-rolling | bash
```

What it does:

- Checks for `xtools` and installs it if missing (using `doas`/`sudo`/`su`).
- Creates `~/.local/bin/vouru` (the launcher).
- Makes the launcher executable.
- **Warns you** if `~/.local/bin` is not in your `PATH` and tells you how to add it (for both `bash` and `zsh`).

After installation, you can use `vouru` as a normal command.

---

## Manual Installation (Local)

If you prefer a static local copy:

```bash
curl -sSL -o vouru https://github.com/javiercplus/vouru/raw/refs/heads/main/vouru
chmod +x vouru
sudo mv vouru /usr/local/bin/   # optional, to have it in your PATH
```

Now you can run it as `vouru <command>`.

---

## Commands

| Command                    | Description |
|----------------------------|-------------|
| `vouru add <git-url>`       | Clone a git repository and add its template(s) to `srcpkgs`. Detects `template` files both in subdirectories (standard VUR layout) and at the repository root (extracting the package name from `pkgname=`). Skips any package that already exists. |
| `vouru search <term>`      | Search for packages in `srcpkgs` that match `<term>` (case‑insensitive, partial match). Lists all found packages with their descriptions. Does **not** compile or install. |
| `vouru install <term>`     | Look for `<term>`; if multiple matches, ask which one to install. Then: <br> • Check if already installed → offer to reinstall <br> • Check if available in official repos → offer to install directly (fast) <br> • Otherwise, show the `template` file, ask for confirmation, compile with `xbps-src pkg`, and install. |
| `vouru update`             | Update the `void-packages` repo (`git pull`), run `bootstrap-update`, run `update-sys`, and finally update the whole system with `xbps-install -Su`. |
| `vouru clean`              | Remove the entire `void-packages` repository (including all compiled packages and source) to free up disk space. Asks for confirmation before deleting. |
| `vouru set-repo <path>`    | Change the location of the `void-packages` repository. If no path is given, it prompts interactively. The setting is saved in `~/.voururc`. |
| `vouru -h` or `--help`     | Show this help message. |
| `vouru` (no arguments)     | Same as `--help`. |

**Aliases:**  
- `search` ↔ `-s`  
- `install` ↔ `-i`  
- `clean` ↔ `-c`  
- `add` ↔ `-a`

---

## Examples

```bash
# Search for packages containing "firefox"
vouru search firefox

# Search using alias
vouru -s libva

# Install VLC (will check repo availability and ask before compiling)
vouru install vlc

# Install using alias
vouru -i mpv

# Full system update (repo + xbps-src + system)
vouru update

# Clean the repository
vouru clean

# Set a custom repository location
vouru set-repo /home/user/my-void-packages

# Add templates from a VUR repository (e.g. zig-nk from codeberg)
vouru add https://codeberg.org/Neko-Void/zig-nk.git

# Run online without installing
curl -sSL https://github.com/javiercplus/vouru/raw/refs/heads/main/vouru | bash -s search firefox
```

---

## How It Works (Under the Hood)

1. **Dependency check** – ensures `git`, `xtools` and a privilege tool (`doas`/`sudo`) are present.
2. **Repository setup** – if the repository directory (as configured in `~/.voururc` or the default `~/.local/share/pkgs/void-packages`) does not exist, it:
   - Asks the user for a custom path (or uses the default).
   - Clones `void-packages` (with `--depth 1`).
   - Runs `./xbps-src binary-bootstrap` (muted, logged to `~/.vouru.log`).
   - Clones the VUR repo and copies `core/*` and `extra/*` into `srcpkgs/`.
3. **Subsequent runs** – if the repository already exists, Vouru skips cloning and bootstrapping, and does **not** clone VUR again (to avoid unnecessary network traffic).
4. **Search** – uses `find` on `srcpkgs/` to list matching directories and reads their `short_desc`.
5. **Add** (`vouru add <git-url>`) – clones the given repository, scans for `template` files up to 4 levels deep, and copies each template's directory into `srcpkgs/`. If a template is at the repository root, the package name is read from the `pkgname=` field; otherwise, the subdirectory name is used. Already existing packages are skipped.
6. **Installation flow**:
   - If the package is **already installed**, you are asked whether to rebuild/reinstall.
   - If the package is **available in the official repositories** (`xbps-query -Rs`), you are offered to install it directly via `xbps-install` (skipping compilation).
   - If you choose compilation, the `template` file is shown with `less` (or `cat`), and you confirm before proceeding.
   - Compilation runs with `./xbps-src pkg` (output is shown live).
   - The resulting package is installed with `xbps-install --repository hostdir/binpkgs`.
6. **Logging** – all muted operations (clone, bootstrap, git pull, update-sys) are appended to `~/.vouru.log` with timestamps.

---

## Log File

All background output is written to:

```
~/.vouru.log
```

Check this file if you encounter errors during cloning, bootstrapping, or updating.

---

## Configuration

Vouru stores its repository path in `~/.voururc`. You can edit this file manually or use `vouru set-repo <path>` to change it.

---

## Troubleshooting

| Issue | Solution |
|-------|----------|
| `Error: 'xtools' is not installed.` | The rolling installer installs it automatically. If using the standalone script, install it with `sudo xbps-install -S xtools`. |
| `Error: Neither 'doas' nor 'sudo' found.` | Install one of them. On Void, `sudo xbps-install -S doas` (or `sudo`). |
| Compilation fails | Check the full output (compilation is **not** muted). Look for missing dependencies in the `template` file. |
| `binary-bootstrap` fails | Run `./xbps-src binary-bootstrap` manually in your `void-packages` directory and inspect the error. Ensure you have internet access and the required system libraries. |
| Spinner keeps spinning | Press `Ctrl+C` to abort, then check `~/.vouru.log` for the exact command that hung. |
| `~/.local/bin` not in PATH | The installer warns you. Add `export PATH="$HOME/.local/bin:$PATH"` to your `~/.bashrc` or `~/.zshrc`. |

---

## License

This script is released under the **BSD 3 CLAUSE License**. Feel free to use, modify, and distribute it.

---

## Contributing

Issues and pull requests are welcome at:  
[https://github.com/javiercplus/vouru](https://github.com/javiercplus/vouru)
