# `AI_USAGE.md` — generative-AI disclosure

**Team:** Andres, Gonzalo, Fernando, Manuel, Diego  
**Cumulative:** one section per milestone.

## Practice 2

| Field | Entry |
|---|---|
| Tool or service | ChatGPT (OpenAI); Claude Code (Anthropic) |
| Purpose | Explanation of the course protocol and metrics; debugging; suggestions for notebook structure and Python implementation; language review of English Markdown; assistance integrating the released canonical fold files and the supplied format validator; review of both notebooks against the statement, fixing the canonical-fold file paths, and rewriting interpretation cells so they cite the executed results. |
| Files / sections affected | `p2_challenge_A_cyber.ipynb`, `p2_challenge_B_storage.ipynb`, `informe.md`, `features.md`, `roles.md`, `comparativa.csv`, and this disclosure. |
| How the output was checked | Both notebooks were executed top-to-bottom on the distributed development tables; schema assertions, grouping invariants, canonical-fold comparisons and fold coverage checks were required to pass; both dry-run submission files were checked with the course-supplied `validate_submission.py`; reported values were read from the executed notebook outputs. The team remains responsible for understanding and defending every submitted step. |

## Practice 3

| Field | Entry |
|---|---|
| Tool or service | ChatGPT (OpenAI) |
| Purpose | Explanation of the Practice 3 requirements; suggestions for reproducible notebook structure and Python implementation; debugging of the fold-local preprocessing and multinomial parameterisation bridge; checking odds-ratio wording; assistance with the L1/L2 comparison and English Markdown review. |
| Files / sections affected | `p3_cyber.ipynb`, `p3_storage.ipynb`, `comparativa.csv`, `features.md`, `roles.md`, and this disclosure. |
| How the output was checked | Both notebooks were executed top-to-bottom on the distributed development tables. Checks include canonical fold coverage, declared reference levels present in every training portion, the explicit unseen-level policy, printed effective logistic penalty parameters, numerical equality of the reference-class probability reconstruction and `predict_proba`, fold-wise metric recomputation, physical-scale inversion with `10 ** z_hat`, and preservation of the original Practice 2 comparison rows. The team remains responsible for understanding and defending every submitted step. |

## Practice 4

_Not yet completed._

## Practice 5

_Not yet completed._

## Final Integration & Defence

_Not yet completed. No generative-AI assistance is permitted during individual defence questioning._
