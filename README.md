# Google Colab CPU Side-Channel Security Research

A responsible disclosure report documenting 23 C++ experiments probing CPU-level side-channel vulnerabilities within Google Colaboratory's shared Intel Xeon infrastructure.

## What's in this repo

| File | Description |
|---|---|
| `google_colab_security_report.pdf` | Full research report (64 KB, ~20 pages) |
| `colableaktest_1778496790880.ipynb` | Original Jupyter notebook with all experiment source code and outputs |

## Report overview

The PDF documents 23 progressive experiments targeting Google Colab's process isolation using:

- **Spectre v1 / v2** — Flush+Reload cache side channels
- **MDS / RIDL** — Microarchitectural Data Sampling via fill buffers
- **Transient execution** — Speculative jumps into kernel address space
- **L3 cache saturation** — Shared last-level cache timing analysis
- **Branch predictor poisoning** — Training-based misprediction attacks

Each experiment includes: attack method, raw output, technical analysis, verdict, and success rate.

## Summary of results

| Outcome | Count |
|---|---|
| Failed / Blocked | 10 |
| Manually Interrupted (inconclusive) | 5 |
| Partial signal observed | 7 |
| Compile error (not executed) | 1 |

Google's existing defenses (KPTI, Retpoline/IBRS, MDS microcode patches, dot-fill sanitization, VM isolation) blocked all experiments. No confirmed theft of kernel or cross-tenant data was achieved.

## Key findings for Google's security team

1. **Experiment 17 — l3_dump.cpp** produced a full 1024-byte hex dump targeting kernel address range `0xFFFFFFFF81000000`. Whether the content reflects actual kernel data or own-process L3 cache residue requires internal verification against ground truth. **Flagged HIGH priority.**

2. **RDTSC is unrestricted** — direct TSC access enables sub-nanosecond timing measurements, the foundational primitive for all Flush+Reload attacks.

3. **Fork rate is not limited** — Experiment 6 spawned 11,500+ child processes in ~2 minutes with no throttling, presenting a CPU denial-of-service path for co-resident tenants.

4. **Timing infrastructure is functional** — The Flush+Reload pipeline in covert_leak.cpp works correctly. Current failure to leak kernel data is due to gadget misconstruction, not a breakdown of the side channel itself.

## Recommendations (summary)

| ID | Severity | Action |
|---|---|---|
| R1 | CRITICAL | Verify l3_dump output against actual kernel memory content |
| R2 | CRITICAL | Restrict or virtualise RDTSC with controlled jitter |
| R3 | HIGH | Implement per-session fork() rate limiting |
| R4 | HIGH | Audit dot-fill sanitization coverage map |
| R5 | MEDIUM | Add per-session thread count limits |
| R6 | MEDIUM | Enable Intel CAT L3 cache partitioning between tenant VMs |
| R7 | LOW | Continue KPTI and microcode patch cadence |
| R8 | LOW | Monitor Intel PSIRT for new Spectre variant gadgets |

## Disclosure

All experiments were conducted within the researcher's own Colab session. No other users' data was accessed or targeted. This repository is shared with Google's security team under responsible disclosure principles.

**Requested response:** Acknowledgement of receipt and an update within 90 days regarding the authenticity of the Experiment 17 (l3_dump.cpp) output and any planned remediation timeline.

**Report date:** May 11, 2026
