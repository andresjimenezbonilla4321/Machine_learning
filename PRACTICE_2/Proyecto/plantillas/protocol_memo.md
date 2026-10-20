# Protocol memo — Practice 2

**Team:** Andres, Gonzalo, Fernando, Manuel, Diego  
**Date:** 2026-10-20

## 1. The protocol we built

| Field | Challenge A — cyber | Challenge B — storage |
|---|---|---|
| Procedure and parameters | `StratifiedGroupKFold(n_splits=5, shuffle=True, random_state=2627)` | `GroupKFold(n_splits=5)` |
| Grouping key | `feature_group` | `config_group` |
| Randomness | splitter consumes the frozen project seed 2627 | none consumed by `GroupKFold` |
| Validation-fold sizes | 6,986; 6,986; 6,985; 6,986; 6,985 | 2,440; 2,440; 2,440; 2,435; 2,435 |

The team-generated partitions were constructed before reading the released sidecars. They were then frozen and retained in the notebooks as evidence of understanding.

## 2. The invariants we verified

| Invariant | How it was checked | Result |
|---|---|---|
| No exact feature group in both portions of any Cyber fold | intersected the `feature_group` sets of the training and validation indices in each round | 0 shared groups in all five folds |
| No Storage configuration split across folds | intersected the `config_group` sets of the training and validation indices in each round | 0 shared groups in all five folds |
| Every Cyber class present in every canonical validation portion | counted distinct `attack_code` values per released `fold_id` | all 10 classes present in every fold |

A positive overlap count, or fewer than ten Cyber classes in a canonical validation fold, would have been evidence of protocol failure.

## 3. What the split-protocol comparison measured

The course-fixed estimator and preprocessing were held constant within each challenge. Only the splitting protocol changed.

| Challenge | Grouped | Ungrouped / row-wise | Observed difference |
|---|---:|---:|---:|
| A — mean macro F1 | 0.944165 | 0.944251 | row-wise − grouped = +0.000085 |
| B — mean MAE(z) | 0.209144 | 0.080919 | grouped − row-wise = +0.128225 |

For Cyber, ordinary stratified row-wise CV can place rows with an identical exact feature vector on opposite sides of a fold. For Storage, ordinary row-wise K-fold can place replicate measurements from the same configuration on opposite sides. In both cases the validation set can then contain observations that are not independent of what the model was allowed to see during fitting; the Storage effect is especially large because KNN can find neighbours from the same configuration.

Grouping is correct whatever the numerical gap happens to be. The intended score estimates generalisation to a different exact feature group in Cyber and to an unseen storage configuration in Storage. A row-wise score estimates an easier question whenever group-linked observations cross the training/validation boundary.

## 4. The `log10` decision

Storage latency is strictly positive and spans from about 1.16e5 ns to 2.53e9 ns, more than four orders of magnitude. The raw distribution is strongly right-skewed. On the raw scale, the same proportional prediction error produces a much larger absolute error at the high-latency end, so raw MAE gives the largest latencies much more influence simply because of the unit scale.

We therefore adopt `z = log10(latency_ns)`. This monotone target representation preserves ordering while changing the loss currency: equal differences in `z` correspond to equal multiplicative factors on the physical scale. That makes comparable proportional errors comparable across low- and high-latency configurations. The decision is based on the scale, loss currency and structure of the target, not on which demonstration score happened to be smaller.

Log-scale metrics are computed directly from `z` and `z_hat`. For a physical metric we first invert each prediction with `latency_ns_hat = 10 ** z_hat` and only then compare it with the raw latency. `10 ** MAE(z)` is not reported as a nanosecond error.

## 5. The metric contract

| Challenge | Primary metric | Why it fits the engineering problem | Secondary operational concern |
|---|---|---|---|
| A — cyber | macro F1 | each of the ten attack classes contributes equally, so a rare class cannot be hidden by the large classes | ransomware recall and false-alarm burden, analysed in P2.5 |
| B — storage | MAE(z) | reports typical absolute error in the adopted log-scale currency without squaring large residuals | physical MAE in nanoseconds after inverse transformation |

Cyber additionally reports weighted F1, accuracy, per-class precision/recall/F1 with support, and a semantically labelled confusion matrix. Storage additionally reports RMSE(z), R2(z), and one correctly inverted MAE_ns.

## 6. Declaration that closes the primary comparison basis

| Challenge | Frozen predictive-information basis | Frozen representation | Method-specific choices |
|---|---|---|---|
| A — cyber | 11 distributed source predictors | `log1p` on the eight numeric predictors | categorical encoding and method-required scaling |
| B — storage | 9 distributed source predictors | target `z = log10(latency_ns)` | categorical encoding and method-required scaling |

`row_id`, `feature_group`, `config_group`, and `fold_id` are never estimator inputs. No derived predictive feature is introduced in P2. Therefore the primary comparison rows are pipeline comparisons unless a later milestone explicitly holds the predictive representation constant or reruns earlier families on the same representation.

## 7. What we adopted, and when

Our own fold constructions and invariant reports were frozen in P2.2 before the released sidecars were read. For both challenges the partitions match the canonical partitions exactly. The only discrepancy is numeric labelling: our loops numbered folds 1–5, while the canonical sidecars use 0–4. After normalising that label convention, agreement is 100% row by row. From P2.2(g) onward every scored development result in both notebooks uses the released canonical assignment.
