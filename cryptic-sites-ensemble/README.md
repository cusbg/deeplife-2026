# Project: Prediction of Cryptic Binding Sites with Conformational Ensemble Generation and P2Rank

## Introduction

Accurate prediction of protein-ligand binding sites is crucial for drug discovery, protein engineering, and understanding biological functions at the molecular level. Binding site prediction can be performed based on either the three-dimensional structure of a protein or its sequence. Traditionally, structure-based methods have been more commonly used, as binding is governed by the structural and physico-chemical complementarity between the protein and its small molecule binding partner.

However, structure-based approaches face inherent challenges because they are typically trained and evaluated on holo (ligand-bound) structures. In these cases, the ligand is removed, leaving a clear cavity in the protein structure, which is then detected by the prediction method. This creates a bias, as methods trained on such data often struggle to predict pockets in apo (ligand-free) protein forms, where the binding site may be obscured or, conversely, too open for the ligand to bind. Binding sites that exhibit different conformations in apo and holo forms are referred to as cryptic binding sites.

Most existing protein-ligand binding site prediction methods are biased toward detecting holo-binding sites because current benchmarks and training datasets predominantly consist of holo structures. As a result, these methods are often ineffective at identifying cryptic binding sites. To address this limitation, a novel dataset and benchmark called [CryptoBench](https://osf.io/pz4a9/) has been introduced, built upon the [AHoJ-DB](https://apoholo.cz/db) — a comprehensive database of apo-holo protein conformations. CryptoBench focuses on apo-holo protein pairs with significant structural rearrangements in their binding sites, and evaluates methods at the level of individual **binding residues** — i.e., predicting which residues in the apo structure are part of a cryptic binding site.

While residue-level predictions are a natural starting point, many downstream tasks — such as docking a small molecule into a predicted pocket, or scoring the druggability of a site — require the identification of coherent, spatially compact **binding regions** rather than isolated residue labels. This is one part of the gap this project sets out to address.

The other part concerns the input to the prediction method. Structure-based binding site predictors work well when the input structure already exposes the pocket — but in apo proteins, cryptic sites may be fully or partially closed. A single apo structure is therefore a limited and potentially misleading input. A natural remedy is to work not with a single structure but with a **conformational ensemble**: a set of diverse structural conformations sampled from the protein's conformational landscape. Some of these conformations may transiently expose the cryptic pocket, making it detectable by standard structure-based methods.

## Goal

The goal of this project is to predict cryptic binding sites by combining **conformational ensemble generation** with a traditional structure-based pocket predictor. Concretely:

1. For each apo protein in the dataset, generate a conformational ensemble from its sequence using a generative structure prediction model.
2. Run a structure-based binding site predictor on each conformation in the ensemble to obtain candidate pockets.
3. Aggregate the per-conformation pocket predictions into a final ranked list of predicted binding sites for the protein.
4. Evaluate the predicted sites against the ground truth binding sites from the holo structures.

**How the aggregation is performed is entirely up to your team.** Some directions worth exploring:

- *Voting / frequency-based*: rank pockets by how often a similar pocket appears across the ensemble.
- *Clustering across conformations*: cluster pocket centers across all conformations and rank clusters by size or mean score.
- *Score aggregation*: average or take the maximum P2Rank confidence score for matching pockets across the ensemble.

As with the aggregation, **the choice of ensemble generation method and its parameters are open**. The primary recommended tool is [BioEmu](https://github.com/microsoft/bioemu), a generative model that takes a protein sequence and samples diverse structural conformations. Exploring how the number of generated conformations or other BioEmu sampling parameters affects downstream prediction quality is a natural and encouraged direction. Alternative ensemble generation methods that could be explored include [AlphaFlow](https://github.com/bjing2016/alphaflow) and [P2DFlow](https://github.com/bleach366/p2dflow), though these have not been tested by the organizers.

## Evaluation

Predicted binding sites will be evaluated using the following criteria (consistent with the CryptoBench benchmark):

- **DCC (Distance from the Center of the Closest Binding Site)**: measures whether the center of a predicted pocket falls within a given distance threshold of the true binding site center. While 4 Å is commonly used, the [LIGYSIS paper](https://academic.oup.com/nar/article/53/D1/D545/7905468) suggests 12 Å as a more permissive and practical threshold — we adopt 12 Å as our recommended criterion.

  Since a protein can contain multiple binding sites, the DCC criterion is evaluated in a **top-N** or **top-N+2** setting: if a protein has N true binding sites, one tests how many of them are correctly recovered (i.e., matched within the DCC threshold) among the top N or top N+2 predicted pockets, ranked by prediction confidence. This rewards methods that prioritize true sites highly without penalizing reasonable additional predictions.

- **Relative Residue Overlap (RRO)**: measures the fraction of true binding site residues that are covered by the predicted site, normalized by the size of the true site.

Both metrics should be computed per structure and aggregated across the dataset (e.g., success rate at the DCC threshold, mean RRO).

## Optional Extension: Visualization with PyMOL

As an optional output, teams are encouraged to produce visualizations of predicted pockets directly on the protein structures using [PyMOL](https://pymol.org/). Given that this project produces an ensemble of conformations, there is also an opportunity to visualize the conformational diversity of the ensemble itself, and to highlight which conformations expose the predicted pocket. PyMOL can be scripted in Python, making it straightforward to integrate into the pipeline.

## Data Provided

The following resources will be provided by the organizers:

- **Train and test splits** from the CryptoBench benchmark, containing PDB IDs of apo-holo structure pairs along with the corresponding binding site residue annotations (ground truth labels). The full dataset is also available from the [CryptoBench OSF project site](https://osf.io/pz4a9/).

## Useful Resources

- Conformational ensemble generation: [BioEmu](https://github.com/microsoft/bioemu) | [AlphaFlow](https://github.com/bjing2016/alphaflow) | [P2DFlow](https://github.com/bleach366/p2dflow).
- Structure-based binding site prediction: [P2Rank](https://github.com/rdk/p2rank) — P2Rank can be run from the command line and produces ranked pocket predictions with associated residue lists and confidence scores.
- Protein structure handling: BioPython's [Bio.PDB package](https://biopython.org/docs/1.75/api/Bio.PDB.html) — a short tutorial is available [here](https://biopython.org/DIST/docs/tutorial/Tutorial.html#sec240). | [Biotite](https://www.biotite-python.org/latest/)
- CryptoBench benchmark and evaluation: [Paper](https://academic.oup.com/bioinformatics/article/41/1/btae745/7927823) | [OSF dataset](https://osf.io/pz4a9/).
- AHoJ-DB (apo-holo structure pairs): [apoholo.cz/db](https://apoholo.cz/db).
- PyMOL scripting documentation: [PyMOL wiki](https://pymolwiki.org/index.php/Main_Page).
