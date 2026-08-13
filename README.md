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
is nothing to install to run `fetch-sequences`, `prep-msa`, and `rank`:

```bash
git clone <your-repo-url>
cd interactor-triage
chmod +x run.sh
```





