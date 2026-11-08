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
| B — mean MAE(z) | 0.209144 | 0.080906 | grouped − row-wise = +0.128238 |

For Cyber, ordinary stratified row-wise CV can place rows with an identical exact feature vector on opposite sides of a fold. For Storage, ordinary row-wise K-fold can place replicate measurements from the same configuration on opposite sides. In both cases the validation set can then contain observations that are not independent of what the model was allowed to see during fitting; the Storage effect is especially large because the five replicates of a configuration have identical predictors, so KNN finds training neighbours of the same configuration at distance zero and the score measures replicate noise instead of error on a new configuration. The Cyber effect is negligible and its sign changes across folds, because only 83 of 34,928 rows share an exact feature vector and the row-wise split leaks only 10–15 groups per fold.

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

# Protocol memo — Practice 3

**Team:** Andres, Gonzalo, Fernando, Manuel, Diego  
**Date:** 2026-11-08

## 1. The protocol we kept

| Field | Challenge A — cyber | Challenge B — storage |
|---|---|---|
| Procedure | canonical `StratifiedGroupKFold(n_splits=5, shuffle=True, random_state=2627)`, read from `cyber_folds.csv` | canonical `GroupKFold(n_splits=5)`, read from `storage_folds.csv` |
| Grouping key | `feature_group` | `config_group` |
| Validation-fold sizes | 6,986; 6,986; 6,985; 6,986; 6,985 | 2,440; 2,440; 2,440; 2,435; 2,435 |
| What changed from P2 | model family (multinomial logistic) and categorical encoding | model family (linear, Ridge, Lasso) and categorical encoding |

No partition was rebuilt in P3. Every learned object (one-hot vocabulary, scaler, model) is fitted on the training portion of each fold and applied unchanged to its validation portion. `PROJECT_RANDOM_STATE = 2627` is printed in the first cell of both notebooks.

## 2. The required interpretation route

| Element | Challenge A — cyber | Challenge B — storage |
|---|---|---|
| Reference levels (dropped indicator) | `proto=tcp`, `service=-`, `conn_state=SF` | `load_type=random`, `io_type=write`, `raid=4+1`, `device_type=nvme` |
| Reason | frequent, operationally readable baselines | frequent baselines; `4+1` is the handout's RAID example |
| Numeric predictors | `log1p`, then `StandardScaler` inside the fold | `StandardScaler` inside the fold |
| Penalty and weights | `penalty=None`, `class_weight=None`, `lbfgs`, `max_iter=1000`, `tol=5e-4`, printed from `get_params()` | none for `LinearRegression`; Ridge and Lasso only in P3.4 |
| Unseen-level policy | `handle_unknown="ignore"`: all zeros, which equals the declared reference | same policy |

## 3. The invariants we verified

| Invariant | How it was checked | Result |
|---|---|---|
| Every declared reference level occurs in every training portion | counted reference-level rows in each training portion | present in all five folds, for all three cyber and all four storage predictors |
| Unseen validation levels | compared training and validation level sets per fold | cyber: `service=gssapi` appears once in fold 3 and is encoded as the reference; storage: none |
| Probability model invariant to parameterisation | rebuilt probabilities from `w_c = beta_c - beta_r`, `b_c = a_c - a_r` and compared with `predict_proba` | maximum absolute difference 3.9e-16 |
| No regularisation in force on the cyber fit | printed `penalty`, `C`, `l1_ratio`, `class_weight` | `penalty=None`, `class_weight=None` |

## 4. The parameterisation decision

`coef_` has shape (10, 30): one vector per class and no class held at zero. This is scikit-learn's symmetric parameterisation. The library does not choose or store a reference class. We chose `normal` as the course reference class because it is the natural operational baseline, and obtained the baseline-category coefficients by our own differencing. Three objects are kept apart throughout: the model's probabilities, the stored symmetric coefficients, and the reference-class coefficients. Every odds-ratio sentence names both classes and, for a categorical predictor, the level and its declared reference level. Original categorical and numeric predictors are never ranked against each other by raw coefficient size.

## 5. The metric contract

| Challenge | Primary metric | Reported in P3 |
|---|---|---|
| A — cyber | macro F1 | mean 0.827519, fold range 0.007422; weighted F1 0.848197; accuracy 0.850235; per-class precision/recall/F1 with support; labelled confusion matrix |
| B — storage | MAE(z) | mean 0.198146, fold range 0.011054; RMSE(z) 0.270346; R²(z) 0.896356; one MAE_ns of 16,289,769 ns computed after inverting each prediction with `10 ** z_hat` |

`10 ** MAE(z)` is never reported as a nanosecond error. Ransomware (support 208) is read beside its support: precision 0.933, recall 0.9375, F1 0.935. The weakest cyber class is `mitm` (recall 0.361, support 805).

## 6. What the comparison against Practice 2 measured

Both families are scored on the same five canonical folds, so the comparison is paired. Differences follow the course convention: macro F1 `new − P2`, MAE(z) `P2 − new`, so a positive value means the new family did better.

| Challenge | P3 model vs P2 KNN | Mean paired difference | SD across folds | Folds where P3 is better |
|---|---|---|---|---|
| A — macro F1 | multinomial logistic | −0.118038 | 0.002098 | 0 of 5 |
| B — MAE(z) | LinearRegression | −0.019509 | 0.003677 | 0 of 5 |
| B — MAE(z) | Ridge (alpha 31.6228) | −0.019383 | 0.003730 | 0 of 5 |
| B — MAE(z) | Lasso (alpha 0.00316228) | −0.018944 | 0.003861 | 0 of 5 |

The change has the same sign on all five folds in both challenges and is large relative to the fold-to-fold variation. This is a descriptive statement only: fold spread is not a significance test and we make no claim of statistical significance.

The primary comparison rows are **pipeline comparisons**, not isolated estimator-family comparisons. P3 changes the categorical representation (reference-dropped one-hot with fold-fitted scaling) together with the model family, and P2 KNN was not rerun on that representation. The evidence therefore licenses a statement about the two submitted pipelines, not about the estimator family alone.

## 7. The regularisation decision (Challenge B)

The unregularised `LinearRegression` stays in the comparison as the comparator. The design matrix has 12,190 rows and 13 columns, rank 13 and condition number about 8.24, so it is neither wide nor near rank-deficient. On the declared grid `alpha = logspace(-4, 2, 13)` the best mean MAE(z) is 0.198020 for Ridge and 0.197580 for Lasso, against 0.198146 unregularised; the selected Lasso sets 2 of the 13 coefficients exactly to zero. These movements are tiny, and the problem gives no structural reason for sparsity. We therefore keep the unregularised fit as the primary interpretable linear model, with Ridge as the safer alternative if correlated predictors became a concern.

## 8. What we appended, and what we did not change

`comparativa.csv` keeps the four Practice 2 rows unchanged and appends four Practice 3 rows with the `route` column filled (`primary`). `features.md` gains a dated Practice 3 section: only representation changes, no derived predictive feature. The optional regularised Challenge A variant was not run.