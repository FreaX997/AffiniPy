flowchart TD

%% ====== TITLE ======
    A0([AffiniPy: Custom Python-Based Virtual Screening Pipeline])

%% ====== STAGE 1: FILESYSTEM SETUP ======
    A1([Filesystem Setup])
    A1 -->|Create folders| A2([Data/, Prep/, Docking/, etc.])

%% ====== STAGE 2: PROTEIN PREPARATION ======
    subgraph S1 [Protein Preparation]
        A3[Input: Protein PDBs]
        A4[pdb2pqr → Protonate at user-defined pH → Output .pqr]
        A5[MDAnalysis → Convert .pqr → .pdbqt]
    end
    A2 --> A3 --> A4 --> A5

%% ====== STAGE 3: LIGAND PREPARATION ======
    subgraph S2 [Ligand Preparation]
        B1[Input: Ligand SDF files]
        B2[Dimorphite-DL → Protonation @ pH]
        B3[RDKit → 3D Conformer Generation (ETKDGv3)]
        B4[MMFF94 Minimization → Select lowest energy conformer]
        B5[Meeko → Convert .sdf → .pdbqt]
    end
    A2 --> B1 --> B2 --> B3 --> B4 --> B5

%% ====== STAGE 4: DOCKING ======
    subgraph S3 [Molecular Docking]
        C1[AutoDock Vina → Dock protein (.pdbqt) with ligand (.pdbqt)]
        C2[Parallel execution via Joblib → Multi-core docking]
        C3[Output: Docking scores & poses (.pdbqt, .log)]
    end
    A5 --> C1
    B5 --> C1
    C1 --> C2 --> C3

%% ====== STAGE 5: DESCRIPTORS & FILTERING ======
    subgraph S4 [Descriptor Calculation]
        D1[RDKit → Physicochemical Descriptors]
        D2[Lipinski’s RO5 & Veber Rule Checks]
        D3[SAScore, Ligand Efficiency calculations]
    end
    C3 --> D1 --> D2 --> D3

%% ====== STAGE 6: COMPOSITE SCORING ======
    subgraph S5 [Composite Scoring & Ranking]
        E1[scikit-learn MinMaxScaler → Normalize scores]
        E2[Weighted composite scoring: Docking + LE + SA + Penalties]
        E3[Sort & rank ligands]
    end
    D3 --> E1 --> E2 --> E3

%% ====== STAGE 7: OUTPUT & VISUALIZATION ======
    subgraph S6 [Result Compilation & Visualization]
        F1[pandas DataFrames → Store results]
        F2[openpyxl → Export to Excel with formatting]
        F3[Color gradients, RO5 highlights]
    end
    E3 --> F1 --> F2 --> F3

%% ====== STAGE 8: BEST POSE GENERATION ======
    subgraph S7 [Best Pose Extraction]
        G1[Extract MODEL 1 from Vina output]
        G2[Open Babel → Convert .pdbqt → .pdb]
        G3[MDAnalysis → Merge ligand + receptor → Complex PDB]
    end
    C3 --> G1 --> G2 --> G3

%% ====== OUTPUT ======
    F3 --> Z1([Final Outputs: Excel Summary, Ranked Ligands, Best Pose PDBs])
    G3 --> Z1
