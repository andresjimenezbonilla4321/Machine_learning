# `features.md` — typed feature register

**Team:** Andres, Gonzalo, Fernando, Manuel, Diego  
**Milestone:** cumulative through P3  
**Date:** 2026-10-20

## Frozen basis

| Challenge | Frozen predictive-information basis | Frozen representation |
|---|---|---|
| A — cyber | the eleven distributed source predictors | `log1p` on the eight numeric predictors |
| B — storage | the nine distributed source predictors | target `z = log10(latency_ns)` from P2.3; method-required predictor scaling only |

## Register

| Date | Milestone | Challenge | Feature | Definition | Motivation | Kind | Route |
|---|---|---|---|---|---|---|---|
| 2026-10-20 | P2 | cyber | `proto` | distributed transport protocol | source column | source feature | primary |
| 2026-10-20 | P2 | cyber | `service` | distributed network service | source column | source feature | primary |
| 2026-10-20 | P2 | cyber | `conn_state` | distributed connection state | source column | source feature | primary |
| 2026-10-20 | P2 | cyber | `duration` | distributed connection duration | source column | source feature | primary |
| 2026-10-20 | P2 | cyber | `src_bytes` | distributed source-byte count | source column | source feature | primary |
| 2026-10-20 | P2 | cyber | `dst_bytes` | distributed destination-byte count | source column | source feature | primary |
| 2026-10-20 | P2 | cyber | `missed_bytes` | distributed missed-byte count | source column | source feature | primary |
| 2026-10-20 | P2 | cyber | `src_pkts` | distributed source-packet count | source column | source feature | primary |
| 2026-10-20 | P2 | cyber | `src_ip_bytes` | distributed source IP-byte count | source column | source feature | primary |
| 2026-10-20 | P2 | cyber | `dst_pkts` | distributed destination-packet count | source column | source feature | primary |
| 2026-10-20 | P2 | cyber | `dst_ip_bytes` | distributed destination IP-byte count | source column | source feature | primary |
| 2026-10-20 | P2 | cyber | `log1p(...)` on the eight numeric predictors | monotone transform of each existing numeric predictor | compresses very different numeric ranges before a distance-based model; no new predictive information | representation transformation | primary |
| 2026-10-20 | P2 | storage | `block_size` | distributed I/O block size | source column | source feature | primary |
| 2026-10-20 | P2 | storage | `n_jobs` | distributed job count | source column | source feature | primary |
| 2026-10-20 | P2 | storage | `iodepth` | distributed I/O depth | source column | source feature | primary |
| 2026-10-20 | P2 | storage | `read_fraction` | distributed read fraction | source column | source feature | primary |
| 2026-10-20 | P2 | storage | `load_type` | distributed load type | source column | source feature | primary |
| 2026-10-20 | P2 | storage | `io_type` | distributed I/O type | source column | source feature | primary |
| 2026-10-20 | P2 | storage | `raid` | distributed RAID configuration | source column | source feature | primary |
| 2026-10-20 | P2 | storage | `n_disks` | distributed disk count | source column | source feature | primary |
| 2026-10-20 | P2 | storage | `device_type` | distributed device type | source column | source feature | primary |
| 2026-10-20 | P2 | storage | `z = log10(latency_ns)` | monotone representation of the positive target | makes the loss currency multiplicative across a target spanning more than four orders of magnitude | representation transformation (target) | primary |

`row_id`, `feature_group`, `config_group`, and `fold_id` are identifiers or evaluation metadata and are not estimator features. No derived predictive feature is introduced in P2.

## Practice 3 representation entries

The predictive-information basis remains frozen from Practice 2. Practice 3 introduces no new predictive source feature; the entries below document the model-required representation used for interpretable linear/logistic coefficients.

| Date | Milestone | Challenge | Feature / transformation | Definition | Motivation | Kind | Route |
|---|---|---|---|---|---|---|---|
| 2026-11-09 | P3 | cyber | reference-dropped one-hot for `proto`, `service`, `conn_state` | one indicator per non-reference level; references `tcp`, `-`, `SF`; unseen levels handled with `ignore` | makes categorical coefficients readable as level-versus-reference contrasts | representation transformation | primary |
| 2026-11-09 | P3 | cyber | StandardScaler after the frozen `log1p` numeric representation | training-fold mean removal and standard-deviation scaling on all eight numeric predictors | puts numeric coefficients on a common one-standard-deviation scale | representation transformation | primary |
| 2026-11-09 | P3 | storage | reference-dropped one-hot for `load_type`, `io_type`, `raid`, `device_type` | references `random`, `write`, `4+1`, `nvme`; unseen levels handled with `ignore` | makes categorical coefficients readable as level-versus-reference contrasts | representation transformation | primary |
| 2026-11-09 | P3 | storage | StandardScaler on the five numeric predictors | training-fold mean removal and standard-deviation scaling | puts numeric coefficients on a common one-standard-deviation scale | representation transformation | primary |

No derived predictive information is added in Practice 3. `row_id`, grouping metadata and `fold_id` remain excluded from estimator inputs.
