# 2. Benchmark Verification

This document presents the metrics and comparative data from the Phase 2 verification process.

## **2.1. Integration Test Runtime Improvements**
Benchmarks were run for several I/O-intensive integration tests to measure the impact of polling and FS optimizations.

| Test Case | Baseline (Phase 1) | Post-Phase 2 | Reduction |
| :--- | :--- | :--- | :--- |
| `test_bash_provenance_modify_20_of_50_tracked` | 19.49s | 10.91s | **~44%** |
| `test_bash_provenance_create_50_files` | 2.76s | 1.51s | **~45%** |

## **2.2. Rebase Benchmark Metrics**
Large-scale rebase operations were performed using the native `sync_dir` caching mechanism and reduced polling latency.

- **Rebase many AI commits (50 feature commits):**
  - **Total Wall Time:** 5.89s
  - **Per-commit Overhead:** ~117.8ms
  - **Observation:** The rebase process remains highly responsive, even with a deep history of AI-attributed lines.

- **Monorepo Benchmark (30 feature commits, 18 files):**
  - **Total Wall Time:** 11.67s
  - **Per-commit Overhead:** ~389.0ms
  - **Observation:** Native synchronization ensures that even with larger repositories, the overhead is dominated by Git's own internal operations rather than the `git-ai` wrapper.

## **2.3. Summary of Impact**
The combination of improvements resulted in an overall **~45% runtime reduction** for the most expensive I/O-heavy portions of the integration test suite on Windows. This significantly exceeds the 20% target set at the beginning of the phase.
