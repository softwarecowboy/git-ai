# Project Handover: Windows Optimization & Stability

## 1. Project Context
- **Project:** `git-ai` (Rust Git extension for AI attribution).
- **Environment:** Windows 10 (`win32`), non-WSL.
- **Goal:** Reach a 20% runtime reduction for the test suite.
- **Baseline:** 21,850s (~6.07 hours).
- **Target:** < 17,480s (~4.85 hours).

## 2. Completed Fixes (Stability)
- **POSIX Utility Compatibility:** Fixed `bash_tool_provenance.rs` failures by dynamically injecting Git's `bin` and `usr\bin` into the child process `PATH`.
- **Current Test Status:** **100% PASS** (3,083 tests).

## 3. Stage 1 Optimizations (Implemented)
The following changes were applied to reduce Windows-specific overhead:
- **Global Path Caching:** Added `OnceLock` to `bash_tool_provenance.rs` and `test_repo.rs` to eliminate redundant shell/config lookups.
- **Polling Efficiency:** Reduced `thread::sleep` from **25ms to 10ms** in daemon readiness and cleanup loops.
- **Cleanup Reliability:** Increased retry count to **400** and added a **200ms grace period** before hard-killing daemons to resolve mandatory file locking issues (Access Denied).

## 4. Current Performance Metrics
| Test Set | Baseline (Old) | Current (Stage 1) | Status |
| :--- | :--- | :--- | :--- |
| **Integration (Total)** | 21,850s | *TBD (Full run needed)* | **PASSING** |
| **bash_tool_provenance** | ~38s | ~35.8s | Improved |
| **Core Lib / Unit** | 390s | *Unchanged* | **PASSING** |

## 5. Next Steps for New Session
1. **Full Measurement:** Run `cargo nextest run --test integration` to establish the new baseline.
2. **Phase 2 Optimizations:**
   - **Bypass Wrapper:** Update `init_template_repo` to skip the `git-ai` wrapper for pure setup tasks.
   - **Native FS Operations:** Replace `sh -c` with `std::fs` for non-provenance tests.
   - **AV Exclusions:** Investigate programmatic temp dir exclusions for Windows Defender.

## 6. Reference Documents
- `dev\git-ai\windows_fix\README.md`: Detailed Windows dev instructions and fix history.
- `optimization_plan.md`: The multi-phase strategy document.
