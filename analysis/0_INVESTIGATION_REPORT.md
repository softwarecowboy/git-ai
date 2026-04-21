# 0. Investigation Report: Windows Performance & Stability

This document outlines the root cause analysis and investigation performed for `git-ai` on Windows (non-WSL).

## **1. Root Cause Analysis**
The following four primary bottlenecks were identified as the main drivers of test suite slowness and instability on Windows.

### **1.1. Wrapper Recursion ("Process Storms")**
- **Symptom:** Integration tests would suddenly slow down, and dozens of `git-ai` background daemons would spawn unexpectedly.
- **Cause:** When calling `git` from within a test, Windows sometimes resolved the `git` command to the installed `git-ai` wrapper in the `PATH` instead of the system's `git.exe`. 
- **Impact:** Every Git command became a recursive `git-ai` invocation, leading to resource exhaustion.

### **1.2. Shell-Based Overhead**
- **Symptom:** Tests involving directory synchronization (e.g., `rsync`) were noticeably slow.
- **Cause:** Relying on `sh -c` or `rsync` requires spawning a shell (often via MinGW/MSYS2/Cygwin) and then the command itself. On Windows, this process spawning is an order of magnitude slower than on Unix.
- **Impact:** Sub-second operations on Linux took multiple seconds on Windows.

### **1.3. Fixed Polling Latency**
- **Symptom:** Cumulative "idle" time was high across the suite.
- **Cause:** Internal polling loops (hooks, commit stats, daemon readiness) used a default `Duration::from_millis(25)`.
- **Impact:** With 3,000+ tests, these 25ms sleeps accumulated into thousands of seconds of unnecessary waiting.

### **1.4. Antivirus (Windows Defender) Interference**
- **Symptom:** I/O operations (file creation/deletion) were jittery and slow in temporary directories.
- **Cause:** Real-time monitoring by Windows Defender scans every small file created by the test suite (e.g., `.git` objects, temporary test repos).
- **Impact:** Significant I/O wait times and increased CPU usage for the AV process.

## **2. Methodology**
- **Detailed Tracing:** Enabled `GIT_AI_DEBUG_PERFORMANCE=2` to get millisecond-level breakdowns of wrapper execution.
- **Baseline Measurements:** Established baseline timings for `bash_tool_provenance` (36.7s) and `rebase_benchmark` (50 commits).
- **Tool Probing:** Verified that `where.exe git` could reliably find the real Git binary on Windows installations.
