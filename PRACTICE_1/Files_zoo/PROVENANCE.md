# `zoo_features.csv` / `zoo_labels.csv` -- the Zoo database of Practice 1

## Source and attribution

| Field | Value |
|---|---|
| Dataset | **Zoo** |
| Repository | UCI Machine Learning Repository, dataset 111 |
| Creator | **Richard Forsyth** |
| DOI | **`10.24432/C5R59V`** |
| Licence | **Creative Commons Attribution 4.0 International (CC BY 4.0)** |

The licence permits use and redistribution, including in teaching material, **provided the
source is attributed**. This file is that attribution, and it travels with the data.

## How these files reached the course tree

The originals `zoo.data` and `zoo.names` were **supplied by the professor**. **Nothing was
downloaded**, nothing was reconstructed and no substitute dataset was used; no part of Practice 1
requires a network connection at any point.

The two course files below were **derived** from those originals by `derive_zoo.py`, which first
verified the source structurally and then split it. The derivation preserves **source row order**
and adds a course-owned integer `row_id`.

### Source identities

| File | Bytes | SHA-256 |
|---|---:|---|
| `zoo.data` (source, not distributed) | 4,126 | `cddc71c26ab9bc82795b8f4ff114cade41885d92720c6af29ffb69bcf73f0315` |
| `zoo.names` (source, not distributed) | 2,587 | `db3dd334beb643c9e14b75603360ea5edcac90a6ac19302e1896039f0c20d428` |

### Derived course files

| File | Bytes | SHA-256 |
|---|---:|---|
| `zoo_features.csv` | 4,356 | `9267c370aa77321de75619606381a675c98fb48add5070fab303f627e27305cb` |
| `zoo_labels.csv` | 510 | `26ff33bc011dd288683cf829d0b3aa8da471addc04d498ab4061aba5403d3261` |
| `zoo_schema.txt` | 1,865 | `32471dc1282393cda25976de90505c9f2642b45dc02c89fdd8b6db671839c418` |

## Verification performed on the source before deriving anything

| Property | Required | Observed |
|---|---|---|
| Observations | 101 | 101 |
| Fields in the source record | 18 | 18 |
| Predictive features | 16 | 16 |
| Missing values | none | 0 |
| Target field | structurally verified against the professor-supplied source; details
intentionally withheld from the student-facing provenance until the reveal stage. |
| Predictor domains | 0/1 except `legs` | verified |

**`animal_name` uniqueness is deliberately NOT checked, because the official data does not have
that property.** The source contains repeated names -- observed here: ['frog'] -- which
the dataset's own documentation notes. `animal_name` is therefore treated as a **display/name
field**, and the course-owned integer `row_id` is the only key used for alignment. Manufacturing
a uniqueness property the source does not possess would have been an error, not a safeguard.

## Why the original `zoo.names` is not distributed

The official `zoo.names` contains the **class breakdown, the class counts and the animal-to-class
listing**. Practice 1's unsupervised seal requires that a student cannot know the number or
distribution of classes before both clustering decisions are frozen, so distributing that file
would have contradicted the seal in the same package that enforces it.

`zoo_schema.txt` is the sanitized student-facing replacement. It gives the field names and types
needed to understand the input and **nothing about the target**: no class names, no class counts,
no cardinality, no memberships.

**This provenance file is student-facing too, and is sanitized on the same rule.** Attribution
must travel with the data, so the file ships; but everything that would disclose the target
domain before the reveal -- the set of class values, how many there are, what they are called,
how many rows each holds, and which animal belongs to which -- is withheld here as well. The
complete structural verification of the target field, including its observed domain, is recorded
in the course `CHANGELOG.md` as instructor evidence, where no student reads it before the seal
is opened.

## Why the data is split into two files

The seal is a property of **what is on disk**, not of programmer discipline. `zoo_features.csv`
contains **no class column at all**, so the pre-reveal code cannot read the target even by
mistake -- there is nothing to drop and nothing to be careful about. `zoo_labels.csv` is opened
only in the reveal stage. The mechanical guard in both notebooks remains a **method guard**: it
proves the order of the executed cells, and it is not a claim of cryptographic access control.

## Not a medical, biological or taxonomic claim

Zoo is used as a **standard educational benchmark for clustering**. Nothing computed in
Practice 1 is a biological or taxonomic claim, and the class codes are treated throughout as a
**nominal** coding with no ordering -- a point Exercise 1(e) makes explicitly by showing that
renumbering them **can change** the result. (The reference also demonstrates a renumbering that
provably cannot: a distance-preserving one. `Can` is the accurate word, not `does`.)

## What these files are not

They are **not** project data. Practice 1 is deliberately terminal: it feeds nothing into
Practices 2--5. Zoo has no relationship to the cybersecurity or storage challenges, no hidden
partition, and no final test.
