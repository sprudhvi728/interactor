# interactor
[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/USERNAME/interactor/blob/main/notebooks/interactor_triage.ipynb)

structural screening of proximity-labeling candidate interactors. it is a command-line pipeline with four stages, each a separate subcommand, chained by files on disk so any stage can be rerun independently:

```
fetch-sequences  →  prep-msa  →  fold  →  rank
```
---
## overview
Living organisms are complex systems composed of interacting components and overlapping biochemical pathways that sustain life. One way to gain insight into these processes is to study these interactions in the molecular environment of live cells. Merging proximity labeling with quantitative proteomics maps protein-protein interactions (PPIs) and local protein neighborhoods within their native cellular context. This approach relies on interactions associated with a bait protein or a protein of interest (POI). PL strategies rely on transiently generating short-lived, highly reactive species within a specific radius of a POI to label proximal proteins with an affinity tag, such as biotin, in a distance-dependent manner. PL experiments provide a snapshot of the neighborhood proteome and protein-protein interactions. 
Recently, various PL-based interactome mapping methods (BioID, APEX, TurboID, MultiMap, etc.) using engineered enzymes or synthetic photocatalysts have been widely applied in cultured cells. 

However, one bottleneck is that even after filtering samples against negative controls, a final protein candidate list can still include chronic background (abundant cytoskeletal or cytosolic proteins that show up in nearly any experiment). Before spending low-throughput validation on every candidate, this tool checks whether a physically plausible bound complex exists between the bait and the protein candidate. AlphaFold-Multimer's ipTM (interface predicted TM-score) is a widely used computational proxy for that. 

## scope
Interactor is built for finalizing a proximity-labeling candidate list before low-throughput experimental validation. It can flag candidates with no plausible predicted interface and can provide a quality check for known postive-control interactors. 


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
   bait: candidate pair, and rank the results.
5. When it finishes, it prints a tiered table and downloads **`ranked_report.csv`**.

> **NOTE:** Test just a few protein candidates at a time. Folding takes a few minutes per pair, and free Colab disconnects
> after a few hours. Consider using Colab Pro or a local GPU for a large list.

## limitations
 This tool assumes an already-finalized list; therefore it does not provide any statistical filtering of raw spectral counts. It does not provide proof of interaction. For example, high ipTM scores indicate plausibility but not evidence of presence in cells. Not a replacement for experimental validation.






