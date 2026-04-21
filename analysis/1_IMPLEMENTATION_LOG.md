# 1. Implementation Log: Phase 2 Optimizations

This document records the chronological development and technical changes implemented during Phase 2.

## **1.1. Real Git Discovery & PATH Sanitization**
**Files:** `tests/integration/repos/test_repo.rs`
- **Change:** Implemented a Windows-specific `find_real_git_by_probe` that uses `where.exe git` to find the actual `git.exe` on the system.
- **Change:** Enabled `PATH` sanitization for Windows. This dynamically removes any directory containing the `git-ai` wrapper from the process environment during tests.
- **Goal:** Ensure test isolation and prevent recursive "process storms."

## **1.2. Native Filesystem Utility (`sync_dir`)**
**Files:** `tests/integration/repos/test_repo.rs`, `tests/integration/rebase_benchmark.rs`
- **Change:** Added a native Rust `sync_dir` function that clears a destination and recursively copies from a source.
- **Change:** Replaced all `rsync` shell calls in benchmarks with `sync_dir`.
- **Goal:** Eliminate dependency on `rsync` and remove shell-spawning overhead.

## **1.3. Global Polling Latency Reduction**
**Files:** `src/commands/git_handlers.rs`, `src/authorship/git_ai_hooks.rs`, `tests/integration/bash_tool_benchmark.rs`, `tests/async_mode.rs`, `tests/daemon_mode.rs`
- **Change:** Reduced the fixed polling interval from **25ms to 10ms** across all components (hooks, commit stats, daemon checks).
- **Goal:** Aggressively reduce total "idle" time across the 3,000+ test suite.

## **1.4. Windows Defender AV Subcommand**
**Files:** `src/utils.rs`, `src/commands/debug.rs`
- **Change:** Added `exclude_path_from_windows_defender` (PowerShell-based).
- **Change:** Exposed this via a new debug command: `git-ai debug exclude-temp-from-av`.
- **Goal:** Allow developers to programmatically speed up tests by excluding temporary I/O from real-time AV scanning.

## **1.5. Restored Performance Logging**
**Files:** `src/observability/wrapper_performance_targets.rs`
- **Change:** Fixed a regression where structured performance JSON (`[git-ai (perf-json)]`) was not being emitted to stdout when `GIT_AI_DEBUG_PERFORMANCE >= 2`.
- **Goal:** Restore functionality for automated performance benchmarking and tests.
