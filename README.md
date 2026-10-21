# PopGen-Iberia-TAPAS-Inference
Population genomic workflow in UPPMAX for ancient DNA: LD pruning, projected smartpca, and ADMIXTURE/PONG to unmask 25 unknown samples across an 8,000-year Iberian demographic transect based on Olalde et al. (2019).

# Population Structure Inference Pipeline (Part 2)

This workflow outlines the population genomics pipeline implemented on UPPMAX (Pelle cluster) to infer the ancestry and temporal context of the 25 unknown samples (TAPAS dataset), co-analyzed with ancient and modern West Eurasian reference panels based on Olalde et al. (2019).

---

## Horizontal End-to-End Pipeline Diagram

```mermaid
flowchart LR
    %% Subgraph 1: LD Pruning
    subgraph S1["1. LD Pruning & Quality Filtering"]
        direction TB
        A["COMPLETE_DATASET_geno0.99<br/>• 597,569 SNPs<br/>• 844 individuals<br/>• 57.65% call rate"] --> B["PLINK v1.9<br/>--indep-pairwise 10 10 0.5<br/>(Window: 10, Step: 10, r²: 0.5)"]
        B --> C["Marker Exclusion<br/>• 126,175 SNPs pruned (plink.prune.out)<br/>• 471,394 SNPs retained (plink.prune.in)"]
        C --> D["COMPLETE_DATASET_geno0.99_LD<br/>• Independent marker set<br/>• Call rate improved to 68.42%"]
    end

    %% Subgraph 2: Conventional PCA Evaluation
    subgraph S2["2. Conventional PCA (Quality Control)"]
        direction TB
        D --> E["PLINK v1.9<br/>--pca 10<br/>(Simultaneous SVD across all 844 samples)"]
        E --> F["Diagnostic Scree & Bivariate Plots<br/>• PC1: 24.7% variance<br/>• PC2: 17.1% variance"]
        F --> G["Severe Shrinkage Bias Identified<br/>• Implicit zero-imputation of missing data<br/>• Ancient & TAPAS pull toward origin (0,0)<br/>• Unprojected PCA rejected for aDNA"]
    end

    %% Subgraph 3: Projected PCA
    subgraph S3["3. Projected PCA (smartpca)"]
        direction TB
        D --> H["File Transformation<br/>PLINK --recode transpose<br/>(.tped & .tfam coordinate files)"]
        H --> I["Scaffold Definition (pca.par)<br/>• MODERN_SAMPLES.txt (N=625 moderns)<br/>• UNKNOWN_SAMPLES.txt (194 ancient + 25 TAPAS)"]
        I --> J["EIGENSOFT smartpca execution<br/>• lsqproject: YES (least-squares)<br/>• shrinkmode: YES<br/>• killr2: YES (r2thresh: 0.7)"]
        J --> K["Unbiased Coordinates (.evec & .eval)<br/>• PC1: 0.53% (East-West cline)<br/>• PC2: 0.31% (North-South cline)<br/>• Resolves true West Eurasian genetic map"]
    end

    %% Subgraph 4: ADMIXTURE & PONG
    subgraph S4["4. Ancestry Modeling & Modal Integration"]
        direction TB
        D --> L["ADMIXTURE v1.3.0<br/>• K = 2 through 6<br/>• 3 iterations per K (-s $RANDOM)<br/>• 10-fold cross-validation (--cv=10)<br/>• 15 total SLURM batch jobs"]
        L --> M["PONG v1.5 Setup<br/>• PONG_FILEMAP.txt (runID, K, .Q path)<br/>• PONG_IND2POP.txt (844 sample-to-pop links)<br/>• PONG_POPORDER.txt (chronological order)"]
        M --> N["PONG Modal Alignment & Merging<br/>• Resolves label switching & multimodality<br/>• Aligns modes across runs (similarity >0.98)<br/>• Web server visualization (port 4000)"]
    end

    %% Subgraph 5: Sample Assignment
    subgraph S5["5. TAPAS Taxonomic Assignment"]
        direction TB
        K & N --> O["Joint PCA & ADMIXTURE Synthesis<br/>Mapped against Olalde et al. (2019) transect"]
        O --> P1["Neolithic / Chalcolithic (UNK_SP_8, 14–18)<br/>• High EEF + WHG resurgence<br/>• Strictly 0% Steppe ancestry<br/>• Co-localizes with Spain_MN / Spain_C"]
        O --> P2["Bronze & Iron Age (UNK_SP_6, 9, 19–22)<br/>• Influx of ~40% Steppe component (purple)<br/>• Shifts rightward in PC1 toward modern Iberians"]
        O --> P3["Historical / Roman / Islamic (UNK_SP_10–13)<br/>• Introduction of Mediterranean/Near Eastern ancestry<br/>• Positive PC2 shift; matches Spain_Medieval"]
        O --> P4["Controls & Anchors (UNK_SP_1–5, 7, 24–25)<br/>• UNK_SP_1/2: Pure Basque yellow component<br/>• UNK_SP_24/25: Modern Spaniards<br/>• UNK_SP_3/4/5/7: Finnish, Russian, Turkish, Hungarian"]
    end

    %% Flow links between major stages
    S1 --> S2
    S2 --> S3
    S1 --> S4
    S3 & S4 --> S5

    %% Stylings
    classDef default font-family:sans-serif,font-size:11px;
    style S1 fill:#f8f9fa,stroke:#495057,stroke-width:1.5px
    style S2 fill:#fff5f5,stroke:#fa5252,stroke-width:1.5px
    style S3 fill:#e7f5ff,stroke:#228be6,stroke-width:1.5px
    style S4 fill:#ebfbee,stroke:#40c057,stroke-width:1.5px
    style S5 fill:#fff9db,stroke:#fab005,stroke-width:1.5px
