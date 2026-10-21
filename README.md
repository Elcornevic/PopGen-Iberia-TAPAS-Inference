# PopGen-Iberia-TAPAS-Inference
Population genomic workflow in UPPMAX for ancient DNA: LD pruning, projected smartpca, and ADMIXTURE/PONG to unmask 25 unknown samples across an 8,000-year Iberian demographic transect based on Olalde et al. (2019).

# Population Structure Inference Pipeline (Part 2)

This workflow outlines the analytical steps performed on the harmonized human genomic dataset to infer the ancestral background and chronological placement of 25 uncharacterized archaeological and modern individuals (**TAPAS**), co-analyzed with ancient and modern West Eurasian reference panels.

---

## Pipeline Flowchart

```mermaid
flowchart TD
    subgraph S1["1. Linkage Disequilibrium Pruning"]
        A["COMPLETE_DATASET_geno0.99<br/>(597,569 SNPs, 844 individuals)"] --> B["PLINK: --indep-pairwise 10 10 0.5"]
        B --> C["plink.prune.in<br/>(126,175 correlated variants pruned)"]
        C --> D["COMPLETE_DATASET_geno0.99_LD<br/>(471,394 independent SNPs | 68.42% call rate)"]
    end

    subgraph S2["2. Conventional PCA Technical Control"]
        D --> E["Conventional PCA (PLINK --pca 10)"]
        E --> F["Severe Shrinkage Bias Detected<br/>Ancient & TAPAS samples falsely collapse toward (0,0)"]
    end

    subgraph S3["3. Projected PCA (smartpca)"]
        D --> G["Transposed File Generation (.tped / .tfam)"]
        G --> H["Population Scaffold Lists:<br/>• MODERN_SAMPLES.txt (N=625, orthogonal basis)<br/>• UNKNOWN_SAMPLES.txt (Ancient + TAPAS)"]
        H --> I["smartpca (EIGENSOFT)<br/>• lsqproject: YES<br/>• shrinkmode: YES"]
        I --> J["Projected Coordinates (.evec / .eval)<br/>Reconstructs true West Eurasian genomic clines"]
    end

    subgraph S4["4. Ancestry Deconvolution & Alignment"]
        D --> K["ADMIXTURE (K=2 through K=6)<br/>• 3 stochastic iterations per K<br/>• Tenfold Cross-Validation (--cv=10)"]
        K --> L["PONG v1.5<br/>• Integrates 15 run outputs (.Q matrices)<br/>• Resolves multimodality & label switching"]
    end

    subgraph S5["5. TAPAS Taxonomic Assignment"]
        J --> M["Joint Integration: Projected PCA + PONG Modes"]
        L --> M
        M --> N1["Neolithic / Chalcolithic (UNK_SP_8, 14-18):<br/>EEF + WHG | strictly 0% Steppe"]
        M --> N2["Bronze / Iron Age (UNK_SP_6, 9, 19-22):<br/>Abrupt ~40% Steppe pastoralist influx"]
        M --> N3["Historical Genomes (UNK_SP_10-13):<br/>Eastern Mediterranean / North African gene flow"]
        M --> N4["Modern Controls (UNK_SP_1-5, 7, 24-25):<br/>Pipeline calibration (Basques, Spanish, anchors)"]
    end

    style S1 fill:#f9f9f9,stroke:#333,stroke-width:1px
    style S2 fill:#fff0f0,stroke:#d9534f,stroke-width:1px
    style S3 fill:#f0f8ff,stroke:#0275d8,stroke-width:1px
    style S4 fill:#f0fff0,stroke:#5cb85c,stroke-width:1px
    style S5 fill:#fffbf0,stroke:#f0ad4e,stroke-width:1px
