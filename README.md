# Genome Assembly Pipeline Overview (Saccharomyces cerevisiae S288C)
This repository documents a complete genome assembly workflow for *Saccharomyces cerevisiae* S288C using PacBio HiFi long reads and Illumina Hi-C data. The pipeline follows a Vertebrate Genome Project (VGP)-style approach, integrating genome profiling, de novo assembly, scaffolding, and multi-layer quality assessment.

---

# Workflow Overview

## Pipeline Flow

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

# 1. Data Acquisition

## Input Data Types

- **PacBio HiFi Reads (FASTA format)**
  - High-accuracy long reads for contig generation

- **Illumina Hi-C Reads (FASTQ.gz format)**
  - Paired-end chromatin interaction data for scaffolding

---

# 2. Data Preprocessing

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

#  3. Genome Profiling (k-mer Analysis)

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
  
<p align="center">
<img width="634" height="2000" alt="genomescope_plot" src="https://github.com/user-attachments/assets/ff9ad3df-be7f-47bd-8a4c-008542b135ef" />
</p>
<p align="center">
  <b>Figure 1: GenomeScope2 31-mer profile showing k-mer distribution and genome characteristics.</b>
</p>

# 4. De novo Genome Assembly

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

## Assembly Evaluation

### gfastats

| Metric | Hap1 | Hap2 |
|------|------|------|
| Contigs | ~16 | ~17 |
| Length | ~11.3 Mb | ~12.2 Mb |

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
<p align="center">

 <img width="634" height="1500" alt="merqury_cn_plot" src="https://github.com/user-attachments/assets/4fca23aa-56dd-4a26-b2d9-b439fcee5081" />

</p>

<p align="center">
  <b>Figure 2: Merqury CN plot. This plot tracks the multiplicity of each k-mer found in the HiFi read set and colors it by the number of times it is found in a given assembly. Merqury connects the midpoint of each histogram bin with a line, giving the illusion of a smooth curve.</b>
</p>
  
<p align="center">

<img width="634" height="494" alt="merqury_hap1hap2_asm" src="https://github.com/user-attachments/assets/80e068af-1ff2-4bcc-b6ab-e6fa0394371b" />
</p>

<p align="center">
  <b>Figure 3: Merqury ASM plot. This plot tracks the multiplicity of each k-mer found in the HiFi read set and colors it according to which assemblies contain those k-mers. This can tell you which k-mers are found in only one assembly or shared between them.</b>
</p>
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

### Results

| Feature | Value |
|--------|------|
| Scaffolds | ~16 |
| Genome Size | ~11.7 Mb |
| Accuracy | High |

---

## Final Results

| Feature | Value |
|--------|------|
| Organism | Saccharomyces cerevisiae |
| Genome Size | ~11.7 Mb |
| Ploidy | Diploid |
| Contigs | ~16–17 |
| Scaffolds | ~16 |
| Heterozygosity | ~0.576% |
| Assembly Quality | Near reference-level |

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

