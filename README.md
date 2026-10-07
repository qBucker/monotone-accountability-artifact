# Reproduction artifact — CUSUM trigger Monte Carlo

Self-contained package that regenerates, with a single command, every cell
of the CUSUM Monte Carlo tables of the accompanying submission (the
false-alarm-rate and detection-delay tables of the statistical layer,
Appendix B, plus its condensed main-text table). Prepared for double-blind
review; no network access is required.

## Contents

- `cusum_mc.py` — the simulation (~55 lines; Python 3 + NumPy only).
- `expected_output.txt` — reference output produced with the fixed seed.
- `verify.py` — compares a fresh run against the reference, cell by cell.
- `SHA256SUMS` — checksums of the root files.
- `engineering-source/` — the full engineering repository as mirrored for
  review: the Rust crate (`src/`, `examples/`, `benches/`), evaluation
  scripts (`scripts/`), raw measurement logs with their own
  `measurements/raw/SHA256SUMS`, and the post-quantum shell tree
  (`rsep-pq-shell/`). Own checksum manifest at
  `engineering-source/SHA256SUMS`.

## Engineering source

Every proof-system figure and every measured row of the paper was produced
from the `engineering-source/` tree with the toolchains pinned in its
`Cargo.lock` (arkworks 0.4, Winterfell 0.13.1, RISC Zero 3.0.6). The raw
per-run logs live under `engineering-source/measurements/raw/` and verify
against the manifest shipped there (52/52 files).

## Requirements

- Python 3.10+ with NumPy (tested with Python 3.12 and NumPy 1.24–2.x;
  the fixed seed makes the sample stream bit-stable across this range).
- Runtime: about 90 seconds on a laptop.

## One-command reproduction

```bash
python3 cusum_mc.py > my_output.txt
python3 verify.py my_output.txt expected_output.txt
```

`verify.py` exits 0 and prints `ALL CELLS MATCH` (162 cells) if and only if
every cell agrees with the reference.

## What is simulated

Standardized observations `x_t ~ N(mu, 1)`; null `mu = 0`, alternative
`mu = mu1`. One-sided CUSUM `S_t = max(0, S_{t-1} + x_t - k)` with reference
value `k = mu1/2`, alarm when `S_t > h`. Grid: `mu1 in {0.5, 1, 2}`,
`h in {2..10}`, `T in {100, 1000, 10000}`, `N = 10^4` streams per cell.
Common random numbers (the H1 stream is the H0 stream shifted by `mu1`)
reduce comparison noise. Fixed seed `20260920` makes every cell
bit-reproducible. The statistic is computed in the vectorized closed form
`S_t = C_t - min_{s<=t} C_s` where `C` is the cumulative sum of `x - k`.

Outputs: FAR `= P(tau <= T | H0)` per cell, and mean detection delay
`E[tau | H1]` versus the design approximation `2h/mu1` (the paper reports
the `T = 10000` rows, where no censoring occurred).

## Scope (honest boundary)

This artifact covers the statistical layer only. The circuit constraint
counts and the proof-system timings reported elsewhere in the paper were
measured with the Rust harness of the companion engineering repository
(released with the camera-ready); reproducing them requires the toolchains
pinned there (arkworks 0.4, Winterfell 0.13.1, RISC Zero 3.0.6) and is out
of scope for this lightweight package. The theoretical results (necessity,
composition, separation) are proofs, not experiments.
