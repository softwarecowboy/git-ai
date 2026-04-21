# Windows Fix & Development Instructions

This directory contains instructions and resources for maintaining and fixing Windows-specific issues in `git-ai`.

## Current Status
Windows support (non-WSL) is currently **experimental**. The primary integration method is via a PowerShell-based installation script and a background daemon.

## Running Windows Tests
To run tests specifically designed for the Windows environment:

```powershell
# Run the Windows install script integration tests
cargo test --test windows_install_script

# Run integration tests in wrapper mode (common on Windows)
cargo test --test integration -- --filter wrapper
```

## Recent Fixes

### 1. POSIX Utility Compatibility in Integration Tests (April 2026)
**Issue:** `bash_tool_provenance` integration tests were failing on Windows because they relied on Unix-style utilities (`sh`, `touch`, `cp`, `sed`, etc.) not found in the standard Windows PATH.

**Fix:** Updated the `run_bash` helper in `tests\integration\bash_tool_provenance.rs` to dynamically locate the Git installation on Windows and inject Git's `bin` and `usr\bin` directories into the child process `PATH`. This allows tests to execute Unix-style shell commands and pipelines natively on Windows without requiring WSL or manual environment configuration.

## Performance Suggestions & Bottlenecks

Windows tests currently take significantly longer (~6 hours) than Unix tests due to several platform-specific factors:

### 1. Process Spawning Costs
Windows `Command::new` is far more expensive than Unix `fork`. Since `git-ai` tests spawn a high volume of `git` and `git-ai` sub-processes, this overhead compounds.
- **Suggestion:** Cache resolved paths for tools like `sh` and `git` globally (using `OnceLock`) instead of re-resolving them (e.g., via `where.exe`) in every test helper.

### 2. NTFS & Anti-Malware Latency
The "stat-diff" mechanism used in provenance tests is sensitive to NTFS metadata update speeds. Windows Defender or EDR tools often perform synchronous scans of the temporary `.git` folders created during tests.
- **Suggestion:** Exclude the `std::env::temp_dir()` or the specific project build directory from real-time anti-virus scanning during development.
- **Suggestion:** Run tests on a RAM Disk (e.g., ImDisk) to bypass disk I/O and NTFS metadata overhead.

### 3. Mandatory File Locking
Unlike Unix's advisory locking, Windows mandatory locking can cause tests to fail if a background daemon or Git process is still closing a handle to the SQLite DB or a Git Note.
- **Suggestion:** Ensure the `DaemonProcess::shutdown` logic is robust and that tests explicitly wait for daemon termination before cleaning up repo directories.

### 4. POSIX Emulation (MSYS2)
Running `sh -c` inside a Rust test introduces an extra layer of indirection (MinGW/MSYS2 layer).
- **Suggestion:** For simple file operations (`touch`, `mkdir`, `cp`), prefer using Rust's `std::fs` within the tests unless the goal is specifically to test the *provenance* of an external bash command.

## Common Windows Issues & Fixes

### 1. Daemon Log Location
The daemon logs on Windows are stored in the user's home directory:
`$HOME\.git-ai\internal\daemon\logs\<PID>.log`

If the daemon fails to start, check the `daemon.pid.json` file in the same directory to identify the active PID.

### 2. `git ai upgrade` Limitation
On Windows, `git ai upgrade` is explicitly disabled to prevent file lock issues during binary replacement. Users must use `git-ai upgrade` (direct binary call) or re-run the `install.ps1` script.

### 3. PowerShell Installation Script
The `install.ps1` script at the root handles:
- Downloading/Copying the `git-ai` binary.
- Setting up the `.git-ai/bin` directory.
- Configuring the Git wrapper.
- Updating the User `PATH` (optional).

When modifying `install.ps1`, ensure that it correctly handles existing daemon processes by stopping them before attempting to replace the binary.

### 4. Path Handling
Always use `PathBuf` and avoid hardcoded `/` or `\` where possible. Be aware that Git on Windows often expects forward slashes in config files but backslashes in shell commands.

## Troubleshooting
- **File Locks:** If `cargo build` fails on Windows, ensure no `git-ai` or `git` (wrapper) processes are running in the background.
- **Process Cleanup:** You can use the following snippet to kill orphaned `git-ai` processes:
  ```powershell
  Get-CimInstance Win32_Process | Where-Object { $_.Name -eq "git-ai.exe" } | Stop-Process -Force
  ```
