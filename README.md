# interactor
[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/USERNAME/interactor/blob/main/notebooks/interactor_triage.ipynb)

structural screening of proximity-labeling candidate interactors. it is a command-line pipeline with four stages, each a separate subcommand, chained by files on disk so any stage can be rerun independently:

```
fetch-sequences  →  prep-msa  →  fold  →  rank
```
---
## overview
Living organisms are complex systems composed of interacting components and overlapping biochemical pathways that sustain life. One way to understand these processes is to study these interactions in the molecular environment of live cells. Merging proximity labeling with quantitative proteomics maps protein-protein interactions (PPIs) and local protein neighborhoods within their native cellular context. This approach relies on interactions associated with a bait protein or a protein of interest (POI). PL strategies transiently generate short-lived, highly reactive species within a specific radius of a POI, labeling proximal proteins with an affinity tag, such as biotin, in a distance-dependent manner. PL experiments provide a snapshot of the neighborhood proteome as well as direct and indirect protein-protein interactions. Recently, various PL-based interactome mapping methods (BioID, APEX, TurboID, MultiMap, etc.) using engineered enzymes or synthetic photocatalysts have been widely applied in cultured cells.

However, one bottleneck is that even after filtering samples against negative controls, a final protein candidate list can still include nonspecific background (abundant cytoskeletal or cytosolic proteins that show up in nearly any experiment). Before spending low-throughput validation on every candidate, this tool checks whether a physically plausible bound complex exists between the bait and the protein candidate. AlphaFold-Multimer's ipTM (interface predicted TM-score) is a widely used computational proxy for that.

## scope
Interactor is built for finalizing a proximity-labeling candidate list before low-throughput experimental validation. It can flag candidates with no plausible predicted interface and can provide a quality check for known positive-control interactors. 


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
           │   (paired bait: candidate, ":"-joined; no GPU —
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

If you'd rather run it in a terminal instead of Colab (e.g., you have a local GPU or want to script it), the core stages (`fetch-sequences`,
`prep-msa`, `rank`) use only the Python standard library — nothing to `pip install`.

``` bash
git clone https://github.com/USERNAME/interactor.git
cd interactor
chmod +x run.sh

./run.sh fetch-sequences --candidates sample_data/cd70_candidates.csv --bait CD70
./run.sh prep-msa
./run.sh fold      # packages a ColabFold job; add --local to fold with colabfold_batch
./run.sh rank      # -> work/ranked_report.csv
```

The `fold` stage is the only GPU-bound step. By default, it packages `work/fold_job.zip` for the free ColabFold notebook; with `--local` it
runs `colabfold_batch` if you've installed it (`pip install "colabfold[alphafold]"`) and have a CUDA GPU.

See the ranking immediately, without folding anything, on precomputed
scores:

``` bash
./run.sh rank --from-scores sample_data/sample_scores.csv
```

</details>

## limitations
 This tool assumes an already finalized list; therefore, it does not statistically filter raw spectral counts. It does not provide proof of interaction. For example, high ipTM scores indicate plausibility but not evidence of presence in cells. Not a replacement for experimental validation.

## references

-  Leung, K. K., Schaefer, K., Lin, Z., Yao, Z. & Wells, J. A. Engineered Proteins and Chemical Tools to Probe the Cell Surface Proteome. *Chem. Rev.* 125,       4069–4110 (2025).
- Jumper, J. et al. Highly accurate protein structure prediction with AlphaFold. *Nature* 596, 583–589 (2021).
- Evans, R. et al. Protein complex prediction with AlphaFold-Multimer. *bioRxiv* (2021).
- Mirdita, M. et al. ColabFold: making protein structure prediction accessible to all. *Nature Methods* 19, 679–682 (2022).
- Steinegger, M. & Söding, J. MMseqs2 enables sensitive protein sequence searching. *Nature Biotechnology* 35, 1026–1028 (2017).
- The UniProt Consortium. UniProt: the Universal Protein Knowledgebase in 2023. *Nucleic Acids Research* 51, D523–D531 (2023).





