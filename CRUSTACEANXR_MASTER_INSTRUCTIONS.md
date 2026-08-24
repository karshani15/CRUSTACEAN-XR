# ╔══════════════════════════════════════════════════════════════════════════════╗
# ║        CrustaceanXR — MASTER INSTRUCTION & WORKFLOW DOCUMENT               ║
# ║  PROJECT: Multi-Evidence Computational Framework for Crustacean Allergen   ║
# ║           Epitope-Level Cross-Reactivity Prediction                        ║
# ║  STUDENT : Karshani Tiwary  |  ID: 7024000056                              ║
# ║  VERSION : 1.0  (Self-Replicating — read §OMEGA before starting)           ║
# ╚══════════════════════════════════════════════════════════════════════════════╝

---

## ⚠ READ THIS FIRST — THE OMEGA CLAUSE (Self-Replication Protocol)

> **This document is a never-ending loop.**
> When this chat becomes too long (token-burn warning, context degradation, or
> the user simply asks for a "new MD"), you MUST:
>
> 1. Read this entire file top-to-bottom.
> 2. Append all new work done since the last version (errors found, commands run,
>    results observed, decisions made) to **Section 13 — Running Session Log**.
> 3. Update **Section 14 — Current Status Snapshot** to reflect exactly where
>    the pipeline stands right now.
> 4. Increment the VERSION field in the header.
> 5. Write the new file as `CRUSTACEANXR_MASTER_v{N}.md` in `~/CrustaceanXR/logs/`.
> 6. The new file IS this file — same structure, same OMEGA clause, everything
>    carried forward, nothing lost.
>
> **The new Claude instance must treat this document as its sole ground truth.
> Never start fresh. Never ask the user to repeat context. The MD is the memory.**

---

## § 0 — OPERATING PRINCIPLES (Read before every task)

### 0.1 Token Economy Rules
- **Think before you run.** Before executing any command, state in one line what
  you expect it to return and why. If the expectation is wrong, that IS the bug.
- **No search-first reflex.** Only web-search when a specific version number,
  an exact flag, or a tool's availability is genuinely unknown. Prefer `--help`,
  `man`, and `--version` locally first.
- **Diagnose before you fix.** Read error messages completely. Identify the root
  cause before proposing any fix. Never shotgun-patch.
- **One fix at a time.** Apply one change, verify it, log it, then move on.
- **No redundant re-runs.** If a step succeeded and its output files exist, skip
  it. Check with `ls -lh` or `wc -l` before rerunning anything.
- **Minimal output.** Prefer `head -20`, `tail -20`, `grep`, `wc` over `cat`
  on large files. Never dump entire FASTAs or alignment files to terminal.

### 0.2 Error Resolution Protocol
```
STEP 1 — READ  : Read the full error message, identify the exact failing line.
STEP 2 — LOCATE: Determine whether it is an environment issue, a path issue,
                  a data issue, or a logic issue.
STEP 3 — REASON: State the probable cause in one sentence before touching anything.
STEP 4 — FIX   : Apply the minimal targeted fix.
STEP 5 — VERIFY: Confirm the fix worked with a direct verification command.
STEP 6 — LOG   : Write to the phase log AND to logpole.txt immediately.
```

### 0.3 Logging Rules (Non-Negotiable)
- Every command run → logged.
- Every error seen → logged with full stderr, root cause, and fix applied.
- Every result → logged with file path, line count or size, and one-line
  interpretation.
- Every decision (tool choice, parameter choice, skip) → logged with rationale.
- **logpole.txt** receives a summary entry after every phase completes.
- Log format: `[YYYY-MM-DD HH:MM] [PHASE-N] [STATUS: OK|ERROR|SKIP] message`

### 0.4 Environment Assumptions
- OS: WSL2 (Ubuntu 22.04 recommended)
- Shell: bash
- Package manager: Conda/Mamba (environment: `crustaceanxr`)
- Working root: `~/CrustaceanXR/`
- GPU: optional (ColabFold local) — if unavailable, use ColabFold web server
  and download results manually
- All commands in this document are **direct bash one-liners or here-doc
  blocks** — paste and run, no save-and-execute steps

---

## § 1 — PROJECT SCOPE SUMMARY

**Project name:** CrustaceanXR
**Formal title:** A Multi-Evidence Computational Framework for Predicting
Epitope-Level Cross-Reactivity in Crustacean Shellfish Allergens

### Scientific Objective
Build a reproducible, benchmarked pipeline that scores IgE-mediated
cross-reactivity between crustacean shellfish allergens (shrimp, crab,
lobster, crayfish) and a comparator set (mollusks, dust mite, cockroach,
Big 8/9 allergens) by fusing five computational layers into one feature
vector per protein pair, then benchmarking a rule-based weighted index
against a supervised ML classifier (RF / XGBoost / SVM).

### Allergen Families in Scope
| Family | Key proteins | Species (examples) |
|---|---|---|
| **Crustacean (primary)** | Tropomyosin, Arginine kinase, SCP, MLC, Troponin C, TPI | Penaeus, Scylla, Homarus, Panulirus, Procambarus |
| **Comparator set** | Homologs of above | Mollusk (squid, oyster, mussel), Dust mite (Der p 10), Cockroach (Per a 7, Bla g 7), Big 8/9 |

### Five Computational Layers (Feature Vector Components)
1. Sequence identity — FAO/WHO 80-aa sliding window (>35% flag), exact 6/8-mer
2. Domain/motif conservation — Pfam/HMMER, InterProScan, PROSITE
3. Structural similarity — AlphaFold2/ColabFold → TM-align/DALI (TM-score, RMSD)
4. Epitope overlap — BepiPred3/ABCpred/SVMTriP (linear B-cell); DiscoTope2/ElliPro
   (conformational); NetMHCIIpan-4.1 (T-cell); consensus ≥2 tools
5. Docking plausibility — HADDOCK/ClusPro/ZDOCK + PRODIGY/FoldX (top candidates only)

### ML Layer
Random Forest, XGBoost, SVM — trained on literature-curated
positive/negative pairs; stratified k-fold CV; ROC-AUC + precision/recall;
permutation testing + Benjamini-Hochberg correction.

### Out of Scope
Vaccine design, immunotherapy formulation, therapeutic development.

---

## § 2 — DIRECTORY STRUCTURE

Run this **once** to create the full working tree:

```bash
mkdir -p ~/CrustaceanXR/{data/{raw/{sequences,structures},processed/{fasta,structures,epitopes},comparator},envs,scripts/{phase1_data,phase2_seq,phase3_struct,phase4_epitope,phase5_scoring,phase6_docking,phase7_ml,phase8_stats},results/{alignments,domains,structures,epitopes,scoring,docking,ml,stats,figures},logs/{phase1,phase2,phase3,phase4,phase5,phase6,phase7,phase8},workflow,docs} && touch ~/CrustaceanXR/logs/logpole.txt && echo "[$(date '+%Y-%m-%d %H:%M')] [INIT] [STATUS: OK] Directory tree created." >> ~/CrustaceanXR/logs/logpole.txt && echo "Directory tree created successfully."
```

### Tree Overview
```
~/CrustaceanXR/
├── data/
│   ├── raw/
│   │   ├── sequences/          ← downloaded FASTAs (unprocessed)
│   │   └── structures/         ← PDB/CIF files (experimental)
│   ├── processed/
│   │   ├── fasta/              ← cleaned, renamed, deduplicated FASTAs
│   │   ├── structures/         ← AF2/ColabFold models + QC-passed PDBs
│   │   └── epitopes/           ← per-tool epitope prediction outputs
│   └── comparator/             ← mollusk, dust mite, cockroach, Big8/9
├── envs/                       ← conda YAML files
├── scripts/
│   ├── phase1_data/
│   ├── phase2_seq/
│   ├── phase3_struct/
│   ├── phase4_epitope/
│   ├── phase5_scoring/
│   ├── phase6_docking/
│   ├── phase7_ml/
│   └── phase8_stats/
├── results/
│   ├── alignments/
│   ├── domains/
│   ├── structures/
│   ├── epitopes/
│   ├── scoring/
│   ├── docking/
│   ├── ml/
│   ├── stats/
│   └── figures/
├── logs/
│   ├── logpole.txt             ← MASTER LOG (never delete)
│   ├── phase1/phase1.log
│   ├── phase2/phase2.log
│   ├── phase3/phase3.log
│   ├── phase4/phase4.log
│   ├── phase5/phase5.log
│   ├── phase6/phase6.log
│   ├── phase7/phase7.log
│   └── phase8/phase8.log
├── workflow/                   ← Snakemake/Nextflow files
└── docs/                       ← manuscript, figures, supplementary
```

---

## § 3 — CONDA ENVIRONMENT SETUP

### 3.1 Create base environment

```bash
conda create -n crustaceanxr python=3.10 -y && conda activate crustaceanxr && echo "[$(date '+%Y-%m-%d %H:%M')] [ENV] [STATUS: OK] Base env crustaceanxr created." >> ~/CrustaceanXR/logs/logpole.txt
```

### 3.2 Install core bioinformatics packages

```bash
conda activate crustaceanxr && mamba install -c bioconda -c conda-forge -y muscle mafft clustalw emboss hmmer interproscan biopython snakemake tmalign freesasa && echo "[$(date '+%Y-%m-%d %H:%M')] [ENV] [STATUS: OK] Core bioinformatics packages installed." >> ~/CrustaceanXR/logs/logpole.txt
```

### 3.3 Install ML and data science packages

```bash
conda activate crustaceanxr && pip install scikit-learn xgboost pandas numpy scipy matplotlib seaborn plotly networkx && echo "[$(date '+%Y-%m-%d %H:%M')] [ENV] [STATUS: OK] ML/data packages installed." >> ~/CrustaceanXR/logs/logpole.txt
```

### 3.4 Verify critical tools

```bash
conda activate crustaceanxr && for tool in muscle mafft hmmer needle water python; do which $tool && echo "$tool: OK" || echo "$tool: MISSING"; done 2>&1 | tee -a ~/CrustaceanXR/logs/logpole.txt
```

### 3.5 Export environment for reproducibility

```bash
conda activate crustaceanxr && conda env export > ~/CrustaceanXR/envs/crustaceanxr.yml && echo "[$(date '+%Y-%m-%d %H:%M')] [ENV] [STATUS: OK] Environment exported to envs/crustaceanxr.yml" >> ~/CrustaceanXR/logs/logpole.txt
```

---

## § 4 — PHASE 1: DATA ACQUISITION AND CURATION (Weeks 1–3)

### 4.1 Objective
Download all crustacean allergen sequences + comparator set from AllergenOnline,
IUIS, UniProt, NCBI Entrez; retrieve PDB structures; download AlphaFold DB models
for sequences lacking experimental structures; deduplicate and normalize.

### 4.2 Log initialization

```bash
echo "[$(date '+%Y-%m-%d %H:%M')] [PHASE-1] [STATUS: START] Data acquisition started." >> ~/CrustaceanXR/logs/logpole.txt && echo "[$(date '+%Y-%m-%d %H:%M')] [PHASE-1] [STATUS: START] Data acquisition started." >> ~/CrustaceanXR/logs/phase1/phase1.log
```

### 4.3 Download from NCBI Entrez (Biopython Entrez)
Target proteins: tropomyosin, arginine kinase, sarcoplasmic calcium-binding
protein, myosin light chain, troponin C, triosephosphate isomerase.
Target taxa: Penaeus, Scylla, Homarus, Panulirus, Procambarus.

```bash
conda activate crustaceanxr && python3 - <<'PYEOF'
from Bio import Entrez, SeqIO
import os, time

Entrez.email = "your_email@institution.ac.in"   # REPLACE THIS

queries = {
    "tropomyosin_crustacean": "tropomyosin[Title] AND (Penaeus[Organism] OR Scylla[Organism] OR Homarus[Organism] OR Panulirus[Organism] OR Procambarus[Organism])",
    "arginine_kinase_crustacean": "arginine kinase[Title] AND (Penaeus[Organism] OR Scylla[Organism] OR Homarus[Organism])",
    "SCP_crustacean": "sarcoplasmic calcium binding protein[Title] AND (Penaeus[Organism] OR Scylla[Organism])",
    "MLC_crustacean": "myosin light chain[Title] AND (Penaeus[Organism] OR Scylla[Organism])",
    "troponin_crustacean": "troponin C[Title] AND (Penaeus[Organism] OR Scylla[Organism])",
    "TPI_crustacean": "triosephosphate isomerase[Title] AND (Penaeus[Organism] OR Scylla[Organism])",
    "comparator_dustmite": "Der p 10[Title] OR (tropomyosin[Title] AND Dermatophagoides[Organism])",
    "comparator_cockroach": "(Per a 7[Title] OR Bla g 7[Title]) AND (Periplaneta[Organism] OR Blattella[Organism])",
}

outdir = os.path.expanduser("~/CrustaceanXR/data/raw/sequences")
log = os.path.expanduser("~/CrustaceanXR/logs/phase1/phase1.log")

with open(log, "a") as lf:
    for label, query in queries.items():
        handle = Entrez.esearch(db="protein", term=query, retmax=50)
        record = Entrez.read(handle); handle.close()
        ids = record["IdList"]
        lf.write(f"[PHASE-1] Query '{label}': {len(ids)} records found\n")
        print(f"{label}: {len(ids)} records")
        if not ids:
            lf.write(f"[PHASE-1][WARN] No records for {label} — check query\n")
            continue
        fetch = Entrez.efetch(db="protein", id=ids, rettype="fasta", retmode="text")
        fasta_path = os.path.join(outdir, f"{label}.fasta")
        with open(fasta_path, "w") as fh:
            fh.write(fetch.read())
        fetch.close()
        lf.write(f"[PHASE-1][OK] Saved {fasta_path}\n")
        time.sleep(0.4)

print("Download complete. Check logs/phase1/phase1.log")
PYEOF
echo "[$(date '+%Y-%m-%d %H:%M')] [PHASE-1] [STATUS: OK] NCBI Entrez download script completed." >> ~/CrustaceanXR/logs/logpole.txt
```

### 4.4 Verify downloads

```bash
ls -lh ~/CrustaceanXR/data/raw/sequences/ && grep -c "^>" ~/CrustaceanXR/data/raw/sequences/*.fasta 2>/dev/null | tee -a ~/CrustaceanXR/logs/phase1/phase1.log
```

### 4.5 Deduplicate and normalize FASTA headers

```bash
conda activate crustaceanxr && python3 - <<'PYEOF'
from Bio import SeqIO
import os, re, hashlib

indir  = os.path.expanduser("~/CrustaceanXR/data/raw/sequences")
outdir = os.path.expanduser("~/CrustaceanXR/data/processed/fasta")
log    = os.path.expanduser("~/CrustaceanXR/logs/phase1/phase1.log")

seen_hashes = set()
dup_count = 0
total = 0

with open(log, "a") as lf:
    for fname in sorted(os.listdir(indir)):
        if not fname.endswith(".fasta"):
            continue
        label = fname.replace(".fasta", "")
        out_records = []
        for i, rec in enumerate(SeqIO.parse(os.path.join(indir, fname), "fasta")):
            total += 1
            seq_hash = hashlib.md5(str(rec.seq).encode()).hexdigest()
            if seq_hash in seen_hashes:
                dup_count += 1
                lf.write(f"[PHASE-1][DUP] {rec.id} in {fname} — skipped\n")
                continue
            seen_hashes.add(seq_hash)
            # Normalize header: label_index_accession
            acc = rec.id.split("|")[1] if "|" in rec.id else rec.id.split()[0]
            rec.id = f"{label}_{i+1}_{acc}"
            rec.description = ""
            out_records.append(rec)
        out_path = os.path.join(outdir, f"{label}_clean.fasta")
        SeqIO.write(out_records, out_path, "fasta")
        lf.write(f"[PHASE-1][OK] {label}: {len(out_records)} unique sequences → {out_path}\n")
        print(f"{label}: {len(out_records)} unique")

    lf.write(f"[PHASE-1][SUMMARY] Total: {total} | Duplicates removed: {dup_count}\n")
    print(f"Total: {total} | Duplicates removed: {dup_count}")
PYEOF
echo "[$(date '+%Y-%m-%d %H:%M')] [PHASE-1] [STATUS: OK] Deduplication and header normalization complete." >> ~/CrustaceanXR/logs/logpole.txt
```

### 4.6 Download PDB structures (where available)

```bash
conda activate crustaceanxr && python3 - <<'PYEOF'
# Known crustacean allergen PDB IDs (expand this list as needed)
from Bio.PDB import PDBList
import os

pdb_ids = [
    "1C1G",  # shrimp tropomyosin Pen a 1
    "2BCT",  # cockroach tropomyosin (comparator)
    # Add more as found in PDB search
]
outdir = os.path.expanduser("~/CrustaceanXR/data/raw/structures")
log    = os.path.expanduser("~/CrustaceanXR/logs/phase1/phase1.log")
pdbl = PDBList()
with open(log, "a") as lf:
    for pid in pdb_ids:
        try:
            pdbl.retrieve_pdb_file(pid, file_type="pdb", pdir=outdir)
            lf.write(f"[PHASE-1][OK] PDB {pid} downloaded.\n")
        except Exception as e:
            lf.write(f"[PHASE-1][ERROR] PDB {pid}: {e}\n")
            print(f"ERROR: {pid}: {e}")
PYEOF
echo "[$(date '+%Y-%m-%d %H:%M')] [PHASE-1] [STATUS: OK] PDB structure download attempted." >> ~/CrustaceanXR/logs/logpole.txt
```

### 4.7 Phase 1 completion log entry

```bash
echo "[$(date '+%Y-%m-%d %H:%M')] [PHASE-1] [STATUS: COMPLETE] Sequences in processed/fasta: $(ls ~/CrustaceanXR/data/processed/fasta/ | wc -l) files. PDB structures in raw/structures: $(ls ~/CrustaceanXR/data/raw/structures/ | wc -l) files." >> ~/CrustaceanXR/logs/logpole.txt && echo "[$(date '+%Y-%m-%d %H:%M')] [PHASE-1] [STATUS: COMPLETE] Phase 1 done." >> ~/CrustaceanXR/logs/phase1/phase1.log
```

---

## § 5 — PHASE 2: SEQUENCE-LEVEL ANALYSIS (Weeks 3–5)

### 5.1 Objective
MSA (MUSCLE + MAFFT cross-check), pairwise alignment (EMBOSS Needle/Water),
domain/motif profiling (Pfam-HMMER, InterProScan), physicochemical properties
(ProtParam via Biopython).

### 5.2 Log initialization

```bash
echo "[$(date '+%Y-%m-%d %H:%M')] [PHASE-2] [STATUS: START] Sequence analysis started." >> ~/CrustaceanXR/logs/logpole.txt && echo "[$(date '+%Y-%m-%d %H:%M')] [PHASE-2] [STATUS: START]" >> ~/CrustaceanXR/logs/phase2/phase2.log
```

### 5.3 Concatenate all processed FASTAs into one master file

```bash
cat ~/CrustaceanXR/data/processed/fasta/*_clean.fasta > ~/CrustaceanXR/data/processed/fasta/ALL_allergens.fasta && echo "Total sequences:" && grep -c "^>" ~/CrustaceanXR/data/processed/fasta/ALL_allergens.fasta | tee -a ~/CrustaceanXR/logs/phase2/phase2.log
```

### 5.4 Multiple Sequence Alignment — MUSCLE

```bash
muscle -align ~/CrustaceanXR/data/processed/fasta/ALL_allergens.fasta -output ~/CrustaceanXR/results/alignments/ALL_muscle.afa 2>&1 | tee -a ~/CrustaceanXR/logs/phase2/phase2.log && echo "[$(date '+%Y-%m-%d %H:%M')] [PHASE-2] [STATUS: OK] MUSCLE MSA complete." >> ~/CrustaceanXR/logs/logpole.txt
```

### 5.5 Multiple Sequence Alignment — MAFFT (cross-check)

```bash
mafft --auto --thread 4 ~/CrustaceanXR/data/processed/fasta/ALL_allergens.fasta > ~/CrustaceanXR/results/alignments/ALL_mafft.afa 2>&1 | tee -a ~/CrustaceanXR/logs/phase2/phase2.log && echo "[$(date '+%Y-%m-%d %H:%M')] [PHASE-2] [STATUS: OK] MAFFT MSA complete." >> ~/CrustaceanXR/logs/logpole.txt
```

### 5.6 Pairwise alignments — EMBOSS Needle (global)
Run all-vs-all pairwise global alignment and extract identity scores:

```bash
conda activate crustaceanxr && python3 - <<'PYEOF'
import os, subprocess
from Bio import SeqIO
from itertools import combinations

fasta = os.path.expanduser("~/CrustaceanXR/data/processed/fasta/ALL_allergens.fasta")
outdir = os.path.expanduser("~/CrustaceanXR/results/alignments/pairwise_needle")
log   = os.path.expanduser("~/CrustaceanXR/logs/phase2/phase2.log")
os.makedirs(outdir, exist_ok=True)

records = list(SeqIO.parse(fasta, "fasta"))
print(f"Running pairwise alignments for {len(records)} sequences...")

with open(log, "a") as lf:
    for i, rec_a in enumerate(records):
        for j, rec_b in enumerate(records):
            if j <= i:
                continue
            pair_id = f"{rec_a.id}__vs__{rec_b.id}"
            out_aln = os.path.join(outdir, f"{pair_id}.needle")
            # Write temp single-seq FASTAs
            fa = f"/tmp/seqa_{i}.fasta"; fb = f"/tmp/seqb_{j}.fasta"
            with open(fa, "w") as f: f.write(f">{rec_a.id}\n{rec_a.seq}\n")
            with open(fb, "w") as f: f.write(f">{rec_b.id}\n{rec_b.seq}\n")
            cmd = ["needle", "-asequence", fa, "-bsequence", fb,
                   "-gapopen", "10", "-gapextend", "0.5",
                   "-outfile", out_aln, "-aformat", "pair"]
            result = subprocess.run(cmd, capture_output=True, text=True)
            if result.returncode != 0:
                lf.write(f"[PHASE-2][ERROR] needle failed for {pair_id}: {result.stderr[:200]}\n")
            else:
                lf.write(f"[PHASE-2][OK] {pair_id} → {out_aln}\n")

print("Pairwise alignment done.")
PYEOF
echo "[$(date '+%Y-%m-%d %H:%M')] [PHASE-2] [STATUS: OK] Pairwise needle alignments complete." >> ~/CrustaceanXR/logs/logpole.txt
```

### 5.7 Extract identity scores and apply FAO/WHO 35% sliding window flag

```bash
conda activate crustaceanxr && python3 - <<'PYEOF'
import os, re
import pandas as pd

needle_dir = os.path.expanduser("~/CrustaceanXR/results/alignments/pairwise_needle")
outcsv     = os.path.expanduser("~/CrustaceanXR/results/alignments/pairwise_identity_matrix.csv")
log        = os.path.expanduser("~/CrustaceanXR/logs/phase2/phase2.log")

rows = []
for fname in os.listdir(needle_dir):
    if not fname.endswith(".needle"):
        continue
    pair = fname.replace(".needle", "")
    parts = pair.split("__vs__")
    if len(parts) != 2:
        continue
    a, b = parts
    with open(os.path.join(needle_dir, fname)) as fh:
        content = fh.read()
    ident_match = re.search(r"Identity:\s+\d+/\d+\s+\((\d+\.\d+)%\)", content)
    sim_match   = re.search(r"Similarity:\s+\d+/\d+\s+\((\d+\.\d+)%\)", content)
    if ident_match and sim_match:
        ident = float(ident_match.group(1))
        sim   = float(sim_match.group(1))
        flag  = "YES" if ident >= 35.0 else "NO"
        rows.append({"seq_a": a, "seq_b": b, "identity_pct": ident, "similarity_pct": sim, "FAOWHO_flag": flag})

df = pd.DataFrame(rows)
df.to_csv(outcsv, index=False)
flagged = df[df["FAOWHO_flag"] == "YES"]
with open(log, "a") as lf:
    lf.write(f"[PHASE-2][OK] Identity matrix saved: {outcsv}\n")
    lf.write(f"[PHASE-2][RESULT] Total pairs: {len(df)} | FAO/WHO flagged (≥35%): {len(flagged)}\n")
print(f"Total pairs: {len(df)} | FAO/WHO flagged: {len(flagged)}")
print(df.describe())
PYEOF
echo "[$(date '+%Y-%m-%d %H:%M')] [PHASE-2] [STATUS: OK] Identity matrix and FAO/WHO flags extracted." >> ~/CrustaceanXR/logs/logpole.txt
```

### 5.8 Domain profiling — HMMER vs Pfam

```bash
conda activate crustaceanxr && hmmpress ~/CrustaceanXR/data/raw/Pfam-A.hmm 2>&1 | tee -a ~/CrustaceanXR/logs/phase2/phase2.log && hmmscan --domtblout ~/CrustaceanXR/results/domains/pfam_scan.domtblout --cpu 4 ~/CrustaceanXR/data/raw/Pfam-A.hmm ~/CrustaceanXR/data/processed/fasta/ALL_allergens.fasta > ~/CrustaceanXR/results/domains/pfam_scan.out 2>&1 | tee -a ~/CrustaceanXR/logs/phase2/phase2.log && echo "[$(date '+%Y-%m-%d %H:%M')] [PHASE-2] [STATUS: OK] Pfam-HMMER domain scan complete." >> ~/CrustaceanXR/logs/logpole.txt
```

> **NOTE on Pfam-A.hmm:** Download from https://ftp.ebi.ac.uk/pub/databases/Pfam/current_release/Pfam-A.hmm.gz
> ```bash
> wget -c https://ftp.ebi.ac.uk/pub/databases/Pfam/current_release/Pfam-A.hmm.gz -P ~/CrustaceanXR/data/raw/ && gunzip ~/CrustaceanXR/data/raw/Pfam-A.hmm.gz
> ```

### 5.9 Phase 2 completion log

```bash
echo "[$(date '+%Y-%m-%d %H:%M')] [PHASE-2] [STATUS: COMPLETE] Alignments: $(ls ~/CrustaceanXR/results/alignments/pairwise_needle/ | wc -l) needle files. Identity matrix: $(wc -l < ~/CrustaceanXR/results/alignments/pairwise_identity_matrix.csv) rows. Pfam hits: $(grep -v '^#' ~/CrustaceanXR/results/domains/pfam_scan.domtblout 2>/dev/null | wc -l) domain hits." >> ~/CrustaceanXR/logs/logpole.txt
```

---

## § 6 — PHASE 3: STRUCTURAL MODELING AND COMPARISON (Weeks 5–8)

### 6.1 Objective
Generate AF2/ColabFold models for sequences without experimental structures.
Quality-check with MolProbity/QMEAN. Structural superposition with TM-align.
Surface accessibility (FreeSASA). Electrostatics (PDB2PQR/APBS) for top candidates.

### 6.2 Log initialization

```bash
echo "[$(date '+%Y-%m-%d %H:%M')] [PHASE-3] [STATUS: START] Structural modeling phase started." >> ~/CrustaceanXR/logs/logpole.txt && echo "[$(date '+%Y-%m-%d %H:%M')] [PHASE-3] [STATUS: START]" >> ~/CrustaceanXR/logs/phase3/phase3.log
```

### 6.3 Identify sequences needing AF2 modeling

```bash
conda activate crustaceanxr && python3 - <<'PYEOF'
# Cross-reference processed FASTAs against downloaded PDB IDs
# Sequences with no PDB match → send to ColabFold
import os
from Bio import SeqIO

fasta = os.path.expanduser("~/CrustaceanXR/data/processed/fasta/ALL_allergens.fasta")
pdb_dir = os.path.expanduser("~/CrustaceanXR/data/raw/structures")
af2_queue = os.path.expanduser("~/CrustaceanXR/data/processed/fasta/AF2_queue.fasta")
log = os.path.expanduser("~/CrustaceanXR/logs/phase3/phase3.log")

pdb_files = set(f.split(".")[0].upper() for f in os.listdir(pdb_dir))
records = list(SeqIO.parse(fasta, "fasta"))
needs_model = []
with open(log, "a") as lf:
    for rec in records:
        # Check if accession matches any PDB
        acc = rec.id.split("_")[-1].upper()
        if acc not in pdb_files:
            needs_model.append(rec)
            lf.write(f"[PHASE-3][AF2_QUEUE] {rec.id} — no PDB found, queuing for AF2\n")
    SeqIO.write(needs_model, af2_queue, "fasta")
    lf.write(f"[PHASE-3][SUMMARY] {len(needs_model)} sequences queued for AF2 modeling → {af2_queue}\n")
    print(f"Queued for AF2: {len(needs_model)} sequences")
PYEOF
echo "[$(date '+%Y-%m-%d %H:%M')] [PHASE-3] [STATUS: OK] AF2 queue built." >> ~/CrustaceanXR/logs/logpole.txt
```

### 6.4 ColabFold local run (if GPU available)

```bash
# Install ColabFold local (one-time):
pip install colabfold[alphafold] 2>&1 | tee -a ~/CrustaceanXR/logs/phase3/phase3.log

# Run prediction on queued sequences:
colabfold_batch ~/CrustaceanXR/data/processed/fasta/AF2_queue.fasta ~/CrustaceanXR/data/processed/structures/af2_models/ --num-recycle 3 2>&1 | tee -a ~/CrustaceanXR/logs/phase3/phase3.log && echo "[$(date '+%Y-%m-%d %H:%M')] [PHASE-3] [STATUS: OK] ColabFold run complete." >> ~/CrustaceanXR/logs/logpole.txt
```

> **If no GPU:** Use ColabFold web server (https://colab.research.google.com/github/sokrypton/ColabFold/blob/main/AlphaFold2.ipynb), download PDB results, place in `~/CrustaceanXR/data/processed/structures/af2_models/`, then log manually:
> ```bash
> echo "[$(date '+%Y-%m-%d %H:%M')] [PHASE-3] [MANUAL] AF2 models downloaded from ColabFold web server." >> ~/CrustaceanXR/logs/phase3/phase3.log
> ```

### 6.5 TM-align all-vs-all structural superposition

```bash
conda activate crustaceanxr && python3 - <<'PYEOF'
import os, subprocess
from itertools import combinations
import pandas as pd

struct_dir = os.path.expanduser("~/CrustaceanXR/data/processed/structures/af2_models")
pdb_dir    = os.path.expanduser("~/CrustaceanXR/data/raw/structures")
outdir     = os.path.expanduser("~/CrustaceanXR/results/structures")
log        = os.path.expanduser("~/CrustaceanXR/logs/phase3/phase3.log")

all_pdbs = []
for d in [struct_dir, pdb_dir]:
    for f in os.listdir(d):
        if f.endswith(".pdb") or f.endswith(".cif"):
            all_pdbs.append(os.path.join(d, f))

rows = []
with open(log, "a") as lf:
    for a, b in combinations(all_pdbs, 2):
        name = f"{os.path.basename(a).split('.')[0]}__vs__{os.path.basename(b).split('.')[0]}"
        cmd = ["TMalign", a, b]
        result = subprocess.run(cmd, capture_output=True, text=True)
        if result.returncode != 0:
            lf.write(f"[PHASE-3][ERROR] TMalign failed: {name}: {result.stderr[:150]}\n")
            continue
        tm1, tm2, rmsd = None, None, None
        for line in result.stdout.splitlines():
            if "TM-score=" in line and "Chain_1" in line:
                tm1 = float(line.split("TM-score=")[1].split()[0])
            if "TM-score=" in line and "Chain_2" in line:
                tm2 = float(line.split("TM-score=")[1].split()[0])
            if "RMSD=" in line and "Seq_ID" in line:
                rmsd = float(line.split("RMSD=")[1].split(",")[0].strip())
        rows.append({"pair": name, "TM_score_1": tm1, "TM_score_2": tm2, "RMSD": rmsd})
        lf.write(f"[PHASE-3][OK] {name}: TM1={tm1} TM2={tm2} RMSD={rmsd}\n")

df = pd.DataFrame(rows)
df.to_csv(os.path.join(outdir, "tmalign_results.csv"), index=False)
lf_path = os.path.expanduser("~/CrustaceanXR/logs/logpole.txt")
with open(lf_path, "a") as lf:
    lf.write(f"[{__import__('datetime').datetime.now().strftime('%Y-%m-%d %H:%M')}] [PHASE-3] [STATUS: OK] TM-align: {len(rows)} pairs. Saved tmalign_results.csv\n")
print(f"TM-align complete. {len(rows)} pairs.")
PYEOF
```

### 6.6 FreeSASA — surface accessibility

```bash
conda activate crustaceanxr && for pdb in ~/CrustaceanXR/data/processed/structures/af2_models/*.pdb; do name=$(basename $pdb .pdb); freesasa $pdb --format=rsa > ~/CrustaceanXR/results/structures/${name}_sasa.rsa 2>&1 && echo "[$(date '+%Y-%m-%d %H:%M')] [PHASE-3] [OK] SASA: $name" >> ~/CrustaceanXR/logs/phase3/phase3.log; done && echo "[$(date '+%Y-%m-%d %H:%M')] [PHASE-3] [STATUS: COMPLETE] FreeSASA complete." >> ~/CrustaceanXR/logs/logpole.txt
```

---

## § 7 — PHASE 4: EPITOPE PREDICTION (Weeks 8–11)

### 7.1 Objective
Linear B-cell: BepiPred 3.0, ABCpred, SVMTriP.
Conformational B-cell: DiscoTope 2.0, ElliPro.
T-cell (MHC-II): NetMHCIIpan-4.1.
Allergenicity: AlgPred 2.0, AllerTOP 2.0.
**Consensus rule: retain epitope residues predicted by ≥ 2 independent tools.**

### 7.2 Log initialization

```bash
echo "[$(date '+%Y-%m-%d %H:%M')] [PHASE-4] [STATUS: START] Epitope prediction started." >> ~/CrustaceanXR/logs/logpole.txt && echo "[$(date '+%Y-%m-%d %H:%M')] [PHASE-4] [STATUS: START]" >> ~/CrustaceanXR/logs/phase4/phase4.log
```

### 7.3 BepiPred 3.0 (web server — download TSV results)
BepiPred 3.0 does not have a stable CLI. Use the web server:
- URL: https://services.healthtech.dtu.dk/service.php?BepiPred-3.0
- Submit `ALL_allergens.fasta`; download results as TSV
- Save to: `~/CrustaceanXR/data/processed/epitopes/bepipred3_raw.tsv`

```bash
echo "[$(date '+%Y-%m-%d %H:%M')] [PHASE-4] [MANUAL] BepiPred 3.0 results downloaded from web server → data/processed/epitopes/bepipred3_raw.tsv" >> ~/CrustaceanXR/logs/phase4/phase4.log
```

### 7.4 Parse BepiPred 3.0 output — extract epitope residues

```bash
conda activate crustaceanxr && python3 - <<'PYEOF'
import pandas as pd, os

tsv  = os.path.expanduser("~/CrustaceanXR/data/processed/epitopes/bepipred3_raw.tsv")
out  = os.path.expanduser("~/CrustaceanXR/results/epitopes/bepipred3_epitopes.csv")
log  = os.path.expanduser("~/CrustaceanXR/logs/phase4/phase4.log")

df = pd.read_csv(tsv, sep="\t")
# BepiPred 3.0 output columns: SeqName, Position, AA, Score, Epitope(T/F)
epitopes = df[df["Epitope"] == True][["SeqName", "Position", "AA", "Score"]]
epitopes.to_csv(out, index=False)
with open(log, "a") as lf:
    lf.write(f"[PHASE-4][OK] BepiPred3: {len(epitopes)} epitope residues across {epitopes['SeqName'].nunique()} sequences → {out}\n")
print(f"BepiPred3 epitope residues: {len(epitopes)}")
PYEOF
echo "[$(date '+%Y-%m-%d %H:%M')] [PHASE-4] [STATUS: OK] BepiPred3 parsed." >> ~/CrustaceanXR/logs/logpole.txt
```

### 7.5 NetMHCIIpan-4.1 — T-cell epitopes
Install locally (academic license): https://services.healthtech.dtu.dk/services/NetMHCIIpan-4.1/

```bash
# After local install, add to PATH, then:
conda activate crustaceanxr && python3 - <<'PYEOF'
import os, subprocess
from Bio import SeqIO

fasta  = os.path.expanduser("~/CrustaceanXR/data/processed/fasta/ALL_allergens.fasta")
outdir = os.path.expanduser("~/CrustaceanXR/results/epitopes/netmhciipan")
log    = os.path.expanduser("~/CrustaceanXR/logs/phase4/phase4.log")
os.makedirs(outdir, exist_ok=True)

# Common HLA-DR alleles relevant to Th2/IgE sensitization
alleles = "DRB1_0101,DRB1_0301,DRB1_0401,DRB1_0701"

with open(log, "a") as lf:
    for rec in SeqIO.parse(fasta, "fasta"):
        tmp = f"/tmp/{rec.id}.fasta"
        with open(tmp, "w") as f:
            f.write(f">{rec.id}\n{rec.seq}\n")
        out_f = os.path.join(outdir, f"{rec.id}_MHCII.xls")
        cmd = ["netMHCIIpan", "-f", tmp, "-a", alleles, "-xls", "-xlsfile", out_f]
        res = subprocess.run(cmd, capture_output=True, text=True)
        if res.returncode != 0:
            lf.write(f"[PHASE-4][ERROR] NetMHCIIpan failed for {rec.id}: {res.stderr[:200]}\n")
        else:
            lf.write(f"[PHASE-4][OK] NetMHCIIpan: {rec.id} → {out_f}\n")
print("NetMHCIIpan done.")
PYEOF
echo "[$(date '+%Y-%m-%d %H:%M')] [PHASE-4] [STATUS: OK] NetMHCIIpan-4.1 complete." >> ~/CrustaceanXR/logs/logpole.txt
```

### 7.6 Consensus epitope filtering (≥ 2 tools agree)

```bash
conda activate crustaceanxr && python3 - <<'PYEOF'
import pandas as pd, os

# Load all tool outputs (adapt column names to actual outputs)
ep_dir = os.path.expanduser("~/CrustaceanXR/results/epitopes")
log    = os.path.expanduser("~/CrustaceanXR/logs/phase4/phase4.log")

tool_dfs = {}
tool_files = {
    "bepipred3": os.path.join(ep_dir, "bepipred3_epitopes.csv"),
    # Add abcpred, svmtrip, discotope, ellipro similarly after parsing
}

# Build consensus: for each sequence, find positions flagged by ≥2 tools
all_results = []
for tool, fpath in tool_files.items():
    if os.path.exists(fpath):
        df = pd.read_csv(fpath)
        df["tool"] = tool
        all_results.append(df)

if all_results:
    combined = pd.concat(all_results, ignore_index=True)
    consensus = combined.groupby(["SeqName", "Position"]).size().reset_index(name="tool_count")
    consensus = consensus[consensus["tool_count"] >= 2]
    out = os.path.join(ep_dir, "consensus_epitopes.csv")
    consensus.to_csv(out, index=False)
    with open(log, "a") as lf:
        lf.write(f"[PHASE-4][OK] Consensus epitopes (≥2 tools): {len(consensus)} residues → {out}\n")
    print(f"Consensus epitopes: {len(consensus)} residues")
else:
    print("No tool output files found yet — add tool outputs and re-run.")
PYEOF
echo "[$(date '+%Y-%m-%d %H:%M')] [PHASE-4] [STATUS: OK] Consensus epitope filtering done." >> ~/CrustaceanXR/logs/logpole.txt
```

---

## § 8 — PHASE 5: COMPOSITE CROSS-REACTIVITY SCORING (Weeks 11–13)

### 8.1 Objective
Build feature vector x(ai, aj) per pair:
`[identity, kmer_match, TM_score, epitope_overlap_jaccard, domain_match]`
Compute weighted rule-based index f_rule(x) = wᵀx.

### 8.2 Log initialization

```bash
echo "[$(date '+%Y-%m-%d %H:%M')] [PHASE-5] [STATUS: START] Cross-reactivity scoring started." >> ~/CrustaceanXR/logs/logpole.txt
```

### 8.3 Build master feature matrix

```bash
conda activate crustaceanxr && python3 - <<'PYEOF'
import pandas as pd, numpy as np, os

base = os.path.expanduser("~/CrustaceanXR")
ident_df    = pd.read_csv(f"{base}/results/alignments/pairwise_identity_matrix.csv")
tmalign_df  = pd.read_csv(f"{base}/results/structures/tmalign_results.csv")
epitope_df  = pd.read_csv(f"{base}/results/epitopes/consensus_epitopes.csv")
domain_df   = pd.read_csv(f"{base}/results/domains/pfam_domain_summary.csv") if os.path.exists(f"{base}/results/domains/pfam_domain_summary.csv") else pd.DataFrame()
log         = f"{base}/logs/phase5/phase5.log" if os.path.exists(f"{base}/logs/phase5") else f"{base}/logs/logpole.txt"

os.makedirs(f"{base}/logs/phase5", exist_ok=True)

# Merge identity with tmalign on pair key
merged = ident_df.copy()
merged["pair"] = merged["seq_a"] + "__vs__" + merged["seq_b"]
tmalign_df["TM_score_mean"] = tmalign_df[["TM_score_1","TM_score_2"]].mean(axis=1)
merged = merged.merge(tmalign_df[["pair","TM_score_mean","RMSD"]], on="pair", how="left")

# k-mer 6-mer exact match (placeholder — implement if needed)
merged["kmer6_match"] = 0  # update with actual kmer analysis

# Domain match (binary): load Pfam output and check shared domains per pair
merged["domain_match"] = 0  # update after Pfam parsing

# Epitope Jaccard overlap (placeholder — implement after epitope atlas is complete)
merged["epitope_jaccard"] = np.nan

# Weighted index (weights from literature — adjust empirically)
W = {"identity_pct": 0.30, "TM_score_mean": 0.25, "epitope_jaccard": 0.25,
     "kmer6_match": 0.10, "domain_match": 0.10}
merged["f_rule"] = (
    merged["identity_pct"].fillna(0)  * W["identity_pct"] +
    merged["TM_score_mean"].fillna(0) * W["TM_score_mean"] +
    merged["epitope_jaccard"].fillna(0) * W["epitope_jaccard"] +
    merged["kmer6_match"].fillna(0)   * W["kmer6_match"] +
    merged["domain_match"].fillna(0)  * W["domain_match"]
)

out = f"{base}/results/scoring/feature_matrix.csv"
merged.to_csv(out, index=False)
with open(log, "a") as lf:
    lf.write(f"[PHASE-5][OK] Feature matrix: {len(merged)} pairs, {merged.shape[1]} features → {out}\n")
print(f"Feature matrix: {merged.shape}")
PYEOF
echo "[$(date '+%Y-%m-%d %H:%M')] [PHASE-5] [STATUS: OK] Feature matrix built." >> ~/CrustaceanXR/logs/logpole.txt
```

---

## § 9 — PHASE 6: MOLECULAR DOCKING (Weeks 13–16)

### 9.1 Objective
Dock top-ranked cross-reactive epitope candidates against an IgE Fab structure.
Tools: HADDOCK (web) / ClusPro (web) / ZDOCK (local).
Binding affinity: PRODIGY. ΔΔG for isoallergen variants: FoldX.

### 9.2 Candidate selection — top candidates from feature matrix

```bash
conda activate crustaceanxr && python3 - <<'PYEOF'
import pandas as pd, os
df = pd.read_csv(os.path.expanduser("~/CrustaceanXR/results/scoring/feature_matrix.csv"))
top = df.nlargest(20, "f_rule")[["seq_a","seq_b","identity_pct","TM_score_mean","f_rule"]]
top.to_csv(os.path.expanduser("~/CrustaceanXR/results/docking/top_candidates.csv"), index=False)
print(top.to_string())
PYEOF
echo "[$(date '+%Y-%m-%d %H:%M')] [PHASE-6] [STATUS: OK] Top 20 docking candidates selected." >> ~/CrustaceanXR/logs/logpole.txt
```

### 9.3 PRODIGY binding affinity (post-docking)

```bash
pip install prodigy-prot 2>&1 | tee -a ~/CrustaceanXR/logs/phase6/phase6.log && for pdb in ~/CrustaceanXR/results/docking/*.pdb; do name=$(basename $pdb .pdb); prodigy $pdb 2>&1 | tee ~/CrustaceanXR/results/docking/${name}_prodigy.txt | grep -E "Predicted|IC|DG" | tee -a ~/CrustaceanXR/logs/phase6/phase6.log; done && echo "[$(date '+%Y-%m-%d %H:%M')] [PHASE-6] [STATUS: OK] PRODIGY binding affinity analysis done." >> ~/CrustaceanXR/logs/logpole.txt
```

---

## § 10 — PHASE 7: MACHINE LEARNING CLASSIFICATION (Weeks 16–18)

### 10.1 Objective
Train RF / XGBoost / SVM on feature_matrix.csv with literature-curated labels.
Stratified k-fold CV. ROC-AUC + precision/recall. Feature importance analysis.

### 10.2 Log initialization

```bash
echo "[$(date '+%Y-%m-%d %H:%M')] [PHASE-7] [STATUS: START] ML classification started." >> ~/CrustaceanXR/logs/logpole.txt && echo "[$(date '+%Y-%m-%d %H:%M')] [PHASE-7] [STATUS: START]" >> ~/CrustaceanXR/logs/phase7/phase7.log
```

### 10.3 Prepare labeled training set

```bash
conda activate crustaceanxr && python3 - <<'PYEOF'
# Literature-curated positive pairs (confirmed cross-reactive):
# shrimp tropomyosin (Pen a 1) vs dust mite (Der p 10) = POSITIVE
# shrimp tropomyosin (Pen a 1) vs cockroach (Per a 7, Bla g 7) = POSITIVE
# Arginine kinase crustacean cross-species = POSITIVE
# Non-crustacean Big 8/9 with <20% identity and no structural match = NEGATIVE

import pandas as pd, os

df = pd.read_csv(os.path.expanduser("~/CrustaceanXR/results/scoring/feature_matrix.csv"))
log = os.path.expanduser("~/CrustaceanXR/logs/phase7/phase7.log")

# Assign labels based on known biology — expand as literature is reviewed
def label_pair(row):
    a, b = row["seq_a"].lower(), row["seq_b"].lower()
    if ("tropomyosin" in a or "pen_a" in a) and ("der_p" in b or "dustmite" in b or "per_a" in b):
        return 1
    if ("arginine" in a and "arginine" in b):
        return 1
    if row["identity_pct"] < 20 and row["TM_score_mean"] < 0.4:
        return 0
    return -1  # unknown — exclude from training

df["label"] = df.apply(label_pair, axis=1)
labeled = df[df["label"] != -1]
labeled.to_csv(os.path.expanduser("~/CrustaceanXR/results/ml/labeled_dataset.csv"), index=False)
with open(log, "a") as lf:
    lf.write(f"[PHASE-7][OK] Labeled dataset: {len(labeled)} pairs ({labeled['label'].sum()} positive, {(labeled['label']==0).sum()} negative)\n")
print(f"Labeled: {len(labeled)} pairs")
PYEOF
```

### 10.4 Train and evaluate RF / XGBoost / SVM

```bash
conda activate crustaceanxr && python3 - <<'PYEOF'
import pandas as pd, numpy as np, os
from sklearn.ensemble import RandomForestClassifier
from sklearn.svm import SVC
from sklearn.model_selection import StratifiedKFold, cross_validate
from sklearn.preprocessing import StandardScaler
from sklearn.pipeline import Pipeline
from sklearn.metrics import make_scorer, roc_auc_score, average_precision_score
import xgboost as xgb
import json

df    = pd.read_csv(os.path.expanduser("~/CrustaceanXR/results/ml/labeled_dataset.csv"))
log   = os.path.expanduser("~/CrustaceanXR/logs/phase7/phase7.log")
outd  = os.path.expanduser("~/CrustaceanXR/results/ml")

features = ["identity_pct", "TM_score_mean", "RMSD", "kmer6_match", "domain_match", "epitope_jaccard"]
feat_cols = [c for c in features if c in df.columns]
X = df[feat_cols].fillna(0).values
y = df["label"].values

cv = StratifiedKFold(n_splits=5, shuffle=True, random_state=42)
scorers = {"roc_auc": "roc_auc", "avg_precision": "average_precision", "f1": "f1"}

models = {
    "RandomForest": Pipeline([("clf", RandomForestClassifier(n_estimators=300, random_state=42))]),
    "XGBoost":      Pipeline([("clf", xgb.XGBClassifier(n_estimators=200, use_label_encoder=False, eval_metric="logloss", random_state=42))]),
    "SVM":          Pipeline([("scaler", StandardScaler()), ("clf", SVC(probability=True, kernel="rbf", random_state=42))]),
}

results = {}
with open(log, "a") as lf:
    for name, model in models.items():
        cv_res = cross_validate(model, X, y, cv=cv, scoring=scorers, return_train_score=False)
        res = {m: {"mean": float(np.mean(v)), "std": float(np.std(v))} for m, v in cv_res.items() if m.startswith("test_")}
        results[name] = res
        lf.write(f"[PHASE-7][RESULT] {name}: ROC-AUC={res['test_roc_auc']['mean']:.3f}±{res['test_roc_auc']['std']:.3f} | AvgPR={res['test_avg_precision']['mean']:.3f}\n")
        print(f"{name}: ROC-AUC={res['test_roc_auc']['mean']:.3f}±{res['test_roc_auc']['std']:.3f}")

with open(os.path.join(outd, "cv_results.json"), "w") as jf:
    json.dump(results, jf, indent=2)

# Feature importance from best RF model
rf = RandomForestClassifier(n_estimators=300, random_state=42)
rf.fit(X, y)
imp_df = pd.DataFrame({"feature": feat_cols, "importance": rf.feature_importances_}).sort_values("importance", ascending=False)
imp_df.to_csv(os.path.join(outd, "feature_importance.csv"), index=False)
print(imp_df)
with open(log, "a") as lf:
    lf.write(f"[PHASE-7][OK] Feature importance saved. Top feature: {imp_df.iloc[0]['feature']}\n")
PYEOF
echo "[$(date '+%Y-%m-%d %H:%M')] [PHASE-7] [STATUS: OK] ML training and CV complete." >> ~/CrustaceanXR/logs/logpole.txt
```

---

## § 11 — PHASE 8: STATISTICAL VALIDATION (Weeks 18–20)

### 11.1 Permutation testing + Benjamini-Hochberg correction

```bash
conda activate crustaceanxr && python3 - <<'PYEOF'
import pandas as pd, numpy as np, os
from scipy.stats import permutation_test
from statsmodels.stats.multitest import multipletests

df = pd.read_csv(os.path.expanduser("~/CrustaceanXR/results/scoring/feature_matrix.csv"))
log = os.path.expanduser("~/CrustaceanXR/logs/phase8/phase8.log")
os.makedirs(os.path.expanduser("~/CrustaceanXR/logs/phase8"), exist_ok=True)

# Example: test if identity_pct is significantly higher in FAOWHO-flagged pairs
flagged     = df[df["FAOWHO_flag"]=="YES"]["identity_pct"].dropna().values
not_flagged = df[df["FAOWHO_flag"]=="NO"]["identity_pct"].dropna().values

def statistic(x, y, axis): return np.mean(x, axis=axis) - np.mean(y, axis=axis)
res = permutation_test((flagged, not_flagged), statistic, n_resamples=10000, alternative="greater")

# Collect all pairwise p-values for BH correction
pvals = [res.pvalue]  # extend with all pairwise tests
reject, pvals_corr, _, _ = multipletests(pvals, alpha=0.05, method="fdr_bh")

with open(log, "a") as lf:
    lf.write(f"[PHASE-8][RESULT] Permutation test identity_pct flagged vs unflagged: p={res.pvalue:.4f}\n")
    lf.write(f"[PHASE-8][RESULT] BH-corrected p={pvals_corr[0]:.4f} | Reject H0: {reject[0]}\n")
    print(f"p={res.pvalue:.4f} | BH-corrected: {pvals_corr[0]:.4f} | Reject: {reject[0]}")
PYEOF
echo "[$(date '+%Y-%m-%d %H:%M')] [PHASE-8] [STATUS: OK] Permutation testing and BH correction done." >> ~/CrustaceanXR/logs/logpole.txt
```

---

## § 12 — LOGPOLE.TXT — MASTER LOG SPECIFICATION

`~/CrustaceanXR/logs/logpole.txt` is the **single source of truth** for the
entire project. It must contain:

| Section | Content |
|---|---|
| `[INIT]` | Directory creation, environment setup |
| `[PHASE-N]` | Phase start, key results, file paths, row counts |
| `[ERROR]` | Full error type, root cause, fix applied, verification |
| `[RESULT]` | Quantitative outcomes (metrics, counts, file sizes) |
| `[DECISION]` | Tool choices, parameter choices, skips with rationale |
| `[MANUAL]` | Steps done outside bash (web server submissions) |
| `[COMPLETE]` | Phase completion summary |
| `[OMEGA]` | MD version increments |

### Append to logpole.txt (template)

```bash
echo "[$(date '+%Y-%m-%d %H:%M')] [PHASE-N] [STATUS: OK|ERROR|SKIP|DECISION|RESULT] <message>" >> ~/CrustaceanXR/logs/logpole.txt
```

### View last 50 entries

```bash
tail -50 ~/CrustaceanXR/logs/logpole.txt
```

### Count errors

```bash
grep -c "\[ERROR\]" ~/CrustaceanXR/logs/logpole.txt
```

### Count resolved errors

```bash
grep "\[ERROR\]" ~/CrustaceanXR/logs/logpole.txt | wc -l && grep -c "RESOLVED" ~/CrustaceanXR/logs/logpole.txt
```

---

## § 13 — RUNNING SESSION LOG
*(Append here every time a new MD version is generated — never truncate)*

```
[SESSION-v1.0] Initial MD created from project proposal. No pipeline run yet.
               Student: Karshani Tiwary. All phases defined. Directory tree command ready.
               Error count: 0 | Resolved: 0
```

---

## § 14 — CURRENT STATUS SNAPSHOT
*(Update this section every time a new MD version is generated)*

| Phase | Status | Key outputs | Notes |
|---|---|---|---|
| ENV setup | ⬜ Not started | — | — |
| Phase 1 — Data | ⬜ Not started | — | — |
| Phase 2 — Sequence | ⬜ Not started | — | — |
| Phase 3 — Structure | ⬜ Not started | — | — |
| Phase 4 — Epitope | ⬜ Not started | — | — |
| Phase 5 — Scoring | ⬜ Not started | — | — |
| Phase 6 — Docking | ⬜ Not started | — | — |
| Phase 7 — ML | ⬜ Not started | — | — |
| Phase 8 — Stats | ⬜ Not started | — | — |

**Total errors encountered:** 0
**Total errors resolved:** 0
**Current working phase:** ENV setup (start here)
**Last MD version written:** v1.0

---

## § 15 — QUICK REFERENCE — KEY FILE PATHS

```
~/CrustaceanXR/logs/logpole.txt                          ← MASTER LOG
~/CrustaceanXR/data/processed/fasta/ALL_allergens.fasta  ← all sequences
~/CrustaceanXR/results/alignments/pairwise_identity_matrix.csv
~/CrustaceanXR/results/structures/tmalign_results.csv
~/CrustaceanXR/results/epitopes/consensus_epitopes.csv
~/CrustaceanXR/results/scoring/feature_matrix.csv
~/CrustaceanXR/results/ml/cv_results.json
~/CrustaceanXR/results/ml/feature_importance.csv
~/CrustaceanXR/docs/                                     ← manuscript files
```

---

## § 16 — OMEGA REMINDER (Repeat at bottom for reinforcement)

> **When this chat is too long or the user says "give me the MD":**
>
> 1. Tail the last 100 lines of `logpole.txt` to recover state.
> 2. Update §13 (Running Session Log) with everything done since last version.
> 3. Update §14 (Status Snapshot) and error counts.
> 4. Increment version in the header.
> 5. Write `CRUSTACEANXR_MASTER_v{N}.md` to `~/CrustaceanXR/logs/`.
> 6. Present the new file. The loop never ends. The log is never lost.
>
> *This document is self-replicating. It carries all context forward.
> Every error, every result, every command — already here.*
```
