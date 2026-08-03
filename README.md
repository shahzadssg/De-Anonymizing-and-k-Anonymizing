# De-Anonymization and $k$-Anonymization in Subject-Object Knowledge Networks

Python implementations and a reproducible analysis framework for two conflicting objectives in subject-object knowledge networks: de-anonymization and $k$-anonymization. Relationships are modelled as bipartite graphs (users accessing resources in cloud storage, students answering questions), and heuristic algorithms either strengthen unique identification for accountability or enforce group indistinguishability for privacy.

Code accompanying:

> **De-Anonymizing and $k$-Anonymizing Individuals using Subject-Object Knowledge Relations**
> *Computers & Security* (under review, Ms. Ref. No. COSE-D-25-05680)
> LIT Secure and Correct Systems Lab, Johannes Kepler University Linz

`harness.py` regenerates **every number in Section 6 of the paper** from a single command.

---

## Introduction

Data increasingly takes the form of complex networks. This creates opportunities for analysis and, at the same time, exposes individuals to structural re-identification. Two objectives pull in opposite directions:

1. **De-anonymization.** Identifying individuals from their access patterns, which supports collusion detection, tracing intellectual property leakage, and identifying compromised accounts. The target property is a **Strong Unique Neighbourhood Network (UNN)**: no subject's neighbourhood is a subset of another's.

2. **$k$-anonymization.** Protecting privacy by making each subject's neighbourhood identical to at least $k-1$ others, so that subjects are indistinguishable within their equivalence class.

The two cannot hold at once. The headline finding is that on real data their costs are sharply asymmetric: accountability is nearly free, privacy is not.

**Scope.** This is an empirical and conceptual quantification of the privacy-accountability trade-off. It is **not** a claim of practical superiority over stronger anonymization mechanisms such as $k$-automorphism, $k$-isomorphism, or differential privacy for graphs. Those are not implemented here and are not compared against.

---

## Problem Formulation

Both objectives are formalised within one bipartite graph model:

* **Subjects** ($V_1$): individuals, users, or entities.
* **Objects** ($V_2$): resources, files, or questions accessed by subjects.
* **Edges** ($E$): a subject's access or relation to an object.

The task is to add a minimal number of edges (and, only when unavoidable, synthetic objects) so that the graph satisfies either the Strong UNN property or $k$-neighbourhood anonymity.

---

## Algorithms Implemented

Everything lives in `harness.py`. Graphs are plain `dict[str, set[str]]`, so the hot path is set operations with no graph-library overhead.

| Function | Purpose |
|---|---|
| `deanon_min_graph` | **DeAnonMinGraph.** Achieves Strong UNN by attaching the least-connected object that breaks containment, introducing a synthetic object only when no existing object can. |
| `kanon_homogenize` | **kAnonHomogenize.** Enforces $k$-neighbourhood anonymity by merging the smallest equivalence classes and assigning each member the union of the group's neighbourhoods. |
| `kanon_homogenize_overlap` | **Overlap-aware variant.** Chooses merge partners to minimise the size of the generalised neighbourhood rather than merging by class size. Order-invariant, and substantially cheaper on graphs with role structure. |
| `k_degree_anonymize` | **$k$-degree baseline** after Liu & Terzi (2008), restricted to edge addition: sort degrees descending, partition into blocks of size $k$ to $2k-1$ by dynamic programming, raise each member to the block maximum. Monotone non-decreasing in $k$. |
| `kanon_degree_lower_bound` | Exact DP lower bound for the degree-only relaxation. Since $k$-neighbourhood anonymity implies $k$-degree anonymity, this is a valid lower bound on the cost of the full problem. |
| `js_utility` | Utility score $1 - \mathrm{JSD}$ over degree histograms. |
| `mean_jaccard` | Mean pairwise Jaccard similarity of subject neighbourhoods. |
| `verify_k_anonymity`, `verify_strong_unn` | Post-condition checks, run on every configuration. |
| `surrogate_from` | Builds a releasable synthetic stand-in matching a private graph's degree sequence and mean overlap. |

### On the utility metric

Utility is $1 - \mathrm{JSD}(\deg_{G'}, \deg_G)$, the Jensen-Shannon distance between normalised degree histograms in base 2. Boundedness is the reason for this choice. JSD lies in $[0,1]$, so an unmodified graph scores 1.000 and the worst case scores 0.000, with everything in between meaningfully ordered. A score built on Kullback-Leibler divergence would be unbounded above and would need clipping, at which point every heavily modified graph collapses to the same value and a degraded degree distribution becomes indistinguishable from a destroyed one. Several configurations below sit in exactly that range, so the distinction is not academic.

### On neighbourhood overlap

`mean_jaccard` is not a diagnostic afterthought. Across controlled experiments it is the structural property that governs anonymization cost, more so than $|V_1|$, $|V_2|$, or density. Computing it on a graph before attempting anonymization tells you which cost regime you are in.

---

## Getting Started

### Prerequisites

Python 3.9 or newer, and three packages:

```bash
pip install -r requirements.txt      # numpy, pandas, openpyxl
```

### Usage

```bash
git clone https://github.com/<your-username>/<your-repo>.git
cd <your-repo>
python3 harness.py --data test.xlsx --out results/
```

A full run takes a few minutes on a laptop.

| Flag | Effect |
|---|---|
| `--data PATH` | Input workbook (Nextcloud "Shared Folders Overview" export format) |
| `--out DIR` | Output directory for CSVs |
| `--seeds N` | Repetitions per configuration (default 30) |
| `--surrogate` | Also write a releasable synthetic stand-in for the input graph |

For an interactive walkthrough that renders each table and cross-checks the figures quoted in the manuscript, open `run_experiments.ipynb`. Inside Jupyter, pass arguments explicitly, since the kernel injects its own into `sys.argv`:

```python
import harness
harness.main(["--data", "test.xlsx", "--out", "results", "--seeds", "30"])
```

### Outputs

| File | Feeds |
|---|---|
| `dataset_meta.json` | Dataset description, Section 6.5 |
| `real_results.csv` | Table 5, all methods on the real graph, $k \in \{2,3,5,10\}$ |
| `ordering_sensitivity.csv` | Section 6.6, subject-order sensitivity over 30 permutations |
| `overlap_sweep.csv` | Table 6, cost against measured overlap, 8 levels $\times$ 30 seeds |
| `scalability.csv` | Table 7, both algorithms on a common ensemble up to 1600 subjects |

Edge counts, percentages and utility scores are deterministic and reproduce to the digit. Wall-clock times vary with hardware.

---

## Using your own data

The loader expects a Nextcloud "Shared Folders Overview" workbook. For any other source, build the graph directly and call the algorithms:

```python
import harness

G = {
    "alice": {"proj/specs", "proj/notes"},
    "bob":   {"proj/specs", "hr/policy"},
    "carol": {"hr/policy"},
}

print(f"{len(G)} subjects, {harness.n_edges(G)} edges, "
      f"mean Jaccard {harness.mean_jaccard(G):.3f}")

anon, added = harness.kanon_homogenize_overlap(G, k=2)
print(f"+{added} edges, k-anonymous: {harness.verify_k_anonymity(anon, 2)}")

uniq, edges, nodes = harness.deanon_min_graph(G)
print(f"+{edges} edges, +{nodes} synthetic objects, "
      f"strong UNN: {harness.verify_strong_unn(uniq)}")
```

Both algorithms are quadratic in the number of subjects, which puts the practical ceiling at a few thousand.

---

## The dataset is not included

Table 5 uses a sharing graph exported from a live Nextcloud file server (37 subjects, 1824 objects, 5484 edges). That export contains personal data of identifiable individuals in both the account names and the folder paths. It is **not** released, and no pseudonymised version is released either, because pseudonymising the account column would not remove the names embedded in the paths.

Run with `--surrogate` to generate a synthetic stand-in that matches the source graph's subject count, object count, **exact degree sequence**, and mean pairwise Jaccard similarity.

**The surrogate does not reproduce Table 5.** At $k=3$ it requires 140.8% additional edges against the real graph's 122.6%; at $k=2$, 89.9% against 49.7%. Mean overlap predicts cost well *within* a graph family but does not determine it *across* families, because higher-order structure such as the nesting of small neighbourhoods inside large ones also matters. Use the surrogate to exercise the code, not to check Table 5.

Everything in Tables 6 and 7 is fully synthetic and reproduces exactly.

---

## Repository layout

```
harness.py             implementation and all experiments
run_experiments.ipynb  notebook driver with per-table output
results/               CSVs behind the numbers reported in the paper
```

The CSVs under `results/` are the exact outputs behind the submitted manuscript, so the figures can be checked without rerunning anything.

---

## Analysis and Results

### The asymmetry

On the real access control graph, with mean pairwise Jaccard similarity $\bar{J} = 0.029$:

| Method | $k$ | Edges added | Increase | Utility |
|---|---|---|---|---|
| `DeAnonMinGraph` | --- | 28 | 0.5% | 0.423 |
| `kAnonHomogenize` | 3 | 6726 | 122.6% | 0.085 |
| `kAnonHomogenize-Overlap` | 3 | 6399 | 116.7% | 0.113 |
| $k$-Degree (lower bound) | 3 | 1700 | 31.0% | 0.438 |

Making every subject structurally unique costs half a percent of the graph. Making every subject 3-anonymous more than doubles it. The reason is visible in $\bar{J}$: because neighbourhoods barely overlap, the graph is already close to a Strong UNN, so de-anonymization has little to do, while homogenising any group of three subjects means giving each the union of three nearly disjoint folder sets.

The $k$-degree column is cheaper but defends against a strictly weaker adversary. Two subjects can share a degree while accessing entirely disjoint resources, so degree-based indistinguishability offers no protection against an adversary who observes neighbourhood content.

### Sensitivity to $k$

| $k$ | `kAnonHomogenize` | Utility |
|---|---|---|
| 2 | 49.7% | 0.238 |
| 3 | 122.6% | 0.085 |
| 5 | 189.4% | 0.054 |
| 10 | 604.0% | 0.000 |

Cost grows faster than linearly, because a group of $k$ subjects receives the union of $k$ neighbourhoods. There is no regime in this data where a large $k$ is affordable.

### What governs the cost

Across synthetic graphs where neighbourhood overlap is a controlled parameter, the overlap-aware merge falls from **167.1% to 71.9%** as $\bar{J}$ rises from 0.05 to 0.12, with no two consecutive 95% bootstrap intervals overlapping. The overlap-agnostic merge falls only from 176.5% to 165.1%: it is largely blind to the structure that would make anonymization cheap.

The cost of $k$-neighbourhood anonymity is therefore governed by neighbourhood overlap rather than by $k$-anonymity as a privacy model. As the surrogate result shows, $\bar{J}$ screens for the expensive regime rather than predicting the exact cost.

### Scalability

Both algorithms run on the same graph ensemble, so their costs are directly comparable. A least-squares log-log fit over the four largest sizes gives an exponent of **2.21**, consistent with and slightly above the $O(|V_1|^2)$ bound. The fit is restricted deliberately: at 50 and 100 subjects the computation takes single-digit milliseconds, and an exponent estimated from endpoint ratios moves between 1.98 and 2.16 across repeated runs on the same machine.

The practical ceiling is a few thousand subjects (8.5 s at 1600). Beyond that, incremental update algorithms, parallelism, or approximation would be needed.

### Known limitations

* One real dataset of 37 subjects is a narrow empirical base. Evaluation on public role-mining benchmarks is the natural next step.
* No comparison against $k$-automorphism, $k$-isomorphism, or differentially private graph release. A partially correct implementation of those would produce a comparison we could not defend, so they are reported as not applicable rather than approximated.
* Static snapshots only. Dynamic graphs where subjects gain and lose access over time are not addressed.
* `kAnonHomogenize` is sensitive to subject processing order (mean 10111.8 edges over 30 random permutations at $k=3$, 95% CI [9554.4, 10648.9]). Prefer the overlap-aware variant, which returns an identical result under every permutation.

---

## Citation

```bibtex
@article{deanon_kanon_2026,
  title   = {De-Anonymizing and $k$-Anonymizing Individuals using
             Subject-Object Knowledge Relations},
  author  = {...},
  journal = {Computers \& Security},
  year    = {2026},
  note    = {Under review}
}
```

## Contributing

Issues and pull requests are welcome. If you report a discrepancy in the numbers, please include the CSVs produced by your run and the output of `harness.main` for the affected configuration.

## Licence

The Nextcloud dataset is not distributed under any licence, because it is not distributed at all.
