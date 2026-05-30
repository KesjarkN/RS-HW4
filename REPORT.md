# RS | Fourth homework assignment

---

## Task 1: Analyzing the performance of GPU kernels in GEM5

| Kernel | CU | loadLatencyDist::mean | vALUInsts | ldsBankAccess | totalCycles | vpc |
|---|---:|---:|---:|---:|---:|---:|
| histogram_opt | 2 | 799,616 | 28,672 | 393,216 | 227,142 | 21.93 |
| histogram_opt | 4 | 1,658,722 | 14,336 | 196,608 | 175,938 | 14.15 |
| histogram_opt | 8 | 4,104,118 | 7,168  | 98,304  | 174,493 | 7.14  |
| histogram_naive | 2 | 6,193,091 | 22,528 | 0      | 828,458 | 3.80  |
| histogram_naive | 4 | 12,398,205| 11,264 | 0      | 808,687 | 1.94  |
| histogram_naive | 8 | 24,180,700| 5,632  | 0      | 783,717 | 1.00  |

**Short observations**
- `histogram_opt` (privatized) achieves substantially lower mean load latency and total cycles versus `histogram_naive` across all CU counts — indicates privatization reduces contention and improves throughput.
- `histogram_opt` uses substantial LDS (`ldsBankAccess` > 0) while `histogram_naive` shows zero LDS access, consistent with privatization using shared/local memory.
- `vpc` (vector-per-cycle) is significantly higher for `histogram_opt`, showing better vector throughput.

---

## Task 2: Implementing Matrix transpose using Shared Memory

For transpose the stats file contains two kernel-period occurrences; values are shown as `kernel1 / kernel2` (first run / second run).

| CU | loadLatencyDist::mean | vALUInsts | ldsBankAccess | totalCycles | vpc |
|---:|---:|---:|---:|---:|---:|
| 2 | 8,027,456 / 2,491,773 | 36,720 / 52,832 | 0 / 260,096 | 527,800 / 315,418 | 7.42 / 21.85 |
| 4 | 18,955,491 / 5,850,951 | 18,504 / 27,144 | 0 / 133,632 | 578,652 / 318,277 | 3.41 / 11.13 |
| 8 | 42,141,729 / 12,512,509 | 9,144  / 13,416 | 0 / 66,048  | 663,934 / 313,278 | 1.47 / 5.59 |

**Short observations**
- The second kernel (run 2) consistently shows much lower `loadLatencyDist::mean` and `totalCycles` across all CU counts, indicating substantially improved memory locality and throughput.
- The second run uses significant LDS (`ldsBankAccess`), while the first run shows zero LDS usage — this aligns with a tiled/shared-memory implementation in the second kernel.
- `vpc` (vector ops per cycle) is markedly higher for the second run, confirming better utilization of vector hardware in the tiled implementation.


---

## Task 3: Analyzing the impact of thread divergence on performance of SpMV kernel

| Kernel   | CU | CFDiv::mean | CFDiv::stdev | vALUInsts | globalReads | globalWrites | coalsrLines |
|---|---:|---:|---:|---:|---:|---:|---:|
| divergent | 2 | 33.496229 | 19.051824 | 6328 | 1544 | 8 | 3104 |
| sorted    | 2 | 61.014577 | 10.370069 | 3256 | 776  | 8 | 1664 |
| divergent | 4 | 33.496229 | 19.052722 | 3164 | 772  | 4 | 3104 |
| sorted    | 4 | 60.620462 | 10.974078 | 1436 | 340  | 4 | 1664 |
| divergent | 8 | 33.496229 | 19.054518 | 1582 | 386  | 2 | 3104 |
| sorted    | 8 | 59.408072 | 12.576833 | 526  | 122  | 2 | 1664 |

**Short observations**
- `CFDiv::mean` is higher for the `sorted` run across all CU counts (for example, 61.01 vs 33.50 at CU=2), so the sorted ordering keeps more lanes active per instruction on average in these snapshots.
- `vALUInsts` is lower for the `sorted` runs (for example, 3256 vs 6328 at CU=2), which means fewer vector ALU instructions were issued in the sorted case.
- `coalsrLines` is lower for the `sorted` run (1664 vs 3104), which suggests the sorted ordering has better memory coalescing.

