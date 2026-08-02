# interactor
structural screening of proximity-labeling candidate interactors. it is a command-line pipeline with four stages, each a separate subcommand, chained by files on disk so any stage can be rerun independently:

```
fetch-sequences  →  prep-msa  →  fold  →  rank
```
---

## workflow

```
candidate list (CSV)                    Interactor
+ bait gene/UniProt ID                        │
        │                                     │
        ▼                                     │
  fetch-sequences ──── UniProt REST ───► sequences/*.fasta + manifest.csv
        │
        ▼
   prep-msa ──── ColabFold MSA server ───► msa/{bait}_{candidate}.a3m
        │        (api.colabfold.com, MMseqs2, no GPU)
        ▼
     fold
        │
        ├── --backend colab (default) ──► zip for the official ColabFold
        │                                  batch notebook (free Colab GPU,
        │                                  manual upload/download step)
        │
        └── --backend local ────────────► colabfold_batch (JAX, needs a
                                            local CUDA GPU)
        │
        ▼
  fold_results/*_scores_rank_001*.json  (ipTM, pTM, pLDDT per candidate)
        │
        ▼
      rank ──► ranked_report.csv
               tiers: High-confidence (ipTM ≥ 0.8) / Ambiguous (0.6–0.8) / Unlikely (< 0.6)
```

---

## Requirements

- Python 3.10+
- Internet connection (UniProt REST API, ColabFold MSA server, and — for the default `fold` backend — Google Colab)


