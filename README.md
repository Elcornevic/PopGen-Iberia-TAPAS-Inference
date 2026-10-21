# PopGen-Iberia-TAPAS-Inference
Population genomic workflow in UPPMAX for ancient DNA: LD pruning, projected smartpca, and ADMIXTURE/PONG to unmask 25 unknown samples across an 8,000-year Iberian demographic transect based on Olalde et al. (2019).

```mermaid
flowchart TD
    %% Fila Superior: LD Pruning y PCA
    subgraph ROW1[" "]
        direction LR
        subgraph S1["<b>1. LD Pruning & Quality Control</b>"]
            direction TB
            A["<b>COMPLETE_DATASET_geno0.99</b><br/>597,569 SNPs | 844 samples<br/>Call rate: 57.65%"] --> B["<b>PLINK v1.9 Pruning</b><br/><code>--indep-pairwise 10 10 0.5</code><br/>Excludes 126,175 SNPs"]
            B --> C["<b>Pruned Dataset (LD)</b><br/>471,394 independent SNPs<br/><b>Call rate: 68.42%</b>"]
        end

        subgraph S2["<b>2. Conventional PCA (Artifact Detection)</b>"]
            direction TB
            D["<b>PLINK --pca 10</b><br/>Simultaneous SVD on 844 samples<br/>PC1: 24.7% | PC2: 17.1%"] --> E["<b>Severe Shrinkage Bias</b><br/>Missing data pulls aDNA to (0,0)<br/><b>Unprojected PCA rejected</b>"]
        end

        subgraph S3["<b>3. Projected smartpca</b>"]
            direction TB
            F["<b>Modern Reference Scaffold</b><br/>N = 625 moderns define PCs<br/><code>altnormstyle: NO</code> | <code>shrinkmode: YES</code>"] --> G["<b>Least-Squares Projection</b><br/><code>lsqproject: YES</code> (aDNA + TAPAS)<br/>PC1: 0.53% | PC2: 0.31% (West Eurasian map)"]
        end
    end

    %% Fila Inferior: ADMIXTURE y Asignación de Muestras
    subgraph ROW2[" "]
        direction LR
        subgraph S4["<b>4. Ancestry Deconvolution & Alignment</b>"]
            direction TB
            H["<b>ADMIXTURE v1.3.0</b><br/>K = 2 to 6 (3 replicates/K)<br/>10-fold CV (<code>--cv=10</code>)"] --> I["<b>PONG Integration</b><br/>Resolves label switching<br/>Similarity > 0.98 across modes"]
        end

        subgraph S5["<b>5. TAPAS Taxonomic Assignment (Olalde et al. 2019)</b>"]
            direction TB
            J["<b>Neolithic / Chalcolithic (UNK_SP_8, 14–18)</b><br/>High EEF + WHG resurgence | <b>0% Steppe</b>"]
            K["<b>Bronze & Iron Age (UNK_SP_6, 9, 19–22)</b><br/><b>~40% Steppe influx</b> (purple component)"]
            L["<b>Historical / Roman / Islamic (UNK_SP_10–13)</b><br/>East Mediterranean & North African gene flow"]
            M["<b>Modern Controls (UNK_SP_1–5, 7, 24–25)</b><br/>Basque (yellow), Spanish, Finnish, Russian, Turkish"]
        end
    end

    %% Conexiones principales entre etapas
    C --> D
    C --> F
    C --> H
    G --> S5
    I --> S5

    %% Estilos de fuentes grandes y colores
    classDef default font-family:Arial,Helvetica,sans-serif,font-size:14px;
    classDef boxStyle fill:#ffffff,stroke:#333333,stroke-width:1.5px;
    
    style ROW1 fill:none,stroke:none
    style ROW2 fill:none,stroke:none
    style S1 fill:#f8f9fa,stroke:#495057,stroke-width:2px
    style S2 fill:#fff5f5,stroke:#e03131,stroke-width:2px
    style S3 fill:#e7f5ff,stroke:#1971c2,stroke-width:2px
    style S4 fill:#ebfbee,stroke:#2f9e44,stroke-width:2px
    style S5 fill:#fff9db,stroke:#f08c00,stroke-width:2px
