# PopGen-Iberia-TAPAS-Inference
Population genomic workflow in UPPMAX for ancient DNA: LD pruning, projected smartpca, and ADMIXTURE/PONG to unmask 25 unknown samples across an 8,000-year Iberian demographic transect based on Olalde et al. (2019).

```mermaid
flowchart LR
    %% COLUMNA 1: Preparación y Control de Calidad
    subgraph COL1 ["1. QC & LD Pruning"]
        direction TB
        A["<b>COMPLETE_DATASET_geno0.99</b><br/>597,569 SNPs | 844 samples<br/>Call rate: 57.65%"]
        B["<b>PLINK v1.9 Pruning</b><br/><code>--indep-pairwise 10 10 0.5</code><br/>Pruned: 126,175 redundant SNPs"]
        C["<b>Pruned Dataset (LD)</b><br/>471,394 independent SNPs<br/><b>Call rate: 68.42%</b>"]
        A --> B --> C
    end

    %% COLUMNA 2: Análisis Estructural (PCA y ADMIXTURE)
    subgraph COL2 ["2. Population Structure Inference"]
        direction TB
        subgraph G_PCA ["Principal Component Analysis"]
            direction TB
            D["<b>Conventional PCA (PLINK --pca 10)</b><br/>Missing data causes <i>shrinkage bias</i><br/>Ancient samples collapse to (0,0) (Rejected)"]
            E["<b>Projected smartpca (EIGENSOFT)</b><br/>N=625 moderns define orthogonal basis<br/><code>lsqproject: YES</code> | Restores genuine clines"]
            D -.->|Correction| E
        end
        subgraph G_ADMX ["Ancestry Deconvolution"]
            direction TB
            F["<b>ADMIXTURE v1.3.0</b><br/>K = 2 to 6 (3 runs/K, <code>--cv=10</code>)"]
            G["<b>PONG Integration</b><br/>Resolves label switching across modes"]
            F --> G
        end
    end

    %% COLUMNA 3: Asignación Arqueogenómica de TAPAS
    subgraph COL3 ["3. TAPAS Taxonomic Assignment"]
        direction TB
        H["<b>Neolithic / Chalcolithic (UNK_SP_8, 14–18)</b><br/>High EEF + WHG resurgence | <b>0% Steppe</b><br/>Matches <i>Spain_MN</i> and <i>Spain_C</i>"]
        I["<b>Bronze & Iron Age (UNK_SP_6, 9, 19–22)</b><br/><b>~40% Steppe influx</b> (purple component)<br/>Turnover post-2000 BCE (Bell Beaker)"]
        J["<b>Historical / Roman / Islamic (UNK_SP_10–13)</b><br/>East Mediterranean & North African gene flow"]
        K["<b>Modern Controls (UNK_SP_1–5, 7, 24–25)</b><br/>Basque (yellow), Spanish, and Eurasian anchors"]
    end

    %% Conexiones entre columnas
    C --> D
    C --> E
    C --> F
    E --> COL3
    G --> COL3

    %% Estilos de tipografía grande y colores
    classDef default font-family:Arial,Helvetica,sans-serif,font-size:15px;
    
    style COL1 fill:#f8f9fa,stroke:#495057,stroke-width:2px
    style COL2 fill:#e7f5ff,stroke:#1971c2,stroke-width:2px
    style COL3 fill:#fff9db,stroke:#f08c00,stroke-width:2px
    style G_PCA fill:#ffffff,stroke:#adb5bd,stroke-width:1px
    style G_ADMX fill:#ffffff,stroke:#adb5bd,stroke-width:1px
