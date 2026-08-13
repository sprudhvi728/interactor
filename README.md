# interactor
[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/USERNAME/interactor/blob/main/notebooks/interactor_triage.ipynb)

structural screening of proximity-labeling candidate interactors. it is a command-line pipeline with four stages, each a separate subcommand, chained by files on disk so any stage can be rerun independently:

```
fetch-sequences  →  prep-msa  →  fold  →  rank
```
---

## workflow

```text
   candidate list (CSV or pasted)              interactor
   + bait  (gene name / UniProt ID)                 │
           │                                        │
           ▼                                        │
   fetch-sequences ──── UniProt REST ───► work/sequences/*.fasta + manifest.json
           │
           ▼
   prep-msa ───────────────────────────► work/queries/{candidate}.fasta
           │   (paired bait:candidate, ":"-joined; no GPU —
           │    the MSA is fetched by colabfold_batch at fold time)
           ▼
   fold
           │
           ├── default ────────────────► work/fold_job.zip
           │                              (upload to the free ColabFold notebook,
           │                               Colab GPU, manual download step)
           │
           └── --local ────────────────► colabfold_batch (JAX, needs a
                                           local CUDA GPU)
           │
           ▼
   work/predictions/*_scores_rank_001*.json   (ipTM, pTM per candidate)
           │
           ▼
   rank ──► work/ranked_report.csv
            tiers: High-confidence (ipTM ≥ 0.8) / Ambiguous (0.6–0.8) / Unlikely (< 0.6)
```

---

## requirements

- Python 3.10+
- Internet connection (UniProt REST API, ColabFold MSA server)

## installation
The core pipeline uses only the Python standard library (Python 3.8+), so there
is nothing to install to run `fetch-sequences`, `prep-msa`, and `rank`.

Everything runs in the browser on a free Colab GPU. You never touch a terminal.

1. Click **[Open in Colab](https://colab.research.google.com/github/USERNAME/interactor/blob/main/notebooks/interactor_triage.ipynb)** (or go to
   [colab.research.google.com](https://colab.research.google.com) → **File → Upload
   notebook** → choose `notebooks/interactor_triage.ipynb`).
2. Turn on the GPU: **Runtime → Change runtime type → T4 GPU → Save**.
3. In the first cell (**Inputs**), set your bait and paste your candidate list —
   gene names or UniProt IDs, one per line:
   ```python
   BAIT = "CD70"
   CANDIDATES = """
   CD27
   TRAF2
   TRAF3
   """
   ```
4. **Runtime → Run all.** The cells install ColabFold, fetch sequences, fold each
   bait:candidate pair, and rank the results.
5. When it finishes it prints a tiered table and downloads **`ranked_report.csv`**.

> **Start small.** Folding takes a few minutes per pair and free Colab disconnects
> after a few hours. Try 3–5 candidates first to confirm it works, then scale up
> (Colab Pro or a local GPU for a large list).







