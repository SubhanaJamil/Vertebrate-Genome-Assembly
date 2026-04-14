# 🧬 Genome Assembly Pipeline Overview (Saccharomyces cerevisiae S288C)
This repository documents a complete genome assembly workflow for *Saccharomyces cerevisiae* S288C using PacBio HiFi long reads and Illumina Hi-C data. The pipeline follows a Vertebrate Genome Project (VGP)-style approach, integrating genome profiling, de novo assembly, scaffolding, and multi-layer quality assessment.

---

# 📊 Workflow Overview

## 🧭 Pipeline Flow

```
┌────────────────────────────────────────────┐
│            1. Data Acquisition             │
│  - PacBio HiFi reads (FASTA)              │
│  - Illumina Hi-C reads (FASTQ.gz)         │
└────────────────────────────────────────────┘
                     │
                     ▼
┌────────────────────────────────────────────┐
│        2. Data Preprocessing              │
│  - Organize HiFi collection               │
│  - Adapter trimming (Cutadapt)            │
│  - Hi-C reads preparation                 │
└────────────────────────────────────────────┘
                     │
                     ▼
┌────────────────────────────────────────────┐
│     3. Genome Profiling (k-mers)          │
│  - Meryl k-mer counting (k=31)            │
│  - Merge k-mer databases                  │
│  - GenomeScope2 analysis                 │
│  - Genome size + heterozygosity estimate  │
└────────────────────────────────────────────┘
                     │
                     ▼
┌────────────────────────────────────────────┐
│     4. De novo Genome Assembly            │
│  - hifiasm (Hi-C phased mode)             │
│  - Output: Hap1 & Hap2 GFA graphs         │
│  - Conversion to FASTA (gfastats)         │
└────────────────────────────────────────────┘
                     │
                     ▼
┌────────────────────────────────────────────┐
│     5. Assembly Evaluation (Pre-scaffold)  │
│  - gfastats (contig statistics)            │
│  - BUSCO (gene completeness)               │
│  - Merqury (k-mer validation)              │
└────────────────────────────────────────────┘
                     │
                     ▼
┌────────────────────────────────────────────┐
│          6. Scaffolding Stage              │
│  ├── Bionano Optical Mapping               │
│  └── Hi-C based scaffolding (YaHS)         │
└────────────────────────────────────────────┘
                     │
                     ▼
┌────────────────────────────────────────────┐
│     7. Hi-C Mapping & Contact Maps        │
│  - BWA-MEM2 alignment                      │
│  - PretextMap generation                   │
│  - Contact map visualization               │
└────────────────────────────────────────────┘
                     │
                     ▼
┌────────────────────────────────────────────┐
│     8. Final Genome Assembly              │
│  - YaHS chromosome scaffolds              │
│  - Final FASTA genome                     │
│  - Pretext validation                     │
└────────────────────────────────────────────┘
```

---

# 📥 1. Data Acquisition

## Input Data Types

- **PacBio HiFi Reads (FASTA format)**
  - High-accuracy long reads for contig generation

- **Illumina Hi-C Reads (FASTQ.gz format)**
  - Paired-end chromatin interaction data for scaffolding

---

# 🧹 2. Data Preprocessing

## Dataset Organization

- HiFi reads grouped into a dataset collection
- Hi-C reads separated into:
  - Forward reads
  - Reverse reads

---

## Adapter Removal (Cutadapt)

- Removes reads containing internal adapter sequences
- Filters low-quality or contaminated reads

**Key parameters:**
- Error rate: 0.1  
- Minimum overlap: 35  
- Discard trimmed reads: Enabled  

---

# 🧬 3. Genome Profiling (k-mer Analysis)

## Meryl k-mer Counting

- k-mer size: 31
- Generates k-mer frequency database from HiFi reads
- Merges multiple datasets into a unified histogram

---

## GenomeScope2 Analysis

Estimates genome characteristics from k-mer spectra.

### Key Results:
- Haploid genome size: ~11.7 Mb  
- Heterozygosity: ~0.576%  
- Coverage peak: ~50×  
- Confirms diploid genome structure (bimodal distribution)

---

# 🧪 4. De novo Genome Assembly

## hifiasm Assembly

- Input: Trimmed HiFi reads
- Output: GFA assembly graphs

### Assembly Mode:
- Hi-C phased mode
  - Generates:
    - Hap1 contigs
    - Hap2 contigs

---

## GFA → FASTA Conversion

- Tool: gfastats
- Converts assembly graphs into FASTA format for downstream analysis

---

# 📏 5. Assembly Evaluation

## 5.1 gfastats Statistics

- Contig count: ~16–17 per haplotype  
- Total genome length: ~11–12 Mb  
- High contiguity (N50 improvement observed)

---

## 5.2 BUSCO Analysis

Evaluates gene completeness using conserved orthologs.

- High percentage of complete single-copy genes
- Very low missing gene fraction
- Indicates strong assembly completeness

---

## 5.3 Merqury Evaluation

Reference-free validation using k-mers.

- Confirms high assembly accuracy
- Validates correct haplotype phasing
- Shows consistent k-mer distribution between reads and assembly

---

# 🧱 6. Scaffolding

## 6.1 Bionano Optical Mapping

- Uses optical maps to order and orient contigs
- Resolves structural inconsistencies
- Produces improved scaffold continuity

---

## 6.2 Hi-C Data Processing

- Reads aligned using BWA-MEM2
- Forward and reverse reads merged
- Used for 3D genome interaction mapping

---

## 6.3 YaHS Scaffolding

- Uses Hi-C contact frequency to construct chromosome-scale scaffolds
- Detects and corrects misassemblies
- Outputs final scaffolded genome

---

# 📡 7. Hi-C Contact Map Validation

## PretextMap & Pretext Snapshot

- Generates Hi-C contact maps from aligned reads
- Visualizes chromosomal interactions

### Observations:
- Strong diagonal interaction patterns
- Clear chromosome separation
- Improved structure after scaffolding

---

# 🧾 8. Final Genome Assembly

## Final Output Features

- Chromosome-scale scaffolds achieved
- Assembly length consistent with reference genome (~11.7 Mb)
- High structural accuracy confirmed via Hi-C maps
- Near-reference-quality genome achieved

---

# 📌 Key Tools Used

- Cutadapt
- Meryl
- GenomeScope2
- hifiasm
- gfastats
- BUSCO
- Merqury
- BWA-MEM2
- Bionano Hybrid Scaffold
- YaHS
- PretextMap / Pretext Snapshot

---

# 🧠 Key Takeaways

- HiFi reads provide high-accuracy contig assembly
- k-mer analysis is essential for genome estimation
- Hi-C data enables chromosome-level scaffolding
- Optical maps improve structural accuracy
- Multiple QC tools ensure assembly reliability

---

