# `features.md` — typed feature register

**Team:** Andres, Gonzalo, Fernando, Manuel, Diego  
**Milestone:** P3 (cumulative; P2 entries kept unchanged)  
**Date:** 2026-11-08

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

## Practice 3 entries

| Date | Milestone | Challenge | Feature | Definition | Motivation | Kind | Route |
|---|---|---|---|---|---|---|---|
| 2026-11-08 | P3 | cyber | one-hot of `proto`, `service`, `conn_state` with declared reference levels `tcp`, `-`, `SF` (reference indicator dropped) | indicator columns, one per level except the reference, fitted inside each fold | a coefficient then means "this level versus the declared reference level"; required interpretation route | representation transformation (no new predictive information) | primary |
| 2026-11-08 | P3 | cyber | `log1p` + `StandardScaler` on the eight numeric predictors | same declared scale for every numeric predictor, scaler fitted on training rows of each fold | coefficients of numeric predictors are comparable only on one common scale | representation transformation (no new predictive information) | primary |
| 2026-11-08 | P3 | storage | one-hot of `load_type`, `io_type`, `raid`, `device_type` with declared reference levels `random`, `write`, `4+1`, `nvme` | indicator columns, one per level except the reference, fitted inside each fold | a coefficient then means "this level versus the declared reference level" on `z`; `4+1` matches the handout's RAID example | representation transformation (no new predictive information) | primary |
| 2026-11-08 | P3 | storage | `StandardScaler` on the five numeric predictors | one common declared scale, fitted on training rows of each fold | numeric coefficients on `z` are comparable as per-standard-deviation effects | representation transformation (no new predictive information) | primary |

Unseen-level policy (both challenges): `handle_unknown="ignore"` encodes an unseen level as all zeros, which with the reference dropped is the same encoding as the declared reference level; this is declared, not implicit.

`Ridge` and `Lasso` (storage) are model families on the same representation; they introduce no new feature. No derived predictive feature is introduced in P3, and `row_id`, `feature_group`, `config_group` and `fold_id` are still not estimator features.