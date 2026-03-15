# Quartic reduction data

This repository stores datasets produced by the script
`random_quartic_reduction_experiments.py`
from the **StabilityFunction** project.

It serves as a data companion to the paper
*Semistable reduction of plane quartics*.

The code for generating the data lives in the `StabilityFunction`
repository. This repository stores the output of runs together with
metadata describing how each dataset was produced.

---

# Repository structure

Each dataset corresponds to one run of the experiment script and is
stored in its own directory.

Example:

    quartic_reduction_data/
      README.md
      runs/
        p3_seed12345_timeout60_2026-03-15/
          data.jsonl
          stats.json
          run_parameters.json

Each run directory contains:

- **data.jsonl** — stored examples (JSON Lines format)
- **stats.json** — aggregate statistics of the run
- **run_parameters.json** — parameters used for the run plus timestamp

Always use a fresh directory for each run.

---

# Producing data

Data are produced from a checkout of the **StabilityFunction**
repository using Sage.

Example run:

    sage -python scripts/random_quartic_reduction_experiments.py \
      --n-samples 500 \
      --prime 3 \
      --seed 12345 \
      --n-terms 8 \
      --coeff-bound 10 \
      --rho 0.0 \
      --kmax 3 \
      --timeout 60 \
      --out /path/to/quartic_reduction_data/runs/p3_seed12345_timeout60_2026-03-15

The script

1. generates random homogeneous quartics `F(x,y,z) ∈ QQ[x,y,z]`,
2. discards those that are singular,
3. computes the `p`-adic stable reduction of the curve,
4. stores the results and summary statistics.

If the output directory does not exist, it is created automatically.

---

# Meaning of parameters

## Sampling parameters

| parameter | meaning |
|-----------|---------|
| `--n-samples` | number of smooth quartics to process |
| `--prime` | prime `p` defining the `p`-adic valuation |
| `--seed` | random number generator seed |
| `--n-terms` | number of monomials used in the quartic |
| `--coeff-bound` | coefficients sampled from `[-bound,bound]` |

A homogeneous quartic has 15 monomials, so `n_terms ≤ 15`.

## Optional p-adic bias

The parameters `rho` and `kmax` bias coefficients toward higher
`p`-adic valuation.

If `rho = 0` (default), coefficients are chosen uniformly.

If `rho > 0`, coefficients are generated as

    c = p^K * u

where

- `u` is uniform in `[-coeff_bound, coeff_bound]`,
- `K` is sampled using a truncated geometric distribution.

| parameter | meaning |
|-----------|---------|
| `--rho` | bias parameter for valuation |
| `--kmax` | maximum exponent used in the bias |

This increases the probability that the naive reduction modulo `p`
is singular.

## Runtime parameters

| parameter | meaning |
|-----------|---------|
| `--timeout` | wall-clock limit in seconds for one example |
| `--max-tries-factor` | maximum attempts = factor × `n_samples` |
| `--checkpoint-every` | interval for writing statistics |
| `--store-all` | also store non-successful samples |
| `--quiet` | reduce console output |

If the computation for one example exceeds the timeout, the example
is aborted and counted as `timeout`.

---

# What the script computes

For each sampled smooth quartic, the script calls the stable-reduction
routine from **StabilityFunction**.

Conceptually the algorithm:

1. computes a **GIT-semistable plane model**,
2. determines the special fibre,
3. if the special fibre is **GIT-stable**, resolves the cusps,
4. computes the **stable reduction graph** and the reduction type.

Possible outcomes:

| status | meaning |
|--------|---------|
| `ok` | stable non-hyperelliptic reduction computed |
| `hyperelliptic` | GIT special fibre not stable (hyperelliptic case) |
| `fail` | computation raised an exception |
| `timeout` | experiment script aborted the computation |

---

# Output files

## run_parameters.json

Records the parameters used for the run and a timestamp.

Typical fields:

    n_samples
    prime
    seed
    n_terms
    coeff_bound
    rho
    kmax
    timeout
    max_tries_factor
    checkpoint_every
    timestamp_utc

This file documents how the dataset was generated.

## stats.json

Contains aggregate statistics for the run.

Typical fields:

    found
    tries
    stored
    time_total_sec
    status_counts
    type_counts

Key quantities:

| field | meaning |
|-------|---------|
| `found` | number of smooth quartics processed |
| `tries` | total random quartics generated |
| `stored` | number written to `data.jsonl` |
| `status_counts` | counts of `ok`, `hyperelliptic`, `fail`, `timeout` |
| `type_counts` | counts of reduction types among `ok` cases |

## data.jsonl

JSON Lines file containing one stored record per sample.

In the default archival mode only successful (`ok`) cases are stored.

Typical record:

    {
      "id": "...",
      "K_defpoly": "QQ",
      "K_gen": null,
      "p": 3,
      "F": "x^4 + ...",
      "reduction_type": "...",
      "git_dfe": [d,f,e],
      "cusp_dfes": [[d,f,e], ...],
      "time_sec": ...
    }

Meaning of fields:

| field | meaning |
|-------|---------|
| `id` | short identifier |
| `K_defpoly` | defining polynomial of the base field |
| `K_gen` | generator of the base field |
| `p` | prime defining the valuation |
| `F` | quartic equation |
| `reduction_type` | genus-3 reduction type |
| `git_dfe` | `[d,f,e]` describing the extension where a GIT-semistable model exists |
| `cusp_dfes` | list of `[d,f,e]` describing cusp-resolution extensions |
| `time_sec` | runtime for the example |

Here

    d = [L : K]
    f = residue degree
    e = ramification index

with `d = e f`.

---

# Reading the data

To inspect a run quickly, read `stats.json`.

To process individual examples, read `data.jsonl` line by line.

Example in Python:

    import json

    with open("data.jsonl") as f:
        for line in f:
            rec = json.loads(line)
            print(rec["p"], rec["reduction_type"])

---

# Remarks

The data describe experimental behaviour of the current implementation.

Observed distributions depend on

- the sampling model,
- the chosen parameters,
- the timeout,
- the machine used.

For reproducibility, always keep together

    data.jsonl
    stats.json
    run_parameters.json

and the corresponding version of the **StabilityFunction** code.
